# IBM Cloud: The Hybrid House

*A solution-architect's deep-dive on IBM Cloud — the platform that IBM rebuilt from the SoftLayer bare-metal heritage and the Cloud Foundry-era Bluemix PaaS into the Red Hat OpenShift-centered hybrid cloud it is today: the history (SoftLayer 2013, Bluemix 2014, the IBM Cloud rebrand 2017, Red Hat 2018/2019), the platform surface (regions and multizone regions, bare metal, VPC, Power Systems Virtual Server, IBM Z and LinuxONE, storage, networking), the Red Hat integration (OpenShift, the Cloud Paks, the hybrid patterns), the services (the watsonx AI portfolio and Granite models, Db2/Cloudant data services, MQ/App Connect/Event Streams integration), the banking angle (the mainframe estate, IBM Cloud for Financial Services, the MAS Notice 658 outsourcing reality, the Singapore context), a head-to-head comparison with the other hyperscalers, and a worked example — Cymbal Bank's hybrid-cloud target architecture on the "hybrid house" pattern.*

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** Cloud-Platforms / Enterprise-Hybrid (technology/)
> **Audience:** Solution architects, integration architects, platform engineers, technical leads confronting IBM estates (mainframe, Power, middleware) and their cloud strategy in banking and beyond
> **Last Updated:** August 2026

**Cross-references:** [technology/ibm_as400_guide.md](technology/ibm_as400_guide.md) (the **platform-lineage** companion — the AS/400 → IBM i → Power Systems story this guide's PowerVS section builds on), [banking/core_banking_systems_guide.md](banking/core_banking_systems_guide.md) and the banking series (the **cores and channels** that the z/OS estate and the OpenShift plane serve — see [banking/banks_in_singapore_guide.md](banking/banks_in_singapore_guide.md) for the market context and [banking/dbs_bank_guide.md](banking/dbs_bank_guide.md) for the flagship Singapore digital-bank pattern), [banking/mas_regulations_guidelines_guide.md](banking/mas_regulations_guidelines_guide.md) (the **MAS instrument pyramid** — Notice 658 outsourcing and the Outsourcing Guidelines this guide's §6.3 builds on), [technology/zero_trust_network_architecture_guide.md](technology/zero_trust_network_architecture_guide.md) and [technology/cybersecurity_guide.md](technology/cybersecurity_guide.md) (the **security posture** the FS-ready controls map onto), [banking/ai_genai_banking_compliance_guide.md](banking/ai_genai_banking_compliance_guide.md) and [technology/ai_llm/enterprise_ai_platforms_guide.md](technology/ai_llm/enterprise_ai_platforms_guide.md) and [technology/ai_llm/rag/](technology/ai_llm/rag/) (the **AI-governance depth** — this guide's §5.1 points there instead of re-deriving AI governance), [banking/payment_rails_guide.md](banking/payment_rails_guide.md), [banking/swift_alliance_access_guide.md](banking/swift_alliance_access_guide.md) and [banking/swiftnet_fileact_guide.md](banking/swiftnet_fileact_guide.md) (the **messaging and file-transfer reality** that MQ and Event Streams integrate with, §5.4), [banking/oracle_banking_microservices_architecture_guide.md](banking/oracle_banking_microservices_architecture_guide.md) (the **Oracle banking-core alternative** in the §7 comparison).

**How to read this guide.** This is the *cloud-platform* deep-dive of the series: the AS/400 guide documents the midrange platform, the banking guides document what runs on the estates, and this guide documents the *cloud that wraps around them*. A fast path: read **§1** (what IBM Cloud is), **§2** (the history — the dates that matter), **§3** (the platform surface — regions, compute, PowerVS, Z, storage, networking), **§4** (the Red Hat integration — OpenShift and the Cloud Paks), **§5** (the services — watsonx, data, integration), **§6** (the banking angle — mainframe estate, FS-ready, MAS outsourcing, Singapore), and **§8** (the worked example that ties it to a real bank architecture); treat **§7** as the market map, **§9** as the summary, **§10**–**§11** as the honest verification record, **§12** as the glossary, and **§13** as the source list. Every factual claim carries a ✅ (verified from a source fetched in this pass), ⚠ (could not be confirmed from primary sources), or ❌ (refuted) marker; the unverified items are consolidated in **§11**.

---

## Table of Contents

1. [The Overview](#1-the-overview)
2. [The History](#2-the-history)
3. [The Platform Surface](#3-the-platform-surface)
4. [The Red Hat Integration](#4-the-red-hat-integration)
5. [The Services: Data, AI, and Integration](#5-the-services-data-ai-and-integration)
6. [The Banking Angle](#6-the-banking-angle)
7. [The Comparisons](#7-the-comparisons)
8. [The Worked Example: Cymbal Bank](#8-the-worked-example-cymbal-bank)
9. [The Summary: The Hybrid House](#9-the-summary-the-hybrid-house)
10. [Verification and Claims-Status](#10-verification-and-claims-status)
11. [What Could Not Be Verified](#11-what-could-not-be-verified)
12. [Glossary](#12-glossary)
13. [References](#13-references)

---

## 1. The Overview

### 1.1 What IBM Cloud Is

**IBM Cloud** is IBM's public-cloud and hybrid-cloud platform: a full-stack cloud offering infrastructure as a service (IaaS), platform as a service (PaaS), container and Kubernetes services, data and AI services, storage, networking, and industry-specific offerings such as IBM Cloud for Financial Services ✅. IBM's own positioning: "IBM Cloud is an enterprise cloud platform for building, running, and managing applications across hybrid and multicloud environments. It combines infrastructure, platform services, AI, and automation with strong security and compliance capabilities, making it ideal for regulated industries and mission-critical workloads" ✅ (ibm.com/cloud, fetched).

The platform's identity is best understood as three layers stacked on one strategic bet:

- **The infrastructure layer** — the SoftLayer heritage: bare-metal servers, virtual servers, and data centers around the world (see §2 and §3). IBM claims "over 230+ products and services covering data, containers, AI, IoT, and blockchain" in the portfolio ✅ (ibm.com/cloud, fetched).
- **The platform layer** — containers and Kubernetes as the center of gravity: IBM Cloud Kubernetes Service (IKS) and, decisively, **Red Hat OpenShift on IBM Cloud**, the fully managed OpenShift offering that has been the strategic platform since the Red Hat acquisition ✅ (§4).
- **The software layer** — the **Cloud Paks**: containerized, pre-integrated software (data, integration, automation, AIOps, network automation, applications) designed to run on OpenShift anywhere — on IBM Cloud, on other clouds, or on-premises ✅ (§4.3) — and the **watsonx** AI portfolio (§5.1).

The one-line summary that the rest of this guide unpacks: **IBM Cloud is not primarily a "big three" style hyperscaler competing on raw service count — it is the cloud built around the hybrid estate**, engineered to connect new cloud-native workloads to the existing IBM software and hardware (z/OS, Power, Db2, MQ, CICS) that large enterprises — and especially banks — already run ✅/⚠ (the "regulated industries and mission-critical workloads" positioning is IBM's own; how well it competes on price/volume is a market question, flagged in §7).

### 1.2 The Hybrid-Cloud Foundation: OpenShift

The Red Hat acquisition (announced October 2018, completed July 2019 — §2.5) made **Red Hat OpenShift** the common plane of IBM's strategy. OpenShift is Red Hat's enterprise Kubernetes container platform — "Build, modernize, and deploy apps at scale" per Red Hat's own description ✅ (redhat.com, fetched). On IBM Cloud it is delivered as **Red Hat OpenShift on IBM Cloud**, a fully managed service with a 99.99% service-level agreement, high-availability regional clusters, IAM integration, and GPU worker nodes for AI workloads ✅ (ibm.com/products/openshift, fetched). The strategic claim: the *same* OpenShift platform runs on IBM Cloud, on other clouds, and on-premises, so an application's deployment target is a choice, not a rewrite — the foundation of the "hybrid house" in §8.

### 1.3 The Overview Table

| Aspect | Description |
|---|---|
| **What it is** | IBM's public/hybrid cloud platform: IaaS, PaaS, containers, data, AI, integration, and industry clouds; positioned for regulated industries and mission-critical workloads ✅ |
| **IaaS heritage** | SoftLayer (acquired 2013): bare-metal and virtual servers, the classic infrastructure layer ✅ |
| **PaaS heritage** | Bluemix (2014): the Cloud Foundry-based PaaS, now folded into IBM Cloud ✅ |
| **Container platform** | IBM Cloud Kubernetes Service (IKS, 2017) and Red Hat OpenShift on IBM Cloud (2019) — the strategic common plane ✅ |
| **Software layer** | IBM Cloud Paks (containerized middleware on OpenShift) + watsonx AI portfolio + Db2/MQ/Event Streams services ✅ |
| **Hybrid reach** | IBM Cloud Satellite extends IBM Cloud services to on-premises and other clouds ✅ |
| **Industry cloud** | IBM Cloud for Financial Services — the FS-ready cloud with the IBM Cloud Framework for Financial Services ✅ |
| **Hardware heritage** | IBM Z (z/OS), LinuxONE, Power Systems (AIX/IBM i) — the estates the cloud is designed to connect to ✅ |
| **Market position** | Not a top-three hyperscaler by revenue; the differentiated play is hybrid + regulated industries ⚠ (market-share claims not independently verified in this pass) |

### 1.4 Why the Platform Matters to This Series

This repository's banking guides document banks running **Silverlake SIBS on IBM AS/400 (Power Systems)** ([banking/ocbc_software_systems_guide.md](banking/ocbc_software_systems_guide.md), [banking/uob_software_systems_guide.md](banking/uob_software_systems_guide.md)), core-banking systems on **IBM Z with CICS and COBOL**, and messaging estates built on **IBM MQ** ([banking/payment_rails_guide.md](banking/payment_rails_guide.md)). IBM Cloud is the platform those estates are increasingly being asked to *connect to, run beside, or move onto* — and this guide is the platform itself: what it is, how it got here, what it actually offers, and how a bank would use it.

---

## 2. The History

### 2.1 The Verified Timeline

The history of IBM Cloud is a story of three identities: the **SoftLayer IaaS** era (2005–2013), the **Bluemix PaaS** era (2014–2017), and the **IBM Cloud + Red Hat hybrid** era (2017–present). Every date below was verified against a source fetched during this research pass (Wikipedia's IBM Cloud article; IBM and PRNewswire press materials; IBM announcement archives); where a date could not be confirmed, it is marked ⚠.

| Year | Event | Status |
|---|---|---|
| 2005 | SoftLayer Technologies founded in Dallas, Texas by Lance Crosby and colleagues; dedicated hosting and cloud infrastructure provider | ✅ (Wikipedia, fetched) |
| 2007 | IBM launches the **Blue Cloud** initiative — a series of cloud-computing technologies for running enterprise applications in a decentralized manner | ✅ (Wikipedia, fetched) |
| 2010 | GI Partners takes a majority stake in SoftLayer (August); SoftLayer merges with competitor The Planet (November) | ✅ (Wikipedia, fetched) |
| 2011 | SoftLayer announces international expansion to Amsterdam and Singapore | ✅ (Wikipedia, fetched) |
| March 2011 | IBM opens its **Asia Pacific Cloud Computing Data Centre in Singapore** — a US$38 million investment; the first Singapore IaaS offering launches in April 2011 | ✅ (DataCenterKnowledge, fetched) |
| **4 June 2013** | **IBM announces a definitive agreement to acquire SoftLayer** — "the world's largest privately held cloud computing infrastructure provider"; financial terms not disclosed | ✅ (PRNewswire, fetched) |
| **8 July 2013** | **IBM closes the SoftLayer acquisition**; SoftLayer becomes the foundation of IBM's public cloud IaaS | ✅ (PRNewswire, fetched) |
| **February 2014** | **Bluemix announced for public beta** — a PaaS based on the open-source Cloud Foundry project, running on SoftLayer infrastructure | ✅ (Wikipedia + IBM announcement archive ENUSAP14-0304, fetched) |
| **July 2014** | **Bluemix general availability** | ✅ (Wikipedia, fetched) |
| 28 April 2014 | IBM announces 30+ additional Bluemix services at Pulse | ✅ (PRNewswire, fetched) |
| April 2015 | Bluemix passes 100 cloud services | ✅ (Wikipedia, fetched) |
| February 2016 | Bluemix adds serverless/FaaS (IBM Cloud Functions, seeded from Apache OpenWhisk) | ✅ (Wikipedia, fetched) |
| May 2017 | IBM Bluemix Container Service launches (Kubernetes); later renamed IBM Cloud Kubernetes Service (IKS) | ✅ (Wikipedia, fetched) |
| **October 2017** | **IBM announces the rebrand of Bluemix/SoftLayer to "IBM Cloud"**, retiring the Bluemix and SoftLayer brands — announced in IBM's blog post "Bluemix is now IBM Cloud" (October 2017), reported by trade press on 2 November 2017 | ✅ (Wikipedia + Cloud Computing News, fetched) |
| March 2018 | IBM launches managed Kubernetes on bare metal (industry first per Wikipedia) | ✅ (Wikipedia, fetched) |
| **October 2018** | **IBM announces intent to acquire Red Hat for ~US$34 billion** — IBM's largest acquisition in its history | ✅ month/amount (Wikipedia, fetched); ⚠ exact announcement day (widely reported as 29 October) not confirmed from a fetched primary source |
| **9 July 2019** | **Red Hat acquisition completed** — all outstanding shares acquired at US$190.00 per share in cash, ~US$34 billion total equity value | ✅ (IBM Investor news, fetched; PRNewswire; Wikipedia) |
| **August 2019** | IBM launches **managed Red Hat OpenShift on IBM Cloud**, about three weeks after the close | ✅ (Wikipedia, fetched) |
| **November 2019** | IBM announces the **financial services-ready public cloud** — "the world's first" — with Bank of America as first committed collaborator and anchor customer | ✅ (Wikipedia, fetched) |
| 2020 | BNP Paribas joins as first European anchor client | ✅ (Wikipedia, fetched) |
| **April 2021** | **General availability of IBM Cloud for Financial Services**, with Red Hat OpenShift support | ✅ (Wikipedia, fetched) |
| 2021 | SAP onboarded; CaixaBank joins on the new Madrid multizone region (September) | ✅ (Wikipedia, fetched) |
| **12 May 2026** | IBM announces **Red Hat AI Inference and Red Hat OpenShift Virtualization Service on IBM Cloud** (newsroom release referenced from the OpenShift product page) | ✅ (newsroom.ibm.com release link, seen on the fetched OpenShift page) |

### 2.2 The SoftLayer Era (2005–2013)

SoftLayer was a Dallas-based provider of **bare-metal servers, managed hosting, and cloud infrastructure** — notable for selling bare-metal compute as a first-class product before most large cloud providers did ✅ (Wikipedia, fetched). It hosted gaming companies and startups initially, then shifted toward enterprise workloads after the IBM acquisition ✅ (Wikipedia, fetched). IBM's June 2013 announcement framed the acquisition as accelerating "adoption of cloud computing in the enterprise" ✅ (PRNewswire, fetched), and the July close made SoftLayer the substrate of everything that followed: **the IBM Cloud classic infrastructure layer (bare metal and virtual servers, data centers like DAL10 and SNG01) is SoftLayer's direct descendant** ✅.

### 2.3 The Bluemix Era (2014–2017)

**Bluemix** was IBM's PaaS play, built on **Cloud Foundry** — "an implementation of the IBM open cloud architecture, that makes use of Cloud Foundry to enable developers to rapidly build, deploy, and manage their cloud applications" ✅ (IBM announcement archive ENUSAP14-0304, fetched). It ran on SoftLayer infrastructure ✅. The PaaS wave gave IBM the developer-facing platform layer it lacked, and by April 2015 it had grown past 100 services ✅. Two structural facts about this era matter for the modern platform:

- Bluemix **did not win the platform war** — Wikipedia's account (with contemporary press) notes that a year after announcement it remained "substantially behind" Microsoft Azure and AWS ✅ (Wikipedia, fetched). IBM's answer was not to out-hyperscale the hyperscalers but to pivot to the hybrid/enterprise play.
- The era's technical legacy survives: **Cloud Foundry**, **OpenWhisk** (the open-source seed of IBM Cloud Functions), and the service-catalog model all flowed into IBM Cloud ✅.

### 2.4 The Rebrand: October 2017

In **October 2017**, IBM announced it would consolidate everything under the **IBM Cloud** brand, "merging all components, thus retiring the Bluemix and SoftLayer brands" ✅ (Wikipedia, fetched; the IBM announcement it cites was not directly retrieved — see §11). The rebrand was the visible marker of a strategy change already underway: the May 2017 launch of Kubernetes support (IKS) signaled that containers, not Cloud Foundry, would be the platform's future ✅.

### 2.5 The Red Hat Acquisition (2018–2019)

The defining event of IBM Cloud's modern era: **IBM announced its intent to acquire Red Hat for approximately US$34 billion in October 2018** — the largest acquisition in IBM's history — **completed on 9 July 2019 at US$190.00 per share in cash** ✅ (Wikipedia; IBM Investor news, fetched). IBM's stated ambition was to become "the world's #1 hybrid cloud provider" by combining Red Hat's open-source technologies (RHEL, OpenShift) with IBM's cloud infrastructure ✅ (Wikipedia, fetched). At close, IBM reported that its cloud revenue for the trailing 12 months through Q1 2019 had grown to **over US$19 billion** ✅ (IBM Investor news, fetched) — the commercial scale against which the Red Hat bet was made. The consequences are the subject of §4: managed OpenShift on IBM Cloud (August 2019), the Cloud Paks repositioned as OpenShift-native software, and the whole go-to-market rebuilt around hybrid cloud.

### 2.6 The Financial-Services Era (2019–present)

In **November 2019** IBM announced it had "designed the world's first financial services-ready public cloud", with **Bank of America** as the first committed collaborator and anchor customer; **BNP Paribas** followed in 2020 as the first European anchor ✅ (Wikipedia, fetched). General availability of **IBM Cloud for Financial Services** came in **April 2021** ✅. This is the offering that makes IBM Cloud a *banking* story rather than just an enterprise-cloud story — its controls and compliance framework are covered in §6.2.

---

## 3. The Platform Surface

### 3.1 Regions, Multizone Regions, and Data Centers

IBM Cloud's location model has three tiers, documented at cloud.ibm.com (fetched):

- **Multizone region (MZR)** — a region spread across physical locations in multiple zones with independent power, cooling, and network connectivity. MZRs offer low latency (< 2 milliseconds within a zone) and high bandwidth (> 1000 Gbps), with a minimum distance of about 1 mile between zones; IBM's regional services in an MZR generally provide 99.99% (tier 3) availability ✅ (IBM Cloud docs, fetched).
- **Single-campus MZR (SC-MZR)** — three zones within a single building or campus, sharing some dependencies but designed for fault independence; still carries the same SLA ✅ (IBM Cloud docs, fetched).
- **Classic data center** — the physical location for classic infrastructure and Power Virtual Server resources, specified by data-center code (e.g. DAL10, SNG01) rather than zone ✅ (IBM Cloud docs, fetched).

**The current MZR and SC-MZR list** (per the fetched locations documentation):

| Region | Type | Zones |
|---|---|---|
| Dallas (`us-south`) | MZR | us-south-1/2/3 |
| Washington DC (`us-east`) | MZR | us-east-1/2/3 |
| Frankfurt (`eu-de`) | MZR | eu-de-1/2/3 |
| London (`eu-gb`) | MZR | eu-gb-1/2/3 |
| Madrid (`eu-es`) | MZR | eu-es-1/2/3 |
| Sao Paulo (`br-sao`) | MZR | br-sao-1/2/3 |
| Toronto (`ca-tor`) | MZR | ca-tor-1/2/3 |
| Sydney (`au-syd`) | MZR | au-syd-1/2/3 |
| Tokyo (`jp-tok`) | MZR | jp-tok-1/2/3 |
| Chennai (`in-che`) | SC-MZR | in-che-1/2/3 |
| Montreal (`ca-mon`) | SC-MZR | ca-mon-1/2/3 |
| Mumbai (`in-mum`) | SC-MZR | in-mum-1/2/3 |
| Osaka (`jp-osa`) | SC-MZR | jp-osa-1/2/3 |

**The Singapore fact, stated precisely:** Singapore appears in IBM Cloud's current documentation as **Singapore 01 (SNG01), a classic data center** — it is *not* listed among the MZRs or SC-MZRs ✅ (IBM Cloud docs, fetched). The classic-infrastructure and Power Virtual Server location for Singapore is SNG01 ✅. IBM's Singapore cloud presence dates to **March 2011**, when IBM opened its Asia Pacific Cloud Computing Data Centre — a US$38 million investment whose first Singapore IaaS offering launched in April 2011 ✅ (DataCenterKnowledge, fetched); SoftLayer announced its own Singapore expansion in July 2011 ✅ (Wikipedia, fetched). The **exact opening date of the SNG01 facility itself** ⚠ could not be confirmed from primary sources in this pass (see §11), and the "six multizone regions" figure that appears in Wikipedia's IBM Cloud article (citing IBM) is dated — the current locations documentation lists nine MZRs plus four SC-MZRs ✅/⚠.

The consequence for architects in Singapore (developed in §6.4 and §8): **there is no Singapore MZR**, so a bank that wants MZR-grade availability from IBM Cloud in Asia must place workloads in the Sydney or Tokyo MZRs (or the Chennai/Osaka SC-MZRs), or run a hybrid design with the on-premises estate and SNG01 as the local footprint.

The **zone-naming model** matters because it appears in every IAM policy, subnet, and Terraform plan: regional services address zones as region-scoped IDs (`us-south-1` … `us-south-3`, the form used in the table above), while the VPC/classic documentation also maps zones onto data-center campuses with names like `us-south-dal10-a` ✅/⚠ (both forms appear in the fetched docs — the §3.1 table and §3.6's VPC bullet; the campus-to-zone mapping table itself was not re-fetched in this expansion pass ⚠).

**Reading the MZR table for a regulated bank** — the Cymbal Bank consequence is drawn in §8; the decision shape is:

| Requirement | Region choice | Reasoning |
|---|---|---|
| MZR-grade regional services for the European book | Frankfurt (`eu-de`) | EU data-residency anchor; three-zone MZR ✅ (§3.1) |
| MZR-grade regional services for Asia | Sydney (`au-syd`) or Tokyo (`jp-tok`) | Nearest MZRs to Singapore ✅ (§3.1); the latency/residency trade-off between them is the bank's design decision ⚠ |
| Singapore-local, residency-sensitive placement | SNG01 classic data center + on-premises estate | The only Singapore location in the current documentation ✅ (§3.1) |
| Cost-optimized secondary placement | An SC-MZR (Chennai `in-che`, Osaka `jp-osa`, Mumbai `in-mum`, Montreal `ca-mon`) | Same SLA class, single-campus topology ✅ (§3.1) |

The **SC-MZR caveat** deserves emphasis: a single-campus MZR delivers the same SLA class but shares one physical campus across its three zones — a disaster-recovery plan should treat an SC-MZR as a same-campus topology and pair it with a genuinely separate region or the on-premises estate ⚠ (architectural reading of the fetched MZR definitions, not an IBM statement).

### 3.2 Compute: Virtual Servers, Bare Metal, Serverless

- **Virtual servers** — provisioned either on the classic infrastructure (the SoftLayer lineage) or on VPC ("Virtual Server for VPC"), the modern virtual networking architecture ✅ (ibm.com/cloud product catalog + Wikipedia, fetched).
- **Bare metal** — IBM Cloud Bare Metal Servers, "dedicated hardware for maximum performance" ✅ (ibm.com/cloud, fetched). Bare metal is the platform's differentiator of record: SoftLayer sold bare metal before most large providers, and IBM Cloud continued to be considered a bare-metal leader (Wikipedia's account mentions over 11 million possible custom configurations with Power, Intel, and AMD CPUs and Nvidia GPUs — ⚠ vendor/market figure, not independently audited).
- **Serverless** — IBM Cloud Code Engine, "serverless container hosting platform as a service" ✅ (ibm.com/cloud, fetched), plus IBM Cloud Functions (the OpenWhisk lineage, §2.3).
- **GPUs** — GPU worker nodes on OpenShift clusters (Nvidia V100, L40s, and H100) ✅ (ibm.com/products/openshift, fetched).
- **Specialized** — IBM Hyper Protect Virtual Servers (sensitive-data workloads) and IBM Cloud for VMware Solutions ✅ (ibm.com/cloud, fetched).

The compute catalog as a table (rows consolidate the verified §3.2–§3.4 facts; the "fit" column is this guide's architectural reading ⚠):

| Option | What it is | Fit for a bank estate | Status |
|---|---|---|---|
| **Virtual Server for VPC** | Modern virtual compute on the VPC network (regions, zones, security groups) | Stateless/microservice workloads, web/API tiers | ✅ (§3.2) |
| **Virtual Server (classic)** | SoftLayer-lineage virtual compute on classic infrastructure | Workloads already on classic; the migration bridge | ✅ (§3.2) |
| **Bare Metal Server** | Dedicated single-tenant hardware (Intel/AMD/Power, GPUs) | Performance-, licensing-, or compliance-driven single-tenancy | ✅ (§3.2); the "11M configurations" figure ⚠ vendor |
| **Power Systems Virtual Server** | Virtual Power servers: AIX, IBM i, Linux; SAP and Oracle | The AS/400-lineage estate in the cloud — [technology/ibm_as400_guide.md](technology/ibm_as400_guide.md) | ✅ (§3.3) |
| **IBM Z / LinuxONE** | Mainframe-class estate: z/OS, Linux, z/TPF, z/VM; Telum on-chip AI | The core — stays on-premises in the §8 pattern; as-a-service ⚠ | ✅ hardware/OS; ⚠ as-a-service (§3.4) |
| **Code Engine** | Serverless container hosting (PaaS-style) | Bursty/event-driven workloads, CI/CD helpers | ✅ (§3.2) |
| **Cloud Functions** | FaaS (OpenWhisk lineage) | Lightweight event handlers | ✅ (§2.3, §3.2) |
| **Hyper Protect Virtual Servers** | Locked-down VMs for sensitive data | Key management, confidential workloads | ✅ (§3.2, catalog) |
| **VMware Solutions** | Managed VMware on IBM Cloud | VMware estate extension or DR target | ✅ (§3.2, catalog) |
| **GPU worker nodes (OpenShift)** | Nvidia V100/L40s/H100 attached to OpenShift clusters | AI inferencing/fine-tuning on the common plane | ✅ (§4.2) |

The placement discipline the table encodes: **each estate tier has a natural compute home** — channels on VSI/Code Engine, IBM-lineage workloads on PowerVS, the core on Z, AI on GPU-enabled OpenShift — and the VPC/classic split is a *network-generation* choice, not a capability ranking ✅/⚠ (the zone model is documented, §3.1/§3.6; the tier-to-home mapping is this guide's).

### 3.3 Power Systems Virtual Server (PowerVS)

**IBM Power Virtual Server** is "a family of configurable, multitenant, virtual IBM Power servers with access to IBM Cloud services, web, DevOps and IBM Db2" ✅ (ibm.com/products/power-virtual-server, fetched). It runs the Power operating systems — **AIX, IBM i, and Linux** — which makes it the cloud home for the AS/400-lineage estate documented in [technology/ibm_as400_guide.md](technology/ibm_as400_guide.md): an IBM i workload can move from on-premises Power hardware to PowerVS without changing the platform ✅/⚠ (the "same experience as on premises" is IBM's claim; the mechanics are covered by IBM Redbooks for AIX, IBM i, and Linux on PowerVS, referenced on the product page ✅). PowerVS also carries **SAP** (RISE with SAP) and **Oracle Database** workloads ✅ (product page, fetched). It connects to "over 250 IBM Cloud services" ✅ (product page, fetched) and integrates with watsonx for AI ✅.

### 3.4 IBM Z and LinuxONE: The Mainframe in the Cloud

**IBM Z** is the mainframe line — "a family of modern infrastructure powered by the IBM Telum processor that runs enterprise operating systems and IBM Z software" ✅ (ibm.com/z, fetched). Current models: **IBM z17** and **IBM z16**; the Telum processor includes **on-chip AI accelerators** for real-time inferencing (fraud detection is the canonical use) ✅ (ibm.com/z, fetched). The operating systems: **z/OS, Linux, z/TPF, and z/VM** ✅ (ibm.com/z, fetched).

**IBM LinuxONE** is the Linux-only sibling: LinuxONE 5 (Telum II) and LinuxONE 4 (Telum), positioned for enterprise Linux workloads with confidential containers and AI inferencing ✅ (ibm.com/products/linuxone, fetched). Banking case studies on LinuxONE include Bank Zero and Sagicor Bank Jamaica ✅ (IBM case studies, referenced from the product page).

**IBM Z / LinuxONE as-a-service:** the notion of consuming mainframe capacity as a cloud service (z/OS on demand, "the mainframe on the cloud") is a real and recurring IBM theme — IBM Z's own site lists "Hybrid cloud: Integrate IBM Z into your hybrid cloud" ✅ (ibm.com/z, fetched) — but a dedicated, current **"IBM Z as a Service" / "LinuxONE as a Service" product page could not be confirmed from primary sources in this pass** ⚠ (the natural URL redirected to the generic product catalog; see §11). What *is* confirmed: **watsonx Code Assistant for Z** — generative AI for mainframe application understanding, refactoring, and **COBOL-to-Java modernization** ✅ (ibm.com/products/watsonx-code-assistant-z, referenced from multiple fetched pages), and the z/OS-facing modernization tooling (Git-based SCM modernization at ANZ, DevOps at Swedbank — ✅ IBM case studies, referenced from ibm.com/z).

### 3.5 Storage

The catalog covers the standard trio — **Object Storage, Block Storage, and File Storage** ✅ (Wikipedia's IBM Cloud product list, fetched; ibm.com/cloud also lists storage among its categories). Block and File Storage are offered for VPC (tiered performance) alongside classic storage; Object Storage (COS) is the S3-style archive/analytics tier. ⚠ Service-level specifics (tiers, IOPS classes, replication options) were not verified from primary documentation in this pass — treat the detail level here as catalog-level only.

The storage picture for a hybrid bank estate, in architecture terms:

| Tier | Offering | Role in the hybrid estate | Status |
|---|---|---|---|
| **Block storage** | Block Storage for VPC (tiered performance); classic block equivalents | VSI/OpenShift node disks, database volumes (Db2 on VSI), low-latency transactional IO | ✅ catalog; ⚠ tier/IOPS specifics not fetched (§11 #10) |
| **File storage** | File Storage for VPC; classic file equivalents | Shared filesystems, application clusters, tooling homes | ✅ catalog; ⚠ specifics not fetched |
| **Object storage** | IBM Cloud Object Storage (COS) | The S3-style tier: backups, archives, analytics lakes; the natural landing zone for watsonx.data object-store data ✅ (§5.1) | ✅ catalog; ⚠ bucket-class/replication detail not fetched |
| **Backup & archive** | Backup services and the object tier for retention ⚠ | Compliance retention (MAS record-keeping), DR copies, mainframe unload/offload landing zones | ⚠ not re-verified in this pass (catalog-level) |

Design notes that follow from the verified facts (not new claims):

- **The tiering discipline is the compliance story**: block for transactional, object for retention — the record-keeping requirements of the banking series ([banking/payment_rails_guide.md](banking/payment_rails_guide.md), [banking/swiftnet_fileact_guide.md](banking/swiftnet_fileact_guide.md)) map naturally onto an object tier with lifecycle policies ⚠ (lifecycle-policy support is standard object-storage behavior; not re-verified for COS in this pass).
- **The MQ/Db2 backup pattern**: queue-manager data and database backups are classic block/object placements; a cross-region copy of the object tier is the DR mechanism ⚠ (replication options not re-verified — §11 #10).

### 3.6 Networking

- **VPC (Virtual Private Cloud)** — IBM Cloud's virtual networking architecture: regions, zones, subnets, security groups, and the zone-mapping model (an account's zones map to physical data centers via universal zone names like `us-south-dal10-a`) ✅ (IBM Cloud docs, fetched). The current VPC generation is commonly called **"VPC Gen 2"** in IBM materials ⚠ (the "Gen 2" branding is widely used in IBM Cloud docs/blogs but was not directly confirmed on a fetched page in this pass).
- **Load balancers** — network and application load balancers are part of the VPC service set ✅/⚠ (catalog-level; per-SKU detail not fetched).
- **Direct Link** — private, dedicated connectivity from on-premises to IBM Cloud, bypassing the public internet. ⚠ The Direct Link documentation page could not be fetched in this pass (the docs site returned a JavaScript-rendered empty page); the service's existence and purpose are corroborated by IBM's data-center and locations documentation and the product catalog ✅/⚠.
- **Transit Gateway** — hub-and-spoke interconnection between VPCs and classic networks. ⚠ Same caveat: not directly fetched; catalog-level corroboration only.
- **Satellite** — IBM Cloud Satellite "take[s] the cloud anywhere — on premises, multicloud and to the Edge" ✅ (ibm.com/cloud, fetched): IBM Cloud services running on customer-controlled infrastructure outside IBM's data centers.

The networking catalog as a table (rows consolidate the verified §3.6 bullets; new interpretation flagged ⚠):

| Service | Function | Hybrid role | Status |
|---|---|---|---|
| **VPC** | Virtual networking: regions, zones, subnets, security groups | The substrate for cloud-native and OpenShift workloads | ✅ (§3.6); "Gen 2" label ⚠ |
| **Direct Link** | Private, dedicated on-premises ↔ cloud connectivity | The z/OS-core ↔ cloud pipe; MQ traffic off the public internet | ✅/⚠ (§3.6 — docs page not fetched) |
| **Transit Gateway** | Hub-and-spoke interconnection between VPCs and classic networks | The routing hub unifying on-prem, SNG01 classic, and MZR VPCs | ✅/⚠ (§3.6) |
| **Load balancers** | Network (NLB) and application (ALB) balancers in VPC | Traffic entry to the OpenShift/API tier | ✅/⚠ (§3.6, catalog-level) |
| **Satellite** | IBM Cloud services on customer-controlled infrastructure | The "cloud anywhere" extension to on-prem/edge | ✅ (§3.6) |

The **hybrid connectivity pattern** (architecture practice, not a new IBM claim): Direct Link terminates the on-premises estate — including the mainframe site — into IBM Cloud; Transit Gateway fans that private connectivity out to VPCs and classic networks; the OpenShift clusters and their MQ endpoints sit inside VPCs on the private network. The result is one privately addressable domain spanning the z/OS core, the SNG01 classic footprint, and the Frankfurt/Sydney MZR workloads — exactly the topology the §8 worked example assumes ⚠ (this guide's synthesis of the verified service functions).

### 3.7 The Platform-Surface Summary Table

| Layer | Offering | Status |
|---|---|---|
| Regions | 9 MZRs + 4 SC-MZRs (Dallas, Washington DC, Frankfurt, London, Madrid, Sao Paulo, Toronto, Sydney, Tokyo; Chennai, Montreal, Mumbai, Osaka) | ✅ (IBM Cloud docs, fetched) |
| Singapore | SNG01 classic data center; not an MZR | ✅ (IBM Cloud docs, fetched); launch date ⚠ |
| Compute | Virtual servers (VPC + classic), bare metal, GPU, Code Engine, Hyper Protect, VMware Solutions | ✅ (catalog, fetched) |
| Power | Power Systems Virtual Server (AIX / IBM i / Linux, SAP, Oracle) | ✅ (product page, fetched) |
| Mainframe | IBM Z (z17/z16, Telum, z/OS), LinuxONE (4/5); as-a-service offering ⚠ | ✅ hardware/OS; ⚠ as-a-service |
| Storage | Object, Block, File | ✅ (catalog); ⚠ detail level |
| Networking | VPC, load balancers, Direct Link, Transit Gateway, Satellite | ✅/⚠ (see §3.6) |

---

## 4. The Red Hat Integration

### 4.1 OpenShift: The Container Platform

**Red Hat OpenShift** is Red Hat's enterprise Kubernetes platform — "Build, modernize, and deploy apps at scale" ✅ (redhat.com, fetched). It adds to upstream Kubernetes the operational layer enterprises need: built-in registry, routing, CI/CD, security policy, and a consistent operator model across environments ✅/⚠ (the feature list is standard OpenShift knowledge; the fetched redhat.com page confirms the platform's positioning but not each feature — treat the feature specifics as ⚠-level).

### 4.2 Red Hat OpenShift on IBM Cloud

**Red Hat OpenShift on IBM Cloud** is the fully managed OpenShift service on IBM Cloud ✅ (ibm.com/products/openshift, fetched). Verified facts from the product page:

- **Fully managed** with automated provisioning, patching, and lifecycle management ✅.
- **99.99% SLA** on regional high-availability clusters ✅.
- Integrated with IBM Cloud **IAM, Schematics (infrastructure as code), and Observability** ✅.
- **GPU worker nodes** — Nvidia V100, L40s, and H100 — for AI ✅.
- **Deployable architectures** that create clusters meeting "hundreds of controls by default" — the regulated-industry story ✅.
- **OpenShift entitlements** from IBM software licenses can be applied to clusters ✅.
- **Red Hat OpenShift Virtualization Service on IBM Cloud** (announced 2026) runs VMs on OpenShift via KubeVirt — the VMware-replacement play ✅ (newsroom release + product page, fetched).
- Launched August 2019, ~3 weeks after the Red Hat close ✅ (Wikipedia, fetched).

The managed OpenShift offering is IBM's analogue of AWS's ROSA (Red Hat OpenShift Service on AWS) — ⚠ the ROSA comparison is standard industry knowledge, not fetched in this pass.

The managed-vs-self-managed decision, in the shape a bank faces it:

| Dimension | Red Hat OpenShift on IBM Cloud (managed) | OpenShift Container Platform (self-managed) |
|---|---|---|
| Operations | IBM handles provisioning, patching, lifecycle ✅ (§4.2) | The bank's platform team operates everything ⚠ (standard OpenShift knowledge) |
| Placement | IBM Cloud regions (the §3.1 MZR/SC-MZR footprint) | Anywhere — the common-plane promise ✅ (§1.2) |
| Regulated-workload story | Deployable architectures meeting "hundreds of controls by default" ✅ (§4.2); FS-ready context ✅ (§6.2) | Bank-owned compliance evidence ⚠ (design consequence) |
| AI workloads | GPU worker nodes (V100/L40s/H100) ✅ (§4.2) | Bring-your-own GPU infrastructure ⚠ |
| Role in §8 | The cloud side of the hybrid house | The on-prem side beside the z/OS core |

The strategic point: the *same* cluster definitions, operators, and CI/CD pipelines run on both sides — the mechanism that makes the §8 exit plan ("redeploy on-prem") a technical possibility rather than a migration fantasy ✅/⚠ (design intent of the common plane; an exercised exit remains a program — §6.3, §11 #13).

### 4.3 The Cloud Paks

**IBM Cloud Paks** are "AI-powered software designed to accelerate application modernization with pre-integrated data, automation and security capabilities... delivered on Red Hat OpenShift" ✅ (ibm.com/cloud-paks, fetched). The set currently listed on the Cloud Paks page (fetched):

| Cloud Pak | Purpose (from the fetched page) |
|---|---|
| **Cloud Pak for Data** | Unify and simplify the collection, organization and analysis of data |
| **Cloud Pak for Business Automation** | Automate business operations to achieve better performance |
| **Cloud Pak for AIOps** | Automate IT operations to deliver actionable insights |
| **Cloud Pak for Integration** | Connect apps and data with an AI-powered, cloud-agnostic integration solution |
| **Cloud Pak for Network Automation** | Automate networks to deliver zero-touch operations |
| **Cloud Pak for Applications** | Deliver an application landscape that transforms with your business |
| (plus) **Cloud Pak System** | Integrated system for private-cloud deployment of VMs and containers |

⚠ **The set has changed over time** — this is a flag area by design:

- **Cloud Pak for Security** is *not* on the current Cloud Paks page (it appears in the page's imagery but not in the product list); it was a headline Cloud Pak at introduction (2019) ⚠ — its current status/placement could not be confirmed from a fetched primary source.
- Earlier Cloud Paks that have since been renamed or retired (e.g. **Cloud Pak for Watson AIOps** → Cloud Pak for AIOps; **Cloud Pak for Automation** → Cloud Pak for Business Automation; **Cloud Pak for Multicloud Management**, which was discontinued) ⚠ — the rename/retirement history is well known in the industry but was not confirmed from a fetched primary source in this pass.

The Cloud Paks' architectural point: they are **containerized middleware** — the classic IBM software stack (integration, data, automation) repackaged to run on OpenShift anywhere, which is what makes them the software layer of the hybrid story (§1.1).

**What each Cloud Pak bundles** — the *component mapping* below is industry-standard knowledge about the product set and was **not re-verified from fetched pages in this expansion pass**; treat every cell as ⚠:

| Cloud Pak | Bundled software (⚠ not re-verified) | Lineage/role |
|---|---|---|
| **Cloud Pak for Integration** | The integration stack — MQ, App Connect, Event Streams (Kafka), API Connect, high-speed transfer ⚠ | The containerized home of the §5.4 integration trio; the glossary's "API/MQ/event integration stack" ✅ (§12) |
| **Cloud Pak for Data** | The data stack — Db2 Warehouse, data-integration tooling, and the watsonx data/AI services on a common data fabric ⚠ | The data-platform Cloud Pak; the deployment home of the watsonx.data lakehouse ⚠ |
| **Cloud Pak for Business Automation** | Workflow/BPM, decisions/rules, document processing (FileNet lineage), RPA ⚠ | The automation Cloud Pak (renamed from Cloud Pak for Automation — ⚠ §4.3) |
| **Cloud Pak for AIOps** | AI-driven IT-operations analytics (Watson AIOps lineage) ⚠ | The IT-ops Cloud Pak (renamed from Cloud Pak for Watson AIOps — ⚠ §4.3) |
| **Cloud Pak for Network Automation** | Telecom network automation — zero-touch operations ⚠ | The CSP/telecom Cloud Pak |
| **Cloud Pak for Applications** | Application modernization runtime (WebSphere lineage, Java) ⚠ | The apps Cloud Pak; the WebSphere-to-OpenShift path |
| **Cloud Pak for Security** | Security operations — threat intelligence, SOAR ⚠ | Not on the current Cloud Paks page ⚠ (§4.3, §11 #5) |
| **Cloud Pak System** | Integrated system for private-cloud deployment of VMs and containers ✅ (re-confirmed on the fetched page in this expansion pass) | The on-prem integrated system; the page positions it as the enterprise private-cloud path |

The architectural consequence for the series: because the Cloud Paks are OpenShift-native, the *software layer* of the hybrid house is as portable as the *platform layer* — a bank's integration (Cloud Pak for Integration) and data (Cloud Pak for Data) estates can run on-prem on OpenShift, on IBM Cloud, or on another certified cloud with the same operators ✅ (OpenShift-native design is the verified Cloud Paks premise — §4.3) / ⚠ (per-Cloud-Pak component detail as flagged above).

### 4.4 The Hybrid-Cloud Architecture Patterns

The Red Hat integration defines the four canonical hybrid patterns this series cares about:

1. **OpenShift as the common plane** — the same OpenShift platform on IBM Cloud, on-premises, and on other clouds; applications move between them without re-platforming ✅ (redhat.com + ibm.com positioning, fetched).
2. **Satellite extension** — IBM Cloud services delivered to on-premises locations via Satellite ✅ (ibm.com/cloud, fetched).
3. **The z/OS offload pattern** — new digital workloads (APIs, channels, AI) run on OpenShift in the cloud while the z/OS core (CICS/COBOL) stays on the mainframe; integration flows over MQ; COBOL-to-Java modernization is assisted by watsonx Code Assistant for Z ✅ (ibm.com/z + watsonx materials, fetched; the *pattern* itself is architecture practice, developed in §8).
4. **Event-driven extension** — Kafka-based Event Streams in the cloud consuming mainframe/Power events, with MQ↔Kafka connectors bridging the transactional and event worlds ✅ (ibm.com/products/mq and /event-streams, fetched).

Two further patterns complete the set for the banking series:

5. **The Satellite data-gravity pattern** — for workloads that must stay close to the data or the compliance boundary (customer data, low-latency local processing), IBM Cloud services run on Satellite-managed infrastructure at the bank's own site: the *service* is cloud-managed while the *data* never leaves the premises ✅/⚠ (Satellite's "cloud anywhere" positioning is verified — §3.6; the specific pattern is this guide's synthesis).
6. **The cloud-as-DR pattern** — the on-premises OpenShift estate (and, via MQ's cross-region replication, the messaging layer) treats a cloud MZR as the disaster-recovery target: same platform, replicated queues, database-level replication ⚠ (MQ cross-region replication is verified — §5.4; the full DR topology is architecture practice, not an IBM offering).

---

## 5. The Services: Data, AI, and Integration

### 5.1 The watsonx Portfolio

**IBM watsonx** is IBM's AI portfolio — "a portfolio of AI products that accelerates the impact of generative AI in core workflows to drive productivity" ✅ (ibm.com/watsonx, fetched). The verified components:

- **watsonx.ai** — the AI studio: "train, validate, tune and deploy AI models" ✅ (product catalog + watsonx page, fetched).
- **watsonx.data** — the hybrid, open **data lakehouse** "unifying data access and automating governance" ✅ (product catalog, fetched).
- **watsonx.governance** — "govern AI across the lifecycle by providing visibility, control, and accountability across any AI, anywhere" ✅ (product catalog, fetched) — the piece that matters most to banks (see the AI-governance cross-references below).
- **watsonx Orchestrate** — AI agents and assistants for workflow automation ✅ (watsonx page, fetched).
- **watsonx BI**, **watsonx.data intelligence**, **watsonx.data integration**, **watsonx Assistant**, and **watsonx Code Assistant for Z** (mainframe COBOL modernization) ✅ (watsonx page + product catalog, fetched).

**AI-governance depth is deliberately NOT re-derived here.** The governance, risk, and compliance treatment of AI in banking — model risk management, explainability, the FEAT-style fairness/ethics considerations, RAG evaluation, prompt-injection defenses — lives in the sibling guides: [banking/ai_genai_banking_compliance_guide.md](banking/ai_genai_banking_compliance_guide.md) (AI-in-banking compliance), [technology/ai_llm/enterprise_ai_platforms_guide.md](technology/ai_llm/enterprise_ai_platforms_guide.md) (the platform comparison), and the [technology/ai_llm/rag/](technology/ai_llm/rag/) subdirectory (RAG architecture and evaluation). This guide's contribution is the *platform* fact: watsonx.governance is IBM's answer to AI-governance-as-a-product, and it is explicitly positioned for regulated industries (the Financial Services page pairs watsonx.ai + watsonx.governance with "responsible, transparent and explainable AI" ✅ (ibm.com/cloud/financial-services, fetched)).

The portfolio as a table (rows consolidate the verified bullets above; the "bank use" column is the §8 mapping ⚠ illustrative):

| Component | Purpose (verified) | Bank use in §8 |
|---|---|---|
| **watsonx.ai** | Train, validate, tune, deploy AI models ✅ | The studio where Cymbal Bank's Granite-based models are built and served (§8.2 layer 5) |
| **watsonx.data** | Hybrid open data lakehouse; unified data access + automated governance ✅ | The lakehouse for the AI tier — AML alert enrichment, trade-document extraction (§8.2 layer 4) |
| **watsonx.governance** | Govern AI across the lifecycle: visibility, control, accountability ✅ | The model-registry/monitoring overlay for every AI workload (§8.2 layer 5) |
| **watsonx Orchestrate** | AI agents and assistants for workflow automation ✅ | Assistants for ops/trade workflows ⚠ (illustrative) |
| **watsonx Code Assistant for Z** | Generative AI for mainframe app understanding and COBOL-to-Java ✅ | The refactoring assistant for the §8.2 offload pattern (§3.4, §6.1) |
| **watsonx Assistant / BI / data intelligence / data integration** | Conversational AI, BI, and data intelligence/integration services ✅ | Channel assistants; BI on the lakehouse ⚠ (illustrative) |

### 5.2 Granite: The Open Model Family

**IBM Granite** is "a family of open, trusted AI models for business... released under an Apache 2.0 license" ✅ (ibm.com/granite, fetched). Verified facts:

- Current generation **Granite 4.2** with language models at 3B, 8B, and 30B parameter sizes ✅ (benchmark table on the fetched page).
- Model families: **Granite Language, Granite Speech, Granite Vision, Granite Guardian** (guardrails), **Granite Embedding**, and **Granite Time Series** ✅.
- Distributed via **Hugging Face** (ibm-granite), Ollama, watsonx.ai, and others ✅.
- Trust story: models **cryptographically signed** (as of April 2026) and the Granite AI Management System is **ISO-certified** ✅ (footnotes on the fetched page).
- Designed for enterprise deployment flexibility — including on-prem, which matters for banks that cannot send data to public clouds ✅ (product positioning, fetched).

The Granite family as a table (verified families from the fetched page; the "bank relevance" column is this guide's mapping ⚠ illustrative):

| Family | Role | Bank relevance |
|---|---|---|
| **Granite Language** (3B/8B/30B in Granite 4.2) | General language modeling ✅ | Document extraction, drafting, classification |
| **Granite Speech** | Speech processing ✅ | Voice-channel assist ⚠ (illustrative) |
| **Granite Vision** | Image understanding ✅ | Trade-document/cheque image processing ⚠ (illustrative) |
| **Granite Guardian** | Guardrails for model input/output ✅ | The safety layer the AI-compliance guides require ([banking/ai_genai_banking_compliance_guide.md](banking/ai_genai_banking_compliance_guide.md)) |
| **Granite Embedding** | Embeddings for retrieval ✅ | RAG — see [technology/ai_llm/rag/](technology/ai_llm/rag/) |
| **Granite Time Series** | Time-series forecasting ✅ | Treasury/balance forecasting ⚠ (illustrative) |

Deployment note (verified positioning): the models are Apache-2.0 and run on-prem as well as on watsonx.ai ✅ (§5.2) — for a bank constrained on data egress, that on-prem capability is the difference between "AI is possible" and "AI is compliant" ✅/⚠ (the on-prem deployment fact is IBM's; the compliance framing is this guide's).

### 5.3 Data Services

- **IBM Db2** — "the database to run your mission-critical workloads... across any cloud, hybrid or on-prem" ✅ (product catalog, fetched). The Db2 family spans Db2 on z/OS (the mainframe database), Db2 for i (the AS/400-lineage database — see [technology/ibm_as400_guide.md](technology/ibm_as400_guide.md)), Db2 LUW, and the managed cloud forms.
- **Db2 Warehouse** — "managed data warehouse on IBM Cloud or AWS" ✅ (ibm.com/cloud catalog, fetched) — notable for being deployable on a competitor's cloud.
- **IBM Cloudant** — the managed JSON document database **based on open-source Apache CouchDB** with compatible API and replication protocols, 99.99% SLA, distributed across availability zones and 6 regions ✅ (ibm.com/products/cloudant, fetched). The CouchDB-compatible replication is the hybrid hook: Cloudant ↔ CouchDB replication lets a bank run the same data tier on-prem and in the cloud ✅.
- Managed relational/NoSQL catalog (PostgreSQL, MongoDB, Redis, etc.) ✅ (Wikipedia product list, fetched; per-engine detail not fetched ⚠).

The data catalog as a table (rows consolidate §5.3 verified facts):

| Service | Type | Verified facts | Hybrid role |
|---|---|---|---|
| **Db2** | Relational family (z/OS, i, LUW, Warehouse) ✅ | "Mission-critical workloads... across any cloud, hybrid or on-prem" ✅ | The ledger tier; Db2 for z/OS stays with the core (§8.2 layer 4) |
| **Db2 Warehouse** | Managed warehouse ✅ | Deployable on IBM Cloud **or AWS** ✅ | Analytics/mart workloads; the cross-cloud data point |
| **Cloudant** | JSON document (CouchDB-based) ✅ | 99.99% SLA; distributed across 6 regions; CouchDB-compatible replication ✅ | Channel state with on-prem CouchDB replication (§8.2 layer 4) |
| **Managed PostgreSQL / MongoDB / Redis et al.** | Managed open-source engines ✅ (catalog) | Per-engine detail ⚠ (§5.3) | App-tier databases for channel microservices ⚠ |

### 5.4 Integration Services

The integration trio is where IBM Cloud meets the messaging reality documented across this repository's payment and SWIFT guides ([banking/payment_rails_guide.md](banking/payment_rails_guide.md), [banking/swift_alliance_access_guide.md](banking/swift_alliance_access_guide.md), [banking/swiftnet_fileact_guide.md](banking/swiftnet_fileact_guide.md)):

- **IBM MQ** — the enterprise message broker, "designed to never lose a message with secure, reliable, exactly-once message delivery across hybrid and multi-cloud environments" ✅ (ibm.com/products/mq, fetched). Verified specifics: **MQ for z/OS** (the mainframe edition), MQ Appliance, MQ SaaS, **managed file transfer** (the MFT capability the SWIFT file-act guides discuss), native HA and **cross-region replication**, and **supported MQ↔Kafka connectors** (source and sink, exactly-once) ✅. Banking is a headline use case: "process real-time or batch transactions with consistent data... assured delivery... complete data for fraud detection" ✅ (product page, fetched). MQ 10.0 is the current release line ✅ (product page, fetched).
- **IBM App Connect** — the hybrid integration platform: 200+ prebuilt connectors, API-led and event-led integration, and connectivity "across enterprise applications, cloud services, databases, APIs, AI services, mainframe systems, IBM MQ and Apache Kafka" ✅ (ibm.com/products/app-connect, fetched). The Karnataka Bank case study (API-led digital banking modernization) is directly relevant to §8 ✅ (referenced from the product page).
- **IBM Event Streams** — the enterprise Kafka platform: fully managed on IBM Cloud (or on-prem via Event Automation / Cloud Pak for Integration), 99.99% availability, deployed across 3 zones in **10 multizone regions**, Schema Registry, Kafka Connect (50+ connectors), Mirror Maker 2 for DR, and an **Enterprise plan that is IBM Financial Services Validated** with PCI-DSS, SOC 2 Type 2, ISO 27001/27017, and GDPR alignment ✅ (ibm.com/products/event-streams, fetched).
- **API Connect** — API lifecycle management ("create, manage and secure APIs... with strong governance") ✅ (product catalog, fetched).

The integration trio as a table (consolidating §5.4):

| Service | Role | Verified facts | Series cross-reference |
|---|---|---|---|
| **IBM MQ** | Transactional message broker | "Never lose a message"; exactly-once; MQ for z/OS; MQ Appliance; MQ SaaS; managed file transfer; HA + cross-region replication; MQ↔Kafka connectors ✅ | [banking/payment_rails_guide.md](banking/payment_rails_guide.md), [banking/swift_alliance_access_guide.md](banking/swift_alliance_access_guide.md), [banking/swiftnet_fileact_guide.md](banking/swiftnet_fileact_guide.md) |
| **App Connect** | Hybrid integration platform | 200+ connectors; API-led and event-led; mainframe/MQ/Kafka connectivity ✅ | The API/format mediation layer of §8.2 layer 3 |
| **Event Streams** | Enterprise Kafka | 99.99%; 3-zone deployment; 10 multizone regions; Schema Registry; Kafka Connect (50+); Mirror Maker 2; FS-validated Enterprise plan ✅ | The event side of §8.2 layer 3; payment-event feeds |
| **API Connect** | API lifecycle management | Create, manage, secure APIs with strong governance ✅ | The API gateway tier of §8.2 layer 1 ⚠ (placement choice) |

### 5.5 DevOps and Platform Services

- **Schematics** — infrastructure as code / Terraform-based provisioning ✅ (referenced on the OpenShift product page, fetched).
- **Code Engine** — serverless container hosting ✅ (ibm.com/cloud, fetched).
- **Container Registry** — private image registry ✅ (ibm.com/cloud, fetched).
- **Observability** (IBM Cloud Monitoring / Logs) and **Security & Compliance Center** — referenced in the FS-ready context ✅ (ibm.com/cloud/financial-services + OpenShift page, fetched).

The DevOps/automation surface as a table (consolidating §5.5; the rows marked ⚠ were not re-fetched in this expansion pass):

| Tool | Role | Status |
|---|---|---|
| **Schematics** | Terraform-based infrastructure as code (workspaces, templates) | ✅ (§5.5) |
| **Code Engine** | Serverless container hosting | ✅ (§5.5) |
| **Container Registry** | Private image registry for the container estate | ✅ (§5.5) |
| **Observability** (Monitoring/Logs) | Metrics and logs for cloud + OpenShift | ✅ (§5.5) |
| **Security & Compliance Center** | Continuous control evidence; the FS-ready reporting surface | ✅ (§5.5, §6.2) |
| **ibmcloud CLI + Terraform provider** | The two automation entry points; Schematics runs Terraform natively | ⚠ not re-verified in this pass (catalog-level knowledge) |
| **Continuous Delivery / Toolchains** | CI/CD pipelines and toolchain automation | ⚠ not re-verified in this pass (service existence is catalog-level; pipeline detail not fetched) |

The house-style note for the CI/CD rows: the *pattern* — Git → build (Container Registry) → deploy (OpenShift/Kubernetes) with IaC (Schematics/Terraform) and evidence (Security & Compliance Center) — follows from the verified rows ✅; the specific Continuous Delivery service capabilities are ⚠ as flagged.

---

## 6. The Banking Angle

### 6.1 The Mainframe Estate: CICS, z/OS, COBOL

The banking angle starts with the estate IBM Cloud is designed to serve. Banks run core systems on **IBM Z with z/OS, CICS (the transaction server), and COBOL** — the same platform lineage family as the AS/400 documented in [technology/ibm_as400_guide.md](technology/ibm_as400_guide.md), one tier up: where the AS/400 (now IBM i on Power) is the midrange core, **z/OS is the enterprise mainframe core** ✅/⚠ (the CICS/z/OS/COBOL dominance in banking is industry consensus; IBM's own materials confirm the platform's banking position — e.g. IBM Z case studies with ANZ, Swedbank, and Broadridge, referenced from ibm.com/z ✅).

Verified anchors for the banking-mainframe story:

- **Telum on-chip AI inference** — IBM positions the Z AI accelerators for "near real-time AI inferencing to help identify fraud" ✅ (ibm.com/z, fetched).
- **watsonx Code Assistant for Z** — generative AI for mainframe app understanding and COBOL-to-Java transformation ✅ (product catalog + watsonx page, fetched) — the tooling behind the "modernize the core without rewriting it blindly" pattern.
- **ANZ case study** — Git-based SCM modernization for core-banking development (90% licensing-cost reduction, 60% less manual ops effort per IBM's figures — ⚠ vendor-reported numbers) ✅/⚠ (ibm.com/z case study, referenced).
- **Swedbank case study** — DevOps retooling on the mainframe (15% time-to-market reduction forecast — ⚠ vendor-reported) ✅/⚠.
- The "$10–12 trillion/day of U.S. Treasury flows secured on IBM Z" figure on ibm.com/z is a striking marketing stat — ⚠ vendor-sourced, not independently audited.

The verified anchors as a table:

| Anchor | Claim | Status |
|---|---|---|
| Telum on-chip AI inference | On-chip accelerators for near-real-time fraud inferencing | ✅ (ibm.com/z, fetched) |
| watsonx Code Assistant for Z | Gen-AI for mainframe app understanding and COBOL-to-Java | ✅ (§3.4, §5.1) |
| ANZ case study | Git-based SCM modernization; 90% licensing-cost reduction, 60% less manual ops | ✅/⚠ vendor-reported figures |
| Swedbank case study | DevOps retooling; 15% time-to-market reduction forecast | ✅/⚠ vendor-reported |
| US Treasury flows | "$10–12 trillion/day" secured on IBM Z | ⚠ vendor-sourced stat |

The estate-frame (architecture practice): a mainframe estate is not monolithic — z/OS runs the transaction cores (CICS regions), the batch/ETL and file-transfer flows (the SWIFT-file-act theme of [banking/swiftnet_fileact_guide.md](banking/swiftnet_fileact_guide.md)), and the Db2-for-z/OS ledgers; each has a different cloud-adjacency profile (online can be API-fronted, batch can be offloaded, the ledger stays) ✅/⚠ (the three-way split is this guide's synthesis of the verified platform facts; the CICS/z/OS/COBOL dominance itself is industry consensus — §6.1).

The bank-series convention this guide follows: **the core stays; the cloud wraps around it** — which is exactly the architecture of §8.

### 6.2 IBM Cloud for Financial Services (FS-ready)

**IBM Cloud for Financial Services** is IBM's industry cloud for regulated finance: "a first-of-its-kind cloud, designed to protect even your most sensitive data and AI workloads. With built-in security and controls informed by the industry... optimize your infrastructure and demonstrate compliance" ✅ (ibm.com/cloud/financial-services, fetched). The verified control architecture:

- **The IBM Cloud Framework for Financial Services** — "informed by the industry" — is the compliance backbone; services that meet its controls carry the **`fs_ready` label** in the IBM Cloud catalog ✅ (FS page + catalog link, fetched).
- **Security & Compliance Center** — continuous, "edge-to-edge" cloud protection with regulatory compliance at the forefront ✅ (FS page, fetched).
- **Hybrid by design + built for AI** — the FS cloud explicitly pairs with watsonx.ai, watsonx.governance, and watsonx Orchestrate for "trusted AI workflows needed for regulated industries" ✅ (FS page, fetched).
- **DORA whitepaper** — IBM publishes its support posture for the EU Digital Operational Resilience Act ✅ (FS page, fetched).
- **Core-banking modernization** — the BPER Banca collaboration to modernize core banking systems on IBM Cloud (newsroom release referenced from the FS page) ✅.
- History: announced November 2019 with Bank of America as anchor; GA April 2021 ✅ (§2.6).

For the security-model mapping, this guide defers to [technology/zero_trust_network_architecture_guide.md](technology/zero_trust_network_architecture_guide.md) (how the FS-ready controls map onto zero-trust network architecture) and [technology/cybersecurity_guide.md](technology/cybersecurity_guide.md) (the control framework itself). The FS-ready point is not that IBM invented new security — it is that the *cloud service catalog* is pre-screened against a financial-services control framework, so a bank's control evidence is partly inherited from the platform ✅/⚠ (the depth of inherited vs. customer-owned controls is contract-specific — ⚠).

The FS-ready control architecture as a table (rows consolidate the §6.2 verified bullets):

| Element | What it is | Status |
|---|---|---|
| **IBM Cloud Framework for Financial Services** | The compliance backbone, "informed by the industry" | ✅ (§6.2) |
| **fs_ready label** | Catalog label for services meeting the framework's controls | ✅ (§6.2) |
| **Security & Compliance Center** | Continuous edge-to-edge protection; regulatory compliance at the forefront | ✅ (§6.2) |
| **Hybrid-by-design + AI pairing** | FS cloud paired with watsonx.ai / watsonx.governance / watsonx Orchestrate | ✅ (§6.2) |
| **DORA whitepaper** | Published support posture for the EU Digital Operational Resilience Act | ✅ (§6.2) |
| **BPER Banca collaboration** | Core-banking modernization on IBM Cloud | ✅ (§6.2) |
| **History** | Announced November 2019 (Bank of America anchor); GA April 2021 | ✅ (§2.6) |

Operationalizing FS-ready (architecture practice): the bank selects fs_ready-labeled services where the catalog offers them, maps the framework's controls onto its own MAS control inventory (the [banking/mas_regulations_guidelines_guide.md](banking/mas_regulations_guidelines_guide.md) instruments), and uses Security & Compliance Center as the continuous-evidence surface for both the bank's TPRM and any MAS review ✅/⚠ (inheritance depth remains contract-specific — §11 #13).

### 6.3 MAS Outsourcing: Notice 658 and the Cloud

For a Singapore-licensed bank, cloud outsourcing is regulated by **MAS Notice 658** (binding outsourcing requirements) and the **Guidelines on Outsourcing** — the regulatory detail is documented in [banking/mas_regulations_guidelines_guide.md](banking/mas_regulations_guidelines_guide.md) and is only summarized here (cross-reference, not re-derivation): the **Guidelines on Outsourcing (Banks) were published 11 December 2023 and effective 11 December 2024**, and they set MAS's expectations for "ongoing outsourcing relevant services" with exceptions per Annex D of Notices 658/1121 ✅ (verified in the MAS guide). The cloud-specific expectations a bank must engineer for:

- **Materiality assessment** — is the IBM Cloud engagement a "material" outsourcing arrangement? (The test is in the MAS guide §4.4; the platform-side implication is that the bank must be able to evidence the *scope* of what is outsourced — a services/controls inventory is the deliverable.)
- **Lifecycle risk management and notification** — MAS expects notification and ongoing oversight of material outsourcing ✅ (MAS guide).
- **Exit plans** — the bank must be able to exit the cloud provider. The platform-side implication: OpenShift portability (the same platform on-prem or elsewhere) is the technical answer to "what happens if we must leave IBM Cloud" ✅/⚠ (portability is the design intent of OpenShift; the operational reality of a complex bank workload migrating is a program, not a switch — ⚠).
- **Data residency** — customer information must be protected and, in practice, MAS-regulated banks keep customer data in Singapore or approved locations ✅/⚠ (the data-residency specifics for cloud are in the MAS guide; the platform fact is that IBM Cloud's Singapore footprint is the SNG01 data center — §3.1).

The platform-side artifacts an MAS assessment needs from the cloud (architecture practice; the MAS requirements themselves are in the MAS guide): a **services inventory** (which IBM Cloud services are in scope), a **data inventory** (what data each service holds, and where), a **control mapping** (FS-ready framework → the bank's control inventory), and an **exit artifact** (the portability mechanisms — OpenShift redeployment, Cloudant/CouchDB replication, Db2 backup/restore, MQ queue-manager portability — exercised as a plan, not assumed) ✅/⚠ (the mechanisms are verified in §5.3–§5.4; the "artifacts" framing is this guide's).

### 6.4 The Singapore Financial-Services Context

The market context is in [banking/banks_in_singapore_guide.md](banking/banks_in_singapore_guide.md) (the licence architecture — Cymbal Bank's own wholesale-bank position among the ~120–130 foreign banks — and the MAS supervisory style) and [banking/dbs_bank_guide.md](banking/dbs_bank_guide.md) (the flagship digital-bank pattern). The IBM-Cloud-specific facts for Singapore:

- **SNG01** is the Singapore data center — classic infrastructure and PowerVS location ✅ (IBM Cloud docs, fetched).
- **No Singapore MZR** — as of the fetched locations documentation, the Asia-Pacific MZRs are Sydney and Tokyo, with SC-MZRs in Chennai, Osaka, and Mumbai ✅. A bank wanting MZR-grade regional services from IBM Cloud designs around Sydney/Tokyo (or the SC-MZRs), with SNG01 for local residency-sensitive placement ✅/⚠ (design consequence, not an IBM statement).
- **MAS + FS-ready alignment** — the FS-ready framework (control evidence, inherited controls) is the platform-side answer to MAS's TPRM expectations; the *assessment* is still the bank's ✅/⚠ (FS-ready reduces but does not remove the bank's MAS obligations — ⚠, contract/regulatory judgment).

---

## 7. The Comparisons

### 7.1 IBM Cloud vs. AWS / Azure / GCP / Oracle Cloud

The honest frame for this comparison: **the IBM column is verified from fetched primary sources; the competitor columns are standard industry knowledge and are flagged ⚠** (none of the competitors' pages were fetched in this pass, and market-share claims were not independently audited). Cross-reference [banking/oracle_banking_microservices_architecture_guide.md](banking/oracle_banking_microservices_architecture_guide.md) for the Oracle-banking-core angle.

| Dimension | IBM Cloud | AWS ⚠ | Azure ⚠ | GCP ⚠ | Oracle Cloud ⚠ |
|---|---|---|---|---|---|
| **Positioning** | Hybrid + regulated industries; enterprise software heritage (✅ IBM's own positioning) | Largest public cloud by market share; service breadth leader | Enterprise Windows/Office-centric hybrid cloud | Data/AI-native, Kubernetes heritage (GKE) | Enterprise database + applications (Fusion, OCI) |
| **Container story** | **Red Hat OpenShift on IBM Cloud** (managed, 99.99% SLA) ✅ | EKS (managed Kubernetes); ROSA (managed OpenShift) ⚠ | AKS; Azure Red Hat OpenShift ⚠ | GKE; OpenShift on GCP via partners ⚠ | OKE (managed Kubernetes) ⚠ |
| **Hybrid story** | OpenShift everywhere + **Cloud Paks** + **Satellite** ✅ | Outposts, EKS Anywhere ⚠ | Azure Arc, Stack HCI ⚠ | Anthos ⚠ | Dedicated Region, Roving Edge ⚠ |
| **Hardware heritage** | **IBM Z, LinuxONE, Power Systems** — the mainframe and midrange estates (✅ ibm.com/z, power-virtual-server) | x86/ARM (Graviton) ⚠ | x86/ARM (Cobalt) ⚠ | x86/ARM (Axion), TPU ⚠ | x86 + Exadata engineered systems ⚠ |
| **Legacy middleware** | **MQ, CICS, Db2, COBOL toolchain (watsonx Code Assistant for Z)** ✅ | — (migration services only) | — (mainframe migration services) ⚠ | — ⚠ | Oracle DB/Fusion heritage ⚠ |
| **Regulated-industry cloud** | **IBM Cloud for Financial Services** (FS-ready, framework, fs_ready label) ✅ | AWS financial-services competency ⚠ | Azure for financial services ⚠ | — ⚠ | OCI for regulated industries ⚠ |
| **AI portfolio** | **watsonx** (ai/data/governance) + **Granite** open models ✅ | SageMaker/Bedrock ⚠ | Azure AI/OpenAI ⚠ | Vertex AI/Gemini ⚠ | OCI Generative AI ⚠ |
| **Banking adoption pattern** | Mainframe/Power banks modernizing beside the core; FS-ready anchor customers (Bank of America, BNP Paribas ✅ per Wikipedia) | Broad digital-bank adoption, core replacement programs ⚠ | Broad adoption incl. core replacements ⚠ | Data/AI-centric banking workloads ⚠ | Banking cores (e.g. Oracle Banking) ⚠ — see [banking/oracle_banking_microservices_architecture_guide.md](banking/oracle_banking_microservices_architecture_guide.md) |
| **Distinctive risk** | Smaller ecosystem/service count than the big three ⚠; niche in the developer mindshare ⚠ | Complexity/cost governance ⚠ | Cost governance, licensing audits ⚠ | Enterprise support perception ⚠ | Ecosystem lock-in, smaller community ⚠ |

### 7.2 Reading the Table

The table says one structural thing: **each provider's hybrid story starts from its own estate.** AWS/Azure/GCP sell *general-purpose* clouds with hybrid extensions; IBM sells the *enterprise-software continuity* story — the cloud that speaks MQ, Db2, CICS, COBOL, and mainframe natively, and whose container platform (OpenShift) is deliberately identical everywhere. For a bank whose estate is already IBM (which is most banks, at least in the core ✅/⚠), the IBM Cloud decision is rarely "best cloud overall" — it is "least-friction cloud for the estate we already run", with the FS-ready framework as the regulated-workload argument ✅/⚠.

### 7.3 The Estate-Fit Matrix

The comparison that matters for this repository is not "which cloud is biggest" but **"which cloud fits which estate"** — the matrix below is this guide's synthesis (architectural judgment, not fetched claims; every row's IBM column rests on verified §3–§5 facts, competitor cells remain ⚠ industry knowledge per §7.1):

| Estate you run | Best-fit cloud | Why (and the ⚠) |
|---|---|---|
| z/OS + CICS + COBOL core | IBM Cloud | Native MQ/Event Streams integration, watsonx Code Assistant for Z, FS-ready ✅; the §8 offload pattern exists only here ⚠ (judgment) |
| AS/400 → IBM i on Power | IBM Cloud (PowerVS) | Same platform in the cloud — [technology/ibm_as400_guide.md](technology/ibm_as400_guide.md) ✅ (§3.3); competitors offer migration services only ⚠ |
| MQ-centric messaging/SWIFT estate | IBM Cloud | MQ everywhere (z/OS, cloud, appliance, SaaS) + MQ↔Kafka connectors ✅ (§5.4) |
| Oracle database/applications estate | Oracle Cloud — or IBM Cloud via PowerVS | OCI-native per [banking/oracle_banking_microservices_architecture_guide.md](banking/oracle_banking_microservices_architecture_guide.md) ✅/⚠; IBM Cloud runs Oracle on PowerVS ✅ (§3.3) |
| Cloud-native greenfield | AWS/Azure/GCP | Breadth, ecosystem, developer mindshare ⚠ (§7.1) — IBM Cloud competes on the regulated/hybrid edge ⚠ |
| Regulated financial services | IBM Cloud for Financial Services | The only industry cloud in the table with a dedicated FS framework and fs_ready catalog ✅ (§6.2); competitors offer competency programs ⚠ (§7.1) |

**When IBM Cloud wins, and when it does not** (judgment, flagged): it wins when the estate speaks IBM (mainframe, Power, MQ, Db2) or when the regulator's lens (MAS, DORA) makes inherited control evidence the deciding factor ✅/⚠; it does not win on raw service breadth, price/volume economics, or developer mindshare against the big three ⚠ (§7.1, §9).

---

## 8. The Worked Example: Cymbal Bank

### 8.1 The Bank and the Problem

**Cymbal Bank** (the only bank persona in this repository — see [banking/banks_in_singapore_guide.md](banking/banks_in_singapore_guide.md)) is the author's firm: the wholesale-banking arm of a large continental European banking group, strong in trade finance, structured finance, capital markets, and treasury, running its Asia business from a Singapore branch. Its estate is the classic IBM banking stack: **core transaction systems on IBM Z (z/OS, CICS, COBOL)**, an **IBM MQ** messaging backbone (the same backbone the payment and SWIFT guides document), Db2 for the ledgers, and a growing set of digital channels that need APIs, events, and AI — fast. The problem: the channels are being strangled by the core's release cycle, the AI pilots have no governance rails, and MAS expects a defensible outsourcing posture for anything that moves to the cloud.

*This worked example is a fictional target architecture, shaped on the verified platform facts of §3–§6. Nothing here describes a real Cymbal Bank engagement.*

### 8.2 The Target Architecture: The Hybrid House

The target architecture has five layers, all on the "hybrid house" pattern — one platform, one governance model, workloads placed by requirement:

1. **The common plane: Red Hat OpenShift.** Red Hat OpenShift on IBM Cloud runs the new channel workloads (API gateway tier, onboarding flows, trade-finance portal, AML-adjacent analytics front-ends) in the Frankfurt MZR for the European book and the Sydney MZR for Asia — with the Singapore branch's local, residency-sensitive services on SNG01-connected infrastructure. The same OpenShift platform runs on-premises beside the z/OS core, so a workload's placement (cloud vs. on-prem) is a policy decision, not an architecture fork ✅ (OpenShift portability is the platform's design intent, §4.1–4.2).

2. **The core stays: the z/OS offload pattern.** The CICS/COBOL core remains on IBM Z. What moves to the cloud is the *traffic*: channels call APIs on OpenShift; the APIs translate into MQ messages to the core; the core answers over MQ. Batch and file-activities stay on the mainframe's schedule. COBOL-to-Java modernization is done incrementally, assisted by watsonx Code Assistant for Z for the refactoring analysis ✅ (§3.4, §6.1) — the "modernize beside the core" pattern of §4.4.

3. **The integration layer: MQ, Event Streams, App Connect.** MQ (with MQ for z/OS on the core side and MQ on the cloud/OpenShift side) carries the transactional traffic — exactly-once, never losing a message ✅ (§5.4). Event Streams (Kafka) carries the event side: trade-lifecycle events, payment-status changes, fraud signals — bridged to MQ via the supported MQ↔Kafka connectors ✅. App Connect handles the API/format mediation, including the file-transfer-style integrations the SWIFT guides describe ([banking/swiftnet_fileact_guide.md](banking/swiftnet_fileact_guide.md)) ✅ (§5.4).

4. **The data layer: Db2, Cloudant, watsonx.data.** Db2 on the cloud mirrors and serves analytics beside the on-prem Db2 (z/OS); Cloudant provides the CouchDB-compatible document tier for channel state with replication to an on-prem CouchDB for sovereignty-sensitive data ✅ (§5.3); watsonx.data is the lakehouse for the AI tier ✅ (§5.1).

5. **The governance overlay: watsonx.governance + FS-ready + MAS.** AI workloads (AML alert enrichment, trade-document extraction, customer-assist) run on Granite models through watsonx.ai, registered and monitored in watsonx.governance ✅ (§5.1–5.2) — with the AI-governance depth (model risk, explainability, evaluation) deferred to [banking/ai_genai_banking_compliance_guide.md](banking/ai_genai_banking_compliance_guide.md) and the [technology/ai_llm/rag/](technology/ai_llm/rag/) guides. The cloud services selected carry the **fs_ready** label where available ✅ (§6.2), the Security & Compliance Center provides the continuous-control evidence ✅, and the whole cloud boundary sits inside the zero-trust design of [technology/zero_trust_network_architecture_guide.md](technology/zero_trust_network_architecture_guide.md).

The five layers as a table (the layer / pattern / service / MAS-658 mapping):

| Layer | Pattern | IBM Cloud / platform element | MAS Notice 658 note |
|---|---|---|---|
| **1. Common plane** | OpenShift as the single platform (cloud + on-prem) | Red Hat OpenShift on IBM Cloud (Frankfurt `eu-de`, Sydney `au-syd`); on-prem OpenShift beside z/OS ✅ (§4.2) | Exit plan: the platform is portable — redeploy on-prem or another certified cloud ✅/⚠ (§8.3) |
| **2. Core offload** | z/OS stays; the traffic moves | APIs on OpenShift → MQ → CICS core; watsonx Code Assistant for Z for incremental COBOL-to-Java ✅ (§3.4, §6.1) | Materiality: the core remains in-house, capping the outsourced scope ✅/⚠ (§8.3) |
| **3. Integration** | Transactional MQ + event Kafka, bridged | MQ (cloud + z/OS), Event Streams, MQ↔Kafka connectors, App Connect, API Connect ✅ (§5.4) | Lifecycle risk: services inventory and TPRM over the integration tier ✅/⚠ (§8.3) |
| **4. Data** | Hybrid data plane | Db2 (cloud mirrors + z/OS), Cloudant ↔ on-prem CouchDB replication, watsonx.data lakehouse ✅ (§5.3) | Data residency: Singapore-branch data stays on SNG01/on-prem ✅/⚠ (§8.3) |
| **5. Governance** | One governance overlay | watsonx.governance + Security & Compliance Center + FS-ready framework; zero-trust per [technology/zero_trust_network_architecture_guide.md](technology/zero_trust_network_architecture_guide.md) ✅ (§6.2) | Notification + evidence: MAS filing and continuous control evidence ✅/⚠ (§8.3) |

### 8.3 The MAS Notice 658 Assessment

The outsourcing assessment for the architecture above, in the shape MAS expects (detail in [banking/mas_regulations_guidelines_guide.md](banking/mas_regulations_guidelines_guide.md) §4.4):

| MAS expectation | Cymbal Bank's answer |
|---|---|
| **Materiality assessment** | The outsourced scope (channel APIs, analytics, AI services on IBM Cloud) is assessed for materiality per Notice 658 / the Outsourcing Guidelines; the z/OS core remains in-house, which caps the materiality of the arrangement ✅/⚠ (assessment methodology is in the MAS guide; the scope statement here is the design's) |
| **Lifecycle risk management** | TPRM process with an inventory of cloud services, risk ratings, and the FS-ready framework as inherited-control evidence ✅/⚠ |
| **Notification** | MAS notification filed for the material arrangement ✅/⚠ (the filing itself is outside this guide) |
| **Exit plan** | OpenShift portability (workloads can redeploy on-prem or on another certified cloud), data-export via Cloudant/CouchDB replication and Db2 backup/restore, MQ queue-manager portability ✅/⚠ (technical mechanisms are real; an exercised exit is a program, not a switch) |
| **Data residency** | Customer data for the Singapore branch stays in Singapore (SNG01) or MAS-acceptable locations; the Sydney/Frankfurt placements carry only non-customer or appropriately governed data ✅/⚠ (residency policy judgment) |

### 8.4 The Phased Roadmap

The hybrid house is built in three phases (architecture practice — the phases are this guide's sequencing of the verified capabilities; each phase's mechanisms are ✅-verified in the cited sections):

| Phase | What moves | Mechanisms (verified) | Exit/control posture |
|---|---|---|---|
| **1. Connect (months 1–6)** | Channels and APIs onto OpenShift; Direct Link + Transit Gateway private connectivity; MQ bridge to the core | OpenShift ✅ (§4.2), MQ ✅ (§5.4), Direct Link/Transit Gateway ✅/⚠ (§3.6) | Smallest outsourced scope; the earliest defensible MAS notification basis ✅/⚠ |
| **2. Data & AI (months 6–18)** | Analytics off the core; watsonx.data lakehouse; Granite models under watsonx.governance; Cloudant for channel state | watsonx ✅ (§5.1), Granite ✅ (§5.2), Cloudant ✅ (§5.3) | FS-ready services selected; the AI model inventory lives in governance ✅/⚠ |
| **3. Modernize (ongoing)** | Incremental COBOL-to-Java refactoring with watsonx Code Assistant for Z; batch-offload candidates | watsonx Code Assistant for Z ✅ (§3.4, §6.1) | The core stays — the materiality cap is preserved ✅/⚠ |

The sequencing logic: connectivity first (so the estate speaks one language), data/AI second (so the AI pilots get governance rails before they scale — the §8.1 problem statement), modernization last and incrementally (so the core is never at risk).

### 8.5 What the Example Is For

The worked example exists to make one architectural claim concrete: **for a bank with an IBM estate, the hybrid-cloud target is not "move the core to the cloud" — it is "run the new world on OpenShift beside the old world, integrate over MQ, govern everything under one framework, and keep the exit door open."** That is the hybrid house (§9).

---

## 9. The Summary: The Hybrid House

IBM Cloud is the platform that IBM built by buying its way into the cloud era twice — the SoftLayer infrastructure (2013) and the Red Hat platform (2018/2019) — with the Bluemix PaaS experiment (2014) in between ✅ (§2). The result is a cloud whose identity is *continuity, not disruption*: it connects to the estates IBM already dominates (z/OS, Power, Db2, MQ, CICS) rather than pretending they do not exist ✅ (§3, §6). Its distinctive assets, verified in this pass: the **multizone region architecture** (9 MZRs + 4 SC-MZRs, with Singapore as the SNG01 classic data center) ✅; **Red Hat OpenShift on IBM Cloud** as a fully managed, 99.99% SLA common plane ✅; the **Cloud Paks** as OpenShift-native middleware ✅; the **watsonx** portfolio and **Granite** open models ✅; the **integration trio** of MQ, App Connect, and Event Streams ✅; and — for this series — **IBM Cloud for Financial Services** with its FS-ready framework ✅. Its honest limits, flagged throughout: the Singapore MZR gap ⚠, the unconfirmed Z/LinuxONE as-a-service product ⚠, the dated "six MZRs" figure ⚠, and the competitive reality that IBM Cloud is not the biggest cloud, but the most *native* cloud for the estates this repository documents ✅/⚠ (§7). The house has a foundation (the estate), a floor plan (OpenShift), pipes (MQ and Event Streams), a governance floor (FS-ready and watsonx.governance), and a door that opens both ways (hybrid portability). That is the hybrid house.

---

## 10. Verification and Claims-Status

This guide was researched with a bounded web pass (direct fetches of ibm.com, redhat.com, cloud.ibm.com, PRNewswire, Wikipedia; no paywalled access). Facts marked ✅ trace to fetched sources listed in §13; facts marked ⚠ are flagged at the point of use and consolidated in §11. Method note: IBM's own pages are primary but vendor-sourced — marketing figures (case-study percentages, throughput claims) are marked ✅/⚠ accordingly and treated as vendor claims.

| Claim | Status | Source |
|---|---|---|
| SoftLayer acquisition announced 4 June 2013, closed 8 July 2013 | ✅ | PRNewswire (IBM releases), fetched |
| Bluemix public beta February 2014; GA July 2014; Cloud Foundry-based | ✅ | Wikipedia IBM Cloud; IBM announcement archive ENUSAP14-0304, fetched |
| Bluemix/SoftLayer rebranded to IBM Cloud, October 2017 | ✅ | Wikipedia; Cloud Computing News (2 Nov 2017, citing IBM's "Bluemix is now IBM Cloud" blog) |
| Red Hat acquisition announced October 2018 (~US$34B); completed 9 July 2019 at US$190/share | ✅ | Wikipedia; IBM Investor news, fetched; announcement day-level ⚠ |
| IBM Singapore cloud data centre opened March 2011 (US$38M; April 2011 IaaS launch) | ✅ | DataCenterKnowledge (7 Mar 2011), fetched |
| Managed Red Hat OpenShift on IBM Cloud launched August 2019 | ✅ | Wikipedia IBM Cloud, fetched |
| Financial services-ready cloud announced November 2019 (Bank of America); FS cloud GA April 2021 | ✅ | Wikipedia IBM Cloud, fetched |
| MZR/SC-MZR list; SNG01 classic data center; MZR definitions and SLA tier | ✅ | cloud.ibm.com locations docs, fetched |
| 60+ data centers / 19 countries / six MZRs (Wikipedia citing IBM) | ⚠ | Wikipedia, fetched; figure dated vs. current docs |
| PowerVS: multitenant Power servers; AIX/IBM i/Linux; SAP/Oracle; 250+ services | ✅ | ibm.com/products/power-virtual-server, fetched |
| IBM Z z17/z16, Telum, z/OS/Linux/z/TPF/z/VM; LinuxONE 4/5 | ✅ | ibm.com/z; ibm.com/products/linuxone, fetched |
| Z/LinuxONE as-a-service product | ⚠ | no dedicated page found; URL redirected to product catalog |
| Cloud Paks current set (Data, Business Automation, AIOps, Integration, Network Automation, Applications + System) | ✅ | ibm.com/cloud-paks, fetched |
| Cloud Pak for Security current status; Cloud Pak rename/retirement history | ⚠ | not on current Cloud Paks page; history not fetched |
| watsonx portfolio components; Granite Apache 2.0, Granite 4.2 families | ✅ | ibm.com/watsonx; ibm.com/granite, fetched |
| Db2, Db2 Warehouse, Cloudant (CouchDB-based, 99.99% SLA, 6 regions) | ✅ | ibm.com/cloud; ibm.com/products/cloudant, fetched |
| MQ exactly-once, MQ for z/OS, MQ↔Kafka connectors, MFT | ✅ | ibm.com/products/mq, fetched |
| Event Streams: Kafka, 99.99%, FS-validated Enterprise plan, 10 MZRs | ✅ | ibm.com/products/event-streams, fetched |
| App Connect: 200+ connectors, mainframe/MQ/Kafka connectivity | ✅ | ibm.com/products/app-connect, fetched |
| IBM Cloud for Financial Services: framework, fs_ready, Security & Compliance Center | ✅ | ibm.com/cloud/financial-services, fetched |
| Notice 658 + Outsourcing Guidelines (Dec 2023, effective Dec 2024) | ✅ | via [banking/mas_regulations_guidelines_guide.md](banking/mas_regulations_guidelines_guide.md) |
| Direct Link / Transit Gateway service detail | ⚠ | docs pages not retrievable in this pass |
| Competitor columns in §7 (AWS/Azure/GCP/Oracle) | ⚠ | standard industry knowledge; no competitor pages fetched |
| Cloud Pak System (integrated VM/container private cloud) | ✅ | ibm.com/cloud-paks, re-fetched in this expansion pass |
| Cloud Pak component bundles (per-Cloud-Pak software) | ⚠ | not re-verified in this expansion pass; industry knowledge only (§4.3) |
| Continuous Delivery / Toolchains; ibmcloud CLI / Terraform provider | ⚠ | catalog-level knowledge; not re-fetched in this expansion pass (§5.5) |
| Storage backup/archive tier and object lifecycle specifics | ⚠ | catalog-level only (§3.5) |
| Vendor-reported figures (US Treasury $10–12T/day; ANZ 90%; Swedbank 15%) | ⚠ | IBM case-study pages, referenced; vendor-sourced |

---

## 11. What Could Not Be Verified

The following claims could not be confirmed from primary sources fetched during this pass; they are flagged ⚠ at their point of use and listed here per the series convention:

1. **Singapore data center (SNG01) launch date** — could not be confirmed from primary sources. What *is* confirmed: SNG01 exists as a classic data center in the current IBM Cloud locations documentation.
2. **IBM Z / LinuxONE "as-a-service" offering** — no dedicated, current product page could be confirmed from primary sources (the natural URL redirected to the generic product catalog). The verified reality is IBM Z and LinuxONE hardware with hybrid-cloud positioning and watsonx Code Assistant for Z tooling.
3. **The IBM press release for the October 2017 Bluemix/SoftLayer → IBM Cloud rebrand** — the rebrand date rests on Wikipedia's citation of IBM's October 2017 announcement; the IBM release itself was not directly retrieved in this pass. Day-level detail is not stated.
4. **Exact announcement/close days for the Red Hat acquisition** (widely reported as 29 October 2018 and 9 July 2019) — month-level and amount (~US$34B) are confirmed; day-level detail was not confirmed from a fetched primary source.
5. **Cloud Pak for Security's current status** — not listed on the fetched Cloud Paks page; its placement (renamed, folded into other products, or simply unlisted) could not be confirmed.
6. **Cloud Pak rename/retirement history** (Watson AIOps → AIOps; Automation → Business Automation; Multicloud Management discontinuation) — industry knowledge, not confirmed from fetched primary sources.
7. **The "60+ data centers across 19 countries and six multizone regions" figure** (Wikipedia citing IBM) — dated relative to the current locations documentation (9 MZRs + 4 SC-MZRs); the older figure is flagged rather than repeated as current.
8. **VPC "Gen 2" branding** — widely used in IBM materials but not directly confirmed on a fetched page in this pass.
9. **Direct Link and Transit Gateway documentation detail** — the cloud.ibm.com docs pages could not be fetched (JavaScript-rendered empty responses); only catalog-level corroboration was obtained.
10. **Storage service-level specifics** (Block/File tiers, IOPS classes, replication options) — not verified from primary documentation in this pass.
11. **All competitor-column claims in §7** (AWS/Azure/GCP/Oracle services, market shares, banking adoption) — none of the competitors' pages were fetched; flagged ⚠ as standard industry knowledge.
12. **Vendor-reported marketing figures** — US Treasury "$10–12 trillion/day" on IBM Z; ANZ 90% licensing-cost reduction / 60% ops effort; Swedbank 15% forecasts; the "11 million bare-metal configurations" figure; case-study percentages throughout — treated as vendor claims, not independently audited.
13. **FS-ready control inheritance depth** — how much control evidence a bank inherits vs. owns under IBM Cloud for Financial Services is contract- and scope-specific; not confirmable from public pages.
14. **Singapore MZR history** — whether IBM ever announced (and later shelved) a Singapore multizone region could not be confirmed; the current documentation lists SNG01 as a classic data center only.
15. **MAS-specific residency/notification rulings for IBM Cloud placements** — regulatory interpretation is outside this guide's scope and is deferred to the MAS guide.
16. **Cloud Pak component bundles** — which software ships inside each Cloud Pak (the integration stack inside Cloud Pak for Integration, the data stack inside Cloud Pak for Data, etc.) is industry knowledge; the Cloud Paks page re-fetched in this expansion pass confirms the product *set* and Cloud Pak System's private-cloud role but does not enumerate bundle contents (§4.3).
17. **Continuous Delivery / Toolchains and the ibmcloud CLI / Terraform provider** — service existence is catalog-level knowledge; capabilities were not re-fetched in this expansion pass (§5.5).
18. **Storage backup/archive tier specifics and object-storage lifecycle policies** — not re-verified in this expansion pass; the storage rows in §3.5 are catalog-level only.

---

## 12. Glossary

| Term | Definition |
|---|---|
| **IBM Cloud** | IBM's public/hybrid cloud platform: IaaS, PaaS, containers, data, AI, integration, and industry clouds (✅ §1) |
| **SoftLayer** | The Dallas-founded bare-metal/cloud infrastructure provider IBM acquired in 2013; the basis of IBM Cloud's classic infrastructure layer (✅ §2.2) |
| **Bluemix** | IBM's Cloud Foundry-based PaaS (2014–2017); folded into the IBM Cloud brand in October 2017 (✅ §2.3) |
| **MZR** | Multizone region: a region across ≥3 zones with independent power/cooling/network; <2ms intra-zone latency, >1000 Gbps; tier-3 (99.99%) regional services (✅ §3.1) |
| **SC-MZR** | Single-campus multizone region: 3 zones within one campus; same SLA class (✅ §3.1) |
| **SNG01** | IBM Cloud's Singapore classic data center; not an MZR per current documentation (✅ §3.1) |
| **VPC** | Virtual Private Cloud — IBM Cloud's virtual networking architecture (regions, zones, subnets, security groups); the current generation is commonly called VPC Gen 2 (✅/⚠ §3.6) |
| **Bare Metal Server** | Dedicated single-tenant hardware on IBM Cloud — the SoftLayer heritage product (✅ §3.2) |
| **Hyper Protect Virtual Servers** | Locked-down VMs for sensitive-data workloads (✅ §3.2) |
| **IKS** | IBM Cloud Kubernetes Service — IBM's managed Kubernetes (2017) (✅ §2.4) |
| **Cloud Functions** | IBM's FaaS, seeded from Apache OpenWhisk (2016) (✅ §2.3, §3.2) |
| **Code Engine** | Serverless container hosting (PaaS-style) (✅ §3.2, §5.5) |
| **Schematics** | Terraform-based infrastructure-as-code provisioning on IBM Cloud (✅ §5.5) |
| **Container Registry** | IBM Cloud's private image registry for the container estate (✅ §5.5) |
| **Observability** | IBM Cloud Monitoring/Logs — the metrics and logging surface (✅ §5.5) |
| **Security & Compliance Center** | Continuous cloud protection and control evidence; the FS-ready reporting surface (✅ §5.5, §6.2) |
| **Deployable Architectures** | Pre-built OpenShift cluster architectures meeting "hundreds of controls by default" (✅ §4.2) |
| **OpenShift Virtualization (KubeVirt)** | VMs on OpenShift via KubeVirt — the VMware-replacement play (✅ §4.2) |
| **RHEL** | Red Hat Enterprise Linux — the OS layer under the OpenShift stack (⚠ not re-verified in this pass) |
| **OpenShift** | Red Hat's enterprise Kubernetes container platform; the common plane of IBM's hybrid strategy (✅ §4.1) |
| **Red Hat OpenShift on IBM Cloud** | IBM's fully managed OpenShift service; 99.99% SLA, HA clusters, GPU nodes (✅ §4.2) |
| **Cloud Pak** | Containerized, pre-integrated IBM software (data, integration, automation, AIOps, network automation, applications) designed to run on OpenShift (✅ §4.3) |
| **Cloud Pak for Data** | The data/analytics Cloud Pak (✅ §4.3) |
| **Cloud Pak for Integration** | The integration Cloud Pak — the home of the API/MQ/event integration stack (✅ §4.3) |
| **PowerVS** | IBM Power Virtual Server — multitenant virtual Power servers on IBM Cloud for AIX, IBM i, and Linux workloads (✅ §3.3) |
| **IBM Z** | The mainframe line (z17/z16); runs z/OS, Linux, z/TPF, z/VM; Telum on-chip AI (✅ §3.4) |
| **LinuxONE** | IBM's Linux-only mainframe-class servers (LinuxONE 4/5, Telum/Telum II) (✅ §3.4) |
| **z/OS** | The mainframe operating system that runs banking cores (CICS/COBOL estates) (✅ §3.4) |
| **CICS** | Customer Information Control System — IBM's mainframe transaction server (✅/⚠ §6.1) |
| **COBOL** | The dominant business language of the mainframe core estate (✅/⚠ §6.1) |
| **Telum** | IBM Z/LinuxONE processor with on-chip AI accelerators (fraud-detection use case) (✅ §3.4) |
| **watsonx** | IBM's AI portfolio: watsonx.ai, watsonx.data, watsonx.governance, Orchestrate, and more (✅ §5.1) |
| **watsonx.ai** | The AI studio — train, validate, tune, deploy models (✅ §5.1) |
| **watsonx.data** | The hybrid open data lakehouse with automated governance (✅ §5.1) |
| **watsonx Orchestrate** | AI agents and assistants for workflow automation (✅ §5.1) |
| **Granite** | IBM's open-source (Apache 2.0) model family: language, speech, vision, guardian, embedding, time series (✅ §5.2) |
| **Granite Guardian** | The Granite guardrails family for model input/output safety (✅ §5.2) |
| **watsonx.governance** | AI governance across the model lifecycle — visibility, control, accountability (✅ §5.1) |
| **watsonx Code Assistant for Z** | Generative-AI tooling for mainframe app understanding and COBOL-to-Java refactoring (✅ §3.4) |
| **Db2** | IBM's relational database family: Db2 for z/OS, Db2 for i, Db2 LUW, Db2 Warehouse on cloud (✅ §5.3) |
| **Db2 Warehouse** | IBM's managed data warehouse — deployable on IBM Cloud or AWS (✅ §5.3) |
| **Cloudant** | IBM's managed JSON document database, built on Apache CouchDB; CouchDB-compatible replication (✅ §5.3) |
| **CouchDB** | The open-source JSON document database Cloudant is built on; its replication protocol is the hybrid hook (✅ §5.3) |
| **MQ** | IBM MQ — the enterprise message broker ("never lose a message"; exactly-once; MQ for z/OS; MQ↔Kafka connectors) (✅ §5.4) |
| **MQ for z/OS** | The mainframe edition of MQ — the queue manager on the core side of the §8 offload pattern (✅ §5.4) |
| **MFT** | Managed file transfer — the MQ capability behind the SWIFT file-act integrations (✅ §5.4) |
| **Event Streams** | IBM's enterprise Apache Kafka platform; fully managed; FS-validated Enterprise plan (✅ §5.4) |
| **Kafka** | Apache Kafka — the open-source event-streaming backbone Event Streams is built on (✅ §5.4) |
| **App Connect** | IBM's hybrid integration platform (200+ connectors; API/event/messaging-led integration; mainframe connectivity) (✅ §5.4) |
| **Satellite** | IBM Cloud Satellite — IBM Cloud services delivered to on-premises and edge locations (✅ §3.6) |
| **Direct Link** | Private dedicated connectivity from on-premises to IBM Cloud (⚠ §3.6) |
| **Transit Gateway** | Hub-and-spoke interconnection between IBM Cloud networks (⚠ §3.6) |
| **IBM Cloud for Financial Services** | IBM's FS-ready industry cloud; the IBM Cloud Framework for Financial Services; fs_ready catalog label (✅ §6.2) |
| **FS-ready / fs_ready** | The label for IBM Cloud services validated against the financial-services control framework (✅ §6.2) |
| **Notice 658** | MAS Notice on outsourcing — binding requirements for Singapore banks outsourcing relevant services (✅ via MAS guide §6.3) |
| **Outsourcing Guidelines** | MAS Guidelines on Outsourcing (Banks): published 11 December 2023, effective 11 December 2024 (✅ via MAS guide) |
| **TPRM** | Third-Party Risk Management — the bank-side discipline MAS outsourcing rules require (✅ via MAS guide) |
| **DORA** | EU Digital Operational Resilience Act — IBM publishes a support whitepaper for it on the FS cloud (✅ §6.2) |
| **Cymbal Bank** | The repository's bank persona: the author's firm, a European wholesale banking group with a Singapore branch (✅ repo convention, §8) |

---

## 13. References

All URLs below were fetched (or, where marked, referenced from a fetched page) during this pass.

1. IBM — *IBM Cloud products* ("230+ products"; enterprise hybrid cloud platform; Satellite). https://www.ibm.com/cloud
2. PRNewswire / IBM — *IBM to Acquire SoftLayer to Accelerate Adoption of Cloud Computing in the Enterprise* (4 June 2013). https://www.prnewswire.com/news-releases/ibm-to-acquire-softlayer-to-accelerate-adoption-of-cloud-computing-in-the-enterprise-210061861.html
3. PRNewswire / IBM — *IBM Closes Acquisition of SoftLayer Technologies* (8 July 2013). https://www.prnewswire.com/news-releases/ibm-closes-acquisition-of-softlayer-technologies-214589711.html
4. IBM announcement archive — *IBM Bluemix provides an open standards-based cloud platform* (ENUSAP14-0304). https://www.ibm.com/docs/en/announcement_archive/ENUSAP14-0304/ENUSAP14-0304.PDF
5. PRNewswire / IBM — *IBM Announces New BlueMix Services* (28 April 2014). https://www.prnewswire.com/news-releases/ibm-announces-new-bluemix-services-to-help-developers-build-applications-in-the-cloud-257022541.html
6. Wikipedia — *IBM Cloud* (history: SoftLayer, Bluemix, October 2017 rebrand, Red Hat acquisition, FS cloud timeline). https://en.wikipedia.org/wiki/IBM_Cloud
7. IBM — *IBM Cloud Paks* (current Cloud Pak set). https://www.ibm.com/cloud-paks
8. Red Hat — *Red Hat OpenShift* (the platform). https://www.redhat.com/en/technologies/cloud-computing/openshift
9. IBM — *Red Hat OpenShift on IBM Cloud* (managed service; 99.99% SLA; GPU nodes; deployable architectures). https://www.ibm.com/products/openshift
10. IBM — *IBM Cloud global data centers*. https://www.ibm.com/cloud/data-centers/
11. IBM Cloud Docs — *IBM Cloud regions and data centers for high availability* (MZR/SC-MZR lists; SNG01; zone mapping; SLAs). https://cloud.ibm.com/docs/overview?topic=overview-locations
12. IBM — *IBM Power Virtual Server* (AIX/IBM i/Linux; SAP; Oracle; 250+ services). https://www.ibm.com/products/power-virtual-server
13. IBM — *IBM Z* (z17/z16; Telum; operating systems; case studies). https://www.ibm.com/z
14. IBM — *IBM LinuxONE* (LinuxONE 4/5; Telum II; confidential containers). https://www.ibm.com/products/linuxone
15. IBM — *IBM watsonx* (portfolio). https://www.ibm.com/watsonx
16. IBM — *Granite* (Apache 2.0; Granite 4.2 model families; signing; ISO certification). https://www.ibm.com/granite
17. IBM — *IBM Cloud for Financial Services* (framework; fs_ready; Security & Compliance Center; DORA; BPER). https://www.ibm.com/cloud/financial-services
18. IBM — *IBM MQ* (exactly-once; MQ for z/OS; MQ↔Kafka connectors; MFT; banking use case). https://www.ibm.com/products/mq
19. IBM — *IBM Event Streams* (Apache Kafka; 99.99%; FS-validated Enterprise plan; 10 MZRs). https://www.ibm.com/products/event-streams
20. IBM — *IBM App Connect* (200+ connectors; mainframe/MQ/Kafka connectivity; Karnataka Bank case). https://www.ibm.com/products/app-connect
21. IBM — *IBM Cloudant* (CouchDB-based; 99.99% SLA; 6 regions; hybrid replication). https://www.ibm.com/products/cloudant
22. IBM Newsroom (referenced from the fetched OpenShift product page) — *IBM Announces Red Hat AI Inference and Red Hat OpenShift Virtualization Service on IBM Cloud* (12 May 2026). https://newsroom.ibm.com/2026-05-12-ibm-announces-red-hat-ai-inference-and-red-hat-openShift-virtualization-service-on-ibm-cloud
23. IBM Newsroom (referenced from the fetched FS page) — *BPER Banca Group Collaborates with IBM to Modernize Core Banking Systems* (27 October 2022). https://newsroom.ibm.com/2022-10-27-BPER-Banca-Group-Collaborates-with-IBM-to-Modernize-Core-Banking-Systems-and-Accelerate-Digital-Transformation-Plan
24. Repo cross-references (verified in earlier guides of this series): [banking/mas_regulations_guidelines_guide.md](banking/mas_regulations_guidelines_guide.md) (Notice 658; Outsourcing Guidelines), [banking/banks_in_singapore_guide.md](banking/banks_in_singapore_guide.md) (licence architecture; Cymbal Bank persona), [banking/ai_genai_banking_compliance_guide.md](banking/ai_genai_banking_compliance_guide.md) and [technology/ai_llm/enterprise_ai_platforms_guide.md](technology/ai_llm/enterprise_ai_platforms_guide.md) (AI governance), [technology/ibm_as400_guide.md](technology/ibm_as400_guide.md) (platform lineage).

---

*End of guide. Series context: the cloud-platform deep-dive of the research repository — read it beside the AS/400 platform guide and the banking series, and treat the ⚠ items in §11 as the honest boundary of what a bounded research pass can confirm. IBM Cloud is not the biggest cloud; it is the cloud built around the estate this series documents — the hybrid house.*
