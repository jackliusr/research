# Assigning Security Context Constraints (SCC) to Service Accounts in OpenShift

> **Author:** Jack Liu Shurui | **Role:** Solution Architect, Crédit Agricole CIB, Singapore
> **Repo:** https://github.com/jackliusr/research | **Category:** technology/ | **OCP Versions:** 4.x (4.11+ emphasis)

---

## 1. What Are SCCs?

Security Context Constraints (SCCs) are OpenShift's security boundary mechanism for controlling pod permissions. They define what a pod is allowed to do at runtime — which capabilities it can use, what UID it runs as, whether it can access host resources, and which volume types it can mount. SCCs are OpenShift's successor to Kubernetes Pod Security Policies (deprecated in K8s 1.21, removed in 1.25). They remain a first-class API in OpenShift.

**Core SCC fields and what they control:**

| Field | Controls |
|---|---|
| `allowPrivilegedContainer` | Whether pods can run in privileged mode |
| `allowedCapabilities` / `defaultAddCapabilities` | Linux capabilities containers may request or get by default |
| `requiredDropCapabilities` | Capabilities forcibly removed from every container |
| `runAsUser` | Which UIDs the container may run as |
| `seLinuxContext` | SELinux context strategy (MustRunAs, RunAsAny, etc.) |
| `fsGroup` / `supplementalGroups` | Volume ownership and group IDs |
| `allowHostNetwork` / `allowHostPorts` | Host network namespace access |
| `allowHostPID` / `allowHostIPC` | Host PID and IPC namespace access |
| `volumes` | Volume types the pod may use |
| `readOnlyRootFilesystem` | Enforce read-only root filesystem |
| `seccompProfiles` | Allowed seccomp profiles |
| `priority` | Priority ranking when multiple SCCs match (higher = preferred) |

SCCs are **cluster-scoped** resources. They are assigned to pods **through the pod's service account (SA)**. When a pod is created, the SCC admission controller finds all SCCs the pod's SA can access via RBAC, sorts them by priority, and picks the first one that validates the pod's security context.

---

## 2. Built-in SCCs

OpenShift ships with these built-in SCCs (set varies by OCP version):

### 2.1 `restricted` (pre-4.11 default)
- **Granted to:** All authenticated users
- **Privileged:** No | **Host features:** All denied
- **RunAsUser:** `MustRunAsRange` (namespace-allocated UID)
- **SELinux:** `MustRunAs` | **FSGroup:** `MustRunAs`
- **Capabilities:** All dropped, none allowed
- **Volumes:** configMap, downwardAPI, emptyDir, persistentVolumeClaim, projected, secret
- **Use:** General-purpose workloads on pre-4.11 clusters

### 2.2 `restricted-v2` (default on OCP 4.11+ fresh installs)
- **Granted to:** All authenticated users (on fresh 4.11+)
- **Difference from `restricted`:** Enforces K8s PSS `restricted` profile; requires `seccompProfile: runtime/default` or unset (SCC defaults it)
- **Use:** New default for all workloads without elevated needs

### 2.3 `nonroot` / `nonroot-v2`
- **Granted to:** All authenticated users
- **RunAsUser:** `MustRunAsNonRoot` — refuses any image/pod running as root
- **Use:** Workloads that must run non-root but need a defined UID range. `nonroot-v2` (4.11+) aligns with PSS `baseline`

### 2.4 `anyuid`
- **Granted to:** Not by default (must be explicitly assigned)
- **RunAsUser:** `RunAsAny` — allows any UID including root (0)
- **Host features:** Denied
- **Use:** Legacy apps, middleware images, OpenShift Pipelines SA

### 2.5 `hostaccess`
- **Granted to:** Not by default
- **Host features:** Allows host network, ports, PID, IPC
- **RunAsUser:** `MustRunAsRange`
- **Use:** Admin pods needing full host-level access

### 2.6 `hostnetwork` / `hostnetwork-v2`
- **Granted to:** Not by default
- **Host network/ports:** Allowed | **RunAsUser:** `MustRunAsRange`
- **Use:** Monitoring agents, node networking. `hostnetwork-v2` (4.11+) requires `seccompProfile: runtime/default`

### 2.7 `hostmount-anyuid`
- **Granted to:** Not by default
- **Host dir volumes:** Allowed | **RunAsUser:** `RunAsAny`
- **Use:** Pods needing host filesystem access as root (node agents, CSI drivers)

### 2.8 `privileged`
- **Granted to:** Not by default
- **Privileged:** Yes | **Host features:** All allowed
- **RunAsUser/SELinux/FSGroup:** `RunAsAny`
- **Capabilities/Volumes:** All
- **Use:** CSI drivers, SDN/network plugins, operators. **Extreme caution required.**

### 2.9 `pipelines` SCC (OCP Pipelines / Tekton)
- **Granted to:** Created by OpenShift Pipelines operator; auto-bound to `pipeline` SA in Tekton-enabled namespaces
- **RunAsUser:** `RunAsAny`
- **Host features:** Denied
- **Use:** Allows Tekton pipeline runs to build/push container images

---

## 3. Default SCC Assignment

When a pod is created: (1) the admission controller retrieves all SCCs the pod's SA can use via RBAC, (2) generates defaults for unset security context fields, (3) sorts SCCs by priority (higher wins), (4) validates against each in order — first valid match is used, (5) if none validates, the pod is rejected.

**Default behavior on OCP 4.11+ fresh install:** All authenticated users and SAs can use `restricted-v2`. Pods using the `default` SA run under `restricted-v2`.

**On upgraded clusters** (from 4.10 or earlier): The legacy `restricted` SCC may still be the default. The `restricted-v2` SCC is not auto-granted to `system:authenticated` on upgrades. Always verify.

> Even with `restricted-v2` as the default on fresh installs, it is **not** automatically available to all SAs in all scenarios. Upgraded clusters and clusters with custom RBAC may differ.

---

## 4. Why Assign an SCC to a Service Account

Assign an explicit SCC when the workload needs more than `restricted`/`restricted-v2` provides:

| Scenario | SCC Needed | Reason |
|---|---|---|
| Container runs as root (UID 0) | `anyuid` or custom | `restricted-v2` forces namespace-allocated UID |
| Privileged mode required | `privileged` or custom | Docker-in-Docker, kernel ops |
| Bind host ports (80, 443) | `hostnetwork` or custom | Ingress, monitoring agents |
| Mount host filesystem paths | `hostmount-anyuid` or custom | Logging agents (Fluentd, Filebeat), CSI drivers |
| Full node access | `privileged` | SDN, storage drivers |
| Pipeline builds/pushes images | `anyuid` or `pipelines` | Buildah/kaniko need root |
| OAuth proxy sidecar | Custom SCC with specific caps | Needs `NET_BIND_SERVICE` but not full anyuid |

**Key principle:** Never grant `privileged` when a more restrictive SCC suffices.

---

## 5. How to Assign SCCs to Service Accounts

All commands require **cluster-admin** privileges.

### 5.1 Granting SCCs (cluster-admin required)

**To a single service account:**
```bash
oc adm policy add-scc-to-user anyuid -z my-sa -n my-ns
oc adm policy add-scc-to-serviceaccount anyuid my-sa -n my-ns    # alias, same effect
```

**To a human user:**
```bash
oc adm policy add-scc-to-user anyuid john.doe
```

**To an entire group (e.g., all SAs in a namespace):**
```bash
oc adm policy add-scc-to-group anyuid system:serviceaccounts:my-ns
oc adm policy add-scc-to-group anyuid system:authenticated       # cluster-wide — cautious
oc adm policy add-scc-to-group privileged my-ops-group           # LDAP group
```

### 5.2 Removing assignments
```bash
oc adm policy remove-scc-from-user anyuid -z my-sa -n my-ns
oc adm policy remove-scc-from-group anyuid system:serviceaccounts:my-ns
```

### 5.3 Difference between the commands
| Command | Target | Use Case |
|---|---|---|
| `add-scc-to-user` | Single user or SA via `-z <sa>` | Individual identity grants |
| `add-scc-to-group` | Group name (e.g., `system:serviceaccounts:<ns>`) | Bulk/broad grants |
| `add-scc-to-serviceaccount` | Alias for `add-scc-to-user -z <sa>` | Quick single-SA assignment |

### 5.4 Viewing current assignments
```bash
oc describe scc anyuid                                          # users/groups in the SCC
oc get scc anyuid -o json | jq '.users, .groups'                # specific fields
oc adm policy who-can use scc privileged -n my-ns               # who can use an SCC
```

---

## 6. Creating a Custom SCC

### 6.1 Example: `monitoring-agent-scc`

This SCC grants host network access and specific capabilities but **not** privileged mode:

```yaml
apiVersion: security.openshift.io/v1
kind: SecurityContextConstraints
metadata:
  name: monitoring-agent-scc
  annotations:
    kubernetes.io/description: >
      Allows host network/ports with runAsAny for UID,
      but denies privileged containers and host PID/IPC.
allowHostDirVolumePlugin: false
allowHostIPC: false
allowHostNetwork: true
allowHostPID: false
allowHostPorts: true
allowPrivilegeEscalation: true
allowPrivilegedContainer: false
allowedCapabilities:
  - SYS_PTRACE
  - NET_ADMIN
  - NET_RAW
requiredDropCapabilities:
  - KILL
  - MKNOD
  - SETUID
  - SETGID
fsGroup:
  type: MustRunAs
  ranges:
    - min: 1001
      max: 1001
readOnlyRootFilesystem: false
runAsUser:
  type: MustRunAsRange
seLinuxContext:
  type: MustRunAs
supplementalGroups:
  type: MustRunAs
  ranges:
    - min: 1001
      max: 1001
volumes:
  - configMap
  - downwardAPI
  - emptyDir
  - persistentVolumeClaim
  - projected
  - secret
priority: 10
seccompProfiles:
  - runtime/default
```

### 6.2 Apply and grant
```bash
oc apply -f monitoring-agent-scc.yaml
oc adm policy add-scc-to-user monitoring-agent-scc -z monitoring-sa -n monitoring
```

### 6.3 Declarative (RBAC) approach for GitOps
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: scc:monitoring-agent-scc
rules:
  - apiGroups: [security.openshift.io]
    resourceNames: [monitoring-agent-scc]
    resources: [securitycontextconstraints]
    verbs: [use]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: monitoring-sa-scc-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: scc:monitoring-agent-scc
subjects:
  - kind: ServiceAccount
    name: monitoring-sa
    namespace: monitoring
```
```bash
oc apply -f monitoring-scc-rbac.yaml
```

This approach is version-controllable and auditable — prefer it for production.

---

## 7. Assigning to a Specific Service Account

**Step 1:** Create the SA if it does not exist.
```bash
oc create sa my-app-sa -n my-app-ns
```

**Step 2:** Grant the SCC.
```bash
oc adm policy add-scc-to-user anyuid -z my-app-sa -n my-app-ns
```

**Step 3:** Configure the pod/deployment to use the SA.
```bash
oc set serviceaccount deployment/my-app my-app-sa -n my-app-ns
```
Or in YAML:
```yaml
spec:
  template:
    spec:
      serviceAccountName: my-app-sa
```

**Step 4:** Verify (see [Section 9](#9-verifying-scc-assignment)).

---

## 8. Assigning to All Service Accounts in a Namespace

Grant an SCC to every SA in a namespace using the virtual group `system:serviceaccounts:<namespace>`:
```bash
oc adm policy add-scc-to-group anyuid system:serviceaccounts:monitoring
```

**Implications:** Every pod in the namespace gets the elevated SCC, including pods from operators or CI/CD pipelines that create new SAs. Use this only when **all** workloads in the namespace legitimately need the SCC. Otherwise, grant to individual SAs.

**Cluster-wide grant (highly permissive — rarely appropriate):**
```bash
oc adm policy add-scc-to-group privileged system:serviceaccounts
```

---

## 9. Verifying SCC Assignment

### 9.1 Check which SCCs an SA can use
```bash
oc adm policy who-can use scc -n my-ns --sa=my-sa
```

### 9.2 Check which SCC a pod is using
The assigned SCC is stored as a pod annotation:
```bash
oc get pod my-pod -o jsonpath='{.metadata.annotations.openshift\.io/scc}{"\n"}'
oc describe pod my-pod | grep -i scc

# All pods in a namespace
oc get pods -n my-ns -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.metadata.annotations.openshift\.io/scc}{"\n"}{end}'
```

### 9.3 SCC details and PSA labels
```bash
oc describe scc anyuid
oc get scc -o custom-columns=NAME:.metadata.name,PRIORITY:.priority,PRIVILEGED:.allowPrivilegedContainer
oc get ns my-ns -o yaml | grep pod-security
```

---

## 10. Common Use Cases

### 10.1 OpenShift Pipelines SA
```bash
oc adm policy add-scc-to-user anyuid -z pipeline -n my-ns
```
If a dedicated `pipelines` SCC exists on your cluster (`oc get scc pipelines`), use it instead.

### 10.2 Monitoring agents needing host network
```bash
oc adm policy add-scc-to-user hostnetwork -z node-exporter -n monitoring
```

### 10.3 CSI storage drivers (privileged)
```bash
oc adm policy add-scc-to-user privileged -z csi-driver-sa -n openshift-storage
```
Many CSI operators define their own SCC (e.g., HPE CSI with SYS_ADMIN, SYS_RAWIO, RunAsAny).

### 10.4 Logging agents needing host filesystem
```bash
oc adm policy add-scc-to-user hostmount-anyuid -z fluentd-sa -n openshift-logging
```
Better: create a custom SCC that allows specific hostPath volumes without full `hostmount-anyuid`.

### 10.5 OAuth proxy sidecar
Grant `NET_BIND_SERVICE` via a custom SCC or use `anyuid` as minimal escalation. Custom SCC template:
```yaml
apiVersion: security.openshift.io/v1
kind: SecurityContextConstraints
metadata:
  name: oauth-proxy-scc
allowPrivilegedContainer: false
allowedCapabilities: [NET_BIND_SERVICE]
runAsUser: {type: MustRunAsRange}
seLinuxContext: {type: MustRunAs}
volumes: [configMap, secret, downwardAPI]
```

### 10.6 Legacy images needing root but not host access
```bash
oc adm policy add-scc-to-user anyuid -z legacy-app-sa -n legacy-ns
```
Best practice: fix the Dockerfile to run as non-root instead.

---

## 11. RBAC Considerations

### 11.1 Direct assignment vs RBAC binding
`oc adm policy add-scc-to-user` modifies the SCC's `users` array directly (not GitOps-friendly). ClusterRole + ClusterRoleBinding uses standard K8s RBAC (GitOps-friendly, auditable). **Prefer the RBAC approach for production.**

### 11.2 Cluster-admin requirement
Only cluster-admins can create/modify/delete SCCs or grant SCC `use` permission. Namespace admins **cannot** — this prevents privilege escalation by design.

### 11.3 Built-in SCC cluster roles
OpenShift provides aggregate ClusterRoles: `system:openshift:scc:{anyuid,privileged,restricted,restricted-v2,hostnetwork,hostaccess,hostmount-anyuid,nonroot}`. Bind declaratively:
```bash
oc create clusterrolebinding my-app-scc-binding \
  --clusterrole=system:openshift:scc:anyuid \
  --serviceaccount=my-ns:my-sa
```

### 11.4 Verification
```bash
oc adm policy who-can use scc anyuid --sa=my-sa -n my-ns
oc get clusterrolebindings -o wide | grep scc
```

---

## 12. Troubleshooting

### 12.1 "unable to validate against any security context constraint"
The pod's security context does not match any SCC the SA can use.
```bash
# Identify SA → Check SCCs available → Inspect securityContext → Check events → Grant SCC
oc get pod my-pod -o jsonpath='{.spec.serviceAccountName}'
oc describe scc | grep -B2 "$(oc get pod my-pod -o jsonpath='{.spec.serviceAccountName}')"
oc get pod my-pod -o yaml | grep -A5 securityContext
oc get events --sort-by='.lastTimestamp' | grep -i scc
oc adm policy add-scc-to-user <scc> -z <pod-sa> -n <ns>
```

### 12.2 "pod has a higher security level than the namespace allows"
Namespace PSA labels are more restrictive than the pod's SCC.
```bash
oc get ns my-ns -o yaml | grep pod-security
oc label ns my-ns pod-security.kubernetes.io/enforce=privileged --overwrite
```

### 12.3 "Forbidden: not usable by user or service account"
The SCC exists but the SA lacks `use` permission. Check SA name and namespace:
```bash
oc get sa -n <ns>                      # Verify SA exists
oc describe scc <scc> | grep -E 'Users|Groups'   # Check SCC assignment
```

### 12.4 Inspect which SCC a pod is using
```bash
oc get pod my-pod -o jsonpath='{.metadata.annotations.openshift\.io/scc}'
oc describe pod my-pod | grep -i scc
oc get events -n my-ns --field-selector involvedObject.kind=Pod
```

### 12.5 Common error messages reference
| Error | Likely Cause | Solution |
|---|---|---|
| `unable to validate against any security context constraint` | No matching SCC | Grant appropriate SCC to pod's SA |
| `not usable by user or service account` | SA lacks `use` permission | `oc adm policy add-scc-to-user <scc> -z <sa> -n <ns>` |
| `pod has a higher security level than the namespace allows` | PSA label too strict | Relax PSA label or restrict pod SCC |
| `Privileged containers are not allowed` | Pod requests `privileged: true` | Grant `privileged` SCC or remove `privileged: true` |
| `hostNetwork: true is not allowed` | Pod requests host network | Grant `hostnetwork` SCC |
| `runAsUser must be set` | Pod missing `runAsUser` | Set `runAsUser` or use SCC with `RunAsAny` |
| `seccompProfile: Invalid value` | `restricted-v2` requires `RuntimeDefault` | Set `seccompProfile.type: RuntimeDefault` or leave unset |

### 12.6 Debugging checklist
1. Identify pod SA: `oc get pod <name> -o jsonpath='{.spec.serviceAccountName}'`
2. Inspect security contexts (pod + container)
3. Check events: `oc get events --sort-by='.lastTimestamp' | tail -20`
4. Verify SA exists: `oc get sa -n <ns>`
5. Check SCC `.users` / `.groups` for matches
6. Check PSA labels: `oc get ns <ns> -o yaml | grep pod-security`
7. Check OCP version: `oc version`
8. Capture full pod YAML *before* deleting

---

## 13. Best Practices

### 13.1 Principle of least privilege

| Instead of | Use | When |
|---|---|---|
| `privileged` | `anyuid` | App needs root but not host access |
| `privileged` | Custom SCC with specific caps | App needs certain Linux capabilities only |
| `hostmount-anyuid` | Custom SCC with specific volumes | App needs specific host paths only |
| Grant to `system:serviceaccounts:<ns>` | Grant to individual SA | Only some workloads need elevation |
| `anyuid` | Fix the Dockerfile to run non-root | When you own the image |

### 13.2 Use `restricted-v2` by default
On OCP 4.11+, `restricted-v2` is the default. It aligns with K8s PSS `restricted`. Only escalate when a workload demonstrably requires it.

### 13.3 Create custom SCCs instead of using `privileged`
The `privileged` SCC disables nearly all security. Create a custom SCC that only allows specific capabilities, denies privileged containers unless required, restricts volume types, and sets `readOnlyRootFilesystem: true` where possible.

### 13.4 Prefer role-based SCC binding over `oc adm policy`
Direct `oc adm policy` mutates the SCC object in-place. Use ClusterRole + ClusterRoleBinding YAML files managed in version control for a GitOps-friendly, auditable approach.

### 13.5 Audit SCC assignments regularly
```bash
# Who can use 'privileged'?
oc get scc privileged -o json | jq '{users: .users, groups: .groups}'
# All pods running with privileged SCC
oc get pods -A -o jsonpath='{range .items[*]}{.metadata.namespace}{"\t"}{.metadata.name}{"\t"}{.metadata.annotations.openshift\.io/scc}{"\n"}{end}' | grep -E '\bprivileged\b'
```

### 13.6 Pin namespace PSA levels
```bash
oc label ns my-ns security.openshift.io/scc.podSecurityLabelSync=false \
  pod-security.kubernetes.io/enforce=privileged \
  pod-security.kubernetes.io/audit=privileged \
  pod-security.kubernetes.io/warn=privileged --overwrite
```

### 13.7 Document every SCC grant
For compliance (SOC 2, PCI-DSS), document: **why** the workload needs it, **what** permissions are required, **who** requested it (ticket reference), and **when** it was last reviewed.

---

## 14. Reference Cheat Sheet

```bash
# Grant SCC to SA
oc adm policy add-scc-to-user <scc> -z <sa> -n <ns>
oc adm policy add-scc-to-serviceaccount <scc> <sa> -n <ns>

# Grant to all SAs in namespace
oc adm policy add-scc-to-group <scc> system:serviceaccounts:<ns>

# Remove from SA
oc adm policy remove-scc-from-user <scc> -z <sa> -n <ns>

# Check which SCC a pod uses
oc get pod <name> -o jsonpath='{.metadata.annotations.openshift\.io/scc}'

# List all pods and their SCCs
oc get pods -A -o jsonpath='{range .items[*]}{.metadata.namespace}{"\t"}{.metadata.name}{"\t"}{.metadata.annotations.openshift\.io/scc}{"\n"}{end}'

# Bind via RBAC (declarative)
oc create clusterrolebinding <name> --clusterrole=system:openshift:scc:<scc> --serviceaccount=<ns>:<sa>

# Check SCC details
oc describe scc <name> ; oc get scc -o custom-columns=NAME:.metadata.name,PRIORITY:.priority,PRIVILEGED:.allowPrivilegedContainer
oc adm policy who-can use scc <scc> --sa=<sa> -n <ns>
```

### SCC selection flow
```
Pod creation → Identify SA → Collect all SCCs SA can use (RBAC) →
Sort by priority (higher first) → Validate against each in order →
First valid match → Admit pod | No match → REJECT with error
```

---

*References: [Red Hat OCP Documentation — Managing SCCs](https://docs.redhat.com/en/documentation/openshift_container_platform/4.x/html/authentication_and_authorization/managing-pod-security-policies), [Red Hat Blog — Managing SCCs in OpenShift](https://www.redhat.com/en/blog/managing-sccs-in-openshift), [Red Hat Blog — Pod Admission and SCCs v2](https://www.redhat.com/en/blog/pod-admission-and-sccs-version-2-in-openshift), [Red Hat Access — Using legacy restricted SCC in 4.11+](https://access.redhat.com/articles/6973044), [K8s Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/)*
