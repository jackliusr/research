# OpenShift Security Context Constraints (SCCs) — Comprehensive Guide

> **Author:** Jack Liu Shurui — Solution Architect, Crédit Agricole CIB, Singapore
> **Version:** 1.0 | **Applies to:** OpenShift 4.x (4.11+ emphasis)
> **Companion:** `openshift_scc_service_account_guide.md` (SCC-to-SA assignment)

---

## 1. What Are SCCs?

SCCs are OpenShift's **admission-time pod security mechanism**, unique to OpenShift. Evolved from K8s Pod Security Policies (PSP), deprecated in 1.21. They offer RBAC-driven, pod-level control — not just namespace policy but what specific SAs/users can do.

| Aspect | Description |
|---|---|
| Controller | `openshift.io/cluster-policy-controller` |
| Scope | Cluster-wide via RBAC |
| API | `securitycontextconstraints.security.openshift.io/v1` |

**Key vs PSA:** RBAC assignment, priority selection, securityContext default-filling.

---

## 2. How SCCs Work — Admission Flow & Selection

### 2.1 Admission Flow

```
Pod Create/Update
        │
        ▼
┌───────────────────────────────────────┐
│ 1. List ALL SCCs the requesting       │
│    SA/user can access (RBAC match)    │
└──────────────────┬────────────────────┘
                   ▼
┌───────────────────────────────────────┐
│ 2. Generate defaults for unset pod    │
│    securityContext fields             │
└──────────────────┬────────────────────┘
                   ▼
┌───────────────────────────────────────┐
│ 3. Sort SCCs by: priority desc,       │
│    then most restrictive (tiebreak)   │
└──────────────────┬────────────────────┘
                   ▼
┌───────────────────────────────────────┐
│ 4. Validate pod against each SCC      │
│    (privileged, caps, runAsUser,      │
│     SELinux, fsGroup, host*, volumes) │
└──────────────────┬────────────────────┘
                   ▼
         FIRST match → Pod ADMITTED (SCC annotation)
         No match    → Pod REJECTED
```

**Detail:** GATHER from SCC.users[], SCC.groups[], or `system:serviceaccount:<ns>:<sa>` bindings (via `oc adm policy add-scc-to-user`). SORT by priority desc (unset=0). VALIDATE all fields. SELECT first passing.

### 2.2 Priority & Default Ranking

| SCC | Priority | Default (≤4.10) | Default (≥4.11) |
|---|---|---|---|
| restricted-v2 | 0 | — | **Default** |
| restricted | 0 | **Default** | — |
| nonroot-v2 | 0 | — | Available |
| nonroot | 0 | Available | Available |
| anyuid | **10** | — | — |
| hostnetwork-v2 | 0 | — | Available |
| hostnetwork | 0 | Available | Available |
| hostaccess | 0 | — | — |
| hostmount-anyuid | 0 | — | — |
| privileged | 0 | — | — |
| pipelines | 0 | — | Available |

Same priority → most restrictive wins. anyuid (priority 10) evaluated before all priority-0 SCCs.

---

## 3. All Built-in SCCs

### 3.1 Comparison Table

| SCC | Priv | RunAsUser | SELinux | FSGroup | Volumes | HstNet | HstPort | Use Case |
|---|---|---|---|---|---|---|---|---|
| **restricted-v2** | no | MustRunAsRange | MustRunAs | MustRunAs | std* | no | no | Default ≥4.11 |
| **restricted** | no | MustRunAsRange | MustRunAs | RunAsAny | std* | no | no | Default ≤4.10 |
| **nonroot-v2** | no | MustRunAsNonRoot | MustRunAs | MustRunAs | std* | no | no | Non-root |
| **nonroot** | no | MustRunAsNonRoot | MustRunAs | RunAsAny | std* | no | no | Non-root legacy |
| **anyuid** | no | RunAsAny | MustRunAs | RunAsAny | std* | no | no | Any UID/root |
| **hostnetwork-v2** | no | MustRunAsRange | MustRunAs | MustRunAs | std* | yes | yes | Host net |
| **hostnetwork** | no | MustRunAsRange | MustRunAs | RunAsAny | std* | yes | yes | Host net legacy |
| **hostaccess** | no | MustRunAsRange | MustRunAs | RunAsAny | **+hPath** | yes | yes | Net+filesystem |
| **hostmount-anyuid** | no | RunAsAny | MustRunAs | RunAsAny | **+hPath** | no | no | Host mounts |
| **privileged** | **yes** | RunAsAny | RunAsAny | RunAsAny | **all** | yes | yes | Full access |
| **pipelines** | no | RunAsAny | RunAsAny | RunAsAny | std* | no | no | Tekton |

> **std\*** = configMap, downwardAPI, emptyDir, persistentVolumeClaim, projected, secret
> **+hPath** = std* + hostPath | **all** = all types

### 3.2 Descriptions

**restricted-v2** — Default ≥4.11. Random UID from namespace range, no priv containers, no host net, `allowPrivilegeEscalation: false`, fsGroup enforced (MustRunAs). Granted to `system:authenticated`. Aligns K8s **Restricted** PSS.

**restricted** — Legacy ≤4.10. `fsGroup: RunAsAny`, `allowPrivilegeEscalation: true`. Not PSA-aligned.

**nonroot-v2** — MustRunAsNonRoot: `runAsNonRoot: true` or image != UID 0. fsGroup enforced.

**nonroot** — Legacy. `fsGroup: RunAsAny`, `allowPrivilegeEscalation: true`.

**anyuid** — Priority 10. Any UID including root. For workloads needing specific UID. Prefer custom SCC with pinned UID.

**hostnetwork/hostnetwork-v2** — Host namespace + host ports. v2: fsGroup enforced, no priv escalation.

**hostaccess** — Host net + hostPorts + hostPath. Most permissive non-privileged SCC.

**hostmount-anyuid** — hostPath volumes + any UID. No host net. Used by PV recycler.

**privileged** — Full access. Granted to system:admin, nodes, builders. ⚠️ Never grant arbitrarily.

**pipelines** — OpenShift Pipelines (Tekton). RunAsUser: RunAsAny.

### 3.3 Example — restricted-v2

```yaml
apiVersion: security.openshift.io/v1
kind: SecurityContextConstraints
metadata:
  name: restricted-v2
allowPrivilegedContainer: false
requiredDropCapabilities: ["KILL", "MKNOD", "SETUID", "SETGID"]
runAsUser: {type: MustRunAsRange}
seLinuxContext: {type: MustRunAs}
fsGroup: {type: MustRunAs}
supplementalGroups: {type: RunAsAny}
allowHostNetwork: false
allowPrivilegeEscalation: false
volumes: [configMap, downwardAPI, emptyDir, persistentVolumeClaim, projected, secret]
groups: [system:authenticated]
```

### 3.4 Example — privileged

```yaml
apiVersion: security.openshift.io/v1
kind: SecurityContextConstraints
metadata:
  name: privileged
allowPrivilegedContainer: true
allowedCapabilities: ["*"]
allowHostNetwork: true
allowHostPorts: true
allowHostPID: true
allowHostIPC: true
runAsUser: {type: RunAsAny}
seLinuxContext: {type: RunAsAny}
fsGroup: {type: RunAsAny}
volumes: ["*"]
groups: [system:cluster-admins, system:nodes, system:build-controller]
users: [system:admin]
```

---

## 4. SCC Fields Reference

| Field | Type | Values | Description |
|---|---|---|---|
| `allowPrivilegedContainer` | bool | t/f | Allow `securityContext.privileged=true` |
| `allowedCapabilities` | []string | Caps or `["*"]` | Caps containers may request |
| `defaultAddCapabilities` | []string | Caps | Caps added to every container |
| `requiredDropCapabilities` | []string | Caps | Caps **must** be dropped |
| `runAsUser` | object | Strategy | Container UID strategy |
| `seLinuxContext` | object | Strategy | SELinux label strategy |
| `fsGroup` | object | Strategy | Volume GID strategy |
| `supplementalGroups` | object | Strategy | Additional GID strategy |
| `groups` | []string | Names | Groups that can use this SCC |
| `users` | []string | SANames | Users/SAs that can use this SCC |
| `priority` | int | Integer | Evaluation order (higher=first) |
| `volumes` | []string | Types/`["*"]` | Allowed volume types |
| `allowHostNetwork` | bool | t/f | Allow `hostNetwork: true` |
| `allowHostPorts` | bool | t/f | Allow host port binding |
| `allowHostPID` | bool | t/f | Allow `hostPID: true` |
| `allowHostIPC` | bool | t/f | Allow `hostIPC: true` |
| `allowPrivilegeEscalation` | bool | t/f | Allow gaining more privileges |
| `readOnlyRootFilesystem` | bool | t/f | Force read-only rootfs |
| `seccompProfiles` | []string | Profiles | Allowed seccomp profiles |
| `forbiddenSysctls` | []string | Sysctls | Disallowed sysctls |
| `allowedUnsafeSysctls` | []string | Sysctls | Allowed unsafe sysctls |

**Strategy Types:**
- **runAsUser:** `MustRunAs` (UID range), `MustRunAsRange` (project UID range), `MustRunAsNonRoot` (non-root only), `RunAsAny` (no restriction)
- **seLinuxContext:** `MustRunAs` (uses ns MCS labels if unset), `RunAsAny`
- **fsGroup/supplementalGroups:** `MustRunAs` (GID range), `RunAsAny` (no restriction)

---

## 5. SCC Lifecycle

### 5.1 Creating

```bash
oc create -f custom-scc.yaml
```

```yaml
apiVersion: security.openshift.io/v1
kind: SecurityContextConstraints
metadata:
  name: custom-restricted-app
allowPrivilegedContainer: false
allowedCapabilities: [NET_BIND_SERVICE]
requiredDropCapabilities: [KILL, MKNOD, SETUID, SETGID]
runAsUser: {type: MustRunAs, uid: 1001}
seLinuxContext: {type: MustRunAs}
fsGroup: {type: MustRunAs, ranges: [{min: 1001, max: 1001}]}
allowHostNetwork: false
allowPrivilegeEscalation: false
volumes: [configMap, downwardAPI, emptyDir, persistentVolumeClaim, projected, secret]
```

### 5.2 Updating / Deleting

```bash
oc apply -f updated-custom-scc.yaml   # Update
oc delete scc custom-restricted-app   # Delete (custom only)
```

⚠️ Never edit built-in SCCs — operator resets on upgrades. Create custom SCCs instead.

### 5.3 Auditing

```bash
oc get scc                                                          # List
oc describe scc restricted-v2                                        # Describe
oc get pod <pod> -o jsonpath='{.metadata.annotations.openshift\.io/scc}' # Pod SCC
oc get pods -A -o custom-columns='NS:.metadata.ns,NAME:.metadata.name,SCC:.metadata.annotations.openshift\.io/scc' # Cluster audit
oc auth can-i --list -z <sa> -n <ns> | grep securitycontextconstraints # SA's SCCs
oc adm policy who-can use scc privileged                             # SCC users
```

---

## 6. SCC vs Pod Security Admission (PSA)

### 6.1 Dual System (OCP 4.11+)

| Mechanism | Scope | Controller |
|---|---|---|
| **SCC** | Pod-level, RBAC-gated | cluster-policy-controller |
| **PSA** | Namespace-level, label-driven | K8s PodSecurity |

**Flow:** SCC admission (assigns SCC, annotates pod) → PSA check (namespace labels). **SCC takes precedence.**

### 6.2 Automatic PSA Sync

OpenShift auto-sets namespace PSA labels based on bound SCCs:
```bash
oc get ns <ns> -o yaml | grep pod-security
# pod-security.kubernetes.io/enforce: restricted
# security.openshift.io/scc.podSecurityLabelSync: "true"
```

Toggle sync: `oc label ns <ns> security.openshift.io/scc.podSecurityLabelSync=false`

### 6.3 Migration Guidance

| Scenario | Guidance |
|---|---|
| **New (≥4.11)** | restricted-v2 default. Custom SCCs only when needed. |
| **Upgrade ≤4.10→4.11+** | Review custom SCCs. PSA auto-generated. Test before enforce. |
| **Dual enforcement issues** | Relax PSA label OR tighten SCC. |
| **Pure PSA?** | Not possible — SCC mandatory on OpenShift. |

---

## 7. SCC with Pod Security Standards

### 7.1 PSS Mapping

| PSS Level | Default SCC |
|---|---|
| Privileged | privileged |
| Baseline | restricted-v2 |
| Restricted | restricted |

**SCC permissiveness → PSA level:**
```
restricted-v2   → PSS Restricted
nonroot-v2      → PSS Restricted
anyuid          → PSS Baseline
hostnetwork-v2  → PSS Baseline
hostaccess      → PSS Privileged
privileged      → PSS Privileged
```

### 7.2 Warnings

When pod SCC exceeds namespace PSA enforce level:
```
Warning: would violate PodSecurity "restricted:latest": ...
```
Fix: `oc label ns <ns> pod-security.kubernetes.io/enforce=baseline --overwrite`

---

## 8. Troubleshooting SCC Issues

### 8.1 Decision Tree

```
START: Pod stuck with SCC error?
  │
  ├─ 1. Check namespace PSA labels
  │    oc get ns <ns> -o yaml | grep pod-security
  │    If enforce=restricted, try enforce=privileged for testing
  │
  ├─ 2. Check pod's assigned SCC
  │    oc get pod <pod> -o jsonpath='{.metadata.annotations.openshift\.io/scc}'
  │    MISSING → blocked before admission
  │
  ├─ 3. Check SCC priority & available SCCs
  │    oc get scc <scc> -o jsonpath='{.priority}'
  │    oc auth can-i --list -z <sa> -n <ns> | grep scc
  │    Higher priority SCC evaluated first
  │
  ├─ 4. Check pod securityContext vs SCC rules
  │    oc get pod <pod> -o yaml | grep -A15 securityContext
  │    Common violations:
  │    • privileged=true but SCC denies
  │    • runAsUser=0 but no root-allowing SCC
  │    • runAsUser outside namespace UID range
  │    • allowedCaps not in SCC / required caps not dropped
  │    • hostNetwork=true but SCC denies
  │    • volume type not in SCC's allowed list
  │
  └─ 5. Check SA permissions
       oc get rolebinding,clusterrolebinding -A | grep <sa>
       Fix: oc adm policy add-scc-to-user <scc> -z <sa> -n <ns>
```

### 8.2 Common Errors & Fixes

| Error | Root Cause | Fix |
|---|---|---|
| `unable to validate against any security context constraint` | Pod requests features disallowed by ALL available SCCs (privileged=true, runAsUser=0, volume/cap not allowed) | Grant permissive SCC: `oc adm policy add-scc-to-user anyuid -z <sa> -n <ns>` OR remove offending fields from pod spec |
| `unable to find SCC that matches` | SA has NO SCCs available | `oc adm policy add-scc-to-user restricted-v2 -z <sa> -n <ns>` |
| `pod has a higher security level than the namespace allows` | PSA label more restrictive than pod's SCC | Relax PSA: `oc label ns <ns> pod-security.kubernetes.io/enforce=baseline --overwrite` |
| Wrong SCC assigned (priority override) | Two SCCs at priority 0, restrictive wins tiebreaker | `oc patch scc custom-scc -p '{"priority": 10}'` |

### 8.3 Debug Commands

```bash
oc describe pod <pod> | grep -A10 "Events:"
oc get pod <pod> -o jsonpath='{.metadata.annotations.openshift\.io/scc}'
oc auth can-i --list -z <sa> -n <ns> | grep securitycontextconstraints
oc describe scc <scc-name>
oc get ns <ns> -o jsonpath='{.metadata.annotations.openshift\.io/sa\.scc\.uidrange}'
```

---

## 9. Monitoring SCC Usage

### 9.1 Audit Logging

```bash
oc adm node-logs --role=master --path=kube-apiserver/audit.log | grep -i securitycontextconstraints
```
Look for: `responseStatus.code: 403` (rejection), `annotations.openshift.io/scc` (assignment).

### 9.2 Prometheus Metrics

| Metric | Type | Description |
|---|---|---|
| `openshift_scc_admission_total` | Counter | Total evaluations |
| `openshift_scc_admission_rejected_total` | Counter | Rejections |
| `openshift_scc_admission_errors_total` | Counter | Errors |

### 9.3 Monthly Audit

```bash
echo "=== SCC AUDIT ==="
oc get scc
oc get pods -A -o custom-columns='SCC:.metadata.annotations.openshift\.io/scc,NS:.metadata.ns,NAME:.metadata.name' | sort
oc get scc -o json | jq '.items[] | select(.users|length>0) | {name, users}'
oc get scc privileged -o json | jq '{users, groups}'
```

---

## 10. Migration OCP 3.x → 4.x

| Aspect | OCP 3.x | OCP 4.x |
|---|---|---|
| Default SCC | restricted | restricted-v2 (≥4.11) |
| PSA | None | PSA + auto label sync |
| Management | Manual | Operator-reconciled |

**Steps:**
```bash
oc get scc -o yaml > ocp3-scc-backup.yaml    # Export
# Adapt API versions, consider v2 equivalents
oc apply -f ocp3-scc-adapted.yaml             # Apply
oc label ns <ns> security.openshift.io/scc.podSecurityLabelSync=true
```

**Breaking changes:** restricted→restricted-v2 breaks pods with explicit runAsUser outside range (use anyuid). `allowPrivilegeEscalation: false` breaks pods needing priv escalation. `fsGroup: MustRunAs` rejects explicit fsGroup outside range.

---

## 11. SCC in CI/CD & GitOps

### 11.1 Declarative Management

```yaml
# cluster/scc/scc-custom-app.yaml
apiVersion: security.openshift.io/v1
kind: SecurityContextConstraints
metadata:
  name: scc-custom-app
  annotations:
    description: "Custom SCC"
    managed-by: "gitops"
```

Git: `cluster/scc/ ├── scc.yaml ├── kustomization.yaml └── rbac/ └── rolebinding.yaml`

### 11.2 ArgoCD Sync Waves

```yaml
# SCC: wave -2, SA: wave -3, RBAC: wave -1, Deployment: wave 0
metadata:
  annotations: {argocd.argoproj.io/sync-wave: "-2"}
```
Sync options: `Replace=true` for cluster-scoped SCCs. Order: SA → SCC → ClusterRoleBinding → Deployment.

---

## 12. SCC Security Best Practices

### 12.1 Least Privilege Order

```
1. restricted-v2    ← MOST SECURE (default ≥4.11)
2. restricted       ← Former default
3. nonroot-v2       ← Non-root, fsGroup enforced
4. nonroot          ← Non-root legacy
5. anyuid           ← Any UID (root)
6. hostnetwork-v2   ← Host network
7. hostnetwork      ← Host network legacy
8. hostaccess       ← Host net + hostPath
9. hostmount-anyuid ← hostPath + any UID
10. privileged      ← LEAST SECURE
```

**Always start with restricted-v2; grant only what's needed.**

### 12.2 Decision Flow

```
Can run under restricted-v2?
  YES → Use it (remove explicit runAsUser, drop caps)
  NO  → Needs specific UID? → anyuid or custom (pinned UID)
        Needs host network? → hostnetwork-v2
        Needs capabilities? → custom SCC (only those caps)
        Needs privileged?   → security review REQUIRED
```

### 12.3 Checklist

**1. restricted-v2 as default** — ensure `system:authenticated` gets it.

**2. PSA labels for defense-in-depth:**
```bash
oc label ns <ns> pod-security.kubernetes.io/enforce=restricted --overwrite
```

**3. Custom SCCs over privileged:**
```yaml
apiVersion: security.openshift.io/v1
kind: SecurityContextConstraints
metadata:
  name: custom-net-admin
allowPrivilegedContainer: false
allowedCapabilities: [NET_ADMIN]   # Only this capability
# ... restrict everything else
```

**4. Bind to SAs, not users:** `oc adm policy add-scc-to-user anyuid -z my-sa -n my-ns` ✅

**5. RBAC-based assignment** (ClusterRole + RoleBinding) over direct SCC mutation.

**6. Regular audit:**
```bash
oc get scc -o json | jq '.items[] | select(.users|length>0) | {name, users}'
```

**7. Test non-production first:**
```bash
oc label ns dev pod-security.kubernetes.io/enforce=baseline --overwrite   # Dev relaxed
oc label ns prod pod-security.kubernetes.io/enforce=restricted --overwrite # Prod strict
```

**8. Use SCC priority** — set `priority: 20` on custom SCCs that must be selected over others.

**9. Alert on rejections:**
```yaml
- alert: SCCPodRejection
  expr: rate(openshift_scc_admission_rejected_total[5m]) > 0
  for: 5m
  labels: {severity: warning}
```

**10. Document non-default bindings** — record reason, approver, review date per SCC grant.

---

## Appendix A: CLI Quick Reference

```bash
oc get scc                                                             # List
oc describe scc restricted-v2                                           # Describe
oc get scc restricted-v2 -o yaml                                        # Export YAML
oc adm policy add-scc-to-user <scc> -z <sa> -n <ns>                    # Grant SA
oc adm policy remove-scc-from-user <scc> -z <sa> -n <ns>               # Remove
oc auth can-i --list -z <sa> -n <ns> | grep securitycontextconstraints # SA's SCCs
oc adm policy who-can use scc privileged                                # SCC users
oc get pod <pod> -o jsonpath='{.metadata.annotations.openshift\.io/scc}' # Pod SCC
oc get pods -A -o custom-columns='NS:.metadata.ns,NAME:.metadata.name,SCC:.metadata.annotations.openshift\.io/scc' # Audit
oc get ns <ns> -o jsonpath='{.metadata.annotations.openshift\.io/sa\.scc\.uidrange}' # UID range
oc create -f scc.yaml                                                  # Create
oc apply -f scc.yaml                                                   # Update
oc delete scc custom-scc                                                # Delete
oc label ns <ns> security.openshift.io/scc.podSecurityLabelSync=false   # PSA sync
```

---

## Appendix B: Custom SCC — Java App

```yaml
apiVersion: security.openshift.io/v1
kind: SecurityContextConstraints
metadata:
  name: java-app-scc
allowPrivilegedContainer: false
allowedCapabilities: [NET_BIND_SERVICE]
requiredDropCapabilities: [KILL, MKNOD, SETUID, SETGID]
runAsUser: {type: MustRunAs, uid: 1001}
seLinuxContext: {type: MustRunAs}
fsGroup: {type: MustRunAs, ranges: [{min: 1001, max: 1001}]}
supplementalGroups: {type: RunAsAny}
allowHostNetwork: false
allowPrivilegeEscalation: false
volumes: [configMap, downwardAPI, emptyDir, persistentVolumeClaim, projected, secret]
priority: 5
```

```bash
oc create -f java-app-scc.yaml
oc adm policy add-scc-to-user java-app-scc -z java-app-sa -n my-namespace
```

---

## References

- [Red Hat — Managing SCCs (OCP 4.18)](https://docs.redhat.com/en/documentation/openshift_container_platform/4.18/html/authentication_and_authorization/managing-pod-security-policies)
- [Red Hat Blog — Pod Admission & SCCs v2](https://www.redhat.com/en/blog/pod-admission-and-sccs-version-2-in-openshift)
- [Red Hat Blog — OpenShift PSA Changes](https://connect.redhat.com/en/blog/important-openshift-changes-pod-security-standards)
- [K8s Recipes — OpenShift SCC Guide](https://kubernetes.recipes/recipes/security/openshift-scc-security-context-constraints/)
- [K8s — Pod Security Admission](https://kubernetes.io/docs/concepts/security/pod-security-admission/)
