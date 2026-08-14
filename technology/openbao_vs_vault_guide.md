# OpenBao vs HashiCorp Vault — The Complete Secrets-Management Platform Comparison

> **Author:** Jack Liu Shurui | **Role:** Solution Architect, Crédit Agricole CIB
> **Last Updated:** August 2026 | **Version:** 1.0
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** security / infrastructure platform deep-dive (companion to [cloud_providers_guide.md](cloud_providers_guide.md), [financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md), [kargo_gitops_guide.md](kargo_gitops_guide.md))

---

## Table of Contents

1. [Secrets-Management Context](#1-secrets-management-context)
2. [HashiCorp Vault — The Industry Standard](#2-hashicorp-vault--the-industry-standard)
3. [The Licensing Drama and the Fork](#3-the-licensing-drama-and-the-fork)
4. [Feature Comparison](#4-feature-comparison)
5. [Differences and Compatibility](#5-differences-and-compatibility)
6. [Migration](#6-migration)
7. [Community and Adoption](#7-community-and-adoption)
8. [Selection Guidance](#8-selection-guidance)
9. [Worked Example — Kubernetes Deployment and a Bank's Migration](#9-worked-example--kubernetes-deployment-and-a-banks-migration)
10. [The Future (2026+)](#10-the-future-2026)
11. [Glossary](#11-glossary)
12. [Fact-Check Notes and Sources](#12-fact-check-notes-and-sources)

---

## 1. Secrets-Management Context

### 1.1 What is a "secret"?

A **secret** is any piece of information that must be tightly controlled because its exposure grants access, identity, or capability. The canonical catalogue:

| Secret type | Examples | What it unlocks |
|---|---|---|
| **API keys / tokens** | `sk-...`, OAuth bearer tokens, service-account keys | Programmatic access to APIs and services |
| **Passwords / credentials** | Database user passwords, app-to-app credentials | Authentication to systems |
| **Certificates** | TLS certs + private keys, client certs | Encrypted channels, mutual TLS identity |
| **Encryption keys** | Data-encryption keys, signing keys | Encrypted data, signed artefacts |
| **Configuration secrets** | SSH keys, cloud access keys (AWS `AKIA...`), JWT signing secrets | Infrastructure and platform access |

The distinguishing property of a secret is *not* its format — it is that **possession implies power**, so it must be stored, transmitted, and used under the same discipline as cash. A secret committed to a Git repo, pasted into a chat, or hard-coded in a container image is a breach waiting to happen (see [penetration_testing_execution_standard_guide.md](penetration_testing_execution_standard_guide.md) for how these leaks are found).

### 1.2 What is secrets management?

**Secrets management** is the discipline of controlling the full lifecycle of secrets. The four pillars:

1. **Centralised storage** — secrets live in one hardened, encrypted store instead of scattered config files, env vars, and vault-like directories on application servers.
2. **Rotation** — secrets change on a schedule or on demand, so a leaked credential has a short useful life (this is the single highest-leverage control; see §2.3 for *dynamic secrets*, which make rotation automatic).
3. **Access control** — fine-grained, identity-based policies decide *who* (which workload, which human, which role) can *do what* (read, write, list, revoke) with *which* secret.
4. **Audit** — every access is logged immutably (who, what, when, from where), which is what makes compliance attestation (PCI-DSS, SOC 2, MAS TRM, ISO 27001) possible.

A secrets manager is the technical embodiment of these four pillars. Without one, organisations typically suffer the "credential sprawl" pattern: thousands of hard-coded secrets, no inventory, no rotation, and no way to answer an auditor's "who can access the production database password?" question.

### 1.3 Why Vault became the standard

**HashiCorp Vault** is the de-facto standard for open-source secrets management. Launched in **April 2015**, it was the first tool to combine the full pillar set in one product:

- A plugin-based **secrets-engine** architecture (KV, database, PKI, transit, cloud credentials — §2.2);
- **Dynamic secrets** — credentials generated on demand with short leases, instead of static shared passwords;
- **Encryption-as-a-service** via the transit engine — applications can encrypt/decrypt without ever holding the keys;
- Identity-based policies, multiple **auth methods**, and a **templated audit log**;
- A simple, scriptable **HTTP API** plus the `vault` CLI, and (later) first-class Kubernetes integration.

No competing tool (commercial or OSS) matched this breadth at the time. AWS Secrets Manager, Azure Key Vault, and GCP Secret Manager are strong *managed* alternatives but are platform-locked and mostly static-secret stores (see [cloud_providers_guide.md](cloud_providers_guide.md) for the provider-by-provider security services). Kubernetes Secrets are a primitive, not a platform — base64 blobs in etcd with weak default access control. By the mid-2020s Vault was so entrenched that "Vault" and "secrets management" were near-synonyms in architecture conversations — which is exactly why its licensing change (2023, §3) mattered so much to so many organisations.

### 1.4 The landscape and the players

| Alternative | Type | Strengths | Limits (vs Vault/OpenBao) |
|---|---|---|---|
| **AWS Secrets Manager** | Managed (AWS) | Zero ops, rotation for RDS, IAM integration | AWS-locked, static secrets focus, per-secret cost |
| **Azure Key Vault** | Managed (Azure) | HSM-backed, tight Entra ID integration, cert management | Azure-locked, no dynamic/lease model |
| **GCP Secret Manager** | Managed (GCP) | IAM-native, cheap, versioning | GCP-locked, no dynamic secrets |
| **Kubernetes Secrets** | Platform primitive | Built-in, simple | Base64-only, weak RBAC by default, no rotation/audit |
| **HashiCorp Vault** | Self-hosted / managed | Full platform: dynamic secrets, PKI, transit, audit | Ops cost; licensing (§3) |
| **OpenBao** | Self-hosted (OSS fork) | Vault-OSS feature set, MPL 2.0, open governance | Younger project, fewer enterprise extras (§4) |

This guide compares the two platform players — **Vault** (HashiCorp, now IBM-owned) and **OpenBao** (the Linux-Foundation-governed fork) — across history, licensing, governance, features, compatibility, migration, adoption, and selection. It is written for enterprise architects in regulated industries (banking context in §8.4), with a worked Kubernetes deployment and migration scenario in §9.

---

## 2. HashiCorp Vault — The Industry Standard

### 2.1 Overview and history

**Vault** is a tool for "securely accessing secrets" — API keys, passwords, certificates, and encryption keys — via a unified interface with tight access control and a detailed audit log. It was first released by HashiCorp in **April 2015** (v0.1.0), part of the same tooling family as Terraform, Consul, and Nomad. The core architectural ideas from the very first release still define the product today:

- **Sealed/unsealed state** — the master key is split (Shamir's secret sharing) so no single person can decrypt the store; on restart, Vault is *sealed* until the key shares are recombined.
- **Storage backends behind an encryption barrier** — all data is encrypted before it touches the storage layer (file, Consul, Raft, cloud buckets).
- **Leases and renewals** — every secret has a time-to-live; nothing is permanent.
- **Pluggable secrets engines and auth methods** — the architecture that made the ecosystem possible.

Over the years Vault added: dynamic database credentials, a full PKI engine (issuing short-lived TLS certs), the transit encryption-as-a-service engine, Kubernetes auth, OIDC, auto-unseal via cloud KMS, integrated Raft storage (removing the Consul dependency), and a Terraform-provider + operator ecosystem.

### 2.2 Vault features — the core catalogue

**Secrets engines** (the pluggable backends that produce or store secrets):

| Engine | What it does |
|---|---|
| **KV v1 / v2** | Static secrets (key/value). v2 adds versioning, CAS, soft/hard delete |
| **Database** | *Dynamic* credentials for PostgreSQL, MySQL, Oracle, MongoDB, MSSQL, etc. — unique creds per lease, auto-rotation of root creds |
| **PKI** | On-demand X.509 certificates with short TTLs, CRLs, and automatic renewal — the basis of modern mTLS and SPIFFE-style workload identity |
| **Transit** | Encryption-as-a-service — encrypt/decrypt/sign/verify via API without the application ever seeing the keys |
| **AWS / Azure / GCP / Alibaba** | Dynamic cloud credentials (IAM users, STS tokens, service principals) |
| **SSH** | One-time SSH passwords and signed SSH client certificates |
| **Identity** | Entity/alias management, group membership, and policy templating across auth methods |
| **Consul / TOTP / LDAP / RabbitMQ / others** | Longer tail of integrations |

**Auth methods** (how a client proves who it is):

| Auth method | Typical use |
|---|---|
| **Token** | The baseline; every other method produces a token |
| **AppRole** | Machine-to-machine auth for CI/CD and services (secret ID + role ID) |
| **Kubernetes** | Workload identity via service-account JWTs — the standard for pods |
| **LDAP / Active Directory** | Human users against the corporate directory |
| **OIDC / JWT** | SSO (Okta, Azure AD/Entra ID, Keycloak, GitHub) |
| **TLS certs, Username/password, Cloud IAM** | Additional paths for humans and machines |

**Core platform features:**

- **Dynamic secrets** — on-demand credentials with short leases. The database engine creates a unique user per request and revokes it when the lease expires; this is the single biggest security upgrade over static secrets, because the credential only exists while the application is using it.
- **Leasing & renewal** — every secret (and every auth token) has a TTL; the API supports renew and revoke.
- **Rotation** — manual/periodic rotation of static secrets, automatic root-credential rotation in the database engine, and PKI certificate rotation via TTLs.
- **Audit logging** — file, syslog, and socket audit devices capturing every request, with configurable HMAC of sensitive values.
- **Policies** — HCL/JSON-based path ACLs (later refined with templating and identity-derived policies).
- **Auto-unseal** — delegate unsealing to a cloud KMS (AWS KMS, Azure Key Vault, GCP KMS) or an HSM, removing the human ceremony.
- **UI** — a web console for operators and self-service secret browsing.

### 2.3 Vault editions

| Edition | What you get | Licensing |
|---|---|---|
| **Vault OSS (Community Edition)** | Everything in §2.2; single cluster, Raft HA | MPL 2.0 (≤1.14.x) → BSL 1.1 (1.15+; §3) |
| **Vault Enterprise** | + replication, namespaces, HSM, performance standbys, MFA, control groups, Sentinel policies, seal wrap, disaster-recovery mode | Commercial |
| **HCP Vault** (HashiCorp Cloud Platform) | Managed Vault (dedicated clusters) on AWS/Azure; now HCP Vault Radar for secrets scanning | SaaS subscription |

### 2.4 Vault Enterprise — the commercial layer

The Enterprise edition is where HashiCorp's revenue model lives, and it is the *feature gap* that defines the OpenBao story. The flagship Enterprise capabilities:

- **HSM integration (PKCS#11)** — seal/unseal keys and root keys held inside a hardware security module; mandatory in many banking and government deployments (§8.4).
- **Performance replication** — read scaling across clusters/regions with low latency; **Disaster Recovery (DR) replication** — write-replica failover and rekeying across data centres.
- **Namespaces** — tenant isolation on a single cluster (isolated policies, mounts, and audit per namespace) — the multi-tenant/ multi-BU feature.
- **Performance standbys** — additional read replicas that keep Vault available during upgrades and scale reads.
- **MFA (Okta, TOTP, Duo, PingID)** — step-up authentication on sensitive paths.
- **Control groups** — quorum approval for sensitive operations (e.g., "3 of 5 approvers must sign off").
- **Sentinel policies** — policy-as-code with a rule language (now being superseded by OPA-style approaches).
- **Seal wrap / response wrapping** — encryption of transit data inside the cluster.

HashiCorp's positioning post-2023 is unambiguous: the OSS core is the on-ramp; the enterprise layer is the product. That positioning is precisely what the OpenBao fork rejects.

### 2.5 Vault in the industry

Vault is the default answer in enterprise architecture reviews: it is the most widely deployed self-hosted secrets platform, it anchors HashiCorp's (now IBM's) product strategy, it has a mature ecosystem (Terraform provider, Vault Secrets Operator, External Secrets Operator support, dozens of community plugins), and it dominates the "dynamic secrets + PKI" use case that managed cloud alternatives do not cover. It is also a favourite acquisition-bait asset: IBM completed its **$6.4B acquisition of HashiCorp on 27 February 2025**, making Vault (and HCP) part of IBM's hybrid-cloud stack alongside Red Hat — a twist with ironic consequences for the fork story (§3.4), since IBM engineers had helped launch OpenBao.

---
## 3. The Licensing Drama and the Fork

### 3.1 The license change: MPL 2.0 → BSL 1.1 (August 2023)

On **10 August 2023**, HashiCorp announced that **all future releases of all its products** (Vault, Terraform, Consul, Nomad, Boundary, and several libraries) would move from the **Mozilla Public License v2.0 (MPL 2.0)** to the **Business Source License v1.1 (BSL/BUSL)**. HashiCorp's stated rationale: cloud vendors were offering HashiCorp products as managed services and competing on HashiCorp's own turf (the "strip-mining" argument used by many open-source companies that made the same move — Redis, Elastic, Confluent, MongoDB, Grafana Labs).

**What BSL 1.1 actually means:**

- Source code remains *visible* (it is "source-available", not open source by OSI definition).
- Free use is allowed for non-production / internal production use *below* a usage threshold (the "Additional Use Grant" — for HashiCorp, largely: don't sell it as a hosted competitor).
- The license automatically converts to Apache 2.0 after a **Change License time** — for HashiCorp products this is **four years** after each release (so BSL releases eventually become Apache-2.0-licensed).
- It is **not** an OSI-approved open-source license; the Open Source Initiative and the FSF both explicitly call BSL non-open-source.

The practical effect for enterprises: **Vault 1.15+ (from November 2023) is BSL-licensed**, meaning no redistribution as a competing service, no unfettered commercial embedding, and a permanent "ask the vendor" cloud over downstream use. For a *consumer* of Vault OSS the change was mostly invisible; for the ecosystem (managed-service providers, distros, vendors embedding Vault, security consultancies shipping Vault-based products) it was a seismic event.

### 3.2 The community reaction

The reaction was fast and loud:

- The **Terraform fork** was announced within weeks: **OpenTofu** (August 2023), backed by a coalition of companies (Gruntwork, Spacelift, env0, Scalr, Harness, etc.), and hosted by the Linux Foundation.
- For Vault, a community proposal for a fork ("OpenVault" proposals circulated on GitHub in September 2023) got traction.
- Hacker News, Reddit, and HashiCorp's own Discuss forum filled with threads questioning the move and debating whether Vault OSS would atrophy ("the open-source on-ramp gets less investment when the revenue is in Enterprise").
- Skeptics noted the four-year BSL→Apache conversion made the license *temporarily* restrictive; optimists noted Vault's OSS releases did continue (1.15 → 1.16 → …) — but the *governance* question ("who owns the roadmap?") was now permanently on the table.

### 3.3 OpenBao — the fork (December 2023)

**OpenBao** emerged from the LF Edge community: the first working-group meeting was held **9 November 2023**, and the project was **publicly announced in December 2023** (the repository appeared ~8–9 December; press coverage followed 11–18 December). Key facts:

| Fact | Detail | Verified? |
|---|---|---|
| **Fork point** | Forked from HashiCorp Vault CE **1.14.x** — specifically a commit between v1.14.8 and v1.14.9 (the last MPL-2.0-licensed line, before BSL applied at 1.15) | ✅ (project statements; secondary sources agree) |
| **License** | **MPL 2.0** (same as pre-BSL Vault) — an OSI-approved open-source license | ✅ (openbao.org, GitHub) |
| **Origin** | Initiated primarily by **IBM engineers** working on the LF Edge **Open Horizon** project (Joe Pearson, Open Horizon project chair, among the organisers); started as an incubating project within **LF Edge** | ✅ (TechTarget, Dec 2023) |
| **Governance** | Linux Foundation family: began under LF Edge, later a Linux Foundation project; accepted as an **OpenSSF sandbox project** (Open Source Security Foundation — the LF's security-focused foundation) | ✅ (OpenSSF announcement; openbao.org) |
| **CNCF status** | **Not a CNCF project.** No CNCF sandbox/incubating/graduated status as of August 2026. (One Russian-language secondary source claims "CNCF participant status"; this appears to conflate the Linux Foundation/OpenSSF with the CNCF — flag: treat as unverified/incorrect.) | ⚠️ (see §12) |
| **Mission** | Maintain an open, community-driven, API-compatible Vault fork with open governance and OSI-approved licensing | ✅ (project FAQ) |

**Naming and early releases:** "Bao" (报 / bào) is a Mandarin word meaning *to hold dear / to protect* — a deliberate nod to Vault's Chinese name usage. The project shipped 1.14.x releases aligned with the fork point, then moved to an independent versioning scheme with **v2.0.0 (April 2024)** — its first release introduced paginated list APIs (a feature Vault still lacked). By **July 2026** the project was at **v2.6.1** (§10).

**Why IBM engineers?** Open Horizon (an LF Edge edge-computing project) had integrated Vault for device secrets and was directly exposed to the BSL change; its engineers had the skills, the LF infrastructure, and the institutional motive. The irony that IBM would later *buy* HashiCorp — and thus own both Vault and a fork its own engineers helped create — was not lost on the community (§3.4).

### 3.4 Governance today

- **OpenBao**: open governance under the Linux Foundation. Maintainers are a mix of IBM, GitLab, SAP-affiliated (ApeiroRA), and independent contributors; decisions go through GitHub issues/discussions; the project operates an OpenSSF-aligned security process (security mailing list, sigstore-signed artefacts, SBOMs — the `bao-hsm` release artefacts ship SBOMs and sigstore signatures). The project's home is the **Linux Foundation**, not the CNCF — an important correction to common belief (§12).
- **Vault**: single-vendor governance. Roadmap, releases, and security fixes are controlled by HashiCorp (now IBM). OSS releases continue but the commercial enterprise layer (§2.4) is where investment concentrates.

### 3.5 Timeline table

| Date | Event |
|---|---|
| **April 2015** | HashiCorp Vault first released (v0.1.0) — MPL 2.0 |
| **Aug 10, 2023** | HashiCorp announces MPL 2.0 → BSL 1.1 for all products, effective for releases after the date |
| **Aug–Sep 2023** | OpenTofu (Terraform fork) announced; OpenVault fork proposals circulate |
| **Nov 9, 2023** | OpenBao first working-group meeting (LF Edge / Open Horizon circle) |
| **Nov 2023** | Vault 1.15 ships under BSL 1.1 |
| **Dec 2023** | OpenBao publicly announced; openbao/openbao repository goes live; 1.14.x releases |
| **March 2024** | OpenBao 2.0.0 (independent versioning begins; paginated lists) |
| **2024** | OpenBao consolidates under Linux Foundation governance; community growth |
| **Apr 2024 → Feb 27, 2025** | IBM announces ($6.4B) and completes acquisition of HashiCorp |
| **2025** | OpenBao accepted as **OpenSSF sandbox project**; GitLab publicly builds CI/CD secrets on OpenBao (FOSDEM 2025); SAP ApeiroRA (EU-funded) adopts OpenBao as its secrets/key-management solution; namespaces land in the 2.x series |
| **Feb 4, 2026** | OpenBao **2.5.0**: horizontal read scalability + HSM (PKCS#11) seal support (`bao-hsm` binaries) |
| **Jul 14–22, 2026** | OpenBao **2.6.0/2.6.1**: per-namespace sealing (tenant key-material partitioning), auto-unseal plugins |
| **2026** | HashiCorp Vault moves to a **2.0 series** (CE 2.0.4 current mid-2026; Enterprise 2.0.3; 1.19–1.21 still patched) |

---

## 4. Feature Comparison

### 4.1 Secrets engines — parity check

OpenBao forked the complete engine set, so **every core engine is present and API-compatible**. Verified list (present in both, as of OpenBao 2.6.x / Vault 2.0.x):

| Secrets engine | Vault OSS | OpenBao | Notes |
|---|---|---|---|
| KV v1 / v2 (versioned) | ✅ | ✅ | Parity |
| Database (dynamic creds) | ✅ | ✅ | Parity; OpenBao actively extends provider support |
| PKI (certificates) | ✅ | ✅ | Parity; OpenBao has contributed PKI improvements (e.g., paginated cert lists) |
| Transit (encryption-as-a-service) | ✅ | ✅ | Parity |
| AWS / Azure / GCP / Alibaba (cloud creds) | ✅ | ✅ | Parity |
| SSH (OTP / signed certs) | ✅ | ✅ | Parity |
| Identity (entities/aliases) | ✅ | ✅ | Parity |
| Consul, TOTP, LDAP, RabbitMQ, etc. | ✅ | ✅ | Long tail ported with the fork |

### 4.2 Auth methods — parity check

| Auth method | Vault OSS | OpenBao | Notes |
|---|---|---|---|
| Token | ✅ | ✅ | Parity |
| AppRole | ✅ | ✅ | Parity |
| Kubernetes (service-account JWT) | ✅ | ✅ | Parity |
| LDAP / Active Directory | ✅ | ✅ | Parity |
| OIDC / JWT | ✅ | ✅ | Parity |
| TLS certificates, username/password, cloud IAM | ✅ | ✅ | Parity |

### 4.3 Core platform features — parity check

| Capability | Vault OSS | OpenBao | Notes |
|---|---|---|---|
| Dynamic secrets (leases, renew, revoke) | ✅ | ✅ | Parity |
| Seal/unseal (Shamir + auto-unseal via AWS/Azure/GCP/OCI/AliCloud KMS) | ✅ | ✅ | Parity (OpenBao 2.6.0 added plugin-based auto-unseal) |
| Audit logging (file/syslog/socket) | ✅ | ✅ | Parity |
| Policies (ACL + templated) | ✅ | ✅ | Parity |
| Raft integrated storage / HA | ✅ | ✅ | Parity |
| Rekey / rotate root | ✅ | ✅ | Parity |
| Web UI | ✅ | ✅ | Parity (OpenBao ships its own UI fork) |

### 4.4 The missing pieces — what OpenBao lacked, and the state of play

This is the heart of the comparison. OpenBao started life without Vault's *Enterprise-only* features; the community has been re-implementing them under MPL 2.0, at remarkable pace:

| Enterprise feature | Vault Enterprise | OpenBao (Aug 2026) | Status |
|---|---|---|---|
| **HSM seal (PKCS#11)** | ✅ | ✅ since **2.5.0** (Feb 2026) — `bao-hsm` cgo binaries, PKCS#11 seal documented | Gap **closed** in 2026 |
| **Namespaces** | ✅ | ✅ added in the 2.x series; **2.6.0** added *per-namespace sealing* (each tenant's storage encrypted under distinct key material — beyond what Vault offers) | Gap **closed** (community implementation) |
| **Performance read scaling** | ✅ (performance standbys) | ✅ since **2.5.0** — "horizontal read scalability" (read replicas) | Gap **closed** via a different mechanism |
| **Replication (performance / DR)** | ✅ | ❌ **not present** as of 2.6.1; HA via Raft + snapshot/restore; replication is a long-standing roadmap item | **Open gap** — the biggest remaining difference |
| **MFA (Okta/TOTP/Duo/PingID)** | ✅ | ⚠️ partial/community work; not verified as full parity | Flag: unverified |
| **Control groups** | ✅ | ❌ not present | Open gap |
| **Sentinel policy-as-code** | ✅ | ❌ not present | Open gap |
| **Seal wrap / response wrapping internals** | ✅ | ⚠️ wrapping exists; seal-wrap internals not verified | Flag: unverified |

**Honest summary of the gap analysis:** by mid-2026 OpenBao has closed the *headline* enterprise gaps that mattered most to enterprises — HSM support, namespaces, and read scalability — and even innovated beyond Vault (per-namespace sealing). The **remaining substantive gaps are replication (performance/DR)**, control groups, and Sentinel-class policy engines. Any document claiming "OpenBao is missing HSM/namespaces" is simply out of date as of the 2.5/2.6 releases.

### 4.5 Master feature comparison table

| Feature | Vault OSS (1.14-/BSL) | Vault Enterprise | OpenBao (2.6.x) |
|---|---|---|---|
| License | BSL 1.1 (1.15+); MPL 2.0 (≤1.14) | Commercial | **MPL 2.0** |
| Governance | HashiCorp (IBM) | HashiCorp (IBM) | **Linux Foundation / OpenSSF** |
| Secrets engines (KV/db/PKI/transit/AWS/SSH) | ✅ | ✅ | ✅ |
| Auth methods (token/AppRole/k8s/LDAP/OIDC) | ✅ | ✅ | ✅ |
| Dynamic secrets + leases + audit | ✅ | ✅ | ✅ |
| Raft HA + auto-unseal | ✅ | ✅ | ✅ |
| Namespaces | ❌ | ✅ | ✅ (2.x; per-ns sealing in 2.6.0) |
| HSM (PKCS#11) seal | ❌ | ✅ | ✅ (2.5.0+) |
| Read scaling | ❌ | ✅ (perf standbys) | ✅ (2.5.0 read replicas) |
| Performance/DR replication | ❌ | ✅ | ❌ |
| MFA | ❌ | ✅ | ⚠️ (unverified) |
| Control groups | ❌ | ✅ | ❌ |
| Sentinel policies | ❌ | ✅ | ❌ (policy engine is plain ACL/templated) |
| Vendor SLA/support | ❌ (community) | ✅ | ⚠️ (8+ vendors offer commercial support per ecosystem listings — unverified detail) |
| Version (Aug 2026) | 2.0.x | 2.0.x | 2.6.x |

---

## 5. Differences and Compatibility

### 5.1 CLI: `vault` vs `bao`

| Aspect | Vault | OpenBao |
|---|---|---|
| Binary | `vault` | `bao` |
| Dev server | `vault server -dev` | `bao server -dev` |
| Command set | `vault kv get …`, `vault secrets enable …`, `vault token create …` | Identical command names: `bao kv get …`, `bao secrets enable …`, `bao token create …` |
| Shell completions / output formats | `-format=json/yaml/table` | Same flags |

The OpenBao CLI is a drop-in rename: commands, flags, positional arguments, and output formats mirror Vault's. (The `vault` command itself is not shipped — scripts calling `vault` need a `vault → bao` alias or a wrapper during migration; see §6.)

### 5.2 Environment variables: `VAULT_ADDR` vs `BAO_ADDR`

| Variable family | Vault | OpenBao |
|---|---|---|
| Server address | `VAULT_ADDR` | `BAO_ADDR` (per project docs, `BAO_*` variables take precedence and legacy `VAULT_*` variables are still honoured as fallbacks for compatibility) |
| Token | `VAULT_TOKEN` | `BAO_TOKEN` (same fallback behaviour) |
| Other | `VAULT_NAMESPACE`, `VAULT_CACERT`, `VAULT_SKIP_VERIFY`, … | `BAO_*` equivalents with the same compatibility fallback |

This dual-prefix design is deliberate: OpenBao documents that it reads the `VAULT_*` set when no `BAO_*` value is set, which is what makes existing CI pipelines and shell profiles work with only a binary swap. (Flag: exact fallback precedence should be confirmed against the docs of the specific OpenBao version you deploy.)

### 5.3 Configuration

Both use the same **HCL configuration format** and the same core blocks:

```hcl
storage "raft" { path = "/data/bao" }        # same as Vault
listener "tcp" { address = "0.0.0.0:8200" }  # same default port 8200
seal "awskms" { region = "ap-southeast-1" }  # auto-unseal blocks identical
api_addr = "https://bao.example.com:8200"
disable_mlock = false
```

Practical differences to expect:

- OpenBao config keys mirror Vault's; enterprise-only keys (e.g., replication/performance-standby related settings) are absent because the features are absent.
- The OpenBao documentation ships its own reference for every block (`storage`, `listener`, `seal`, `telemetry`, …), so a Vault config almost always imports as-is.
- Default paths: OpenBao keeps the same listener defaults; storage paths are whatever you set (migration keeps the same raft path so data is reused in place — §6).

### 5.4 API compatibility

OpenBao's stated, maintained goal is **API compatibility with Vault**: the HTTP API (`/v1/…` paths, JSON envelopes, status codes, `X-Vault-Token` header, lease/TTL semantics) is carried over from the fork, and the project commits to keeping it compatible. Practical consequences:

- Clients written against Vault's HTTP API work against OpenBao (curl, Go/Python/Java clients, Terraform provider, External Secrets Operator) with the base URL and token header swapped.
- The OpenBao SDK is published (`github.com/openbao/openbao/sdk/v2`) for plugin and client development.
- Minor divergences are possible in edge cases (new OpenBao features like paginated lists or namespaces are additive; Vault 2.x may rename/behave differently in isolated endpoints). Treat "drop-in" as a *design goal with a very high — not absolute — compatibility rate*.

### 5.5 Storage backends

| Storage backend | Vault OSS | OpenBao | Notes |
|---|---|---|---|
| Raft (integrated) | ✅ | ✅ | Default; the migration path relies on this (§6) |
| Consul | ✅ | ✅ | Parity |
| File | ✅ | ✅ | Parity |
| In-memory (dev) | ✅ | ✅ | Parity |
| Cloud object stores (S3/GCS/Azure) via Consul | ✅ (with Consul) | ✅ (with Consul) | Parity |

### 5.6 Compatibility summary — is OpenBao a "drop-in replacement"?

**Honest assessment:**

- ✅ **Yes, for Vault OSS (Community Edition) workloads:** same engines, auth methods, API, CLI (renamed), config format, storage backends, and an officially documented in-place migration from Vault CE 1.14.x (§6). For the "Vault OSS in production" population — which is the majority of real deployments — OpenBao is the most drop-in replacement that exists in the secrets-management world.
- ⚠️ **With caveats:** (1) external/community plugins compiled against Vault must be rebuilt against the OpenBao SDK; (2) anything using `VAULT_*` env vars should be switched to `BAO_*` (fallbacks exist but are best-effort); (3) Vault 1.15+ BSL releases introduced features/behaviours not in the fork base — check feature-by-feature if you're migrating *from* a BSL-era release (OpenBao generally tracks Vault API surface, but not byte-for-byte); (4) the UI, while present, is the fork's own.
- ❌ **No, for Vault Enterprise:** any deployment using replication, control groups, Sentinel, or enterprise MFA cannot move to OpenBao without losing those capabilities (HSM, namespaces, and read scaling have *open-source equivalents* in OpenBao, but DR replication remains missing).

---

## 6. Migration

### 6.1 Vault → OpenBao — the official path

OpenBao publishes an official guide: **"In-Place Migration from Vault CE"** (openbao.org/docs/guides/migration). The principle: Vault CE 1.14.x and OpenBao share the storage format and API, so the same raft data can be reused — you swap the binary, not the data. Canonical steps:

| Step | Action | Notes |
|---|---|---|
| 1. **Pre-flight audit** | Inventory engines, auth methods, policies, plugins, and any Enterprise features in use | Enterprise-feature users: stop here unless you accept the gaps in §4.4 |
| 2. **Back up** | Take a raft snapshot: `vault operator raft snapshot save backup.snap` | Also back up config files and unseal keys/root token in a vault (the real one) |
| 3. **Downgrade/align** | Ensure source is Vault CE 1.14.x-compatible (OpenBao is forked from the 1.14 line); the migration guide targets a Vault CE cluster | If running BSL-era Vault (1.15+), validate feature/behaviour differences first |
| 4. **Stop Vault** | Gracefully stop all Vault nodes; do NOT let a partial raft quorum run during swap | Planned maintenance window |
| 5. **Swap binary** | Replace `vault` with `bao` on every node; keep the same raft data path | Optionally add a `vault`→`bao` alias for legacy scripts |
| 6. **Adjust config** | Port config to OpenBao syntax (usually identical); rename `VAULT_*` env vars to `BAO_*` where possible | Storage block stays the same — data is reused in place |
| 7. **Start & unseal** | `bao server` … unseal with existing key shares; verify raft leader election | Unseal keys and root token carry over |
| 8. **Verify** | Read-back critical secrets, run a dynamic-db cred cycle, check audit log, test auth methods (AppRole/k8s/OIDC) | Create a verification checklist per engine |
| 9. **Cut over clients** | Point applications at the same address; update CLI aliases; monitor | DNS/load-balancer unchanged if same address |

### 6.2 Migration considerations

- **What carries over:** raft data (all secrets, keys, policies, leases), unseal keys/root token, config structure, API surface, audit history (stored in raft), UI logins.
- **Plugins:** built-in engines carry over. *External/community plugins* must be recompiled against the OpenBao SDK — this is the most common migration surprise.
- **Environment:** switch `VAULT_ADDR`/`VAULT_TOKEN` to `BAO_ADDR`/`BAO_TOKEN` in pipelines, Helm values, and operators. The documented `VAULT_*` fallback helps, but don't rely on it forever.
- **Rollback:** because the raft data is shared, rolling back means restoring a Vault CE binary onto the same data (keep the pre-migration snapshot!). OpenBao writes may use newer raft/format features over time — re-verify rollback viability per version.
- **Risk profile:** the migration is an in-place data-path swap, so it is lower-risk than a re-platform (no re-import of secrets), but it is *not* a zero-risk flag-day: test in a staging clone first, and treat the first production migration as a change-managed event.

### 6.3 The reverse path: OpenBao → Vault

Rare, but possible in principle: same API, same raft format lineage. There is **no official HashiCorp guide** for importing OpenBao data into Vault; organisations that revert typically re-import secrets or restore a Vault-side snapshot taken before the migration. Treat it as a recovery scenario, not a documented path — and note that Vault 2.x and OpenBao 2.6.x are now independently evolved, so cross-import guarantees are thinner than the official Vault→OpenBao direction.

### 6.4 Migration table (condensed)

| Step | Action | Key notes / risks |
|---|---|---|
| 1 | Inventory & feature audit | Enterprise features = blocker for full parity |
| 2 | Raft snapshot backup | Non-negotiable; store off-box |
| 3 | Align on Vault CE 1.14 line | BSL-era sources need extra validation |
| 4 | Stop cluster | Maintenance window; quorum discipline |
| 5 | Swap binary (vault → bao) | Keep raft path; alias legacy commands |
| 6 | Config + env-var port | `VAULT_*` → `BAO_*`; storage block unchanged |
| 7 | Start, unseal, verify | Test every engine + auth method |
| 8 | Cut over clients | Address unchanged = low client churn |
| 9 | Rebuild external plugins | Compile against OpenBao SDK |
| 10 | Monitor & keep rollback snapshot | Watch audit log, lease renewals, raft health |

---
## 7. Community and Adoption

### 7.1 The OpenBao community

- **Maintainers:** a distributed team under Linux Foundation governance, including people with IBM (Open Horizon) roots, GitLab security engineers, and independent contributors; decisions are made in the open (GitHub issues/discussions, weekly community calls). Leadership roles are documented on the project site — check the current maintainers list for names; the cast has been growing steadily through 2025–2026.
- **Governance bodies:** Linux Foundation (project home, began under LF Edge) + **OpenSSF sandbox membership** (security best-practices, coordinated disclosure via the `openbao-security@lists.lfedge.org` list, sigstore signing, SBOMs on release artefacts).
- **Shape of the project:** ~25 repos under the `openbao` GitHub org (core, helm chart, docs, UI, SDK, various tooling). Momentum indicators: regular minor releases (5–6 per year), a published roadmap (e.g., "Roadmap and Community Direction for 2025–2026"), and conference presence (Open Source Summit Europe, FOSDEM).

### 7.2 Adoption — who runs OpenBao

Verified/credible public signals (approximate, check live for current lists):

| Adopter | Evidence |
|---|---|
| **GitLab** | FOSDEM 2025 talk: "OpenBao @ GitLab — building native secrets for GitLab CI/CD pipelines" — GitLab uses/embeds OpenBao for CI/CD secret management |
| **SAP (ApeiroRA initiative)** | OpenBao's GitHub org states SAP's ApeiroRA — supported and funded by the EU — adopted OpenBao as its secrets and key-management solution, and that adoption "directly enabled vital contributions to the project" |
| **IBM (Open Horizon)** | Founding institutional sponsor; IBM listed among project partners/adopters |
| **Others** | The project maintains an adopters list (see repo); several managed-service and distro vendors (e.g., OpenShift ecosystem charts, third-party managed OpenBao offerings) ship it — individual names not exhaustively verified here |

### 7.3 The Vault community

- HashiCorp (now IBM) runs a large, mature community: HashiCorp Discuss forums, a huge plugin ecosystem, official Terraform provider, Vault Secrets Operator, cloud-agnostic documentation, certifications (Vault Associate/Professional), and a dominant mindshare among security engineers.
- Enterprise adoption spans most of the Fortune 500; HCP Vault adds a managed-SaaS path. Post-IBM acquisition, Vault is positioned inside IBM's hybrid-cloud security portfolio (alongside Red Hat), and HashiCorp's BSL posture is unchanged as of writing.

### 7.4 The ecosystem

| Ecosystem layer | Vault | OpenBao |
|---|---|---|
| Kubernetes operators | Vault Secrets Operator (official), External Secrets Operator (provider) | External Secrets Operator (OpenBao provider), community operators; the OpenBao helm chart |
| IaC | Terraform provider (official) | Terraform provider works against OpenBao API; OpenBao-specific bits in community providers |
| Helm | `hashicorp/vault` chart (vault-helm) | `openbao/openbao` chart (openbao-helm) |
| Client SDKs | Official (Go, Python, Java, Ruby, .NET, Node) | OpenBao SDK (Go) + Vault-compatible clients via API compatibility |
| Managed services | HCP Vault | Third-party managed OpenBao offerings (8+ vendors claim support, per ecosystem listings — flag: unverified detail) |

### 7.5 Community comparison (approximate — verify live)

| Dimension | Vault | OpenBao |
|---|---|---|
| GitHub stars (repo) | ~30k+ (one of the most-starred security tools) | Low thousands (a fraction of Vault's) — approximate |
| Contributors | Large, mostly HashiCorp/IBM employees | Dozens–hundreds, community + corporate backers |
| Release cadence | Quarterly-ish + patch lines (2026: 2.0.x; 1.19–1.21 patched) | 5–6 minor releases/year (2.6.1 July 2026) |
| Mindshare | Dominant | Growing, especially in OSS-first orgs and Europe (EU-funded ApeiroRA) |
| Risk perception | Vendor-backed, commercially supported | Governance-backed (LF/OpenSSF), community-supported |

*The honest framing:* Vault still dwarfs OpenBao in raw adoption and ecosystem breadth; OpenBao's significance is structural — it proved a Vault-class platform can live under open governance with an OSI license, and it is winning real institutional adopters (GitLab, SAP). Momentum is real but numbers are an order of magnitude apart; treat any specific star/contributor counts as approximate and check the live repositories.

---

## 8. Selection Guidance

### 8.1 The decision factors

1. **Enterprise features** — do you need DR/performance replication, control groups, Sentinel-style policy-as-code, enterprise MFA? Vault Enterprise is the only place those exist (§4.4). If your needs are namespaces, HSM, and read scaling — OpenBao now has open-source equivalents.
2. **Licensing** — BSL 1.1 (source-available, no redistribution-as-a-service, converts to Apache 2.0 after 4 years) vs MPL 2.0 (OSI-approved, redistribution and embedding permitted under copyleft-at-file-level). For vendors, managed-service providers, and OSS-first shops, MPL 2.0 is usually decisive.
3. **Governance** — single-vendor roadmap (HashiCorp/IBM; responsive but proprietary) vs Linux Foundation + OpenSSF open governance (community-controlled roadmap, but no single throat to choke).
4. **Stability & maturity** — Vault: 10+ years, battle-tested, huge install base, commercial SLA available. OpenBao: fork from the mature 1.14 codebase, so it inherited stability; its own 2.x innovations are younger and less battle-hardened.
5. **Ecosystem & integrations** — Vault wins breadth today (operators, SDKs, plugins, certifications, HCP). OpenBao leverages API compatibility to inherit most of it, but the Vault-specific operator and certification paths are Vault-only.
6. **Support** — Vault Enterprise/HCP: vendor SLA. OpenBao: community + a growing set of third-party commercial support vendors (verify availability in your region — this matters for regulated banks).

### 8.2 Selection table

| Factor | HashiCorp Vault | OpenBao |
|---|---|---|
| **Best for** | Enterprises wanting commercial support, enterprise features, managed SaaS (HCP) | OSS-first orgs, regulated orgs needing MPL licensing, vendor-agnostic governance |
| **DR/performance replication** | ✅ (Enterprise) | ❌ (roadmap) |
| **HSM seal** | ✅ (Enterprise) | ✅ (2.5.0+) |
| **Namespaces** | ✅ (Enterprise) | ✅ (2.x; per-ns sealing 2.6.0) |
| **Read scaling** | ✅ (Enterprise perf standbys) | ✅ (2.5.0 read replicas) |
| **Control groups / Sentinel** | ✅ (Enterprise) | ❌ |
| **License** | BSL 1.1 (source-available) | MPL 2.0 (OSI open source) |
| **Governance** | HashiCorp (IBM) | Linux Foundation / OpenSSF |
| **Maturity** | 10+ years; de-facto standard | Fork of mature codebase + 2.5 years of independent evolution |
| **Vendor SLA** | ✅ | ⚠️ third-party only |
| **Ecosystem** | Largest | Compatible-with-Vault + growing |
| **Cost** | OSS free; Enterprise/HCP priced (six-figure class) | Free, MPL 2.0; support from third parties |

### 8.3 Recommendation

- **Choose Vault when:** you need DR replication, control groups/Sentinel, enterprise MFA, a vendor SLA, HCP managed service, or HashiCorp ecosystem lock-in (Vault Secrets Operator, certifications); or when your compliance office demands a commercially supported, single-vendor product. For regulated financial firms this is still the conservative default.
- **Choose OpenBao when:** you run (or want to run) Vault-OSS-class features under an OSI-approved license; you want open governance (roadmap immunity to vendor strategy); you are a vendor/service provider that cannot accept BSL; you want namespaces/HSM/read-scaling without Enterprise pricing; or you are migrating off BSL-era Vault and accept the §4.4 gaps. GitLab and SAP's choices show it is credible at serious scale.
- **The 2026 nuance:** the old "OpenBao = Vault OSS minus HSM/namespaces" mental model is obsolete (2.5/2.6 closed those gaps). The decision now hinges on **replication, control groups/Sentinel, vendor SLA, and licensing/governance preference** — a much narrower and more strategic set of trade-offs than it was in 2024.

### 8.4 Banking context (Crédit Agricole CIB lens)

For a bank, secrets management is a **risk-and-compliance platform**, not a developer convenience:

- **HSM custody** — key material for payments, signing, and encryption often must live in FIPS-validated HSMs under dual control; the seal path must terminate in hardware. Vault Enterprise's PKCS#11 integration is the proven route; OpenBao 2.5.0's PKCS#11 seal is now viable but *younger* — vet it against your bank's HSM vendor (Thales Luna, Utimaco, Gemalto/Entrust) and change-management requirements. See [financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md) for the broader compliance-systems landscape (PCI-DSS, MAS TRM, SOX, audit trails).
- **DR/BCP** — banks run active/active or DR sites; **Vault Enterprise DR replication is the only first-class answer today** (OpenBao's raft snapshot/restore is a recovery procedure, not replication — RTO/RPO maths will usually favour Vault Enterprise in the banking DR plan).
- **Audit & e-discovery** — both produce audit logs; Vault Enterprise adds seal-wrap and control-group governance that map neatly onto four-eyes and maker-checker controls.
- **Vendor-risk / procurement** — a BSL-licensed component or a community-governed one both raise procurement questions; banks typically prefer a vendor with an SLA (Vault Enterprise/HCP) unless the architecture explicitly demands MPL 2.0.
- **Bottom line for a bank:** Vault Enterprise remains the low-risk default for core banking secrets; OpenBao is a credible candidate for *non-core* estates (CI/CD secrets, dev/test environments, GitLab runners, cloud-native workloads) where cost and open licensing matter — a "two-tier secrets strategy" is a legitimate architecture (§9.3).

---

## 9. Worked Example — Kubernetes Deployment and a Bank's Migration

### 9.1 Deploying on Kubernetes: vault-helm vs openbao-helm

Both projects ship official Helm charts with near-parallel value structures:

| Aspect | vault-helm (HashiCorp) | openbao-helm (openbao/openbao) |
|---|---|---|
| Repo | `hashicorp/vault` (chart: `vault`) | `openbao/openbao` (chart: `openbao`) |
| Modes | `server.standalone.enabled`, `server.ha.enabled` | `server.standalone.enabled`, `server.ha.enabled` |
| Storage | `server.dataStorage` + raft; Consul option | Same pattern (`server.dataStorage`, raft) |
| HA/unseal | `server.ha.raft.enabled: true`; unseal via k8s jobs/init containers | Same (`server.ha.raft.enabled`); OpenBao init/unseal containers |
| Ingress/service | Standard values | Standard values (port 8200, same conventions) |
| Injector/agent | Vault Agent Injector (official) | OpenBao does **not** ship the Agent Injector — use the sidecar pattern manually or rely on the API/operator instead |
| Operator | Vault Secrets Operator (official, Vault-specific) | Use **External Secrets Operator** with the OpenBao provider (works for both platforms) |

**Typical values comparison (HA + raft):**

```yaml
# vault-helm                          # openbao-helm
server:                               server:
  ha:                                   ha:
    enabled: true                         enabled: true
    raft:                                 raft:
      enabled: true                         enabled: true
      config: |                             config: |
        storage "raft" { ... }                storage "raft" { ... }
        listener "tcp" { ... }                listener "tcp" { ... }
  ingress:                              ingress:
    enabled: true                         enabled: true
    host: vault.example.com               host: bao.example.com
```

The chart-level difference is cosmetic (binary + env-var prefixes); the real differences are the **injector** (Vault-only) and the **operator** (Vault Secrets Operator is Vault-specific; ESO covers both). GitOps readers: see [kargo_gitops_guide.md](kargo_gitops_guide.md) for how the promotion of such infrastructure changes can be staged.

### 9.2 A Vault-to-OpenBao migration sketch (mid-size bank, CI/CD estate)

**Context:** a bank's platform team runs Vault OSS 1.14.x for CI/CD secrets (GitLab runners, Jenkins, dev/test DB credentials), with AppRole + Kubernetes auth, KV v2, database engine, and file/syslog audit. Core banking secrets stay on Vault Enterprise (not in scope).

1. **Audit:** confirm no Enterprise features in use; list external plugins (e.g., a custom engine) — plan to rebuild against the OpenBao SDK.
2. **Clone & rehearse:** stand up a staging OpenBao cluster from a raft snapshot; run the full verification checklist (kv get/set, dynamic DB creds, k8s auth login, OIDC flow, audit entries).
3. **Comms & window:** schedule a maintenance window; snapshot raft (`vault operator raft snapshot save`); store snapshot + unseal keys per bank change-management policy (CAB approval, dual control).
4. **Swap:** stop Vault; replace binaries with `bao` on each node (same raft path); port config; set `BAO_ADDR`/`BAO_TOKEN` in pipelines (leave `VAULT_*` fallbacks during transition).
5. **Verify:** unseal with existing shares; confirm leader election; smoke-test each engine; spot-check audit log continuity.
6. **Cut over:** repoint GitLab runners/Jenkins; watch for plugin and env-var stragglers; keep the rollback snapshot for 30 days.
7. **Retrospective:** document RTO/RPO, lessons for the core-banking estate, and the case for/against extending OpenBao to other non-core estates.

**Key risks:** (a) external-plugin ABI mismatch; (b) `VAULT_*` env vars lurking in 200 pipelines; (c) feature regression if anyone quietly used an Enterprise flag; (d) team skills drift from the Vault tooling they know; (e) **no DR replication** — the DR story for this estate changes from "replicated" to "snapshot-restore" until OpenBao ships replication.

### 9.3 The decision for a mid-size bank

| Factor | Weight | Vault Enterprise | OpenBao (2.6.x) |
|---|---|---|---|
| HSM seal (Thales/Utimaco) | High | ✅ proven | ✅ new (2.5.0) — pilot first |
| DR replication (RTO/RPO) | High | ✅ | ❌ — decisive for core |
| Namespaces (BU isolation) | Medium | ✅ | ✅ |
| Compliance (audit, four-eyes) | High | ✅ seal-wrap/control groups | ⚠️ audit yes; control groups no |
| License/procurement | Medium | BSL + commercial | MPL 2.0 |
| Vendor SLA | High | ✅ | ⚠️ third-party |
| Cost | Medium | High | Free |
| Ecosystem (operator/SDK) | Medium | ✅ | ✅ via ESO/API compat |

**The choice:** a **two-tier strategy**. Core banking secrets, payments keys, and DR-mandated workloads → **Vault Enterprise** (replication + control groups + HSM + SLA are non-negotiable). CI/CD, dev/test, cloud-native app secrets → **OpenBao** (MPL 2.0, no Enterprise licence cost, namespaces + HSM + read scaling available, ESO integration). Revisit annually: the gap that forces Vault Enterprise on the core tier is now essentially *replication and control groups* — if OpenBao ships either, the calculus shifts again.

---

## 10. The Future (2026+)

### 10.1 OpenBao roadmap — recent releases

| Release | Date | Headline |
|---|---|---|
| 2.0.0 | Apr 2024 | Independent versioning; paginated list APIs |
| 2.5.0 | Feb 4, 2026 | **Horizontal read scalability**; **HSM (PKCS#11) seal** (`bao-hsm` binaries) |
| 2.6.0 | Jul 14, 2026 | **Per-namespace sealing** (tenant key-material partitioning); auto-unseal plugins |
| 2.6.1 | Jul 22, 2026 | Current release (patch) |

Direction of travel (from the project's published roadmap): read scaling, HSM, namespaces, plugins and auto-unseal breadth, PKI/transit depth, OpenSSF security hardening — and, long-requested, **replication** (the last big enterprise gap).

### 10.2 Vault roadmap — HashiCorp under IBM

- Vault moved to a **2.0 series in 2026** (CE 2.0.4 current mid-2026; Enterprise 2.0.3; 1.19–1.21 lines still receiving patches). Exact 2.0.0 launch date not pinned in this guide — verify against the changelog.
- Strategic direction under IBM: HCP managed offerings, integration with IBM/Red Hat stack, secrets scanning (HCP Vault Radar), enterprise compliance features, and continued BSL licensing (no verified news of relicensing as of August 2026).

### 10.3 Divergence

The two platforms are now **independently versioned and independently innovating**:

- OpenBao ships features Vault still doesn't have OSS-side (paginated lists; per-namespace sealing with separate key material; plugin-based auto-unseal).
- Vault 2.x evolves under IBM's roadmap; Enterprise-only capabilities (replication, control groups, Sentinel, enterprise MFA) remain the proprietary moat.
- CLI, env-var prefixes, and version numbers will continue to drift (`vault` 2.0.x vs `bao` 2.6.x). API compatibility remains a stated OpenBao commitment, but every release makes "identical" less automatic.

### 10.4 Convergence?

- **License:** the most-discussed convergence vector — HashiCorp returning to OSI licensing — has **no verified basis** as of August 2026; BSL persists under IBM. Don't plan on it.
- **Merging codebases:** OpenBao re-importing Vault 2.x code would face license (BSL) and architectural friction; the fork has already rewritten/added enough that a merge is unrealistic. Convergence via *standards* is more plausible: both platforms plug into neutral layers — External Secrets Operator, SPIFFE/SPIRE-style workload identity, OIDC, cloud KMS auto-unseal.
- **The stable end-state:** two platforms, one API family. Vault: commercial, enterprise-layer, IBM-backed. OpenBao: open governance, MPL 2.0, closing the enterprise gap feature by feature.

### 10.5 Trends summary

1. **The fork survived and thrived** — OpenBao is a living project with institutional adopters (GitLab, SAP), not a zombie snapshot.
2. **The enterprise moat is shrinking** — HSM, namespaces, and read scaling are now OSS-available; replication is the remaining frontier.
3. **Consolidation irony** — IBM owns Vault and helped birth OpenBao; expect IBM to keep both viable (Vault commercial, OpenBao as open-source hedge and community signal).
4. **Selection is now strategic, not feature-based** — the 2026 decision is about replication, control groups, SLA, and governance/licensing philosophy.
5. **Ecosystem convergence via neutral layers** (ESO, OIDC, KMS) makes both platforms substitutable at the integration boundary — which lowers migration risk in both directions.

---

## 11. Glossary

| Term | Definition |
|---|---|
| **Secrets** | Sensitive data whose exposure grants access/identity: API keys, passwords, certificates, tokens, encryption keys |
| **Secrets management** | The discipline of centrally storing, rotating, access-controlling, and auditing secrets |
| **Vault** | HashiCorp's (now IBM's) secrets-management and data-protection platform; the de-facto industry standard; launched April 2015 |
| **HashiCorp** | The company behind Vault, Terraform, Consul; acquired by IBM (closed 27 Feb 2025, $6.4B) |
| **OpenBao** | Open-source, Linux-Foundation-governed fork of Vault (announced Dec 2023); MPL 2.0; current 2.6.x |
| **BSL / BUSL** | Business Source License 1.1 — source-available (not OSI-open-source) license; HashiCorp moved to it in Aug 2023; converts to Apache 2.0 after 4 years |
| **MPL** | Mozilla Public License 2.0 — OSI-approved; file-level copyleft; Vault's license before BSL, and OpenBao's license today |
| **Fork** | A project derived from another project's source code that develops independently |
| **Linux Foundation** | Non-profit hosting neutral open-source governance; OpenBao's home (initially via LF Edge; also OpenSSF) |
| **CNCF** | Cloud Native Computing Foundation — **not** OpenBao's home (common misconception; OpenBao is Linux Foundation/OpenSSF) |
| **OpenSSF** | Open Source Security Foundation — LF's security foundation; OpenBao is an accepted sandbox project |
| **Secrets engine** | A pluggable backend in Vault/OpenBao that produces or stores secrets (KV, database, PKI, transit, AWS, SSH…) |
| **KV** | Key/value secrets engine; v1 (unversioned) and v2 (versioned, CAS, soft-delete) |
| **Database engine** | Creates short-lived, per-lease database credentials on demand (dynamic secrets) |
| **PKI** | Engine issuing short-lived X.509 certificates with CRLs and auto-renewal |
| **Transit** | Encryption-as-a-service engine: encrypt/decrypt/sign via API without exposing keys |
| **Dynamic secrets** | Credentials generated on demand with leases, revoked when the lease expires |
| **Lease** | Time-bound grant of a secret/token with renewal and revocation semantics |
| **Rotation** | Replacing a secret's value on a schedule or on demand to limit leaked-credential exposure |
| **Auth method** | How a client authenticates (token, AppRole, Kubernetes, LDAP, OIDC…) |
| **Token** | The base credential issued by any auth method; carries policies and TTL |
| **AppRole** | Machine-to-machine auth (role ID + secret ID) for CI/CD and services |
| **Kubernetes auth** | Workload identity via service-account JWTs |
| **LDAP** | Directory-based human auth (Active Directory) |
| **OIDC** | OpenID Connect SSO (Okta, Entra ID, Keycloak) |
| **HSM** | Hardware Security Module — tamper-resistant hardware for key custody; PKCS#11 is the standard interface (Vault Enterprise; OpenBao since 2.5.0) |
| **Replication** | Copying data between clusters for read scaling (performance) or DR failover — Vault Enterprise only; OpenBao's open gap |
| **Namespaces** | Tenant isolation within one cluster (isolated mounts/policies/audit) — Enterprise in Vault; community-implemented in OpenBao (2.x) |
| **Performance standby** | Vault Enterprise read replicas for scaling and upgrade availability |
| **HCP** | HashiCorp Cloud Platform — managed Vault (and other HashiCorp services) as SaaS |
| **CLI** | Command-line interface: `vault` (Vault) vs `bao` (OpenBao); command syntax is otherwise identical |
| **bao** | The OpenBao CLI binary (Mandarin: "to hold dear / protect") |
| **vault** | The Vault CLI binary (and the product's name) |
| **VAULT_ADDR** | Vault environment variable for server address (e.g., `https://vault:8200`) |
| **BAO_ADDR** | OpenBao's equivalent; `BAO_*` vars take precedence, `VAULT_*` honoured as fallback |
| **Storage backend** | Where the encrypted data lives: raft, Consul, file, in-memory |
| **Raft** | Integrated consensus-based storage (the default for HA clusters) |
| **Consul** | HashiCorp's service-discovery/consensus product; a Vault storage backend |
| **Migration** | Moving from Vault to OpenBao (documented in-place path) or vice versa (recovery-style) |
| **Drop-in** | Interchangeable without code changes — OpenBao's design goal vs Vault OSS (API/CLI/config compatible; plugins must be rebuilt) |
| **Helm chart** | Kubernetes package: `vault-helm` (HashiCorp) vs `openbao-helm` (OpenBao org) |
| **vault-helm** | Official HashiCorp Helm chart for Vault |
| **openbao-helm** | OpenBao project's Helm chart for OpenBao (chart name `openbao`) |

---

## 12. Fact-Check Notes and Sources

**Verified against primary/secondary sources (Aug 2026):**

- ✅ Vault launched April 2015 (v0.1.0); MPL 2.0 until the Aug 10, 2023 HashiCorp announcement moving all products to BSL 1.1 (HashiCorp license FAQ; Discuss announcement).
- ✅ OpenBao: forked from Vault CE 1.14.x (commit between 1.14.8/1.14.9 — last MPL line); working group from Nov 9, 2023; public announcement Dec 2023; initiated with IBM/Open Horizon (LF Edge) involvement (TechTarget; project FAQ).
- ✅ OpenBao license MPL 2.0; governed under Linux Foundation; accepted as OpenSSF sandbox project (OpenSSF blog).
- ✅ OpenBao versions: 2.0.0 (Apr 2024) → 2.5.0 (Feb 4, 2026: HSM PKCS#11 seal + horizontal read scalability) → 2.6.0/2.6.1 (Jul 2026: per-namespace sealing, auto-unseal plugins) — from GitHub release metadata (tag dates, `bao-hsm` artefacts, release notes).
- ✅ HashiCorp Vault in 2026: 2.0.x CE series (2.0.4 on the install page; CVE-2026-12624 fixed in CE/ENT 2.0.3 and 1.19.19/1.20.13/1.21.8) — exact 2.0.0 launch date not pinned.
- ✅ IBM completed the HashiCorp acquisition on 27 Feb 2025 ($6.4B, $35/share).
- ✅ openbao-helm exists in the openbao org; official OpenBao Helm chart.
- ✅ Adopters: GitLab (FOSDEM 2025), SAP ApeiroRA (EU-funded; per openbao org statement), IBM/Open Horizon.

**Flagged as unverified / approximate (do not cite as fact without re-checking):**

- ⚠️ CNCF status: OpenBao is **not** a CNCF project; one secondary source's "CNCF participant" claim appears to conflate Linux Foundation/OpenSSF with CNCF.
- ⚠️ MFA parity in OpenBao, seal-wrap internals: not verified in this pass.
- ⚠️ Specific star counts / contributor numbers and "8+ vendors offer OpenBao support": approximate, check live.
- ⚠️ Exact OpenBao release that first introduced namespaces: present in the 2.x series (demonstrated in 2026 sources; per-namespace sealing in 2.6.0), exact minor not pinned.
- ⚠️ `BAO_*`/`VAULT_*` env-var fallback precedence: as documented by OpenBao; confirm per deployed version.
- ⚠️ Vault 2.0.0 exact release date: not pinned.

**Key sources:** openbao.org (docs: migration guide, PKCS#11 seal, helm, blog), github.com/openbao/openbao (releases, org statement), openssf.org (OpenBao sandbox announcement), hashicorp.com license FAQ, discuss.hashicorp.com (Aug 2023 announcement), IBM newsroom (Feb 27, 2025 acquisition close), developer.hashicorp.com (Vault install/versions), TechTarget & FOSDEM 2025 material (OpenBao history and GitLab adoption).

*End of guide. Companion pieces in this repository: [cloud_providers_guide.md](cloud_providers_guide.md) (managed alternatives: AWS Secrets Manager, Azure Key Vault, GCP Secret Manager), [financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md) (HSM/compliance for banking), [kargo_gitops_guide.md](kargo_gitops_guide.md) (GitOps delivery of infrastructure changes), [penetration_testing_execution_standard_guide.md](penetration_testing_execution_standard_guide.md) (finding leaked secrets in practice).*
