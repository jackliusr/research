# Software-Defined Data Center - The Control Plane Is the Asset

> **Author:** Jack Liu Shurui, Solution Architect
> **Context:** Technology Research - Infrastructure / Private-Cloud series; the abstraction layer itself: what "software-defined" actually hides, where the control plane now lives, what no abstraction can hide, and why both the bill and the failure modes follow from the declaration rather than from the tin
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** Steve Herrod's 2012 VMware CTO post (Wayback capture, 16 January 2014); Broadcom's "Broadcom Completes Acquisition of VMware" release, 22 November 2023; the VMware Cloud Foundation 9.1 datasheet (November 2026) and the VCF product page (retrieved 28 November 2026); the KubeVirt user guide and project pages (retrieved 28 November 2026); IBM's "What is a software-defined data center (SDDC)?" explainer (retrieved 14 May 2026); Wikipedia's SDDC article and the CRN piece it cites (13 May 2013); a measured grep of this repository, 28 November 2026
> **Last Updated:** November 2026

---

### Table of Contents

1. [Overview - The Decoder, the Gap and the Thesis](#1-overview---the-decoder-the-gap-and-the-thesis)
2. [The Four Abstractions](#2-the-four-abstractions)
3. [The Control Plane Is the Asset - and It Moved](#3-the-control-plane-is-the-asset---and-it-moved)
4. [The Architecture, Layer by Layer](#4-the-architecture-layer-by-layer)
5. [What the Abstraction Buys, Honestly](#5-what-the-abstraction-buys-honestly)
6. [What It Costs](#6-what-it-costs)
7. [The Economics Are Licence Economics](#7-the-economics-are-licence-economics)
8. [The Vendor Landscape, Structurally](#8-the-vendor-landscape-structurally)
9. [The Failure Modes of the Abstraction](#9-the-failure-modes-of-the-abstraction)
10. [The Regulated and Banking Angle](#10-the-regulated-and-banking-angle)
11. [The State of Practice in 2026](#11-the-state-of-practice-in-2026)
12. [The Anti-Patterns](#12-the-anti-patterns)
13. [The Cymbal Bank Worked Example](#13-the-cymbal-bank-worked-example)
14. [The Claims Audit](#14-the-claims-audit)
15. [What Could Not Be Verified](#15-what-could-not-be-verified)
16. [Glossary, Cross-References and Closing Summary](#16-glossary-cross-references-and-closing-summary)

### How to Read This Guide

This guide owns exactly one thing: **the abstraction layer**. It does not describe a building, and it does not compare products.

- **Facilities are owned elsewhere.** Power, cooling, PUE and the other efficiency ratios, Uptime Institute tiers, racks and layout, physical security, certification, sustainability and colocation economics all belong to [data_center_guide.md](data_center_guide.md), and are cross-referenced there by section number: its section 2 Facility Anatomy, section 3 Taxonomy, section 4 Uptime Tiers, section 5 Power, section 6 Cooling, section 7 Layout and Racks, section 8 Physical Security, section 9 Certification and Operations, section 10 Metrics, section 11 Sustainability, section 12 Colocation Economics. That guide is 706 lines of facilities and contains no section about software. This guide is the other half of the same building.
- **The vendor head-to-heads are owned elsewhere.** Six sibling guides run the product comparisons this repository has already published: [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md), [cloudstack_vs_openstack_guide.md](cloudstack_vs_openstack_guide.md), [nutanix_enterprise_ai_vs_openshift_ai_guide.md](nutanix_enterprise_ai_vs_openshift_ai_guide.md), [charmed_kubernetes_vs_openshift_guide.md](charmed_kubernetes_vs_openshift_guide.md), [charmed_kubeflow_vs_openshift_ai_guide.md](charmed_kubeflow_vs_openshift_ai_guide.md) and [devstack_openstack_guide.md](devstack_openstack_guide.md). This guide names vendors only as structural positions in a market (section 8), never as contestants in a feature comparison. No section here is a bake-off, and section 8 says so explicitly.
- **Nothing here has a performance number or a price.** That is deliberate. The subject is mechanism and trade-off; the moment a latency figure or a per-unit rate enters, the argument becomes a procurement memo for one product against another, which is the failure mode this guide exists to avoid.

**A note on verification.** Claims carry one of three marks inline: check-mark "✅ **Verified**" (confirmed at the named primary source), warning "⚠️ **Flagged**" (true but approximate, dated, self-reported by a vendor, or disputed), cross "❌ **Rejected / not established**" (could not be confirmed, and therefore not asserted). Each mark is followed by its source and date. Section 14 collects the claims into an audit table with version and kind; section 15 lists what could not be verified, including where the tooling itself failed.

---

## 1. Overview - The Decoder, the Gap and the Thesis

### 1.1 The gap this guide was written to fill, measured

Before any argument, the evidence that the argument was missing. On 28 November 2026 the repository was grepped for the concept and for its parts. The command, run once per pattern from the repository root:

```
grep -rliE '<pattern>' --include='*.md' . | wc -l
```

| Pattern | Files matching |
|---|---|
| `software.defined data cent` or `\bSDDC\b` | 0 |
| `\bNSX\b` | 2 |
| `vSAN` | 3 |
| `software.defined (networking\|storage\|compute)` | 5 |
| `hyperconverged` or `\bHCI\b` | 12 |
| `\bNutanix\b` | 12 |
| `vSphere` | 14 |
| `private cloud` | 45 |

One note on reproducing this table. The greps were run **before** this file existed, which is why the concept row is a true zero rather than a one. Re-running them now returns exactly one more file in every row: this guide, which of course mentions all of these terms. The correct control for a reader checking the claim is to exclude it, or to compare against the vendor shelf directly:

```
grep -rliE 'software.defined data cent|\bSDDC\b' --include='*.md' . | grep -v software_defined_data_center_guide
```

That command returning nothing is the claim in one line. If it returns files, the gap has been filled since and this paragraph is out of date - which is the only honest way to write a measurement into a document that changes the thing it measures.

Read the shape of that table, not the individual cells. **Absolute zero** files mention the concept or its acronym. **Near-zero** mention its components (two for NSX, three for vSAN, five for the phrase "software-defined networking or storage or compute"). And **forty-five** mention the label that replaced it. The repository is not ignorant of the technology; it is fluent in everything except the idea that binds the technology together, and it is fluent in the marketing category that displaced the idea.

Meanwhile the two adjacent shelves are already full. The physical layer is owned: [data_center_guide.md](data_center_guide.md) runs to 706 lines, 14 numbered sections, all facilities, none about software. The virtualisation shelf is owned: the head-to-heads run to 737 lines and (for the CloudStack comparison) 1,088 lines. But those guides **assume** the concept rather than explain it. A reader who wants to know what "software-defined" actually means, what it hides, what it cannot hide, and why the resulting estate fails the way it fails, has nowhere to go.

So this guide owns the abstraction layer and nothing else. That is a narrow claim, and the narrowness is the point: the concept is not explained by adding another product review to a shelf of product reviews.

### 1.2 The decoder

Nine words carry the whole argument, and several of them are used loosely enough in the market to mean their opposite. Fixed for the length of this guide:

- **SDDC (software-defined data center)** - a data centre in which compute, storage, networking and the management of all three are exposed as software services behind an API, so that the estate's shape is declared rather than assembled. The term is a vendor's framing from 2012 (section 1.3), not a standard.
- **Abstraction** - a boundary that hides the layer beneath it and presents a simpler, uniform surface above it. An abstraction is honest when it hides *irrelevant* detail and dishonest when it hides *relevant* detail. Most of this guide is about telling the two apart.
- **Control plane** - the part of the platform that decides what exists and where: it receives a request, chooses a host, allocates storage, assigns an address, records the result. It is where the inventory, the policy and the permission model live.
- **Data plane** - the path the actual work takes: instructions to a CPU, blocks to a disk, packets between workloads. The control plane configures the data plane; it does not carry its traffic.
- **Declaration** - a written statement of intended state ("three replicas of this volume, on hosts in different racks, reachable at this name") that the platform is expected to make true and keep true. A declaration is the atomic unit of a software-defined estate, and the reason section 14's audit table has a column called "Kind".
- **Tenancy** - the mapping from a workload to the legal or organisational entity that owns it. In a virtualised estate this mapping is metadata, and metadata is only as good as the control plane that stores it.
- **HCI (hyperconverged infrastructure)** - a packaging pattern in which compute, storage and the storage network are delivered as one cluster of identical nodes running one distributed software layer, rather than as separate server, array and switch tiers.
- **SDS (software-defined storage)** - storage whose placement, protection and tiering decisions are made by software across a pool of generic devices rather than by a dedicated array controller.
- **SDN (software-defined networking)** - networking whose topology and policy are defined in software (an overlay, a plugin, a program) rather than by physical cabling and switch-by-switch configuration.
- **Private cloud** - the procurement category that replaced SDDC in vendor marketing (section 11). It describes where the tin is (yours, not a hyperscaler's), not what the control plane does. That distinction matters more than it looks.

### 1.3 Provenance: 2012, 2013, 2023, 2026

The term has a birth certificate, a hostile reception, a corporate rename, and a current status that is stranger than either. Four dated beats:

| Date | What happened | Status of the term |
|---|---|---|
| 8 May 2012 | A vendor CTO names the pattern at a trade show | Coined, as marketing |
| 13 May 2013 | Trade press calls it "nonsense", "marketecture", "software-defined hype" | Contested |
| 22 November 2023 | The originator's new owner renames the category | Retired by its owner |
| November 2026 | Product literature keeps the adjective, drops the acronym | Survives as an adjective |

**2012 - the origin point.** Steve Herrod, then VMware's CTO, posting from Interop in Las Vegas on 8 May 2012, described "the opportunity for IT to move away from data centers and infrastructure defined by the devices and the hardware that we buy, and towards what we are calling the 'Software-Defined Datacenter.'" ✅ **Verified** (Steve Herrod, "Interop and the Software-Defined Datacenter", cto.vmware.com, 8 May 2012, via Wayback capture web.archive.org/web/20140116202230). The post is explicit about the mechanism, and the mechanism is a sequence: virtualisation starts by **abstracting** the logical view from the hardware, then comes **pooling** many servers into one pool, then **automation** speeds operations and improves reliability; the same treatment is then extended from compute to storage, and finally to networking and security through VXLAN and OpenFlow.

Two things follow from reading the primary source rather than the summaries. First, the pattern was always a *sequence*, not a product: abstraction, then pooling, then automation. Second, the term was a conference framing by a vendor executive - "what we are calling" - not the output of a standards body, and never ratified by one. Every later argument about what SDDC "really means" is an argument about a phrase one company chose in a keynote season.

**2013 - the reception.** The ambiguity was present from the start, and the contemporary record says so. Wikipedia's article on the subject describes SDDC as "a marketing term" and records that by 2013 critics were calling the concept "nonsense", "marketecture" and "software-defined hype" ✅ **Verified** (Wikipedia, "Software-defined data center", citing CRN, 13 May 2013; retrieved 28 November 2026). This is worth stating carefully: the criticism is trade-press commentary, not peer-reviewed analysis, and it is quoted here as evidence of *contested reception*, not as a verdict on the architecture. But it establishes that the scepticism was not invented later by people who failed to understand the term. The people who watched it launch thought it was partly a sales device, in the same year.

**2023 - the rename at the origin.** Broadcom's release announcing completion of the VMware acquisition, dated 22 November 2023, says the company "will invest in VMware Cloud Foundation, the software stack that serves as the foundation of private and hybrid clouds" ✅ **Verified** (Broadcom, "Broadcom Completes Acquisition of VMware", 22 November 2023). The release does not use the string "SDDC" at all. On the first day of new ownership, the company that coined the term reclassified its flagship stack as "private and hybrid cloud". That is not a technical event; it is a repositioning of the buying motion.

**2026 - the adjective survives, the acronym does not.** As of 28 November 2026, the VMware Cloud Foundation product page headlines "Private Cloud Platform" and "Modern Private Cloud". The VCF 9.1 datasheet, dated November 2026, describes the product as "a full-stack, private cloud platform" and "a unified private-cloud platform with integrated enterprise-class compute, storage, networking, Kubernetes, management, and security". Neither uses "SDDC". ✅ **Verified** (VCF product page, retrieved 28 November 2026; VCF 9.1 datasheet, November 2026).

But both still use "software-defined" as a lowercase adjective. The datasheet's own phrasing is "advanced software-defined compute, storage, and networking" ✅ **Verified** (VCF 9.1 datasheet, November 2026). That is the single most diagnostic observation in this guide: **the adjective survived and the acronym did not.** The technique is still how the product works. The noun phrase that named the technique is gone from the marketing, because the noun phrase was never describing the technique - it was describing a purchase.

### 1.4 The thesis

> **An SDDC is not a data centre with virtualisation in it; it is a data centre whose control plane is the asset - which is why its failures are governance failures and its economics are licence economics.**

That sentence is the spine of the guide and its closing line. Everything between exists to earn it with mechanism and evidence rather than assertion.

The claim has two halves, and they are connected by a single move. The first half says where the value sits: not in the servers, which are substitutable commodities, but in the software that decides what runs where, who may ask, and what the result must look like. The second half says what follows: if the asset is the decision-making software, then (a) the way it breaks is by making bad or unauditable decisions, which is a governance problem, and (b) the way it is charged for is by licensing that software, which is where the estate's cost curve actually comes from.

### 1.5 The concession - what the headline overstates

The headline, read carelessly, says "software defines everything". It does not, and pretending otherwise would make the rest of the guide useless to anyone who has ever carried a replacement disk.

**A disk still dies. A switch still drops a link. A storage tier still saturates. A power supply still fails.** These are physical failures. No control plane governs them. A hypervisor cannot make a bad disk good; it can only notice the bad disk later than a RAID controller would, or migrate away from it faster, or fail to notice at all. An overlay network cannot stop a physical port from flapping. A declarative API cannot cool a hot row.

So the headline overstates if it is read as a claim about physics. Physics is undefeated. The facilities guide owns that layer precisely because it never went away: [data_center_guide.md](data_center_guide.md) section 5 and section 6 are still about electricity and heat, and no amount of abstraction has moved a single BTU.

### 1.6 The amended thesis

The defensible version is narrower, and it is the version this guide argues:

> **The decisions that define the data centre become declarations, and a declaration is a governance artefact.**

What changed is not that physics stopped mattering. What changed is that the *decisions* moved - from being made by hands on hardware, one box at a time, recorded in a build sheet and a person's memory, to being written down, versioned, reviewed and enforced by software. A rack-and-stack decision lived in a ticket and died with the ticketing system. A declaration lives in a repository, has an author, has a diff, and can be produced in an audit two years later.

Three consequences follow, and each gets a section:

- Because the decisions are declarations, the failures that matter are **failures of declaration** - drift between what is written and what is true, a workload that leaks out of the abstraction, a tenancy that was never declared at all (section 9).
- Because the declarations must be read and enforced by something, the thing that reads them becomes the estate's single point of both control and failure (section 3, section 6).
- Because that thing is licensed software, the bill is a bill **for the thing that reads the declarations**, not for the hardware underneath (section 7).

### 1.7 The boundary, stated once

This guide is the abstraction layer, and it stays inside that boundary:

- **Against [data_center_guide.md](data_center_guide.md):** every facility fact - power, cooling, PUE, DCiE, CUE, WUE, Uptime tiers, racks, physical security, certification, sustainability, colocation economics - is that guide's, cross-referenced by section number and never restated.
- **Against the six head-to-heads:** every product comparison is theirs. Section 8 places vendors as structural positions with a one-paragraph bet each, then hands the reader to the guides that actually compare them, and says in terms that this guide does not.
- **Against the banking guides:** the regulatory argument in section 10 is generic. No institution is named in it, and the only bank in any worked example is the fictional Cymbal Bank of section 13.

---

## 2. The Four Abstractions

An SDDC is usually described as four abstractions stacked into one estate: compute, storage, network, management. The useful way to hold them is not "what do they do" but **what do they hide, what can they not hide, and how does the hidden thing come back**. Each of the four blocks below is symmetrical, because the failure pattern is symmetrical: every abstraction leaks at exactly the boundary it was built to erase.

### 2.1 Compute - the hypervisor, then the container

**What it hides.** Which physical host a workload is on, how many cores that host has, how much of them is idle, and which of several identical servers is doing the work. Above the abstraction there are only virtual CPUs, virtual memory and a name. This is the oldest and most complete abstraction in the estate, which is why it is the one people trust most and inspect least.

**What it cannot hide.** Two things physically refuse to be hidden. The first is **CPU steal** - the time a virtual CPU was runnable but the physical CPU was busy with someone else. Steal is not a bug in the abstraction; it is the abstraction working as designed, and it is visible inside the guest as unexplained slowness. The second is the **NUMA boundary**: memory attached to one socket is not equally distant from a core on another socket, and no scheduler can make distance disappear. A guest that straddles a NUMA boundary pays for it no matter how elegant the declaration was.

**What leaks.** Both of the above surface as operational artefacts with names: CPU **pinning** (tying a virtual CPU to a physical core, which is an admission that the scheduler's default placement is wrong for this workload), **hugepages** (pre-allocating large memory pages, which is an admission that the guest's memory access pattern cannot tolerate the abstraction's default), and reservations. Every one of those settings is a place where the abstraction was partially withdrawn, in writing, by an operator who did not trust it. The container abstraction repeats the pattern one level up: it hides the host operating system, and then leaks as cgroup limits and as the kernel version the container actually got.

### 2.2 Storage - SDS

**What it hides.** Which spindle, which flash device, which controller, and which shelf. Above the abstraction there is a volume with a size and a policy attached to it. The workload does not know whether its blocks are on one device or spread across twenty, and does not need to.

**What it cannot hide.** A **queue depth** is a physical property of a device: once the queue is full, further requests wait, and the wait is observable from above. A **rebuild** is a physical process with a physical duration: when a device is lost and the software re-creates its data elsewhere, everything else on the same pool competes with the reconstruction. Neither is a software decision, and both arrive as latency that the abstraction promised not to have.

**What leaks.** Tiering and placement policy, and noisy-neighbour latency. The moment an operator attaches a policy that says "this volume must stay on flash", they have reached through the abstraction and named a device class - a legitimate, documented withdrawal, and still a withdrawal. The moment a workload's latency rises because a neighbour is rebuilding, the abstraction has hidden contention it could not remove. In an HCI cluster this is sharper than in a dedicated array, because compute and storage share the same nodes: the rebuild is eating the CPU that the workloads also need.

### 2.3 Network - SDN

**What it hides.** Cable, port, VLAN identifier and physical path. Above the abstraction two workloads are on the same virtual network and can address each other by name, and nobody had to touch a switch. This is the abstraction that changed the most between 2012 and 2026, because it moved from an overlay bolted onto a hypervisor to the native fabric of a Kubernetes cluster: the **CNI** (container network interface) plugin, and increasingly **eBPF** programs loaded into the kernel to implement policy and load balancing without a separate virtual switch hop.

**What it cannot hide.** **MTU** - the largest frame a path will carry - is a property of the physical links and every device in between, and an overlay adds encapsulation headers that consume part of it. **Asymmetry** is a property of routing: if the reply takes a different path than the request, a stateful device in the middle will drop it. And a **stateful middlebox** - a firewall that remembers flows, an intrusion-prevention device, a load balancer - violates the abstraction's premise that the network is a set of endpoints, because it is a participant that must see both directions.

**What leaks.** MTU and PMTU blackholes (a connection that establishes and then stalls on the first large packet, which is the classic symptom of an encapsulation that did not account for the underlay), hairpins (traffic leaving a host and returning to the same host through a physical appliance, which is the architecture admitting it needed a middlebox the overlay could not provide), and the asymmetric-routing ticket that takes three teams to close. Every one of these is the network abstraction failing at a boundary it drew in software but did not own in hardware.

### 2.4 Management - the control plane

**What it hides.** The inventory. Above the abstraction there is a catalogue: this many clusters, this many volumes, this many networks, these permissions. Nobody counts hosts by walking the racks, and nobody maintains a spreadsheet of what exists.

**What it cannot hide.** **Its own outage.** Every other abstraction hides something beneath it; the management abstraction hides the inventory, and when the management layer is down the inventory is unavailable while every workload keeps running perfectly. That is the strangest failure in the estate: everything healthy, nothing changeable (section 9, failure mode d).

**What it leaks.** The fact that **the API is now the only legitimate way to change anything**. That is not a defect; it is the design. But it converts every out-of-band change - a console session, a hand-edited configuration file, a switch changed by the network team on a Saturday - into drift. The management abstraction's leak is not a technical artefact like steal or MTU. It is an organisational one: the moment the API is authoritative, every human who bypasses it is creating a difference between the declaration and the truth.

### 2.5 The symmetry, stated

Four abstractions, one pattern. Each hides a physical fact that is genuinely irrelevant most of the time; each cannot hide a physical fact that becomes relevant under stress; each leaks that fact back as a *named operator action* (pinning, a placement policy, an MTU setting, an out-of-band change). The abstraction is not a lie. It is a contract with an exceptions clause, and the exceptions clause is written by whoever is on call at three in the morning.

### 2.6 The four, side by side

| Abstraction | Hides | Cannot hide | Leaks as |
|---|---|---|---|
| Compute | Which host, how much is idle | CPU steal, NUMA boundary | Pinning, hugepages, reservations |
| Storage | Which device, which shelf | Queue depth, rebuild | Placement policy, neighbour latency |
| Network | Cable, port, VLAN | MTU, asymmetry, middlebox | PMTU blackholes, hairpins |
| Management | The inventory | Its own outage | Out-of-band change becomes drift |

Read the last column as a list of confessions. Every entry is a place where an operator, or an architect, wrote down in configuration that the abstraction's default was not good enough - and each of those places is where the estate's real behaviour is decided. An estate can be described completely by its declarations plus its exceptions, and the exceptions are usually the more interesting document.

### 2.7 A stack is not a hierarchy of trust

One more structural point before leaving this section. The four abstractions are stacked, but they are not trusted equally, and the difference is not technical - it is historical. Compute is trusted completely, because it is thirty years old and everyone has been burned by it and recovered. Storage is trusted with conditions, because the conditions are visible as policies. The network is trusted least, which is why it attracts the most tooling and the most specialists. Management is trusted least of all in practice, even though it is trusted most in design - which is why so many estates keep a spreadsheet beside the platform that is supposed to have replaced the spreadsheet.

That asymmetry predicts where the incidents will be blamed (section 6.3) and where the governance effort will be spent (section 9). It also explains a common design error: teams build the abstraction they trust most to the highest standard and leave the one they trust least in its default state, which is the same as leaving the estate's change control in its default state.

---

## 3. The Control Plane Is the Asset - and It Moved

### 3.1 Where the asset sat in the hypervisor era

For roughly the first fifteen years of enterprise virtualisation, the estate had an obvious centre of gravity, and it was not the hypervisor. The hypervisor was small, boring, installed once and rarely touched. The centre was the hypervisor's **manager**: the server that held the inventory of what existed, the policy that said where it was allowed to run, and the permission model that said who could ask for it.

That is why the management plane was treated as the crown jewel. It held three things at once, and losing any one of them was worse than losing a host:

- **The inventory** - the only authoritative record of what the estate contained. Lose it and you do not lose workloads; you lose the ability to describe your own estate, which is worse.
- **The policy** - affinity rules, resource pools, storage profiles, network placement. The estate's behaviour under stress was encoded here, not in the workloads.
- **The permission model** - who may create, who may destroy, who may only look. This is the estate's identity system in practice, whatever the directory service says.

A host could fail and the estate would heal. The manager could fail and the estate would keep running while becoming unmanageable, which is the failure shape that section 9 calls "everything healthy, nothing changeable". The asset was always the thing that knew what the estate was.

### 3.2 Where it sits now - and the move is real

The control plane has not disappeared; it has changed address and, more importantly, changed *kind*. The modern orchestrator does not merely record state; it holds the **declared** state as its primary data, and reconciles reality toward it.

The clearest evidence that virtual machines have been pulled into that model is KubeVirt. Its user guide states that "With the installation of KubeVirt, new types are added to the Kubernetes API to manage Virtual Machines" and that "Every VirtualMachineInstance object represents a single running virtual machine instance" ✅ **Verified** (KubeVirt user guide, "Virtual Machine Instances", kubevirt.io, retrieved 28 November 2026). The project page lists v1.10.0 released 18 November 2026 and a "10 Years of KubeVirt" post dated 10 November 2026, with adopters including Red Hat, SUSE, Oracle, Microsoft, Equinix, Cloudflare, NVIDIA and CoreWeave ✅ **Verified** (kubevirt.io project page, retrieved 28 November 2026).

Read that carefully, because the significance is not "KubeVirt runs VMs". The significance is a category change: **the virtual machine became an API object**. Once a VM is an object in the Kubernetes API, then the thing that stores that API - its database, its watch mechanism, its admission control, its role-based access, its audit log - is the control plane for virtual machines too. The declaration is no longer a description of a VM that some other system will build. The declaration *is* the VM's record of existence, and a controller's whole job is to make the world match it.

The consequence is the sentence this section exists to defend: **whoever owns the API owns the estate's change control, its audit trail and its identity model.** Not metaphorically. Concretely:

- **Change control** - a change is a write to the API. If it is not a write, it is drift, and drift is invisible until it causes an incident.
- **Audit trail** - the API server's log is the record of who asked for what. A change made around the API has no entry, so the audit trail is complete only to the extent that the API is the only path.
- **Identity model** - the API's authentication and authorisation decide who may act. In a Kubernetes-native control plane that means the cluster's identity integration *is* the estate's permission model, whether or not the security team thinks of it that way.

### 3.3 The honesty paragraph - the migration is partial

The claim that the control plane "moved" is true as a direction and false as a completion, and the difference matters enough to state flatly.

**The migration is partial.** In 2026 the majority of enterprise virtualised estates still run on hypervisor-native managers - a vSphere estate managed by vCenter, or an HCI estate managed by the vendor's own console - and the Kubernetes API plays no part in how those VMs are created, placed or audited. KubeVirt is an **add-on**, not a replacement for a hypervisor's management plane: it runs on top of a Kubernetes cluster, it needs that cluster to exist, and the VMs it manages are a subset of an estate that usually has a much larger conventional footprint beside it.

Three reasons the migration stalls, and none of them is technical elegance:

- **Installed base.** An estate built over a decade has its inventory, its policy and its permission model already encoded in the incumbent manager. Moving the control plane means migrating all three, and the third - permissions - is the one nobody wants to touch, because getting it wrong is a security incident rather than an outage.
- **Certification.** A platform is only usable in a regulated estate if its vendor's support matrix, its version compatibility and its audit evidence are already accepted by the auditors. A new control plane starts that conversation from zero.
- **Operational skill.** The people who can debug a hypervisor manager are common. The people who can debug a Kubernetes control plane *and* a virtualisation stack underneath it are not, and the second group is smaller than the first by a wide margin.

So the defensible statement is directional: the centre of gravity has moved toward API-declared state, the move is visible and documented, and it is nowhere near complete. Anyone who tells you the hypervisor manager is obsolete is describing a roadmap, not an estate.

### 3.4 What would falsify this claim

A claim worth defending is a claim that can be wrong. This one would be falsified by any of the following:

- If estates that adopted an API-declared control plane showed **no** difference in change-failure rate or audit effort compared with estates managed through a hypervisor console, then owning the API would not be owning change control in any meaningful sense.
- If the audit trail that regulators actually accept were routinely reconstructed from configuration-management databases rather than from the control plane's own log, then the control plane would be a convenience, not the evidence.
- If enterprises that moved VMs under a Kubernetes API kept a separate, authoritative permission model outside it, then the identity claim in section 3.2 would be false for them.

None of those has been tested here. They are the questions a reader should hold, and section 15 says plainly that the evidence for the *magnitude* of the shift does not exist in this research.

---

## 4. The Architecture, Layer by Layer

Six layers, bottom to top. The purpose of the walk is not completeness - each layer has whole guides of its own - but to locate exactly where the abstraction's guarantee stops and the workload's responsibility begins.

### 4.1 Layer 1 - physical

Servers, disks, switches, racks, power and cooling. This layer is owned by [data_center_guide.md](data_center_guide.md) and is not described here: its section 2 covers facility anatomy, section 5 power, section 6 cooling, section 7 layout and racks, section 8 physical security. What matters for this guide is only the interface: the physical layer presents homogeneous x86 compute, block and file storage, and Ethernet, and it presents them with physics attached - heat, failure rates, cable lengths, port counts.

### 4.2 Layer 2 - virtualisation

The hypervisor turns one physical server into many virtual ones and, in doing so, creates the first pool: capacity that can be assigned rather than bought per application. Containers repeat the move one level up, pooling the operating system rather than the hardware. This layer's product is *elasticity within a host* and *isolation between tenants on that host*.

### 4.3 Layer 3 - software-defined storage and network

Two abstractions that share a shape: take many devices, present one pool, and make placement a policy decision. Storage pools devices into a datastore or a storage class; the network pools links into an overlay or a virtual fabric. In an HCI design these two are fused with layer 2 into a single cluster, which is why HCI is best understood as a *packaging* decision rather than a separate architecture.

### 4.4 Layer 4 - orchestration and control plane

The layer that holds the declaration and reconciles toward it: inventory, placement, policy, permissions, and the API. This is the asset of section 3. It is also the layer with no redundancy of *meaning*: you can run two instances of it, but you cannot run two different versions of the truth.

### 4.5 Layer 5 - platform services

What the platform offers above raw infrastructure so that a workload does not rebuild it: load balancing, DNS, certificate issuance, secrets, identity, logging, metrics, backup hooks, a Kubernetes runtime, sometimes a database-as-a-service or a message queue. These are the services that make a private cloud feel like a cloud, and they are also where a platform team's real product lives.

### 4.6 Layer 6 - workload

The application, its data, its configuration and its own operational logic. Everything above is a supplier to this layer; nothing above can make it correct.

### 4.7 The boundary - what the platform guarantees versus what the workload still owns

This table is the concession of section 1.5 given technical teeth. Each row is a distinction drawn from mechanism, not a vendor commitment; no vendor in this guide promises the left column in these words.

| The platform guarantees | The workload still owns |
|---|---|
| That the workload is placed on a host with capacity, and restarted if that host fails | That restarting is safe - that the workload is idempotent and can tolerate a duplicate start |
| That a virtual network exists between two workloads, with the addresses declared | Transport security and application authorisation: TLS, certificates, who may call which endpoint |
| That a volume is attached to the workload at the declared size and policy | Backup consistency: that the data captured is a coherent point in time, not a torn write |
| That the platform's own identity system authenticates the operator | Data-level authorisation: which row, which field, which customer, for which user |
| That capacity exists in the pool, within the declared quota | That the workload's demand profile fits the quota it was given |
| That the platform will be upgraded on a supported path | That the application remains compatible with the upgraded platform |
| That a name resolves through the platform's DNS | That the certificate behind that name is valid, rotated and trusted |
| That hosts agree on time to within the platform's configuration | That the application's ordering, locking and replay logic survives a clock adjustment |

The pattern in the table is the guide's whole argument in miniature. The platform guarantees **existence, placement and connectivity**. The workload owns **correctness, consistency and identity of data**. Every incident that is blamed on "the platform" and turns out to be a workload defect is a row where someone read the left column and assumed the right.

---

## 5. What the Abstraction Buys, Honestly

Four benefits are claimed for the software-defined estate. Each is real. Each has a limit that the sales conversation omits, and the fourth limit is the sharpest.

### 5.1 Elasticity

**What it delivers.** Capacity becomes a pool rather than a set of allocations. A workload can be given more of something without a purchase order, and the pool can be re-divided as demand moves. This is genuine and it is the benefit that started the whole movement in 2012.

**The limit.** Elasticity is bounded by the pool's physical size, and the pool was bought against a forecast. Above the pool there is no elasticity at all, only a queue. An estate that has pooled everything and filled the pool is exactly as rigid as an estate that never pooled anything - it is just rigid in one place instead of many, which is better for operations and identical for a workload that needs capacity the pool does not have.

### 5.2 Reproducibility

**What it delivers.** A declared environment can be created again: the same topology, the same policy, the same network shape, from the same text. This is the benefit that makes platform engineering possible, and it is the one most often overstated.

**The limit, stated plainly: a declaration reproduces the shape of an environment, not its state.** The declaration says three replicas, this network, this size, this policy. It does not contain the data inside the volume, the identities of the users who will touch it, the external services it calls, the DNS records that point at it, the certificates that prove it is itself, or the time it believes it is. Those live outside the declaration, in systems the declaration does not own, and they are exactly the things that make a rebuilt environment not the same environment. A team that has tested "we can rebuild from the repository" has tested the shape. Whether the rebuilt thing *works* depends on five dependencies that were never in the repository.

### 5.3 Policy enforcement

**What it delivers.** Rules about placement, isolation, retention and access can be written once and applied to everything, and - crucially - a violation can be *prevented* rather than detected later. This is the benefit that matters most to a regulated estate, and section 10 argues it is the real reason such estates buy the abstraction at all.

**The limit.** A policy is only enforced at the point where change happens. Anything created before the policy existed, or created around the API, is outside it. Policy enforcement therefore degrades with the age of the estate and with the number of paths into it - which is why the guardrail for drift (section 9) is not a better policy engine but a smaller number of doors.

### 5.4 Metering and chargeback

**What it delivers.** Because the platform allocates capacity, it can record who was allocated what. That record is the input to showback or chargeback, and it is the only honest way to make a shared estate's consumers behave like consumers.

**The limit.** The meter measures **allocation**, not value, and usually not even consumption. A workload that reserved capacity and uses a fraction of it is billed for the reservation, which is correct for capacity planning and wrong for any argument about efficiency. Worse, the meter's unit is whatever the licence counts, so the chargeback model inherits the vendor's commercial model rather than the business's cost model - the subject of section 7. Chargeback that nobody believes is an anti-pattern in its own right (section 12).

### 5.5 The honest summary of the four

Elasticity is real but bounded by the pool. Reproducibility is real but shallow - it reproduces shape, not state. Policy enforcement is real but only at the door. Metering is real but measures the vendor's unit. None of these limits argues against the abstraction; they argue against expecting the abstraction to be the thing that makes an estate correct.

### 5.6 What the business case assumes versus what the platform delivers

| Benefit | The assumption in the business case | What the platform actually delivers |
|---|---|---|
| Elasticity | Capacity appears when demand does | Reallocation inside a pool somebody already bought |
| Reproducibility | Environments that are the same | Environments with the same shape and different state |
| Policy | Compliance | Compliance for changes that pass through the door |
| Metering | Cost truth | Allocation, expressed in the licence's unit |

The table is not an argument that the benefits are fake. It is an argument that each one is **conditional**, and that the condition is stated in the right-hand column in words a business case never uses. A programme that writes its benefits as the left column will be measured against the right column at the first post-implementation review, and the gap between them will be attributed to the technology rather than to the sentence.

### 5.7 The benefit that is real and never claimed

One benefit is genuinely delivered and almost never appears in a business case, because it is not a benefit to the business - it is a benefit to the *evidence*. A declared estate can be described, diffed and audited in a way an assembled one cannot (section 10.1). Regulated institutions discover this after the fact, when an auditor asks a question that the old estate could only answer with testimony, and the answer turns out to be a query. It is the strongest argument for the abstraction in this guide, and it is the one vendors sell least, because it is a benefit to the buyer's control function rather than to the buyer's speed.

---

## 6. What It Costs

The abstraction is not free, and its costs are not the ones in the business case. Four of them recur in every estate that has actually run one.

### 6.1 A new failure domain - the control plane itself

The estate gains a component that did not exist before: the thing that holds the declaration. It is a distributed system - an API server, a database, a set of controllers, a reconciliation loop - and it fails like a distributed system, which is to say partially and confusingly.

The characteristic failure is not an outage. It is **everything healthy and nothing changeable**. Every workload is running, every disk is serving, every user is connected, and no one can create, resize, move or delete anything, because the component that accepts the request is unavailable. That state is worse than an outage for two reasons: it is invisible to the monitoring that watches workloads, and it is invisible to the business until the first change is needed, which is usually during an incident when speed matters most.

The control plane also concentrates risk in a way the physical estate never did. A rack failure costs you the workloads in that rack. A control-plane failure costs you the ability to act on *every* rack at once - the blast-radius discussion in section 10.

### 6.2 Skill inversion

The deepest expertise in the estate moves. In a hand-built estate the person who mattered most knew how a RAID controller behaved under a degraded battery, how a switch's spanning-tree topology would reconverge, and which of the twelve identical servers was the one with the flaky DIMM. In a software-defined estate the person who matters most reads an API, writes a declaration, and understands the reconciliation loop.

This is not a complaint about progress; it is a staffing consequence with a clock on it. The first kind of knowledge is **not written down anywhere** - it lives in people - and it retires. The second kind is written down (that is the point), which means it can be taught, but it is also a different discipline that the existing team did not sign up for. An estate that adopts the abstraction without replacing the first kind of expertise ends up with neither: the hardware experts have gone, and the API experts were never hired.

### 6.3 The debugging tax

When something is slow, five layers can each be the cause: the workload, the platform services, the orchestrator, the storage or network abstraction, and the physical device. Each layer produces telemetry, and - this is the tax - each layer's telemetry is owned by a different team, stored in a different tool, and retained for a different period.

So the incident is not hard because the cause is obscure. It is hard because assembling the evidence requires four people from four teams to look at four systems with four different time bases, and the one who can correlate them is the one who has been in the estate longest and is therefore also the one in every other incident. The abstraction did not create this; it multiplied the number of layers that can plausibly be blamed, and it did so at the same time as it removed the shared physical reference point (the same rack, the same switch, the same array) that used to let people argue productively.

### 6.4 Licence concentration

The fourth cost is commercial rather than operational, and it is large enough to have its own section. Because the bundle that provides compute, storage, network and management is bought as one thing from one vendor, the buyer loses the ability to substitute at the layer that gets repriced. section 7 works the mechanism; the summary here is that the abstraction's cost structure is not a hardware cost structure with software on top. It is a software cost structure with hardware underneath.

### 6.5 The costs, ranked honestly

If a reader takes one ranking away: the debugging tax is felt first, the skill inversion is felt second, the new failure domain is felt at the first serious incident, and the licence concentration is felt at the first renewal - which is precisely why it is the one that gets least attention during the design, when everyone is looking at the first two.

### 6.6 The cost that never appears in the business case - decision latency

The estate becomes much faster at the change it has done before and slower at the change it has not. A change that fits an existing pipeline is produced in minutes; a change that does not fit - a new workload class, a one-off tenancy, an exception to a placement policy - must go through design, policy approval, and an amendment to the declaration itself. That path is longer and more deliberative than the old ticket ever was, and it is longer *by design*, because the deliberation is the point.

This is a real cost, and it is systematically absent from business cases, because the case is written by comparing the platform's best case (a repeated change) against the old world's average case (any change). The comparison flatters the platform every time.

The second-order effect is the one to watch. When the declared path is slow and the console is fast, someone will use the console "just this once", because the business needs the thing on Friday. That is not disobedience; it is a rational response to an incentive the platform created. It is also exactly how drift begins (section 9.3) - which is why the guardrail for decision latency is not a stricter policy but a faster legitimate path for the exception, so that the approved route is never slower than the unapproved one.

---

## 7. The Economics Are Licence Economics

No figures appear in this section by design. The subject is the **mechanism** by which a software-defined estate's cost behaves, because the mechanism is what a reader can apply to their own contract, while any number printed here would be wrong within a quarter and would turn the section into a price list.

### 7.1 Subscription versus perpetual - what each does to the vendor's revenue

A perpetual licence sells a right to use a version, once. The vendor's revenue arrives at the sale and then must wait for the refresh cycle - the point at which the customer's hardware is old enough, or the version far enough behind, that buying again is easier than not. Perpetual licensing therefore **defers** the vendor's revenue to a rhythm the customer controls, and the customer's cost is lumpy: a large capital event every few years, with a support charge in between.

A subscription sells access for a period. The vendor's revenue arrives continuously, and the customer's cost becomes smooth - but the smoothness hides a transfer of control. Under perpetual, the timing of the next purchase is the customer's decision. Under subscription, the timing of the next payment is the calendar's, and the amount is a function of the estate's size at that moment. **Subscription pulls the vendor's revenue forward and makes the estate's cost a function of the estate's size rather than of its use.** That is the whole mechanism, and it is not a criticism: it is a trade the buyer makes, usually in exchange for support, currency and the ability to stop paying.

### 7.2 Concentration - why substitution fails at the repriced layer

The licence-concentration argument does not depend on any vendor behaving badly. It depends on the shape of the bundle.

In an integrated stack, compute, storage, networking, management and a Kubernetes runtime are one product from one supplier - which is exactly how the current VMware Cloud Foundation bundle is documented: vSphere for compute, vSAN for storage, NSX for networking, vSphere Kubernetes Service, VCF Operations and VCF Automation, plus Private AI services, with security, compliance and recovery sitting in separately purchased advanced-services add-ons ✅ **Verified** (VCF 9.1 datasheet, November 2026). The same datasheet lists four purchase paths - direct from Broadcom, through channel partners, as OEM integrated systems, and as a subscription service from a public cloud service provider ✅ **Verified** (VCF 9.1 datasheet, November 2026).

Now apply the substitution test at each layer:

- **Below the bundle, substitution works.** The hardware is x86 servers, Ethernet switches and flash devices from a competitive market. If a server vendor raises its price, the buyer changes vendor and the abstraction does not notice.
- **Inside the bundle, substitution fails.** There is no market price for "the storage layer of this bundle" separate from the bundle, because the storage layer is not sold separately in a way that preserves the management plane above it. The buyer can replace the whole stack or none of it.
- **Above the bundle, substitution is expensive.** Workloads, automation and skills are written against the bundle's API and its object model. Replacing the bundle means rewriting the declarations.

So the asymmetry is: **the thing underneath is substitutable, the thing above is not, and the thing in the middle sets the price.** Repricing a control plane reprices the estate, and the estate has no competitive response at the layer that was repriced.

### 7.3 The second-order effect - an estate cannot shrink its way out

Because the billable unit tracks **capacity** - the size of the estate the platform is licensed to manage - and not the work the estate performs, a reduction in workload does not reduce the bill. Consolidating twenty workloads onto five hosts reduces the hardware footprint and therefore the hardware cost; it does not reduce the licence, because the licence was never counting workloads.

This produces a perverse operational incentive that shows up in real programmes: the estate becomes reluctant to decommission, because decommissioning frees hardware and frees nothing else. The rational move becomes to fill the licensed capacity, which is the opposite of efficiency. An estate that cannot shrink has a cost floor set by its peak, and the peak was set by a forecast made before the platform was chosen.

### 7.4 The governance effect - the renewal is the architecture review

The final mechanism is the one that connects this section to the guide's thesis. A perpetual estate re-decides its architecture when the hardware is refreshed, which is an operational event that arrives gradually. A subscription estate re-decides its architecture **at renewal**, which is a dated event with a negotiating table attached.

That is not a small difference. At renewal, every question the design deferred comes back at once: whether to stay on the bundle, whether to move a workload class to a hyperscaler, whether the exit that was never tested is still available, whether the declarations are portable. The renewal is, in practice, the only moment at which the estate's architecture is genuinely re-opened - **whether or not anyone in the room intends it as an architecture review**. An organisation that treats renewal as a procurement task has outsourced its architecture decision to a contract negotiation, and will discover that the two are the same meeting.

---

## 8. The Vendor Landscape, Structurally

This section places five positions in the market and states, in one paragraph each, the structural bet that position makes. **It does not compare them.** There is no feature table here, no scorecard, and no ranking. The six sibling guides named at the end of this section run those comparisons, and the reader who wants one should go there.

A position, as the word is used here, is not a company size and not a product tier. It is defined by a single question: **which layer does this supplier insist on owning?** The answer decides almost everything downstream - the cost structure (section 7), the failure shape (section 6), and what the buyer retains the ability to substitute. Two suppliers can ship nearly identical technology and occupy different positions, because the position is a commercial and architectural stance rather than a feature set. That is why the paragraphs below describe bets rather than capabilities.

### 8.1 The integrated bundle - Broadcom and VMware Cloud Foundation

**The bet: that a buyer will pay a premium to have one supplier own the whole stack, and that the supplier can then price the stack as a single object.** Everything in the design follows from that. The components are chosen to be complete rather than best-of-breed at each layer, the management plane is the integration point, and the purchase paths are arranged so that the same bundle can arrive as a licence, as an appliance, or as a service from a cloud provider. The bet's strength is that it removes integration risk from the customer. The bet's cost is that it removes substitutability, which is section 7.2 stated as a business model.

### 8.2 The commercial HCI position - Nutanix

**The bet: that the unit of purchase should be the cluster, not the layer.** An HCI vendor sells nodes in which compute, storage and the storage network are one software-defined system, with the management console included. The structural consequence is that the customer never assembles a stack, so the customer never has to decide where storage ends and compute begins - a decision that consumes enormous engineering effort in a disaggregated design. The bet's weakness is the mirror of its strength: the cluster is the unit of both scaling and pricing, so growth is granular by node and the estate's shape is decided by node economics.

### 8.3 The open-source IaaS assembly - OpenStack and CloudStack

**The bet: that the control plane should be separable from the vendor, so that the buyer can own the software and choose who supports it.** Both projects deliver infrastructure-as-a-service control planes assembled from services or from a single orchestrator, and both are usually consumed through a distribution that supplies the support contract. The structural consequence is that the abstraction is *portable* - the declarations belong to the customer, and the vendor relationship is a support relationship rather than a licence relationship. The bet's cost is that integration is the customer's problem, either directly or through a distribution; see [cloudstack_vs_openstack_guide.md](cloudstack_vs_openstack_guide.md) for the two shapes of that problem.

### 8.4 The Kubernetes-native platform - Red Hat OpenShift, Canonical Charmed Kubernetes

**The bet: that the correct control plane for a modern estate is the Kubernetes API, and that virtual machines, containers and platform services should all be objects in it.** This position inherits the argument of section 3.2 and turns it into a product: the cluster is the estate's control plane, the operator pattern is how platform services are delivered, and virtualisation arrives as an extension (KubeVirt, and its downstream packaging in OpenShift Virtualization) rather than as the foundation. The bet's strength is that it aligns the estate with where application delivery has already gone. The bet's cost is that it asks the operations team to run a distributed control plane as the *primary* system, with the virtualisation stack as a guest inside it - the skill inversion of section 6.2 in its sharpest form.

### 8.5 The cloud-provider abstraction - the hyperscalers

**The bet: that most buyers would rather rent the control plane's output than own the control plane.** A hyperscaler sells virtual private clouds, managed Kubernetes, managed storage and managed identity, and the customer's declarations are written against the provider's API. The structural consequence is that the abstraction is real but **not owned by the customer**: the declaration is portable in principle and the implementation is not, because the provider's control plane is the product. This position is the strongest available answer to section 6 (no operational cost, no control-plane failure domain of your own) and the weakest available answer to section 7.4 and section 10 (the renewal is not yours to schedule, and the evidence for audit is produced by someone else's system).

### 8.6 What this section deliberately does not do

Every one of the five positions above could be expanded into a comparison, and every comparison would be a bake-off that this repository has already published elsewhere. The division of labour:

- **Nutanix against OpenStack** - [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md), which owns the HCI-versus-disaggregated axis, the OpenStack component roster and the commercial-distribution market.
- **CloudStack against OpenStack** - [cloudstack_vs_openstack_guide.md](cloudstack_vs_openstack_guide.md), which owns the single-management-server-versus-integrated-services axis and the CloudStack material in full.
- **Nutanix enterprise AI against OpenShift AI** - [nutanix_enterprise_ai_vs_openshift_ai_guide.md](nutanix_enterprise_ai_vs_openshift_ai_guide.md).
- **Charmed Kubernetes against OpenShift** - [charmed_kubernetes_vs_openshift_guide.md](charmed_kubernetes_vs_openshift_guide.md).
- **Charmed Kubeflow against OpenShift AI** - [charmed_kubeflow_vs_openshift_ai_guide.md](charmed_kubeflow_vs_openshift_ai_guide.md).
- **DevStack, the OpenStack lab** - [devstack_openstack_guide.md](devstack_openstack_guide.md).

Those guides compare products on named axes and name no winner. This guide names vendors only to explain why a *position in a market* produces a particular cost structure and a particular failure shape. If a reader finds themselves scoring two of the five paragraphs above against each other, the section has been misread: the paragraphs describe bets, and bets are not features.

---

## 9. The Failure Modes of the Abstraction

Five ways a software-defined estate fails. Each is given as **Symptom**, **Cause**, **Guardrail**, because a failure mode is only useful if it can be recognised before it is diagnosed. All five are analytic categories drawn from mechanism, not findings from a survey; section 14 records them as inferences.

### 9.1 Failure mode (a) - the leak

- **Symptom.** A workload's design contains a statement about physical placement: it must not share a host with a specific other workload, or it must run only on a particular storage class, or it needs a local device that is not pooled. The declaration has grown a clause about tin.
- **Cause.** The abstraction hid something the workload genuinely needs to know. The classic cases are anti-affinity (two instances of a database must not fail together), licensing that counts sockets on the host, and a database whose log device must be locally attached for its own correctness assumptions. In each case the abstraction's default placement is *wrong for this workload*, and the operator reaches through to correct it.
- **Guardrail.** Treat every placement clause as a **documented exception**, not a configuration detail. Record why it exists, who owns it, and what would let it be removed. An estate with fifty undocumented placement exceptions has an abstraction that is nominally in force and practically withdrawn in fifty places - and nobody can say which fifty.

### 9.2 Failure mode (b) - the noisy neighbour

- **Symptom.** Latency rises for a workload whose own metrics are unchanged, at a time when nothing in its own history changed. Investigation finds a co-tenant doing something heavy on the same shared resource.
- **Cause.** This is the abstraction working exactly as designed. Pooling exists to let workloads share, and sharing means contention is possible. The abstraction hides the neighbour, which is the point, and therefore hides the contention until it becomes an incident. The physical layer did not fail; the pooling did what pooling does.
- **Guardrail.** Contention has to be made **visible above the abstraction**, because it cannot be prevented below it. That means per-tenant monitoring of the shared resources (compute steal, storage queueing, network drops), quotas that are enforced rather than advisory, and an acceptance that "noisy neighbour" is a capacity-governance problem, not a bug to be fixed by a better hypervisor.

### 9.3 Failure mode (c) - drift

- **Symptom.** The declaration says one thing, the estate does another, and the difference is discovered during an audit, an incident, or a rebuild that produces something that does not work.
- **Cause.** A change was made around the API: a console session, a hand-edited file, a vendor support engineer's fix, a script written before the platform existed. The change was reasonable at the time and left no record in the place the declaration lives. Repeat for a few years.
- **Guardrail.** Drift can only be governed by **making the API the only door** and by continuously comparing declared state to actual state, reporting the difference as an exception with an owner. Detection alone is not enough; an estate that detects drift and tolerates it has simply automated its own fiction.

### 9.4 Failure mode (d) - the control-plane outage

- **Symptom.** Every workload is running and serving. Nothing can be created, changed, moved or deleted. The monitoring is green, because the monitoring watches workloads, and the business has not noticed yet.
- **Cause.** The component holding the declaration is unavailable, degraded, or has lost quorum. It is a distributed system with its own failure modes, and it is the one component whose failure does not degrade service but degrades *agency*.
- **Guardrail.** Treat the control plane as a **tier-zero service** with its own availability objective, its own backup and restore test, and its own out-of-band recovery procedure that does not depend on the control plane being up. And rehearse the specific scenario of "everything is fine and we can change nothing", because it is the one nobody rehearses.

### 9.5 Failure mode (e) - the ticket-shaped estate

- **Symptom.** The platform is fully software-defined. The API exists, the declarations exist, the automation exists. And the way a change actually happens is that someone emails a request, a person reads it, and that person clicks something in a console or edits the declaration by hand.
- **Cause.** The abstraction was implemented and the *process* was not. The platform offers a programmable interface and the organisation continues to use a human one, because the human one is how approvals, accountability and politics work. The API is not unused out of ignorance; it is unused because using it would require the organisation to change how it decides things.
- **Guardrail.** This is not fixable in the platform. It is fixed by moving the approval into the change itself - review on the declaration, automated policy checks, an audit trail that the approver's name lands in - so that the ticket becomes the pull request rather than a parallel process beside it.

### 9.6 Which of the five are governance failures

The thesis predicts that most failures in a software-defined estate are governance failures. The five modes test that prediction. The honest answer is that three of the five carry a governance element - two of them primarily, and one through the reach of its blast radius - while two are genuinely technical.

| Mode | Primary nature | Why |
|---|---|---|
| (a) the leak | Technical / architectural | The abstraction's contract was incomplete for this workload; the fix is a documented exception |
| (b) the noisy neighbour | Technical | Contention is physics plus pooling; governance can only make it visible and bounded |
| (c) drift | Governance | The declaration became fiction because nobody owned the difference |
| (d) the control-plane outage | Technical, with a governance blast radius | The component failed technically; how far it reaches is a design and tenancy decision |
| (e) the ticket-shaped estate | Governance | The platform was adopted and the decision process was not |

That is a three-out-of-five, not a clean sweep, and the guide should say so rather than force the thesis. What the table does establish is the direction: the failures that are hardest to fix, and that persist longest, are the ones in the governance column - because a technical failure has a fix and a governance failure has an owner who has not been appointed.

---

## 10. The Regulated and Banking Angle

A regulated institution buys a software-defined estate for a reason that rarely appears in the vendor's material, and it is not elasticity. It is **evidence**.

### 10.1 A declaration is evidence; a hand-built server is a memory

When a configuration is declared in text and enforced by a control plane, then the question "what was this system's configuration on 14 March?" has an answer that does not depend on anyone's recollection: the declaration at that revision, its approval, and the control plane's record of what it applied. That is a different kind of answer from "the engineer says it was not changed", and auditors know the difference.

A hand-built server, by contrast, is a memory. Its configuration exists in the state of the machine, which is now, and in whatever build documentation survived. Proving what it was at a past date requires reconstructing from change tickets and hope. The software-defined estate converts a question of testimony into a question of records, and that conversion - not elasticity - is what a bank is buying when it buys the abstraction.

### 10.2 A shared control plane is a shared blast radius

The physical estate's risk was shaped like its racks: a failure took out what was in the rack, and the rest of the business continued. The software-defined estate's risk is shaped like its control plane: **one API outage is every business line at once**.

That is a genuinely different risk shape, not a bigger version of the old one. A rack failure is an availability event with a boundary. A control-plane failure is an *agency* event with no boundary - it does not take a business line down, it takes away the ability to act on all of them, at the moment when acting matters. A risk register built around facility tiers will not contain it, because no tier classification describes it. The mitigation is structural: separate control planes for genuinely separate risk domains, and an out-of-band path that does not depend on the thing that is broken.

### 10.3 Tenancy and segregation - what a hypervisor does not answer

Regulators ask questions of the form "can you demonstrate that this legal entity's data could not have been placed on shared infrastructure with another entity's?" A hypervisor answers a different question: it isolates memory and CPU between virtual machines. Isolation is a technical property. **Segregation is an obligation.**

The gap between them is metadata. "Which legal entity's data is on this host" is not a fact the hypervisor knows; it is a fact that the control plane's inventory knows, if anyone declared it. So the answer to the regulator comes from the control plane or from nowhere. An estate that retrofits segregation with VLANs has answered a network question and left the tenancy question open - which is the anti-pattern in section 12.6.

Two practical consequences: the tenant identifier must exist as a first-class object in the control plane (not a naming convention in a hostname), and the placement policy must be expressible in terms of it, so that "tenant A never shares a host with tenant B" is a declared rule the platform enforces rather than an operational habit.

### 10.4 Change control and the evidence for audit

Three properties make a software-defined estate auditable in a way a hand-built one is not:

- **Attribution.** Every change has an identity attached, because the change was an API call and API calls are authenticated.
- **Temporal proof.** The declaration's history is a version history, so "unchanged between two dates" is a diff, not an assertion.
- **Prevention over detection.** A policy can refuse a non-compliant change rather than flag it afterwards, which converts a finding into a non-event.

None of the three is automatic. Each depends on the API being the only door (section 9.5), on the version history being retained, and on the policy being written before the change rather than after the audit. An estate with a control plane and no discipline has the *capability* for evidence and produces none of it.

### 10.5 The exit problem - what you can prove you can leave

The question a regulator, a board or an acquiring institution eventually asks is not "does it work" but "can you get out". In a software-defined estate, exit has two components: the data (portable, with effort) and the **declarations** (portable only if they were written against something portable). A declaration expressed in one vendor's object model is not a description of an intent; it is a description of an intent *in that vendor's language*, and translating it is a migration project.

So exit is provable only if it has been tested, and testing it means rebuilding a representative slice of the estate somewhere else from the declarations alone. That test is uncomfortable, because it usually reveals that the declarations were incomplete and the missing pieces lived in an engineer's head - which is exactly the finding worth having before a renewal rather than after.

The `banking/` guides in this repository carry the regulated-sector context in general terms; this section deliberately names no institution, because the argument is structural and holds for any regulated entity with a shared platform.

---

## 11. The State of Practice in 2026

### 11.1 Who still says SDDC

The acronym is not dead. It survives in four places, and the pattern of survival is informative:

- **Vendor glossaries and explainer pages.** IBM still maintains "What is a software-defined data center (SDDC)?" on ibm.com/think, which defines SDDC as extending virtualisation from compute to storage and networking, and which stresses that "The primary hurdle is gaining cross-functional agreement" ✅ **Verified** (IBM, "What is a software-defined data center (SDDC)?", ibm.com/think, retrieved 14 May 2026). Note what the surviving explainer says: the hard part is organisational, not technical. Even the literature that keeps the term has concluded that the term is not the problem.
- **RFP and tender templates.** Procurement documents are copied forward. An RFP written in 2016 and refreshed by editing the dates will still ask vendors to describe their "SDDC strategy", and vendors will answer in the language of the template because that is how the answer gets scored.
- **Documents written years ago and never rewritten.** Architecture decisions, standards, and design documents in regulated estates have long lives and formal change control, which means the vocabulary of 2018 is still authoritative in 2026.
- **Job adverts.** Requisition text is copied from the last requisition. A role advertised as "SDDC engineer" is usually a virtualisation-and-automation role described in the vocabulary of the person who last held it.

### 11.2 What they say instead

Four substitutions, each carrying a different claim:

| Instead of SDDC | The substitute says | What it centres |
|---|---|---|
| Private cloud | The tin is yours | Location and ownership |
| HCI | The cluster is the unit | Packaging |
| Cloud platform | The API is the product | Control plane |
| Platform engineering | The team is the product | Operating model |

### 11.3 What the substitution reveals

The industry did not rename the architecture. It renamed the **buying motion**. "Private cloud" is a procurement category: it tells a buyer what they are purchasing and where it will sit, and it deliberately does not say anything about how the control plane works. That is diagnostic. If the replacement term describes a purchase and not a mechanism, then the term it replaced was probably also doing procurement work - which is consistent with everything in section 1.3: a phrase coined in a keynote, mocked as "marketecture" within a year, and retired by its own owner when the owner changed how it wanted to sell.

### 11.4 The honest counterpoint

There is a real argument that "private cloud" is the **worse** engineering term of the two. "Software-defined data center" at least names the mechanism: software defines the data centre. "Private cloud" names only where the hardware is, and it invites the reader to assume that the properties of a public cloud - elasticity, self-service, metering - come from the privacy rather than from the control plane. They do not. A private cloud without a declarative control plane is a virtualised data centre with a new name on the door, and the name is doing the work the architecture was supposed to do.

So the substitution is a loss in precision and a gain in commercial clarity, which is the most honest thing that can be said about it. The acronym died because it was a sales object; the replacement lives because it is a better sales object; and the architecture underneath has not changed at all.

### 11.5 How to read a document that uses the old word

Three checks are enough, and they take a minute:

- **The date.** If the document predates 2023 and has not been through change control since, it is a historical artefact. That does not make it wrong - the architecture it describes may still be exactly what runs - but its vocabulary is evidence about the year, not about the estate.
- **What it describes.** If the paragraph after "SDDC" is about procurement, tiers and bundling, the word is being used as a purchase category. If it is about declarations, policy and reconciliation, it is being used as an architecture term. Both uses are legitimate and they are not the same use.
- **Whether the reader is assumed to know the mechanism.** A document that says "our SDDC" without ever saying what is software-defined is asking the reader to accept a label as an explanation. That is the failure mode section 12.1 describes, one level up: the vocabulary becomes the theatre.

The reverse check is equally useful. A document that uses "private cloud" and then describes how a change is declared, reviewed and enforced is describing an SDDC in the current dialect, and a reader who rejects it for using the wrong noun has confused the label with the thing.

---

## 12. The Anti-Patterns

Nine patterns that recur in estates that adopted the abstraction. Each is a **name**, then what it looks like and why it happens. None of them is a technology failure; all of them are decisions that were not made.

### 12.1 The declarative theatre

Declarations are written, stored in a repository, reviewed in pull requests - and not enforced. The estate's real configuration is still made by hand, and the declaration is a parallel document that describes an intention nobody applies. It looks like platform engineering and functions as documentation. **Diagnostic:** delete a declaration and see whether anything changes. If nothing does, it was theatre.

### 12.2 The control plane as a pet

One cluster, one control plane, hand-built, carefully nursed, and never upgraded because the person who built it is the only one who understands it and the change window is never long enough. The estate has achieved software-defined infrastructure and reintroduced a pet at the centre of it. **Diagnostic:** ask for the date of the last successful control-plane upgrade and the documented procedure. Two answers that do not match is the pattern.

### 12.3 Abstraction as an excuse

The platform team owns the abstraction; nobody owns the physical layer. Disks are replaced when they fail loudly rather than when they are predicted to fail; firmware is whatever shipped; cabling is whoever was on site. The abstraction is used as an argument that the layer beneath it no longer needs attention, when in fact the abstraction is *more* dependent on that layer, because it pooled it. **Diagnostic:** who is accountable for firmware currency? If the answer is a vendor's support contract, the pattern is present.

### 12.4 The five-layer alibi

Every incident is someone else's layer. The application team blames the platform, the platform blames the network, the network blames the storage, the storage blames the hardware, and the incident closes with an action item for a team that was not in the room. The abstraction made this possible by creating five plausible suspects, each with its own telemetry and its own owner (section 6.3). **Diagnostic:** count how many incidents in the last quarter ended with a cause attributed to a different team than the one that owned the service - and how many of those attributions were ever verified.

### 12.5 Licence-shaped architecture

A design decision that exists because of how the product is licensed rather than because of what the workload needs: a cluster boundary drawn where the contract divides, a management domain sized to a licence tier, a feature declined because it sits in a higher bundle. The architecture is not wrong; it is *shaped by the invoice*, and nobody wrote that down, so the next architect will assume it was a technical choice. **Diagnostic:** for each major boundary in the design, ask "would we draw this line if the licence were flat?" If the answer is no, the boundary is commercial.

### 12.6 The tenancy afterthought

Segregation is retrofitted with VLANs after the platform is built, because the tenancy model was never a first-class object. The result is a network that separates traffic and a control plane that cannot answer which legal entity's data is on which host (section 10.3). **Diagnostic:** can the platform answer "list every host that has ever run a workload for tenant X" without a spreadsheet? If not, tenancy is an afterthought.

### 12.7 The irreproducible declaration

A golden template, a reference architecture, a "standard build" - that nobody can actually rebuild from. The template is a snapshot of a working system rather than a set of instructions that produces one, and the difference is discovered during the rebuild, which is the worst possible time. **Diagnostic:** rebuild a non-production copy from the repository alone, with no access to the original team. If it takes longer than a week or needs a phone call, the declaration is a photograph, not a recipe.

### 12.8 The exit that was never tested

The exit plan exists as a document. It has never been executed, not even for a slice, and it assumes the declarations are portable and the data is exportable. Both assumptions are usually true in principle and false in the specific. **Diagnostic:** the date of the last exit test. If there is no date, there is no exit - only an intention. section 13 shows what this costs.

### 12.9 Metering without a meter

Chargeback is announced, a dashboard is published, and nobody believes the numbers - because the meter counts allocation rather than consumption (section 5.4), or because the unit is the licence's unit rather than the business's, or because the first month's bill was quietly written off. Once the numbers are disbelieved, the whole showback programme is dead and cannot be revived without a re-launch. **Diagnostic:** ask a business unit's technology lead whether they would make a decision based on their chargeback figure. If the answer is a laugh, the meter is decorative.

### 12.10 The decision each anti-pattern is avoiding

The list is long, and a reader cannot act on nine things. It is shorter if each is read as the trace of a decision nobody was willing to make:

| Anti-pattern | The decision that was never taken |
|---|---|
| Declarative theatre | That the declaration is authoritative over the console |
| Control plane as a pet | That the control plane is a service with an owner, a version and an upgrade path |
| Abstraction as an excuse | Who is accountable for firmware, disks and cabling once the layer is pooled |
| Five-layer alibi | Who owns end-to-end causality in an incident |
| Licence-shaped architecture | That the architecture is decided before the contract, not after it |
| Tenancy afterthought | That tenancy is an object in the control plane, not a VLAN in the network |
| Irreproducible declaration | That the declaration must be sufficient on its own to rebuild |
| Exit never tested | That exit is a capability to be exercised, not a document to be filed |
| Metering without a meter | What the unit of account is, and who agreed it |

Read down the right-hand column and the pattern is the guide's thesis in a different tense: each is a governance decision, and each was deferred to a point after the technology had already made it by default.

---

## 13. The Cymbal Bank Worked Example

> **This section is explicitly fictional and illustrative.** Cymbal Bank is the only bank in this repository's worked examples, and the decision below is invented to demonstrate the argument of sections 3, 7 and 10 - not to record a real deployment, a real procurement or a real vendor choice. No real institution is described, and no product is named, because naming one would turn an argument about provability into a comparison, which this guide does not do.

### 13.1 The decision (invented)

Cymbal Bank runs a payments-adjacent estate: services that do not settle transactions but sit close enough to them that their configuration is inside the scope of the bank's control framework. The estate is being moved off hand-built infrastructure onto a software-defined platform, and the programme has to choose what it is optimising for.

Two cases were put to the design authority, and the honest scoring is the interesting part.

### 13.2 The elasticity case was weak

Demand on this estate is flat and forecast. The payment volumes are contractual, the peak is known, and the forecast horizon is long enough that capacity is a planned purchase rather than a response. There is no seasonal spike to absorb, no product launch to survive, no tenant whose usage is unpredictable.

So the elasticity benefit was real in principle and small in practice: the estate would gain the ability to reallocate capacity within a pool it had already bought, and it did not need that ability. A programme that selected on elasticity would have been buying an option it would never exercise - and, worse, would have justified the purchase with a benefit nobody could later measure, which is how business cases become unfalsifiable.

### 13.3 The evidence case was strong

The requirement that actually bound the decision came from the control framework, and it had two parts.

- **Temporal proof.** Cymbal had to be able to demonstrate, to an auditor, that a given system's configuration had not changed between two dates - or, if it had, exactly what changed, who approved it, and when. On hand-built infrastructure this was answered by testimony and tickets. On a declared platform it is answered by a diff against a version history, which is a different category of answer (section 10.1).
- **Placement proof.** Cymbal had to demonstrate that a particular tenant's workloads could not have landed on hosts shared with a defined set of other tenants. This is not a question a hypervisor answers (section 10.3); it is a question the control plane's tenancy metadata answers, and only if the tenant was declared as a first-class object and the placement rule was expressed in terms of it.

Neither requirement is about speed, density or flexibility. Both are about **provability**, and provability is a property of the declaration, not of the hardware.

### 13.4 The consequence - provability pinned the architecture

Choosing provability as the dominant criterion had a consequence that the programme did not fully price at the time, and it is the reason this example exists.

To make both proofs cheap, Cymbal needed the declaration, the tenancy model, the placement policy and the audit record to live in **one** control plane with **one** vendor's accountability behind it. A composed estate - storage from one supplier, network from another, management assembled in-house - could in principle produce the same proofs, but it would produce them by correlating four systems, and the correlation would be Cymbal's to defend in front of an auditor. One integrated control plane made the evidence a query rather than an investigation.

That decision is defensible. It is also, structurally, the licence-concentration position of section 7.2 chosen deliberately: the estate bought provability and paid for it with substitutability at every layer inside the bundle.

Three things followed, in the order they were felt:

- **It could not shrink.** The estate's billable unit tracked capacity, so consolidating workloads onto fewer hosts reduced hardware and left the licence where it was (section 7.3). Decommissioning became commercially pointless, and the team stopped proposing it.
- **It could not substitute at the repriced layer.** When the renewal arrived with new terms, the hardware underneath was still a competitive market and the control plane above it was not. There was no alternative supplier for "the management layer of this bundle", and the declarations were written in its object model, so leaving meant rewriting them (section 7.2).
- **The renewal became the architecture review.** The questions the design had deferred - which workloads genuinely need to be here, whether a workload class should move, whether the exit exists - all arrived in one meeting with a date on it (section 7.4). Cymbal treated it as a procurement negotiation and discovered it was an architecture decision.

### 13.5 What Cymbal got wrong

Two things, and both are governance failures rather than technical ones, which is the thesis arriving on schedule.

- **It did not test the exit.** The exit plan was a document. No slice of the estate was ever rebuilt elsewhere from the declarations alone, so nobody knew which declarations were complete and which depended on an engineer's memory until the renewal made the question urgent (section 12.8).
- **It did not record which declarations were load-bearing.** The estate had many declarations and no register of which ones the proofs actually depended on. The two audit requirements rested on a small number of placement rules and one tenancy object, and nobody had written down which - so the team defended all of them with equal effort and could not say which ones it could safely change.

A third, smaller failure is worth naming because it is common: the programme measured its success by the absence of incidents rather than by the cost of producing evidence, so it never knew whether the provability it had bought was being delivered cheaply or expensively.

### 13.6 The amended thesis, restated

Cymbal did not buy elasticity and get less than it paid for. It bought **provability**, and provability is exactly what the amended thesis says the abstraction sells: the decisions that define the data centre became declarations, a declaration is a governance artefact, and the estate's evidence is the declaration's history. What Cymbal got wrong was not the choice of criterion. It was failing to notice that the thing it had bought was a governance capability, and therefore failing to govern it - no exit test, no register of load-bearing declarations, no measure of what the evidence cost to produce.

---

## 14. The Claims Audit

Every substantive claim in this guide, with its mark, source, date, version where applicable, and kind. Kind distinguishes a **documented fact** (read at a primary source), a **vendor claim** (a supplier's statement about its own product, which is evidence of what the vendor says, not of what is true), a **standard** (a ratified specification), and an **inference** (this guide's reasoning, which a reader is entitled to dispute).

| Claim | Mark | Source | Date | Version | Kind |
|---|---|---|---|---|---|
| Herrod names "the Software-Defined Datacenter" at Interop | ✅ | cto.vmware.com, Wayback capture | 8 May 2012 | capture 2014-01-16 | documented fact |
| The post's mechanism is abstraction, then pooling, then automation, extended to storage and then to networking and security | ✅ | same post | 8 May 2012 | same | documented fact |
| SDDC is characterised as "a marketing term" | ✅ | Wikipedia, SDDC article | retrieved 28 Nov 2026 | n/a | documented fact, tertiary |
| 2013 critics called it "nonsense", "marketecture" and "software-defined hype" | ✅ | CRN, 13 May 2013, via Wikipedia | 13 May 2013 | n/a | trade press |
| Broadcom names VCF as the foundation of "private and hybrid clouds" and does not use SDDC | ✅ | Broadcom acquisition release | 22 Nov 2023 | n/a | corporate statement |
| The VCF product page headlines "Private Cloud Platform" | ✅ | VCF product page | retrieved 28 Nov 2026 | 9.x | vendor claim |
| VCF is described as "a full-stack, private cloud platform" | ✅ | VCF 9.1 datasheet | Nov 2026 | 9.1 | vendor claim |
| The datasheet retains "software-defined compute, storage, and networking" as an adjective | ✅ | VCF 9.1 datasheet | Nov 2026 | 9.1 | vendor claim |
| vSphere is the compute component of the bundle | ✅ | VCF 9.1 datasheet | Nov 2026 | 9.1 | vendor claim |
| vSAN is the storage component | ✅ | VCF 9.1 datasheet | Nov 2026 | 9.1 | vendor claim |
| NSX is the networking component | ✅ | VCF 9.1 datasheet | Nov 2026 | 9.1 | vendor claim |
| vSphere Kubernetes Service is the Kubernetes runtime | ✅ | VCF 9.1 datasheet | Nov 2026 | 9.1 | vendor claim |
| VCF Operations and VCF Automation are the management components | ✅ | VCF 9.1 datasheet | Nov 2026 | 9.1 | vendor claim |
| Security, compliance and recovery are separately purchased advanced services | ✅ | VCF 9.1 datasheet | Nov 2026 | 9.1 | vendor claim |
| Purchase paths are direct, channel, OEM integrated systems and subscription via a public cloud provider | ✅ | VCF 9.1 datasheet | Nov 2026 | 9.1 | vendor claim |
| KubeVirt adds new types to the Kubernetes API to manage virtual machines | ✅ | kubevirt.io user guide | retrieved 28 Nov 2026 | v1.10.0 | documented fact |
| Every VirtualMachineInstance object represents a single running virtual machine instance | ✅ | kubevirt.io user guide | retrieved 28 Nov 2026 | v1.10.0 | documented fact |
| v1.10.0 released 18 Nov 2026; adopter list includes Red Hat, SUSE, Oracle, Microsoft, Equinix, Cloudflare, NVIDIA, CoreWeave | ✅ | kubevirt.io project page | 18 Nov 2026 | v1.10.0 | project claim |
| IBM's explainer still defines SDDC and says "the primary hurdle is gaining cross-functional agreement" | ✅ | ibm.com/think | 14 May 2026 | n/a | vendor explainer |
| Repository grep counts: 0, 2, 3, 5, 12, 12, 14, 45 | ✅ | measured on this repository | 28 Nov 2026 | branch main | documented fact |
| SDDC is a standardised architecture with a ratifying body | ❌ | no standard located in this research | - | - | rejected |
| The control plane's migration to an API-declared model is partial, not complete | ⚠️ | inference from installed base, certification and skills | 28 Nov 2026 | - | inference |
| Licence concentration reprices the layer that cannot be substituted | ⚠️ | structural inference from single-bundle design | Nov 2026 | 9.1 | inference |
| The five failure modes are analytic categories, not survey findings | ⚠️ | author's analysis | Nov 2026 | - | inference |
| A declaration reproduces the shape of an environment, not its state | ⚠️ | inference from what declarations contain | Nov 2026 | - | inference |
| Provability, rather than elasticity, is what motivates regulated adoption | ⚠️ | inference; no regulator document is cited | Nov 2026 | - | inference |

### 14.1 What the audit shows about the evidence base

Two patterns are worth naming. First, the **facts about the term** are strong: the 2012 origin, the 2013 reception, the 2023 rename and the 2026 adjective-only usage are all documented at dated sources, and they are the load-bearing historical claims. Second, the **claims about practice** are all marked as inference, because none of them has been measured - how far the control plane has actually moved, how much provability drives regulated adoption, whether the five failure modes exhaust the space. That asymmetry is deliberate and it is the honest state of the research: the terminology is documented, the consequences are argued.

---

## 15. What Could Not Be Verified

Seven entries, in the order they affected the writing. Each states what is missing, why, and what the guide does instead. The list is not a disclaimer bolted onto the end; it is the part of the audit that tells a reader where the argument is load-bearing and where it is merely plausible.

Of the seven, two are failures of the instrument rather than gaps in the record, one is a constraint this guide imposed on itself, and four are genuine holes in what is publicly documented. That breakdown matters, because a reader who treats all seven as "the author could not find it" will discount claims that are in fact well evidenced and trust claims that are not.

- **V1 - Tool limitation: web search.** On this host, `web_search` returned empty result sets for most queries during this run. It had been documented as dead for several days before 22 November 2026 and remained intermittent on 28 November 2026. This is recorded as a **tool limitation, not as absence of material**: primary pages were read directly with `web_extract` instead, which is why the source pack is weighted toward pages that could be fetched by URL. Any reader who concludes from a thin-looking reference list that the material does not exist would be drawing the wrong inference from a broken instrument.
- **V2 - The exact licensing metric in force for VCF 9.x.** Whether the current bundle is metered by processor core or by whole processor, and how bundling interacts with that metric, could not be established from a primary document in this run. ⚠️ **Flagged** if it is mentioned at all, and section 7 deliberately describes the mechanism - capacity-based billing, concentration, the non-shrinkable estate - without committing to the metric. The mechanism survives a change of metric; a printed metric would not.
- **V3 - Current published prices.** Not verified, and deliberately not printed. Per the guide's constraints, no licence price, per-unit rate or total-cost figure appears anywhere, so the absence costs the argument nothing.
- **V4 - How many enterprises still use the acronym internally.** No independent audit exists in this research, and none was attempted. The claim in section 11 is qualitative: the acronym persists in glossaries, templates, legacy documents and job adverts. The *prevalence* is unmeasured, and section 11 does not put a number on it.
- **V5 - Red Hat's primary documentation for OpenShift Virtualization.** `docs.redhat.com` returned errors during this run, so the KubeVirt-to-OpenShift-Virtualization relationship is reported from kubevirt.io's own adopter list and integration references rather than from Red Hat primary documentation. The relationship is asserted at the level the cited sources support: OpenShift Virtualization is downstream packaging of KubeVirt, not a separate implementation.
- **V6 - Whether KubeVirt-managed VM share is material outside cloud providers.** kubevirt.io lists adopters including several large cloud and infrastructure providers, but no independent figure establishes what share of enterprise virtualised workloads run under a Kubernetes API. This is why section 3.3 insists the migration is partial: the direction is documented, the magnitude is not.
- **V7 - The claim that vendor RFPs still specify SDDC.** Asserted from practice - templates are copied forward, and procurement language has a long half-life - but not measured here. No sample of live tenders was available, and inventing one would have been worse than saying so.

### 15.1 What the seven add up to

Only one entry touches a load-bearing argument: V2. Section 7's mechanism - capacity-based billing, failed substitution at the repriced layer, an estate that cannot shrink, a renewal that is an architecture review - does not depend on which metric is currently in force, which is why it was written that way. V4, V6 and V7 bound the claims in sections 11 and 3 respectively: the vocabulary's persistence, the migration's magnitude and the tender language are all asserted qualitatively and none is quantified.

V3 is a constraint, not a gap: the guide prints no prices by design, so there is nothing to verify. V1 and V5 are instrument failures, and they are recorded here rather than smoothed over because a reference list is evidence about the researcher's reach as well as about the subject. A guide that hides a broken tool produces a bibliography that looks complete and a set of claims that are unearned.

---

## 16. Glossary, Cross-References and Closing Summary

### 16.1 Glossary

- **SDDC** - a data centre whose compute, storage, networking and management are exposed as software services behind an API, so the estate's shape is declared rather than assembled. A vendor framing from 2012, not a standard.
- **HCI** - hyperconverged infrastructure: compute, storage and the storage network delivered as one cluster of identical nodes running one distributed software layer.
- **SDS** - software-defined storage: placement, protection and tiering decided by software across pooled generic devices rather than by an array controller.
- **SDN** - software-defined networking: topology and policy defined in software rather than by cabling and per-switch configuration.
- **CNI** - container network interface: the plugin contract by which a Kubernetes cluster gives pods their network identity.
- **eBPF** - a kernel mechanism that runs sandboxed programs in the operating system, increasingly used to implement network policy and load balancing without a separate virtual switch.
- **Overlay** - a virtual network built on top of a physical one by encapsulating packets, which hides the underlay and consumes part of its frame size.
- **Control plane** - the part of a platform that decides what exists and where, and holds the inventory, policy and permissions.
- **Data plane** - the path the work takes: instructions, blocks, packets. Configured by the control plane, never carried by it.
- **Declaration** - a written statement of intended state that the platform is expected to make and keep true. The atomic unit of a software-defined estate.
- **Drift** - divergence between declared state and actual state, usually caused by a change made around the API.
- **Tenancy** - the mapping from a workload to the legal or organisational entity that owns it. Metadata, held by the control plane or by nobody.
- **Blast radius** - the scope of what a single failure can affect. In a software-defined estate the control plane's blast radius is the estate.
- **Hypervisor** - the layer that turns one physical server into many virtual ones and isolates them from each other.
- **Orchestrator** - the component that holds the declaration and drives the platform toward it; in a Kubernetes-native estate, the API server plus its controllers.
- **Golden image** - a pre-built machine image treated as the standard starting point. Useful as a starting point, dangerous as a substitute for a declaration.
- **Chargeback** - attributing platform cost to the consumers who caused it. Usually built on allocation rather than consumption, which is why it is often disbelieved.
- **NUMA** - non-uniform memory access: memory attached to one CPU socket is closer to that socket than to another, a physical boundary no scheduler removes.
- **MTU** - maximum transmission unit: the largest frame a link will carry. An overlay consumes part of it, which is how encapsulation produces blackholes.
- **VPC** - virtual private cloud: a provider-defined isolated network boundary, the hyperscaler's version of the tenancy-and-network declaration.
- **IaaS** - infrastructure as a service: provisioning of virtual machines, storage and networks through an API.
- **Platform engineering** - the discipline of running the internal platform as a product, with the declaration and the API as its interface.

### 16.2 Cross-references

- **[data_center_guide.md](data_center_guide.md)** - the facilities half of this subject, cross-referenced by section: section 2 Facility Anatomy, section 3 Taxonomy, section 4 Uptime Tiers, section 5 Power, section 6 Cooling, section 7 Layout and Racks, section 8 Physical Security, section 9 Certification and Operations, section 10 Metrics, section 11 Sustainability, section 12 Colocation Economics, section 13 its worked example, section 14 its glossary. Nothing in this guide restates any of it.
- **The six head-to-heads** - [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md), [cloudstack_vs_openstack_guide.md](cloudstack_vs_openstack_guide.md), [nutanix_enterprise_ai_vs_openshift_ai_guide.md](nutanix_enterprise_ai_vs_openshift_ai_guide.md), [charmed_kubernetes_vs_openshift_guide.md](charmed_kubernetes_vs_openshift_guide.md), [charmed_kubeflow_vs_openshift_ai_guide.md](charmed_kubeflow_vs_openshift_ai_guide.md), [devstack_openstack_guide.md](devstack_openstack_guide.md). These own every product comparison; section 8 of this guide deliberately does not compete with them.
- **[openshift_ai_alternatives_guide.md](openshift_ai_alternatives_guide.md)** - the adjacent survey of AI platform alternatives, for readers whose estate is being shaped by a GPU workload rather than by a general-purpose one.
- **[ai_factory_build_guide.md](ai_factory_build_guide.md)** - the AI-factory build pattern, which is the most demanding current test of an abstraction layer: dense compute, shared storage, and a scheduler that must know about topology.
- **[cloud_providers_guide.md](cloud_providers_guide.md)** - the vendor-neutral cloud comparison that frames the hyperscaler position of section 8.5.
- **[ibm_cloud_guide.md](ibm_cloud_guide.md)** - the regulated-industry cloud angle, adjacent to section 10.
- **[finops_guide.md](finops_guide.md)** - the cost lens that section 7's mechanism feeds into: chargeback, showback and the discipline of attributing spend.

### 16.3 What this guide does not settle

Three questions are left open deliberately, because the evidence does not exist here and a confident answer would be an invention:

- **How far the control plane has actually moved.** Section 3 asserts a direction and a partial migration; it does not put a number on the share of enterprise workloads that are declared through an API rather than managed through a hypervisor console. That number would settle the argument and nobody has published it (V6).
- **Whether provability or operating cost dominates adoption in practice.** Section 10 and the worked example argue that provability is what a regulated estate is really buying, and that argument is built from mechanism rather than from a measured sample of decisions. It could be wrong for estates whose regulator is content with testimony.
- **What replaces "private cloud".** The term's own history - coined in 2012, mocked in 2013, retired by its owner in 2023 - suggests labels have a commercial half-life of roughly a decade. A reader in 2036 will be reading a document that calls this estate something else, and the architecture in it will be recognisably the one described here.

### 16.4 Closing Summary

The term was coined at a trade show in 2012, mocked as marketecture within a year, renamed by its own owner in 2023, and by 2026 it survives only as a lowercase adjective in a datasheet that calls the product a private cloud platform. None of that touched the architecture. The technique - abstract, pool, automate, then declare - is exactly what the product does, and the acronym was always a description of a purchase rather than of a mechanism.

What the technique actually changes is narrower than the slogan and more consequential than the sceptics allowed. Physics is undefeated: a disk still dies, a switch still drops, a storage tier still saturates, a power supply still fails, and no control plane governs any of it. What moved is the **decision** - from hands on hardware to text in a repository, with an author, a diff, an approval and an enforcement mechanism. That is a governance artefact, and it explains both halves of the thesis. The failures that matter are failures of declaration: drift, leak, undeclared tenancy, a control plane nobody dares upgrade, an API nobody uses. And the bill is a bill for the thing that reads the declarations, which is why an estate cannot shrink its way out of it and why the renewal is the architecture review whether or not anyone scheduled one.

the decisions that define the data centre became declarations, and a declaration is a governance artefact - which is why an SDDC's failures are governance failures and its economics are licence economics.
