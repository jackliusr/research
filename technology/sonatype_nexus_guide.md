# Sonatype Nexus Repository — Comprehensive Guide

> **Author:** Jack Liu Shurui | **Updated:** July 2026 | **Version:** Nexus 3.x  
> **Context:** Solution Architect at Crédit Agricole CIB, Singapore

---

## 1. What is Sonatype Nexus?

**Sonatype Nexus Repository** (formerly Nexus Repository Manager) is an artifact repository and binary manager — a centralised platform to store, proxy, manage, and distribute software artifacts (JARs, Docker images, npm packages, PyPI wheels, Helm charts, etc.) across the SDLC.

Launched in **2008**, Nexus grew from a Maven-specific repo manager into a universal platform supporting **20+ package formats**. Sonatype was acquired by **TPG Capital** in 2022.

**Key value props:** centralised artifact store, proxy caching (faster builds, less external dependency), release immutability, and software supply chain control (paired with Nexus IQ/Firewall).

**Editions:**
| Edition | License | Limits |
|---|---|---|
| Nexus Repository OSS | Open Source (EPL/Apache 2.0) | No SAML/SSO, no HA, restricted format support |
| Nexus Repository Pro | Commercial | Full formats, HA, S3/GCS blob stores, content selectors, staging |
| Nexus Lifecycle / IQ | Commercial add-on | Policy engine, vulnerability/license scanning |
| Nexus Firewall | Commercial add-on | Proactive malware blocking at proxy download time |

---

## 2. Architecture

### Core Components

Nexus is a **Java (OSGi) server** managing three abstractions:

1. **Blob Stores** — physical storage (filesystem, S3, GCS, Azure Blob) decoupled from repository config
2. **Repositories** — logical containers with format-specific behaviour (proxy/hosted/group)
3. **Repository Groups** — virtual endpoints aggregating multiple repos under one URL

```
Nexus Server
├── REST API / Web UI / Docker Registry
├── Repository Manager (Format Adapters Layer)
└── Blob Store Layer (FS | S3 | GCS | Azure)
```

### Repository Types

| Type | Behaviour | Use Case |
|---|---|---|
| **Proxy** | Caches remote registry content (Maven Central, Docker Hub, npmjs, PyPI). Serves cached copy on repeat requests; fetches on cache miss. | Faster builds, resilience when remotes go down, bandwidth reduction |
| **Hosted** | Stores your organisation's published artifacts. Supports release (immutable) and snapshot (mutable) policies. | Internal library publishing, Docker images, third-party binaries |
| **Group** | Combines multiple proxy + hosted repos behind one URL. Searches members in configured order, returns first match. | Single endpoint for all client tools |

### Supported Formats (20+)

| Format | Proxy | Hosted | Group | OSS/Pro |
|---|---|---|---|---|
| Maven (maven2) | ✅ | ✅ | ✅ | OSS |
| npm | ✅ | ✅ | ✅ | OSS |
| Docker / OCI | ✅ | ✅ | ✅ | OSS |
| PyPI | ✅ | ✅ | ✅ | OSS |
| NuGet | ✅ | ✅ | ✅ | OSS |
| Helm | ✅ | ✅ | ✅ | OSS |
| Conan (C/C++) | ✅ | ✅ | ✅ | OSS |
| Conda | ✅ | ✅ | ✅ | OSS |
| RubyGems | ✅ | ✅ | ✅ | OSS |
| Go | ✅ | ✅ | ✅ | OSS |
| APT, YUM (RPM) | ✅ | ✅ | ✅ | OSS |
| Raw (generic) | ✅ | ✅ | ✅ | OSS |
| CocoaPods, Git LFS, Bower, P2 | ✅ | ✅ | ✅ | Pro |
| Hugging Face | ✅ | ❌ | ❌ | Pro |

Docker repos implement the **OCI Distribution Specification** — compatible with Docker, Podman, Helm, ORAS, Cosign, Syft, Skopeo.

### Blob Stores

Decoupled storage layer. Each blob store type:
- **File** — local filesystem (default, single-node)
- **S3** — Amazon S3 or S3-compatible (MinIO, Ceph)
- **GCS** — Google Cloud Storage
- **Azure Blob** — Azure block blobs

Best practice: separate blob stores per lifecycle domain (e.g., proxy cache vs. hosted releases).

### High Availability (Pro Only)

Active/active clustering via **external PostgreSQL** (required for HA). Nodes share the same blob store (S3/GCS recommended) behind a load balancer. OpenShift Operator supported for Kubernetes deployments.

---

## 3. Core Features

### Proxy Caching
Every proxy repository caches downloaded components locally. Benefits: microsecond local retrieval vs. 100ms+ internet fetch; builds succeed when remotes are offline; admins can audit exactly which external components are consumed.

### Hosted Publishing & Release Management
- **Immutable releases**: once published, a version cannot be overwritten (prevents substitution attacks)
- **Snapshot overwrite**: development iterations can freely overwrite snapshots
- **Component metadata**: format-specific metadata (pom.xml, package.json, Docker manifest) automatically parsed and indexed

### Smart Search
Format-aware search across all repos: Maven GAV coordinates, npm package name, Docker image/tag, free-text, custom tags (Pro).

### Staging (Pro)
Controlled promotion through lifecycle: **Build → Stage Repo → Validate → Close → Promote → Release**. Supports automated CI/CD gate: create staging repo, upload, run checks, close (lock), promote to releases, or drop on failure. Backbone of release governance in regulated environments.

### Cleanup Policies
Automated rules to prevent disk exhaustion:
- Last downloaded N days ago
- Last blob updated N days ago
- Component age > N days
- Format-specific (untagged Docker images, old Maven snapshots)
- Runs via scheduled task; separate compact-blob-store task reclaims space

### Content Selectors & Privileges (Pro)
**CSEL** (Content Selector Expression Language) for path-level access control:
```
format == "maven2" and path =~ "^/com/mycompany/.*"
```
Assign CSEL-based privileges to roles → users/LDAP groups. Enables fine-grained access: "Team A uploads to com/myteam/, Team B reads from com/thirdparty/".

### Routing Rules
Allow/block specific external remote URLs. Used for compliance (block unapproved registries) or routing specific paths to different remotes.

### REST API
Full OpenAPI 3.0 at `/service/rest/swagger.json`. CRUD for repos, blob stores, users, roles, privileges, content selectors; component upload/download/search; staging lifecycle; health checks; Groovy scripting.

---

## 4. Nexus Repository vs IQ vs Firewall

| Product | Purpose | Deployment |
|---|---|---|
| **Nexus Repository** | Host, proxy, cache artifacts | Self-hosted or Sonatype Cloud |
| **Nexus IQ (Lifecycle)** | Policy engine — scan components for vulnerabilities, license issues, architecture risks | Self-hosted JVM or Cloud |
| **Nexus Firewall** | Proactive block of malicious components at download time (before they enter cache) | Inline between proxy and remote |

**IQ integration flow:** When a dependency is resolved → Nexus proxies it → IQ evaluates against policies (CVSS thresholds, license types) → block (quarantine), warn, or notify. IQ uses Sonatype's **Component Intelligence** — deep call-path analysis to identify **reachable** (not just transitive) vulnerabilities.

**Firewall** analyses components in real-time before caching, protecting against **freshly published malware** not yet in public CVE databases.

**Typical bundles:**
- Repository only (base)
- Repository + Lifecycle (security scanning)
- Repository + Lifecycle + Firewall (full supply chain defence)

---

## 5. Repository Management

### Standard Repository Layout

**Java shop:**
```
maven-central  → proxy (https://repo1.maven.org)
maven-releases → hosted (immutable)
maven-snapshots → hosted (mutable)
maven-all      → group (releases + snapshots + central)
```

**Polyglot shop:** parallel layouts for Docker (docker-hub proxy + docker-releases hosted + docker-all group), npm (npmjs proxy + npm-releases hosted + npm-all group), PyPI (pypi proxy + pypi-releases hosted + pypi-all group).

### Monitoring Health
- Blob store usage % and growth rate
- Proxy cache hit rate (target >80%)
- Stale components (not downloaded in 90+ days)
- Failed proxy requests (indicates remote registry issues)
- Repository sprawl tracking

### Migration from Nexus 2.x
Nexus 2.x is EOL. Migration to 3.x uses the **Content Migration Tool** (reads from Nexus 2 REST API, rebuilds in Nexus 3 blob store format). Run incrementally: copy → verify → switch DNS.

### Migration from Artifactory
Not natively supported. Approach: export via AQL/direct filesystem → bulk-import via Nexus REST API (`POST /service/rest/v1/components`) → rebind CI/CD configs.

---

## 6. Nexus in CI/CD

### Client Configuration Examples

**Maven** (settings.xml):
```xml
<mirror><id>nexus-all</id><mirrorOf>*</mirrorOf>
  <url>https://nexus.example.com/repository/maven-all/</url></mirror>
```
Deploy in pom.xml: `<distributionManagement>` pointing to maven-releases / maven-snapshots.

**Gradle:**
```groovy
repositories { maven { url "https://nexus.example.com/repository/maven-all/" } }
```

**npm** (.npmrc):
```
registry=https://nexus.example.com/repository/npm-all/
```

**Docker:**
```bash
docker login nexus.example.com:9443
docker push nexus.example.com/repository/docker-releases/myapp:1.0.0
```

**PyPI** (pip.conf):
```ini
[global]
index-url = https://user:token@nexus.example.com/repository/pypi-all/simple
```

### CI/CD Platform Integrations

| Platform | Method |
|---|---|
| Jenkins | Nexus Artifact Uploader Plugin, Nexus Platform Plugin, or raw curl/mvn |
| GitLab CI | Standard Maven/npm/Docker commands + staging API |
| GitHub Actions | `sonatype-nexus-community/nexus-action` or custom scripts |
| Azure DevOps | Nexus Upload/Download tasks from Marketplace |
| Bitbucket Pipelines | Manual client config |

### Promotion Workflow (REST API)

```bash
# Create staging repo
curl -u admin:pass -X POST \
  "https://nexus.example.com/service/rest/v1/staging/repositories" \
  -d '{"repository":"maven-releases","description":"v1.2.3-RC1"}'

# Upload component
curl -u admin:pass -X POST \
  "https://nexus.example.com/service/rest/v1/components?repository=maven-releases" \
  -F "maven2.groupId=com.mycompany" \
  -F "maven2.artifactId=my-lib" \
  -F "maven2.version=1.2.3" \
  -F "maven2.asset1=@target/my-lib-1.2.3.jar"

# Close (triggers IQ evaluation)
curl -X POST ".../staging/repositories/STG-123/close"

# Promote to release
curl -X POST ".../staging/repositories/STG-123/promote"
```

---

## 7. Security

### Authentication
- **Local users**: full CRUD, password policies, API tokens
- **LDAP/AD**: configurable pools, group mapping, fallback auth
- **SAML (Pro)**: Okta, Azure AD, OneLogin, ADFS
- **Anonymous access**: per-realm configurable, read-only

### Authorization (3-Tier Model)
```
User → Role → Privilege → Repository / Content Selector (CSEL)
```
- Repository-level: browse, read, edit, add, delete, admin
- Content Selector (Pro): path-level within a repo
- System-level: manage users, roles, blob stores, settings

### SSL/TLS
- Inbound: terminates TLS directly (Java keystore) or behind reverse proxy (Nginx, ALB)
- Outbound: verifies remote registry certificates (configurable truststore)
- Mutual TLS supported

### Audit
- Access logs: every request with timestamp, user, IP, action, resource
- Audit log (Pro): system-level config changes, privilege modifications
- Download tracking via audit log or JavaScript API

### IQ Security Integration
- Vulnerability scanning against CVE + Sonatype curated intelligence
- License compliance (GPL detection, policy enforcement)
- **Reachable vulnerability analysis**: call-path analysis reduces false positives
- Waivers for time-bound risk acceptance

---

## 8. Nexus vs Artifactory

| Feature | Nexus Repository | JFrog Artifactory |
|---|---|---|
| Open Source | ✅ Nexus Repository OSS | ❌ Artifactory OSS deprecated |
| Supported formats | 20+ | 30+ (more niche: Terraform, Vagrant, Opkg) |
| Docker registry | ✅ OCI (OSS supports Docker) | ✅ OCI (all editions) |
| HA clustering | ✅ Pro only | ✅ Enterprise |
| Storage backends | FS, S3, GCS, Azure | FS, S3, GCS, Azure |
| Security scanning | ✅ Nexus IQ (separate product) | ✅ Xray (integrated) |
| CI/CD native | API + community plugins | ✅ JFrog CLI, Pipelines |
| Query language | Smart search (text/GAV/format) | ✅ AQL (structured metadata queries) |
| Release promotion | ✅ Staging API (Pro) | ✅ Build promotion with full trace |
| AI/ML models | ✅ Hugging Face proxy (Pro) | ✅ ML management (Enterprise) |
| Cloud offering | ✅ Sonatype-hosted SaaS | ✅ JFrog-hosted SaaS |
| Licensing | OSS free / Pro subscription | Pro / Enterprise subscription |
| Typical buyer | Java-centric, banks, mid-market | Polyglot enterprises, cloud-native |

**Choose Nexus when:** you want OSS, are Java/Maven-heavy, prefer lightweight admin, need blob store separation without complexity, or already have separate security tooling.

**Choose Artifactory when:** you need niche formats (Terraform, Vagrant), integrated CI/CD (JFrog Pipelines), AQL for complex queries, or want a full DevSecOps platform.

---

## 9. Deployment & Operations

### Installation

**Standalone:**
```bash
wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz
tar -xzf latest-unix.tar.gz && cd nexus-3.x.y-*
./bin/nexus start   # Port 8081; admin password in sonatype-work/nexus3/admin.password
```

**Docker:**
```bash
docker run -d --name nexus -p 8081:8081 -p 9443:9443 -v nexus-data:/nexus-data sonatype/nexus3:latest
```
(Uses UID 200; mount volumes must be writable by that user.)

**Kubernetes:** OpenShift Operator for automated provisioning, external PostgreSQL, HA, blue-green upgrades, Prometheus metrics.

### Minimum Requirements

| Environment | RAM | CPU | Storage |
|---|---|---|---|
| Dev/small team | 4 GB | 2 cores | 50 GB SSD |
| Production (medium) | 8 GB | 4 cores | 200 GB+ SSD |
| Production (large) | 16-32 GB | 8+ cores | 1 TB+ SSD |
| HA cluster (per node) | 8-16 GB | 4-8 cores | Shared blob store (S3/GCS) |

Critical: **Use SSDs** for blob stores. HDDs cause severe degradation under concurrent Docker/npm access.

### Performance Tuning

**JVM** (in `nexus.vmoptions`):
```
-Xms8g -Xmx8g -XX:+UseG1GC -XX:+AlwaysPreTouch
```
- Set `-Xms` = `-Xmx` (avoid resizing overhead)
- Don't exceed 80% of physical RAM
- G1GC is recommended

**Other:** Place blob stores on separate disks from app; use soft/hard quotas; tune thread pool via `nexus.threadpool.maxSize` (default 200).

### Monitoring

| Method | Endpoint/Detail |
|---|---|
| Health check | `GET /service/rest/v1/status` |
| JMX | MBeans for heap, threads, blob store metrics |
| Prometheus | Community `nexusmetrics` exporter or OpenShift Operator built-in |
| Logs | `$data-dir/log/nexus.log`, request.log, access.log |
| Alerts | Disk space, heap >80%, cache hit rate drop, failed remotes |

### Backup & Restore

Three components to back up:
1. **Blob stores** — artifact content (largest; rsync/s3 sync/cloud snapshot daily)
2. **Configuration** — `$data-dir/etc/` (tarball weekly or post-change)
3. **Database** — OrientDB (hot backup supported) or PostgreSQL pg_dump (daily)

Pro tip: Use S3/GCS blob stores so the storage layer IS the backup.

### Upgrade
In-place upgrades within 3.x supported. Process: backup → stop → replace binaries → start (auto-migration). For critical instances, prefer blue-green: stand up new version, test, switch DNS.

---

## 10. Nexus in Banking/Enterprise Context

### Regulatory Alignment

| Regulation | Requirement | Nexus Solution |
|---|---|---|
| MAS TRM (Singapore) | Secure SDLC, supply chain risk | Proxy caching + IQ vulnerability scanning |
| HKMA CRAF | App security, dependency control | Staging + promotion audit trail |
| BCBS 239 / Basel | IT risk data aggregation | Immutable repos + component audit logs |
| PCI DSS | Secure coding, change management | RBAC, signed releases |
| SOX | Internal IT controls | Approval-gated promotions |
| DORA (EU) | ICT risk, supply chain security | Firewall + IQ proactive blocking |

### Governance Patterns

**Controlled external sourcing:** Only curated proxy repos configured; all other remote registries blocked via routing rules. New sources require change ticket approval.

**Internal component publishing:** Every internal library published to Nexus hosted repos. All builds resolve from Nexus group repos only (no direct internet). Ensures repeatable builds.

**Air-gapped deployment:**
1. Seed: deploy Nexus in DMZ with outbound-only access to approved registries
2. Pre-populate all required components via proxy cache
3. Move Nexus to fully isolated network segment (no outbound connectivity)
4. All builds resolve entirely from air-gapped cache

This prevents any new unvetted external dependencies from entering the build pipeline.

### Compliance with Nexus IQ
- **License scanning**: IQ flags GPL/AGPL/copyleft automatically; legal reviews and issues waivers
- **Vulnerability SLAs**: policy blocks CVSS >= 7.0 from release repos without documented waiver
- **Policy bundles**: pre-configured for OWASP, NIST, CIS benchmarks

### Build Reproducibility
Every dependency ever resolved is cached; even if removed from Maven Central, the cached copy persists. Combined with lock files (`pom.xml` exact versions, `package-lock.json`), organisations achieve **fully deterministic builds** — critical for regulated audits.

### Banking Reference Architecture

```
Developer Desktop (no internet)
        │
        ▼
CI/CD (Jenkins / GitLab / Azure DevOps)
        │
        ▼
Nexus Group Repos (unified endpoint)
        │
    ┌───┼───────────┐
    ▼   ▼           ▼
Hosted  Proxy      Proxy
Releases Maven     npmjs
Snapshots Central  DockerHub
        │           │
        └─────┬─────┘
              ▼
      Nexus Firewall
      (blocks malware)
              │
              ▼
      External Registries
```

**Key decisions:**
- No direct internet access for dependency resolution
- Firewall blocks zero-day malware at download time
- IQ evaluates every new component before cache entry
- Staging gates all releases: QA sign-off → promote → immutable release
- Blob stores on encrypted S3 for durability
- External PostgreSQL for HA/DR

---

## 11. References

| Resource | URL |
|---|---|
| Sonatype Official | https://www.sonatype.com/products/sonatype-nexus-repository |
| Help Documentation | https://help.sonatype.com/ |
| GitHub (OSS) | https://github.com/sonatype/nexus-public |
| Repository Types | https://help.sonatype.com/en/repository-types.html |
| Formats Matrix | https://help.sonatype.com/en/formats.html |
| Cleanup Policies | https://help.sonatype.com/en/cleanup-policies.html |
| Staging | https://help.sonatype.com/en/staging.html |
| Blob Stores | https://help.sonatype.com/en/configuring-blob-stores.html |
| Nexus vs Artifactory (Sonatype) | https://www.sonatype.com/compare/sonatype-nexus-versus-jfrog-artifactory |
| Nexus vs Artifactory (JFrog) | https://jfrog.com/blog/artifactory-vs-nexus-integration-matrix/ |
| Nexus CI Examples | https://github.com/sonatype-nexus-community/nexus-ci-examples |
| Docker Image | https://hub.docker.com/r/sonatype/nexus3 |
| Pricing Guide | https://www.cloudrepo.io/articles/sonatype-nexus-repository-pricing-guide |

---

*Maintained as part of the research repository by Jack Liu Shurui. For corrections, submit a PR.*
