# PCI DSS Compliance in a Kubernetes Environment

**The Cluster-Scope Deep-Dive - the Overview, the Identity and the Decoder (Including the Two False Friends); the Parts of PCI DSS That an Estate of Containers Actually Stresses (Verified at the Council: v4.0.1, 11 June 2024, and the Numbers the Council Itself Cites); Why Kubernetes Breaks the Cardholder Data Environment (Scope Is a Drawing, Placement Is an Event); Segmentation Inside a Cluster (NetworkPolicy and Its CNI Dependency, Node Pools and Taints, Separate Clusters, Service Mesh); the Control Plane and the Data Plane (etcd, the API Server, Encryption at Rest, Audit, RBAC); Pod-Level Controls (Pod Security Standards and Pod Security Admission, and What They Do Not Buy); the Secrets Problem (Base64, etcd, Service-Account Tokens and Rotation); the Supply Chain (Provenance, Signing, Admission); Logging and the Audit Trail; the Assessment (the Questions, the Artefacts and What Each Artefact Does Not Prove); the Banking and Regulated Angle; the Failure Modes; the Cymbal Bank Worked Example; the Anti-Patterns; the Claims Audit; and What Could Not Be Verified - the Final Word: the standard does not care that you run Kubernetes, it cares where the data can go, and a scheduler decides that at runtime**

> **Author:** Jack Liu Shurui, Solution Architect
> **Purpose:** Research reference for **PCI DSS compliance in a Kubernetes environment** - the question of how a cardholder data environment (CDE) that is declared as a scope on a diagram can be shown to still be bounded at runtime, when the thing that decides where a workload lands is a scheduler making a fresh decision at admission, at every restart, at every eviction and at every node drain. The guide covers the standard's verified version and status; the requirement families a container estate stresses; the structural reasons a cluster resists a static scope; the controls that actually constitute a boundary (and, for each, what it does not prove); the control plane as the real perimeter; pod-level hardening and its asymmetry; secrets and service-account tokens; image provenance and admission; the audit log and what it can and cannot establish; the assessment questions and the artefacts that answer them; the regulated-institution frame; the failure modes and the anti-patterns; a worked example at Cymbal Bank; the claims audit; and the honest ledger of what this pass could not verify.
> **Repo:** https://github.com/jackliusr/research
> **Scope note:** this guide **owns plain-Kubernetes PCI DSS scope** - the CDE bounded on a cluster that is not OpenShift, the runtime-versus-scope argument, the segmentation options and what each proves, the control-plane perimeter, the pod-level and secret-level controls as they bear on cardholder data, the evidence an assessment asks for, and the failure modes specific to a scheduler-driven estate. It deliberately does **not** re-derive the release gate, the OpenShift security-context machinery, the non-human identity discipline or the supply-chain inventory; those are owned by named siblings and cross-referenced by section throughout.
> **Primary Sources:** **pcisecuritystandards.org** ✅ read this pass - the PCI DSS standard page (the purpose statement, the intended-audience statement, the QSA and ASV definitions, and the Council's line that whether an entity must validate compliance is at the discretion of a payment brand or acquirer); the Council document library ✅ (PCI DSS v4.0.1 listed as the current standard, with the Summary of Changes v4.0 to v4.0.1); the Council's machine-readable document index ✅ (`docs-pub.pcisecuritystandards.org/doc_library.json`, where v4.0.1 carries a `last_updated` stamp of 2024-06-11 and v4.0 is marked archived); the Council blog post "Just Published: PCI DSS v4.0.1" ✅ (11 June 2024; a limited revision with no additional or deleted requirements; v4.0 retired 31 December 2024; the 31 March 2025 effective date unchanged); the Council blog post on the future-dated requirements ✅ (v3.2.1 retired 31 March 2024; Requirement 11.3.2 named as the ASV scan requirement). **kubernetes.io** ✅ read this pass, each page with its last-modified date recorded in sec. 15 - releases, network policies, pod security standards, pod security admission, the PSP migration task, secrets, encryption at rest, auditing, service accounts, validating admission policy, taints and tolerations. **docs.sigstore.dev** ✅ (cosign verification, keyless identity-based verification, attestation verification). **cisecurity.org** ✅ (the CIS Kubernetes Benchmark listing, version 2.1.0, with the benchmark text itself behind a download form and therefore not read).
> **Last Updated:** October 2026
> **Companion guides (sibling, same folder - plain filenames):** [InfoSec Release Readiness](infosec_release_readiness_guide.md) (**sec. 4.6 owns PCI DSS as one standard among many inside a release gate, and that section asserts no requirement number at all - the difference is stated in sec. 1.5 and is the cleanest statement of the boundary between the two guides**), [Security-by-Design](security_by_design_guide.md) (**sec. 6 owns DevSecOps, CI/CD gates, SBOM and SLSA - this guide consumes its output as evidence for sec. 8 rather than re-deriving it**), [Non-Human Identity](non_human_identity_guide.md) (**owns the identity discipline for workloads, tokens, rotation and credential lifetime - sec. 7.6 cross-references it rather than restating it**), [SBOM Generation for C, Pro\*C and COBOL](sbom_c_proc_cobol.md) (**owns component inventory for the procedural estate; sec. 8.1 uses its output as the composition half of provenance**), [Secure Red Hat OpenShift](secure_red_hat_openshift_guide.md), [OpenShift SCC Comprehensive](openshift_scc_comprehensive_guide.md), [OpenShift SCC and Service Accounts](openshift_scc_service_account_guide.md), [OpenShift Workload Availability](openshift_workload_availability_guide.md) and [OpenShift Random UID and Username Mapping](openshift_random_uid_username_mapping.md) (**these five own Security Context Constraints, random UIDs, and OpenShift's own defaults - sec. 6.5 says which statements in this guide are upstream Kubernetes and which are not, and refuses to conflate the two**)
> **Companion guides (banking/, prefix `../banking/`):** [Operational Resilience Framework](../banking/operational_resilience_framework_guide.md) (**owns the important-business-service and impact-tolerance machinery, and the restore question a supervisor asks - sec. 11.2 cross-references it**), [MAS Regulations and Guidelines](../banking/mas_regulations_guidelines_guide.md) (**owns the verified MAS instrument map; sec. 11 does not re-derive it**)
> **Companion guides (management/, prefix `../management/`):** [Vendor Management](../management/vendor_management_guide.md) (**owns third-party assessment and exit; sec. 11.3 leans on it for the cluster-as-supplied-service problem**)

---

**How to use this guide:** Section 1 is the overview - the one-line answer, the thesis, the decoder, the two false friends that make searching this repository for either acronym produce nonsense, and the boundary declared by name against the near-neighbour guides. Section 2 states precisely what is verified about the standard at the Council and which requirement numbers may therefore be asserted. Section 3 is the central argument: why a scope drawn statically and a scheduler acting dynamically are in tension, and where the evidence complicates that argument. Section 4 takes the segmentation options one at a time and, for each, states what it does not prove. Section 5 is the control plane, which is where the real perimeter sits. Sections 6 to 9 are the workload-level controls, secrets, supply chain and logging. Section 10 is the assessment - the questions, the artefacts and the limits of each artefact. Section 11 is the regulated-institution frame. Sections 12 and 14 separate what breaks from what people convince themselves is fine. Section 13 is the worked example. Section 15 is the claims audit. Section 16 is the residual gaps, the glossary, the cross-references and the close. **Integrity convention:** ✅ = verified this pass against the issuing body's own publication, with the page and its last-modified date recorded in sec. 15; ⚠ = flagged and unverified, never to be relied on without a re-check; ❌ = rejected, meaning the claim was tested and does not hold.

---

**Table of Contents**

1. [The Overview, the Identity and the Decoder](#1-the-overview-the-identity-and-the-decoder)
   - 1.1 [The Short Answer and the Thesis in One Line](#11-the-short-answer-and-the-thesis-in-one-line)
   - 1.2 [What Compliance in a Cluster Is and Is Not](#12-what-compliance-in-a-cluster-is-and-is-not)
   - 1.3 [The Decoder](#13-the-decoder)
   - 1.4 [The Two False Friends](#14-the-two-false-friends)
   - 1.5 [The Boundary Declared](#15-the-boundary-declared)
   - 1.6 [The Overview Table](#16-the-overview-table)
2. [PCI DSS, the Parts That Matter Here](#2-pci-dss-the-parts-that-matter-here)
   - 2.1 [What Is Verified at the Council](#21-what-is-verified-at-the-council)
   - 2.2 [The Four Families the Council Names](#22-the-four-families-the-council-names)
   - 2.3 [The Twelve Families and the Licence Gate](#23-the-twelve-families-and-the-licence-gate)
   - 2.4 [The Requirement Table](#24-the-requirement-table)
   - 2.5 [What the Standard Asks of an Estate](#25-what-the-standard-asks-of-an-estate)
3. [Why Kubernetes Breaks the CDE](#3-why-kubernetes-breaks-the-cde)
   - 3.1 [Scope Is a Drawing and Placement Is an Event](#31-scope-is-a-drawing-and-placement-is-an-event)
   - 3.2 [The Namespace Is a Label](#32-the-namespace-is-a-label)
   - 3.3 [The Kernel and the Network Are Shared](#33-the-kernel-and-the-network-are-shared)
   - 3.4 [The Pod Is Not a Trust Boundary](#34-the-pod-is-not-a-trust-boundary)
   - 3.5 [Where the Evidence Complicates the Thesis](#35-where-the-evidence-complicates-the-thesis)
4. [Segmentation Inside a Cluster](#4-segmentation-inside-a-cluster)
   - 4.1 [What a Boundary Requires](#41-what-a-boundary-requires)
   - 4.2 [NetworkPolicy and Its CNI Dependency](#42-networkpolicy-and-its-cni-dependency)
   - 4.3 [Node Pools, Taints and Tolerations](#43-node-pools-taints-and-tolerations)
   - 4.4 [Separate Clusters](#44-separate-clusters)
   - 4.5 [Service Mesh](#45-service-mesh)
   - 4.6 [The Segmentation Table](#46-the-segmentation-table)
5. [The Control Plane and the Data Plane](#5-the-control-plane-and-the-data-plane)
   - 5.1 [etcd the Store Behind Everything](#51-etcd-the-store-behind-everything)
   - 5.2 [The API Server Is the Real Boundary](#52-the-api-server-is-the-real-boundary)
   - 5.3 [Encryption at Rest](#53-encryption-at-rest)
   - 5.4 [Audit of the Control Plane](#54-audit-of-the-control-plane)
   - 5.5 [RBAC](#55-rbac)
   - 5.6 [The CIS Kubernetes Benchmark](#56-the-cis-kubernetes-benchmark)
   - 5.7 [Why a Compromised Control Plane Ends the Argument](#57-why-a-compromised-control-plane-ends-the-argument)
6. [Pod-Level Controls](#6-pod-level-controls)
   - 6.1 [The Three Profiles](#61-the-three-profiles)
   - 6.2 [Pod Security Admission](#62-pod-security-admission)
   - 6.3 [The Enforce Asymmetry](#63-the-enforce-asymmetry)
   - 6.4 [The Restricted Controls in Detail](#64-the-restricted-controls-in-detail)
   - 6.5 [Upstream Versus OpenShift](#65-upstream-versus-openshift)
   - 6.6 [What PSA Does Not Do](#66-what-psa-does-not-do)
7. [The Secrets Problem](#7-the-secrets-problem)
   - 7.1 [Base64 Is Not Encryption](#71-base64-is-not-encryption)
   - 7.2 [Encryption at Rest and Its Limits](#72-encryption-at-rest-and-its-limits)
   - 7.3 [External Secret Stores](#73-external-secret-stores)
   - 7.4 [The Service Account Token](#74-the-service-account-token)
   - 7.5 [The Rotation Question](#75-the-rotation-question)
   - 7.6 [The Non-Human Identity Cross-Reference](#76-the-non-human-identity-cross-reference)
8. [The Supply Chain](#8-the-supply-chain)
   - 8.1 [Provenance and the Build Claim](#81-provenance-and-the-build-claim)
   - 8.2 [Signing and Verification](#82-signing-and-verification)
   - 8.3 [Admission Control](#83-admission-control)
   - 8.4 [The Unsigned Image Finding](#84-the-unsigned-image-finding)
   - 8.5 [Tags, Digests and Registries](#85-tags-digests-and-registries)
9. [Logging and the Audit Trail](#9-logging-and-the-audit-trail)
   - 9.1 [The API Audit Log](#91-the-api-audit-log)
   - 9.2 [Stages and Levels](#92-stages-and-levels)
   - 9.3 [Backends and the Omitted Flag](#93-backends-and-the-omitted-flag)
   - 9.4 [Container, Node and CNI Logs](#94-container-node-and-cni-logs)
   - 9.5 [What an Audit Log Proves](#95-what-an-audit-log-proves)
   - 9.6 [Toward Requirement 10](#96-toward-requirement-10)
10. [The Assessment](#10-the-assessment)
    - 10.1 [What the Assessment Is Asked to Believe](#101-what-the-assessment-is-asked-to-believe)
    - 10.2 [The Questions](#102-the-questions)
    - 10.3 [The Artefact Table](#103-the-artefact-table)
    - 10.4 [A Design Document Is Not Evidence](#104-a-design-document-is-not-evidence)
    - 10.5 [Scope Versus Runtime](#105-scope-versus-runtime)
11. [The Banking and Regulated Angle](#11-the-banking-and-regulated-angle)
    - 11.1 [Two Regimes and One Cluster](#111-two-regimes-and-one-cluster)
    - 11.2 [Operational Resilience](#112-operational-resilience)
    - 11.3 [Third-Party Risk](#113-third-party-risk)
    - 11.4 [Change Governance](#114-change-governance)
    - 11.5 [What a Supervisor Asks](#115-what-a-supervisor-asks)
    - 11.6 [The Council Line on Applicability](#116-the-council-line-on-applicability)
12. [The Failure Modes](#12-the-failure-modes)
    - 12.1 [How to Read This Section](#121-how-to-read-this-section)
    - 12.2 [The Table](#122-the-table)
    - 12.3 [The Common Thread](#123-the-common-thread)
13. [The Cymbal Bank Worked Example](#13-the-cymbal-bank-worked-example)
    - 13.1 [The Estate as Designed](#131-the-estate-as-designed)
    - 13.2 [The Decision the Runtime Forced](#132-the-decision-the-runtime-forced)
    - 13.3 [What It Cost](#133-what-it-cost)
    - 13.4 [The Reconciliation Burden](#134-the-reconciliation-burden)
    - 13.5 [The Thesis Restated](#135-the-thesis-restated)
14. [The Anti-Patterns](#14-the-anti-patterns)
    - 14.1 [Failure Modes and Anti-Patterns](#141-failure-modes-and-anti-patterns)
    - 14.2 [The Table](#142-the-table)
    - 14.3 [Why They Survive](#143-why-they-survive)
15. [The Claims Audit](#15-the-claims-audit)
    - 15.1 [The Method](#151-the-method)
    - 15.2 [The Claims Table](#152-the-claims-table)
    - 15.3 [Flagged](#153-flagged)
    - 15.4 [Rejected](#154-rejected)
    - 15.5 [The Honest Ledger](#155-the-honest-ledger)
16. [What Could Not Be Verified, the Glossary, the Cross-References and the Closing Summary](#16-what-could-not-be-verified-the-glossary-the-cross-references-and-the-closing-summary)
    - 16.1 [What Could Not Be Verified and the Residual Gaps](#161-what-could-not-be-verified-and-the-residual-gaps)
    - 16.2 [The Glossary](#162-the-glossary)
    - 16.3 [Cross-References](#163-cross-references)
    - 16.4 [The Closing Summary](#164-the-closing-summary)

---

## 1. The Overview, the Identity and the Decoder

### 1.1 The Short Answer and the Thesis in One Line

**The short answer.** PCI DSS compliance in a Kubernetes cluster is not a property of the cluster. It is a claim that a defined set of data - cardholder data and sensitive authentication data - is confined to a defined set of systems and people, and that the confinement holds continuously, not merely on the day the architecture was approved. A cluster can host a cardholder data environment (CDE) perfectly well. What a cluster cannot do on its own is *keep* the CDE bounded, because the component that decides where a workload runs is not the diagram. It is the scheduler, and the scheduler makes a new decision on every pod creation, every restart, every eviction, every reschedule and every node drain.

**The thesis, in one line.** PCI DSS does not care that you are running Kubernetes; it cares where the cardholder data can go - and a scheduler decides that at runtime.

**The difficulty in a paragraph.** An assessment argument is built on a scope: a diagram, a data-flow description, an inventory of systems in scope, and a set of controls around them. That argument is static. A cluster is a runtime decision engine: a pod is placed by a scheduling decision, on a node that may also host workloads from outside the scope, sharing one kernel, one container runtime and a network fabric whose segmentation is delegated to a plugin. The two artefacts - the diagram and the running system - agree only if something enforces the agreement, continuously, and produces evidence that it did. Every section of this guide is an attempt to answer the same question, which is the question an assessor is really asking: *how do you demonstrate that the CDE is bounded, and bounded now rather than at design time?*

### 1.2 What Compliance in a Cluster Is and Is Not

**It is a boundary argument, not a product decision.** There is no Kubernetes setting that makes cardholder data compliant. There is only a set of mechanisms - admission, network policy, node isolation, identity, encryption, logging - which, taken together and evidenced, support a claim that data cannot leave a defined envelope. The envelope is the deliverable. The mechanisms are how the envelope is maintained.

**It is not a namespace.** The most common error in this domain is to treat a namespace as the CDE. A namespace is an API scoping construct: it names a set of objects and gives RBAC and policy a place to attach. It does not isolate the kernel, it does not isolate the network unless a CNI plugin enforces policy, and anyone who can create a pod in that namespace can read any Secret in that namespace - a property the project's own Secrets documentation states plainly (sec. 7.1). A namespace is a label on the diagram. It becomes a boundary only when several independent mechanisms are aligned behind it, and sec. 3.5 and sec. 4.6 show why the boundary then belongs to those mechanisms rather than to the namespace.

**It is not a property you can inherit from a managed service.** A managed control plane moves operational responsibility but not the scope argument. The data still lands on nodes, the nodes still run a kernel, and the network policy is still enforced - or not - by whichever plugin the provider ships. Who owns the control is a vendor-management question (sec. 11.3), and it is a different question from whether the control exists.

**It is not a one-time gate.** A cluster that was in scope on the day of the assessment and out of scope three upgrades later is a scope failure that no document catches. The evidence that matters is dated, continuous and machine-generated (sec. 10.3), because the thing being evidenced - placement - happens thousands of times a day.

**It is not achieved by the OpenShift defaults.** This deserves saying because most container material in this repository is OpenShift-shaped: Security Context Constraints, random UID assignment and the platform's own restricted policies are OpenShift behaviours, owned by the five OpenShift guides named in the front matter, and they are not restated here. sec. 6.5 draws the line explicitly, because a reader who carries an OpenShift mental model into a plain-Kubernetes cluster will believe controls exist that do not.

### 1.3 The Decoder

| Term | What it means here | Why it matters to the argument |
| --- | --- | --- |
| **PCI DSS** | The card industry's baseline of technical and operational requirements for protecting payment account data; current version v4.0.1, published 11 June 2024 (sec. 2.1) | It is a requirement set about data and the systems that touch it, not about any particular runtime |
| **The CDE** | The cardholder data environment: the people, processes and technology that store, process or transmit cardholder data or sensitive authentication data, plus those that could impact its security | The scope that must be bounded; in a cluster it is a claim about placement, not a namespace name |
| **CHD** | Cardholder data - the account data the standard exists to protect | The thing whose location the scheduler decides |
| **SAD** | Sensitive authentication data - the authentication elements associated with a card | The most tightly constrained data; if it transits a cluster it dominates the scope argument |
| **The cluster** | A set of nodes managed by one control plane, sharing an API server and a datastore | Shares a kernel and a network; the sharing is what makes per-namespace isolation a claim rather than a fact |
| **The namespace** | An API scoping and naming boundary inside a cluster | A label, not a security boundary, unless several mechanisms are aligned behind it |
| **The pod** | The smallest deployable unit; one or more containers sharing a network namespace and, often, a service-account identity | Not a trust boundary; a sidecar in the same pod is inside the same blast radius |
| **The service account** | A namespaced non-human identity that a pod authenticates as | Whoever can create a pod can assume that identity and read that namespace's Secrets |
| **The node** | The machine that runs pods, with one kernel and one kubelet | The real isolation unit in most clusters; node pools are where a boundary can actually be drawn |
| **Scope** | The declared set of systems, data flows and people subject to the standard | Static by nature, and therefore always at risk of describing last quarter's cluster |
| **Segmentation** | The mechanisms that keep in-scope and out-of-scope systems apart, and reduce the scope of an assessment | In a cluster, delegated to CNI, admission, node pools and cluster separation - each with a different proof |
| **The QSA** | A Qualified Security Assessor: an independent organisation qualified and trained by the Council to perform PCI DSS assessments | Asks for evidence of the boundary; this guide describes the questions and the artefacts, and asserts nothing about what an assessor would accept |

### 1.4 The Two False Friends

Both acronyms in this guide's title are unreliable search terms, and the unreliability is measured rather than hypothetical. A reader who greps this repository for either one will get a result set that is mostly about something else, and will conclude - wrongly - that the estate already has a body of work on card-data scope in containers.

**False friend one: PCI is not always PCI DSS.** Measured across the repository's markdown files, 84 files match the string `PCI`, but only 29 of them contain `PCI DSS`. Thirteen are the PCIe bus - the interconnect in `A100 80GB PCIe`, `H100 PCIe` and `A100 80GB SXM` - because GPU hardware descriptions use the same three letters. A narrower join, `PCI` appearing in the same file as `kubernetes` or `k8s`, returns four files, and all four are incidental: a compliance-benchmark table row in a mitigation guide, a scanner's policy-pack list in the audit-as-code guide, a passing mention in the OpenShift security guide, and a GPU provider comparison where the match is the string `A100 80GB PCIe`. Counts of this kind are command-dependent, which is itself part of the point: a case-sensitive substring search returns 84 files, a case-insensitive one 86, and a word-boundary search for the three letters alone returns 67 - because the boundary excludes the PCIe bus, so the search that looks more precise is the one that hides the false friend. Every count in this paragraph excludes this guide. The finding does not move: the join produces no PCI DSS Kubernetes material.

**False friend two: CDE is not always the cardholder data environment.** Six files in this repository match the standalone acronym, and the word is used for at least three different things - a university's College of Design and Engineering (formed in 2022), a data-platform compliance environment, and the cardholder data environment. Nothing in the filename or the surrounding prose reliably tells you which. A grep for `CDE` therefore produces a set that is smaller than `PCI` and even less trustworthy, because the false positives look like real hits: they sit in architecture and data-governance documents where a compliance environment is exactly what the sentence is discussing. Again the count depends on the pattern: seven files match once the spelled-out phrase is admitted, both figures excluding this guide. The direction of the finding does not move at all.

**Why this is in the guide and not in a footnote.** Because the practical consequence is that keyword search cannot establish whether a control exists. An estate that "has PCI DSS documentation" may have a benchmark table and a GPU spec sheet. The only way to know is to read the artefacts - which is also the answer to the assessment question in sec. 10.4.

### 1.5 The Boundary Declared

**Against [InfoSec Release Readiness](infosec_release_readiness_guide.md).** That guide owns the release gate: the evidence set, the decision, the sign-off, risk acceptance, and - at its sec. 4.6 - PCI DSS as one standard among many inside a release gate, sitting beside SSDF, ASVS, SAMM, SLSA, SPDX and the EU Cyber Resilience Act. Its sec. 4.6 asserts **no PCI DSS requirement number, requirement title or sub-requirement at all**, and its own claims audit records why: the standard's text could not be retrieved at the Council's endpoints in that pass. This guide is the first in the repository to assert requirement numbers, and it does so only for the numbers the Council itself cites in its own publications: 3, 6, 8 and 12 as the families its revision note describes, and 11.3.2, 6.4.3 and 11.6.1 as requirement numbers the Council names in its own posts and document titles (sec. 2.1). That difference is the boundary: the release-readiness guide asks *may this ship*; this guide asks *is the data still inside the line*. Neither is a substitute for the other, and a payment-scope release should be gated by both.

**Against the OpenShift guides.** [Secure Red Hat OpenShift](secure_red_hat_openshift_guide.md), [OpenShift SCC Comprehensive](openshift_scc_comprehensive_guide.md), [OpenShift SCC and Service Accounts](openshift_scc_service_account_guide.md), [OpenShift Workload Availability](openshift_workload_availability_guide.md) and [OpenShift Random UID and Username Mapping](openshift_random_uid_username_mapping.md) own Security Context Constraints, the restricted-v2 and nonroot-v2 policy families, random UID assignment, the namespace annotation model for UID ranges, and OpenShift's own workload-availability defaults. **None of that is restated here.** Where a control in this guide has an OpenShift-specific analogue, sec. 6.5 says so in one sentence and points at the owning guide; the reader should assume that every statement about Pod Security Standards, Pod Security Admission and securityContext fields in this guide is about upstream Kubernetes, and that the OpenShift equivalent is somewhere else.

**Against the plain-Kubernetes gap.** It is worth naming that this guide fills a hole rather than extending a shelf: the repository's container material is OpenShift-shaped throughout, and no plain-Kubernetes guide existed before this one. That is why sec. 6.5 spends a subsection on the distinction and why the failure modes in sec. 12 are all plain-Kubernetes failure modes - a cluster with no SCC layer has no platform default to fall back on, and the defaults it does have are permissive.

### 1.6 The Overview Table

| Aspect | What this guide says |
| --- | --- |
| The unit of compliance | The boundary around the data, not the cluster, the namespace or the manifest |
| The central tension | Scope is declared statically; placement is decided at runtime, repeatedly |
| The strongest in-cluster boundary | A separate cluster, then a dedicated node pool with taints, then a CNI-enforced network policy domain - in that order of strength |
| The weakest control commonly mistaken for a boundary | A namespace |
| The control plane | The actual perimeter: whoever can write pods can place cardholder data anywhere |
| Pod hardening | Necessary, unevenly enforced, and namespace-scoped, so it inherits the namespace's weakness as a boundary |
| Secrets | Base64 in the API object, unencrypted in etcd by default, readable by anyone who can create a pod in the namespace |
| Supply chain | An unsigned image in a card-data node pool is an assessment finding, not a hygiene issue |
| Logging | The audit log proves an identity made a request at a time; it does not prove the pod was in scope |
| The assessment | Artefacts that are dated, machine-generated and continuous; a design document is not evidence |
| The regulated frame | The bank must be able to restore the service and name the control owner, not only pass an assessment |
| The honest limit | Several facts behind this argument sit behind a licence gate or an unread page, and sec. 15 says which |

---

## 2. PCI DSS, the Parts That Matter Here

### 2.1 What Is Verified at the Council

The version discipline comes first, because every other statement in this section depends on it. **PCI DSS v4.0.1 is the current version** ✅. The Council's document library lists it as a featured document alongside a *Summary of Changes v4.0 to v4.0.1*, and links the standard PDF. The Council's own machine-readable document index gives v4.0.1 a `last_updated` stamp of 2024-06-11 and marks v4.0 archived; a `last_updated` stamp is not a printed publication date field, and this guide does not present it as one - the publication date is confirmed by the Council's own announcement instead.

**The revision is limited, and the limit is the point.** The Council's announcement, posted **11 June 2024**, states: "To address stakeholder feedback and questions received since PCI DSS v4.0 was published in March 2022, the PCI Security Standards Council (PCI SSC) has published a limited revision to the standard, PCI DSS v4.0.1. It includes corrections to formatting and typographical errors and clarifies the focus and intent of some of the requirements and guidance. There are no additional or deleted requirements in this revision." ✅ A guide written against v4.0 therefore does not become wrong on the day v4.0.1 lands; what changes is clarity, not obligation.

**Two dates follow, both verified.** "PCI DSS v4.0 will be retired on 31 December 2024. After that point, PCI DSS v4.0.1 will be the only active version of the standard supported by PCI SSC." ✅ And, on the future-dated requirements: "Does PCI DSS v4.0.1 change the 31 March 2025 effective date for the new requirements? No. This limited revision does not impact the effective date of these new requirements." ✅ The predecessor is gone too: the Council's later post records that "As of 31 March 2024, the PCI Security Standards Council (PCI SSC) officially retired PCI DSS v3.2.1." ✅ So as of this pass there is exactly one active version - v4.0.1 - and the future-dated set has been effective since 31 March 2025.

**One number to treat carefully.** The same later post quotes an assessor, not the Council, saying "Of the 64 new requirements, 51 are future-dated and will be effective as of 31 March 2025." That is an interviewee's characterisation in a Council blog post, and this guide attributes it to the interviewee rather than to the standard or to the Council ✅⚠. It is useful context for why a 2022-era container design may not address the current obligation set; it is not a citable count.

**What the standard is for, in the Council's words.** PCI DSS "was developed to encourage and enhance payment card account data security and facilitate the broad adoption of consistent data security measures globally", and "provides a baseline of technical and operational requirements designed to protect payment account data" ✅. The intended audience is "Entities that store, process, or transmit cardholder data (CHD) and/or sensitive authentication data (SAD) or could impact the security of the cardholder data environment (CDE). This includes all entities involved in payment card processing - including merchants, processors, acquirers, issuers, and service providers." ✅ That sentence is the whole scope argument in one line: the audience is defined by data and by *impact on the environment that holds it*, not by technology. A cluster that can impact the CDE is in scope whether or not it holds card data.

### 2.2 The Four Families the Council Names

The Council's revision note describes changes by requirement family, which gives four verified anchors - not because the other families do not matter, but because these are the four the Council itself names ✅. They also happen to be the four that a container estate stresses hardest.

**Requirement 3 - stored account data.** The v4.0.1 changes: "clarified Applicability Notes for issuers and companies that support issuing services; added a Customized Approach Objective and clarified applicability for organizations using keyed cryptographic hashes to render Primary Account Numbers (PAN) unreadable." ✅ In a cluster, this family lands on where the data rests: a persistent volume, a database pod, a snapshot, a backup, an etcd entry that was never meant to hold card data at all but does because someone put it in a ConfigMap.

**Requirement 6 - secure development and patching.** The v4.0.1 changes: "reverted to PCI DSS v3.2.1 language that installing patches/updates within 30 days applies only for 'critical vulnerabilities'; added Applicability Notes clarifying how the requirement for managing payment page scripts applies." ✅ Two things follow for a container estate. First, patching is a claim about the *image* and about the *base layer*, which means the rebuild cadence is the control and the running pod is only its evidence. Second, payment page scripts are in the requirement set at all, which puts the web tier - the pod that serves the payment page - inside the argument even when it holds no card data.

**Requirement 8 - identity and authentication.** The v4.0.1 change: "added an Applicability Note that multi-factor authentication for all (non-administrative) access into the CDE does not apply to user accounts authenticated only with phishing-resistant authentication factors." ✅ The container version of this family is uncomfortable: most identities inside a cluster are non-human, they authenticate with bearer tokens rather than factors, and "access into the CDE" is precisely what a service account token grants. sec. 7.4 and sec. 7.6 are this family's real content.

**Requirement 12 - policy and third parties.** The v4.0.1 change: "updated Applicability Notes on relationships between customers and third-party service providers (TPSPs)." ✅ For a cluster this is the managed-platform and CNI-vendor question: the network policy is enforced by somebody's plugin, and the scope claim depends on a supplier's behaviour (sec. 11.3).

**Three specific numbers the Council names, and nothing more.** Requirement **11.3.2** is identified in the Council's own post as the ASV vulnerability-scan requirement ("completing PCI DSS Requirement 11.3.2 for the first time") ✅. Requirements **6.4.3** and **11.6.1** are identified by the Council's own supporting-document title, *Guidance for PCI DSS Requirements 6.4.3 and 11.6.1*, whose subject is payment page scripts and whose index entry carries a 2025-04-22 stamp ✅. Those three numbers, plus 3, 6, 8 and 12, are the entire set this guide asserts - and the reason for that discipline is next.

### 2.3 The Twelve Families and the Licence Gate

The standard is organised into twelve requirement families, numbered 1 to 12. This guide states them in the order they are universally published: 1 network security controls, 2 secure configuration, 3 stored account data, 4 cryptography in transit, 5 malware, 6 secure development, 7 access by business need, 8 identity and authentication, 9 physical, 10 logging and monitoring, 11 security testing, 12 policy and governance. ⚠ The wording of those twelve titles is the standard's published family subject matter as it is generally circulated, but **the standard text itself was not read at the issuing body in this pass**, so the titles are flagged rather than cited, and the numbers the guide relies on for specific claims are only those the Council names in its own publications (sec. 2.1).

**Why not read it.** Every PDF under the Council's document-hosting domain returns an HTTP 403 with an HTML interstitial titled "Official PCI Security Standards Council Site - Document". The interstitial is a click-through licence - a "Read and Copy License" - that requires a session established against a separate session endpoint plus agreement cookies set through a token-protected call. This pass did not forge that acceptance. The consequence is stated here rather than buried: a requirement number or sub-requirement sentence that has not been read at the issuing body is exactly the kind of claim that turns a research guide into a liability, so this guide asserts the numbers the Council itself cites and flags the rest.

**What a practitioner should do instead.** Take the requirement numbers, titles, applicability notes and testing procedures straight from the current standard PDF through the Council's own document library, pin them to v4.0.1, and record the retrieval date on the control. A control written against an unversioned "PCI DSS" drifts the moment the Council revises the standard - and the Council documents its own transitions, which is how the version discipline in sec. 2.1 is possible at all.

### 2.4 The Requirement Table

The rows below are the families a container estate stresses. The family numbers are the standard's structure; the middle two columns are this guide's argument about containers, not the standard's text. Requirement-family titles are ⚠ per sec. 2.3; the numbers and the four Council-named families are ✅.

| Requirement family | What it asks of an estate | What a cluster makes hard | What evidence answers it |
| --- | --- | --- | --- |
| 1 - network security controls ⚠ | That the CDE is separated from untrusted networks and that traffic into it is restricted to what is needed | Segmentation is delegated to a CNI plugin; a policy object with no implementing controller is inert and looks identical to one that works (sec. 4.2) | Enforced-policy configuration read from the CNI itself, plus denied-connection evidence from the plugin's own logs, dated |
| 2 - secure configuration ⚠ | Hardened systems, no vendor defaults, no unnecessary services or accounts | Node images drift; the control plane's flags are the hardening; defaults are permissive (no audit policy file means no audit records, sec. 9.3) | Rendered kubelet and API server configuration as deployed, benchmark output with the benchmark version named (sec. 5.6), image build records |
| 3 - stored account data ✅ (Council-named) | That stored account data is minimised, protected at rest and not retained beyond need | Card data appears in places nobody classified: ConfigMaps, PersistentVolumes, backup buckets, crash dumps, ephemeral scratch | Data-flow inventory cross-checked against actual volume and object contents; at-rest encryption configuration (sec. 5.3) |
| 4 - cryptography in transit ⚠ | Strong cryptography for card data crossing open networks | Pod-to-pod traffic is plaintext by default; a service mesh or mTLS is an add-on, not a default | Mesh or mTLS configuration with issuer and rotation parameters; TLS inspection records |
| 6 - secure development ✅ (Council-named) | Secure software, patched within the stated window, payment page scripts controlled | Patching is a rebuild-and-redeploy claim about images, and the running container cannot be patched in place | Image digest inventory with build dates against the vulnerability window; admission evidence that only rebuilt images deploy (sec. 8.3) |
| 7 - access by business need ⚠ | Access limited to what the role requires | RBAC is additive and cluster-scoped bindings are easy to grant; the default service account exists in every namespace | Effective-permission review per identity, including indirect paths - create-pod implies read-secret (sec. 5.5) |
| 8 - identity and authentication ✅ (Council-named) | Unique identities, strong authentication, no shared or default credentials | Workload identities are bearer tokens; long-lived tokens in Secrets do not expire; the default account is auto-assigned | Token lifetime and audience configuration, audit records of token issuance and use, inventory of Secret-based tokens (sec. 7.4) |
| 10 - logging and monitoring ⚠ | That access to systems and data is logged, protected and reviewed | The API audit log is off unless a policy file is supplied; container logs are ephemeral; node and CNI logs live outside the cluster's evidence | Audit policy as loaded, log pipeline integrity, retention and review records, with the gap documented (sec. 9.5) |
| 11 - security testing ⚠ | Testing of the environment and of segmentation, plus external scanning (11.3.2) ✅ | Segmentation testing must probe the runtime boundary, not the manifest; scan scope must include node images and registries | Segmentation test results against actual pod placement; scan configuration and results naming the assessed scope |
| 12 - policy and third parties ✅ (Council-named) | Documented policy, assigned responsibility, managed service providers | Responsibility for network policy, node hardening and the control plane is split across platform, application and provider teams | A responsibility matrix mapped to named controls, and provider assurance for the parts the bank does not operate (sec. 11.3) |

### 2.5 What the Standard Asks of an Estate

Read across the table, three properties of the standard matter more in a cluster than anywhere else, and none of them is a technology choice.

**It asks about systems that *could impact* the CDE, not only systems that hold data.** The Council's audience statement is explicit about this ✅. In a cluster, almost everything can impact the CDE: the CNI plugin that carries the traffic, the admission controller that decides what runs, the registry that supplies images, the log pipeline that receives audit records, the node that hosts a card-data pod alongside a build agent. Scope therefore does not stop at the namespace boundary, which is the first reason sec. 3 exists.

**It asks for continuity, not configuration.** Every family in the table is a claim about a state that must persist: patched *within* a window, logged *continuously*, separated *at all times*. A cluster re-establishes its state constantly - pods are replaced, nodes are recycled, manifests are re-applied - so a control that was true at deploy time is a control that has to be re-proven on a schedule. This is why the evidence table in sec. 10.3 is organised around what an artefact does *not* prove rather than around what it shows.

**It asks who is responsible, and a cluster distributes responsibility unusually widely.** The platform team owns the control plane, the application team owns the pod spec, the security function owns the policy, the network team owns the CNI, and a provider may own all of it. The standard's policy family is, in a container estate, mostly a question about whether those five parties can point at the same boundary. That question is sec. 11's subject, and it is the one a supervisor asks in a different vocabulary.

## 3. Why Kubernetes Breaks the CDE

### 3.1 Scope Is a Drawing and Placement Is an Event

A scope is a statement about where data is. In a conventional estate, that statement is durable: a database server sits in a rack, a subnet has an address range, a firewall rule names both, and the diagram remains true until somebody changes it deliberately. In a cluster the same statement is a *prediction* about a scheduling decision, and the scheduler is not consulting the diagram.

**The sequence, concretely.** A workload object is applied. A controller creates a pod. The scheduler evaluates the pod's requests, constraints, affinities, taints and the current state of every node, and binds the pod to one node. The kubelet on that node pulls the image and starts the containers. None of those steps reads a document that says which nodes are in the cardholder data environment. The only inputs that carry scope intent are the ones somebody encoded into the objects - a node selector, a toleration, an affinity rule, a namespace-level admission policy - and each of them is a *request* that something else must enforce.

**And the decision is made again, and again.** A pod is rescheduled when its node fails. It is evicted when the node comes under memory or disk pressure. It is replaced on every rolling update, which in a well-run cluster means every deployment, every config change and every image rebuild. It is drained and recreated on every node upgrade - and the project maintains release branches for the most recent three minor releases, 1.37, 1.36 and 1.35 ✅, with 1.37.1 released on 2026-09-15 ✅ - so a cluster on the current line is being upgraded on a cadence measured in months, not years. Each of those events is a fresh placement, and the scope diagram does not move with it. A CDE that was "the three nodes with the taint" is, after an upgrade, "the three nodes that currently carry the taint", which is a different statement and needs different evidence.

**The bypass is documented, which makes it worse rather than better.** The taints documentation is explicit that the scheduler honours taints but that a pod can skip the scheduler entirely: "if you manually specify the `.spec.nodeName` for a Pod, that action bypasses the scheduler; the Pod is then bound onto the node where you assigned it, even if there are `NoSchedule` taints on that node that you selected." ✅ A pod with a fixed nodeName lands wherever it is told. The taint is not a wall; it is a filter in a component that can be bypassed by a field in the pod spec - and anyone who can create a pod can set that field.

**What this means for the thesis.** The standard's question is "where can the cardholder data go". In a cluster, the honest answer is "wherever a pod that handles it can be scheduled, plus wherever its volumes, backups, logs and secrets can be read from". That is a runtime set, not a drawn one, and it changes without anyone editing the drawing. Everything else in this guide is about making that set small and making its membership provable.

### 3.2 The Namespace Is a Label

A namespace does three real things: it scopes object names, it is the attachment point for role bindings, and it is the attachment point for policy that is expressed per namespace - Pod Security Admission labels being the clearest example. None of those three is a security boundary in the sense the standard means.

**It does not confine credentials.** The Secrets documentation states the position without hedging: "Anyone with API access can retrieve or modify a Secret, and so can anyone with access to etcd. Additionally, anyone who is authorized to create a Pod in a namespace can use that access to read any Secret in that namespace; this includes indirect access such as the ability to create a Deployment." ✅ Read that again as a scope statement. The permission that application teams need in order to deploy - create a pod, or create a Deployment that creates pods - is the same permission that reads every credential in the namespace. A namespace holding card-data credentials is therefore only as isolated as the RBAC on pod creation inside it, and pod creation is the one permission a delivery pipeline cannot do without.

**It does not confine policy, because policy is attached to it by a label.** Pod Security Admission is configured with namespace labels, and the project's own migration guidance states the consequence: "anyone who can update (or patch or create) a namespace can also modify the Pod Security level for that namespace, which could be used to bypass a more restrictive policy." ✅ The control that hardens the workloads in a namespace is protected by a permission on the namespace object itself. If a team can create its own namespaces - a normal self-service arrangement - it can create one with a permissive level and deploy there.

**It does not confine the kernel or the network.** Two pods in different namespaces on the same node share a kernel, a container runtime and a kubelet, and by default they can reach each other over the pod network. Namespace separation changes none of that. Only a CNI that enforces policy changes the network, and no namespace label changes the kernel.

**What a namespace is good for.** It is an excellent *organising* construct: it gives RBAC a granularity, it gives quota a scope, it gives policy a place to attach, and it makes the inventory legible. The mistake is not using namespaces; it is treating the organising construct as the security construct, and then writing a scope document that says "the CDE is the `payments` namespace".

### 3.3 The Kernel and the Network Are Shared

**One kernel per node.** Containers are processes with restricted views, not virtual machines. Isolation comes from namespaces, cgroups, capabilities, seccomp and Mandatory Access Control, and every one of those is a mitigation around a shared kernel rather than a replacement for it. A kernel vulnerability reachable from a container is reachable from every container on that node, and the blast radius is the node - which is why the node, not the namespace, is the unit at which a boundary can be drawn. This is also why the pod-level controls in sec. 6 matter: they reduce the attack surface that reaches the shared kernel, and they do not remove the sharing.

**Privileged means the node.** The Secrets documentation says it in one sentence: "Any containers that run with `privileged: true` on a node can access all Secrets used on that node." ✅ So a single privileged pod - a debug container, a CNI agent, a node-level agent installed by a vendor - is a credential-access path to every secret on the host, across every namespace. In a card-data node pool, that is the whole boundary, in one line of YAML.

**A flat network by default.** Pod-to-pod traffic is unrestricted unless a network policy selects the pod and a plugin enforces it. The NetworkPolicy documentation states the default from the pod's side: "By default, a pod is non-isolated for egress; all outbound connections are allowed" ✅, and the ingress default is stated the same way. The cluster therefore starts from allow-all, and every restriction is an object somebody had to write, in a plugin that has to implement it (sec. 4.2). Compare this with a conventional DMZ, where the default is deny and the rules are the exception.

**The sharing is not a defect to be fixed.** It is the economic basis of the platform: density, bin-packing, fast scheduling, cheap failover. The point is not that clusters are insecure; it is that the properties that make them useful are the properties that make a static scope claim unreliable. A bank that wants both density and a bounded CDE has to buy the boundary explicitly - in node pools, in clusters, in admission, in the CNI - and then evidence it.

### 3.4 The Pod Is Not a Trust Boundary

A pod is a co-scheduling and co-networking unit. Containers in it share a network namespace, which means they reach each other over localhost, and they frequently share volumes, a service-account token and a lifecycle. That design is why sidecars work, and it is why a pod cannot be a boundary between trusted and untrusted code.

**The untrusted sidecar is the ordinary case, not the exotic one.** A service-mesh proxy injected into every pod is a container the application team did not write, running with its own configuration and its own network position. A log-forwarding sidecar has read access to whatever the application writes. An agent injected by a security vendor has whatever privileges its DaemonSet or its injection template grants. Any of these can be compromised, and if it is compromised while sitting in a pod that touches cardholder data, the compromise is inside the boundary. The standard's "could impact the security of the CDE" language covers this exactly: the sidecar does not store card data, and it can absolutely affect the security of the environment that does.

**The service-account token is shared by default.** Containers in a pod authenticate as the pod's service account unless the workload disables automounting, and the token is available to every container in the pod. A sidecar that does not need the API does not, by default, lack the ability to use it.

**What follows.** Pod-level hardening reduces what a compromised container can do (sec. 6). It does not create a boundary *within* a pod. If a workload's trust level differs from its sidecar's, the answer is a separate pod with a separate identity, or a separate node pool, not a securityContext.

### 3.5 Where the Evidence Complicates the Thesis

The thesis is that the scheduler, not the diagram, decides where card data goes. The evidence complicates it in one important direction, and the honest version of the argument has to say so.

**A namespace can be made into a meaningful boundary - by making it something else.** Consider a namespace that is (a) admitted only onto a dedicated, tainted node pool, (b) a network-policy domain whose CNI actually enforces deny-by-default, and (c) an admission domain where a policy rejects pods that lack the required node selector and toleration, and rejects privileged pods outright. In that arrangement, a pod in the namespace really cannot be scheduled outside the node pool, really cannot talk to pods outside its policy domain, and really cannot run with host access. The namespace is now a boundary in practice.

**But the boundary is the node pool plus the CNI plus the admission policy, not the namespace.** Say it that way, because the difference has consequences. The namespace did not acquire isolating properties; three other mechanisms were aligned behind it, and each of them can fail independently. If the admission policy is deleted, pods can be scheduled elsewhere while the namespace label stays. If the CNI is swapped during a platform upgrade for one that does not implement policy, the NetworkPolicy objects remain and do nothing. If a toleration is added to a pod spec by anyone with write access, the taint stops repelling. The namespace is the *name* of the boundary, not its mechanism - and an assessment that accepts the name without testing the mechanisms is testing the diagram.

**The second complication: cluster separation is a different kind of claim.** Two clusters are separated by more than a label - separate API servers, separate datastores, separate node pools, separate credential stores, usually separate network planes. That is a boundary of a different order, and sec. 4.4 treats it as such. But even then, the argument has holes: a shared registry, a shared log pipeline, a shared identity provider and a shared CI system all cross the line, and each of them is a path by which data or authority can move. The strong boundary is not automatically a complete one.

**The third complication: the control plane is in scope either way.** Whatever is decided about namespaces, node pools and clusters, the thing that writes pods and reads secrets is the API server, and it is shared by everything in the cluster. That is why the next section after segmentation is the control plane, and why sec. 5.7 concludes that a compromised control plane makes the CDE boundary a fiction regardless of how carefully the data plane was drawn.

---

## 4. Segmentation Inside a Cluster

### 4.1 What a Boundary Requires

Before comparing mechanisms, it is worth being precise about what would count as a boundary, because most of the disagreement in this domain is people using the word for different things.

**Three properties.** A boundary must be (1) enforced by something the workload cannot reconfigure - a mechanism outside the pod's control plane, not a convention inside it; (2) deny-by-default at the layer it operates on, so that the set of allowed paths is enumerable; and (3) observable, so that an assessment can see both the configuration and the traffic that was refused. A control that has properties (1) and (2) but not (3) is real but unevidenced. A control that has (2) and (3) but not (1) is a request.

**The four candidate mechanisms** in a Kubernetes cluster are network policy enforced by the CNI, node pools with taints and tolerations, separate clusters, and a service mesh. They operate at different layers - network, scheduling, infrastructure, application identity - and none of them substitutes for another. The rest of this section takes them one at a time and ends each with the sentence that matters most: what it does not prove.

### 4.2 NetworkPolicy and Its CNI Dependency

**What it does.** A NetworkPolicy selects pods and declares allowed ingress and egress. It is the closest thing in upstream Kubernetes to a firewall rule set inside the cluster, and it is the mechanism most often cited as the CDE's network boundary.

**What it depends on.** Everything. The documentation is unambiguous: "Your cluster must use a network plugin that supports NetworkPolicy enforcement." ✅ And, more sharply: "Network policies are implemented by the network plugin. To use network policies, you must be using a networking solution which supports NetworkPolicy. Creating a NetworkPolicy resource without a controller that implements it will have no effect." ✅ The API server will accept the object regardless. There is no admission check that asks the plugin whether it can enforce what was just written. A cluster whose CNI ignores policy will hold a complete, well-formed, entirely inert set of NetworkPolicy objects, and every dashboard that counts them will report the control as present.

**The semantics that trip people up.** A pod is non-isolated until a policy selects it, and the default is allow: "By default, a pod is non-isolated for egress; all outbound connections are allowed" ✅, with ingress stated the same way. Policies are additive - "The effects of those egress lists combine additively." ✅ - so two policies that select the same pod produce the union of what they allow, never an intersection. There is no deny rule in the API. If a policy is meant to restrict a workload, every other policy that selects it is a hole in that intention, and adding a policy can only ever widen access.

**The exception that is always allowed.** "traffic to and from the node where a Pod is running is always allowed, regardless of the IP address of the Pod or the node." ✅ Node-level traffic is outside the policy model. Anything that can reach a pod from its own node - a node-level agent, a host-network pod, a compromised kubelet, a debug session - is not stopped by any NetworkPolicy, and host-network pods are a category worth auditing separately for exactly this reason.

**And the project's own caveat.** The documentation describes isolation as "not absolute, rather it means 'some restrictions apply'" ✅. That is the right mental model: a NetworkPolicy produces a reduction in reachability, not a wall.

**What it does not prove.** That the plugin enforces it. That the policy set covers every pod in the namespace rather than the three that were top of mind. That traffic from the node itself is restricted. That DNS resolution was allowed deliberately rather than accidentally - a deny-all egress policy without an explicit DNS allowance breaks name resolution, and the fix people apply under pressure is a broad allowance. And it does not prove anything about the data path that bypasses the pod network: a volume mounted from a CSI driver, a backup agent reading a PersistentVolume, a log shipper writing to an endpoint over the node's network.

### 4.3 Node Pools, Taints and Tolerations

**What it does.** A node pool is a set of machines with the same labels, instance type and configuration. Taints make a node repel pods that do not tolerate them; tolerations are the pod-side claim that permits scheduling. The documentation's framing is exact: taints "allow a node to repel a set of pods" ✅. And the effects are graded: `NoSchedule` keeps new pods off but does not evict running ones, `NoExecute` evicts pods that do not tolerate it, `PreferNoSchedule` is a preference the scheduler tries to honour. ✅

**Why it is the strongest *scheduling* control.** A tainted node pool is the mechanism that most directly encodes "card data runs here and nowhere else". It is enforced by the scheduler, which is outside the workload, and it is visible in the node object. Combined with an admission policy that requires the toleration for pods in the CDE namespace, it produces a genuine coupling between the namespace and a set of machines.

**What it does not prove.** Two things, both fatal if unaddressed. First, a toleration is a pod-side field, so anyone who can write a pod spec can schedule onto the pool: the taint repels pods that do not *ask*, and asking is free. The boundary therefore depends on admission control restricting who may set the toleration - the taint alone is a filter, not a permission. Second, the documented bypass: a pod with an explicit `nodeName` skips the scheduler and binds to the node "even if there are `NoSchedule` taints on that node" ✅, so the boundary depends on admission control rejecting that field too. A tainted pool without admission is a pool that is probably isolated, which is not the same as isolated.

**A related trap.** Node autoscaling adds nodes to a pool. A new node inherits the pool's taint only if the pool's template says so. Where the taint is applied by a post-provision step, there is a window - and in a rapidly scaling pool, a recurring one - during which a node exists without the taint and without the label the admission policy keys on.

### 4.4 Separate Clusters

**What it does.** A separate cluster for the card-data path gives separate API servers, separate etcd, separate node pools, separate credential stores and, in most designs, separate network planes and separate cloud accounts or projects. It is the only option in this list that removes the shared control plane, and the shared control plane is the largest single hole in every in-cluster boundary (sec. 5.7).

**Why it is the strongest claim available.** The boundary is no longer a policy that could be misconfigured; it is an absence of shared infrastructure. A workload in the non-CDE cluster has no credential that reaches the CDE cluster's API server, cannot be scheduled onto its nodes, and cannot read its secrets unless somebody grants a cross-cluster path. Demonstrating it is also easier: the evidence is two inventories, two credential scopes, two network paths, and a firewall rule set between them.

**What it does not prove.** That nothing crosses. The shared services are the usual holes: one container registry serving both clusters, one log pipeline receiving both clusters' audit records, one CI system with deploy rights to both, one identity provider issuing tokens valid in both, one backup target. Each is a legitimate path and each is a scope question in its own right. A separate cluster also costs something specific - duplicated platform tooling, duplicated policy, an upgrade cadence multiplied by the number of clusters, and a developer experience where the payment path behaves differently from everything else (sec. 13.3). It is the right answer often enough that the cost is worth naming rather than discovering.

### 4.5 Service Mesh

**What it does.** A mesh injects a proxy into each pod's network path, gives every workload a cryptographic identity, and moves traffic policy to layer 7: which service may call which, over which method, with what authentication. Mutual TLS means the traffic between two workloads is encrypted and both ends are authenticated by certificate rather than by network position, which addresses a real gap - pod-to-pod traffic is plaintext by default (sec. 2.4, family 4).

**Why it helps the scope argument.** Identity-based policy is a finer instrument than IP-based policy, and it survives rescheduling: a workload's identity does not change when it moves to a different node, so a policy expressed in terms of identities is stable across exactly the events that break a diagram (sec. 3.1). That is a genuine answer to part of the thesis's problem.

**What it does not prove.** That the mesh is in enforcing mode - permissive mode is the normal migration state and it looks like a mesh while allowing everything. That the proxy is actually in the path - injection depends on namespace labels and admission, and a pod that opted out is unprotected. That the mesh isolates the kernel, because it does not: the proxy is a container in the same pod, sharing the same kernel and the same node. And that the mesh's control plane is itself in scope - it holds the identities and the policy, so it is a system that could impact the security of the CDE, which puts it inside the assessment.

### 4.6 The Segmentation Table

| Mechanism | What it does | What it does not prove |
| --- | --- | --- |
| NetworkPolicy | Selects pods and declares allowed ingress and egress; deny-by-default for selected pods | That the CNI implements policy at all; that every pod is selected; that node-level traffic is restricted; that policies were not unioned into a wider allowance than intended |
| Node pool with taints and tolerations | Repels pods from nodes unless the pod tolerates the taint; encodes "card data runs here" in the node object | That only permitted pods carry the toleration; that admission rejects a raw `nodeName`; that autoscaled nodes were tainted before use; nothing at all about the network |
| Separate cluster | Removes the shared control plane, datastore, node pool and usually the network plane | That registries, log pipelines, CI systems, identity providers and backup targets are not shared paths across the line |
| Service mesh with mTLS | Workload identity, encrypted pod-to-pod traffic, layer-7 authorisation that survives rescheduling | That the mesh is in strict mode; that the proxy is injected into every relevant pod; anything about the kernel; and it adds a control plane that is itself in scope |
| Pod Security Admission | Rejects pods that violate a profile, at the namespace level, at pod creation | That the workload object complied before the controller made the pod (sec. 6.3); that the namespace label was not changed by whoever can edit namespaces |
| Admission policy on placement | Rejects or mutates pods so that placement fields cannot be set freely | That the policy's CEL expression covers every field a pod can use to influence placement |

**Read the table as a set of questions, not a set of answers.** No single row is a boundary. The arrangement that comes closest in one cluster is rows one, two and six together, and even that arrangement leaves the control plane shared - which is why the next section is about the control plane and why sec. 5.7 is the shortest path to understanding what "bounded" can honestly mean here.

---

## 5. The Control Plane and the Data Plane

### 5.1 etcd the Store Behind Everything

**What it is.** The consistent key-value store that holds the cluster's entire state: every object, every spec, every status, every Secret, every role binding, every namespace label. If the API server is the door, etcd is the room everything is in.

**Why it dominates the scope argument.** Access to etcd is access to everything the cluster knows, including the credentials that the cluster's own controls depend on. Encryption at rest (sec. 5.3) exists precisely because of this, and its documentation is honest about the limits: at-rest encryption is "additional to any system-level encryption for the etcd cluster or for the filesystem(s) on hosts where you are running the kube-apiserver" ✅. It is a layer, not the answer. A backup of etcd taken before encryption was enabled is a plaintext copy of the cluster; a snapshot restored into a test environment carries the production CDE's secrets with it.

**The version requirement.** At-rest encryption requires the cluster's control plane to use etcd v3.x - "major version 3, any minor version" ✅. That is a hard precondition, and it is the kind of fact that is worth checking rather than assuming on an inherited cluster.

### 5.2 The API Server Is the Real Boundary

**The pipeline.** A request authenticates, is authorised, passes admission, and is persisted. Every control in this guide that actually holds is enforced somewhere in that pipeline or in a component downstream of it: RBAC at authorisation, Pod Security Admission and validating admission policies at admission, the scheduler after that. A control that lives only in a manifest convention is not in the pipeline at all.

**The two permissions that define the boundary.** Whoever can create pods can place workloads - and therefore cardholder data - on any node the scheduler will accept. Whoever can read Secrets can read the credentials in a namespace, and per the documentation, the first permission implies the second: "anyone who is authorized to create a Pod in a namespace can use that access to read any Secret in that namespace" ✅. A CDE boundary in a cluster is, in practice, a statement about who holds those two permissions and what admission does with the objects they create. Everything else is decoration.

**The third permission, less obvious.** Whoever can write the audit policy or restart the API server with different flags can change what is recorded. Whoever can write admission configuration can change what is enforced. Those are control-plane administration rights, and they belong to the same trust tier as cluster-admin for scope purposes - a point sec. 5.7 makes explicit.

### 5.3 Encryption at Rest

**The default is nothing.** "By default, the API server stores plain-text representations of resources into etcd, with no at-rest encryption." ✅ That sentence is the whole reason a Secret is not a protected object on a default cluster. The mechanism is opt-in: the API server accepts `--encryption-provider-config` pointing at an `EncryptionConfiguration`, and if the first provider in the list is `identity`, "the default `identity` provider does not provide any confidentiality protection" ✅ - a configuration file that exists and does nothing.

**The providers, and what the documentation says about them.** The page lists identity, `aescbc`, `aesgcm`, `secretbox`, `kms v1` and `kms v2` ✅. The project's own table is unusually candid: `aescbc` is marked "Weak" and "Not recommended due to CBC's vulnerability to padding oracle attacks"; `aesgcm` must be rotated every 200,000 writes and is "Not recommended for use except when an automated key rotation scheme is implemented"; `secretbox` "may not be considered acceptable in environments that require high levels of review"; `kms v1` is deprecated since v1.28; `kms v2` is available as stable from v1.29 ✅. So the choice is not "encryption on or off" - it is which provider, with which key custody, rotated how.

**Version facts worth pinning.** Encrypting a custom resource requires Kubernetes v1.26 or newer; wildcard matching in the configuration requires v1.27 or newer ✅. Both matter in a card-data cluster, because the objects that leak are as likely to be a custom resource written by an operator as a Secret.

**Where it stops.** The documentation says it directly: "Encrypting secret data with a locally managed key protects against an etcd compromise, but it fails to protect against a host compromise." ✅ The keys sit in the configuration file on the control-plane host. Under KMS envelope encryption the attacker needs "to compromise etcd **and** the third-party KMS provider" ✅ - which is a materially different claim, and the difference is a key-custody decision, not a Kubernetes feature.

**What it does not prove.** That existing objects were rewritten. Enabling encryption changes what is written *next*; every object already in etcd stays as it was until it is written again, and the documentation's own migration step is a read-and-replace of every Secret ✅. A cluster that enabled encryption last year and believes all its Secrets are encrypted has a belief, not an observation, unless the rewrite was done and can be shown. Nor does it prove the key is not in the same backup as the datastore it protects.

### 5.4 Audit of the Control Plane

The audit log is the control plane's own record of who asked for what (sec. 9 covers its mechanics in detail). Two points belong here, in the control-plane section, because they change what a boundary claim can rest on.

**It records authority, not data.** The audit log will show that an identity created a pod in the CDE namespace, or read a Secret, or bound a role. It will not show what the pod did afterwards, whether the Secret was the card-data credential, or whether the pod landed on an in-scope node. It answers "who changed the cluster" and not "where the data went".

**It is off by default.** "You can pass a file with the policy to kube-apiserver using the `--audit-policy-file` flag. If the flag is omitted, no events are logged." ✅ A cluster with no audit policy file produces no audit records, and nothing in the cluster announces the absence. That single sentence is the most useful thing in this guide for the failure-mode section, and it is the reason sec. 9.3 exists.

### 5.5 RBAC

**What it does.** Role-based access control authorises requests by identity against roles bound to subjects, at namespace or cluster scope. It is the mechanism that decides who holds the two permissions from sec. 5.2.

**The properties that matter for scope.** RBAC in Kubernetes is additive: there are no deny rules, so a subject's effective permission is the union of every binding that reaches it, and a single over-broad binding anywhere in the cluster can undo a carefully written set of narrow ones. Cluster-scoped bindings apply to namespaced resources everywhere. Built-in roles include cluster-admin, and the aggregated roles mean a new rule added to an aggregated ClusterRole propagates to every subject bound to it. These properties are widely documented and were not re-read at kubernetes.io in this pass ⚠.

**The specific trap for a CDE namespace.** The permission a delivery pipeline needs - create pods, or create Deployments - carries secret-read with it (sec. 5.2). So "least privilege" in a card-data namespace is not achievable by restricting the pipeline to its own namespace; it is achieved by ensuring the namespace contains no credential whose disclosure would matter, and by moving the sensitive credentials out of the namespace into an external store with its own authorisation (sec. 7.3). Where that is not possible, the pipeline identity is a CDE-scoped identity and should be treated as one.

**What it does not prove.** That the effective permission set is what the role definitions suggest. That is an empirical question, answerable per identity, not per manifest.

### 5.6 The CIS Kubernetes Benchmark

**What is verified.** The Center for Internet Security lists a CIS Kubernetes Benchmark at version **2.1.0** on its own benchmark page, alongside provider-specific benchmarks for the major managed Kubernetes services and for OpenShift ✅. That is a verified version fact and nothing more.

**What is not.** The benchmark's control text sits behind a download form, so this pass did not read any of its recommendations and cites no control numbers ⚠. A guide that quoted "CIS control 5.2.1" without having read it would be doing exactly what sec. 2.3 refuses to do with the standard.

**How it relates to scope.** The benchmark is a secure-configuration baseline: it tells you whether the control plane and the nodes are hardened. It does not tell you which node holds cardholder data, whether the CNI enforces policy, or whether the placement of a pod was inside the documented boundary. Hardening is necessary and orthogonal - a fully benchmarked cluster can still schedule card data onto a node that is not in the CDE, and a scrupulously bounded CDE can run on an unhardened control plane. The two arguments meet only in the evidence table (sec. 10.3), where both are artefacts.

### 5.7 Why a Compromised Control Plane Ends the Argument

**The argument, in five steps.** An attacker who holds write access to the API server can (1) create a pod anywhere the scheduler will accept, including in the CDE node pool, and read any Secret in the namespace it creates; (2) read every Secret in every namespace it can reach, which on a permissive RBAC set is all of them; (3) delete or rewrite the NetworkPolicy objects, and if the CNI is the enforcer, the network boundary goes with them; (4) rewrite the admission policies that were enforcing placement and privilege; and (5) change the audit configuration so that none of the above is recorded. Each step is a normal API operation on a normal object. None of them requires exploiting a vulnerability in Kubernetes.

**What that means for the CDE.** The boundary around cardholder data is only as strong as the control plane's authorisation model, because the control plane can rewrite every other control. This is not a reason to despair; it is a reason to place the control plane inside the scope rather than outside it - which is exactly what the Council's audience statement implies when it includes systems that "could impact the security of the cardholder data environment" ✅. The control plane of a cluster that hosts card data is such a system by definition.

**The practical consequences, which are design decisions rather than warnings.** Separate the cluster (sec. 4.4) so that the control plane of the non-CDE estate cannot reach the CDE estate at all. Keep control-plane administration in a different trust tier from application delivery, with separate identities and separate approval paths. Treat the audit pipeline as part of the boundary, because it is the only record that survives a control-plane compromise - and ship it off the cluster, since an attacker with API write access can also reach the log backend if the backend is a pod. And accept the honest conclusion: in a single cluster, "the CDE is bounded" means "bounded against everyone except the people who administer the cluster", and that residual set has to be named in the scope document rather than left implicit.

## 6. Pod-Level Controls

### 6.1 The Three Profiles

The Pod Security Standards define "three different policies to broadly cover the security spectrum", and the policies are "cumulative and range from highly-permissive to highly-restrictive" ✅.

- **Privileged** - "Unrestricted policy, providing the widest possible level of permissions. This policy allows for known privilege escalations." ✅
- **Baseline** - "Minimally restrictive policy which prevents known privilege escalations. Allows the default (minimally specified) Pod configuration." ✅
- **Restricted** - "Heavily restricted policy, following current Pod hardening best practices." ✅

**Why the cumulative structure matters for scope.** Restricted includes everything in Baseline, so a namespace at Restricted gets the Baseline controls as a floor: host namespaces must be disallowed (`hostNetwork`, `hostPID`, `hostIPC`), privileged containers disallowed, capabilities limited to a named allowlist (AUDIT_WRITE, CHOWN, DAC_OVERRIDE, FOWNER, FSETID, KILL, MKNOD, NET_BIND_SERVICE, SETFCAP, SETGID, SETPCAP, SETUID, SYS_CHROOT), HostPath volumes forbidden, host ports restricted, and seccomp not explicitly set to Unconfined ✅. Those are the controls that stand between a compromised container and the node's kernel and filesystem - which, per sec. 3.3, is where the real blast radius is.

### 6.2 Pod Security Admission

**What it is.** A built-in admission controller that enforces the profiles. "Feature state: Stable since Kubernetes v1.25" ✅, and restrictions "are applied at the namespace level when pods are created" ✅.

**The three modes.** Enforce - "Policy violations will cause the pod to be rejected." Audit - a violation "will trigger the addition of an audit annotation to the event recorded in the audit log, but are otherwise allowed". Warn - a user-facing warning, otherwise allowed ✅. "A namespace can configure any or all modes, or even set a different level for different modes." ✅

**The labels.** `pod-security.kubernetes.io/<MODE>: <LEVEL>` for each of enforce, audit and warn, plus an optional `pod-security.kubernetes.io/<MODE>-version` pinned to a minor version such as v1.37 or to `latest` ✅. The version pin is the detail most often missed: pinning matters because the profile's content changes between minor releases - the Restricted profile's use of `spec.os.name` changed in v1.25, for instance ✅ - so an unpinned namespace is enforced against whatever the cluster's current version ships.

**Exemptions.** Exemptions can be configured statically for usernames, RuntimeClass names and namespaces, and exempt requests have "all `enforce`, `audit` and `warn` behaviors" skipped ✅. The documentation adds a caution worth quoting into any control document: controller service accounts "should generally not be exempted, as doing so would implicitly exempt any user that can create the corresponding workload resource" ✅.

### 6.3 The Enforce Asymmetry

This is the single most important sentence in the Pod Security Admission documentation for anyone using the control as evidence: "both the audit and warning modes are applied to the workload resources. However, enforce mode is not applied to workload resources, only to the resulting pod objects." ✅

**What it means in practice.** A Deployment whose pod template violates Restricted is *accepted* by the API server. The Deployment is created, the ReplicaSet is created, and the controller then tries to create pods - which are rejected. The operator sees a Deployment that exists and never becomes available, and a ReplicaSet accumulating failure events. The `kubectl apply` in the pipeline succeeds. The manifest in Git looks compliant enough to have been merged. The only thing that is actually blocked is the pod.

**Why that is a scope problem, not just an ergonomics problem.** Three consequences follow, and all three are evidenced in a card-data estate.

First, a delivery pipeline that gates on apply-success gates on nothing. The gate has to watch rollout status, or the admission layer has to be a validating policy that sees the pod template in the workload object (sec. 8.3).

Second, a policy repository can contain a large number of violating workload manifests that no check ever surfaced, because enforce never looked at them. The clean-up is real work and it is invisible until the day a controller restarts the workload.

Third, and most subtly, the asymmetry changes what an auditor can conclude from a clean cluster. "No pods were rejected this week" is compatible with "no violating pods were attempted" and with "violating pods were attempted and rejected", which is why the admission controller's own metrics - evaluations, errors and exemptions - are the artefact that distinguishes them ✅.

### 6.4 The Restricted Controls in Detail

The Restricted profile's published control list, read this pass ✅, contains:

| Control | Requirement in the profile | Why it matters for a card-data pod |
| --- | --- | --- |
| Privilege escalation | `allowPrivilegeEscalation` must be `false` (Linux, where `spec.os.name` is not windows) | Blocks set-user-ID and set-group-ID escalation inside the container |
| Running as non-root | `runAsNonRoot` must be `true`, at pod or container level | A root process in a container is one kernel bug away from the node |
| Running as a non-root user | `runAsUser` must not be `0` | Closes the case where runAsNonRoot is set but the image's user is root |
| Seccomp | Profile must be explicitly `RuntimeDefault` or `Localhost`; both `Unconfined` and the *absence* of a profile are prohibited | The syscall filter is the main reduction of the kernel attack surface |
| Capabilities | Must drop `ALL`, and may add back only `NET_BIND_SERVICE` | Removes the capability set a container breakout usually needs |
| Volume types | Only configMap, csi, downwardAPI, emptyDir, ephemeral, persistentVolumeClaim, projected and secret | Excludes hostPath and other node-reaching volume types |
| Everything from Baseline | Host namespaces, privileged containers, hostPath, host ports, the capability allowlist | The floor |

**One control that is conspicuously not in the list.** A read-only root filesystem is a widely recommended hardening measure and is not among the controls the Restricted profile publishes ✅. It is worth knowing because teams routinely describe their workloads as "restricted-compliant" while running a writable root filesystem, and the phrase invites the assumption that everything they associate with hardening is included. It also has to be said in the other direction: `readOnlyRootFilesystem: true` is a good idea for a card-data container and it is not a substitute for any of the controls above.

**The version history that matters.** The seccomp control dates from v1.19, capabilities from v1.22, the non-root-user control from v1.23, and the OS-specific handling of these three from v1.25 ✅. A cluster on an older minor is not enforcing the same profile even with the same label value, which is why the version label exists.

### 6.5 Upstream Versus OpenShift

This distinction is stated once, plainly, because conflating the two produces controls that do not exist on the platform in question.

**Upstream Kubernetes, as described in this guide:** Pod Security Standards (the three profiles), Pod Security Admission (the built-in controller, stable since v1.25, with the three modes and namespace labels), the `securityContext` fields themselves (`runAsNonRoot`, `runAsUser`, `allowPrivilegeEscalation`, `seccompProfile`, `capabilities`, `readOnlyRootFilesystem`), and the fact that PSA is a *non-mutating* admission controller - it validates pods and will not modify them, which the project's migration guidance states directly ✅.

**OpenShift-specific, owned elsewhere and not restated here:** Security Context Constraints as the policy object, the restricted-v2 and nonroot-v2 policy families, random UID assignment and the namespace annotation model for UID ranges, and the platform's own defaults for workloads. These are the subjects of [Secure Red Hat OpenShift](secure_red_hat_openshift_guide.md), [OpenShift SCC Comprehensive](openshift_scc_comprehensive_guide.md), [OpenShift SCC and Service Accounts](openshift_scc_service_account_guide.md), [OpenShift Random UID and Username Mapping](openshift_random_uid_username_mapping.md) and [OpenShift Workload Availability](openshift_workload_availability_guide.md). The single most consequential difference for the argument in this guide is behavioural: PSA validates and does not mutate, whereas the OpenShift model can assign values the workload did not specify - which means the same manifest can be compliant on one platform and not the other, and the *evidence* for compliance is a different artefact on each.

**Why this is a scope issue and not pedantry.** A control described as "restricted PSA" and evidenced with an SCC listing is an assessment finding waiting to happen, in both directions: the artefact does not correspond to the control, and the control does not run on the platform. sec. 15 records the conflation as a rejected claim.

### 6.6 What PSA Does Not Do

**It does not make the namespace a boundary.** PSA is namespace-level admission control driven by labels, and the labels are protected by permission on the namespace object (sec. 3.2) ✅. It hardens what runs inside the namespace; it does not keep anything out of the namespace, and it does not isolate the namespace from the rest of the cluster.

**It does not see inside the image.** A pod can satisfy every Restricted control and still run a container image with a vulnerable base layer, a shell, a cryptocurrency miner, or an embedded credential. PSA validates the pod spec, not the artefact (sec. 8).

**It does not cover the workload object in enforce mode** (sec. 6.3), so a policy repository can be full of violations that the cluster has never evaluated.

**It does not restrict the network, the data, or the credentials.** A Restricted pod can still reach every pod on the node unless a NetworkPolicy says otherwise, can still mount a Secret, and can still use its service-account token to read every Secret in its namespace (sec. 7.4).

**It does not apply everywhere.** Namespaces labelled privileged are unrestricted by design, cluster infrastructure runs in namespaces that are typically exempt or privileged, and exemptions skip all three modes for the subjects they name ✅. A cluster-level claim of "we enforce restricted" is a claim about a subset of namespaces, and the subset is the thing to enumerate.

---

## 7. The Secrets Problem

### 7.1 Base64 Is Not Encryption

A Kubernetes Secret's `data` field holds base64-encoded values. Base64 is an encoding: it exists to carry arbitrary bytes through a text-oriented API, it has no key, and anyone can decode it. This is visible in any Secret manifest - the examples in the project's own documentation show values such as `dmFsdWUtMg0KDQo=` in the `data` field ✅ - and it is not a contested claim: `kubectl get secret -o yaml` returns the value, and `base64 -d` returns the plaintext.

**Why the confusion persists.** The object is called a Secret, it is separated from ConfigMaps in the API, the kubelet takes care to store mounted secret data in `tmpfs` so it is not written to durable storage ✅, and access to it is governed by RBAC. Those are real properties and they create a feeling of protection. What they do not do is encrypt anything.

**And the storage default is worse than the encoding.** "Kubernetes Secrets are, by default, stored unencrypted in the API server's underlying data store (etcd). Anyone with API access can retrieve or modify a Secret, and so can anyone with access to etcd. Additionally, anyone who is authorized to create a Pod in a namespace can use that access to read any Secret in that namespace; this includes indirect access such as the ability to create a Deployment." ✅ That paragraph is the whole threat model of the object, and it is in the project's own documentation.

### 7.2 Encryption at Rest and Its Limits

sec. 5.3 covers the mechanism and the providers. Two points belong here, because they are what a Secrets-focused assessment will ask.

**It protects the datastore, not the API.** Encryption at rest changes what is on disk. It does not change the fact that an authorised API read returns plaintext, and it does not change the fact that a pod-creating identity can read secrets. An assessment that asks "are Secrets encrypted" and stops at "yes, at rest" has answered a narrower question than the one that matters in a card-data namespace.

**The rewrite is the control, not the flag.** Enabling the provider configuration only affects objects written afterwards; the migration step is a read-and-replace of every Secret in the cluster ✅. Until that has been run and can be shown to have been run, the honest statement is "new Secrets are encrypted".

### 7.3 External Secret Stores

**What the project recommends.** Among the prescribed steps for using Secrets safely: "Consider using external Secret store providers" ✅, with the Secrets Store CSI driver linked as the mechanism.

**What it changes.** The credential's authoritative copy lives in a secrets manager with its own authentication, authorisation, audit and rotation. The pod receives it at mount time, through a CSI volume, under an identity the store can authorise separately from the cluster's RBAC. The namespace stops holding the credential as an object, so the permission that reads every Secret in the namespace no longer yields the card-data credential.

**What it does not prove.** That the pod cannot read the credential - of course it can, it is using it; the question is what else can. That the CSI driver's own identity is narrow - a driver that can read a whole path in the store is a single point of credential access, and it is a control-plane-adjacent component that belongs in scope. That the mounted value never touches the node's disk, which depends on the driver's mount semantics. And that a "sync to a Kubernetes Secret" mode, where the driver materialises an object for compatibility, is not in use - because that mode reintroduces the etcd copy the architecture was meant to remove ⚠ (this mode exists in the driver's documentation, which was not read in this pass, so it is flagged rather than asserted).

### 7.4 The Service Account Token

**What it is.** "A service account is a type of non-human account that, in Kubernetes, provides a distinct identity in a Kubernetes cluster." ✅ Namespaced, and every namespace gets a `default` ServiceAccount on creation; the default accounts "get no permissions by default other than the default API discovery permissions" ✅. A pod that does not specify a service account is assigned the namespace's default ✅.

**How the token is delivered now, and how it used to be.** "In v1.22 and later, Kubernetes gets a short-lived, automatically rotating token using the `TokenRequest` API and mounts the token as a projected volume." ✅ "In versions earlier than 1.22, Kubernetes provides a long-lived, static token to the Pod as a Secret." ✅ Projected tokens "expire automatically, and the kubelet rotates the token before it expires" ✅, whereas tokens mounted as Secrets "don't expire and don't rotate" ✅. The `LegacyServiceAccountTokenNoAutoGeneration` feature gate was enabled by default from v1.24 to v1.26 and removed in v1.27 when the behaviour reached GA ✅ - so on any currently supported cluster, a long-lived token exists only if somebody created it deliberately.

**Audience and expiry are the two fields that make a token narrow.** A token "might also have an expiry time, an audience, and a time after which the token starts being valid" ✅, and the API server checks audience claims as part of validation ✅. TokenRequest tokens are *bound*: "the API server also checks that the specific object reference that is using the ServiceAccount still exists, matching by the unique ID of that object" ✅, and the project notes that TokenReview validation invalidates a bound token immediately when its pod is deleted, whereas OIDC validation would treat it as valid until expiry ✅. For a card-data workload, that difference is the difference between a credential that dies with the pod and one that outlives it.

**Version facts.** Audience restriction for kubelet-issued tokens (`ServiceAccountNodeAudienceRestriction`) is beta and enabled by default since v1.33 ✅. The `kubernetes.io/enforce-mountable-secrets` annotation is deprecated since v1.32, and the replacement advice is "Use separate namespaces to isolate access to mounted secrets" ✅ - which is the project telling practitioners to use namespaces as the isolation unit for secrets, and therefore worth reading alongside sec. 3.2's argument about what a namespace can and cannot prove.

### 7.5 The Rotation Question

Three different clocks run in a card-data cluster, and conflating them is how a rotation control becomes a document rather than a mechanism.

**Token rotation** is automatic for projected tokens and tied to the pod's lifetime (sec. 7.4). It needs no process; it needs the legacy Secret-based path to be absent.

**Credential rotation** - the database password, the payment gateway key, the signing key - is a process, and in a cluster it has a specific shape: the external store rotates, the pod re-reads, and the application tolerates the change without a restart. Where the credential is a Secret object, rotation means updating the object and restarting or signalling the workload, and the audit question becomes who did it and when.

**Encryption-key rotation** is the at-rest key, and the documentation's own procedure is multi-step: add a new key as the second entry, restart all API servers, make a secure backup, promote the new key to first, restart again, rewrite every Secret, then remove the old key ✅. A control that says "keys are rotated annually" without that procedure having been executed is a control that describes an intention.

### 7.6 The Non-Human Identity Cross-Reference

The discipline that makes the above a programme rather than a set of object configurations - identity inventory, credential lifetime, rotation mechanics, the difference between authentication and authorisation for workloads, and the ownership model for non-human identities - is [Non-Human Identity](non_human_identity_guide.md). This section states only what a card-data scope needs from it: that every identity able to read a CDE credential is enumerated, that its credential has a bounded lifetime, and that the enumeration can be regenerated from the cluster rather than maintained by hand. Re-deriving the discipline here would duplicate it and, worse, produce a second version that drifts.

---

## 8. The Supply Chain

### 8.1 Provenance and the Build Claim

**What provenance is.** A signed statement about how an artefact was produced: which source revision, which build system, which pipeline, which parameters. It is the difference between "this image is called payments-api:2.4.1" and "this image was built by this pipeline from this commit".

**What it proves, and what it does not.** It proves the artefact's origin, and therefore that the thing deployed is the thing that was reviewed and scanned. It does not prove the source is free of defects, that the dependencies are current, or that the build environment was not compromised. The component half of the question - what is inside - is the SBOM, owned for the procedural estate by [SBOM Generation for C, Pro\*C and COBOL](sbom_c_proc_cobol.md), and the pipeline half is [Security-by-Design](security_by_design_guide.md) sec. 6. This guide's interest is narrower: whether the artefact that landed in the CDE node pool can be tied to a build at all.

### 8.2 Signing and Verification

**The mechanism, from the project's own documentation.** Sigstore's cosign verifies a signature over an image: `cosign verify` takes a key, a key URL or a KMS URI and an image reference, and keyless verification uses an identity and an issuer instead of a key - `cosign verify <image URI> --certificate-identity=... --certificate-oidc-issuer=...` ✅. Attestations are verified separately with `cosign verify-attestation` ✅. The documentation also states the property that makes the check meaningful: "Signature payloads created by `cosign` included the digest of the container image they are attached to. By default, `cosign` validates that this digest matches the container during `cosign verify`." ✅ A signature over a tag would be worthless, because tags move; a signature over a digest is a signature over an artefact.

**What it proves.** That a specific identity signed a specific digest. **What it does not prove.** That the identity should be trusted for this purpose - that is a policy decision made in the verifier, and it is where keyless verification's flexibility becomes a requirement to pin both the identity and the issuer. Nor does it prove that anybody *checked*. A signed image in a registry is a claim; a signature verified at admission is a control.

### 8.3 Admission Control

**The modern in-process option.** ValidatingAdmissionPolicy is "Stable since Kubernetes v1.30" ✅ and offers "a declarative, in-process alternative to validating admission webhooks" ✅. Policies are written in CEL; a ValidatingAdmissionPolicy and a ValidatingAdmissionPolicyBinding are both required for the policy to have any effect, and an optional parameter resource makes the policy's thresholds concrete ✅.

**Why in-process matters for a CDE.** A webhook is a network call in the API server's admission path. If the webhook is unavailable, the failure policy decides whether the cluster rejects everything or allows everything, and both outcomes are bad in a card-data estate. An in-process policy has no such dependency. It also evaluates the *workload object* as well as the pod, which closes the sec. 6.3 asymmetry: a policy can reject a Deployment whose pod template violates the rules, at the moment the Deployment is submitted, rather than at the moment the controller tries to create a pod.

**What upstream does not do.** Kubernetes has no built-in image-signature verification. A policy that requires a valid signature needs either a policy engine with a verifier, or a webhook that calls one - which returns the availability problem above. That is a real architectural choice and it belongs in the design, not in the deployment checklist.

**The rules a CDE admission layer should carry**, stated as requirements rather than as CEL: images must be referenced by digest or by an immutable tag; images must be signed by an identity in an allowed set; pods in the CDE namespace must carry the node selector and toleration for the CDE pool; pods must not set `nodeName`; privileged and host-namespace pods must be rejected regardless of namespace label; and service-account automounting must be disabled unless the workload needs the API.

### 8.4 The Unsigned Image Finding

**Why it is a finding and not a hygiene issue.** An unsigned image in a card-data node pool means the artefact running in the CDE cannot be tied to a build. There is no evidence that it is the artefact that was reviewed, scanned or approved - and in an estate where the assessment's argument rests on the boundary being maintained at runtime, an artefact of unknown origin inside the boundary is a hole in the argument at its most basic level. The question is not "was it signed" in the abstract; it is "can you show that what is running now came from your pipeline".

**The ordinary ways it happens.** A debug image pulled by hand during an incident and left running. A vendor agent installed by a DaemonSet from a public registry. A base image rebuilt by a CI system that has no signing step for that repository. A rollback to an older digest that predates the signing programme. Each is a normal operational event, and each produces the same assessment gap.

### 8.5 Tags, Digests and Registries

**Tags move; digests do not.** A deployment that references a tag is a deployment that will run something different the next time the tag is pushed, and the pod spec gives no indication of which artefact it actually got. A digest reference is immovable, and it is the only form in which "the artefact in the CDE" is a stable claim.

**The registry is a trust anchor and therefore in scope.** Whoever can push to the registry from which the CDE pulls can put code inside the boundary without touching the cluster at all - no Kubernetes permission required. Registry authorisation, image retention and immutability settings, and the pull credentials held in the cluster (which are themselves Secrets, sec. 7.1) are all part of the boundary argument, and none of them is a Kubernetes control.

---

## 9. Logging and the Audit Trail

### 9.1 The API Audit Log

**What it is, in the project's words.** "Kubernetes auditing provides a security-relevant, chronological set of records documenting the sequence of actions in a cluster." ✅ It records requests to the API server: who, what verb, what resource, what namespace, what the response was, and when.

**What that makes it good for.** Establishing that an identity did something: created a pod, read a Secret, changed a role binding, deleted a policy. In a scope argument, that is the record of who changed the boundary. It is the only artefact that survives a control-plane compromise as evidence of what the compromise did (sec. 5.7), which is why shipping it off the cluster is part of the boundary design rather than part of the logging design.

### 9.2 Stages and Levels

**Stages.** RequestReceived, ResponseStarted, ResponseComplete and Panic ✅. ResponseStarted applies to long-running requests - a watch, for instance - where the response headers are sent before the body completes.

**Levels.** None, Metadata, Request and RequestResponse ✅. Metadata records who asked for what without bodies; Request adds request bodies; RequestResponse adds response bodies. The level choice is the difference between an audit trail that fits in a retention budget and one that captures the contents of every Secret read - which is itself a security decision, because an audit log at RequestResponse on secrets is a copy of the credentials.

**The rules requirement.** The audit policy must contain rules: "The `rules` field must be provided in the audit policy file. A policy with no (0) rules is treated as illegal." ✅ A file that exists but has no rules is not a permissive configuration; it is an invalid one, and the API server will not start with it.

### 9.3 Backends and the Omitted Flag

**Backends.** Log files and webhooks ✅. Truncation is disabled by default ✅ - relevant because a truncated audit record of a large object is not a record of what was written.

**The sentence that decides whether a cluster has an audit trail at all.** "You can pass a file with the policy to kube-apiserver using the `--audit-policy-file` flag. If the flag is omitted, no events are logged." ✅ No events. Not a reduced set, not a warning - nothing. And nothing in the cluster, the API or the dashboards reports the absence: a cluster with auditing off looks exactly like a cluster with auditing on but quiet.

**Three consequences for evidence.** First, "we have audit logging" has to be evidenced by the loaded policy, not by the intent to have one - the flag and its file are the artefact. Second, a log backend that runs as a pod in the same cluster is reachable by anyone who has compromised the control plane, so the durable copy has to be outside. Third, the policy file is configuration on the control-plane host, which means changes to it are themselves an event worth alerting on - and if the change removes a rule before the event is written, the record of the change may be the only trace.

### 9.4 Container, Node and CNI Logs

**Container logs.** Whatever the process writes to stdout and stderr, collected by the kubelet and rotated at the node. They are ephemeral in the specific sense that matters: when the pod is deleted, replaced or evicted, its log stream ends, and when the node is removed, the local files go with it. A card-data application's own log of what it did with a PAN is therefore a node-level artefact that needs a collector with its own durability, and the collector is a privileged DaemonSet - a workload inside the boundary with read access to every container's output.

**Node logs.** The kubelet, the container runtime and the OS audit subsystem. These are where the evidence of container escapes, runtime-level anomalies and node-level access lives, and they are outside the cluster's own API-visible state.

**CNI logs.** The plugin's own records of policy decisions, including denies. This is the only place where "the NetworkPolicy actually blocked something" can be evidenced (sec. 4.2): the policy object proves intent, and the plugin's counters and logs prove enforcement. A cluster with a policy set and no plugin logging has a control with no observation.

### 9.5 What an Audit Log Proves

**It proves:** that a request was made, by an identified principal, at a recorded time, with a recorded verb, resource, namespace and outcome. That is a strong and specific claim, and it is the claim the standard's logging family is really about.

**It does not prove:** that the pod which made the request was in the CDE - the log records the pod's service account and namespace, not which node it landed on, and node placement is exactly the runtime fact the scope argument depends on (sec. 3.1). That the data involved was cardholder data - the log records that a Secret was read, not what was in it. That the request's *effect* was what the caller intended - a successful pod creation is not a successful workload. That anything happened inside the container, which is the application's log and not the API's. And that the record is complete, if the policy's level omitted the stage or the rule that would have captured it.

**The practical consequence.** The audit log answers "who changed the cluster", and the scope question is "where is the data". Those are different questions, and a card-data estate needs both answered - the first from the API audit log, the second from placement evidence: node labels and taints as they were at the time, the scheduler's binding records, and the workload inventory reconciled against the node pool (sec. 10.3).

### 9.6 Toward Requirement 10

Requirement 10 is the logging and monitoring family, and this guide asserts its number while flagging its title per sec. 2.3. What a cluster contributes to it, and what it does not, is worth stating precisely.

**Contributes:** a complete record of control-plane actions, if the policy file exists and is scoped correctly; per-workload application logs if a collector is durable; CNI enforcement records; and node-level records. Together these are a substantial part of what the family asks for.

**Does not contribute:** the review. The standard's logging family is about logs being generated, protected, retained and *reviewed*, and no cluster feature performs the review. A log pipeline that writes to durable storage and is never read is a control that produces evidence nobody examines - and the review record is an organisational artefact, produced outside the cluster, which is why sec. 10.3 lists it as a separate item with its own limits.

**Does not contribute either:** time integrity. Records from the API server, the nodes, the CNI and the application are only correlatable if their clocks agree, and clock synchronisation is a node-configuration fact rather than a cluster feature. An audit trail whose timestamps disagree by minutes cannot establish sequence, and sequence is most of what an investigation needs.

## 10. The Assessment

### 10.1 What the Assessment Is Asked to Believe

An assessment of a card-data estate on a cluster is being asked to believe one proposition: that the cardholder data environment is bounded, and that it is bounded now. Everything else - the policies, the profiles, the encryption, the signatures - is instrumental to that proposition.

**A note on what this section is and is not.** The questions below are the questions this guide's argument implies, and in practitioner experience they are the ones that arise when a cluster-hosted CDE is examined. That is a statement about practice, not about any standard, and it is offered as practitioner experience rather than as a description of how an assessor will behave. This guide does not assert what a QSA would accept, because that is a judgement made by a qualified organisation in a specific engagement, and any guide that told you the answer would be guessing about somebody else's professional judgement.

**What the assessment has that the operator does not.** An assessor sees the boundary claim as a whole and tests it for internal consistency: does the diagram match the inventory, does the inventory match the cluster, does the cluster match the evidence, does the evidence cover the period. Each of those comparisons is a place where a cluster-hosted CDE commonly fails, and the failures are usually mismatches rather than absences.

### 10.2 The Questions

1. **Where is the cardholder data?** Not which namespace - which datastores, which volumes, which backups, which log streams, which caches, which message queues, and which clusters. The answer is an inventory of locations, not a diagram of components.
2. **Which nodes can run a pod that can reach that data?** The answer is a set of nodes, identified by label and taint, and it has to be the set as it exists today.
3. **Who can create a pod in the namespace that holds the credentials?** Because that permission implies reading every Secret in the namespace (sec. 5.2), the identity list is a scope fact, not an operational detail.
4. **Who can read those Secrets directly, and through which bindings?** Including the indirect paths: a ClusterRole binding, an aggregated role, a controller's service account.
5. **What enforces the network boundary, and how do you know it is loaded and working?** The policy objects are intent; the plugin's configuration and its deny records are enforcement (sec. 4.2).
6. **What proves that no pod ran outside the pool in the last period?** This is the question the whole guide exists to answer, and the only honest answers are reconciliations that were run and kept.
7. **Who can change the boundary, and what would show that they did?** Admission policies, namespace labels, node taints and the audit policy are all boundary-defining objects, and changes to them are boundary changes.
8. **What happens to the boundary during an upgrade, a failover or an incident?** A CDE that is bounded except during maintenance windows is bounded for part of the year.

### 10.3 The Artefact Table

| Question | Artefact | What the artefact does not prove |
| --- | --- | --- |
| Where is the data | Data-flow and data-location inventory, reconciled against actual volumes, object storage and database instances | That the inventory is complete; that a workload did not write card data somewhere unclassified |
| Which nodes can run CDE pods | Node inventory with labels and taints, plus the node-pool definitions from the provisioner | That the current membership matches the definition; that an autoscaled node was tainted before it took work |
| Who can create pods | Rendered role bindings and an effective-permission review per identity | That the review covered indirect paths such as controller service accounts and aggregated roles |
| What enforces the network | CNI configuration showing policy enforcement enabled, plus policy objects and plugin deny counters or logs | That the plugin enforced every policy; that node-level traffic was restricted at all |
| That placement stayed in scope | A dated reconciliation of pod-to-node placement against the node pool, run repeatedly and retained | That the reconciliation ran often enough to catch a short excursion; that the underlying node labels were accurate at the time |
| That workloads were hardened | PSA labels per namespace, admission controller metrics including evaluations, errors and exemptions, and rejected-pod events | That the workload objects were compliant when submitted (sec. 6.3); that exempted subjects were safe |
| That secrets were protected | Encryption provider configuration, evidence of the rewrite of existing Secrets, and the external store's authorisation model | That the rewrite covered every object; that the API read path is restricted |
| That images are traceable | Signature verification results per deployed digest, with the verifier's identity policy | That verification happened at admission rather than in a report; that the signed artefact is free of vulnerabilities |
| That the trail is complete | The audit policy as loaded, the audit backend configuration, and retention and review records | That every relevant action was in scope of a rule; that anyone read the records |
| That the boundary is governed | A responsibility matrix naming an owner for each boundary-defining object | That the owner exercised the ownership; that the matrix is current after the last reorganisation |

**How to read the table.** Each row's third column is longer than its second, and that is the point. An artefact is a claim about a moment, produced by a process, limited by its own method - and the assessment question is always whether the artefact's limits have been accounted for. A submissions pack that presents artefacts without their limits is asking an assessor to fill in the gaps by inference, which is precisely the inference a cluster makes unsafe.

### 10.4 A Design Document Is Not Evidence

**The design document is a claim about intent.** It says what the boundary was meant to be, who was meant to own it, and which controls were meant to enforce it. All of that is necessary - a scope has to be declared before it can be tested - and none of it is evidence that the intent holds now.

**Why the gap is larger in a cluster than elsewhere.** In a static estate, the design and the running system diverge slowly: somebody has to change a firewall rule or install software on a server, and both leave traces. In a cluster, the running system diverges from the design *by operating normally*. A rolling update replaces every pod. A node upgrade drains and recreates. An autoscaler adds machines. A developer adds a toleration to make a deployment schedule. None of these is a change to the design, and each of them can change the answer to "where can the card data go".

**What replaces it.** Artefacts that are generated by the system, dated, and repeatable: the reconciliation in sec. 10.3, the admission controller's own metrics, the CNI's deny records, the audit log's record of boundary-defining object changes. A design document plus a year of dated reconciliations is an argument. A design document alone is a hope with a version number.

### 10.5 Scope Versus Runtime

**Two scopes exist, and they are not the same scope.** The declared scope is the set of systems the assessment covers, written down and agreed. The runtime scope is the set of systems that can actually reach the data right now. The compliance claim is that the second is a subset of the first, and the interesting cases are the ones where it is not.

**Why the subset relationship fails.** Three mechanisms, each ordinary. Placement drift: a pod that belongs in the CDE lands outside it and still holds the data in memory, which means the data is now on a node outside the declared scope (sec. 3.1). Credential reach: an identity outside the scope can read a Secret inside it, so a system outside the declared scope can obtain inside authority (sec. 5.2). Shared infrastructure: a log pipeline, registry or backup target serves both scopes and is therefore in the runtime scope whether or not it is in the declared one (sec. 4.4).

**The question that resolves it.** For each of the three, the question is the same one: *what stops it, and what would show that the stopping happened?* A taint stops placement, and the reconciliation shows it held. An RBAC binding stops credential reach, and the effective-permission review shows the binding. A separate cluster stops shared infrastructure, and the two inventories show the separation. In every case the answer is a mechanism plus an observation, and in no case is the answer a document.

---

## 11. The Banking and Regulated Angle

### 11.1 Two Regimes and One Cluster

**The card-industry standard and prudential supervision ask different questions about the same cluster.** The standard asks whether cardholder data is protected within a defined scope. Supervision asks whether the bank's critical operations can withstand and recover from disruption, whether risk is governed and owned, and whether third parties are managed. A bank running a card-data estate on Kubernetes answers to both, and the answers are not interchangeable: a scoped, well-evidenced CDE says nothing about whether the payment service can be restored within its impact tolerance, and an operational-resilience programme says nothing about whether the CNI enforces network policy.

**Why the two collide in a cluster.** The properties that satisfy the standard - isolation, dedicated capacity, minimal blast radius - are the properties that reduce operational flexibility. A dedicated node pool cannot absorb load from the general estate. A separate cluster doubles the upgrade work. Encryption key rotation is a change with an outage risk. Every scope decision is therefore also a resilience decision, and sec. 13.3 is where that trade is priced.

### 11.2 Operational Resilience

**The restore question.** The test of an operational-resilience claim is not "is there a backup" but "can the service be restored, by whom, in what time, with what data loss". For a cluster-hosted CDE that decomposes into several specific questions: can the etcd snapshot be restored, and has it been (an untested snapshot is a file); can the workloads be recreated from Git, and are the images still in the registry at the pinned digests; do the external secret stores still hold the credentials the restored workloads expect; and does the restored cluster come back with its taints, labels and admission policies, or does it come back as a generic cluster onto which the CDE workloads are then scheduled by an unconstrained scheduler?

**The failover scope problem.** A disaster-recovery site is a second CDE. If the standby cluster runs in a different region or account, the card data can go there, which means the scope includes it, the evidence includes it, and the controls have to hold there too. Estates routinely assess the primary and describe the standby, and the standby is where the taints were never applied because the runbook was written for the primary.

**The upgrade window.** A cluster on a supported release line is upgraded several times a year, and each upgrade drains nodes. The boundary has to hold across the drain - which means the replacement pods have to land in the same pool, which means the taint, the toleration and the admission policy have to survive the upgrade. The machinery for the resilience side of this is [Operational Resilience Framework](../banking/operational_resilience_framework_guide.md); the scope side is sec. 12.2's first row.

### 11.3 Third-Party Risk

**Five suppliers in one boundary.** The cloud or platform provider owns the control plane and the physical estate; the CNI vendor owns whether network policy is enforced; the registry vendor owns what artefact is served; the secrets-manager vendor owns the credential store; the observability vendor owns whether the audit trail survives. A CDE on a managed cluster therefore has a supply chain that is wider than the application's dependency list, and each supplier is a party whose behaviour the scope claim depends on.

**What to ask a supplier, in scope terms.** Not "are you certified" but: which of the boundary-defining controls do you operate, what evidence do you produce for them, at what frequency, and what happens to that evidence if we leave. A CNI vendor that cannot show policy enforcement telemetry has sold a plugin, not a control. The assessment and exit machinery is [Vendor Management](../management/vendor_management_guide.md); this section's contribution is the list of controls to point it at.

**The concentration question.** If the same provider operates the CDE cluster, the non-CDE cluster, the registry and the log pipeline, then the separation in sec. 4.4 is a logical separation inside one supplier's estate. That may be entirely acceptable, and it is a different claim from physical separation - the scope document should say which one it is making.

### 11.4 Change Governance

**Every boundary-defining object is a change-controlled object.** Namespace labels, node taints, admission policies, NetworkPolicies, role bindings, encryption configuration and the audit policy all define the boundary. In most estates they are changed by pull request against a GitOps repository, reviewed by whoever reviews application changes - which means the boundary is governed by the same process as a feature flag.

**The three changes that most often escape review.** A toleration added to a deployment to fix a scheduling failure, which widens where the workload can land. A namespace label changed from restricted to baseline to unblock a release. An audit policy rule removed to reduce log volume. Each is a small diff, each is a boundary change, and each looks like an operational fix in a pull request.

**What good looks like, in process terms.** Boundary-defining objects live in a separate repository or directory with a distinct reviewer set; changes carry a scope impact statement; the change record links to the evidence that the boundary still holds afterwards. That is a governance answer, and it is the only one that survives staff turnover - which is also the answer to the supervisor's question below.

### 11.5 What a Supervisor Asks

**Can you restore the service?** Not "do you have a plan" - the impact tolerance for the payment service, the last successful restore test, and who decided the recovery objective. A cluster estate answers this with etcd restore drills, Git-based rebuilds and registry retention, and the answer is a date.

**Who owns the control?** Not the platform team in aggregate - a named role for network policy, for node hardening, for admission policy, for the audit pipeline. A cluster distributes these across four teams and a supplier, and the failure mode is that each believes another owns it. The responsibility matrix in sec. 10.3 is the artefact, and the supervisor's version of the question is whether the matrix survives a reorganisation.

**What happens when the cluster is unavailable?** The concentration question: which services stop, what the manual fallback is, and whether the fallback itself touches card data outside the CDE - because a manual workaround that copies a PAN into a spreadsheet is a scope failure created by an outage.

**And the exit question.** If the platform must be replaced, what happens to the data, the keys and the evidence. A CDE whose encryption keys, audit history and image provenance all live inside one supplier's estate has an exit problem that is a resilience problem and a scope problem at once.

### 11.6 The Council Line on Applicability

**The sentence to quote to anyone who asks whether the standard applies.** "Whether an entity is required to comply with or validate compliance to a PCI SSC standard is at the discretion of organizations that manage compliance programs, such as a payment brand, acquirer, or other entity." ✅

**Why it matters for a bank.** It means the obligation's shape - which validation, at what level, on what cycle - comes from the acquirer or the brand rather than from the standard itself. The practical consequence is that a bank should record what was contractually required of it, and by whom, alongside its technical scope. A CDE that is scoped more narrowly than the acquirer's requirement is a contractual problem even if it is a defensible technical position, and a CDE scoped more broadly is an unnecessary cost. Neither is a question this guide can answer, and neither is answered by a figure - which is why no figure appears anywhere in this guide.

---

## 12. The Failure Modes

### 12.1 How to Read This Section

The rows below are failures: things that break, with a symptom an operator would actually observe, a cause that explains why the symptom appears, and a guardrail that would have caught it. They are distinct from sec. 14's anti-patterns, which are beliefs rather than breakages - what people convince themselves is fine. The distinction matters because failures get incident reviews and beliefs do not.

**On the symptoms.** Each is written the way it appears in a real operational setting: a reconciliation report, a failed test, a scanner finding, a dashboard with a flat line. The value of writing them this way is that a reader can recognise one, and recognition is the only thing that turns a list of risks into an improvement.

### 12.2 The Table

| Symptom | Cause | Guardrail |
| --- | --- | --- |
| A weekly placement report shows pods from the card-data namespace running on nodes that do not carry the CDE label, typically for the duration of a node upgrade | The workload expressed its placement preference as a node affinity preference rather than a requirement, and its toleration was copied into a template that also runs elsewhere; the drain moved the pods and the scheduler honoured the preference loosely | A validating admission policy that rejects any pod in the CDE namespace lacking a required node affinity, the toleration and a tolerated taint, plus a reconciliation that runs daily and pages on any out-of-pool placement |
| A segmentation test connects successfully from a non-CDE pod to the card-data service, while every NetworkPolicy object is present and unchanged | The CNI was replaced during a platform upgrade with a plugin that does not implement policy, or policy enforcement was disabled in the plugin's configuration; the API server accepted the objects regardless (sec. 4.2) | A standing segmentation test that attempts a denied connection and alerts when it succeeds, plus a check that reads the CNI's own enforcement configuration rather than counting policy objects |
| A repository scanner reports a base64 blob that decodes to a production database password, in a GitOps repository with broad read access | The Secret was committed as a manifest because the pipeline needed a value, and base64 was mistaken for protection (sec. 7.1) | Commit-time secret scanning with blocking, an external secret store so the manifest carries a reference rather than a value, and a rotation runbook that assumes any committed credential is compromised |
| A pod in an unrelated namespace reads a card-data credential, and the review finds no policy was violated | A ClusterRole binding granted secret-read broadly, and the namespace boundary was believed to be a security boundary (sec. 3.2) | Effective-permission review per identity including cluster-scoped bindings, and an admission policy that refuses cross-namespace secret references for CDE workloads |
| The image inventory contains digests in the CDE node pool with no signature and no attestation, usually a vendor agent or a debug image | The workload was installed by a DaemonSet from a public registry, or an image was pulled by hand during an incident and never replaced (sec. 8.4) | Admission that verifies signatures for every pod in the CDE namespace with no namespace-level exemption for infrastructure, and an inventory job that fails when a running digest has no verification record |
| The SIEM returns zero audit events for the API server over the whole retention window, and nobody noticed until an investigation | The API server was started without `--audit-policy-file`, so no events were logged at all (sec. 9.3) | Configuration drift detection on the API server flags, an alert on the *absence* of audit events rather than on their content, and a control test that queries the SIEM for a known event |
| A pipeline reports success and the release never serves traffic; the Deployment exists, the ReplicaSet shows repeated pod-creation failures | Pod Security Admission's enforce mode applies to pods, not workload objects, so the violating Deployment was accepted and only its pods were rejected (sec. 6.3) | Gate on rollout status rather than apply-success, and a validating admission policy that evaluates the pod template in the workload object at submission |

### 12.3 The Common Thread

**Every row is a control that was configured and not observed.** In each case something was set up correctly at some point - a policy, a taint, a Secret strategy, an audit intent - and the failure is that nothing watched whether it was still true. The cluster did not defeat the control; the estate stopped looking.

**Why a cluster makes this the dominant failure mode.** Because the running state is rebuilt constantly, the question "is the control in place" has a different answer every day, and a review that answers it annually answers it about a cluster that no longer exists. The guardrail column is therefore almost entirely made of observations - reconciliations, standing tests, absence alerts, effective-permission reviews - rather than of additional configuration. That is the practical form of the thesis: you cannot document your way to a bounded CDE, because the thing you are documenting changes without you.

---

## 13. The Cymbal Bank Worked Example

*Cymbal Bank is a fictional institution used here to make the trade-offs concrete. The estate described is illustrative, not a description of any real deployment, and the numbers are deliberately absent - the argument is about shape, not scale.*

### 13.1 The Estate as Designed

**The design.** Cymbal's card-processing estate ran in one shared cluster. A `payments` namespace held three workloads: a card-authorisation API that received authorisation requests, held a PAN in memory for the duration of a call and wrote a token to the ledger; a settlement batch that reconciled the day's transactions; and a checkout service that served the payment page and loaded the scripts that page required. The namespace had NetworkPolicies restricting ingress to the authorisation API from the checkout service and the ingress gateway, Pod Security Admission labels at restricted, and its own service accounts. Secrets for the ledger and the payment gateway were Secret objects in the namespace, encrypted at rest with a KMS provider. Images were built by a pipeline with an SBOM and a signature. On paper - and on the diagram in the scope document - the CDE was the `payments` namespace.

**What the design got right.** A great deal. The network policy set was written deny-by-default and reviewed. The Restricted profile was enforced and the workloads complied. Encryption at rest used the KMS provider rather than a local key, so a copy of etcd was not a copy of the credentials. Signing and SBOM generation were real. The scope document named the namespace, the data flows, and the systems outside the cluster that touched the data.

**What the design assumed.** That the namespace was where the data was. That the pods in it ran on whichever nodes the scheduler chose, and that this did not matter because the namespace was the boundary. That the shared node pool was acceptable because the workloads in it were all "internal". That the payment page, being a web tier, was a lower-risk component than the authorisation API and could share the cluster.

### 13.2 The Decision the Runtime Forced

**The event.** A minor-version upgrade drained the shared node pool over a weekend. The authorisation API's Deployment carried a node affinity expressed as a preferred term - a leftover from a capacity experiment - and a toleration for a taint that had been removed months earlier. During the drain, the replacement pods were scheduled onto the general pool, where they ran for the duration of the maintenance window. The data did not leave the cluster and nothing was disclosed. What happened was that the cardholder data environment spent several hours on nodes that the scope document said were outside it, and the reconciliation that would have shown it did not exist.

**The reasoning that followed.** The team's first instinct was to fix the affinity. The analysis that survived review was different: the reason the drift was possible was that the boundary had been expressed as a namespace, and a namespace has no opinion about nodes. Three options were considered. Keep the namespace and add admission that forces placement - cheap, and it leaves the payment page and the authorisation API sharing a control plane and a kernel. Move the whole CDE to a dedicated node pool with taints and admission - better, and it still leaves an internet-facing web tier inside the same control plane as the authorisation path. Separate the clusters - expensive, and it addresses the control-plane problem that no in-cluster option touches.

**What Cymbal chose.** A hybrid, and the shape of it is the point. The authorisation API and the settlement batch moved to a dedicated, tainted node pool in a cluster reserved for card data, with admission policy enforcing the node affinity, the toleration and the prohibition on raw `nodeName`; the pool's provisioning template carries the taint so autoscaled nodes arrive tainted. The checkout service moved to its own cluster, separate from the card-data cluster, because it is internet-facing, because it loads third-party scripts, and because its change cadence is an order of magnitude faster - and because Requirement 6's payment-page-script applicability notes (sec. 2.2) put it in the argument regardless of the fact that it holds no PAN.

**Why not the dedicated pool alone.** Because the control plane is shared by everything in a cluster (sec. 5.7), and a compromise of the checkout service's ability to write pods would have been a compromise of the same API server that hosts the authorisation path. Cluster separation was the only option that removed that path, and it was bought for the internet-facing component rather than for the whole estate - which is the compromise most banks end up making, and it should be described as a compromise rather than as a design principle.

### 13.3 What It Cost

**Capacity.** A dedicated node pool is sized for the card-data peak and cannot borrow from the general estate. Cymbal's utilisation in that pool settled far below the shared pool's, and the difference is the price of the boundary. It is a real cost and it is the first thing a finance partner will ask about; the answer is that the alternative is a scope that cannot be evidenced.

**Upgrade drag.** Two clusters means two upgrade windows, two sets of node drains, two opportunities for the boundary to move during maintenance. The card-data cluster's upgrades are now scheduled with the reconciliation running before and after, which turns a routine operation into a controlled one - correct, and slower.

**Duplication.** Policy repositories, observability pipelines, registry configuration, CI pipeline definitions and on-call rotations were duplicated or parameterised. The duplication is small in absolute terms and large in maintenance: every platform change now has two places to land, and the two drift unless something checks.

**The developer-experience tax.** The payment page team's local development no longer matches production, because production has a cluster boundary the local environment does not. Manifests carry tolerations and node affinities that exist for scope reasons rather than functional ones. Onboarding takes longer. This tax is paid by the teams least equipped to see why it exists, and the mitigation that worked was documentation written for developers rather than for assessors - what the constraints are, which ones can be changed, and who to ask.

**The reconciliation burden.** A scheduled job now compares every pod's node against the CDE pool's membership, records the result, and raises a ticket on any mismatch - sec. 13.4 sets out what it does and does not prove. The cost is a standing maintenance obligation that did not exist before the boundary did, and the benefit is that the scope document became an input to automation rather than a document beside it. That is the healthiest thing that came out of the exercise.

### 13.4 The Reconciliation Burden

**What the job actually does.** It reads the pods in the in-scope namespaces, resolves each to a node, checks the node's labels and taints against the CDE pool definition, and writes a dated record. It also reads the admission policies and namespace labels, so that a change to a boundary-defining object appears in the same record as a change in placement. The output is boring, which is the point: a year of boring records is an argument, and one interesting record is an incident.

**What it does not do.** It does not prove the data was not copied somewhere by an application. It does not prove the CNI enforced anything. It does not cover the standby cluster unless the job runs there too - which, after the first DR test, it does. Its limits are the limits in sec. 10.3, and they are written on the report rather than discovered later.

### 13.5 The Thesis Restated

**Cymbal's scope document did not change on the weekend of the upgrade.** The namespace was still called `payments`, the policies were still in Git, the diagram was still accurate about intent. What changed was where the pods ran, and it changed because a scheduler made a decision that no document was consulting.

**The fix was not a better document.** It was a set of mechanisms - a tainted pool, an admission policy, a separate cluster, a reconciliation - each of which constrains a runtime decision and each of which produces an observation. The document now describes those mechanisms and points at their evidence, which is the only relationship between a scope document and a cluster that survives an upgrade.

**And the general lesson, which is the guide's thesis.** PCI DSS does not care that you are running Kubernetes; it cares where the cardholder data can go - and a scheduler decides that at runtime.

## 14. The Anti-Patterns

### 14.1 Failure Modes and Anti-Patterns

Section 12 listed things that break. This section lists things that do not break, and that is what makes them dangerous: each is a belief that is close enough to the truth to survive a review, a design discussion or an audit interview, and each substitutes a name for a mechanism. The distinguishing question is always the same: *what would show that this is false?* If the answer is "nothing we collect", the belief is doing the work that evidence should be doing.

### 14.2 The Table

| Symptom - what you hear in the review | Cause - the belief underneath | Guardrail |
| --- | --- | --- |
| "We have NetworkPolicies covering the CDE namespace." Nobody has read the CNI's configuration, and no denied connection has ever been attempted | That the existence of the object is the enforcement of the policy; the API server accepts policy objects whether or not any controller implements them (sec. 4.2) | A standing segmentation test that must fail to connect, plus a check of the plugin's enforcement mode, plus the plugin's deny counters as a monthly artefact |
| "The namespace is isolated, so the card data cannot leave it." | That an API scoping construct is a security boundary; the belief survives because RBAC does scope to namespaces and because the namespace shows up in every diagram (sec. 3.2) | Name the mechanism that isolates it - taint, CNI policy, admission - and produce evidence for each; if the answer is only "the namespace", there is no boundary |
| "Secrets are protected - they are Secrets, not ConfigMaps, and the values are encoded." | That base64 is encryption and that the object type confers protection; the belief survives because the object is called a Secret and because mounted data goes to tmpfs (sec. 7.1) | Show the encryption provider configuration and the evidence that existing objects were rewritten, and show that pod-creating identities cannot read the values |
| "We enforce PSA restricted, so the pods are safe." | That pod-spec hardening is workload security; the belief survives because the profile is genuinely strong at what it covers (sec. 6.4) | Pair the label with image provenance, network policy, secret access and an inventory of exempted namespaces and subjects; state that the profile does not see inside the image |
| "The diagram is the scope - it was signed off by the architecture board." | That a design-time artefact describes a runtime system; the belief survives because the diagram is accurate about intent and because nobody re-derives it (sec. 10.4) | A dated reconciliation of placement against the node pool, retained; the scope document references the evidence rather than replacing it |
| "Auditing is on - the logs are in the SIEM." | That a log pipeline implies an audit policy; the belief survives because the SIEM genuinely has Kubernetes events, just not audit events (sec. 9.3) | Query the SIEM for a specific API audit event type and alert on its absence; treat the loaded audit policy as the artefact |
| "Admission control is in place - we have a validating webhook." | That the webhook's existence is its availability; the belief survives because the webhook works in normal operation (sec. 8.3) | Know the failure policy, test it by making the webhook unavailable, and prefer in-process policies for the rules that must not fail open |
| "The CDE node pool is tainted, so nothing else can run there." | That a taint is a permission rather than a filter; the belief survives because the taint does repel pods that do not tolerate it (sec. 4.3) | Admission that controls who may set the toleration and rejects a raw `nodeName`; a periodic audit of which pods carry CDE tolerations and why |
| "Images are pinned - we deploy version tags, not latest." | That a version tag is immutable; the belief survives because the tag looks like a version and because most tags are never overwritten (sec. 8.5) | Digest references in the deployed spec, and admission that rejects tag-only references in the CDE namespace |

### 14.3 Why They Survive

**Each belief is a name for a real thing.** NetworkPolicy is real, namespaces are real, PSA restricted is real, taints are real. The anti-pattern is not a fantasy; it is a real control standing in for a stronger claim than it supports. That is why the beliefs survive review - a reviewer who knows the object exists has no reason to ask whether it is enforced, and asking requires a second artefact that nobody produced.

**They are cheap to hold and expensive to test.** Holding the belief costs nothing. Testing it costs a segmentation test, a reconciliation job, an effective-permission review, an audit-event query - and each of those produces a result that might be uncomfortable. The economics favour the belief until something forces the test, which in practice is an incident or an assessment.

**They are reinforced by the platform's own ergonomics.** The API accepts objects it cannot enforce. A Deployment with a violating template applies successfully. A cluster with auditing off looks identical to one with auditing on. The system is designed to accept declarations, and the anti-patterns are declarations mistaken for observations. The correction is structural rather than cultural: make the observation a required artefact, and the belief stops being sufficient.

---

## 15. The Claims Audit

### 15.1 The Method

**What was read, and how.** Every standards claim in this guide was read at the issuing body's own site during this pass, and every Kubernetes claim was read at kubernetes.io with the page's last-modified date recorded below. **The retrieval date for every page cited below is 2026-10-27**; the last-modified date given in each row is the page's own publication stamp, not the retrieval date. Page extraction was direct; no claim rests on a search snippet, a vendor blog or a secondary summary. Repository measurements were made locally with `grep` over the markdown files and are reproducible with the commands implied by sec. 1.4.

**What the verdicts mean.** ✅ means verified this pass against the issuing body's publication. ⚠ means flagged and not to be relied on. ❌ means the claim was tested and does not hold. Where a claim is standard practice that no primary source states in the form asserted, it is marked ⚠-structural and said to be so.

### 15.2 The Claims Table

| Claim | Where cited | Verdict | Verification basis and quality |
| --- | --- | --- | --- |
| PCI DSS v4.0.1 is the current version, published 11 June 2024 | sec. 1.3, sec. 2.1 | ✅ | Council document library lists v4.0.1 as current; Council blog post dated 11 June 2024 announces it. Two independent Council pages, high quality |
| v4.0.1 is a limited revision with no additional or deleted requirements | sec. 2.1 | ✅ | Direct quotation from the Council's announcement post; high quality - the issuing body describing its own revision |
| v4.0 retired 31 December 2024; the 31 March 2025 future-dated effective date unchanged | sec. 2.1 | ✅ | Same Council post, two separate questions answered verbatim; high quality |
| v3.2.1 retired 31 March 2024 | sec. 2.1 | ✅ | Council post on the future-dated requirements; high quality |
| The Council's document index carries a `last_updated` stamp of 2024-06-11 for v4.0.1, which is a stamp and not a printed publication date | sec. 2.1 | ✅ | Council's own machine-readable index, read directly; high quality, with the interpretive caveat stated in the text |
| Requirement families 3, 6, 8 and 12 as the four the Council's revision note describes | sec. 2.2, sec. 2.4 | ✅ | The Council post enumerates changes by requirement number and subject; high quality for the numbers and subjects, not for the titles |
| Requirement 11.3.2 is the ASV vulnerability-scan requirement; 6.4.3 and 11.6.1 concern payment page scripts | sec. 2.2 | ✅ | Council post names 11.3.2 in that role; the Council's own supporting-document title names 6.4.3 and 11.6.1 with a payment-page-script subject; high quality |
| The twelve requirement-family titles as stated in sec. 2.3 | sec. 2.3, sec. 2.4 | ⚠ | Titles are the standard's published family subjects as generally circulated; the standard text was not read at the Council this pass (licence gate, sec. 2.3). Flagged throughout |
| The figures of 64 new requirements and 51 future-dated | sec. 2.1 | ⚠ | Quoted from an interviewee (an assessor) in a Council blog post, not from the Council or the standard; attributed in the text |
| No PCI DSS sub-requirement text is quoted anywhere in this guide | Whole guide | ✅ | By construction: every quotation is from a Council page that was read, not from the standard PDF |
| PCI is a false friend: 84 files mention `PCI` (case-sensitive substring; 86 case-insensitive, 67 on a word boundary), 29 mention `PCI DSS`, 13 are PCIe, and the four-file join with kubernetes or k8s is incidental | sec. 1.4, sec. 15.4 | ✅ | Measured locally this pass over `*.md`, excluding this guide; reproducible, and the command basis is stated in the text because the count changes with case sensitivity and word boundaries |
| CDE is a false friend: 6 files match, used for a university college, a data-platform compliance environment and the cardholder data environment | sec. 1.4, sec. 15.4 | ✅ | Measured locally this pass; the alternate senses were read in the matching files |
| No PCI DSS Kubernetes guide existed in this repository before this one | sec. 1.5 | ✅ | Local search: 29 files mention PCI DSS, none is a PCI DSS guide; the densest mention is the release-readiness guide's sec. 4.6 |
| The release-readiness guide's sec. 4.6 asserts no requirement number | sec. 1.5 | ✅ | Read directly in that guide, including its own claims-audit entry saying so |
| No penalty, fine, card-scheme fee, assessment-cost or revenue figure appears anywhere in this guide | Whole guide | ✅ | Checked by eye and by search for currency symbols and numeric penalty prose; none present. Recorded here as the check itself |
| NetworkPolicy requires a CNI that implements it, and a policy object with no implementing controller has no effect | sec. 3.3, sec. 4.2 | ✅ | kubernetes.io network policies, last modified 2026-04-14, quoted verbatim |
| Policies are additive; the default is non-isolated; node traffic is always allowed | sec. 4.2 | ✅ | Same page, quoted verbatim; high quality |
| Taints repel pods; a pod with `nodeName` bypasses the scheduler even past a `NoSchedule` taint | sec. 3.1, sec. 4.3 | ✅ | kubernetes.io taints and tolerations, last modified 2026-07-27, quoted verbatim; high quality |
| Pod Security Standards define three cumulative profiles with the controls listed in sec. 6.4 | sec. 6.1, sec. 6.4 | ✅ | kubernetes.io pod security standards, last modified 2026-10-16; control list read directly, including the v1.19, v1.22, v1.23 and v1.25 control ages |
| A read-only root filesystem is not among the controls the Restricted profile publishes | sec. 6.4 | ✅ | Absence, read from the same page's enumerated control list this pass; high quality as a reading of the page |
| Pod Security Admission is stable since v1.25, has three modes, uses namespace labels, and enforce does not apply to workload objects | sec. 6.2, sec. 6.3 | ✅ | kubernetes.io pod security admission, last modified 2024-03-07; the enforce asymmetry quoted verbatim |
| Anyone who can update, patch or create a namespace can change its Pod Security level | sec. 3.2, sec. 6.6 | ✅ | kubernetes.io migration task, last modified 2023-04-15, quoted verbatim; high quality |
| Pod Security Admission is a non-mutating admission controller | sec. 6.5 | ✅ | Same migration page, stated directly |
| PodSecurityPolicy was deprecated in favour of Pod Security Admission as of July 2021 | sec. 6.5 | ✅ | Pod security standards page, FAQ; the removal minor version is not asserted anywhere in this guide and is flagged below |
| Secrets are unencrypted in etcd by default; anyone authorised to create a pod can read any Secret in the namespace, including via a Deployment | sec. 1.2, sec. 3.2, sec. 7.1 | ✅ | kubernetes.io secrets, last modified 2026-07-23, quoted verbatim; high quality |
| A privileged container on a node can access all Secrets used on that node | sec. 3.3, sec. 7.1 | ✅ | Same page, quoted verbatim; high quality |
| Secret `data` values are base64-encoded | sec. 7.1 | ✅ | Visible in the same page's own manifest examples and in every Secret object; stated as the encoding fact it is |
| At-rest encryption is off by default; the identity provider gives no confidentiality; the provider list and their caveats; etcd v3.x required; custom resources need v1.26; wildcards need v1.27 | sec. 5.1, sec. 5.3 | ✅ | kubernetes.io encrypting data at rest, last modified 2025-05-09; provider table read directly, including the kms v1 deprecation at v1.28 and kms v2 stable at v1.29 |
| Local key storage protects against etcd compromise and not host compromise; KMS requires compromising both etcd and the provider | sec. 5.3 | ✅ | Same page, quoted verbatim |
| Enabling encryption does not rewrite existing objects; the migration is a read-and-replace of every Secret | sec. 5.3, sec. 7.2 | ✅ | Same page, whose own procedure is that read-and-replace |
| Audit stages are RequestReceived, ResponseStarted, ResponseComplete and Panic; levels are None, Metadata, Request and RequestResponse; omitting `--audit-policy-file` means no events are logged; a policy with zero rules is illegal | sec. 5.4, sec. 9.2, sec. 9.3 | ✅ | kubernetes.io auditing, last modified 2025-03-16, each sentence quoted verbatim; high quality |
| Audit backends are log files and webhooks, with truncation disabled by default | sec. 9.3 | ✅ | Same page |
| Service account tokens are short-lived and auto-rotating from v1.22 via TokenRequest and a projected volume; earlier versions used long-lived static tokens in Secrets; those do not expire | sec. 7.4 | ✅ | kubernetes.io service accounts, last modified 2026-03-16, quoted verbatim |
| `LegacyServiceAccountTokenNoAutoGeneration` was default-enabled v1.24 to v1.26 and removed in v1.27; node audience restriction is beta and default-enabled since v1.33; `enforce-mountable-secrets` is deprecated since v1.32 | sec. 7.4 | ✅ | Same page, stated directly |
| ValidatingAdmissionPolicy is stable since v1.30, is in-process, is declared in CEL, and needs both a policy and a binding | sec. 8.3 | ✅ | kubernetes.io validating admission policy, last modified 2026-09-28, quoted verbatim |
| The current release line is 1.37, with 1.37.1 released 2026-09-15 and 1.37, 1.36 and 1.35 supported | sec. 3.1 | ✅ | kubernetes.io releases, read this pass; the three-minor support statement is quoted |
| cosign verifies signatures by key or by keyless identity and issuer, verifies attestations separately, and validates that the signed digest matches the image | sec. 8.2 | ✅ | docs.sigstore.dev verification page, read this pass; the digest statement quoted verbatim. The page carries no last-modified date |
| The CIS Kubernetes Benchmark is at version 2.1.0 | sec. 5.6 | ✅ | cisecurity.org benchmark page, read this pass; version only |
| CIS Kubernetes Benchmark control text | sec. 5.6 | ⚠ | Behind a download form; no control number is cited anywhere in this guide |
| PSA, PSS and securityContext are upstream Kubernetes; SCC, random UIDs and restricted-v2 are OpenShift-specific | sec. 1.2, sec. 6.5 | ✅ | Upstream side read at kubernetes.io this pass; OpenShift side attributed to the five owning guides, not restated |
| RBAC is additive with no deny rules, and cluster-scoped bindings apply everywhere | sec. 5.5 | ⚠-structural | Standard, widely documented behaviour; the RBAC page was not re-read in this pass and no quotation is offered |
| The four families named by the Council (3, 6, 8, 12) are the verified anchors | sec. 2.2 | ✅ | Council revision note, as above |

### 15.3 Flagged

| Item | Why it is flagged | What would settle it |
| --- | --- | --- |
| The wording of the twelve requirement-family titles | The standard text sits behind a click-through licence at the Council's document host; this pass did not accept the licence and therefore did not read it (sec. 2.3) | Accept the licence in a browser session, retrieve the current PDF, and pin the titles and numbers to v4.0.1 |
| The 64 new requirements and 51 future-dated figure | An interviewee's count in a Council blog post, not Council text or standard text | Count them in the standard, or find a Council statement of the number |
| CIS Kubernetes Benchmark control content | Download form; the PDF was not read | Download the benchmark under the organisation's CIS terms and cite section numbers from the copy held |
| The minor version in which PodSecurityPolicy was removed | The pages read this pass describe the deprecation and the migration but did not state the removal version in the text read | Read the deprecation and removal notes for the specific release |
| The Secrets Store CSI driver's "sync to a Kubernetes Secret" mode | The driver's documentation was not read this pass; the mode is described in sec. 7.3 as a risk and flagged there | Read the driver's own documentation for the current release |
| RBAC's additive semantics | Not re-read at kubernetes.io in this pass; marked structural in sec. 5.5 | Read the RBAC page and quote it, or cite the authorisation overview |
| Any specific CNI plugin's enforcement behaviour | No plugin's documentation was read; sec. 4.2 makes the general argument from the project's own statement that enforcement is the plugin's responsibility | Read the chosen plugin's policy documentation and test enforcement in the cluster |
| Service mesh permissive-mode defaults | Not read at any mesh project's documentation this pass; described as the normal migration state from practitioner experience | Read the mesh's own authentication-policy documentation |
| What an assessor will accept | Never asserted as fact anywhere in this guide; assessor expectations are labelled practitioner experience in sec. 10.1 | An actual engagement, with the qualified organisation's own judgement |

### 15.4 Rejected

| Claim | Where tested | Verdict | Why it was rejected |
| --- | --- | --- | --- |
| "PCI" in this repository means PCI DSS | sec. 1.4 | ❌ | 84 files mention PCI, 29 mention PCI DSS, 13 are the PCIe bus. The acronym is not a reliable search term, and the four-file join with Kubernetes is incidental in all four |
| "CDE" in this repository means the cardholder data environment | sec. 1.4 | ❌ | Six files match; the acronym is used for a university's College of Design and Engineering, for a data-platform compliance environment and for the cardholder data environment. A grep returns nonsense |
| This repository already contains PCI DSS material that covers Kubernetes scope | sec. 1.5 | ❌ | No PCI DSS guide exists; the densest mention is one section of a release-readiness guide that expressly asserts no requirement number, and every container guide in the repository is OpenShift-shaped |
| The OpenShift guides can be cited for plain-Kubernetes pod-security behaviour | sec. 6.5 | ❌ | SCC, random UIDs and the restricted-v2 family are OpenShift mechanisms. Citing them for PSA behaviour, or PSA for SCC behaviour, mismatches the control to the artefact |
| A namespace is a security boundary | sec. 3.2, sec. 14.2 | ❌ | It scopes names and RBAC and hosts policy labels; it does not confine credentials, the kernel or the network, and its own policy label is writable by anyone who can edit the namespace |
| A requirement number not read at the issuing body can be asserted | sec. 2.3 | ❌ | It is exactly the fabrication the discipline in sec. 2.3 exists to prevent; the guide asserts only the numbers the Council itself cites |

### 15.5 The Honest Ledger

**The largest gap is the standard's own text, and it is a licence gate rather than an oversight.** Every PDF under the Council's document host returns an HTTP 403 with a click-through "Read and Copy License" interstitial requiring a session and agreement cookies. This pass did not forge that acceptance, which means the twelve family titles are flagged, the sub-requirement text is absent, and the requirement numbers this guide asserts are only the ones the Council names in its own publications. A reader who needs the titles and testing procedures should take them from the current PDF, pinned to v4.0.1, and record the retrieval.

**The second gap is benchmark content.** The CIS Kubernetes Benchmark is verified to exist at version 2.1.0 and its text is behind a download form, so no control number appears in this guide. That is a smaller gap than it sounds - the benchmark is a hardening baseline and this guide's argument is about scope - but it means any claim of the form "benchmark-compliant" in a downstream document is unverified here.

**The third gap is the Kubernetes pages that were not read.** RBAC's semantics, the scheduler's internals, the network plugin interface and the pod-security exemption configuration were not read at kubernetes.io in this pass, and the claims that depend on them are marked structural or omitted. Where a claim needed a quotation to be safe, the quotation is present; where it does not, the claim is marked.

**And the gap that is not a gap.** This guide asserts nothing about what an assessor would accept, and it prints no figure of any kind - no penalty, no fee, no assessment cost, no revenue. Those are not omissions to be filled in later; they are the boundary of what a research pass can honestly say.

---

## 16. What Could Not Be Verified, the Glossary, the Cross-References and the Closing Summary

### 16.1 What Could Not Be Verified and the Residual Gaps

**The requirement-family titles, and the licence gate behind them.** The twelve families are numbered 1 to 12 throughout this guide, and their subjects are stated in the order the standard publishes them. The *wording* of those titles was not read at the Council, because the standard's PDFs sit behind a click-through licence that this pass declined to accept. The four families the Council itself describes in its revision note - 3, 6, 8 and 12 - are verified by the Council's own words, as are the requirement numbers 11.3.2, 6.4.3 and 11.6.1. Everything else about the twelve is flagged.

**The benchmark text.** The CIS Kubernetes Benchmark exists at version 2.1.0 ✅ and its content was not read ⚠. No control number is cited anywhere in this guide.

**The removal version of PodSecurityPolicy.** The deprecation is verified ✅ and the migration path is documented ✅; the minor version of removal is not asserted, because the pages read this pass did not state it in the text that was read.

**Per-plugin and per-mesh behaviour.** No CNI plugin's or service mesh's documentation was read in this pass. The argument in sec. 4 is built from the project's own statement that enforcement is the plugin's responsibility, which is enough to establish the dependency and not enough to describe any particular implementation.

**Everything that depends on a specific institution's contract.** Which validation applies, at what level, on what cycle, is at the discretion of the payment brand or acquirer ✅, and no guide can supply it.

### 16.2 The Glossary

| Term | Definition as used in this guide |
| --- | --- |
| **PCI DSS** | The card industry's baseline of technical and operational requirements for protecting payment account data; current version v4.0.1 |
| **CDE** | Cardholder data environment: the people, processes and technology that store, process or transmit cardholder data or sensitive authentication data, plus those that could impact its security |
| **CHD** | Cardholder data - the account data the standard protects |
| **SAD** | Sensitive authentication data - the authentication elements associated with a card |
| **QSA** | Qualified Security Assessor: an independent organisation qualified and trained by the Council to perform PCI DSS assessments |
| **ASV** | Approved Scanning Vendor: an organisation qualified and trained by the Council to conduct external vulnerability scanning under the applicable requirement |
| **Namespace** | An API scoping construct inside a cluster; a naming, RBAC and policy-label boundary, not a security boundary |
| **NetworkPolicy** | A Kubernetes object selecting pods and declaring allowed ingress and egress; enforced only by a CNI plugin that implements it, and additive where several policies select the same pod |
| **CNI** | Container Network Interface: the plugin layer that provides pod networking and, in most implementations, the enforcement of network policy |
| **PSA** | Pod Security Admission: the built-in admission controller that enforces the Pod Security Standards at the namespace level; stable since v1.25 |
| **PSS** | Pod Security Standards: the three cumulative profiles - Privileged, Baseline, Restricted |
| **SCC** | Security Context Constraints: the OpenShift policy object for pod security; OpenShift-specific, owned by the OpenShift guides, not part of upstream Kubernetes |
| **Service account** | A namespaced non-human identity that a pod authenticates as, via a token that is short-lived and auto-rotating from v1.22 unless a legacy Secret-based token is used |
| **etcd** | The consistent key-value store holding the cluster's entire state, including Secrets, unencrypted by default |
| **Taint** | A node attribute that repels pods which do not carry a matching toleration; a scheduling filter, not a permission |
| **Admission controller** | A component in the API server's request pipeline that can validate or mutate an object before it is persisted; where Pod Security Admission, validating admission policies and webhooks run |

### 16.3 Cross-References

**Sibling guides in `technology/`:**

- [InfoSec Release Readiness](infosec_release_readiness_guide.md) - sec. 4.6 owns PCI DSS as one standard among many inside a release gate, and asserts no requirement number; the release gate and this guide's scope argument are complementary (sec. 1.5).
- [Security-by-Design](security_by_design_guide.md) - sec. 6 owns DevSecOps, CI/CD gates, SBOM and SLSA; sec. 8 of this guide consumes its output as provenance and signing evidence.
- [Non-Human Identity](non_human_identity_guide.md) - owns identity inventory, credential lifetime and rotation discipline; sec. 7.6 cross-references it rather than re-deriving it.
- [SBOM Generation for C, Pro\*C and COBOL](sbom_c_proc_cobol.md) - owns component inventory for the procedural estate; sec. 8.1 uses it as the composition half of provenance.
- [Secure Red Hat OpenShift](secure_red_hat_openshift_guide.md), [OpenShift SCC Comprehensive](openshift_scc_comprehensive_guide.md), [OpenShift SCC and Service Accounts](openshift_scc_service_account_guide.md), [OpenShift Workload Availability](openshift_workload_availability_guide.md), [OpenShift Random UID and Username Mapping](openshift_random_uid_username_mapping.md) - own SCC, random UIDs and OpenShift defaults; sec. 6.5 states the upstream-versus-OpenShift boundary and does not restate them.
- [Technology Lifecycle Management](technology_lifecycle_management_guide.md) - owns the release-line and end-of-life discipline that sets the upgrade cadence sec. 3.1 depends on.

**Guides in `banking/`:**

- [Operational Resilience Framework](../banking/operational_resilience_framework_guide.md) - owns the impact-tolerance, important-business-service and restore-testing machinery; sec. 11.2 uses its vocabulary for the failover-scope problem.
- [MAS Regulations and Guidelines](../banking/mas_regulations_guidelines_guide.md) - owns the verified MAS instrument map; sec. 11 does not re-derive it.

**Guides in `management/`:**

- [Vendor Management](../management/vendor_management_guide.md) - owns third-party assessment and exit; sec. 11.3 points it at the five suppliers inside a managed-cluster boundary.

### 16.4 The Closing Summary

**The argument in five sentences.** A scope is a statement about where data is, and in a cluster that statement is a prediction about a scheduling decision that is remade on every creation, restart, eviction and drain. A namespace scopes names, RBAC and policy labels, and confines neither credentials nor the kernel nor the network. The mechanisms that do constitute a boundary - a CNI that enforces policy, a tainted node pool with admission behind it, a separate cluster - each prove less than they appear to, and each needs an observation rather than a declaration. The control plane is the real perimeter, because whoever can write pods can place cardholder data anywhere and read any Secret in the namespace. And the evidence that answers the assessment question is dated, machine-generated and continuous, because the thing being evidenced changes daily.

**What to do on Monday.** Enumerate the namespaces, node pools and clusters that can hold card data. For each, name the mechanism that keeps it there, and then name the observation that shows the mechanism worked. Where the second answer is a document, replace it with a job. Where the answer is a namespace, find the real mechanism or accept that there is none.

**What this guide does not do.** It does not tell you what an assessor will accept, because that is a qualified organisation's judgement in a specific engagement. It does not supply requirement text it could not read at the Council. It does not print a figure of any kind. And it does not claim that a cluster cannot host a CDE - it claims that a cluster will not keep a CDE bounded unless somebody builds the boundary out of mechanisms and then watches them.

**The last word.**

PCI DSS does not care that you are running Kubernetes; it cares where the cardholder data can go - and a scheduler decides that at runtime.
