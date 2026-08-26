# Tools to Compare Docker Images — A Comprehensive Guide

> **Audience:** Solution Architects, DevOps Engineers, Platform Engineers, Security Teams
> **Author:** Jack Liu Shurui | Cymbal Bank
> **Context:** Containerized deployments, CI/CD pipelines, container security in enterprise/banking environments

---

## 1. Why Compare Docker Images

### 1.1 Identify Layer Changes Between Versions
Every `RUN`, `COPY`, or `ADD` instruction creates a layer. When an image grows unexpectedly, layer-level diff pinpoints which instruction introduced bloat.

### 1.2 Detect Bloat and Unnecessary Files
Build artifacts (`.pyc` files, cached `node_modules`, leftover package downloads) can persist across layers even after deletion. Image comparison reveals these hidden files.

### 1.3 Compare Base Image Variants (Alpine vs. Ubuntu vs. Distroless)
Comparing Alpine-based, Ubuntu-based, and distroless variants of the _same application_ reveals the exact delta in layers and packages.

### 1.4 Validate Dockerfile Changes
After merging `RUN` commands, reordering layers, or switching package managers, a diff confirms intended changes occurred and no unintended files leaked in.

### 1.5 Audit Security Vulnerability Introduction
A passing build may introduce a vulnerable OpenSSL or compromised npm package. Vulnerability-focused diff tools (Grype, Trivy, Docker Scout) flag newly introduced CVEs.

### 1.6 Debug Unexpected Image Size
"Why is this image 1.2 GB when the code is 50 MB?" Tools surface which files consume space in each layer — cached package managers, debug symbols, dev dependencies.

### 1.7 Ensure Reproducible Builds
In regulated environments (banking, healthcare), builds must be reproducible. Comparing images from the same commit at different times reveals variance from timestamps, seeds, or network fetches.

### 1.8 Compliance — Verify Only Approved Changes
Image comparison provides an auditable diff between the previous release and the current candidate for change management review.

### 1.9 Migration — Validate Base Image Upgrades
Upgrading Ubuntu 22.04→24.04 should change only the OS layer. Comparing layer digests validates the application layer is identical.

---

## 2. Category Overview

| Category | Description | Representative Tools |
|---|---|---|
| **CLI Analysis Tools** | Deep dive into layers, file-level diff, packages | Dive, diffoci, crane, Skopeo, Syft+Grype |
| **CI/CD Integration Tools** | Automated comparison in pipelines, policy gating | Docker Scout, Trivy, Grype, crane, Dive |
| **Web / SaaS Platforms** | Online comparison, dashboards, team collaboration | Docker Scout Dashboard, Snyk, JFrog Xray |
| **IDE / Desktop Extensions** | Visual exploration in IDE | Docker Desktop (Scout tab), VS Code Docker ext. |
| **Registry-Native Tools** | Built into registries | Docker Hub (Scout), ECR (Inspector), GCR (Container Analysis), Quay |

---

## 3. CLI Analysis Tools

### 3.1 Dive
**Repo:** github.com/wagoodman/dive | **License:** Apache 2.0

The most popular open-source layer exploration tool with an interactive TUI.

**Key Features:**
- Interactive TUI for layer-by-layer file changes (added/modified/deleted).
- Per-layer size breakdown and file tree with individual file sizes.
- Image efficiency score (0-100%) showing "wasted" space from modified files that duplicate earlier content.
- CI mode with pass/fail based on configurable thresholds.
- Docker and Podman support.

**Commands:**
```bash
dive <image>:<tag>          # Interactive TUI
CI=true dive <image>:<tag>  # Headless CI mode
```

**CI Configuration (.dive-ci):**
```yaml
rules:
  lowestEfficiency: 0.95           # Fail if <95% efficient
  highestWastedBytes: 50MB         # Fail if >50MB wasted
  highestUserWastedPercent: 0.05   # Fail if >5% wasted
```

**Strengths:** Best-in-class TUI, file-level diff between layers, efficiency scoring, strong community (17k+ GitHub stars).

**Weaknesses:** Compares layers _within_ one image only (not side-by-side of two different images), no API, no two-image diff.

**Best For:** Exploring single image layers, identifying optimization opportunities, understanding what each layer adds.

---

### 3.2 container-diff (Deprecated)

**Repo:** github.com/GoogleContainerTools/container-diff | **Status:** Archived | **Language:** Go

Google's original image diff tool, now **deprecated** with no active maintenance. It supported multiple "analyzers" (file system, apt, npm, pip) and could diff both single images and pairs. While still functional, known bugs and no development make it unsuitable for production pipelines. Community recommends migrating to `diffoci`, `crane`, or Docker Scout.

**Original usage (for reference):**
```bash
container-diff diff alpine:3.19 alpine:3.20 --type=file
container-diff diff ubuntu:22.04 ubuntu:24.04 --type=apt
container-diff analyze nginx:latest --type=file --json
```

---

### 3.3 diffoci
**Repo:** github.com/reproducible-containers/diffoci | **License:** Apache 2.0

Purpose-built successor to container-diff from the Reproducible Containers group.

**Key Features:**
- Direct two-image diff: `diffoci diff <image1> <image2>`.
- Designed for reproducible build validation.
- Optionally uses containerd image store (v1.7+).
- Cross-platform with `--platform` flag.

**Commands:**
```bash
diffoci diff alpine:3.19 alpine:3.20
diffoci diff --format=json app:v1 app:v2
```

**Strengths:** Actively maintained, reproducible build focus, clean interface, containerd support.

**Weaknesses:** Newer tool with smaller community, limited output format options.

**Best For:** Validating reproducible builds, replacing container-diff.

---

### 3.4 Docker Scout CLI
**Docs:** docs.docker.com/scout | **Model:** Proprietary (free tier + paid)

Docker's official image analysis platform — vulnerability scanning, SBOM generation, policy evaluation, and image comparison.

**Comparison Capabilities:**
```bash
docker scout compare registry/repo:v1.0 --to registry/repo:v2.0
docker scout recommendations myapp:latest
docker scout sbom myapp:v1 --output sbom-v1.spdx
```

Output shows: added/removed/upgraded/downgraded packages, new vulnerabilities, policy pass/fail.

**Integration:** Docker Desktop (GUI), Docker CLI plugin, Docker Hub, GitHub Actions (`docker/docker-scout-action`), GitLab CI, Jenkins, CircleCI.

**Strengths:** Deep Docker ecosystem integration, policy-based gating, vulnerability diff, SBOM analysis, active Docker Inc. development.

**Weaknesses:** Requires Docker account, limited free tier, not open-source, Docker-focused (limited Podman support).

**Best For:** Teams using Docker Hub/Desktop needing vulnerability-focused comparison and CI/CD gating.

---

### 3.5 Syft + Grype (Anchore)
**Repos:** github.com/anchore/syft | github.com/anchore/grype | **License:** Apache 2.0

Syft generates SBOMs from container images; Grype scans for vulnerabilities. Together they form a powerful package-level comparison pipeline.

**Commands:**
```bash
syft alpine:latest                          # Generate SBOM
syft alpine:latest -o spdx-json             # SBOM in SPDX format
grype alpine:latest                         # Vulnerability scan
grype alpine:3.19 --diff alpine:3.20        # Vulnerability diff
```

**Strengths:** Fast (Go), open-source, strong SBOM support (SPDX/CycloneDX), vulnerability-focused diff, CI/CD native with clear exit codes.

**Weaknesses:** Not a general image diff tool (focuses on packages/vulnerabilities), no layer visualization, CLI-only.

**Best For:** Vulnerability-focused image comparison, SBOM generation and diff, CI/CD gate on vulnerability introduction.

---

### 3.6 Skopeo
**Repo:** github.com/containers/skopeo | **License:** Apache 2.0

Lightweight image management from the Podman/Buildah ecosystem. Inspects images without pulling.

**Commands:**
```bash
skopeo inspect docker://alpine:latest          # Inspect metadata (no pull)
skopeo inspect --config docker://app:v1        # Raw config
skopeo list-tags docker://library/alpine       # List tags
skopeo copy docker://src:tag docker://dst:tag  # Copy between registries
```

**Comparison example (manual):**
```bash
diff <(skopeo inspect docker://app:v1 | jq '.Layers') \
     <(skopeo inspect docker://app:v2 | jq '.Layers')
```

**Strengths:** No pull needed for inspection, works across all registries (Docker Hub, Quay, GCR, ECR, ACR), scriptable.

**Weaknesses:** No built-in diff command (manual jq scripting required), no visualization, no package-level comparison.

**Best For:** Quick metadata comparison, CI scripts comparing configs across registries, inspecting images without pulling.

---

### 3.7 Crane (go-containerregistry / OCI Image Tool)
**Repo:** github.com/google/go-containerregistry | **License:** Apache 2.0

Google's Go library and CLI for OCI image operations. `crane diff` is the standout for side-by-side image comparison.

**Commands:**
```bash
crane diff nginx:1.25 nginx:1.26    # Side-by-side file diff
crane ls <repository>                # List tags
crane digest <image>                 # Get digest
crane copy <src> <dst>              # Copy between registries
```

`crane diff` output shows added and removed files:
```bash
$ crane diff alpine:3.19 alpine:3.20
Added:
  /sbin/apk
  /etc/apk/keys/alpine-devel@lists.alpinelinux.org-4a6a0840.rsa.pub
Removed:
  /etc/apk/keys/alpine-devel@lists.alpinelinux.org-5243ef4b.rsa.pub
```

**Strengths:** True side-by-side file diff (unique differentiator), no pull needed, fast Go-based, all major registries, open-source, available as Go library.

**Weaknesses:** CLI-only, file-level diff only (no package awareness), less well-known than Dive, no built-in policy engine.

**Best For:** CI/CD pipelines comparing two image versions, quick CLI diff without pulling images.

---

### 3.8 Dreg
**Repo:** github.com/genuinetools/dreg | **Status:** Read-only / low activity

Docker registry CLI with a `diff` subcommand for comparing images. Shows added/modified/deleted files and layer comparison. Project is no longer actively maintained — prefer `crane diff` or `diffoci`.

---

### 3.9 Hadolint
**Repo:** github.com/hadolint/hadolint | **License:** GPL-3.0

Dockerfile linter, not an image comparison tool, but relevant for pre-build quality. Checks best practices that directly impact image size and layer efficiency:

```bash
hadolint Dockerfile
```

Rules like `DL3059` (consolidate RUN commands), `DL3009` (delete apt lists), and `DL3008` (pin versions) ensure the Dockerfile produces clean, comparable images.

---

## 4. Web-Based and SaaS Platforms

### 4.1 Docker Image Compare (Web)
**Website:** dockerimagecompare.top

Free online tool — paste `docker inspect` output for two images, get side-by-side comparison of layers, size, ENV, ports, volumes, entrypoint, and config metadata. No login required.

### 4.2 ImageObsolescence
**Repo:** github.com/pl4nty/image-obsolescence

Open-source tool tracking container image freshness. Monitors when base images receive updates and reports which of your images run outdated bases.

### 4.3 Snyk Container
**Website:** snyk.io | **Model:** Commercial (free tier)

Vulnerability scanning with diff capabilities. Compares image versions for new vulnerabilities, provides policy-based gating, base image recommendations, and fix guidance. Integrates with GitHub, GitLab, Bitbucket, Jenkins, CircleCI, and all major registries.

**Strengths:** Broad integration, excellent fix guidance, policy engine.
**Weaknesses:** Commercial pricing scales with usage.

### 4.4 JFrog Xray
**Website:** jfrog.com/xray | **Model:** Commercial (part of JFrog Platform)

Binary analysis with artifact comparison. Compares container image versions, diffs dependencies and vulnerabilities, and provides licence compliance checks. Deep integration with Artifactory.

**Strengths:** Deep binary analysis, dependency graph traversal, licence compliance.
**Weaknesses:** Requires full JFrog Platform, significant infrastructure investment.

### 4.5 Aqua Trivy
**Repo:** github.com/aquasecurity/trivy | **License:** Apache 2.0

Open-source all-in-one security scanner with vulnerability diff:

```bash
trivy image alpine:latest                           # Scan image
trivy image --diff alpine:3.19 alpine:3.20          # Vuln diff
trivy image --format cyclonedx alpine:latest         # SBOM
trivy image --severity CRITICAL,HIGH --exit-code 1   # CI gate
```

**Strengths:** Fully open-source, fast, multi-purpose (container, filesystem, repo, IaC), multi-language (10+ ecosystems).
**Weaknesses:** No layer visualization, CLI-focused, no native file-level diff.

### 4.6 Docker Scout Dashboard
**Website:** scout.docker.com

Browser-based view of image analysis, vulnerabilities, tag comparison, policy evaluation, and fleet-wide repository insights. Includes base image recommendation feature that quantifies vulnerability reduction of switching bases.

---

## 5. IDE and Desktop Extensions

- **Docker Desktop (Scout tab):** Built-in layer exploration, vulnerability listings per layer, package inventory per layer, tag comparison interface. Ships with Docker Desktop 4.17+. Provides graphical tag comparison, letting you select any two tags and view package differences, vulnerability delta, and layer size diff side by side — no CLI needed.
- **VS Code Docker Extension:** Sidebar view of images and containers, Hadolint integration for Dockerfile linting, image inspection (layers, env, ports), and Docker Scout CLI integration via the integrated terminal. Registry explorer lets you browse remote repositories and inspect images.
- **Podman Desktop:** Layer browsing, image inspection, and extension-based integration with Docker Scout and Skopeo for extended comparison capabilities. Useful for teams using Podman as their container engine.
- **IntelliJ IDEA Docker Plugin:** Basic image inspection and layer browsing from within JetBrains IDEs. Useful for Java/Kotlin teams who need quick access to container metadata without switching tools.

---

## 6. Registry-Native Tools

| Registry | Built-in Tool | Capabilities |
|---|---|---|
| **Docker Hub** | Docker Scout | Compare tags, vuln diff, policy evaluation |
| **Amazon ECR** | Amazon Inspector | Vuln scanning, CIS benchmarks, continuous monitoring |
| **Google GCR / Artifact Registry** | Container Analysis | Vuln scanning, package metadata, Binary Authorization |
| **Azure ACR** | Microsoft Defender for Cloud | Vuln scanning, registry-level security posture |
| **Red Hat Quay** | Quay Security Scanner | Vuln scanning, layer inspection, tag history |
| **Harbor** | Pluggable scanners | Trivy/Clair integration, replication, retention policies |

---

## 7. Comparison Matrix

| Tool | Primary Purpose | Comparison Type | Viz | CI/CD | OSS | Best For |
|---|---|---|---|---|---|---|
| **Dive** | Layer exploration | Single-image layer diff | TUI | ✓ | ✓ | Layer optimization, understanding composition |
| **diffoci** | Image diff | Two-image diff | CLI | ✓ | ✓ | Reproducible builds, container-diff replacement |
| **Docker Scout** | Vulnerability + diff | Two-image, policy | Web/CLI | ✓ (native) | ✗ | Docker-native teams, vulnerability gating |
| **Syft+Grype** | SBOM + vulnerability | Two-image (vuln diff) | CLI | ✓ (native) | ✓ | Package/vuln comparison, SBOM compliance |
| **Skopeo** | Image management | Metadata comparison | CLI | ✓ (script) | ✓ | Registry inspection without pull |
| **Crane (diff)** | File-level image diff | Two-image file diff | CLI | ✓ (native) | ✓ | CI/CD file-level diff, automation |
| **Snyk** | Vuln scanning | Two-image vuln diff | Web/CLI | ✓ | ✗ (free tier) | Enterprise vuln management |
| **JFrog Xray** | Binary analysis | Artifact diff | Web | ✓ | ✗ | Dependency graph, licence compliance |
| **Trivy** | All-in-one security | Vulnerability diff | CLI | ✓ | ✓ | Fast open-source vuln diff, IaC scanning |
| **Hadolint** | Dockerfile linting | N/A (pre-build) | CLI | ✓ | ✓ | Dockerfile quality before building |

---

## 8. CI/CD Integration Patterns

### 8.1 PR Gate
Compare PR image vs. main branch image. Fail if: new critical CVEs introduced, size exceeds budget, or unexpected files added.

```bash
crane diff main:latest feature:pr-123
grype feature:pr-123 --diff main:latest
CI=true dive feature:pr-123
```

### 8.2 Release Validation
Compare release candidate vs. previous release. Generate diff report as build artifact. Fail if: vulnerability count increased, size regressed, or unauthorized changes detected. The diff report should be archived alongside release notes for audit trails.

### 8.3 Base Image Upgrade Validation
Build app on old base and new base. Compare using `diffoci diff` or `crane diff`. Fail if any application-layer file differs (indicates compatibility break) or if the new base introduces more critical CVEs than the old one.

### 8.4 Migration Guard (Dockerfile Refactoring)
After merging RUN commands, switching to multi-stage builds, or changing package managers, compare old and new Dockerfile outputs. Fail if required files missing, entrypoint changed, or image size regressed beyond threshold.

### 8.5 Scheduled Compliance (Drift Detection)
On a schedule (daily/weekly), compare current production images against a known-good golden baseline. Alert on: new layers detected, new packages installed, or new vulnerabilities above severity threshold. Useful for detecting unauthorized changes in regulated environments.

### 8.6 CI/CD Pipeline Tooling

| Platform | Tool / Action |
|---|---|
| **GitHub Actions** | `docker/docker-scout-action`, `anchore/scan-action`, custom crane/grype scripts |
| **GitLab CI** | `docker scout` CLI, `grype` CLI, custom `crane diff` jobs |
| **Jenkins** | Docker Scout plugin, Trivy plugin, shell steps with crane/grype |
| **CircleCI** | Docker Scout Orb, custom orb with Trivy |
| **ArgoCD / Flux** | Pre-deployment webhook checks using Trivy or Grype |

### 8.7 Pipeline Flow Diagram

```
Build Image → Tag with Commit SHA → Compare with Main Image →
  ├── Pass/Fail on Size Budget
  ├── Pass/Fail on Vulnerability Delta
  ├── Pass/Fail on Unexpected File Changes
  └── Pass → Push to Registry → Deploy to Environment
```

---

## 9. Example CI Workflows

### 9.1 GitHub Actions — Docker Scout Gate
```yaml
- name: Compare with Docker Scout
  uses: docker/docker-scout-action@v1
  with:
    command: compare
    image: myapp:${{ github.sha }}
    to-image: myapp:main
    exit-code: true
```

### 9.2 GitLab CI — Crane + Grype Pipeline
```yaml
compare:
  image: gcr.io/go-containerregistry/crane:latest
  script:
    - crane diff $CI_IMAGE $MAIN_IMAGE || true
    - crane manifest $CI_IMAGE | jq '.config.size' | awk '{if($1>500000000) exit 1}'

vulnerability-scan:
  script:
    - apk add grype
    - grype $CI_IMAGE --fail-on critical --only-fixed
```

### 9.3 Jenkins — Trivy Diff Gate
```groovy
stage('Compare') {
    sh 'trivy image --diff myapp:main myapp:${BRANCH_NAME} \
        --severity CRITICAL,HIGH --exit-code 1'
}
```

---

## 10. Best Practices for Enterprise Adoption

### 10.1 Make Comparison a Mandatory CI Gate
Every image that reaches production must pass comparison checks: layer changes, new vulnerabilities (critical/high), and size regression against a per-service budget.

### 10.2 Set Size Budgets Per Service
```yaml
services:
  api-gateway: 200MB
  auth-service: 150MB
  frontend: 300MB
  worker: 400MB
```

### 10.3 Maintain a Golden Baseline
For each service, keep the latest known-good production image as the baseline. New builds compare against it. Passing builds become the new baseline.

### 10.4 Automate Base Image Upgrade Validation
Build twice (old base, new base), compare with `diffoci diff` or `crane diff`, validate OS-layer-only changes, verify vulnerability delta.

### 10.5 Save Diff Reports for Compliance
Archive diff reports as build artifacts. Each report: files added/modified/deleted, package version changes, vulnerability delta, layer digest changes.

### 10.6 Generate SBOMs at Every Build
```bash
syft <image> -o spdx-json > sbom.spdx.json
trivy image --format cyclonedx --output sbom.cdx.json <image>
```
Compare SBOMs between releases per regulations (US EO 14028, EU Cyber Resilience Act).

### 10.7 Optimize Layer Caching
Use `skopeo inspect` or `crane diff` on consecutive builds. Matching layers above the app layer = caching works. Different layers = cache config needs fixing.

### 10.8 Tool Selection Decision Tree

The choice of tool depends on the specific comparison you need to perform. Use this decision tree to quickly identify the right tool for your scenario:

```
Need to compare layers & composition?     → Dive
Need side-by-side file diff of two images? → crane diff or diffoci
Need vulnerabilities between versions?     → Grype/Trivy (OSS) or Docker Scout/Snyk
Need package-level diff (apt, npm, pip)?   → Syft (SBOM) or Trivy
Need reproducible build validation?        → diffoci
Need quick metadata diff without pull?     → Skopeo + jq
Need SBOM for compliance?                  → Syft or Trivy
Need Dockerfile quality checks?            → Hadolint
Need multi-registry copy + compare?        → Skopeo or crane
Need enterprise policy gating at scale?    → Docker Scout, Snyk, or JFrog Xray
```

---

## 11. Summary and Recommendations

| Use Case | Recommended Tool(s) |
|---|---|
| Ad-hoc layer exploration | Dive |
| CI/CD file-level diff | crane diff, diffoci |
| CI/CD vulnerability gate | Grype, Trivy, Docker Scout |
| SBOM compliance | Syft, Trivy |
| Base image migration | diffoci, crane diff |
| Reproducible builds | diffoci |
| Multi-registry management | Skopeo, crane |
| Enterprise policy gating | Docker Scout, Snyk, JFrog Xray |
| Dockerfile quality | Hadolint |
| Supply chain transparency | Syft + Grype |

### Recommended Enterprise Stack (Banking/Compliance)

1. **Hadolint** — Pre-build Dockerfile quality.
2. **Syft** — SBOM at every build (supply chain transparency).
3. **Grype** — Vulnerability scan and diff (CI gate).
4. **crane diff** + **diffoci** — File-level and layer comparison (migration validation).
5. **Trivy** or **Docker Scout** — Policy-based CI/CD gating (automated compliance).
6. **Dive** — Ad-hoc bloat investigation (developer debugging).

This stack is entirely open-source (except Docker Scout, replaceable with Trivy) and covers all nine use cases from Section 1 with auditable artifacts for compliance reviews.

---

*Last updated: July 2026*
*Maintainer: Jack Liu Shurui — Solution Architect, Cymbal Bank*
*Repository: github.com/jackliusr/research — see `technology/` directory*
