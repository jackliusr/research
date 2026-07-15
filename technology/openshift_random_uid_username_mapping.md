# Dynamically Mapping Random UID to Username in OpenShift

> **Author:** Jack Liu Shurui | **Date:** July 2026  
> **Role:** Solution Architect — Crédit Agricole CIB  
> **Repo:** https://github.com/jackliusr/research

---

## 1. The Problem: Random UIDs in OpenShift

OpenShift is a security-hardened Kubernetes distribution. One of its key
security features: containers run with an **arbitrarily assigned, random UID**
rather than whatever `USER` directive is in the Dockerfile.

This breaks applications that expect a known UID:

- **`/etc/passwd` lookups fail** — `getpwuid()` / `getpwnam()` return no match
  because the random UID has no entry. Tools like `id`, `whoami`, `ls -l`, and
  `ssh` all depend on these lookups.
- **File permission errors** — The runtime UID differs from the build-time UID.
- **NFS mount problems** — `root_squash` remaps UIDs on the server side,
  creating mismatches.
- **Kerberos / GSSAPI failures** — Principal lookup from UID breaks.
- **Java JVM crashes** — The JVM calls `getpwuid()`; a missing entry causes
  `NullPointerException` or `"anonymous uid"` in logs.
- **SSH / Git failures** — Both require a valid passwd entry with shell and
  home directory.

Applies to: application teams, platform engineers, ISV vendors, migrations.

---

## 2. How OpenShift Assigns UIDs

### 2.1 Namespace-Level UID Allocation

When a namespace/project is created, OpenShift allocates three security
identifiers stored as namespace annotations:

| Annotation | Description | Example |
|---|---|---|
| `openshift.io/sa.scc.uid-range` | UID range for the namespace | `1000650000/10000` |
| `openshift.io/sa.scc.supplemental-groups` | Supplemental GID range | `1000650000/10000` |
| `openshift.io/sa.scc.mcs` | SELinux MCS label | `s0:c26,c14` |

The `uid-range` annotation specifies a **start UID and block size**.
`1000650000/10000` means the namespace can use UIDs `1000650000` through
`1000659999`. Every pod receives a UID from this range.

### 2.2 Pod Admission Flow

1. Pod is submitted. The admission controller evaluates all SCCs the pod's
   service account can access.
2. The highest-priority matching SCC is selected (typically `restricted` or
   `restricted-v2`).
3. If the SCC has `MustRunAsRange` for `runAsUser`, OpenShift allocates the
   first available UID from the namespace's `uid-range`.
4. CRI-O (the container runtime) creates the container with the allocated UID.
   On OpenShift 4.2+, CRI-O automatically appends the random UID to
   `/etc/passwd` — but only the UID, no username mapping. On 3.x/early 4.x,
   even the UID entry is absent.

### 2.3 The Restricted SCC (Default)

`restricted` / `restricted-v2` enforces:

- `runAsUser: MustRunAsRange` — UID from namespace range
- `runAsNonRoot: true` — No UID 0
- `seLinuxContext: MustRunAs` — MCS label from namespace
- `fsGroup: MustRunAs` — Supplemental group from namespace range
- `supplementalGroups: MustRunAs` — Groups from namespace range
- `capabilities.drop: ["ALL"]`
- `allowPrivilegeEscalation: false`
- `seccompProfile: RuntimeDefault` (restricted-v2 only)

### 2.4 UID vs Service Account — Key Distinction

The **container process UID** and the **service account identity** are
completely independent:

- **Service account** (`system:serviceaccount:ns:name`) — an OpenShift API
  object for API authentication and RBAC.
- **Container UID** — a Linux process-level identifier assigned by the SCC.
  OpenShift does NOT automatically map the service account name to the
  container UID.

---

## 3. Options for Mapping UID to Username

### Option A: nss_wrapper (Recommended)

**nss_wrapper** (`libnss_wrapper.so`) intercepts NSS calls — `getpwuid()`,
`getpwnam()`, `getgrgid()`, `getgrnam()` — and serves results from
user-defined passwd/group files instead of the system `/etc/passwd`.

**How it works:**
1. Entrypoint creates temp files. Writes runtime UID with a configurable name.
2. `NSS_WRAPPER_PASSWD`, `NSS_WRAPPER_GROUP`, and `LD_PRELOAD=libnss_wrapper.so`
   are set so all process lookups hit the custom files.

**Advantages:**
- Works with **any random UID** — no SCC changes needed
- **Zero filesystem modification** — no writes to `/etc/passwd`
- Works across **all OpenShift versions** (3.x, 4.x+)
- Compatible with **all SCCs** (restricted-v2, nonroot, anyuid)
- Industry standard — Red Hat, IBM Cloud Paks, Apache Spark operator

**Disadvantages:**
- Requires `nss_wrapper` package in the container image
- `LD_PRELOAD` must be exported (applies to all forked processes)
- Some statically-linked binaries ignore `LD_PRELOAD`

#### Basic Snippet

```bash
USER_ID=$(id -u)
GROUP_ID=$(id -g)
export NSS_WRAPPER_PASSWD=/tmp/passwd
export NSS_WRAPPER_GROUP=/tmp/group
echo "appuser:x:${USER_ID}:${GROUP_ID}:App User:/home/appuser:/sbin/nologin" \
  > "${NSS_WRAPPER_PASSWD}"
echo "appuser:x:${GROUP_ID}:" > "${NSS_WRAPPER_GROUP}"
export LD_PRELOAD=libnss_wrapper.so
```

---

### Option B: Pre-create User in Container Image

```dockerfile
RUN useradd -u 1001 -r -g 0 -d /home/appuser -s /sbin/nologin -c "App User" appuser
USER 1001
```

**Limitations:** Only works with non-restricted SCCs. Under `restricted` SCC,
`MustRunAsRange` overrides the build-time UID. Requires granting `anyuid` or a
custom SCC. The fixed UID is not guaranteed to match the runtime UID.

---

### Option C: Security Context Constraints Modification

For applications that absolutely require a fixed UID (databases, licensed
software), create a custom SCC.

#### Custom SCC Example

```yaml
apiVersion: security.openshift.io/v1
kind: SecurityContextConstraints
metadata:
  name: myapp-fixed-uid
allowPrivilegedContainer: false
runAsUser:
  type: MustRunAs
  uid: 1001
seLinuxContext:
  type: MustRunAs
fsGroup:
  type: MustRunAs
  ranges:
    - min: 1001
      max: 1001
supplementalGroups:
  type: MustRunAs
  ranges:
    - min: 1001
      max: 1001
```

```bash
oc create -f custom-scc.yaml
oc adm policy add-scc-to-user myapp-fixed-uid -z myapp-sa -n myproject
```

**Security implications:**
- **Overly permissive SCCs are dangerous** — `anyuid` allows UID 0.
- **Bind to specific service accounts**, never to `system:authenticated`.
- Custom SCCs appear in audit logs — track via `oc describe scc`.

**When to use:** Database containers (PostgreSQL, MySQL), licensed software
with UID binding, SSH containers, privileged ports (<1024).

---

### Option D: Service Account-to-User Mapping via Downward API

OpenShift maps service accounts to virtual users via OAuth. Use the Downward
API to inject the service account name, then pass it to nss_wrapper:

```yaml
env:
  - name: SERVICE_ACCOUNT
    valueFrom:
      fieldRef:
        fieldPath: spec.serviceAccountName
```

```bash
# In entrypoint.sh
SERVICE_ACCOUNT=${SERVICE_ACCOUNT:-appuser}
echo "${SERVICE_ACCOUNT}:x:$(id -u):0:${SERVICE_ACCOUNT}:/home/${SERVICE_ACCOUNT}:/sbin/nologin" > /tmp/passwd
```

Bridges the gap between API-level identity and process-level UID for audit logging.

---

## 4. Comprehensive Solution: nss_wrapper with Environment-Aware Mapping

### 4.1 Dockerfile

```dockerfile
FROM registry.access.redhat.com/ubi8/ubi-minimal:latest

# Install nss_wrapper and gettext (for envsubst)
RUN microdnf install -y nss_wrapper shadow-utils gettext && \
    microdnf clean all

# Create home directory writable by root group (GID 0)
RUN mkdir -p /home/appuser && \
    chown -R 0:0 /home/appuser && \
    chmod -R g=u /home/appuser

COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

# Numeric UID (not username) — OpenShift overrides this under restricted SCC
USER 1001

ENTRYPOINT ["/entrypoint.sh"]
```

**Key points:**
- `chown -R 0:0` / `chmod -R g=u` — root group owns everything; GID 0 grants
  write access to any runtime UID.
- `USER 1001` is **numeric** — avoids failure if named user absent.

### 4.2 Entrypoint Script

```bash
#!/bin/bash
# entrypoint.sh — OpenShift nss_wrapper dynamic UID mapper
# Detects runtime UID, creates passwd/group entries via nss_wrapper,
# optionally fixes volume perms, then exec's CMD.
set -euo pipefail

# Config (override via env vars)
USER_NAME="${USER_NAME:-appuser}"
USER_HOME="${USER_HOME:-/home/appuser}"
USER_SHELL="${USER_SHELL:-/sbin/nologin}"
USER_GECOS="${USER_GECOS:-OpenShift Container User}"
DEBUG="${DEBUG:-false}"

# Detect runtime identity
USER_ID=$(id -u); GROUP_ID=$(id -g)
echo "[entrypoint] Starting as UID=${USER_ID}, GID=${GROUP_ID}"

# Use SA name from Downward API if available
[ -n "${SERVICE_ACCOUNT:-}" ] && USER_NAME="${SERVICE_ACCOUNT}"

# Skip if UID already has a passwd entry (e.g. CRI-O auto-insertion)
if getent passwd "${USER_ID}" > /dev/null 2>&1; then
    echo "[entrypoint] UID ${USER_ID} already mapped"
else
    echo "[entrypoint] Creating nss_wrapper entry for UID ${USER_ID} as ${USER_NAME}"
    export NSS_WRAPPER_PASSWD=/tmp/nss_wrapper_passwd
    export NSS_WRAPPER_GROUP=/tmp/nss_wrapper_group
    echo "${USER_NAME}:x:${USER_ID}:${GROUP_ID}:${USER_GECOS}:\
${USER_HOME}:${USER_SHELL}" > "${NSS_WRAPPER_PASSWD}"
    echo "${USER_NAME}:x:${GROUP_ID}:" > "${NSS_WRAPPER_GROUP}"
    echo "root:x:0:" >> "${NSS_WRAPPER_GROUP}"
    export LD_PRELOAD=libnss_wrapper.so
    [ "${DEBUG}" = "true" ] && echo "[entrypoint] $(id); $(whoami 2>&1)"
fi

# Volume permission fix
if [ -n "${VOLUME_MOUNT:-}" ] && [ -d "${VOLUME_MOUNT}" ] \
   && [ "$(stat -c '%u' "${VOLUME_MOUNT}")" != "${USER_ID}" ]; then
    chown -R "${USER_ID}:0" "${VOLUME_MOUNT}"
fi

echo "[entrypoint] Executing: $@"
exec "$@"
```

### 4.3 Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: myproject
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      serviceAccountName: myapp-sa
      securityContext:
        fsGroup: 0
      containers:
        - name: myapp
          image: myapp:latest
          env:
            - name: USER_NAME
              value: "myapp"
            - name: SERVICE_ACCOUNT
              valueFrom:
                fieldRef:
                  fieldPath: spec.serviceAccountName
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          volumeMounts:
            - name: data
              mountPath: /data
      volumes:
        - name: data
          persistentVolumeClaim:
            claimName: myapp-data
```

### 4.4 Edge Cases and Pitfalls

**UID 0 (Root SCC):** If running under `anyuid` SCC (UID 0), the script
detects root already exists in `/etc/passwd` and skips nss_wrapper. Handle
gracefully:

```bash
if [ "${USER_ID}" = "0" ]; then
    echo "[entrypoint] Running as root — nss_wrapper not required"
fi
```

**UID already in /etc/passwd:** On OpenShift 4.2+, CRI-O may already insert
the UID. The script checks with `getent passwd` and skips.

**Multiple containers sharing volumes:** Use `fsGroup: 0` so volumes are
group-writable by GID 0 (always available to OpenShift pods).

**Static binaries:** Some ignore `LD_PRELOAD`. Fall back to writing directly
to `/etc/passwd` (requires `chmod g=u /etc/passwd` at build time).

**Missing home directory:** Ensure `USER_HOME` exists and is group-writable:

```dockerfile
RUN mkdir -p /home/appuser && chmod -R g=u /home/appuser
```

---

## 5. Using the Pod's Service Account for Identity

### 5.1 Audit Logging Pattern

Inject service account, pod name, and namespace into the container for
comprehensive audit trails:

```yaml
env:
  - name: SERVICE_ACCOUNT
    valueFrom:
      fieldRef:
        fieldPath: spec.serviceAccountName
  - name: POD_NAME
    valueFrom:
      fieldRef:
        fieldPath: metadata.name
  - name: POD_NAMESPACE
    valueFrom:
      fieldRef:
        fieldPath: metadata.namespace
```

Application logs now include:
```
[2026-07-15T10:30:00Z] [system:serviceaccount:myproject:myapp-sa]
  Pod myapp-7d9f8c6b4-xk3lz — action completed
```

### 5.2 Combining Service Account with Filesystem Access

Use `fsGroup` for volume access. OpenShift assigns the first GID from the
namespace's supplemental group range when the SCC uses `MustRunAs`. Volumes
are chowned to this GID at mount time.

---

## 6. Common Scenarios and Solutions

| Scenario | Problem | Solution |
|---|---|---|
| **NFS mount** | UID mismatch, squashed to `nobody` | `no_root_squash` export. Allow namespace UID range. nss_wrapper for display. |
| **PV write with restricted SCC** | Random UID can't write to PV owned by UID 1001 | `fsGroup` in pod spec. Group-writable by GID 0. Init container `chown`. |
| **PostgreSQL** | Expects UID 26 (`postgres`) | Official image with nss_wrapper. Custom SCC with fixed UID 26. |
| **Java principal check** | NPE / `"anonymous uid"` | nss_wrapper. Or `-Duser.name=appuser`. |
| **SSH containers** | Needs valid passwd entry | Not feasible under restricted. Custom SCC + fixed UID. |
| **Kerberos auth** | Principal lookup fails | Map SA to Kerberos principal externally. nss_wrapper + annotations. |
| **Git operations** | "please tell me who you are" | nss_wrapper for `getpwuid`. Set `GIT_COMMITTER_NAME`. |
| **Maven/Ivy cache** | `~/.m2` under random UID home | Set `HOME` to writable dir. Group-writable `/home/appuser`. |
| **Apache Spark** | Executor NPE on UID lookup | Spark operator entrypoint has nss_wrapper. Set env vars. |
| **Multi-container pods** | Different UIDs can't share files | `fsGroup: 0`. All pods run with GID 0. |
| **IBM Cloud Paks** | Compliance needs UID mapping | Base images include nss_wrapper. `USER_NAME` in Helm values. |

### 6.1 Init Container Pattern for Volume Permissions

```yaml
initContainers:
  - name: volume-permissions
    image: registry.access.redhat.com/ubi8/ubi-minimal:latest
    command: ["/bin/sh", "-c", "chown -R 1000650003:0 /data"]
    volumeMounts:
      - name: data
        mountPath: /data
    securityContext:
      runAsUser: 0
```

> Init container must run as root (UID 0) to chown, requiring an SCC that
> allows `runAsUser: 0`. Alternatively, make volumes group-writable at the
> storage layer.

### 6.2 GID 0 (Root Group) Pattern

OpenShift ensures all containers run with GID 0 as primary or supplemental
group. This is the foundation of the filesystem model:

```dockerfile
RUN mkdir -p /data && chown -R 0:0 /data && chmod -R g=u /data
RUN chmod g=u /etc/passwd /etc/group
```

`g=u` copies owner permissions to group. Since OpenShift grants GID 0,
the runtime UID can write to these files.

---
## 7. References and Best Practices

### 7.1 Official Documentation

- **A Guide to OpenShift and UIDs** — https://www.redhat.com/en/blog/a-guide-to-openshift-and-uids
- **Modification of /etc/passwd** — https://access.redhat.com/articles/4859371
- **Managing SCCs (OCP 4.12)** — https://docs.redhat.com/documentation/openshift_container_platform/4.12/html/authentication_and_authorization/managing-pod-security-policies
- **Creating Images Guidelines** — https://docs.openshift.com/enterprise/3.1/creating_images/guidelines.html
- **Managing SCCs in OpenShift** — https://www.redhat.com/en/blog/managing-sccs-in-openshift
- **Pod Admission and SCCs V2** — https://www.redhat.com/en/blog/pod-admission-and-sccs-version-2-in-openshift

### 7.2 Community Resources

- **nss_wrapper** — https://cwrap.org/nss_wrapper.html
- **IBM SCC Tutorial** — https://developer.ibm.com/learningpaths/secure-context-constraints-openshift/scc-tutorial/
- **Spark Operator entrypoint.sh** — https://github.com/kubeflow/spark-operator/blob/master/entrypoint.sh
- **OpenShift Cookbook** — https://cookbook.openshift.org/users-and-role-based-access-control/how-can-i-enable-an-image-to-run-as-a-set-user-id.html
- **Docker Hardened Images** — https://docs.docker.com/guides/dhi-openshift/

### 7.3 Best Practices Summary

1. **nss_wrapper as default** — works with all SCCs, no cluster-admin needed.
2. **Design for GID 0** — `chmod g=u` on writable dirs. OpenShift always
   grants GID 0, ensuring write access regardless of random UID.
3. **`USER` as numeric UID** in Dockerfiles — named users may not exist in
   `/etc/passwd`.
4. **Use the Downward API** — inject service account, pod name, namespace.
5. **Custom SCCs over `anyuid`** — grant only specific UIDs, bind to SA.
6. **`fsGroup: 0`** for pods accessing PVs — ensures write via GID 0.
7. **Test with `restricted-v2`** — if it runs here, it runs under any SCC.
8. **Init containers for volume perms** — don't run main container as root.
9. **Document the UID strategy** — range, SCC bindings, troubleshooting.
10. **12-Factor App principles** — UID is ephemeral, never hardcode.

### 7.4 Onboarding Checklist

- [ ] nss_wrapper installed in image
- [ ] Entrypoint creates passwd/group for runtime UID
- [ ] `LD_PRELOAD=libnss_wrapper.so` exported
- [ ] `USER` in Dockerfile uses numeric UID
- [ ] Writable dirs owned by GID 0 with `chmod g=u`
- [ ] `fsGroup: 0` in pod security context
- [ ] Downward API injects service account name
- [ ] No hardcoded UID assumptions in code
- [ ] Tested under `restricted-v2` SCC
- [ ] NFS exports allow namespace UID range (if using NFS)

---

> **Contributing:** Corrections and additions welcome via
> https://github.com/jackliusr/research.
