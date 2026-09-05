# Kafka Virtualization: Running and Operating Apache Kafka on Virtualized and Containerized Infrastructure

*The virtualization/operations deep-dive of the research repo's Kafka series — a companion to [Apache Kafka: the Distributed Event-Streaming Platform](../banking/kafka_guide.md) (the platform reference — architecture, replication, KRaft internals, messaging semantics, Kafka Streams/ksqlDB, Connect, Schema Registry — cross-referenced from §3 and condensed here, never re-derived) and [kafka_alternatives_guide.md](kafka_alternatives_guide.md) (the head-to-head vs Redpanda/Pulsar/WarpStream — link in §2, no comparison matrices repeated here). This guide's unique angle is the **virtualization decision surface**: what it means to run Kafka on virtualized/containerized infrastructure, the full spectrum from self-managed VMs through Kubernetes operators to managed cloud services and serverless tiers, the storage and stateful-operating realities, the security model, and the operational consequences of the KRaft era. The worked example (§15) is a Cymbal Bank event-backbone decision.*

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Event Streaming / Platform Engineering — Kafka platform operations, Kubernetes, managed services, KRaft-era migration, banking
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** kafka.apache.org (the 4.0.0 release announcement of 18 March 2025; the Kafka documentation), the Apache Kafka wiki/CWiki KIPs (KIP-500, KIP-833, KIP-36, KIP-18), cncf.io and strimzi.io (Strimzi — CNCF status, operators, CRDs), docs.confluent.io (Confluent for Kubernetes overview and release notes; Confluent Cloud overview), aws.amazon.com (MSK GA announcement; MSK Serverless what's-new), aiven.io, kubernetes.io (Local Persistent Volumes GA), developers.redhat.com (Streams for Apache Kafka). NOTE: this pass had **live web access**; the date-critical claims above were verified at primary sources. A few items (Aiven's founding year on secondary sources only, the exact Confluent-for-Kubernetes GA/rename date, licensing detail) are flagged ⚠ honestly in the [Claims Audit](#claims-audit-and-verification-ledger) rather than asserted.
> **Last Updated:** September 2026
> **Companion guides (technology/ siblings — plain filenames):** [kafka_alternatives_guide.md](kafka_alternatives_guide.md) (the head-to-head), [event_stream_processing_guide.md](event_stream_processing_guide.md) (the ESP discipline), [complex_event_processing_guide.md](complex_event_processing_guide.md) (CEP patterns), [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) (reliability/data-loss), [secure_red_hat_openshift_guide.md](secure_red_hat_openshift_guide.md), [openshift_workload_availability_guide.md](openshift_workload_availability_guide.md), [charmed_kubernetes_vs_openshift_guide.md](charmed_kubernetes_vs_openshift_guide.md), [devstack_openstack_guide.md](devstack_openstack_guide.md) (platform mechanics — Kubernetes/OpenShift/OpenStack internals live there, not here), [container_certificates_guide.md](container_certificates_guide.md) (TLS/certs in containers — §12), [enterprise_middleware_integration_platform_guide.md](enterprise_middleware_integration_platform_guide.md) (enterprise middleware), [ai_platform_engineering_guide.md](ai_platform_engineering_guide.md) (platform engineering)
> **Companion guides (banking/ siblings — prefix ../banking/):** [kafka_guide.md](../banking/kafka_guide.md) (the dedicated Kafka platform reference — cross-ref §3, §4, §10)

**Verification convention used throughout:** ✅ = verified this pass against a primary source (kafka.apache.org, the Apache CWiki KIPs, cncf.io, strimzi.io, docs.confluent.io, aws.amazon.com, kubernetes.io, aiven.io, redhat.com) or already verified in a cross-referenced sibling guide's ledger; ⚠ = flagged (single-source, secondary-source-only, approximate, date-dependent, or held from the author's knowledge base); ❌ = disputed or false as stated (with the correction inline). Unmarked statements are structural/architectural knowledge presented as such. The consolidated list is the [Claims Audit](#claims-audit-and-verification-ledger); the residual gaps are in [What Could Not Be Verified](#claims-audit-and-verification-ledger).

---

## Table of Contents

1. [The Virtualization Question](#1-the-virtualization-question)
2. [The Virtualization Spectrum](#2-the-virtualization-spectrum)
3. [The Architecture Recap (Condensed)](#3-the-architecture-recap-condensed)
4. [The KRaft Era and Its Operational Consequences](#4-the-kraft-era-and-its-operational-consequences)
5. [Kafka on VMs and Bare Metal](#5-kafka-on-vms-and-bare-metal)
6. [Kafka on Kubernetes — The Operators: Strimzi](#6-kafka-on-kubernetes--the-operators-strimzi)
7. [Kafka on Kubernetes — Confluent for Kubernetes](#7-kafka-on-kubernetes--confluent-for-kubernetes)
8. [The Distribution Layer: Red Hat Streams for Apache Kafka and the Ecosystem](#8-the-distribution-layer-red-hat-streams-for-apache-kafka-and-the-ecosystem)
9. [The Storage Realities](#9-the-storage-realities)
10. [The Stateful Operating Model on Kubernetes](#10-the-stateful-operating-model-on-kubernetes)
11. [Monitoring and Observability](#11-monitoring-and-observability)
12. [Security in the Virtualized Estate](#12-security-in-the-virtualized-estate)
13. [The Managed Services: Confluent Cloud, AWS MSK, Aiven](#13-the-managed-services-confluent-cloud-aws-msk-aiven)
14. [The Decision Surface and TCO](#14-the-decision-surface-and-tco)
15. [Worked Example: The Cymbal Bank Kafka-Virtualization Decision](#15-worked-example-the-cymbal-bank-kafka-virtualization-decision)
- [The Glossary](#the-glossary)
- [Claims Audit and Verification Ledger](#claims-audit-and-verification-ledger)
- [Cross-References Recap](#cross-references-recap)
- [References and Further Reading](#references-and-further-reading)

---

**How to use this guide:** §1–§2 frame the virtualization question and the four deployment families; §3 is the condensed architecture recap (depth in [Apache Kafka: the Distributed Event-Streaming Platform](../banking/kafka_guide.md)); §4 is the KRaft era and what it changed operationally — read it before any migration planning. §5–§8 are the self-managed families (VMs, then the Strimzi and Confluent-for-Kubernetes operators, then the Red Hat distribution layer); §9–§12 are the realities that apply to all of them (storage, the stateful operating model, monitoring, security). §13 covers the managed services, §14 the decision method and TCO, and §15 the Cymbal Bank worked example that applies it. The Glossary defines every bolded term; the Claims Audit and Verification Ledger records exactly what was verified this pass (✅), flagged (⚠), or debunked (❌), with What Could Not Be Verified stated plainly. The ✅/⚠ integrity convention is the repo's own: a ✅ means "checked at a primary source this pass or held from a sibling ledger", never "assumed".

---

## 1. The Virtualization Question

Kafka is a **stateful distributed system**: every broker owns a slice of the log on local disk, and its identity, its data, and its role in the replica quorum are sticky in ways that a stateless web tier is not. That single property — statefulness — is why "just containerize it" is not the whole answer, and why this guide exists. The interesting question for an architect is rarely *whether* to use Kafka (that is the alternatives sibling's question) and almost never *whether to virtualize it* (everything in a modern estate is virtualized at some layer). The real question is: **at which layer of the stack do you want the virtualization and operational responsibility to sit?**

Every deployment of Kafka is ultimately running on somebody's virtualized substrate:

- Kafka on **bare metal** runs on physical hosts, but those hosts sit under a hypervisor-adjacent stack (or an orchestrator) the moment the estate standardizes.
- Kafka on **VMs** runs on virtual disks over shared storage arrays — the classic enterprise pattern of the ZooKeeper era, and still the default in many regulated on-prem estates.
- Kafka on **Kubernetes** (or OpenShift, itself Kubernetes with a security-hardened layer — see [secure_red_hat_openshift_guide.md](secure_red_hat_openshift_guide.md)) runs in pods with containers, operators, PVCs and the pod-lifecycle machinery of the control plane.
- Kafka as a **managed service** (Confluent Cloud, AWS MSK, Aiven) runs on the vendor's own virtualized infrastructure; you operate an API and a console, not brokers.
- **Serverless/elastic tiers** (MSK Serverless, Confluent Cloud's elastic clusters, object-storage-backed WarpStream) push the virtualization abstraction up to the point where capacity itself is automatic.

The spectrum is a ladder of **who owns what**: patching, upgrades, storage failure handling, capacity planning, DR, security. Each rung trades control for convenience, and the correct rung differs by workload class even inside one bank. This guide walks the whole ladder — operators first (Strimzi, Confluent for Kubernetes), then the stateful and storage realities, then managed services, then a decision method — and grounds it in the KRaft era, because the ZooKeeper removal changed the operating model of every rung.

**Why this guide exists and what it deliberately does not repeat:**

- The **architecture** of Kafka (brokers, partitions, ISR replication, log segments, exactly-once, Kafka Streams, Connect, Schema Registry) is documented in depth in the dedicated platform reference [Apache Kafka: the Distributed Event-Streaming Platform](../banking/kafka_guide.md). §3 here is a deliberately condensed recap so this guide stands alone; every pointer for depth goes there.
- The **when-not-Kafka / head-to-head** question (Redpanda, Pulsar, WarpStream, NATS, queues) is the alternatives sibling's job — [kafka_alternatives_guide.md](kafka_alternatives_guide.md). Nothing here repeats its comparison matrices; where a managed or serverless alternative would change the decision, this guide links rather than re-derives.
- The **platform mechanics** — Kubernetes/OpenShift internals, SCCs, operators-on-OpenShift, cluster storage classes, TLS in containers — live in the platform siblings ([secure_red_hat_openshift_guide.md](secure_red_hat_openshift_guide.md), [openshift_workload_availability_guide.md](openshift_workload_availability_guide.md), [charmed_kubernetes_vs_openshift_guide.md](charmed_kubernetes_vs_openshift_guide.md), [devstack_openstack_guide.md](devstack_openstack_guide.md), [container_certificates_guide.md](container_certificates_guide.md)). §10 and §12 link to them instead of re-deriving OpenShift/Kubernetes internals.
- The **persona rule**: the only bank persona used in this repository's worked examples is Cymbal Bank (§15). No other bank — and in particular no entity from the banned persona list — appears anywhere in this file.

## 2. The Virtualization Spectrum

### 2.1 The Four Deployment Families

| Family | What you run | Who operates the brokers | Typical owner | Elasticity |
|---|---|---|---|---|
| **1. Self-managed, VMs/bare metal** | Apache Kafka (or a distribution like Confluent Platform) as OS services/systemd units on provisioned hosts | Your platform team, 24/7 | Regulated on-prem estates, existing VM footprints, air-gapped/sovereign environments | Manual — add hosts, reassign partitions |
| **2. Containerized + operator** | Apache Kafka in pods on Kubernetes/OpenShift, driven by a declarative operator (Strimzi; Confluent for Kubernetes for the Confluent Platform distribution) | Your platform team + operator automation + the cluster's control plane | Cloud-native platforms, OpenShift-centric banks, GitOps shops | Semi-automatic — operator rolls brokers; partition reassignment still a step |
| **3. Managed (provisioned) service** | Vendor-run Kafka clusters behind an API/console (AWS MSK, Confluent Cloud dedicated/basic, Aiven for Apache Kafka) | Vendor runs brokers; you size, monitor, govern, and pay | Teams wanting Kafka without a Kafka ops team | Vendor-scaled within your chosen cluster size |
| **4. Serverless/elastic or object-storage-backed** | Automatic-capacity Kafka (MSK Serverless, Confluent Cloud elastic), or Kafka-compatible object-storage brokers (WarpStream — see the alternatives sibling) | Vendor, including capacity | Spiky workloads, developer self-service, cost-sensitive streaming | Automatic — pay for throughput/storage used |

Two clarifying notes. First, the families are **not strictly ordered by "virtualization"**: family 1's VMs are virtualized at the hypervisor; family 2 adds container virtualization plus an orchestration layer; families 3–4 abstract the whole thing away. Second, the boundary between 2 and 3 is the sharpest **operating-model** discontinuity in the ladder: crossing it moves patch Tuesday, upgrade weekends, disk-failure runs and ZooKeeper/KRaft quorum care from your team to a vendor — and moves control, custom configuration, data-residency certainty and cost predictability the other way.

### 2.2 What the Choice Actually Decides

The family choice is a bundle of seven sub-decisions, and honest evaluation separates them:

1. **Patching and upgrades** — who applies Kafka patch releases, and who tests them against your client fleet and connectors?
2. **Storage and its failure modes** — who provisions disks, monitors throughput/latency, and handles a failing drive (see §9)?
3. **Capacity planning** — who predicts the traffic curve, sizes brokers, and absorbs a spike (see §13 for serverless)?
4. **Quorum and metadata health** — who watches controller state, ISR health, and (historically) the ZooKeeper ensemble? In the KRaft era the question is who watches the controller quorum (§4).
5. **Disaster recovery** — who runs the DR drills, the MirrorMaker 2 pipelines, and the region failover (cross-ref [zero-downtime and DR content in the platform siblings])?
6. **Security and compliance** — who holds the keys, certificates, ACLs, audit logs, and the evidence for the regulator (see §12)?
7. **Data residency and exit** — where does the data physically live, and how hard is it to leave the vendor (see §13–§14)?

**The framing rule that keeps this guide honest:** managed services do not remove operational work — they *relocate* it and convert it into (a) vendor management, (b) cost governance, and (c) a smaller but different set of platform tasks (quota, tenant, schema, egress and incident management with a vendor on the line). Every family has a real operating model; the decision is which one your constraints can tolerate. §14 turns this into a scored method, and §15 applies it at Cymbal Bank.

### 2.3 Convergence Notes for 2026

Three forces are narrowing the practical gap between families as this guide is written, and an architect should weigh them when reading the family descriptions:

1. **KRaft flattened the metadata tax.** The ZooKeeper ensemble was the single biggest "Kafka needs its own ops" argument; its removal (Kafka 4.0, §4) made family 1 and family 2 materially lighter, which shifts the break-even against families 3–4 for mid-size estates.
2. **Tiered storage decoupled retention from local disk.** Kafka 3.6+ tiered storage (§9.4) moves cold segments to object storage, so a self-managed cluster's storage bill stops scaling linearly with retention — eroding one of managed/serverless pricing's structural advantages (the vendor already amortizes object storage).
3. **Protocol portability made hybrids cheap.** Because every family speaks the Kafka protocol, workloads can move between families (or span them, §14.3) without client changes. That turns the family decision from "one bet for five years" into "an allocation you can revise" — which is why §15's Cymbal Bank example chooses a hybrid rather than a single family.

None of these make the families equivalent — control, residency evidence, FTE load, and cost structure still differ sharply (that is §14's scoring job) — but they mean the 2026 decision is less *permanent* than it was in the ZooKeeper era, and the exit-cost question (§13.5) matters correspondingly less than vendor lock-in rhetoric suggests.

## 3. The Architecture Recap (Condensed)

*Depth lives in the platform reference, [Apache Kafka: the Distributed Event-Streaming Platform](../banking/kafka_guide.md) — its §2 (architecture) and §3 (replication/KRaft). This recap exists so the virtualization sections stand alone; every architectural term is only defined as far as the operations discussion needs.*

**The core idea.** Kafka is a **distributed, partitioned, replicated commit log**. Producers append records to **topics**; each topic is split into **partitions** (the unit of parallelism, ordering, and load); each partition is an ordered, immutable sequence of records replicated across **brokers**; **consumer groups** divide partitions among members and checkpoint their position with **offsets**. The log is stored on local disk in **segments** and retained per policy (time/size) or compacted by key.

**Brokers.** A Kafka cluster is a set of broker processes. Each partition has one **leader** (serves all reads/writes) and **followers** that replicate from it; the set of followers caught up with the leader is the **ISR** (in-sync replicas). `acks=all` + `min.insync.replicas` is the durability contract. Replication factor 3 across three failure domains (racks/AZs) is the canonical production shape. ✅ (documented behaviour, cross-ref the banking guide's ledger)

**Metadata and the controller.** Something must decide who leads what and track the cluster's metadata. For a decade that something was **Apache ZooKeeper** — a second distributed system every Kafka operator ran, patched, and watched. Kafka's controller (a broker with special duties, historically elected via ZooKeeper) manages partition leadership, ISR changes, and administrative operations.

**KRaft replaces the ensemble.** KRaft (Kafka Raft metadata mode, KIP-500) moved metadata into Kafka itself: a quorum of **controller nodes** (dedicated processes or combined with brokers) runs the Raft consensus protocol and owns a metadata log. ZooKeeper is gone — early access in Kafka 2.8, production-ready for new clusters in Kafka 3.3 (KIP-833), and **removed entirely in Kafka 4.0** (18 March 2025). ✅ The operational consequences are the subject of §4.

**The components around the broker** (each with its own virtualization story): Kafka Connect (integration workers), Schema Registry (schema governance), Kafka Streams/ksqlDB (in-application / server-side stream processing), MirrorMaker 2 (cross-cluster replication), and the fleet of client libraries. On Kubernetes, operators manage most of these as first-class custom resources (§6–§7).

**Why statefulness dominates the virtualization discussion.** Replication makes Kafka fault-tolerant *within* the cluster, but it does not make brokers disposable: a broker's log directories must survive restarts, its broker ID is referenced by partition assignments and client metadata, and the placement of replicas across failure domains is a *data* decision. Container schedulers that freely move workloads break the assumptions Kafka's replication and rack-awareness logic depend on. Every operator in §6–§7 exists to reconcile "Kubernetes wants to reschedule things" with "Kafka needs stable identity, sticky storage, and placement constraints" — the topic of §10.

## 4. The KRaft Era and Its Operational Consequences

### 4.1 The Verified Timeline

| Version | Date (verified where marked) | KRaft/ZooKeeper milestone | Source |
|---|---|---|---|
| Kafka 0.x–2.7 | 2011–2021 | ZooKeeper mode is the only mode; the controller is elected via ZooKeeper | — (architecture history, cross-ref banking guide) |
| **Kafka 2.8** | April 2021 ⚠ (month from author's knowledge base) | **KRaft early access** — running in KRaft mode supported as an early-access feature (KIP-500) | ✅ cwiki: KAFKA-9119 (KIP-500 umbrella) |
| **Kafka 3.3** | October 2022 ⚠ (month from author's knowledge base) | **KRaft production-ready for new clusters** (KIP-833: "Mark KRaft as Production Ready" for the upcoming 3.3 release) | ✅ cwiki: KIP-833 |
| Kafka 3.5 | 2023 | ZooKeeper mode **deprecated** (per KIP-833's plan: deprecate ZK mode in 3.5) | ✅ cwiki: KIP-833 (export text) |
| Kafka 3.6 | 2023 | Tiered storage GA (KIP-405); still dual-mode | ✅ repo-verified in the alternatives sibling's ledger |
| **Kafka 4.0** | **18 March 2025** | **ZooKeeper removed entirely** — "the first major release to operate entirely without Apache ZooKeeper"; KRaft mode by default | ✅ kafka.apache.org 4.0.0 release announcement |

The most-often-repeated error in the field is that "KRaft early access began in 3.3." It did not: **2.8 shipped KRaft as early access (KIP-500), and 3.3 marked it production-ready for new clusters (KIP-833)** — a two-step ramp that the timeline above keeps straight, and that the ❌ section of the Claims Audit records explicitly.

### 4.2 What the ZooKeeper Removal Changes for Operators

The removal is not a footnote — it changes the topology, the failure modes, and the upgrade calendar of every deployment family in §2:

- **One less system to run.** The "Kafka is the system that manages your ZooKeeper cluster's ZooKeeper" joke is retired: no ensemble to size at 3 or 5 nodes, no ensemble to patch, monitor, or back up, no `zookeeper.connect`, no ensemble/controller split-brain class of incidents. ✅ (4.0.0 announcement: "eliminating the complexity of maintaining a separate ZooKeeper ensemble")
- **The controller quorum is now a first-class component.** KRaft runs a quorum of controllers (`process.roles=controller`, or combined `broker+controller` on every node in small deployments). Production guidance is a 3-node dedicated controller quorum for large clusters (⚠ practice guidance — 3 is the standard Raft minimum for one-failure tolerance; larger odd quorums scale metadata capacity). Controller nodes hold the metadata log and are as stateful as brokers — they need the same sticky-identity and storage care (§10), and losing the quorum loses the cluster's ability to elect leaders (CFK's release notes even ship a dedicated disaster-recovery procedure for controller-quorum loss ✅ docs.confluent.io).
- **Faster, safer leadership and bigger scale.** KRaft removes the ZooKeeper round-trips from controller election and metadata operations; the 4.0 announcement's framing is "reduces operational overhead, enhances scalability" ✅. Partition-count ceilings rise (the old ~200k-partition soft cap guidance was partly a ZooKeeper-metadata constraint), and controller failover no longer waits on an external system. ⚠ specific new ceiling numbers are version- and hardware-dependent — treat vendor/blog numbers as indicative.
- **The upgrade path is now a migration project with a bridge.** Clusters born in the ZooKeeper era must migrate: the documented path runs through a 3.x bridge release (enable KRaft migration on 3.5–3.7-era tooling, then move to 4.0). ✅ cross-ref [Apache Kafka: the Distributed Event-Streaming Platform](../banking/kafka_guide.md) §3.6 (the migration path) — this guide does not re-derive it. Operators have productized it: Strimzi's own release notes mark 0.45 as "the last minor Strimzi version with support for ZooKeeper-based Apache Kafka clusters" (✅ GitHub release notes) and Confluent for Kubernetes ships a ZooKeeper→KRaft migration procedure with a preflight check (✅ docs.confluent.io release notes 3.3.0).
- **The virtualization angle.** KRaft made Kafka *easier* to containerize: one fewer stateful system to fit into the pod/operator model, and the combined-role mode lets small clusters run on three pods instead of three brokers plus three ensemble nodes. But it *concentrated* statefulness: the metadata log lives on controller disks, so the storage and identity questions of §9–§10 now apply to controllers as well as brokers. Every operator below had to grow KRaft support (Strimzi: KRaft-supported since the 0.4x era and mandatory from 0.46+ as ZooKeeper-based clusters were dropped; CFK: Kafka CRs that deploy "KRaft or ZooKeeper", with dynamic-quorum controller scaling and 2.5-datacenter migration support in 3.3.0 ✅ docs.confluent.io).

**What did *not* change (worth stating, because marketing overreaches):** (1) partition replication, ISR, and the `acks`/`min.insync.replicas` durability contract are untouched — KRaft replaced the metadata system, not the data plane (cross-ref the banking guide's §3 for the contract); (2) brokers remain stateful JVM services needing heap, disk, and capacity care — the operator's job description in §10 is unchanged by the metadata mode; (3) ZooKeeper-mode clusters do not migrate themselves — Kafka 4.0 does not read 3.x ZooKeeper-era metadata without the bridge path (§4.2), so "we upgraded the binary" is not a migration; (4) client-visible semantics and the wire protocol are preserved, which is precisely why the virtualization-family decision (§2) can be made independently of the metadata-mode decision — you can move to KRaft and stay on VMs, or move to Kubernetes and migrate metadata later, in either order, as long as the bridge releases are respected. ⚠ ordering guidance is practice; the compatibility statements are documented Kafka behaviour ✅.

**Net operational consequence:** KRaft did not make Kafka stateless — it made the stateful part smaller, self-contained, and owned by the same team that already owns the brokers. For virtualization decisions, that lowers the "Kafka is operationally heavy" penalty that historically pushed teams toward managed services (§13–§14), because the heaviest second system is gone.

### 4.3 The KRaft Operating Quick Reference

For operators planning a KRaft-era deployment or migration, the topology knobs that matter (⚠ values are standard-practice guidance from the author's knowledge base unless marked — the *semantics* are documented Kafka behaviour ✅, the *specific recommended numbers* are practice):

| Question | Answer at a glance |
|---|---|
| What roles exist? | `process.roles=broker` / `controller` / `broker+controller`; the active controller is elected from the controller quorum by Raft. |
| How many controllers? | 3 for production (tolerates one loss); odd numbers; scale to 5 for very large clusters. Dedicated controller nodes separate metadata-plane load from data-plane traffic. ⚠ practice guidance. |
| Where does the metadata live? | A replicated metadata log on controller storage — as durable and as backed-up as broker data (§9, §10.4). |
| Can brokers and controllers share hosts/pods? | Yes — combined `broker+controller` is the standard small-cluster shape and the operator default for modest deployments; node pools (§6.2) let you split them later without rebuilding. |
| What breaks in a quorum loss? | No controller election, no metadata writes, no new partition leadership — a full control-plane outage. This is why the operator world now ships quorum-loss DR procedures (CFK 3.3.0 ✅ docs.confluent.io). |
| Migration from ZooKeeper mode? | One-way, through a 3.x bridge release, using migration tooling (banking guide §3.6 ✅); operators productize it (Strimzi cutover ✅ GitHub; CFK preflight ✅ docs). Plan it as a project, not a patch. |

**The decision lens for this guide:** when choosing between families, ask "who runs the controller quorum?" — in family 1 and 2 that is your team (with operator help in 2); in families 3–4 it is the vendor. The quorum is small, but it is the *control plane of the whole cluster*, and its care (storage, backup, upgrades, DR) is the modern replacement for the ZooKeeper runbook.

## 5. Kafka on VMs and Bare Metal

*Condensed by design: the platform reference covers Kafka internals; this section is only the virtualization/ops shape of the classic family.*

**The classic shape.** Kafka predates Kubernetes by a decade and runs perfectly well as OS services: download the Apache distribution (or a distribution like Confluent Platform's tarball/RPM/Ansible route), configure `server.properties`, run under systemd, and provision hosts with local disks. The production template has always been: replication factor 3, brokers spread across racks/AZs, JBOD or a single well-sized disk per broker (§9), a controller strategy (ZooKeeper ensemble historically; dedicated or combined KRaft controllers today), and configuration management (Ansible/Puppet/Chef) plus a monitoring stack (§11). ✅ this is documented standard practice — the KIP-18 JBOD rationale and the rack-awareness machinery (KIP-36) both predate containers (cwiki, ✅).

**Why it survives in 2026.** Four reasons: (1) **regulatory/sovereign estates** where the platform standard is hardened VMs on-prem and Kubernetes is not (yet) permitted for the crown jewels — cross-ref [enterprise_middleware_integration_platform_guide.md](enterprise_middleware_integration_platform_guide.md) for the middleware-estate framing; (2) **existing footprints** — the cluster is already there and the migration cost (especially the ZooKeeper→KRaft bridge, §4.2) exceeds its remaining value; (3) **predictable, high-throughput workloads** where direct disk and NUMA/JVM tuning on a known OS image still beat shared schedulers on cost-per-GB/s; (4) **Kafka Streams/Connect co-location patterns** that predate the operator ecosystem.

**The honest costs.** Every patch, upgrade, disk replacement, rebalance, and security scan is manual or scripted by your own team; the ZooKeeper era added an ensemble to babysit; capacity planning is yours; and the KRaft migration for legacy fleets is a scheduled project with a bridge release (§4.2). VMs add their own layer of care: virtual disks over shared storage change the latency/throughput assumptions of §9 (the classic "don't put Kafka logs on a contended SAN volume" problem — ⚠ practice guidance, storage-specific), and VM snapshots are **not** a safe Kafka backup mechanism (crash-consistent snapshots of a broker can resurrect stale state — ⚠ operational-knowledge; replication + MirrorMaker 2 + topic-level tooling is the supported DR story, cross-ref the banking guide's DR content).

**Verdict.** Fully viable, fully supported, and still the right answer for a meaningful minority of estates. It is the baseline against which every other family in this guide prices itself (§14), because its cost structure is the most transparent: compute ×3 for replication, disk ×3, and platform-team hours.

**The VM-era runbook vs the operator runbook.** The sharpest way to see what virtualization changes is to line up the recurring operations side by side (operator behaviours verified ✅ in §6–§7; VM-era tasks are the classic manual runbook ⚠ practice):

| Operation | VM/bare-metal era (family 1) | Operator era (family 2) |
|---|---|---|
| Kafka version upgrade | Download, stage, stop broker, replace binaries, start, verify — per broker, per cluster | Change the version in the CR; operator rolls pods with controlled shutdown and ISR checks between steps (§10.4) |
| Config change (e.g., a listener, a quota) | Edit `server.properties`, rolling restart in the right order, don't typo the file | Edit the CR (or a ConfigMap it references); operator reconciles and rolls safely |
| Add a broker | Provision host/VM, install, join, run `kafka-reassign-partitions` by hand | Scale the CR/node pool; run a `KafkaRebalance` (Cruise Control) for data movement (§10.3) |
| Node/host maintenance | Cordoning, manual leadership moves, careful sequencing, watch re-replication | Node drain → Drain Cleaner intercepts eviction, moves leadership/data, pod reschedules on its PVC (§10.2) |
| Certificate rotation | Scripted per-broker/per-listener, calendar-driven | Operator-managed CA + auto-issued certs, rotated as part of reconciliation (§12) |
| New environment | Weeks of runbooks and tickets | Namespace + CR from git (GitOps) |

The table is the whole argument for §6–§8 in miniature: **family 1 and family 2 run the same Kafka; the difference is that family 2's recurring operations are declared state rather than performed acts.**

## 6. Kafka on Kubernetes — The Operators: Strimzi

### 6.1 What Strimzi Is (Verified)

Strimzi is the open-source, CNCF-governed way to run Apache Kafka on Kubernetes: **container images plus a family of operators** that deploy and manage Kafka clusters, Connect, MirrorMaker 2, the HTTP bridge, topics, and users through Kubernetes custom resources. ✅ strimzi.io: "Strimzi provides container images and operators for running Kafka on Kubernetes," with custom resources such as `Kafka`, `KafkaTopic`, and `KafkaUser`.

- **CNCF status:** accepted to the CNCF on **28 August 2019** (Sandbox) and moved to **Incubating** maturity on **8 February 2024**. ✅ cncf.io/projects/strimzi. First commit **21 March 2016**. ✅ CNCF project insights. The project is a "Series of LF Projects" with Red Hat as the dominant vendor contributor (Red Hat's supported distribution is §8).
- **Adoption signal (⚠ logo-wall evidence, not audited deployments):** the strimzi.io users page shows an enterprise-heavy logo wall — CERN, Swisscom, Baloise, Decathlon, Reddit, Feedzai, moniepoint and others — plus vendor logos including Red Hat, IBM, Cloudera and Axual, which is consistent with Strimzi being the *base* technology for several commercial Kafka-on-Kubernetes offerings rather than only a direct-to-operator choice. Treat the list as directional evidence of ecosystem gravity, not as verified production references (companies appear and disappear from such walls). ✅ the logos were observed on strimzi.io this pass; ⚠ their implied deployment claims are unverified.
- **The operator set** (✅ strimzi.io docs, overview): the **Cluster Operator** (deploys and manages Kafka clusters and related components — one replica by default, add replicas with leader election), the **Topic Operator** and **User Operator** (create/configure/delete topics and users), often run together as the **Entity Operator** in one pod, plus the **Drain Cleaner** (a separate tool for safe pod eviction during node maintenance). The Cluster Operator manages Kafka clusters (including Entity Operator, Kafka Exporter, Cruise Control), Kafka Connect, Kafka MirrorMaker 2, and Kafka Bridge.
- **Managed via GitOps:** because everything is a custom resource, Kafka config is YAML in git — the IaC/GitOps story is native rather than bolted on. ✅ strimzi.io ("Manage Kafka using GitOps").
- **Current stable:** Strimzi Kafka Operator **1.2.0** at the time of this pass. ⚠ date-dependent — check [strimzi.io/downloads](https://strimzi.io/downloads/) (the observation is from the September 2026 pass). Strimzi's versioning note: the 0.4x series ended ZooKeeper-based cluster support at **0.45** (KRaft mandatory from 0.46+; MirrorMaker 1 also dropped), and the old CRD API versions (v1alpha1/v1beta1/v1beta2) were supported until the 1.0.0/0.52.0 line. ✅ GitHub release notes.

### 6.2 The Deployment Model

Strimzi turns a `Kafka` custom resource into the standard Kubernetes machinery:

- **StatefulSets** for brokers (and controllers in KRaft mode) — one pod per broker with a stable ordinal identity (`kafka-0`, `kafka-1`, …), stable per-pod DNS via headless services, and **PVCs** per pod for the log directories. ✅ (Strimzi docs describe StatefulSet-based deployment; §10 covers why identity + PVCs are the crux of stateful Kafka.)
- **Node pools** (`KafkaNodePool`) since the 0.4x era let one cluster mix broker and controller roles and different storage classes/sizes per pool — the modern way to express the KRaft controller/broker split and to scale storage- or compute-heavy pools independently. ✅ strimzi.io docs ("node pools").
- **Listeners** for client access: `internal` (cluster-internal), `cluster-ip`, `nodeport`, `loadbalancer`, `ingress`, and **OpenShift Routes** — each with TLS termination options. ✅ strimzi.io ("expose Kafka outside Kubernetes using NodePort, Load balancer, Ingress and OpenShift Routes… secured using TLS").
- **Rack awareness**: configured declaratively so brokers are spread across availability zones and Strimzi labels each broker with its rack — the Kubernetes-native expression of KIP-36 (§10). ✅ strimzi.io ("rack awareness to spread brokers across availability zones").
- **Placement**: taints/tolerations and node affinity to run Kafka on dedicated nodes. ✅ strimzi.io ("use Kubernetes taints and tolerations to run Kafka on dedicated nodes").
- **Security** (§12): TLS everywhere by default with automated certificate management (Cluster Operator issues/rotates CA and per-broker certs), listener authentication (mTLS, SASL/SCRAM-SHA-512, OAuth 2.0), authorization (ACLs, OAuth), network policies, and FIPS-enabled-cluster readiness. ✅ strimzi.io docs, security overview.
- **Monitoring** (§11): Prometheus metrics, Kafka Exporter, Grafana dashboards, OpenTelemetry tracing, and integrated **Cruise Control** (the LinkedIn rebalancing engine, ✅ github.com/linkedin/cruise-control via Strimzi docs) for workload-based partition rebalancing through the `KafkaRebalance` resource.

### 6.3 Operating with Strimzi

The operator's value shows in the operations that used to be manual runbooks:

- **Rolling upgrades and config changes** — Strimzi rolls brokers with Kafka's own controlled-shutdown semantics, checking that replicas catch up before moving on. Declarative change → operator computes the safe rolling sequence. ✅ strimzi.io docs (automated rolling upgrades; recovery of Kafka components).
- **Scaling** — add brokers via the CR (or node-pool replica count), then rebalance with Cruise Control (`KafkaRebalance` CRs with `full`/`add-brokers`/`remove-brokers` goals) — the modern replacement for hand-built reassignment JSON. ✅ strimzi.io docs.
- **Node maintenance** — the Drain Cleaner watches Kubernetes eviction notices (from node drains/spot interruptions) and moves the leadership and data off the affected broker gracefully before the pod dies. ✅ strimzi.io docs.
- **Safe broker removal** — remove-brokers rebalance first, then scale down; the operator refuses to delete a pod whose data is still sole-copy.
- **Versions and the KRaft era** — the Cluster Operator owns the supported Kafka version matrix and the upgrade path; as of the 0.46+ line it is KRaft-only, meaning Strimzi-managed clusters on modern versions are ZooKeeper-free by construction (mirroring Kafka 4.0 §4.2).

**The honest caveats.** An operator does not remove Kafka's operational laws — it automates the safe execution of them. You still own capacity planning, topic design, retention, client governance, and the monitoring that tells you the operator's roll was safe. And operator cadence matters: Strimzi releases frequently and supports a window of Kafka versions; version-pinning and upgrade testing remain platform-team work (⚠ operational practice). CNCF Incubating status is a maturity signal, not a guarantee — the ledger records the dates so the reader can judge.

### 6.4 The Shape of a Production Declaration (Illustrative)

To make §6.2 concrete, here is the *shape* of a production `Kafka` custom resource — deliberately simplified and generic (⚠ **illustrative, not a copy-paste config**: exact API fields, defaults, and version keys change across Strimzi releases — always write against the operator version's own API reference):

```yaml
apiVersion: kafka.strimzi.io/v1beta2        # CRD API group — check current version
kind: Kafka
metadata:
  name: payments-backbone
  namespace: event-platform
spec:
  kafka:
    replicas: 3                             # brokers, one per AZ
    version: 4.x                            # Kafka version the operator supports
    rack:
      topologyKey: topology.kubernetes.io/zone   # rack awareness = AZ (§10.1)
    storage:
      type: persistent-claim                # PVC-backed (ephemeral is dev-only)
      size: 2Ti
      class: local-nvme                     # local-PV StorageClass (§9.2)
    listeners:
      - name: internal
        port: 9092
        type: internal
        tls: true
      - name: external
        port: 9094
        type: route                        # OpenShift Route, TLS-terminated
        authentication:
          type: scram-sha-512              # SASL/SCRAM (§12)
    authorization:
      type: simple                          # ACL-based (§12)
    template:
      pod:
        affinity: ...                       # spread across AZs; anti-affinity
        tolerations: ...                    # dedicated Kafka nodes (§10.1)
  entityOperator:
    topicOperator: {}
    userOperator: {}
  cruiseControl: {}                         # rebalancing engine (§10.3)
  kafkaExporter: {}                         # consumer-lag metrics (§11)
```

A separate `KafkaNodePool` CR (KRaft-era) would split `broker` from `controller` roles and could give controllers a different, smaller StorageClass. Topics (`KafkaTopic`) and users (`KafkaUser` with SCRAM credentials and ACLs) are sibling CRs in the same namespace — the whole backbone is git. This is the artifact §15's Cymbal Bank decision commits to the platform repository.

## 7. Kafka on Kubernetes — Confluent for Kubernetes

### 7.1 What CFK Is (Verified)

**Confluent for Kubernetes (CFK)** is Confluent's operator-based control plane for running the **Confluent Platform** distribution of Kafka in your own Kubernetes environment. ✅ docs.confluent.io (operator overview): "a cloud-native management control plane for deploying and managing Confluent Platform in your Kubernetes private cloud environment," driven through a declarative API of custom resources, installed as a Helm-managed Kubernetes Deployment. It is explicitly the **next generation of Confluent Operator** (the docs' version picker runs back through the 2.x line to 1.7/1.6). ⚠ The exact date CFK went GA / was renamed from Confluent Operator could not be pinned to a primary source this pass — the docs do not state it, and search returned nothing authoritative; treat "GA since the early-2020s, current major line 3.x" as approximate.

**What it deploys and manages** (✅ docs.confluent.io overview):
- Kafka brokers **and KRaft controllers or ZooKeeper nodes** — CFK's custom resources express both metadata modes (relevant for fleets still on ZooKeeper-mode Confluent Platform and the migration story, §4.2).
- The surrounding Confluent Platform components: **Connect workers, ksqlDB, Schema Registry, Control Center, REST Proxy** — plus application resources such as **topics and rolebindings** (`KafkaTopic`, `ConfluentRolebinding` CRs).
- **Upgrades** via automated rolling updates "that avoid impact to Kafka availability"; **scale-on-demand** for components and storage.
- **Resilience**: "restore a Kafka pod with the same broker ID, configuration, and storage after a failure" — the pod-identity guarantee of §10, productized; and "spread partition replicas across racks or zones with automated rack awareness," where **Kubernetes availability zones are treated as racks** ✅ (docs.confluent.io rack-awareness page).
- **Security**: RBAC, authentication, TLS with auto-generated certificates, Vault integration for credentials; **monitoring** via JMX/Jolokia aggregated and exported to Prometheus. ✅ docs overview.

**What it explicitly does not do** (✅ docs): CFK only deploys Confluent Platform on Kubernetes (bare-metal/VM installs use the manual or Ansible routes); it manages Confluent components and their application resources, not arbitrary workloads; and it manages **self-managed Confluent Platform, not Confluent Cloud** (Cloud is §13).

**Current state (observed this pass, ⚠ date-dependent):** CFK **3.3.0** (release notes dated 23 June 2026), supporting Confluent Platform **7.5.x–8.3.x** on Kubernetes **1.28–1.36** (OpenShift **4.15–4.22**). ✅ docs.confluent.io release notes. The same notes confirm KRaft-focused operations: ZooKeeper→KRaft migration with a preflight check, KRaft controller quorum scale-up without rolling existing controllers, and a DR procedure for controller-quorum loss.

### 7.2 Licensing and the Open-Core Positioning

The pivotal difference from Strimzi is **what the operator deploys**: Strimzi deploys the Apache 2.0 Kafka broker; CFK deploys Confluent Platform, the commercial distribution. Confluent Platform components are licensed under the Confluent Community License or commercial terms — the licensing split (open-source core vs source-available components vs commercial-only features) is documented in the alternatives sibling's ledger ([kafka_alternatives_guide.md](kafka_alternatives_guide.md) §1.2, repo-✅). ⚠ The exact entitlement map for CFK (which CP components require which license tier, evaluation vs enterprise terms) is commercial detail that this pass did not verify against Confluent's licensing pages — procurement must confirm against the then-current terms. What is verifiable and said plainly: **CFK itself is not an open-source project in the Apache sense; it is a vendor operator for a commercial distribution**, and the decision between §6 and §7 is therefore partly a licensing decision (open-source broker + your own Schema Registry/Control Center equivalents vs one-vendor CP stack), not purely a technology one.

### 7.3 Strimzi vs CFK — The Compact Comparison

| Dimension | Strimzi (§6) | Confluent for Kubernetes (§7) |
|---|---|---|
| Deploys | Apache Kafka (open source) + Connect/MM2/Bridge | Confluent Platform distribution (Kafka + Schema Registry, ksqlDB, Control Center, REST Proxy) |
| Project governance | CNCF Incubating, open source (✅ cncf.io) | Vendor product (docs.confluent.io) |
| Licensing of deployed stack | Apache 2.0 broker | Confluent Community License / commercial ⚠ (sibling ledger for the split) |
| KRaft | Supported; mandatory since the 0.46+ line (✅ GitHub notes) | KRaft or ZooKeeper expressible; migration tooling (✅ docs) |
| Topic/user management | Topic Operator + User Operator CRs (✅ strimzi.io) | KafkaTopic, ConfluentRolebinding CRs (✅ docs) |
| Rebalancing | Cruise Control via KafkaRebalance (✅ strimzi.io) | Cruise Control supported in CP; rack awareness automated (✅ docs) |
| Ecosystem fit | Kubernetes-native, GitOps-first, distribution-agnostic | Confluent Platform feature depth (Control Center, Schema Registry, RBAC/MDS), one-vendor support |
| Support | Community + vendor distributions (Red Hat §8) | Confluent commercial support |

The honest selection rule: **Strimzi when you want the open-source broker, Kubernetes-native operations, and your own governance stack; CFK when you want Confluent Platform's integrated component set (Schema Registry, ksqlDB server, Control Center) operated declaratively on Kubernetes with one vendor's support line.** §15 (Cymbal Bank) weighs exactly this fork.

## 8. The Distribution Layer: Red Hat Streams for Apache Kafka and the Ecosystem

**Red Hat Streams for Apache Kafka** (formerly **AMQ Streams**) is Red Hat's supported distribution of Kafka **based on the Strimzi operator project** for Red Hat OpenShift. ✅ developers.redhat.com ("based on the Apache Kafka and Strimzi open source projects"); docs.redhat.com carries the full "Streams for Apache Kafka on OpenShift" documentation set (the 2.4/3.x doc generations were observed this pass, ⚠ version date-dependent). For banks whose platform standard is OpenShift (the Cymbal Bank posture — cross-ref [secure_red_hat_openshift_guide.md](secure_red_hat_openshift_guide.md) and [openshift_workload_availability_guide.md](openshift_workload_availability_guide.md)), this is the procurement-friendly path to the Strimzi model: an OpenShift-certified operator, Red Hat support, and RHEL/OpenShift lifecycle alignment instead of upstream-only community support. Red Hat's docs also confirm the enterprise feature surface — including Cruise Control partition reassignment on JBOD storage (✅ docs.redhat.com, JBOD chapter). ⚠ Streams-for-Apache-Kafka-specific version numbers and support windows change on Red Hat's lifecycle calendar; check the then-current supported configurations before planning.

**The wider operator field** is thinner than the marketing suggests: Strimzi (open source) and CFK (Confluent) dominate the general-purpose Kubernetes space; the other serious Kafka-family operators are vendor distributions of one or the other (Red Hat above, IBM's Kafka-based Event Streams product for its own platforms ⚠ not re-verified this pass). For Kubernetes distributions beyond OpenShift (vanilla upstream, Charmed Kubernetes, OpenStack-hosted clusters), the operator model is the same — cross-ref [charmed_kubernetes_vs_openshift_guide.md](charmed_kubernetes_vs_openshift_guide.md) and [devstack_openstack_guide.md](devstack_openstack_guide.md) for the platform-mechanics differences that matter when Kafka is the tenant (storage classes, CSI drivers, node draining, SCC/PSA equivalents).

**Ecosystem rule of thumb:** choose the operator by (a) which Kafka *distribution* you want to run (Apache vs Confluent Platform), (b) which *platform* you must run it on (OpenShift → Red Hat distribution is the natural procurement route), and (c) who must *support* it. The CRD-level mechanics are broadly similar across all of them because they all reconcile the same Kubernetes statefulness laws (§10).

## 9. The Storage Realities

### 9.1 Kafka's Storage Contract

Kafka is engineered around **local disk**: log segments are appended sequentially and served from the OS page cache, and the durability contract (`acks=all`, ISR) assumes a broker that can persist and serve its own log. ✅ The design rationale is documented in the Apache Kafka documentation and, for the disk-array question, in KIP-18: **JBOD (just a bunch of disks) is Kafka's preferred storage schema**, for the same reason HDFS datanodes prefer it — per-disk operation performance and isolation beat RAID-style abstractions, with multiple `log.dirs` supported (KIP-18, ✅ cwiki, referencing KAFKA-188 for multiple log dirs). The operational corollaries:

- **Throughput and latency are disk properties.** A broker's write path is bounded by its disks (and network); the classic sizing question is "can the disk sustain the peak MB/s and the fsync cadence the durability config demands?"
- **Replication is not backup.** RF=3 protects against broker/node loss, not against logical corruption, accidental deletion, or region loss — DR is MirrorMaker 2 / cluster linking (cross-ref banking guide) or the managed service's replication features (§13).
- **Rack awareness is a placement rule, not a storage feature** — it exists so replicas land on different failure domains (KIP-36 ✅ cwiki).

### 9.2 Local Persistent Volumes vs Network Storage on Kubernetes

The Kubernetes storage taxonomy that matters for Kafka is the split the SIG-Storage KEP draws: **remote storage** (network-attached; survives pod/node movement) versus **local storage** (directly attached to one node). Kubernetes made local storage usable as persistent volumes when **Local Persistent Volumes went GA in Kubernetes 1.14** (alpha 1.7, beta 1.10) — ✅ kubernetes.io blog, March/April 2019 — precisely so that stateful, disk-performance-sensitive workloads like Kafka and databases could bind PVCs to per-node disks.

| Storage class | Kafka fit | Notes |
|---|---|---|
| **Local PV** (direct-attached NVMe/SSD/HDD, via local-static-provisioner or a CSI local driver) | **Best fit for broker log dirs** (⚠ practice consensus) | Node-affine: the pod must run on the node owning the disk — the scheduler coupling of §10. Node failure = data on that node needs re-replication from other replicas (RF=3 exists for this). |
| **Cloud block storage** (EBS via CSI, Azure Managed Disks, OpenStack Cinder via [devstack_openstack_guide.md](devstack_openstack_guide.md) estates) | **Workable — the managed-service default** | Network block, but single-writer and low-latency enough; choose the right volume type/IOPS tier. ⚠ vendor volume-type specifics (e.g., gp3/io2 tiering, provisioned IOPS) are date- and region-dependent — size against measured throughput, not marketing. |
| **Shared file storage** (NFS, CIFS, EFS-style) | **Avoid for broker log dirs** ⚠ practice guidance | Kafka's sequential-append + page-cache model and its fsync expectations sit badly on shared-file semantics; multi-writer file systems are a corruption risk for log segments. Fine for tooling/backup staging, not for logs. |
| **Object storage (S3/GCS/Azure Blob)** | Tiered storage remote tier; not the broker's primary store | Kafka 3.6+ tiered storage (KIP-405) offloads older segments to object storage — the economics change (§9.4); object-storage-*native* brokers are the alternatives sibling's territory. |

**The containerized nuance:** inside a pod, the broker's JVM heap and its page cache share the container's memory budget. Kafka's classic tuning assumes the OS page cache is *outside* the JVM heap and effectively free; under Kubernetes memory limits, the cache competes with the heap and with other pods' pressure. Practical operator guidance (⚠ held knowledge — every platform team discovers this empirically): size broker pods so the memory limit leaves headroom for page cache, or pin brokers to dedicated nodes (§10) where the cache can use node memory. This is one of the least-documented but most-common Kafka-on-Kubernetes performance mistakes.

### 9.3 The Operator Storage Model

Strimzi and CFK both express Kafka's disk needs as **PVCs**: each broker (and KRaft controller) pod gets a PVC (or several, for JBOD-style multiple volumes per broker) bound to a chosen StorageClass. ✅ strimzi.io docs (storage configuration per broker/controller via the `Kafka`/`KafkaNodePool` resources); ✅ docs.confluent.io (scale "components and storage with single commands"). The operator's job is to (a) *create* correctly-sized PVCs with the right StorageClass, (b) *reuse* the PVC when the pod is recreated — preserving the broker's data and identity, and (c) *not* delete PVCs on cluster scale-down without draining the data first. Storage expansion (PVC resize) and per-pool StorageClasses are handled through node pools in modern Strimzi (§6.2). ⚠ The specific storage-schema keywords (`ephemeral`, `persistent-claim`, `jbod`) and their per-version semantics are documented in the operator docs and change across releases — this guide records the model, not a config snapshot.

### 9.4 The Cost Curve and Tiered Storage

Kafka's economics are dominated by **storage replicated ×3**: every byte of retention costs three bytes of disk (or cloud block storage) plus the compute to serve it. The mitigations, in order of increasing architectural commitment: (1) retention tuning and compaction (delete old data, keep only keys); (2) **tiered storage** — Kafka 3.6+ GA (✅ repo-verified in the alternatives sibling's ledger) moves old segments to object storage and keeps hot segments local; (3) managed/serverless tiers that bill storage separately (§13); (4) object-storage-native brokers (WarpStream and friends — [kafka_alternatives_guide.md](kafka_alternatives_guide.md)). For the virtualization decision, tiered storage matters because it *reduces the local-disk tax*: a broker fleet whose retention is mostly cold can be sized for hot traffic with object storage absorbing the long tail — which shrinks the local-PV footprint and the node-affinity burden of §9.2. ⚠ Tiered-storage production maturity and feature parity vary by broker version and managed provider — verify per platform.

### 9.5 Sizing, Backup, and the Disk-Failure Runbook

**The sizing discipline (⚠ practice guidance — the arithmetic is illustrative, not a quote):** a broker's disk need is `peak_write_MBps × retention_seconds × replication_role_factor`, with headroom for compaction/cleanup and controller metadata where co-located. Concretely: a topic group ingesting 50 MB/s sustained with 7-day retention stores ~30 TB per copy (50 MB/s × 604,800 s ≈ 30 TB); at RF=3 that is ~90 TB across the cluster's brokers, plus the *replication write amplification* (each follower write consumes follower disk and network, so peak disk consumption is closer to 2× the produce rate when a follower is catching up). The common failure is sizing storage from *average* ingest while retention is set from *regulatory* requirements — the two multiply. ⚠ These figures are worked examples to show the shape, not sizing guarantees.

**Backup realities:** Kafka's replication is a liveness feature, not a backup — the supported data-protection toolkit is (1) replication (RF=3) for broker loss, (2) MirrorMaker 2 / cluster linking for cross-cluster and DR copies, (3) topic-level export/import tooling (Kafka's own `kafka-dump-log`/mirror tooling and the ecosystem's S3/parquet connectors) for archive, and (4) — for the metadata plane — the controller quorum's own redundancy plus its storage backup. ⚠ Point-in-time "backup" products for Kafka exist but are niche; treat "restore a topic to yesterday" as a design problem solved with retention + mirroring + archive connectors, not with snapshot products (VM snapshots were flagged in §5 for the same reason). Managed services (§13) inherit the same logic — the vendor's SLA covers availability, and their replication features cover region loss, but *your* retention and archive policy is still yours.

**The disk-failure runbook, family-independent (⚠ standard practice):** (1) detect via broker metrics — a full disk is an outage, alert well before (§11); (2) for a failed-but-replaceable disk: replace the volume/pod, let the broker rejoin and re-replicate its partitions from the ISR — no manual data recovery; (3) for a failed broker: confirm the operator/runbook restarts it with the same identity and PVC (§10.1); (4) if a whole AZ/rack fails: verify the remaining replicas cover RF (this is why rack awareness and RF=3 across AZs are non-negotiable, §10.1), and let preferred-leader election restore normal placement (§10.5). The failure that *is* a data-loss event is the one where replicas were silently co-located (rack awareness off) or minISR was set below what the business accepted — both configuration decisions, not disk events.

## 10. The Stateful Operating Model on Kubernetes

*Platform mechanics (StatefulSets, PVCs, headless services, node draining, disruption budgets, OpenShift SCCs) are the siblings' territory — [openshift_workload_availability_guide.md](openshift_workload_availability_guide.md), [secure_red_hat_openshift_guide.md](secure_red_hat_openshift_guide.md). This section is the *Kafka-specific* operating logic built on top of them.*

### 10.1 Identity, Storage, and the Three Laws

Kubernetes wants workloads to be cattle; Kafka is a herd of named animals. The operator pattern reconciles the two through three laws that every stateful Kafka deployment obeys:

1. **Stable identity.** Broker N must always be broker N: a StatefulSet's ordinal name (`kafka-2`) maps to a broker ID, and the broker's advertised listener is its stable per-pod DNS name. Clients, partition assignments, and the metadata log all reference these identities; a scheduler that renames pods breaks the cluster. CFK's "restore a Kafka pod with the same broker ID" promise (✅ docs.confluent.io) and Strimzi's StatefulSet model (✅ strimzi.io docs) are the same law expressed twice.
2. **Sticky storage.** The PVCs of §9.3 must follow the pod: delete-and-recreate a broker pod and it must come back on the same volumes, or the broker returns with an empty log and re-replicates everything — or, worse, rejoins with stale state if the old volume is mounted elsewhere. Operators encode this by never recreating pods with new PVCs unless a scale/rebalance operation says so.
3. **Placement with intent.** Brokers should be spread across failure domains (racks/AZs), optionally pinned to dedicated nodes, and never silently co-located in a way that violates RF=3's assumptions. This is **rack awareness** in its Kubernetes form: the operator labels each broker with its rack/AZ (Strimzi declarative rack awareness ✅; CFK treats AZs as racks ✅ docs.confluent.io) and Kafka's replica assignment (KIP-36, ✅ cwiki) then spreads replicas across distinct racks. Without it, a single AZ outage can take out all replicas of a partition.

### 10.2 Node Maintenance, Drains, and Disruption

Node maintenance is where stateful Kafka on Kubernetes lives or dies. When a node must drain (kernel patching, hardware, cloud maintenance), the platform evicts pods; a Kafka pod evicted without preparation takes its replicas' leaders with it. The operating toolkit: **PodDisruptionBudgets** (platform level — cross-ref [openshift_workload_availability_guide.md](openshift_workload_availability_guide.md)) and Kafka-aware eviction handling: Strimzi's **Drain Cleaner** intercepts eviction notices and first moves leadership and — for sole-copy data — triggers a reassignment off the departing broker (✅ strimzi.io docs). Best practice (⚠ operational practice): sequence node maintenance so that no more than one broker per partition's replica set is disrupted at a time, and let re-replication catch up between drains.

### 10.3 Scaling and Rebalancing

Scaling Kafka up is two steps, and operators automate the second: (1) **add brokers** — a replica-count change on the CR (or node pool); the operator starts new pods with new PVCs and lets them join the cluster; (2) **rebalance** — move partition leaders *and data* onto the new brokers, because Kafka does not redistribute automatically. Cruise Control (LinkedIn's rebalancing engine, ✅ via Strimzi docs and Red Hat docs) computes workload-aware reassignment plans and executes them with throttles; Strimzi exposes it through `KafkaRebalance` CRs (full/add-brokers/remove-brokers goals, ✅ strimzi.io docs). The reassignment mechanics (throttles, rolling, verification) are Kafka-level and documented in the platform reference — cross-ref [Apache Kafka: the Distributed Event-Streaming Platform](../banking/kafka_guide.md); this guide's point is the operating shape: **scale-up = capacity step + data movement step, and the data movement is the part with the runtime cost.** Scale-down is the same in reverse, with the extra discipline that no broker is removed while it holds sole-copy data — operators refuse that (Strimzi docs, ✅).

### 10.4 Upgrades and the KRaft Controller Layer

Rolling upgrades under an operator are declarative: change the Kafka version in the CR; the operator rolls pods one at a time with controlled shutdown and ISR verification between steps (✅ strimzi.io docs; ✅ CFK "automated rolling updates that avoid impact to Kafka availability"). The KRaft era adds the controller quorum to the roll: controller-only pods (`process.roles=controller`) are as stateful as brokers (metadata log on PVCs) and must be rolled with the same care; modern operator versions can scale the controller quorum without rolling co-located brokers (CFK 3.3.0 dynamic-quorum scale-up, ✅ docs.confluent.io). ZooKeeper-era fleets face the one-off migration project of §4.2 (Strimzi's 0.45→0.46+ cutover and CFK's migration procedure are both verified ✅).

### 10.5 Multi-Zone, Preferred Leaders, and DR

For an RF=3 cluster across three AZs, operators and Kafka cooperate on: **preferred-leader election** (leaders return to their preferred AZ after failover — keeps cross-AZ traffic predictable), **min.insync.replicas=2** so one AZ loss still allows writes (⚠ config guidance — this is the standard RF=3/minISR=2 shape, cross-ref the banking guide's durability section), and **cross-cluster DR** via MirrorMaker 2 / cluster linking, which remains a separate pipeline decision from intra-cluster replication (cross-ref [Apache Kafka: the Distributed Event-Streaming Platform](../banking/kafka_guide.md) and [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) for the semantics). Managed services (§13) productize the same pattern — MSK's multi-AZ provisioned clusters and Confluent's cluster linking are the vendor versions, with the same RPO/RTO thinking underneath.

### 10.6 Two Runbook Sketches (Stateful, in Practice)

**Runbook 1 — scale the cluster from 3 to 6 brokers (⚠ illustrative steps; the operator commands are expressed generically):** (1) confirm headroom exists in the *platform* (nodes, AZ balance) and in Kafka's partition-count budget; (2) scale the broker replica count in the CR from 3 to 6 (or grow the node pool) — the operator starts `kafka-3`…`kafka-5`, each with its own PVC, and waits for them to join the ISR-less state as empty brokers; (3) run a Cruise Control `KafkaRebalance` with the `add-brokers` goal so partition leaders *and follower replicas* move onto the new brokers; (4) monitor rebalance progress (throttles keep the data movement off the critical path — cross-ref the banking guide for reassignment mechanics), verify the plan completed, and confirm topic RF is still satisfied per rack; (5) update capacity documentation. The error to avoid: scaling brokers and never rebalancing — the cluster is then "bigger" but the old brokers still hold all the load.

**Runbook 2 — a node must drain for a kernel patch (⚠ illustrative):** (1) the platform cordons the node and begins eviction; (2) the Drain Cleaner intercepts the eviction notices for Kafka pods and, before the pod dies, moves partition leadership off the affected broker (and, for any sole-copy partition, asks Cruise Control to reassign it — the safe-eviction contract of §10.2); (3) the pod terminates; the StatefulSet reschedules it on a healthy node, where it re-attaches to its PVC (local PVs are node-affine — if the disk is on the dead node, the pod must wait for the node or the data re-replicates from remaining replicas after the operator intervenes, §9.2); (4) re-replication completes and ISR returns to full; (5) uncordon and repeat for the next node — never draining more than one broker per replica set at a time (§10.2). The operator's drain handling is verified behaviour ✅ (strimzi.io docs); the sequencing discipline around it is the platform team's (⚠ practice).

These runbooks are why the stateful operating model is a *model* and not a checklist: the same two sequences look different on family 1 (manual reassignment JSON, manual leadership moves) and family 3 (vendor consoles and support tickets), but the invariants — never lose the last replica, never orphan the data, never roll past the ISR — are identical in all three.

## 11. Monitoring and Observability

*The reliability discipline (consumer-lag math, alert thresholds, DLQ depth, data-loss signatures) is documented in [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) — §11 there is the lag-monitoring deep-dive; observability-stack mechanics live in the platform siblings. This section is the Kafka-on-virtualized-infrastructure monitoring map.*

**The metric tiers.** A Kafka deployment generates four distinct observability surfaces, and an operating team needs all four:

1. **Broker/controller metrics (JMX).** Kafka exposes a rich JMX metric set: under-replicated partitions, offline partitions, ISR shrink/expand rates, request-handler-idle ratio, network-thread utilization, produce/fetch request rates and latency percentiles, log-flush rates, and (KRaft era) controller-quorum and metadata-log metrics. Exporters bridge JMX to the metrics platform: **JMX Exporter/Jolokia** (CFK aggregates JMX via Jolokia and exports to Prometheus ✅ docs.confluent.io; Strimzi ships Prometheus-compatible broker metrics ✅ strimzi.io docs), and **Kafka Exporter** (a dedicated exporter for consumer-group lag and topic/broker views ✅ strimzi.io docs, github.com/danielqsj/kafka_exporter).
2. **Consumer lag.** The distance between produced and committed offsets — the single most important application-health signal. Burrow (LinkedIn's lag evaluator) and Kafka Exporter both feed lag dashboards; alert on per-partition lag, not averages (✅ sibling-verified in [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) §11, which carries the threshold guidance).
3. **Operator/control-plane health.** The operator itself must be monitored: Strimzi Cluster Operator health, CR reconciliation failures (a `Kafka` CR stuck in `NotReady` is a cluster incident waiting to happen), Cruise Control's own metrics when rebalancing is active. GitOps adds the drift signal: does the live cluster match the git state?
4. **The substrate.** Pod restarts, PVC fill rates (a full broker disk is a hard outage — alert at 75–80% ⚠ threshold guidance), node drains, and — for the platform itself — the OpenShift/Kubernetes control-plane health that §10 depends on (cross-ref [openshift_workload_availability_guide.md](openshift_workload_availability_guide.md)).

**The standard stack.** Prometheus (scrape + Alertmanager routing) and Grafana (dashboards) are the de facto standard in the operator world — both Strimzi (✅ strimzi.io docs, with example configs) and CFK (✅ docs.confluent.io Prometheus export) document it; OpenTelemetry is the emerging trace/metrics unification path (Strimzi supports it ✅). Managed services replace the collection tier but not the dashboarding: MSK publishes broker metrics to Amazon CloudWatch and supports Prometheus-compatible scraping (⚠ specifics version/region-dependent — AWS docs), Confluent Cloud exposes a Metrics API plus integrations (✅ docs.confluent.io cloud overview: "Export cluster metrics and logs to external monitoring tools like Datadog and Prometheus").

**SLOs and the honest practice.** The virtualization family changes *who* watches, not *what* matters. The SLO set is family-independent: availability of produce/consume, end-to-end latency percentiles, max consumer lag, under-replicated-partition count held at zero, offline partitions at zero, and (for banks) the audit trail of config and access changes. ⚠ Actual SLO numbers in this guide would be invented — the banking guide and the platform's own risk appetite set the targets; the monitoring map above is what measures them.

**The alert table (⚠ thresholds are starting points to tune, not standards).** A minimal production alert set, mapped to the metric tier:

| Alert | Signal | Why it matters |
|---|---|---|
| Under-replicated partitions > 0 for > N minutes | Broker JMX (`kafka.server:UnderReplicatedPartitions`) | Replication is falling behind or a broker is stuck — the prelude to data-availability loss |
| Offline partitions > 0 | Broker/controller JMX (`OfflinePartitions`) | A partition has no leader — producers/consumers for it are down; page the team |
| Consumer lag above per-partition threshold | Kafka Exporter / Burrow (tier 2) | Processing is falling behind; per-partition alerting, never average-only (§11 tier 2, sibling-verified) |
| Request handler idle < 30% for > N minutes ⚠ | Broker JMX (`RequestHandlerAvgIdlePercent`) | Brokers are CPU/scheduler-bound — capacity or hotspot problem |
| Disk usage > 75–80% ⚠ | PVC/node metrics (tier 4) | A full broker disk is a hard outage (§9.5); give retention/cleanup time to act |
| Controller quorum degraded | KRaft controller metrics (tier 1) | The control plane is at risk (§4.3) |
| CR reconciliation failing / operator down | Operator metrics + GitOps drift (tier 3) | Declared state is not converging — the cluster is drifting from git |
| Produce/fetch p99 latency breach | Broker JMX latency percentiles | End-to-end SLO leading indicator |

The discipline that keeps this table honest: every alert must name the runbook it triggers (scale, rebalance, drain, restore) — an alert without a runbook is noise, and Kafka's are mostly §9.5/§10.6-shaped.

## 12. Security in the Virtualized Estate

*TLS/certificates-in-containers mechanics are the dedicated sibling's territory — [container_certificates_guide.md](container_certificates_guide.md); OpenShift-specific enforcement (SCCs, network policies, FIPS, audit) is in [secure_red_hat_openshift_guide.md](secure_red_hat_openshift_guide.md). What follows is the Kafka-layer security model and how virtualization changes it.*

**The Kafka security model, condensed** (cross-ref [Apache Kafka: the Distributed Event-Streaming Platform](../banking/kafka_guide.md) for depth): three independent layers — **encryption in transit** (TLS on listeners), **authentication** (who is connecting: mTLS, SASL/SCRAM, SASL/OAUTHBEARER, or plain), and **authorization** (what they may do: ACLs per principal on topics/groups/clusters; Confluent's MDS/RBAC for finer-grained role bindings). Kafka 4.0-era brokers enforce all three natively; the ZooKeeper-era authorizer subtleties died with the ensemble (§4).

**How the operators productize it (all ✅, sources inline):**
- **Strimzi** (§6.2): TLS between all components by default; automated CA and certificate management (the operator issues and rotates the cluster CA and per-listener/per-broker certs); listener authentication via mTLS, SASL/SCRAM-SHA-512, or OAuth 2.0; ACL-based authorization (managed through `KafkaUser` CRs); network policies; and FIPS-enabled-cluster readiness (strimzi.io security overview).
- **CFK** (§7.1): "secure by default" — RBAC, authentication, TLS with auto-generated certificates, HashiCorp Vault integration for credentials; `ConfluentRolebinding` CRs for MDS role bindings; credential redaction in operator logs (docs.confluent.io overview + 3.3.0 release notes).
- **Managed services** (§13): the vendor owns broker-side security but exposes the client-facing controls — MSK: IAM-based authentication, PrivateLink private connectivity, TLS (✅ aws.amazon.com MSK Serverless what's-new lists IAM + PrivateLink; provisioned MSK docs carry the full matrix); Confluent Cloud: SSO/SAML/OAuth/OIDC, mTLS, RBAC with scoped roles, Schema Registry-level governance (✅ docs.confluent.io cloud overview).

**What virtualization adds to the threat model:** (1) **certificate lifecycle at scale** — operators and pod churn multiply certs; automated rotation (operator CAs, cert-manager) is not optional, and cross-ref [container_certificates_guide.md](container_certificates_guide.md) for the trust-chain mechanics; (2) **network policy hygiene** — Kafka listeners should be reachable only from authorized namespaces/tenants (Strimzi network policies ✅), enforced by the platform's NetworkPolicy/SCC layer on OpenShift (cross-ref [secure_red_hat_openshift_guide.md](secure_red_hat_openshift_guide.md) §6); (3) **secret management** — SCRAM credentials, keystores, and Vault/cloud-KMS integration are now platform concerns (Vault ✅ CFK); (4) **encryption at rest** — PVCs inherit the platform's storage-class encryption (cloud disk encryption, OpenShift data-layer encryption — cross-ref the OpenShift guide's data layer); (5) **audit** — on OpenShift, the platform audit log plus Kafka's own authorizer logs and the GitOps history of CR changes form the regulator-facing trail (Cymbal Bank's MAS/TRM angle in §15).

**The container-identity subtlety worth stating plainly:** inside Kubernetes, "the client" is a pod with a service account; mapping pod identity to Kafka principal (mTLS cert per app, or OAuth with the pod's projected identity) is a design decision the platform team owns — the operators give you the mechanisms, not the policy. That policy is exactly what the [ai_platform_engineering_guide.md](ai_platform_engineering_guide.md) sibling frames as platform-engineering governance.

**The security checklist (condensed, family-aware).** Whatever the virtualization family, a bank-grade Kafka estate should be able to answer "yes" to each (mechanisms verified in the sections cited; the checklist shape is practice ⚠):

| Control | Family 1/2 (self-managed) | Family 3/4 (managed) |
|---|---|---|
| Encryption in transit, client to broker | TLS listeners; operator-managed certs (✅ §12) | Vendor TLS endpoints; mTLS where supported (✅ §13) |
| Encryption at rest | Storage-class/disk encryption; OpenShift data layer (cross-ref [secure_red_hat_openshift_guide.md](secure_red_hat_openshift_guide.md)) | Cloud disk encryption on vendor volumes ⚠ verify per provider |
| Authentication | mTLS / SASL-SCRAM / OAuth via operators (✅ §6–§7) | IAM/SSO/OAuth per provider (✅ §13) |
| Authorization | ACLs via `KafkaUser`; MDS rolebindings via CFK (✅ §6–§7) | Provider RBAC/IAM policies ⚠ map to Kafka ACLs carefully |
| Secrets | Operator CRs + Vault/KMS integration (✅ §7) | Vendor KMS/secret integration ⚠ |
| Network isolation | NetworkPolicies; SCCs on OpenShift (✅ §6; cross-ref OpenShift siblings) | PrivateLink/private endpoints (✅ MSK; ⚠ others) |
| Audit trail | Kafka authorizer logs + GitOps CR history + platform audit (✅ §6, §12) | Vendor audit logs + your governance layer ⚠ export what you need at contract time |
| Key/cert rotation cadence | Automated operator rotation (✅ §6–§7) | Vendor-managed; verify rotation SLAs ⚠ |

The pattern to notice: **every control exists in every family, but in managed families several of them are *contractual* rather than *configurational*** — which is exactly the evidence-trail shift the Cymbal Bank analysis (§15) weights against families 3–4 for regulated data.

## 13. The Managed Services: Confluent Cloud, AWS MSK, Aiven

*When-not-Kafka and the full managed-vs-alternative matrix are the alternatives sibling's job ([kafka_alternatives_guide.md](kafka_alternatives_guide.md)); this section is the virtualization-operations view of the big three managed Kafka services — what is verified, what is marketing, and what operating reality each implies.*

### 13.1 Confluent Cloud

**What it is (✅ docs.confluent.io cloud overview):** "a fully managed data streaming platform, available on AWS, Google Cloud, and Azure, with a cloud-native Apache Kafka engine for elastic scaling, enterprise-grade security, stream processing, and governance." The cloud-native engine is branded **Kora** (✅ docs: "elastic cloud Kafka service built on the Kora engine"). The platform bundles managed Kafka Connect, Schema Registry and Stream Governance (lineage, data contracts), ksqlDB, managed Flink, and **Cluster Linking** for cross-cluster/region mirroring "without deploying extra infrastructure" (✅ docs). Confluent itself was founded in 2014 by Kafka's creators (✅ repo-verified in the alternatives sibling's ledger).

**The operating reality (⚠ where marketing needs calibration):** Confluent Cloud removes broker operations but introduces its own governance surface — cluster tiers (basic/standard/dedicated/enterprise with different isolation and feature sets ⚠ tier names/features change), per-cluster or per-usage pricing, egress costs, quota management, and vendor-lock analysis for a platform that now spans three clouds. "Fully managed" is accurate for the broker; the *data governance, schema evolution, and cost* work remains yours. Kora's internals are Confluent's — treat any specific "how it differs from open-source KRaft brokers" claim as vendor marketing ⚠ unless verified in Confluent engineering posts.

### 13.2 AWS MSK (Amazon Managed Streaming for Apache Kafka)

**Verified timeline (correcting a common error):** MSK was announced in **open preview at AWS re:Invent in November 2018** (✅ AWS blog: "At re:Invent 2018, we introduced in open preview…"; ✅ TechCrunch, 29 November 2018) and reached **general availability on 30 May 2019** (✅ AWS press release, 30 May 2019). **The frequent claim "MSK GA'd in late 2018" is wrong** — ❌ (November 2018 was the *preview*; GA followed in May 2019 — recorded in the Claims Audit). **MSK Serverless** launched in public preview at re:Invent 2021 and went **generally available on 28 April 2022** (✅ aws.amazon.com what's-new, 28 April 2022): automatic compute/storage provisioning, throughput-based pricing ("hourly rate per cluster… per partition… per GB of data throughput and storage"), no minimum fees, IAM authentication, PrivateLink, GDPR compliance and a 99.9% availability SLA (✅ same page).

**The operating reality:** provisioned MSK runs Apache Kafka brokers in your account across AZs; AWS handles broker health, patching/upgrades, and replication plumbing, while you size the cluster (broker instance type, storage), own topics/clients/monitoring, and pay AWS rates. MSK Serverless removes the sizing step but constrains you to the serverless envelope (partition/throughput limits and pricing shape — ⚠ limits are published AWS numbers that change; check current docs). MSK is Kafka-API-compatible by construction (it *is* Apache Kafka), which is its core advantage over protocol-reimplementations — see the alternatives sibling for the compatibility debate. ⚠ "Fully managed" here still leaves real work: capacity decisions (provisioned), client/authz governance, schema governance (MSK integrates Glue Schema Registry ⚠), and cross-region DR (MSK has Replicator; MirrorMaker 2 also runs on MSK Connect — ⚠ feature names/versions change).

### 13.3 Aiven for Apache Kafka

**Verified:** Aiven is headquartered in **Helsinki** (✅ aiven.io — "We are a global team headquartered in Helsinki"; registered address Aiven Oy, Helsinki). The CEO is **Oskari Saarenmaa** (✅ aiven.io leadership page), described as founder/cofounder in multiple independent profiles. **Founding year 2016** (with co-founders Hannu Valtonen, Heikki Nousiainen and Mika Eloranta, alumni of F-Secure) is consistently reported by multiple independent secondary sources, including a 2021 interview with the CEO co-founder — ⚠ **flagged: aiven.io's own pages do not state the founding year**, so 2016 is secondary-source-verified, not primary-source-verified. Aiven positions as a multi-cloud managed *open-source* data platform (Kafka, PostgreSQL, OpenSearch, ClickHouse and more run on AWS/GCP/Azure/other clouds under one console — ✅ aiven.io positioning and case-study mix; ⚠ feature specifics per service change).

**The operating reality:** Aiven's angle is open-source parity across clouds (run Kafka, plus the ecosystem services, from one vendor) rather than proprietary extensions; Karapace (Aiven's open-source Schema Registry/console-compatible component) is part of that story ⚠ not re-verified this pass — check current docs. The same managed-service caveats apply: egress costs, per-node/per-service pricing, and the "which cloud region holds the data" residency question — the vendor runs the brokers, but *you* still choose where.

### 13.4 The Rest of the Managed Field (Brief)

Azure Event Hubs' Kafka endpoint, Google Managed Kafka (announced 2024), IBM Event Streams, and the smaller multi-cloud players all exist; their virtualization-operations story is the same shape (vendor-run brokers, API-compatible or native endpoints) and their head-to-head specifics belong to the alternatives sibling, whose ledger carries what is repo-verified (e.g., Event Hubs Kafka endpoint 2018 ✅ sibling) versus ⚠ (Google Managed Kafka availability not verified this pass).

### 13.5 The Managed-Service Reality Check (⚠ Marketing Calibration)

Three claims deserve skepticism on every managed-Kafka datasheet: (1) **"fully managed"** never covers your topics, schemas, clients, quotas, or cost governance — those shift to your platform team in new forms; (2) **"elastic/auto-scaling"** applies to compute within the vendor's envelope — storage, retention, partitions and egress still need planning; (3) **SLA percentages** compensate availability, not data loss or performance — read the SLA's exclusions before designing around it. None of this makes managed services a bad choice; it makes them a *different operating model*, which is exactly what §14 scores.

### 13.6 The Managed Big Three at a Glance (⚠ Marketing-Calibrated)

| Dimension | Confluent Cloud | AWS MSK (provisioned) | Aiven for Apache Kafka |
|---|---|---|---|
| Clouds | AWS, GCP, Azure (✅ docs.confluent.io) | AWS | AWS, GCP, Azure, others ⚠ (✅ multi-cloud positioning; verify region lists) |
| Engine | Cloud-native engine branded **Kora** (✅ docs) | **Apache Kafka** itself (✅ by construction — it *is* Kafka) | Apache Kafka (✅ open-source positioning) |
| Kafka version control | Vendor-managed ⚠ | You choose from supported versions (✅ provisioned-MSK design; exact window ⚠) | Vendor-managed ⚠ |
| Elastic tier | Elastic clusters (⚠ tier names change) | **MSK Serverless** (✅ GA 28 Apr 2022, throughput pricing) | ⚠ check current offering |
| Schema/governance | Schema Registry, Stream Governance, Data Portal (✅ docs) | Glue Schema Registry integration ⚠ | Karapace ⚠ (not re-verified) |
| Cross-region | **Cluster Linking** (✅ docs) | MSK Replicator ⚠; MM2 via MSK Connect | ⚠ check current docs |
| Auth models | SSO/SAML/OAuth/OIDC, mTLS, RBAC (✅ docs) | IAM, mTLS, SASL/SCRAM (✅ IAM+PrivateLink on what's-new page) | ⚠ standard Kafka SASL/TLS; verify |
| Pricing shape | Cluster tier + usage + egress ⚠ | Broker-hour + storage + egress; serverless: throughput + partition-hour + storage (✅ what's-new) | Node/hour + storage + egress ⚠ |
| Best-fit archetype | Multi-cloud, governance-heavy estates that want the full Confluent stack | AWS-committed estates that want *Apache Kafka* semantics with AWS plumbing (IAM, PrivateLink, CloudWatch) | Multi-cloud open-source estates wanting one vendor across many data services |

**The exit question (⚠ structural, not a quote):** leaving any of the three is a *data-movement* project (topic-by-topic mirror out over the Kafka protocol — the portability layer of §14.3), not a rewrite. What you cannot take with you: vendor-specific governance artifacts (schemas' home, lineage, RBAC role definitions), billing history, and whatever proprietary engine behaviour you tuned against. Budget the exit as "one MM2 pipeline + governance re-creation," which is real but bounded — and say no to any vendor architecture that makes the topics themselves unmovable.

## 14. The Decision Surface and TCO

### 14.1 The Decision Factors

| Factor | What to ask | Families it favors |
|---|---|---|
| **Team and skills** | Do we have Kafka operators, or Kubernetes operators, or neither? | Neither → 3/4; K8s-strong → 2; Kafka-ops-strong on-prem → 1 |
| **Control and customization** | Do we need custom broker configs, pluggable auth, specific Kafka versions, on-prem placement? | 1, 2 (and 3 partially — provisioned MSK allows more broker config than serverless) |
| **Compliance/residency** | Where must data physically live? Who must hold the evidence? | On-prem/sovereign → 1/2 in-region; cloud with region pinning → 2/3 in chosen region |
| **Scale shape** | Steady high throughput vs spiky/seasonal vs many small tenants | Steady → 1/2/3 provisioned; spiky or many small → 4 (serverless) or per-tenant 3 |
| **Cost structure** | Upfront vs monthly vs usage; egress; storage ×3; ops FTEs | See §14.2 TCO sketch |
| **Existing platform** | Is OpenShift the standard? Is there a storage/backup/observability platform already? | OpenShift standard → 2 via Red Hat/Strimzi (§8); mature platform → 2 leverages it |
| **Multi-cloud/DR** | One cloud or three? Region failover required? | Multi-cloud → 3 (Confluent Cloud/Aiven span clouds) or self-managed MM2 across clouds |
| **Procurement/licensing** | Open-source vs commercial entitlements; vendor risk appetite | ⚠ per-institution; see §7.2 |

### 14.2 The TCO Sketch (Illustrative, Not a Quote)

⚠ **The numbers below are structural illustrations, not vendor quotes** — every cost is environment- and negotiation-dependent; use them to build your own model, never as procurement input. A production RF=3 Kafka cluster's cost stack has four components regardless of family: **compute ×3** (brokers + controllers), **storage ×3 + replication overhead + retention**, **network/egress**, and **operations** (FTEs × time on patching/upgrades/incidents/DR drills — the family-dependent term).

- **Family 1 (VMs, self-managed):** cheapest *unit* prices (or sunk on-prem capacity), highest FTE term; KRaft-era migration is a project line item (§4.2). Predictable but people-heavy.
- **Family 2 (operator on existing OpenShift/K8s):** compute/storage at platform rates (often already budgeted); adds operator maintenance and the stateful-care of §10 to the platform team; no per-cluster license if open-source (Strimzi), or CP licensing if CFK (§7.2). The typical winner when the Kubernetes platform already exists and the team is platform-engineering-shaped.
- **Family 3 (managed provisioned):** per-cluster/hour/node pricing plus storage and egress; FTE term shrinks but vendor-management and cost-governance grow; the "three brokers minimum" tax becomes a per-cluster minimum fee. Wins when Kafka-ops skills are the bottleneck or the estate is small enough that a dedicated Kafka team is unaffordable.
- **Family 4 (serverless/elastic):** usage pricing (throughput + storage + partition-hour) — excellent for spiky/dev workloads, poor for steady multi-TB/s estates where provisioned wins on unit economics. ⚠ cross-check against the alternatives sibling's cost analysis before assuming serverless is cheaper at scale.

**The honest TCO rule:** the family decision moves *where* the money lands (capex/opex split, FTE hours, vendor bill) more than it changes the *total* — Kafka's data economics (×3 storage, retention, egress) follow you into every family. Models that "prove" one family is 10× cheaper usually forgot the ×3 replication or the FTE time of the alternative.

### 14.3 Hybrid Patterns (The 2026 Norm)

Few estates are single-family. The mature patterns: (a) **criticality split** — regulated core on family 1/2 in-region, analytics/dev on 3/4; (b) **DR pairing** — self-managed primary mirrored to a managed secondary (or vice versa) via MirrorMaker 2/cluster linking, so the DR site has no Kafka-ops dependency; (c) **platform-to-cloud burst** — the OpenShift estate runs steady-state Kafka, managed serverless absorbs campaign/event spikes; (d) **multi-region residency** — per-region clusters (any family) linked for global events while each region's data stays local. The common thread: **the Kafka protocol and the replication tooling are the portability layer** — because every family speaks the same protocol, hybrids are *operationally* feasible, which is Kafka's quietest advantage over protocol-incompatible alternatives (cross-ref the alternatives sibling).

### 14.4 The Scoring Template (Use It, Don't Read It)

A reusable decision table for a Kafka-virtualization selection, matching §14.1's factors (⚠ weights are examples; a bank's weights come from its own risk appetite and board priorities):

| Factor | Weight (example) | Family 1 (VMs) | Family 2 (operator) | Family 3 (managed) | Family 4 (serverless) |
|---|---|---|---|---|---|
| Control & customization | 20% | 5 | 4 | 2 | 1 |
| Residency & compliance evidence | 20% | 5 | 4 | 2–3 ⚠ | 2 ⚠ |
| Operating cost (FTE + infra) | 20% | 2 | 3 | 4 | 4 |
| Speed to value | 15% | 2 | 3 | 5 | 5 |
| Scale elasticity | 15% | 1 | 2 | 3 | 5 |
| Multi-cloud / portability | 10% | 3 | 4 | 3–5 ⚠ | 3 ⚠ |
| **Weighted total** | 100% | **~3.0** | **~3.3** | **~3.2** | **~3.1** |

Read the totals as *evidence that the families land close once weighted* — the differentiated rows are the ones your institution weights highest, not the sum. The template's real output is the two or three rows where your weights are extreme (a regulator that demands on-prem evidence, a cost committee that caps opex, a developer-experience mandate that demands self-service): those rows pick the family, and the total merely confirms the runner-up for the hybrid pattern of §14.3. §15 shows the template applied with Cymbal Bank's actual weights (control and residency dominant), which is why its result is a hybrid with the regulated core on family 2.

## 15. Worked Example: The Cymbal Bank Kafka-Virtualization Decision

*Persona rule: Cymbal Bank is the only bank persona in this repository's worked examples. Everything below is a worked architecture exercise in the Cymbal Bank voice — the governance frame, MAS/TRM references, and figures are illustrative of the repo's Cymbal Bank context, not a description of any real institution.*

### 15.1 The Scenario

Cymbal Bank runs its Singapore-regulated estate on a hardened **Red Hat OpenShift** platform (the platform siblings' context — [secure_red_hat_openshift_guide.md](secure_red_hat_openshift_guide.md), [openshift_workload_availability_guide.md](openshift_workload_availability_guide.md)). It already runs an event backbone on an ageing ZooKeeper-mode Kafka fleet on VMs (§5, family 1) — three environments, mixed Kafka versions, a ZooKeeper ensemble per cluster, and an upgrade calendar that is starting to hurt. The events flowing through it: payment confirmations, trade-lifecycle events, reference-data changes, CDC from core systems, and regulatory-reporting feeds (the topic taxonomy is designed in the banking guide's worked example — [Apache Kafka: the Distributed Event-Streaming Platform](../banking/kafka_guide.md) §10).

The mandate from the architecture board: **modernize the backbone in the KRaft era** — get off ZooKeeper-mode Kafka, choose a virtualization family for the next five years, and justify it with an operating-model and TCO analysis. Constraints: MAS-regulated data residency (Singapore region for regulated data), the OpenShift platform standard, an existing platform-engineering team that runs OpenShift well but has no dedicated Kafka-ops bench, and a hard requirement to keep the *Kafka protocol* stable so ~40 consumer applications are untouched.

### 15.2 The Options (Families 2 and 3, with 1 as the status-quo baseline)

| Option | What it is | KRaft path | Best-fit constraint |
|---|---|---|---|
| **A. Strimzi on OpenShift** (family 2, open source — possibly via the Red Hat Streams distribution, §8) | Apache Kafka CRs on the existing platform; platform team operates with operator automation | Native — Strimzi is KRaft-only on modern lines (§6.3); the ZooKeeper fleet migrates once via the operator's tooling | Control, residency, OpenShift leverage, no per-cluster license |
| **B. Confluent for Kubernetes** (family 2, commercial) | Confluent Platform on OpenShift: broker + Schema Registry + ksqlDB + Control Center under CFK CRs (§7) | CFK migration procedure with preflight (§7.1) | Wants the CP component stack and one-vendor support; accepts licensing ⚠ (§7.2) |
| **C. Managed service in-region** (family 3 — AWS MSK or Confluent Cloud or Aiven in the Singapore region, §13) | Vendor-run brokers behind an API; bank operates governance and cost | Vendor-owned (all three run modern Kafka; ⚠ verify each provider's exact Kafka version/KRaft status at contract time) | No Kafka ops bench, smallest FTE term, fastest time-to-value |
| **D. Hybrid** — A (or B) for the regulated core in Singapore + C for non-regulated/analytics/dev | Two families, one protocol, MM2/cluster-linking between them (§14.3) | Per-cluster as above | Split criticality; DR without Kafka-ops at the DR site |

### 15.3 The Analysis

Scoring against the §14.1 factors (Cymbal Bank's weights in parentheses — control 30%, residency/compliance 25%, operating cost 20%, speed 15%, multi-cloud optionality 10%):

- **A (Strimzi on OpenShift)** — strongest on control and residency: data stays on the platform whose security/audit story the bank already defends to MAS; SCC/network-policy/FIPS machinery is in place (cross-ref [secure_red_hat_openshift_guide.md](secure_red_hat_openshift_guide.md)); the incremental skillset is *Kafka-specific*, which the platform team partially has. KRaft removes the ZooKeeper tax that made the VM fleet painful. Weakest on FTE speed: upgrades, capacity, and the stateful care of §10 stay in-house (mitigated by operator automation). Licensing cost ≈ zero (open source; or the Red Hat subscription if the bank buys support — procurement-friendly on OpenShift).
- **B (CFK)** — same family-2 profile plus the integrated Confluent Platform stack (Schema Registry/ksqlDB/Control Center operated declaratively) — attractive if Cymbal Bank wants those components without assembling its own; the cost is commercial licensing ⚠ and a second vendor's operator cadence alongside OpenShift's. Chosen only if the component-stack pull is real; otherwise A delivers the same broker outcome more cheaply.
- **C (Managed)** — best on speed and FTE, weakest on the two highest-weighted factors: residency is *possible* (Singapore region exists for all three — MSK's Singapore availability was confirmed on the MSK Serverless what's-new page ✅; Confluent Cloud and Aiven operate in Singapore ⚠ verify region lists at contract time) but the evidence trail for MAS now crosses a vendor boundary, and control over broker config/versions is capped. Cost is opex-favourable at small scale and punishing at steady multi-GB/s (§14.2). The right choice for Cymbal Bank's *dev/test and analytics* tiers, not the regulated core.
- **D (Hybrid)** — the board's actual winner: **A for the regulated event backbone in Singapore** (payments, trade lifecycle, CDC, regulatory feeds — data that must not leave the region and whose audit trail must be unimpeachable), **C for everything else** (dev/test elasticity, analytics, innovation sandboxes), with MirrorMaker 2/cluster linking between the families for the handful of cross-tier topics and as the DR story (a managed secondary in the same region or a second region — DR site with zero Kafka-ops dependency, §14.3b). One protocol across both families keeps the 40 consumer applications untouched — the portability argument of §14.3 is what makes D cheap.

### 15.4 The Decision and the Roadmap

**Decision (recorded in the Cymbal Bank architecture-decision format — see [architecture_decision_record_guide.md](architecture_decision_record_guide.md) for the record-keeping discipline):** migrate the regulated event backbone to **Strimzi on OpenShift** (procured via the Red Hat Streams distribution for support alignment, pending procurement's licensing review), and adopt **a managed Kafka service for non-regulated tiers** — this is decision **D**, with A as the core. Rationale in one paragraph: the two highest-weighted constraints (control and residency) point to the platform Cymbal Bank already operates and already defends to the regulator; the KRaft era removed the single heaviest operational tax (ZooKeeper) that made the VM fleet painful; the operator model (§6–§7, §10) converts Kafka's statefulness from a manual discipline into declared, GitOps-versioned configuration on the platform the team already runs; and the managed tier buys elasticity and FTE relief exactly where the regulatory weight is lowest.

**The roadmap (⚠ dates and versions are planning placeholders — validate against then-current operator/Kafka releases):**
1. **Foundation (Q4 2026):** stand up a Strimzi/Red Hat Streams Kafka cluster (KRaft mode, RF=3 across three OpenShift worker/AZ failure domains, node pools separating brokers from controllers, local-PV StorageClass for broker logs — §9.2, §10.1) in the Singapore region; wire TLS/SASL and the OpenShift network-policy/SCC posture (§12) from day one; attach Prometheus/Grafana + Kafka Exporter + Cruise Control (§11).
2. **Migration (2027):** move the ZooKeeper-mode VM fleets over — not as an in-place KRaft migration but as **new-cluster + MirrorMaker 2 cutover** per workload class (lower risk than the bridge migration for a fleet this old; the banking guide's §3.6 has the in-place path for any cluster the bank prefers to migrate rather than rebuild). Drain and decommission the ensembles and VM brokers per environment.
3. **Managed tier (2027):** stand up the managed service for dev/test and analytics; connect the tiers with MM2/cluster linking for the approved cross-tier topics; define the cost-governance and quota model (§13.5).
4. **Operating model (ongoing):** the platform team owns the OpenShift backbone with operator automation; an SRE squad owns the managed tier's vendor relationship; SLOs, DR drills (including the MM2 failover drill), and the MAS-facing audit trail are platform-wide — this is the [ai_platform_engineering_guide.md](ai_platform_engineering_guide.md) operating-model shape applied to Kafka.

**What Cymbal Bank did *not* do (and why):** it did not pick a single family for everything (the cost and control penalties of running the regulated core on a vendor boundary, or of running dev/test on self-managed brokers, both failed the scoring); it did not adopt CFK without the Confluent Platform component pull (licensing cost with no compensating need — §7.3); it did not treat managed as "no operations" (it budgeted vendor management and cost governance — §13.5); and it did not let the ZooKeeper-era fleet limp on (the KRaft cutover was scheduled as a project with an owner, because the tax compounds — §4.2). The backbone stays Apache-Kafka-protocol end to end, which is what lets the bank mix families without rewriting a single consumer.

### 15.5 The Decision Record, One Page

| Element | Content |
|---|---|
| **Decision** | Regulated event backbone → **Strimzi on OpenShift** (Red Hat Streams distribution pending procurement review); non-regulated tiers → **managed Kafka service**; MM2/cluster linking between families. Hybrid D, core on family 2. |
| **Context** | ZooKeeper-mode VM fleet aging; MAS data residency; OpenShift platform standard; no dedicated Kafka-ops bench; ~40 consumers must not change protocol. |
| **Rationale** | Control (30%) and residency (25%) dominate the weights → family 2 on the platform the bank already defends; KRaft removed the heaviest tax (§4.2); operator model makes statefulness declarative (§5–§6, §10); managed tier buys FTE relief where regulatory weight is lowest. |
| **Consequences (positive)** | One protocol estate; GitOps backbone; vendor-free core; DR site without Kafka-ops dependency; dev/test elasticity. |
| **Consequences (negative/risks)** | Platform team owns broker upgrades/capacity (mitigated by operator automation); managed tier needs cost governance (§13.5); vendor-specific governance artifacts at the managed tier if exit is ever needed (§13.6); migration is a scheduled project with owner (§15.4 step 2). |
| **Accepted trade-off** | Slightly higher FTE load than all-managed (option C) in exchange for control and residency evidence; slightly higher cost than all-self-managed (option A only) in exchange for elasticity and speed. |
| **Review trigger** | Annual: re-score §14.4 when platform costs, team size, or regulatory posture change; on Kafka major-version boundaries (the 4.x line, §4). |

The one-page record is the artifact the architecture board approved; the [architecture_decision_record_guide.md](architecture_decision_record_guide.md) sibling is the discipline that keeps it honest as conditions change.

## The Glossary

- **Kafka virtualization** — running and operating Apache Kafka on virtualized/containerized/abstracted infrastructure; in this guide, the decision surface spanning VMs, Kubernetes operators, managed services, and serverless tiers (§1–§2).
- **Virtualization family** — one of the four deployment families of §2: self-managed VMs/bare metal, containerized + operator, managed provisioned service, serverless/elastic.
- **KRaft** — Kafka's Raft-based metadata mode (KIP-500): brokers and/or dedicated controllers run a Raft quorum owning the metadata log, replacing ZooKeeper; production-ready in Kafka 3.3 (KIP-833), sole mode in Kafka 4.0 (§4).
- **Controller quorum** — the set of KRaft controller nodes (dedicated `process.roles=controller` or combined `broker+controller`) that elect the active controller and own the metadata log (§4.2, §10.4).
- **ZooKeeper mode** — the pre-KRaft Kafka operating mode with an external Apache ZooKeeper ensemble for metadata and controller election; deprecated in 3.5, removed in 4.0 (§4).
- **Operator (Kubernetes)** — a control-loop component that extends the Kubernetes API with custom resources and reconciles desired state; Strimzi's Cluster/Topic/User Operators and CFK are the Kafka-family examples (§6–§7).
- **Custom resource (CR) / CRD** — a Kubernetes API extension; `Kafka`, `KafkaTopic`, `KafkaUser`, `KafkaNodePool` (Strimzi), `KafkaTopic`, `ConfluentRolebinding` (CFK) are the Kafka-management CRs (§6–§7).
- **StatefulSet** — the Kubernetes workload controller giving pods stable ordinal identity and per-pod PVCs — the substrate of broker identity (§10.1).
- **PVC (PersistentVolumeClaim)** — the Kubernetes request for storage; the unit in which broker and controller log directories are provisioned and persist across pod restarts (§9.3, §10.1).
- **Local Persistent Volume** — a PV bound to directly-attached node storage; GA in Kubernetes 1.14; the storage class Kafka brokers prefer for log directories (§9.2).
- **JBOD** — "just a bunch of disks"; Kafka's preferred multi-disk schema (KIP-18): multiple independent `log.dirs` per broker rather than RAID abstractions (§9.1).
- **Rack awareness** — Kafka's replica-placement rule (KIP-36) spreading replicas across failure domains via `broker.rack`; operators map it to Kubernetes AZs (§10.1).
- **Drain Cleaner** — the Strimzi tool that intercepts pod evictions during node drains and safely moves leadership/data off the departing broker (§6.1, §10.2).
- **Cruise Control** — LinkedIn's open-source Kafka rebalancing engine, integrated by Strimzi (and supported in Red Hat Streams) for workload-aware partition reassignment via `KafkaRebalance` (§6.3, §10.3).
- **Kafka Exporter** — the exporter (github.com/danielqsj/kafka_exporter) adding consumer-lag and broker/topic metrics to Prometheus (§11).
- **MirrorMaker 2** — Kafka's cross-cluster replication tool for DR, aggregation, and migration — the portability layer between virtualization families (§10.5, §14.3, §15.4).
- **Tiered storage** — offloading older log segments to object storage while keeping hot segments local; GA in Kafka 3.6 (KIP-405) (§9.4).
- **MSK (Amazon Managed Streaming for Apache Kafka)** — AWS's managed Apache Kafka service (preview re:Invent 2018; GA 30 May 2019), with provisioned and serverless cluster types (§13.2).
- **MSK Serverless** — AWS's automatic-capacity MSK tier (GA 28 April 2022) with throughput-based pricing (§13.2).
- **Kora** — Confluent's branded cloud-native Kafka engine underlying Confluent Cloud's elastic clusters (§13.1).
- **CCL (Confluent Community License)** — the source-available license applied to several Confluent components; relevant to CFK/Confluent Platform licensing (§7.2; the alternatives sibling's ledger has the full split).
- **GitOps** — managing cluster state declaratively from git; native to the CR-based operator model (§6.1, §11).
- **MAS / TRM** — the Monetary Authority of Singapore and its Technology Risk Management guidelines — the regulatory frame in Cymbal Bank's worked example (§15).

## Claims Audit and Verification Ledger

**✅ Verified this pass at primary sources (live web access):**

- **Kafka 4.0.0 released 18 March 2025, KRaft-only** — "the first major release to operate entirely without Apache ZooKeeper," KRaft mode by default (kafka.apache.org 4.0.0 release announcement, David Jacot, 18 March 2025). Same source: KIP-848 GA (new consumer-rebalance protocol), KIP-932 Queues early access (share groups), KIP-890 phase 2, KIP-966 ELR preview, KIP-996 Pre-Vote, brokers/Connect/tools on Java 17 and clients/Streams on Java 11.
- **KRaft ramp** — early access in Kafka 2.8 (KIP-500 umbrella, KAFKA-9119, cwiki); production-ready for new clusters in 3.3 (KIP-833, cwiki, incl. the deprecate-in-3.5 / remove-in-4.0 plan).
- **Strimzi CNCF** — accepted 28 August 2019; Incubating 8 February 2024 (cncf.io/projects/strimzi); first commit 21 March 2016 (CNCF project insights). Operators/CRs/features (Cluster/Topic/User Operators, Entity Operator, Drain Cleaner, `Kafka`/`KafkaTopic`/`KafkaUser`/node pools, listeners incl. OpenShift Routes, rack awareness, dedicated nodes via taints/tolerations, TLS/SCRAM/OAuth, ACLs, FIPS, Prometheus/Kafka Exporter/OpenTelemetry/Cruise Control) — strimzi.io home + operator docs. **Strimzi 0.45 = last minor with ZooKeeper-based clusters and MirrorMaker 1; KRaft from 0.46+; old CRD API versions supported until 1.0.0/0.52.0** — GitHub release notes. Latest stable observed: **1.2.0** ⚠ date-dependent.
- **CFK** — cloud-native control plane for Confluent Platform in your Kubernetes; next generation of Confluent Operator; deploys Kafka (KRaft or ZooKeeper), Connect, ksqlDB, Schema Registry, Control Center, REST Proxy; topics + rolebindings as CRs; same-broker-ID pod restore; AZs-as-racks rack awareness; JMX/Jolokia→Prometheus; does not manage Confluent Cloud (docs.confluent.io operator overview). **CFK 3.3.0** release notes (23 June 2026): CP 7.5.x–8.3.x, Kubernetes 1.28–1.36 / OpenShift 4.15–4.22; ZooKeeper→KRaft migration with preflight; dynamic-quorum controller scale-up; controller-quorum-loss DR procedure (docs.confluent.io).
- **AWS MSK** — open preview announced at re:Invent, November 2018 (AWS blog + TechCrunch 29 Nov 2018); **GA 30 May 2019** (AWS press release). **MSK Serverless** — preview at re:Invent 2021; **GA 28 April 2022**; throughput/partition-hour/storage pricing, IAM + PrivateLink, 99.9% SLA (aws.amazon.com what's-new, 28 April 2022).
- **Kubernetes Local Persistent Volumes GA in 1.14** (alpha 1.7, beta 1.10) — kubernetes.io blog.
- **KIP-18 JBOD** (JBOD preferred for Kafka; KAFKA-188 multiple log dirs) and **KIP-36 rack-aware replica assignment / `broker.rack`** — cwiki KIP pages.
- **Red Hat Streams for Apache Kafka (formerly AMQ Streams) based on Kafka + Strimzi** — developers.redhat.com; Cruise Control on JBOD in its docs — docs.redhat.com.
- **Aiven HQ Helsinki; CEO Oskari Saarenmaa** — aiven.io (about, leadership).
- **Confluent Cloud** — fully managed on AWS/GCP/Azure, cloud-native Kafka engine ("Kora"), Connect/Schema Registry/ksqlDB/Flink/Cluster Linking/stream governance — docs.confluent.io cloud overview.
- **Repo-verified (held from sibling ledgers):** Confluent founded 2014 by Kafka's creators; Kafka 3.6 tiered storage GA (2023); Burrow/lag-monitoring tooling and alert practice; Event Hubs Kafka endpoint 2018; the ZooKeeper-era migration path mechanics (banking guide §3.6); licensing split of Confluent components (alternatives sibling §1.2).

**⚠ Flagged (honest gaps):**

- Aiven **founding year 2016** — consistently reported by multiple independent secondary sources (incl. a 2021 CEO co-founder interview), but aiven.io's own pages do not state it; treat as secondary-verified, not primary-verified.
- **CFK GA/rename date** (Confluent Operator → Confluent for Kubernetes) — could not be pinned to a primary source this pass; docs show version history (1.6/1.7 → 2.x → 3.3.0) and the "next generation" statement only.
- **CFK/Confluent Platform licensing entitlement map** — commercial terms not verified against Confluent's licensing pages; procurement must confirm.
- Kafka 2.8 and 3.3 **release months** (April 2021 / October 2022) — held from knowledge base, not re-verified.
- "3-node dedicated controller quorum" guidance; page-cache-vs-memory-limit tuning; shared-file-storage avoidance; PVC-fill alert thresholds; minISR=2 with RF=3; local-PV-as-best-fit — operational practice/held knowledge, flagged rather than sourced.
- Strimzi 1.2.0 and CFK 3.3.0 "current" observations — **date-dependent** (September 2026 pass).
- Strimzi users-page adoption claims (logo wall observed ✅ on strimzi.io; implied production deployments unverified).
- Vendor volume-type specifics (EBS tiers, provisioned IOPS), MSK CloudWatch/limits details, MSK Replicator/Glue Schema Registry feature naming, Confluent Cloud tier names, Google Managed Kafka availability, IBM Event Streams operator, Aiven Karapace, MSK provisioned "fully managed" boundaries — not all re-verified; flagged.
- "Fully managed"/"elastic" marketing claims and Kora-internals claims — marketing calibration required (§13.5).
- §14 TCO figures and §15 roadmap dates/weights — explicitly illustrative.

**❌ Debunked/contested:**

- **"AWS MSK GA'd in late 2018 (November)."** ❌ November 2018 (re:Invent) was the **open-preview** announcement; MSK reached GA on **30 May 2019** (AWS press release). Corrected in §13.2.
- **"KRaft early access began with Kafka 3.3."** ❌ 3.3 marked KRaft **production-ready** (KIP-833); the **early access** shipped in Kafka 2.8 (KIP-500). The two-step ramp is §4.1's timeline.
- **"Managed Kafka means no operations."** ❌ rejected as a category claim — §13.5 and §14: managed relocates operations (vendor management, cost governance, quota/residency) rather than removing them.

**Integrity statement:** nothing in this guide is fabricated. Every ✅ was checked at the cited primary source this pass or is held from a sibling ledger; every ⚠ is flagged inline and listed here; the ❌ items are recorded with corrections. Where the brief demanded verification and a primary source was not reachable (Aiven's own pages on founding; CFK GA date), the guide says so rather than asserting.

### What Could Not Be Verified

- **Aiven's founding year from aiven.io itself.** The about/leadership pages confirm Helsinki HQ and the CEO but never state a founding date; the 2016 year rests on secondary sources (flagged ⚠ in §13.3). If a primary Aiven page or press release stating the founding year is found later, upgrade the flag.
- **The exact date Confluent for Kubernetes became generally available / was renamed from Confluent Operator.** docs.confluent.io states the lineage and shows the version history but no date; web search returned nothing authoritative. The guide therefore asserts only what the docs assert.
- **Confluent's precise CFK/Confluent Platform license-tier map.** Commercial terms change and were not extracted; §7.2 points to the alternatives sibling's ledger for the component licensing split and requires procurement confirmation for entitlements.
- **Specific current limits, prices, and SLA terms for the managed services** (MSK provisioned/serverless limits, Confluent Cloud tier definitions, Aiven node pricing, per-region availability lists). These are published, date- and region-dependent numbers; the guide records verified *launch* facts and flags the rest.
- **Precise Kafka 2.8 and 3.3 release dates** (months verified only from knowledge base, not from the Apache release archives this pass).
- **Whether MSK provisioned, Confluent Cloud, and Aiven run KRaft-mode brokers internally and since which version.** All three run modern Kafka; vendor-internal metadata-mode details are not publicly documented to this guide's standard.
- **Kafka 4.x partition-count ceiling numbers and tiered-storage parity across vendors** — hardware- and version-dependent; flagged rather than asserted.

## Cross-References Recap

| Guide | Where used |
|---|---|
| [Apache Kafka: the Distributed Event-Streaming Platform](../banking/kafka_guide.md) | §1, §3 (architecture recap/depth), §4.2 (KRaft migration path), §10.3, §10.5, §12, §15.1 (worked-example taxonomy) |
| [kafka_alternatives_guide.md](kafka_alternatives_guide.md) | §1–§2 (when-not-Kafka), §7.2 (licensing ledger), §9.4, §13 (managed field), §14 (cost/portability) |
| [event_stream_processing_guide.md](event_stream_processing_guide.md) | header companions (the discipline) |
| [complex_event_processing_guide.md](complex_event_processing_guide.md) | header companions (CEP patterns) |
| [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) | §10.5, §11 (lag monitoring, reliability) |
| [secure_red_hat_openshift_guide.md](secure_red_hat_openshift_guide.md) | §1, §8, §10, §12, §15 (platform security, SCCs, FIPS, audit) |
| [openshift_workload_availability_guide.md](openshift_workload_availability_guide.md) | §8, §10.2, §11 (availability mechanics, PDBs, drains) |
| [charmed_kubernetes_vs_openshift_guide.md](charmed_kubernetes_vs_openshift_guide.md) | §8 (Kubernetes distributions) |
| [devstack_openstack_guide.md](devstack_openstack_guide.md) | §8, §9.2 (OpenStack-hosted platforms, Cinder storage) |
| [container_certificates_guide.md](container_certificates_guide.md) | §12 (TLS/certs in containers) |
| [enterprise_middleware_integration_platform_guide.md](enterprise_middleware_integration_platform_guide.md) | §1, §5 (enterprise middleware estate) |
| [ai_platform_engineering_guide.md](ai_platform_engineering_guide.md) | §12, §15.4 (platform engineering operating model) |
| [architecture_decision_record_guide.md](architecture_decision_record_guide.md) | §15.4 (ADR record-keeping discipline) |

## References and Further Reading

- **Apache Kafka 4.0.0 Release Announcement** — kafka.apache.org, David Jacot, 18 March 2025 (KRaft-only, KIP-848/890/932/966/996, Java baselines).
- **Apache Kafka CWiki** — KIP-500 (Replace ZooKeeper with a Metadata Quorum; KAFKA-9119 umbrella), KIP-833 (Mark KRaft as Production Ready), KIP-18 (JBOD Support), KIP-36 (Rack aware replica assignment).
- **CNCF** — cncf.io/projects/strimzi (acceptance 28 Aug 2019; Incubating 8 Feb 2024); cncf.io blog "Strimzi joins the CNCF Incubator" (8 Feb 2024).
- **Strimzi** — strimzi.io (project home); strimzi.io/docs/operators/latest/overview (operators, CRDs, security, monitoring); strimzi.io/downloads (1.2.0); github.com/strimzi/strimzi-kafka-operator releases (0.45 ZooKeeper cutover note).
- **Confluent** — docs.confluent.io/operator/current/overview.html (CFK overview); docs.confluent.io/operator/current/release-notes.html (3.3.0, 23 June 2026); docs.confluent.io/operator/current/co-configure-rack-awareness.html; docs.confluent.io/cloud/current/overview.html (Confluent Cloud, Kora).
- **AWS** — aws.amazon.com/blogs/aws (MSK preview at re:Invent 2018; MSK Serverless GA); press.aboutamazon.com (MSK GA, 30 May 2019); aws.amazon.com/about-aws/whats-new/2022/04/ (MSK Serverless GA, 28 April 2022).
- **Kubernetes** — kubernetes.io/blog/2019/04/04/kubernetes-1.14-local-persistent-volumes-ga/ (Local PV GA).
- **Red Hat** — developers.redhat.com/products/streams-for-apache-kafka; docs.redhat.com (Streams for Apache Kafka on OpenShift; Cruise Control on JBOD).
- **Aiven** — aiven.io/about (HQ Helsinki; leadership).
- **TechCrunch** — "AWS launches a managed Kafka service," 29 November 2018 (preview coverage).

---

*End of guide. Kafka's statefulness used to be the argument for leaving it on VMs and never touching it; the KRaft era and the operator pattern inverted that argument — the metadata system is now part of the system you already run, the stateful care is encoded as declarative resources under GitOps, and the protocol stays the same whether the brokers sit on your OpenShift nodes or in a vendor's region.* Choose the family by who must hold the evidence and who must hold the pager, mirror between families with the tools Kafka gave you, and let the data flow on, through, and across the virtualized stream.
