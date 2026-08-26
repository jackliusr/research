# Adding Certificates to Containers: A Practical Guide

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore  
> **Context:** Container Security & Infrastructure — Docker, Kubernetes, OpenShift, Enterprise Banking  
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)  
> **Last Updated:** July 2026

---
## Table of Contents

1. [Why Certificates Matter in Containers](#1-why-certificates-matter-in-containers)
2. [The Container Certificate Landscape](#2-the-container-certificate-landscape)
3. [Best Practices Overview](#3-best-practices-overview)
4. [Methods for Adding CA Certificates](#4-methods-for-adding-ca-certificates)
5. [Special Cases by Runtime and Base Image](#5-special-cases-by-runtime-and-base-image)
6. [OpenShift-Specific Considerations](#6-openshift-specific-considerations)
7. [Kubernetes Certificate Mounting Best Practices](#7-kubernetes-certificate-mounting-best-practices)
8. [Certificate Rotation Strategy](#8-certificate-rotation-strategy)
9. [cert-manager: Automated Certificate Management](#9-cert-manager-automated-certificate-management)
10. [mTLS Between Containers](#10-mtls-between-containers)
11. [Security Best Practices](#11-security-best-practices)
12. [Monitoring Certificate Expiry](#12-monitoring-certificate-expiry)
13. [Incident Response for Certificate Failure](#13-incident-response-for-certificate-failure)
14. [Validation and Testing](#14-validation-and-testing)
15. [Troubleshooting](#15-troubleshooting)
16. [Key Takeaways](#16-key-takeaways)

---

## 1. Why Certificates Matter in Containers

### 1.1 The Trust Problem

A single containerized workload routinely calls an internal API gateway, pulls from a private registry, authenticates against LDAP/AD, and opens TLS connections to databases, brokers, and microservices. Every TLS client must decide one thing: **do I trust the certificate the server presents?** That decision is delegated to a *trust store* — a curated set of root CA certificates the client believes in. When the server's chain leads to a CA in the store, the handshake succeeds; otherwise it fails with `x509: certificate signed by unknown authority` and the connection is refused.

In containers this is more fragile than on a workstation because:

- **Base images ship only public CAs** (Mozilla bundle, ~140 roots); nothing internal is included.
- **Enterprise environments use private/internal CAs.** Banks run their own PKI (AD CS, HSM-backed roots, Venafi/smallstep); internal services are signed by the corporate hierarchy — a container trusting only Mozilla roots cannot talk to them.
- **Images are immutable and ephemeral.** You cannot "click through" a warning inside a container; if the CA is missing, the call simply fails.
- **Runtimes read different stores.** Java uses its own `cacerts`; Python `requests` uses `certifi`; Node reads the OS store; curl/OpenSSL read yet another. A CA added in one place is not trusted everywhere.

### 1.2 The Banking Reality

In a bank, TLS trust is a regulatory and operational requirement, not optional hygiene: internal APIs (payment rails, market data, reference data) are fronted by gateways signed by the bank's PKI; corporate-proxy egress (internet, SaaS) requires trusting the proxy's MITM CA; private registries (Harbor, Nexus, Artifactory), database TLS (Oracle, PostgreSQL, SQL Server), LDAP/AD TLS, and Kafka/MQ brokers all present internal certificates; and compliance (MAS TRM, BCBS 239, SOC 2) expects documented, controlled trust management — not ad-hoc `NODE_TLS_REJECT_UNAUTHORIZED=0` hacks.

The cost of getting this wrong is not a red banner — it is **service outage**: handshake failures cascade through the dependency graph, health checks fail, deployments roll back.

### 1.3 What Happens Without Proper CA Trust

| Symptom | Meaning |
|---|---|
| `x509: certificate signed by unknown authority` | Server's CA is not in the client's trust store |
| `SSL certificate problem: unable to get local issuer certificate` | curl/wget cannot find the issuer CA in its bundle |
| `PKIX path building failed: unable to find valid certification path` | Java truststore is missing the CA or an intermediate |
| `certificate verify failed` | Generic OpenSSL verification failure |
| Handshake failures on some services but not others | Store mismatch — some runtimes trust the CA, others don't |

### 1.4 Common Scenarios

| Scenario | What the container needs | Typical fix |
|---|---|---|
| Call internal API gateway over TLS | Trust internal CA chain (root + intermediates) | Add CA to OS store and app stores |
| Pull from private registry | Trust registry's CA (Docker/containerd/podman) | Add CA to image + container runtime trust |
| Traverse corporate proxy | Trust proxy MITM CA | Add CA to OS store and app stores |
| Connect to LDAP/AD, DB over TLS | Trust AD CS / DB CA | Add CA to OS store or app-specific store |
| Container *exposes* a TLS service | Server certificate + private key | Key via Secret; cert via Secret/ConfigMap |
| Service-to-service mTLS | Client cert + key + CA | Mount client cert from Secret; mesh or SPIFFE |
| Call public internet APIs | Public CA bundle | Already in base image — verify `ca-certificates` is installed |
| Java/Node/Python workload | Runtime-specific trust | cacerts / NODE_EXTRA_CA_CERTS / certifi |

---

## 2. The Container Certificate Landscape

### 2.1 What Base Images Ship With

Official base images include the distro's `ca-certificates` package, which packages the **Mozilla public CA bundle** (~140 roots):

- **Ubuntu/Debian:** bundle at `/etc/ssl/certs/ca-certificates.crt` (one concatenated PEM file).
- **Alpine:** bundle at `/etc/ssl/certs/ca-certificates.crt`; the package must be installed explicitly (`apk add ca-certificates`).
- **RHEL/UBI/CentOS/Fedora:** bundle at `/etc/pki/tls/certs/ca-bundle.crt`, a symlink into `/etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem`.
- **Distroless (Google):** includes the bundle at `/etc/ssl/certs/ca-certificates.crt` — but **no package manager** and no update tooling; **scratch** has nothing at all, so you must add every byte of trust yourself.

**None of these trust internal CAs.** Adding internal trust is always your job.

### 2.2 Where Certificates Live on Linux

| Distro family | Trust anchor directory | Bundle file | Update command |
|---|---|---|---|
| Debian / Ubuntu | `/usr/local/share/ca-certificates/*.crt` | `/etc/ssl/certs/ca-certificates.crt` | `update-ca-certificates` |
| Alpine | `/usr/local/share/ca-certificates/*.crt` | `/etc/ssl/certs/ca-certificates.crt` | `update-ca-certificates` |
| RHEL / CentOS / Fedora / UBI | `/etc/pki/ca-trust/source/anchors/*.crt` | `/etc/pki/tls/certs/ca-bundle.crt` | `update-ca-trust extract` |
| NSS-based apps | `/etc/pki/nssdb` (SQLite DB) | N/A (DB, not file) | `certutil -A` |
| Java apps | `$JAVA_HOME/lib/security/cacerts` | N/A (keystore) | `keytool -importcert` |

### 2.3 Different Runtimes Read Different Stores

This is the single most important concept in this guide: **"the trust store" does not exist. There are many, and they are independent.**

| Consumer | Default trust source | How to add internal CA |
|---|---|---|
| OpenSSL / curl / wget / git | OS store | `update-ca-*`, or `SSL_CERT_FILE`, `--cacert`, `http.sslCAInfo` |
| Go (crypto/x509) | OS store | `update-ca-*` or `SSL_CERT_FILE` / `SSL_CERT_DIR` |
| Python `ssl` / urllib3 | OS store (via OpenSSL) | `update-ca-*` or `SSL_CERT_FILE` |
| Python `requests` | **certifi bundle** — *not* the OS store | `REQUESTS_CA_BUNDLE` or replace certifi |
| pip | certifi (for index TLS) | `pip --cert` / `PIP_CERT` |
| Node.js | OS store (bundled OpenSSL) | `NODE_EXTRA_CA_CERTS=/path/to/ca.pem` |
| Java (JVM) | `cacerts` truststore — *not* the OS store | `keytool -importcert`, or `-Djavax.net.ssl.trustStore` |
| .NET | OS store on Linux | `update-ca-*` or `SSL_CERT_FILE` |
| NSS apps (some curl builds, browsers) | `/etc/pki/nssdb` | `certutil -A -d sql:/etc/pki/nssdb` |
| Rust (reqwest/rustls) | Bundled `webpki-roots` or OS store | Depends on crate; env var or `rustls-native-certs` |
| Electron | Chromium's store | OS store + `session.setCertificateVerifyProc` |

**The key insight:** if a container runs a Java service calling an internal API, updating the OS store alone is not enough — the JVM still fails with `PKIX path building failed`. You must add the CA to **every store your application stack actually uses**: the OS store, Java `cacerts`, Node's `NODE_EXTRA_CA_CERTS`, Python's certifi, and so on. "curl works, but the app fails" is almost always a store mismatch.

### 2.4 Environment Variables Reference

| Variable | Consumer | Effect |
|---|---|---|
| `SSL_CERT_FILE` | OpenSSL, Go, Python ssl, .NET | Replace default CA bundle file |
| `SSL_CERT_DIR` | OpenSSL, Go, Python ssl | Replace default CA directory |
| `NODE_EXTRA_CA_CERTS` | Node.js | **Append** CAs to defaults (does not replace) |
| `REQUESTS_CA_BUNDLE` | Python requests | Override certifi bundle |
| `CURL_CA_BUNDLE` | curl | Override default bundle |
| `GIT_SSL_CAINFO` | git | Override default bundle |
| `PIP_CERT` | pip | CA bundle for PyPI/index TLS |
| `JAVA_TOOL_OPTIONS` | JVM | Inject `-Djavax.net.ssl.trustStore=...` |
| `AWS_CA_BUNDLE` | AWS SDKs | CA bundle for AWS endpoints |

---

## 3. Best Practices Overview

1. **Use a dedicated CA bundle file in the image.** Copy CA PEMs in explicitly (`COPY internal-ca.crt /usr/local/share/ca-certificates/`) rather than generating or fetching certs during builds.
2. **Update the system trust store with the distro mechanism.** `update-ca-certificates` (Debian/Ubuntu/Alpine) or `update-ca-trust` (RHEL/UBI). Never hand-edit bundle files — these tools manage the symlink farm and bundle regeneration.
3. **Configure application-specific trust stores too.** Java `cacerts`, `NODE_EXTRA_CA_CERTS`, `REQUESTS_CA_BUNDLE` — cover every runtime in the image, not just the OS store.
4. **Never bake private keys into images.** Keys belong in Kubernetes Secrets, Vault, or Docker secrets — image layers are readable by anyone with pull access, and `docker history` exposes them.
5. **Use multi-stage builds.** Curate the certificate layer in a builder stage and COPY it into the final stage, keeping the final image small and free of build tooling.
6. **Prefer mounting certs at runtime over baking when possible.** ConfigMap/Secret volumes make rotation a config change instead of an image rebuild — especially when certs differ per environment.
7. **Document certificate lifecycle.** Every CA/cert needs an owner, an expiry date, a rotation procedure, and a monitored alert. Treat certs as tracked configuration.
8. **Pin CA trust to the minimum set.** Trust only the CAs your workloads need (least privilege). Do not shovel the entire internal PKI into every image.
9. **Sign and verify images.** Image signing (cosign/sigstore) and provenance attestation prevent tampering with trust configuration itself.
10. **Test certificate trust in CI before deployment.** A curl/openssl verification step catches missing CAs before they become production outages.

---

## 4. Methods for Adding CA Certificates

Five established methods exist. They are not mutually exclusive — production systems typically combine build-time baking with runtime mounting.
### 4.1 Method 1 — Bake Into the Image at Build Time

Recommended when: the CA set is stable and shared across environments; the image must be self-contained (air-gapped, exported); or the runtime cannot mount files (scratch, distroless).

**Debian/Ubuntu** — drop CAs into the trust anchor directory (`*.crt` extension required), then regenerate the bundle and symlinks:

```dockerfile
FROM ubuntu:24.04
RUN apt-get update && apt-get install -y --no-install-recommends ca-certificates \
    && rm -rf /var/lib/apt/lists/*
COPY internal-root-ca.crt internal-intermediate.crt \
    /usr/local/share/ca-certificates/
RUN update-ca-certificates
```

**Alpine** — the package must be installed explicitly:

```dockerfile
FROM alpine:3.20
RUN apk add --no-cache ca-certificates
COPY internal-root-ca.crt /usr/local/share/ca-certificates/internal-root-ca.crt
RUN update-ca-certificates
```

**RHEL / UBI:**

```dockerfile
FROM registry.access.redhat.com/ubi9/ubi-minimal:latest
COPY internal-root-ca.crt /etc/pki/ca-trust/source/anchors/internal-root-ca.crt
RUN update-ca-trust extract
```

On RHEL-family images, `/etc/pki/tls/certs/ca-bundle.crt` is a symlink into `/etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem`, regenerated by `update-ca-trust`.

| Pros | Cons |
|---|---|
| Self-contained — works identically anywhere | Rebuild required for every cert rotation |
| Immutable, consistent, reproducible | Slightly larger image (a few KB per CA) |
| No runtime dependency on mounts or init logic | Certs visible in layers / `docker history` |
| Works with scratch, distroless, air-gapped, non-root | One image variant per CA set (or use hybrid) |

### 4.2 Method 2 — Mount at Runtime

Recommended when: certs differ per environment; rotation must not require rebuilds; or a platform team centrally manages trust (typical in enterprise Kubernetes).

**Option A — mount the full bundle file (simplest).** Overwrite the system bundle path with a ConfigMap containing the *complete* bundle (public + internal CAs concatenated):

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: ca-bundle
data:
  ca-certificates.crt: |
    -----BEGIN CERTIFICATE-----
    ... (public CAs from the base image bundle, then internal CAs)
    -----END CERTIFICATE-----
```

```yaml
spec:
  containers:
    - name: app
      volumeMounts:
        - name: ca-bundle
          mountPath: /etc/ssl/certs/ca-certificates.crt
          subPath: ca-certificates.crt
          readOnly: true
  volumes:
    - name: ca-bundle
      configMap:
        name: ca-bundle
        defaultMode: 0444
```
> ⚠️ Mounting over `/etc/ssl/certs/ca-certificates.crt` on Debian/Ubuntu **replaces** the whole bundle — include the public CAs too or public TLS breaks. On RHEL images the path is a symlink, so mount at `/etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem` instead.

**Option B — mount into the trust anchor directory + update in an initContainer** (keeps distro tooling in charge):

```yaml
spec:
  initContainers:
    - name: update-ca-certificates
      image: ubuntu:24.04
      command: ["update-ca-certificates"]
      volumeMounts:
        - name: ca-anchors
          mountPath: /usr/local/share/ca-certificates
        - name: ca-store
          mountPath: /etc/ssl/certs
  containers:
    - name: app
      volumeMounts:
        - name: ca-store
          mountPath: /etc/ssl/certs
```

| Pros | Cons |
|---|---|
| No image rebuild — rotation = ConfigMap update | Runtime dependency on mount availability |
| No certs in image layers | Per-pod configuration (unless cluster-wide injection) |
| Central management by platform team | Apps caching the trust store at startup need a restart after rotation |
| Works with read-only root filesystems | Mounting over system paths can break the base bundle if done wrong |

### 4.3 Method 3 — Hybrid (Bake Common Certs, Mount Environment-Specific Certs)

**Recommended for enterprise/banking platforms.** The image bakes the *global* internal root CA (stable, company-wide, HSM-backed); environment-specific CAs (dev/test/prod intermediates, regional proxies) are mounted at runtime:

```dockerfile
COPY global-root-ca.crt /usr/local/share/ca-certificates/global-root-ca.crt
RUN update-ca-certificates
```

```yaml
spec:
  containers:
    - name: app
      env:
        - name: SSL_CERT_FILE
          value: /etc/ssl/certs/ca-certificates.crt:/run/certs/env-ca-bundle.pem
```

Baking covers the parts that rarely change (reproducibility); mounting covers the parts that do (operational flexibility).

### 4.4 Method 4 — The initContainer Pattern

Useful when: the app runs **non-root** with a **read-only root filesystem** (the default under OpenShift's restricted SCC); the app cannot write to `/etc/ssl/certs`; or you must merge a mounted CA into the OS store without replacing it.

An initContainer runs as root, merges certs into a shared `emptyDir`, and the app container mounts that `emptyDir` over its trust path:

```yaml
spec:
  initContainers:
    - name: cert-init
      image: registry.access.redhat.com/ubi9/ubi-minimal:latest
      command:
        - /bin/sh
        - -ec
        - |
          cp /certs-in/*.crt /etc/pki/ca-trust/source/anchors/ &&
          update-ca-trust extract &&
          cp -a /etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem /shared/
      volumeMounts:
        - name: certs-in
          mountPath: /certs-in
          readOnly: true
        - name: cert-share
          mountPath: /shared
  containers:
    - name: app
      image: my-app:1.0
      securityContext:
        runAsNonRoot: true
        readOnlyRootFilesystem: true
      volumeMounts:
        - name: cert-share
          mountPath: /etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem
          subPath: tls-ca-bundle.pem
          readOnly: true
  volumes:
    - name: certs-in
      configMap:
        name: internal-ca-certs
        defaultMode: 0444
    - name: cert-share
      emptyDir: {}
```

This is the standard way to make CA trust work under **OpenShift random UIDs** and **read-only root filesystems** without weakening security posture. An app-level alternative is to skip the OS store entirely and use environment variables (Method 5).

### 4.5 Method 5 — Application-Level Configuration

Recommended when: the app is the only consumer; the container runs non-root on a read-only filesystem; or you want zero system-store coupling. These override or extend trust *for that process only*:

| Runtime | Mechanism | Example |
|---|---|---|
| OpenSSL / any | `SSL_CERT_FILE=/path/to/bundle.pem` | `SSL_CERT_FILE=/run/secrets/ca-bundle.pem myapp` |
| Node.js | `NODE_EXTRA_CA_CERTS` (appends) | `NODE_EXTRA_CA_CERTS=/run/secrets/internal-ca.pem node app.js` |
| Java | `JAVA_TOOL_OPTIONS=-Djavax.net.ssl.trustStore=...` | `JAVA_TOOL_OPTIONS="-Djavax.net.ssl.trustStore=/run/secrets/truststore.p12 -Djavax.net.ssl.trustStorePassword=..."` |
| Python requests | `REQUESTS_CA_BUNDLE` | `REQUESTS_CA_BUNDLE=/run/secrets/ca-bundle.pem python app.py` |
| Go | `SSL_CERT_FILE` | `SSL_CERT_FILE=/run/secrets/ca-bundle.pem ./app` |
| git | `GIT_SSL_CAINFO` | `GIT_SSL_CAINFO=/run/secrets/ca-bundle.pem git clone ...` |
| curl | `--cacert` / `CURL_CA_BUNDLE` | `curl --cacert /run/secrets/ca-bundle.pem https://internal-api` |
| .NET | `SSL_CERT_FILE` / `SSL_CERT_DIR` | `SSL_CERT_FILE=/run/secrets/ca-bundle.pem dotnet app.dll` |

| Pros | Cons |
|---|---|
| No system-store modification | Per-app config duplication ("env var sprawl") |
| Per-app control (different apps, different CAs) | Easy to miss an app → inconsistent behavior |
| Works non-root, read-only FS, scratch | Must be set in every deployment manifest |

### 4.6 Comparison Table

| Method | When to use | Pros | Cons |
|---|---|---|---|
| 1. Bake at build time | Stable global CAs; self-contained/air-gapped images; scratch/distroless | Immutable, reproducible, zero runtime deps | Rebuild per rotation; certs in layers |
| 2. Mount at runtime | Env-specific CAs; frequent rotation; central platform control | No rebuild; no certs in layers; central mgmt | Mount dependency; restart needed on change |
| 3. Hybrid | Enterprise defaults + per-env overrides | Best of 1+2 | Two mechanisms to maintain |
| 4. initContainer | Non-root / read-only FS / OpenShift random UID | Works under restricted SCC; merges safely | Extra container; init overhead |
| 5. App-level env config | Single-runtime apps; per-app isolation | No system changes; precise control | Env var sprawl; easy to miss apps |

---

## 5. Special Cases by Runtime and Base Image

### 5.1 Java Applications

Java is the classic trap: **the JVM never consults the OS trust store.** It uses its own `cacerts` keystore (default password `changeit` in OpenJDK images). Even after `update-ca-certificates`, Java apps fail with `PKIX path building failed: unable to find valid certification path to requested target`.

```dockerfile
FROM eclipse-temurin:21-jre
COPY internal-ca.crt /tmp/internal-ca.crt
RUN keytool -importcert -alias internal-ca -file /tmp/internal-ca.crt \
    -keystore "$JAVA_HOME/lib/security/cacerts" -storepass changeit -noprompt \
    && rm /tmp/internal-ca.crt
```

- **Java 9+:** use the `-cacerts` shortcut: `keytool -importcert -alias internal-ca -file internal-ca.crt -cacerts -storepass changeit -noprompt`.
- **Formats:** Java 8 defaults to JKS; Java 9+ to PKCS12. Modern images ship PKCS12; do not convert unless you must.
- **Passwords:** `changeit` is public knowledge — in hardened images set a custom truststore password passed at runtime from a Secret.
- **JAVA_TOOL_OPTIONS at runtime:** `JAVA_TOOL_OPTIONS="-Djavax.net.ssl.trustStore=/run/secrets/truststore.p12 -Djavax.net.ssl.trustStorePassword=$(cat /run/secrets/ts-pass)"` — ⚠️ overriding `trustStore` **replaces** cacerts entirely, so the mounted truststore must also contain the public CAs or public TLS breaks.
- **Runtime alternative:** an initContainer builds/imports cacerts into a shared volume the app mounts at `$JAVA_HOME/lib/security/cacerts` (or `/etc/ssl/certs/java/cacerts` on Debian images).

### 5.2 Node.js

- Node uses the OS trust store (bundled OpenSSL) — after `update-ca-certificates`, most Node TLS just works.
- **Add CAs without replacing the bundle:** `NODE_EXTRA_CA_CERTS=/path/to/internal-ca.pem` (PEM file, may contain multiple certs). This is the recommended mechanism for internal CAs.
- `NODE_TLS_REJECT_UNAUTHORIZED=0` disables verification entirely — **never in production**. It silently disables all TLS validation and trips scanners and audit findings. It is deprecated in Node 20+, which prints a warning when set.
- **Electron apps** carry their own Chromium stack: OS-store updates are not enough, and `ignore-certificate-errors` is a security hole. Handle trust in the main process via `session.setCertificateVerifyProc`. In banking, avoid Electron for internal TLS services or wrap it with a reviewed trust configuration.

### 5.3 Python

- `ssl` / urllib3 use the OS store via OpenSSL — system updates apply.
- **`requests` is different:** it bundles `certifi` and uses *that* bundle, ignoring the OS store. This is the most common "curl works, Python fails" cause in containers.
  - Fix: `REQUESTS_CA_BUNDLE=/etc/ssl/certs/ca-certificates.crt`, or per-request `verify='/path/to/bundle.pem'`, or override `certifi.where()`.
- **pip** verifies PyPI over TLS with its own config: `pip install --cert /path/to/bundle.pem` or `PIP_CERT` env var (needed when pip traverses a corporate MITM proxy).

### 5.4 Go

- `crypto/x509` uses the OS store at runtime (`SystemCertPool`) — system updates apply with no rebuild.
- `SSL_CERT_FILE` / `SSL_CERT_DIR` are honored on all platforms — convenient for non-root/read-only containers.
- Compiled binaries do not embed trust at build time (unless you deliberately vendor roots), so runtime mounting works cleanly.

### 5.5 .NET

- On Linux, .NET uses the OS store via OpenSSL — system updates apply.
- `SSL_CERT_FILE` / `SSL_CERT_DIR` override the bundle per process.
- Custom validation callbacks (`ServerCertificateCustomValidationCallback`) are possible, but callbacks that return `true` unconditionally are an audit finding — restrict to specific hosts and CA fingerprints.

### 5.6 NSS-Based Applications

Some curl builds, browsers, and legacy apps use the NSS database at `/etc/pki/nssdb` instead of the OpenSSL bundle:

```bash
certutil -A -n "Internal CA" -t "C,," -d sql:/etc/pki/nssdb -i internal-ca.crt
```

`-t "C,,"` marks the cert as a trusted CA for TLS. If a component misbehaves after system trust updates, check whether it links against NSS (`ldd $(which curl) | grep nss`).

### 5.7 Distroless Images

Google distroless images (`gcr.io/distroless/base`, `.../java`, `.../nodejs`) ship the CA bundle but **no package manager, no shell, no update tooling** — you cannot run `update-ca-certificates` inside them. Options:

- **Pre-bake** the complete bundle: in a builder stage run `update-ca-certificates`, then `COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/ca-certificates.crt` into the distroless stage.
- **Mount at runtime** the full bundle (public + internal) at `/etc/ssl/certs/ca-certificates.crt` (ConfigMap/Secret volume).
- Use app-level env vars (`NODE_EXTRA_CA_CERTS`, `REQUESTS_CA_BUNDLE`, `JAVA_TOOL_OPTIONS`) pointing at a mounted bundle file.

### 5.8 scratch Images

`FROM scratch` has nothing — no bundle, no `/etc/ssl/certs`. Add trust explicitly:

```dockerfile
FROM scratch
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/ca-certificates.crt
COPY myapp /myapp
ENTRYPOINT ["/myapp"]
```

Where `builder` is any stage that ran `update-ca-certificates`. Go static binaries (`CGO_ENABLED=0`) read this file at runtime — it must contain the internal CAs if the binary talks TLS.

### 5.9 Multi-Stage Builds for Certificates

```dockerfile
FROM ubuntu:24.04 AS certs
RUN apt-get update && apt-get install -y --no-install-recommends ca-certificates \
    && rm -rf /var/lib/apt/lists/*
COPY internal-ca.crt /usr/local/share/ca-certificates/
RUN update-ca-certificates

FROM alpine:3.20
COPY --from=certs /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/ca-certificates.crt
COPY app /app
```

Keeps build tooling out of the final image and makes the cert layer reviewable and cacheable.

### 5.10 UBI (Red Hat Universal Base Image)

- UBI uses the RHEL mechanism: anchors in `/etc/pki/ca-trust/source/anchors/`, `update-ca-trust extract`.
- **UBI minimal / UBI micro** lack `update-ca-trust` and a shell by default — bake the final bundle in a builder stage and COPY it, or mount the bundle at runtime at `/etc/pki/tls/certs/ca-bundle.pem`.
- UBI is the natural fit for OpenShift's cluster-wide CA injection because the injected bundle path matches the RHEL layout (Section 6).

---

## 6. OpenShift-Specific Considerations

OpenShift adds platform-level certificate machinery most Kubernetes distributions lack. Most "works in Docker, fails in OpenShift" cert incidents trace to **random UID**, **read-only root filesystems**, and **cluster-wide CA injection**.

### 6.1 Cluster-Wide Trusted CA (Proxy Configuration)

OpenShift can inject a trusted CA bundle into **every pod** — the enterprise-grade version of Method 2. Create a ConfigMap in `openshift-config` with the internal CAs under key `ca-bundle.crt`, then reference it in the cluster proxy:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: internal-ca-bundle
  namespace: openshift-config
data:
  ca-bundle.crt: |
    -----BEGIN CERTIFICATE-----
    ...
    -----END CERTIFICATE-----
```

```yaml
apiVersion: config.openshift.io/v1
kind: Proxy
metadata:
  name: cluster
spec:
  trustedCA:
    name: internal-ca-bundle
```

The cluster network operator distributes the merged bundle (cluster CA + internal CAs) via the ConfigMap `openshift-config-managed/trusted-ca-bundle`, mounted into pods at `/etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem` (and as `/etc/pki/tls/certs/ca-bundle.pem`). This is why **UBI-based images get CA trust "for free"** on OpenShift.

> ⚠️ Debian/Ubuntu images do **not** read the RHEL path. For those, use an initContainer to merge the injected bundle into `/etc/ssl/certs`, or point apps at the injected file via `SSL_CERT_FILE=/etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem`.

### 6.2 Service CA and Serving Certificates

OpenShift's **service CA operator** provides automatic TLS for internal services:

- The service CA is mounted in every pod at `/var/run/secrets/kubernetes.io/serviceaccount/service-ca.crt` — trust it for internal service calls.
- **Serving certificates:** annotate a `Service` to get a signed cert + key in a Secret, automatically renewed:

```yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    service.beta.openshift.io/serving-cert-secret-name: my-service-tls
spec:
  selector:
    app: my-service
  ports:
    - port: 443
      targetPort: 8443
```

OpenShift generates `my-service-tls` (`tls.crt` / `tls.key`), signed by the service CA, valid for the service DNS names (`my-service.ns.svc`, `my-service.ns.svc.cluster.local`). This is the recommended way to get server certificates for internal services — no manual issuance.

### 6.3 OpenShift Builds and Image Registry TLS

- OpenShift Builds (BuildConfig/S2I) run in pods that share cluster trust — with the cluster proxy CA configured, build pods trust the internal registry. You can also add CAs directly in the build's Dockerfile (Method 1).
- For Buildah/podman pulling from a private registry, per-registry CA files live at `/etc/containers/certs.d/<registry-host>/ca.crt`. When pushing to an internal registry whose cert is not in the cluster bundle, add that CA to the build pod's trust. `--tls-verify=false` is **only** acceptable in isolated, non-production pipelines — it is an audit finding in production.
- The integrated registry presents the cluster CA, already trusted by pods.

### 6.4 Random UID and Read-Only Root Filesystems

OpenShift's default **restricted** SCC runs containers with a **random, non-root UID** (e.g., `1000730000`) and, in hardened clusters, `readOnlyRootFilesystem: true`. The container therefore cannot write to `/etc/ssl/certs` or `/etc/pki/...`:

| Constraint | Effect on certs | Fix |
|---|---|---|
| Random UID | `update-ca-certificates` at container start fails (permission denied) | Bake certs at build time, or initContainer + shared volume, or app-level env vars |
| Read-only root FS | Cannot write trust store at runtime at all | Same as above; never `chmod` your way around it |
| Non-root app | App cannot install CAs itself | App-level config (Method 5) is cleanest |

If you must relax constraints, do it per-workload via SCC or `securityContext` with justification — but prefer patterns that work under restricted SCC; production namespaces usually will not let you relax it.

### 6.5 Cluster CA and Kube API Trust

The cluster's own CA (`kube-root-ca.crt`) is available in every namespace and mounted at `/var/run/secrets/kubernetes.io/serviceaccount/ca.crt` for API server connections — client libraries handle this automatically; do not override it.

---

## 7. Kubernetes Certificate Mounting Best Practices

### 7.1 ConfigMap vs Secret

- **CA certificates are public** (no secret material) → **ConfigMap** is correct and keeps them visible in GitOps.
- **Private keys, server cert+key pairs, client certs** → **Secret** (prefer encryption at rest via KMS).
- Server certificates (`tls.crt`) are public in content but come bundled with their key in the same Secret — keep the pair in a Secret.

### 7.2 Mounting Rules

- **Always mount read-only:** `readOnly: true`; `defaultMode: 0444` for certs, `0600` for keys.
- **Prefer `subPath` for single-file mounts.** ⚠️ Caveat: `subPath` mounts **do not receive ConfigMap updates** — content is fixed at pod start, so rotation requires a pod restart anyway (which you need for trust-store caching).
- **Avoid mounting over entire system directories** like `/etc/ssl/certs` unless replacing the complete bundle deliberately — a ConfigMap directory mount hides the distro symlink farm and breaks TLS for everything else in the image. Mount to a custom path and point apps at it, or merge via initContainer.
- **Use projected volumes** for multiple sources in one directory:

```yaml
volumes:
  - name: trust
    projected:
      sources:
        - configMap:
            name: internal-ca
            items:
              - key: ca.crt
                path: internal-ca.pem
        - secret:
            name: client-cert
            items:
              - key: tls.crt
                path: client.crt
              - key: tls.key
                path: client.key
                mode: 0600
```

- **Set file permissions explicitly** — certs `0644`, keys `0600`.

### 7.3 Restart Policies on Certificate Rotation

- ConfigMap/Secret updates reach the kubelet within ~1 minute, but **most runtimes load the trust store once at process start** — a cert change is invisible to a running app until it restarts.
- Standard pattern: **rollout restart when the trust ConfigMap changes**, using a checksum annotation:

```bash
CM_SHA=$(kubectl get cm internal-ca -o jsonpath='{.data.ca\.crt}' | sha256sum | cut -d' ' -f1)
kubectl annotate deployment my-app cert-hash=$CM_SHA
kubectl rollout restart deployment/my-app
```

Or automate with a controller (e.g., Stakater Reloader watches ConfigMaps/Secrets and restarts workloads on change). Make sure the annotation actually changes so the rollout triggers.

---

## 8. Certificate Rotation Strategy

| Step | Practice |
|---|---|
| **Monitor expiry** | cert-manager metrics, cert-exporter, blackbox probes (Section 12); alert ≥ 30 days before expiry |
| **Automate renewal** | cert-manager (internal CA issuer, Vault PKI, ACME), or Venafi/smallstep pipelines for the corporate PKI |
| **Roll out after rotation** | Checksum-annotation rollout restart (Section 7.3); stage environments first |
| **Test after rotation** | Re-run the validation suite (Section 14) against the new bundle |
| **Document** | Owner, expiry, rotation procedure, RPO/RTO per service (banking audits expect this) |

Avoid these anti-patterns: rotating the CA but not the mounted bundles in namespaces (ConfigMap drift); rotating a server cert without including its intermediate in the served chain; relying on "the app re-reads trust on each request" (true for almost no runtimes); and rotating keys by baking new images (forces emergency rebuilds — keys belong in Secrets).

---

## 9. cert-manager: Automated Certificate Management

[cert-manager](https://cert-manager.io) is the de-facto standard for Kubernetes certificate lifecycle automation and is essential in enterprise platforms.

### 9.1 Core Concepts

- **Issuer** (namespace-scoped) and **ClusterIssuer** (cluster-wide): the CA that signs requests. Types: `SelfSigned`, `CA` (local CA keypair in a Secret), `ACME` (Let's Encrypt via HTTP-01/DNS-01), `Vault`, `Venafi`, external issuers (smallstep, Google CAS).
- **Certificate** resources request a cert; cert-manager issues it into a **Secret** (`tls.crt`/`tls.key`, plus `ca.crt` when applicable) and **renews automatically** (default 2/3 of lifetime).
- **CA injector:** injects a CA bundle into ConfigMaps/Secrets based on annotations — the cleanest way to distribute trust to pods.

### 9.2 Example: Internal CA Issuer + Service Certificate

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: internal-ca
spec:
  ca:
    secretName: internal-ca-keypair
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: api-gateway-tls
  namespace: payments
spec:
  secretName: api-gateway-tls
  issuerRef:
    name: internal-ca
    kind: ClusterIssuer
  dnsNames:
    - api-gateway.payments.svc.cluster.local
    - api-gateway.payments.svc
  duration: 2160h
  renewBefore: 720h
```

### 9.3 CA Injection into Pods (the annotation pattern)

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-ca-bundle
  annotations:
    cert-manager.io/inject-ca-from: payments/api-gateway-tls
data:
  ca-bundle.pem: ""
```

The deployment mounts this ConfigMap; the CA injector keeps the bundle in sync with the Certificate, so trust follows the cert lifecycle automatically. `cert-manager.io/inject-ca-from-secret` works the same way for Secrets.

### 9.4 cert-manager Best Practices

- Use **ClusterIssuer for shared internal CAs**; namespace-scoped **Issuers for isolation** (teams issue against their own intermediates).
- Annotate every workload that consumes a Certificate's CA with `inject-ca-from` so bundles update on rotation.
- **Back up cert-manager resources** (Issuers, Certificates, CA keypair Secrets) — losing the CA keypair invalidates every cert it signed; back up the CA Secret encrypted (Velero or etcd backups).
- Monitor cert-manager metrics (Section 12) and the `CertificateReady` condition; alert on renewal errors and expiry; pin versions in GitOps and enforce issuance authorization via `approver-policy` in regulated environments.

---

## 10. mTLS Between Containers

Mutual TLS (client + server certificates) is the standard way to authenticate service-to-service calls in banking microservices.

### 10.1 Options

| Approach | How it works | When to use |
|---|---|---|
| **Service mesh (Istio, Linkerd)** | Sidecar proxies terminate mTLS automatically; workload identity via SPIFFE; policy via PeerAuthentication | Many services, dynamic workloads; automatic rotation, zero per-app code |
| **Application-level mTLS** | Each app presents a client cert from a mounted Secret; validates peers against a CA bundle | Few services, or apps outside the mesh (legacy, databases, gateways) |
| **SPIFFE/SPIRE** | Workload attestation issues short-lived SVIDs (x509 certs) per workload identity | Zero-trust platforms; fine-grained identity; certs measured in hours |
| **Vault PKI / cert-manager per-pod** | Each pod gets its own client cert via CSI driver or sidecar | Per-pod identity without a mesh |

### 10.2 Banking mTLS Patterns

- **Microservices:** mesh-managed mTLS (Istio `PeerAuthentication` with `STRICT` mode); SPIFFE identities map to application roles.
- **API gateways:** the gateway terminates client-cert auth for institutional clients; internal service calls use a service-account cert. Partner client certs are managed by the PKI team, never baked into images.
- **Databases:** PostgreSQL (`sslcert`/`sslkey`/`sslrootcert`), Oracle wallet, SQL Server encrypted connections — client certs mounted from Secrets, CA from ConfigMap, paths referenced in connection strings.

### 10.3 How to Add Client Certificates to Containers

- **Never bake client keys into images.** Mount at runtime:

```yaml
spec:
  containers:
    - name: app
      volumeMounts:
        - name: client-tls
          mountPath: /etc/tls/client
          readOnly: true
  volumes:
    - name: client-tls
      secret:
        secretName: svc-payments-client-tls
        defaultMode: 0600
```

The Secret holds `tls.crt`, `tls.key`, optionally `ca.crt` (the server-side CA the client must trust). `defaultMode: 0600` keeps the key unreadable by other users in the pod. In meshes you never handle certs — the sidecar manages identity; in SPIRE, the agent provisions SVIDs via a Unix socket or mounted directory.

- **Rotation:** with mounted Secrets, rotation = issuing a new Secret + rollout restart. With mesh/SPIRE, rotation is automatic and continuous (SPIRE SVIDs live hours, not months).

---

## 11. Security Best Practices

1. **Never store private keys in images.** Even a key "removed" in a later layer is recoverable from earlier layers (`docker history`, layer extraction). Keys arrive at runtime from Secrets, Vault, or the mesh.
2. **Encrypt Secrets at rest.** Enable etcd encryption with KMS (OpenShift: `spec.encryption.type` on the APIServer config).
3. **External secret management.** Vault (dynamic PKI), AWS Secrets Manager, Azure Key Vault, GCP Secret Manager via CSI drivers or External Secrets Operator for platform-level and high-value certs.
4. **HSM for high-value keys.** Corporate root CA signing keys and protected intermediates belong in an HSM, never a Secret. cert-manager and Vault integrate with PKCS#11 HSMs.
5. **Key permissions.** `0600` for keys, `0644` for certs; read-only mounts; non-root users.
6. **Read-only root filesystems** where possible — blocks runtime writes to trust stores (which also blocks malicious cert installation).
7. **Minimal CA set (least privilege).** Trust only the CAs each workload needs; a "trust everything internal" bundle increases the blast radius of any CA compromise.
8. **Audit certificate usage.** Log issuance, renewal, validation failures into the SIEM. Control who can issue (approver-policy, RBAC on Issuers).
9. **Scan images for embedded keys.** Trivy (`trivy fs --secret-config`), Grype, gitleaks/trufflehog in CI catch accidentally baked keys before they reach a registry.
10. **SBOM for certificate inventory.** Generate SBOMs (syft) and track cert resources as inventory — know which workloads use which CA.
11. **Separate CAs per environment** (dev/test/prod hierarchies) and **per service** where justified — a compromised dev CA must not mint prod-valid certs.
12. **Use short-lived certificates where possible.** SPIFFE SVIDs (hours), cert-manager certs (90 days) — short lifetimes shrink the abuse window and force automation maturity.
13. **Image signing and provenance.** Sign with cosign/sigstore and enforce verification at admission so nobody ships an image with a tampered trust configuration.

---

## 12. Monitoring Certificate Expiry

"Certificates expire" is not an incident — it is a scheduled event your monitoring should catch first.

| Tool | What it monitors | Notes |
|---|---|---|
| **cert-exporter** (Prometheus community) | Cert expiry from files, k8s Secrets, Java keystores | `cert_exporter_cert_expires_in_seconds` |
| **cert-manager metrics** | `certmanager_certificate_expiration_timestamp_seconds`, `certmanager_certificate_ready_status`, `certmanager_certificate_renewal_timestamp` | Alert on renewal within 30 days and `ready_status != True` |
| **blackbox-exporter** | TLS probes of service endpoints | Catches trust failures and served-chain problems from outside |
| **openssl cron job** | Ad-hoc expiry checks | `openssl x509 -in cert.pem -noout -enddate` over a cert inventory |
| **PKI platform dashboards** (Venafi, smallstep, AD CS) | Corporate CA inventory | Source of truth for externally issued certs |

**Alerting policy (banking-appropriate):** `WARNING` at < 30 days remaining; `CRITICAL` at < 7 days or on renewal failure; `PAGE` when a cert is expired or a validation failure occurs in production. Route alerts to both the platform team and the certificate owner (documented in the service runbook).

---

## 13. Incident Response for Certificate Failure

### 13.1 Runbook

1. **Detect.** Alert fires: TLS handshake failures, `x509` errors in app logs, blackbox probe failures, gateway 502s.
2. **Contain.** If a bad bundle/rotation was just rolled out, roll back the ConfigMap/Secret/Deployment to last known-good (`kubectl rollout undo`, restore from GitOps). Stop further rollout of the faulty cert.
3. **Identify.** Determine *which* cert/CA: `openssl s_client -connect host:port` to inspect the served chain, `openssl x509 -in cert -noout -subject -issuer -dates` for local files; check expiry and SANs; confirm which store the failing client uses (OS, Java, Node, certifi).
4. **Remediate.** Reissue via cert-manager/Vault (`cmctl renew`), restore the CA keypair Secret from backup if lost, or reissue through the PKI team. Update bundles in ConfigMaps; add missing intermediates.
5. **Verify.** Run the validation suite (Section 14) from inside the affected pods; confirm handshake success and that no other store is missing the CA.
6. **Rollback/restore.** If remediation fails, roll back the image/bundle and communicate a new RTO.
7. **Post-mortem.** Root-cause: expiry (missed monitoring), missing intermediate (chain packaging), store mismatch (Java vs OS), or clock skew (NTP). Update runbooks, thresholds, and the certificate inventory; feed findings into audit documentation.

### 13.2 Why Certificate Incidents Cascade

A single expired CA or missing intermediate can take down every service sharing that trust path simultaneously — databases, gateways, and health checks fail in parallel, looking like a platform outage. That is why banks treat certificates as critical configuration with owners, SLAs, and rehearsed runbooks, and why automation (cert-manager + monitored bundles) beats manual renewal.

---

## 14. Validation and Testing

### 14.1 Verify Trust Inside a Container

```bash
curl -v https://internal-api.example.internal/v1/health                      # handshake + CA verification
openssl s_client -connect internal-api.example.internal:443 \
    -CAfile /etc/ssl/certs/ca-certificates.crt -showcerts < /dev/null        # inspect the served chain
openssl verify -CAfile /etc/ssl/certs/ca-certificates.crt internal-api.crt   # verify cert against bundle
openssl x509 -in internal-api.crt -noout -text                               # subject, issuer, SANs, validity
openssl x509 -in internal-api.crt -noout -enddate                            # expiry date
openssl crl2pkcs7 -nocrl -certfile /etc/ssl/certs/ca-certificates.crt \      # bundle is well-formed PEM
    | openssl pkcs7 -print_certs -noout > /dev/null
```

### 14.2 Test the Built Image

```bash
docker run --rm my-image:1.0 curl -v https://internal-api.example.internal/v1/health
docker run --rm my-image:1.0 ls -la /etc/ssl/certs/
docker run --rm my-image:1.0 grep -c "BEGIN CERTIFICATE" /etc/ssl/certs/ca-certificates.crt
```

### 14.3 CI/CD Integration

- Add a **cert-trust smoke test** to the pipeline: build image → run container → curl the internal endpoint(s) with the expected CA → fail the build on any handshake error.
- **Testcontainers** for integration tests: spin up containers that mount the test CA bundle and assert service-to-service calls succeed — catches store-mismatch bugs before deploy.
- **BDD/Gherkin scenarios** for TLS behavior:

```gherkin
Scenario: Service trusts internal CA
  Given the internal CA bundle is mounted in the container
  When the service calls https://internal-api.example.internal/health
  Then the response status is 200
  And no TLS verification errors are logged
```

A companion scenario asserts that an expired server certificate fails the handshake with a logged, alerted certificate error.

- **Per-environment verification:** a GitOps post-sync job (ArgoCD) or CI step that runs `curl -v` against each environment's endpoints with that environment's bundle.

---

## 15. Troubleshooting

### 15.1 Error Catalog

| Error | Root cause | Fix |
|---|---|---|
| `x509: certificate signed by unknown authority` | CA not in the store the client actually uses | Add the CA to the right store(s) — OS store, Java cacerts, certifi, NODE_EXTRA_CA_CERTS |
| `x509: certificate has expired or is not yet valid` | Expired cert, or **clock skew** (container clock wrong / NTP unsynced) | Check `date` in the container; fix NTP; renew the cert |
| `x509: certificate is valid for <other>, not for <host>` | **SAN mismatch** — cert lacks the hostname used | Use the cert's SAN hostname (`openssl x509 -noout -text \| grep -A1 "Subject Alternative Name"`); fix ingress/DNS; reissue with correct SANs |
| `unable to get local issuer certificate` | Chain incomplete — only the leaf shipped, intermediates missing | Serve/ship the full chain (leaf + intermediates + root); add intermediates to the bundle |
| `self-signed certificate` | A self-signed cert where a CA-signed one is expected | Add the self-signed cert to the trust store deliberately, or use a proper internal CA |
| `SSL certificate problem: unable to get local issuer certificate` (curl) | curl's bundle lacks the CA | `curl --cacert /path/to/bundle` or fix the system bundle |
| `certificate verify failed` (curl/openssl) | General verification failure (chain, expiry, hostname) | Use `curl -v` / `openssl s_client -showcerts` to see which check failed |
| `connection refused` | Network/service issue — **not** a TLS problem | Check the service is up, DNS resolves, network policies allow it |
| `PKIX path building failed: unable to find valid certification path` (Java) | CA missing from `cacerts` | `keytool -importcert` into the JVM's cacerts, or fix `-Djavax.net.ssl.trustStore` |
| `SSLHandshakeException: ... certificate_unknown` (Java) | Chain or truststore issues (client- or server-auth) | Add the full chain to the truststore; verify the served chain |
| `DEPRECATION: NODE_TLS_REJECT_UNAUTHORIZED` (Node 20+) | Env var set — verification disabled | Remove it; use `NODE_EXTRA_CA_CERTS` instead |
| `tls: failed to verify certificate: x509: ...` (Go) | System pool or `SSL_CERT_FILE` missing the CA | Point `SSL_CERT_FILE` at the right bundle or update the OS store |
| `Failed to verify the signature of certificate` (NSS apps) | NSS DB lacks the CA | `certutil -A -d sql:/etc/pki/nssdb` |

### 15.2 Systematic Debug Procedure

1. **Reproduce verbosely:** `curl -v https://target` — read the exact verification stage that fails.
2. **Inspect the served chain:** `openssl s_client -connect target:443 -showcerts < /dev/null` — complete? intermediates included?
3. **Check system time:** `date` in the container; `timedatectl` on the host. Clock skew produces bogus "expired/not yet valid" errors.
4. **Check trust store contents:** `ls -la /etc/ssl/certs/`, `grep -c "BEGIN CERTIFICATE" /etc/ssl/certs/ca-certificates.crt`; on RHEL, `ls /etc/pki/ca-trust/source/anchors/`.
5. **Check environment variables:** `env | grep -iE "SSL|CERT|CA_BUNDLE|NODE_EXTRA"` — a stray `SSL_CERT_FILE` pointing at a stale bundle is a classic.
6. **Check mounts and permissions:** `mount | grep -i cert` — is a ConfigMap/Secret volume hiding the system bundle?
7. **Verify the chain offline:** `openssl verify -CAfile bundle.pem cert.pem`; check SANs with `openssl x509 -noout -text`.
8. **Cross-check with different tools:** if `curl` works but the app fails, suspect a runtime-specific store (Java cacerts, certifi, NSS); check app logs for the exact error string.
9. **Compare with a known-good image:** run a fresh `ubuntu:24.04` with the same mounts and test — isolates image drift from platform issues.
10. **Verify image content:** `docker run --rm my-image ls -la /etc/ssl/certs/` — confirm the cert layer actually landed (a common CI bug: COPYing to the wrong stage in multi-stage builds).

---

## 16. Key Takeaways

1. **Containers trust nothing beyond the ~140 public roots by default.** Internal/private CAs (the norm in banking) must be added explicitly — via build-time baking, runtime mounting, or app-level config.
2. **There is no single trust store.** OS store, Java cacerts, Python certifi, Node's OpenSSL, NSS — add the CA to *every* store your stack uses, or you will debug "curl works but the app fails" at 3 a.m.
3. **Bake stable global CAs into images; mount environment-specific CAs at runtime.** The hybrid approach fits enterprise platforms best.
4. **Never bake private keys.** Secrets, Vault, KMS, HSMs — and `0600` permissions on anything key-shaped.
5. **On OpenShift, design for random UIDs and read-only root filesystems from day one** — initContainer + emptyDir patterns and cluster-wide `trustedCA` injection are your friends; UBI-based images benefit most from cluster injection.
6. **Automate the lifecycle:** cert-manager for issuance/renewal, CA-injection annotations for bundle distribution, checksum-annotated rollout restarts for rotation, expiry alerting at T-30 days.
7. **Test trust in CI, not in production:** curl/openssl smoke tests, Testcontainers, and BDD scenarios catch missing CAs before they become outages.
8. **Treat certificates as critical configuration** — owned, documented, monitored, and rehearsed in incident-response runbooks. In a bank, a certificate failure is a platform incident with audit implications.

---
*Related guides in this repository: OpenShift (random UID & security constraints), workload availability patterns, Docker image tooling.*
