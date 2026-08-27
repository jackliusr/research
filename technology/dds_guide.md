# DDS: The Data-Centric Bus — A Comprehensive Guide

**The Data-Centric Bus — the OMG Data Distribution Service (DDS): the Overview (the December 2004 Adoption, the DCPS Model, the DLRL Demise), the History (DDS 1.0 → 1.4, the RTPS Wire Protocol, XTypes, DDS-Security), the Core Concepts (Domains, Domain Participants, Topics, Instances, Keys, Data Writers and Data Readers, Partitions), the Data Model (OMG IDL, the XTypes Extensible and Dynamic Topic Types), the RTPS Wire Protocol and Discovery (SPDP, SEDP, the Built-in Topics), the QoS Policies (Reliability, Durability, History, Deadline, Latency Budget, Liveliness, Ownership), the DDS Security Model (the PKI-DH Authentication Plugin, the Access Control Plugin, the AES-GCM Cryptographic Plugin), the Implementations (RTI Connext, Eclipse Cyclone DDS, eProsima Fast DDS, OpenDDS, GurumDDS), the Ecosystem and Use Cases (ROS 2, Autonomous Vehicles, Defense, Industrial IoT vs OPC UA, Aerospace, Medical, Financial Market Data), the Head-to-Head Comparison (MQTT, AMQP, Apache Kafka, Eclipse Zenoh), the Cymbal Bank Market-Data Worked Example, the Claims Audit, and the Glossary — from the December 2004 OMG Adoption to the Brokerless Trading-Floor Data Bus**

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Technology / Middleware — the DDS deep-dive of the messaging-and-streaming cluster: what the OMG Data Distribution Service actually is (a data-centric, brokerless publish-subscribe middleware standard for real-time systems), the DCPS model and the DLRL, the RTPS wire protocol and its discovery machinery (SPDP/SEDP), the QoS policy system that makes DDS *predictable* rather than merely fast, the DDS-Security plugin architecture (PKI-DH authentication, access control, AES-GCM cryptography), the implementation landscape (RTI Connext, Eclipse Cyclone DDS, eProsima Fast DDS, OpenDDS, GurumDDS), the ecosystem (ROS 2's default middleware, autonomous vehicles, defense, industrial IoT, aerospace, medical, and low-latency financial market data), and the head-to-head against MQTT, AMQP, Apache Kafka and Eclipse Zenoh. The guide cross-references the OT/IIoT angle to the sibling [SCADA, ICS and OT Security](scada_guide.md) guide (the DDS-vs-OPC-UA discussion lives there) and the financial angle to the [FIX Protocol](../banking/fix_protocol_guide.md) and [Apache Kafka](../banking/kafka_guide.md) guides — it does **not** re-derive FIX, Kafka or OPC UA. The worked example is a Cymbal Bank trading-floor market-data distribution architecture: domain design, QoS profile, DDS-Security deployment, and the integration boundary with the FIX execution layer.
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** the OMG specification catalogue, verified live on omg.org this pass — **DDS 1.4** (formal, publication/adoption March 2015, OMG file id formal/15-04-10; version history: 1.0 December 2004, 1.1 December 2005, 1.2 December 2006, 1.4 March 2015 — https://www.omg.org/spec/DDS/1.4), **DDSI-RTPS 2.5** ("DDS Interoperability Wire Protocol" — Real-Time Publish-Subscribe Protocol, formal April 2022, formal/22-04-01; history 2.0 April 2008 → 2.5 April 2022 — https://www.omg.org/spec/DDSI-RTPS/2.5), **DDS-SECURITY 1.2** (formal **February 2026**, formal/25-03-06; history 1.0 August 2016, 1.1 July 2018, 1.2 February 2026 — https://www.omg.org/spec/DDS-SECURITY/1.2), **DDS-XTypes 1.3** ("Extensible and Dynamic Topic Types for DDS", formal February 2020, formal/20-02-04; history 1.0 October 2012 → 1.3 February 2020 — https://www.omg.org/spec/DDS-XTypes/1.3). Vendor/ecosystem facts verified against rti.com, cyclonedds.io / eclipse.dev, eprosima.com, opendds.org, gurumdds.com, docs.ros.org, docs.oasis-open.org, kafka.apache.org and zenoh.io — see §11. Everything verified this pass is marked ✅; anything not confirmed is flagged ⚠ honestly. No dates, version numbers, or policy names were invented.
> **Last Updated:** August 2026
> **Companion guides (sibling, same folder — plain filenames):** [SCADA, ICS and OT Security](scada_guide.md) (**the OT/IIoT companion — the DDS-vs-OPC-UA industrial discussion cross-refs §8.4 here; OPC UA/IEC 62541 itself is covered there, not re-derived**), [Kafka Alternatives](kafka_alternatives_guide.md) (the message-bus alternatives map — cross-ref §9), [Event Stream Processing](event_stream_processing_guide.md) (the stream-processing paradigm next door to the pub-sub bus — cross-ref §9), [Complex Event Processing](complex_event_processing_guide.md) (the CEP layer that typically sits *on top of* a DDS bus — cross-ref §8.7 and §10), [Zero Downtime System Design](zero_downtime_system_design_guide.md) (the availability discipline for the trading-floor estate — cross-ref §10)
> **Companion guides (banking/, prefix `../banking/`):** [FIX Protocol](../banking/fix_protocol_guide.md) (**the execution-layer companion — the integration boundary in §10.5 — cross-ref, do not re-derive**), [Apache Kafka Guide](../banking/kafka_guide.md) (**the streaming-platform companion — cross-ref §7.4 and §9**)

---

**How to use this guide:** Section 1 is the overview — the short answer, the key-facts table, the DCPS model and the DLRL story. Section 2 is the history — the December 2004 adoption, the version timeline, and the companion specifications (RTPS, XTypes, DDS-Security). Section 3 is the core concepts — domains, participants, topics, instances, keys, writers/readers, partitions, and the data model (OMG IDL, XTypes). Section 4 is the RTPS wire protocol and discovery — SPDP, SEDP and the built-in topics. Section 5 is the QoS policies — the eight headline policies and the rest of the catalogue. Section 6 is the DDS security model — the plugin architecture and the three built-in plugins. Section 7 is the implementations — RTI Connext, Eclipse Cyclone DDS, eProsima Fast DDS, OpenDDS and GurumDDS, with licenses and origins. Section 8 is the ecosystem and use cases — ROS 2, autonomous vehicles, defense, industrial IoT (cross-ref'd to the SCADA guide), aerospace, medical, and financial market data (cross-ref'd to the FIX and Kafka guides). Section 9 is the head-to-head comparison — DDS vs MQTT, AMQP, Apache Kafka and Eclipse Zenoh, with the comparison table. Section 10 is the Cymbal Bank worked example — a trading-floor market-data distribution architecture: domain design, QoS profile, DDS-Security deployment, and the FIX integration boundary. Section 11 is the claims audit (✅/⚠/❌). Section 12 is "What Could Not Be Verified". Section 13 is the glossary. Section 14 is cross-references and further reading. Cross-references follow the repository convention: sibling guides in `technology/` are plain filenames; guides in `banking/` are prefixed `../banking/`. **Integrity convention:** ✅ = verified this pass against a primary or cited source (URL given in §11); ⚠ = flagged/unverified/approximate; ⚠-knowledge = well-documented industry knowledge not re-verified live. No spec dates, version numbers, or policy names were invented — anything that could not be confirmed is flagged ⚠ honestly.

---

## Table of Contents

1. [The Overview](#1-the-overview)
   - 1.1 [The Short Answer](#11-the-short-answer)
   - 1.2 [The Key-Facts Table](#12-the-key-facts-table)
   - 1.3 [The Definition — Data-Centric Publish-Subscribe](#13-the-definition--data-centric-publish-subscribe)
   - 1.4 [The DCPS Model — and the DLRL](#14-the-dcps-model--and-the-dlrl)
   - 1.5 [The Brokerless, Peer-to-Peer Architecture](#15-the-brokerless-peer-to-peer-architecture)
   - 1.6 [What DDS Is — and Is Not](#16-what-dds-is--and-is-not)
2. [The History — 2004 to Today](#2-the-history--2004-to-today)
   - 2.1 [The December 2004 Adoption](#21-the-december-2004-adoption)
   - 2.2 [The Version Timeline](#22-the-version-timeline)
   - 2.3 [The Companion Specifications — RTPS, XTypes, DDS-Security](#23-the-companion-specifications--rtps-xtypes-dds-security)
   - 2.4 [The DLRL Story — Deprecated, Then Removed](#24-the-dlrl-story--deprecated-then-removed)
   - 2.5 [The Roadmap Signals](#25-the-roadmap-signals)
3. [The Core Concepts](#3-the-core-concepts)
   - 3.1 [The Domain and the Domain Participant](#31-the-domain-and-the-domain-participant)
   - 3.2 [The Topic, the Instance and the Key](#32-the-topic-the-instance-and-the-key)
   - 3.3 [The Data Writer and the Data Reader](#33-the-data-writer-and-the-data-reader)
   - 3.4 [The Partition](#34-the-partition)
   - 3.5 [The Data Model — OMG IDL and XTypes](#35-the-data-model--omg-idl-and-xtypes)
4. [The RTPS Wire Protocol and Discovery](#4-the-rtps-wire-protocol-and-discovery)
   - 4.1 [RTPS — the Wire Protocol](#41-rtps--the-wire-protocol)
   - 4.2 [SPDP — Simple Participant Discovery Protocol](#42-spdp--simple-participant-discovery-protocol)
   - 4.3 [SEDP — Simple Endpoint Discovery Protocol](#43-sedp--simple-endpoint-discovery-protocol)
   - 4.4 [The Built-in Topics](#44-the-built-in-topics)
5. [The QoS Policies](#5-the-qos-policies)
   - 5.1 [The QoS Model](#51-the-qos-model)
   - 5.2 [Reliability — BEST_EFFORT vs RELIABLE](#52-reliability--best_effort-vs-reliable)
   - 5.3 [Durability — VOLATILE, TRANSIENT_LOCAL, TRANSIENT, PERSISTENT](#53-durability--volatile-transient_local-transient-persistent)
   - 5.4 [History — KEEP_LAST vs KEEP_ALL](#54-history--keep_last-vs-keep_all)
   - 5.5 [Deadline and Latency Budget](#55-deadline-and-latency-budget)
   - 5.6 [Liveliness](#56-liveliness)
   - 5.7 [Ownership](#57-ownership)
   - 5.8 [The Rest of the Catalogue](#58-the-rest-of-the-catalogue)
6. [The DDS Security Model](#6-the-dds-security-model)
   - 6.1 [The Specification and the Plugin Architecture](#61-the-specification-and-the-plugin-architecture)
   - 6.2 [Authentication — DDS:Auth:PKI-DH](#62-authentication--ddsauthpki-dh)
   - 6.3 [Access Control — DDS:Access:Permissions](#63-access-control--ddsaccesspermissions)
   - 6.4 [Cryptographic — DDS:Crypto:AES-GCM](#64-cryptographic--ddscryptoaes-gcm)
   - 6.5 [Protecting Discovery](#65-protecting-discovery)
7. [The Implementations](#7-the-implementations)
   - 7.1 [The Landscape Table](#71-the-landscape-table)
   - 7.2 [RTI Connext — Real-Time Innovations](#72-rti-connext--real-time-innovations)
   - 7.3 [Eclipse Cyclone DDS — ZettaScale](#73-eclipse-cyclone-dds--zettascale)
   - 7.4 [eProsima Fast DDS](#74-eprosima-fast-dds)
   - 7.5 [OpenDDS — Object Computing Inc](#75-opendds--object-computing-inc)
   - 7.6 [GurumDDS — GurumNetworks](#76-gurumdds--gurumnetworks)
   - 7.7 [The Market-Share Caveat](#77-the-market-share-caveat)
8. [The Ecosystem and Use Cases](#8-the-ecosystem-and-use-cases)
   - 8.1 [ROS 2 — the Default Middleware](#81-ros-2--the-default-middleware)
   - 8.2 [Autonomous Vehicles](#82-autonomous-vehicles)
   - 8.3 [Defense](#83-defense)
   - 8.4 [Industrial IoT — DDS vs OPC UA](#84-industrial-iot--dds-vs-opc-ua)
   - 8.5 [Aerospace](#85-aerospace)
   - 8.6 [Medical](#86-medical)
   - 8.7 [Financial Market Data](#87-financial-market-data)
9. [The Head-to-Head Comparison](#9-the-head-to-head-comparison)
   - 9.1 [The Comparison Table](#91-the-comparison-table)
   - 9.2 [DDS vs MQTT](#92-dds-vs-mqtt)
   - 9.3 [DDS vs AMQP](#93-dds-vs-amqp)
   - 9.4 [DDS vs Apache Kafka](#94-dds-vs-apache-kafka)
   - 9.5 [DDS vs Eclipse Zenoh](#95-dds-vs-eclipse-zenoh)
10. [The Cymbal Bank Worked Example — Market Data on the Trading Floor](#10-the-cymbal-bank-worked-example--market-data-on-the-trading-floor)
    - 10.1 [The Problem](#101-the-problem)
    - 10.2 [The Domain Design](#102-the-domain-design)
    - 10.3 [The QoS Profile](#103-the-qos-profile)
    - 10.4 [The DDS-Security Deployment](#104-the-dds-security-deployment)
    - 10.5 [The Integration Boundary — DDS to FIX](#105-the-integration-boundary--dds-to-fix)
11. [The Claims Audit — Verified, Flagged, Rejected](#11-the-claims-audit--verified-flagged-rejected)
12. [What Could Not Be Verified](#12-what-could-not-be-verified)
13. [The Glossary](#13-the-glossary)
14. [Cross-References and Further Reading](#14-cross-references-and-further-reading)

---

## 1. The Overview

### 1.1 The Short Answer

DDS — the **Data Distribution Service** — is a middleware standard published by the Object Management Group (OMG) for **real-time, data-centric publish-subscribe** communication between distributed systems. It was first adopted as an OMG formal specification in **December 2004** (DDS 1.0) ✅, and the current formal revision is **DDS 1.4, published March 2015** ✅.

Where message queues (JMS, AMQP, Kafka) route messages *through* a broker, DDS is **brokerless**: every application node runs a DDS implementation that talks **directly, peer-to-peer** with every other node that shares its *domain*. Producers (DataWriters) and consumers (DataReaders) rendezvous on named **Topics** with typed data models, and the middleware itself — not the application — takes care of discovery, matching, reliability, delivery timing, and fault detection, all driven by an unusually rich set of **Quality of Service (QoS) policies** that let each data stream specify its own contract: reliable or best-effort, durable or volatile, how fast updates must arrive, and what happens when a peer dies.

Because there is no broker to become a bottleneck or a single point of failure, DDS is the standard of choice for hard real-time, embedded, and safety-critical systems: autonomous vehicles, defense platforms, industrial control, aerospace, medical devices — and, increasingly, low-latency financial market data distribution. It is also, since the ROS 2 transition, the default middleware of the Robot Operating System.

The OMG frames the purpose of the standard as: *"Efficient and Robust Delivery of the Right Information to the Right Place at the Right Time"* ✅ (omg.org, DDS 1.4 specification page).

### 1.2 The Key-Facts Table

| Aspect | Fact | Status |
| --- | --- | --- |
| Standard body | Object Management Group (OMG) | ✅ |
| First formal version | DDS 1.0, December 2004 | ✅ omg.org |
| Current formal version | DDS 1.4, March 2015 (formal/15-04-10) | ✅ omg.org |
| Wire protocol | DDSI-RTPS 2.5 — "DDS Interoperability Wire Protocol" / Real-Time Publish-Subscribe Protocol, April 2022 (formal/22-04-01) | ✅ omg.org |
| Type system | DDS-XTypes 1.3 — "Extensible and Dynamic Topic Types for DDS", February 2020 (formal/20-02-04) | ✅ omg.org |
| Security | DDS-SECURITY 1.2, February 2026 (formal/25-03-06) | ✅ omg.org |
| Model | Data-Centric Publish-Subscribe (DCPS): API + communication semantics | ✅ omg.org |
| Architectural pattern | Brokerless, peer-to-peer, decentralized discovery | ✅-knowledge |
| Typical transports | UDP/IP (unicast + multicast), TCP/IP, shared memory, vendor-specific | ⚠-knowledge |
| Typical latency | Microseconds to low milliseconds on LAN; vendor benchmarks vary | ⚠-knowledge |
| Reference implementations | RTI Connext, Eclipse Cyclone DDS, eProsima Fast DDS, OpenDDS, GurumDDS (see §7) | ✅ §7 |
| Notable deployments | ROS 2 default middleware; autonomous vehicles; defense; market data | ✅/⚠ §8 |

### 1.3 The Definition — Data-Centric Publish-Subscribe

The OMG DDS specification page describes the standard as follows (✅ verified on omg.org):

> "The DDS specification describes a Data-Centric Publish-Subscribe (DCPS) model for distributed application communication and integration. This specification defines both the Application Interfaces (APIs) and the Communication Semantics (behavior and quality of service) that enable the efficient delivery of information from information producers to matching consumers."

Three words in that sentence carry the whole architecture:

- **Data-centric.** The system is organized around *data*, not around messages or requests. Applications publish *samples of typed data* (e.g. `MarketDataTick`, `OrderState`, `VehiclePose`) onto named Topics. Readers subscribe to Topics by name and type. The middleware treats each topic as a *state* that evolves over time (with instances identified by keys), which is what enables the durability and liveliness machinery — the bus remembers, replays, and notices when data stops flowing.
- **Publish-subscribe.** Producers and consumers are fully decoupled in space, time, and flow: a writer does not know the readers' addresses (discovery finds them), readers can join and leave at any time, and the QoS policies govern the flow between them.
- **Semantics, not just API.** The spec standardizes *behavior* — the QoS policies, the discovery contracts, the wire protocol — so that implementations from different vendors interoperate, not merely that they share an API shape.

### 1.4 The DCPS Model — and the DLRL

The DDS standard is organized in two layers:

- **DCPS — Data-Centric Publish-Subscribe.** The core model: the API (DomainParticipant, Topic, DataWriter, DataReader, Publisher, Subscriber) and the communication semantics (QoS policies, listener callbacks, conditions/wait-sets). This is the layer every implementation ships, and the layer everything in this guide is about.
- **DLRL — Data Local Reconstruction Layer.** An optional, higher-level object-oriented layer that sat on top of DCPS and presented topic data as a *local, object-oriented cache* of the global data space (so applications could navigate related objects with references instead of reading raw samples). The DLRL was **deprecated in the DDS 1.2 revision (December 2006)** and is not part of the DDS 1.4 core; the DDS 1.4 specification page still lists `dds_dlrl.idl` among the normative machine-readable documents (✅ verified — the IDL is still distributed with the 1.4 package), but no mainstream implementation ships the DLRL layer today, and the OMG's own 1.4 specification text no longer carries it as an active clause ⚠ (see §2.4 for the full record).

In practice the industry treats "DDS" as "DCPS"; the DLRL is a historical footnote that matters mostly for reading older literature.

### 1.5 The Brokerless, Peer-to-Peer Architecture

The single most distinctive architectural fact about DDS: **there is no broker.** In JMS/AMQP/Kafka-style systems, producers and consumers connect to one or more central brokers that route, buffer, and fan out messages; the broker is the availability, latency, and capacity bottleneck, and it must be clustered and failed over. In DDS:

- Every participant runs the middleware locally and discovers its peers **directly** over the network (see §4 — SPDP/SEDP over RTPS).
- Data flows **directly from writer to reader** — on a LAN, typically over UDP with multicast fan-out (one packet, many readers), which is precisely the pattern low-latency market data wants.
- There is **no single point of failure**: the loss of any node only affects the data that node produced; readers of a topic with multiple writers keep receiving from the survivors.
- Discovery is **decentralized** (each participant announces itself and matches endpoints), though implementations may add optional central "discovery servers" for large or WAN-spanning deployments (⚠-knowledge — vendor-specific).

The trade-off: DDS assumes a network where participants can reach each other (LAN, multicast-capable, or a configured peer list), and it is *not* designed around the store-and-forward, durable-queue, exactly-once-replay semantics that brokered systems provide — that lane belongs to Kafka (§7.4, §9.4).

### 1.6 What DDS Is — and Is Not

- **Is:** a real-time data bus for typed, QoS-governed publish-subscribe between peers; a data-centric integration layer for distributed systems; the standard middleware for robot/vehicle/defense/industrial platforms; a credible low-latency market-data distribution layer.
- **Is not:** a message queue (no broker, no named queues, no store-and-forward to offline consumers — durability here means *late-joining* readers, see §5.3); a request/reply RPC framework (though request/reply can be built on it via patterns); a stream-processing or log platform (Kafka's lane — see the [Apache Kafka Guide](../banking/kafka_guide.md)); a REST/HTTP API (see the [Kafka Alternatives](kafka_alternatives_guide.md) guide for where each pattern fits).

---

## 2. The History — 2004 to Today

### 2.1 The December 2004 Adoption

DDS emerged from the real-time systems community's frustration with CORBA-style request/reply and with proprietary pub-sub middleware in defense and industrial systems. The OMG's formal version history records the first formal version as **DDS 1.0, adopted December 2004** ✅ (omg.org spec history table). The original specification work was led by a consortium that included Real-Time Innovations (RTI), Thales, and Objective Interface Systems — the DDS 1.4 page lists RTI, Thales and Objective Interface Systems among the copyright contributors (✅ verified, omg.org).

DDS 1.0 established the DCPS model and the QoS policy catalogue. It did **not** yet standardize a wire protocol — interoperability between vendors was a stated goal but not yet a reality; each implementation had its own protocol.

### 2.2 The Version Timeline

The formal-version history on omg.org (✅ verified this pass):

| Version | Adoption Date | Notes |
| --- | --- | --- |
| DDS 1.0 | December 2004 | First formal version; DCPS model, QoS catalogue |
| DDS 1.1 | December 2005 | Maintenance revision |
| DDS 1.2 | December 2006 | Maintenance revision; DLRL deprecated |
| (1.3 never published formally) | — | ⚠ no formal 1.3 appears in the omg.org history |
| DDS 1.4 | March 2015 | Current formal revision (formal/15-04-10) |

The five-year gap between 1.2 (2006) and 1.4 (2015) is real: the effort in those years went into the companion specifications (RTPS 2.x, XTypes, DDS-Security) rather than the core spec. The OMG issue tracker referenced from the DDS page ("DDS15") suggests a future revision is being worked (⚠ — not yet formal as of this pass).

### 2.3 The Companion Specifications — RTPS, XTypes, DDS-Security

The DDS "family" of OMG specifications, all verified on omg.org this pass:

- **DDSI-RTPS** — *DDS Interoperability Wire Protocol*, also titled *Real-Time Publish-Subscribe Protocol*. The wire protocol that makes vendor interoperability possible (first formal version 2.0, April 2008; current **2.5, April 2022**). Covered in §4.
- **DDS-XTypes** — *Extensible and Dynamic Topic Types for DDS*. The type system and type-evolution rules (first formal version 1.0, October 2012; current **1.3, February 2020**). Covered in §3.5.
- **DDS-SECURITY** — *DDS Security*. The security model and Service Plugin Interface (SPI) architecture with three built-in plugins (first formal version 1.0, August 2016; **1.1, July 2018; current 1.2, February 2026**). Covered in §6.

### 2.4 The DLRL Story — Deprecated, Then Removed

The DLRL (Data Local Reconstruction Layer) was part of the original DDS vision: an optional object-oriented layer that reconstructed a local, navigable object model of the distributed data space on top of DCPS. Its trajectory (⚠ where not directly re-verified this pass):

- **DDS 1.0 (2004):** DLRL specified as the second layer of the standard, alongside DCPS.
- **DDS 1.2 (December 2006):** DLRL **deprecated** — the OMG marked the layer as deprecated in the 1.2 revision ⚠-knowledge (widely documented; not re-fetched from the 1.2 PDF this pass).
- **DDS 1.4 (March 2015):** the DLRL is no longer an active part of the specification text ⚠; the omg.org 1.4 document list still ships the legacy `dds_dlrl.idl` (file id ptc/06-04-12) as a normative machine-readable artifact ✅ (verified from the omg.org DDS 1.4 page), which is a historical carry-over rather than a live mandate.
- **Today:** no mainstream implementation (RTI Connext, Cyclone DDS, Fast DDS, OpenDDS, GurumDDS) ships a DLRL layer; the industry consensus is that DCPS's data-centric model made the DLRL redundant ⚠-knowledge.

Net: the DLRL is dead as a product layer but still occasionally surfaces in old documentation and in the 1.4 IDL bundle.

### 2.5 The Roadmap Signals

Two signals from omg.org suggest the family is still moving (⚠ — both are *signals*, not formal versions): the DDS page's issue tracker references "DDS15" (a future core revision) and the DDSI-RTPS page references "DDSIRTP26" (a future wire-protocol revision). Neither had a formal specification page as of this pass — this guide therefore treats **DDS 1.4 / RTPS 2.5 / XTypes 1.3 / DDS-SECURITY 1.2** as the current formal versions.

---

## 3. The Core Concepts

### 3.1 The Domain and the Domain Participant

- **The Domain.** A DDS *domain* is a virtual communication boundary identified by a numeric **DomainId**. Applications (and even entire estates) can share one network while remaining fully isolated from each other by using different domain ids — a trading floor's market-data domain and a bank's building-management domain can coexist on the same LAN without ever seeing each other's traffic. Two entities can communicate only if they belong to the same domain. ⚠-knowledge (core DDS semantics; the domain concept is defined throughout the DDS 1.4 spec but not re-quoted here).
- **The DomainParticipant.** The *DomainParticipant* is the entry point into a domain: the container object from which every other entity — Topics, Publishers, Subscribers, DataWriters, DataReaders — is created. Each application process typically creates one participant per domain it needs. The participant also carries participant-level QoS (e.g. USER_DATA for tagging) and, in the DDS-Security model, the participant identity is the *principal* that gets authenticated (§6).

The entity hierarchy is a containment tree: DomainParticipant → Publisher → DataWriter, and DomainParticipant → Subscriber → DataReader, with Topics attached to the participant. The DDS 1.4 specification's PIM description (§2.2.2 of the spec — Infrastructure, Topic-Definition, Publication and Subscription modules) is exactly this entity model, and its structure is verified from the spec's table of contents ✅ (DDS 1.4 PDF).

### 3.2 The Topic, the Instance and the Key

- **The Topic.** A *Topic* is the rendezvous point of the data-centric model: a named, typed data stream. It is identified by a unique name within the domain and bound to a data *type* (defined in OMG IDL, §3.5) plus a set of topic-level QoS policies. A DataWriter publishes samples *on a topic*; a DataReader subscribes *to a topic*; both must agree on name and type (with XTypes type-compatibility rules softening exact-match requirements).
- **The Instance.** Because DDS is data-centric, a topic is not a firehose of anonymous messages: it is a *collection of state objects*, each called an **instance**. The type's **key fields** (marked `@key` in IDL) determine the instance identity. Every distinct combination of key values is one instance, and the middleware tracks each instance's lifecycle: **ALIVE**, **NOT_ALIVE_DISPOSED** (a writer explicitly disposed it), and **NOT_ALIVE_NO_WRITERS** (all writers for it went away). The DDS 1.4 spec text confirms the relationship between OWNERSHIP and instance state — "If OWNERSHIP is set to EXCLUSIVE, then the instance_state becomes NOT_ALIVE_DISPOSED only if the [owning writer disposes it]" ✅ (DDS 1.4 PDF).
- **The Sample.** A *sample* is one data value written for an instance at a point in time. The reader-side API exposes, per sample, a **SampleInfo** carrying the sample state (READ / NOT_READ), the view state (NEW / NOT_NEW) and the instance state. This is the machinery behind "the bus remembers" — durability, history and liveliness are all defined per instance.

A worked miniature: a topic `MarketDataTick` with type `struct MarketDataTick { string symbol; @key ...; double price; ... }` keyed on `symbol` — then "SGD=X", "AAPL.O" and "US10Y" are three instances, each with its own last-value, its own deadline accounting, and its own ownership.

### 3.3 The Data Writer and the Data Reader

- **The DataWriter** (created by a Publisher on a Topic) is the producer endpoint. The application calls `write(sample)`; the middleware takes care of serialization, batching, transport selection, and the QoS contract. Writers *offer* QoS; they can be listened to (the spec's communication-status model — e.g. `OFFERED_DEADLINE_MISSED`, `LIVELINESS_LOST` — is verified in the DDS 1.4 PDF ✅).
- **The DataReader** (created by a Subscriber on a Topic) is the consumer endpoint. The application either polls with `read()`/`take()` (which detach samples and their SampleInfo) or is notified via **Listeners**, **Conditions** and **Wait-sets** — the DDS 1.4 spec dedicates §2.2.4 to "Listeners, Conditions, and Wait-sets" and §2.2.4.1 to "Communication Status" ✅ (verified from the spec TOC).
- **Matching.** A writer and a reader are *matched* when: same domain, compatible topic (name + type under XTypes rules), compatible partitions (§3.4), and **QoS-compatible** — the reader's *requested* QoS must be satisfiable by the writer's *offered* QoS (see §5.1). Matching happens automatically via discovery (§4); neither side addresses the other.

### 3.4 The Partition

The **PARTITION** QoS policy (one of the 22 policies in the verified DDS 1.4 catalogue, §2.2.3.13 ✅) subdivides a domain without creating new domain ids. A partition is a string (or list of strings, with wildcard matching); a writer publishing into partition `P` is only matched to readers subscribed to partitions that intersect `P`. Partitions are the standard tool for logical segregation inside one domain — per asset class, per book, per environment (prod/test) — and are used heavily in the Cymbal Bank worked example (§10.2). ⚠-knowledge (policy semantics; the policy's presence in the catalogue is verified).

### 3.5 The Data Model — OMG IDL and XTypes

- **OMG IDL.** DDS topic types are defined in the OMG Interface Definition Language (IDL). The ROS 2 design article states it directly: "DDS uses the Interface Description Language (IDL) as defined by the Object Management Group (OMG) for message definition and serialization" ✅ (design.ros2.org, "ROS on DDS").
- **XTypes — Extensible and Dynamic Topic Types for DDS** (formal 1.3, February 2020, ✅ omg.org). XTypes is the type-system companion to DDS: it formally defines the type model (in UML), the **TypeObject** representation, the rules for **type compatibility and evolution**, and the mechanisms for *dynamic* types (runtime-defined types usable with `DynamicData`). The machine-readable documents shipped with XTypes 1.3 include the TypeObject IDL and the discovery built-in-topic types IDL ✅ (omg.org file list). Its three **extensibility kinds** — `@final`, `@extensible`, `@mutable` — govern whether fields can be appended or changed without breaking interoperating peers; the DDSI-RTPS 2.5 spec itself uses an `@final struct ShapeType` example with key field `color` ✅ (RTPS 2.5 PDF — the same example also shows the CDR_LE serialized layout, §4.1).
- **Serialization.** On the wire, samples are serialized with the OMG **CDR — Common Data Representation**; the RTPS 2.5 spec's worked example shows the `SerializedPayload` layout "with encoding version 1 and Little Endian byte order", representation identifier `CDR_LE` ✅ (RTPS 2.5 PDF).

---

## 4. The RTPS Wire Protocol and Discovery

### 4.1 RTPS — the Wire Protocol

The **DDS Interoperability Wire Protocol (DDSI-RTPS™)** — full title on the specification cover: *"The Real-time Publish-Subscribe Protocol DDS Interoperability Wire Protocol (DDSI-RTPS) Specification, Version 2.5"* ✅ (RTPS 2.5 PDF, formal/2022-04-01) — is what makes DDS a *standard* rather than a family of compatible-looking APIs: it defines the wire-level behavior that any two compliant implementations use to interoperate. Key verified facts:

- Current formal version **2.5, April 2022** ✅; the version line runs 2.0 (April 2008) → 2.1 (Nov 2010) → 2.2 (Sep 2014) → 2.3 (May 2019) → 2.5 (April 2022) ✅ (omg.org).
- RTPS models the communication in terms of **RTPS Writers and Readers** with attached **HistoryCaches**; reliability on the wire is driven by **Heartbeat/ACKNACK** exchanges between writer and reader caches ⚠-knowledge (the canonical RTPS reliability mechanism; the spec's clauses were not individually re-quoted this pass).
- **Transport:** RTPS is typically carried over **UDP/IP — unicast and multicast** (multicast is what makes one-packet-many-readers fan-out possible), and the DDS spec added TCP support in version 1.2 ✅ (design.ros2.org: "OMG also added support for DDS over TCP in version 1.2 of their specification"). Vendors additionally offer shared-memory and other transports (RTI's own documentation notes UDPv4 + shared memory as Connext defaults ⚠-knowledge, per the design article's quote).
- **Serialization:** CDR with `CDR_LE` (encoding version 1, little-endian) as shown in the spec's worked byte layout ✅ (RTPS 2.5 PDF).
- **Security hook:** RTPS 2.5 references the DDS-SECURITY specification for protecting the protocol ✅ (RTPS 2.5 PDF references list).

### 4.2 SPDP — Simple Participant Discovery Protocol

Discovery in RTPS is explicitly mandated: *"Implementations must implement the Simple Participant and Endpoint Discovery Protocols to enable the [interoperability]..."* ✅ (RTPS 2.5 PDF). The two protocols:

- **SPDP — Simple Participant Discovery Protocol** (RTPS §8.5.3): "uses a simple approach to announce and detect the [participants]" ✅. Each participant creates two built-in endpoints: the **SPDPbuiltinParticipantWriter** — specified as *"an RTPS Best-Effort StatelessWriter"* whose HistoryCache "contains a single data-object of type **SPDPdiscoveredParticipantData**" — and the **SPDPbuiltinParticipantReader** ✅ (RTPS 2.5 PDF, quoted). Participants periodically announce themselves (by default on a well-known multicast group, plus unicast to configured peers ⚠-knowledge for the default group details) and thereby learn each other's GUIDs, locators and QoS.
- SPDP answers only "which participants exist and where to reach them" — it is participant-level discovery.

### 4.3 SEDP — Simple Endpoint Discovery Protocol

- **SEDP — Simple Endpoint Discovery Protocol** (RTPS §8.5.4): "specifies how to exchange discovery information on local [writers and readers]" ✅. Once participants know each other via SPDP, they exchange *endpoint* (DataWriter/DataReader) announcements over RTPS built-in topics, so each side can run the matching logic (topic, type, partition, QoS — §3.3) and then let data flow **directly** between the matched endpoints.
- The RTPS spec notes the mapping explicitly: "The SPDP, which concerns itself with how Participants discover each other, maps the DDS built-in Entities for the 'DCPSParticipant' Topic" ✅ (RTPS 2.5 PDF) — connecting the wire-level protocols to the DDS built-in topic names (§4.4).

### 4.4 The Built-in Topics

Discovery is itself implemented as ordinary DDS publish-subscribe over reserved **built-in topics**:

- **DCPSParticipant** — participant announcements (name verified in the RTPS 2.5 PDF ✅).
- **DCPSPublication / DCPSSubscription / DCPSTopic** — writer and reader endpoint announcements and topic descriptions ⚠-knowledge (standard built-in topic names; the XTypes 1.3 package ships the *discovery built-in topics* IDL file — `dds-xtypes_discovery_builtin_topics.idl` ✅ (omg.org) — which extends this set with type-information topics).

A practical consequence worth internalizing: **discovery traffic is DDS traffic**, which is exactly why DDS-Security's protections (§6.5) apply to discovery too — an unprotected SPDP is an attack surface (spoofed participants, discovery flooding), and a secured SPDP is the first line of defense.

---

## 5. The QoS Policies

### 5.1 The QoS Model

The QoS policy system is what makes DDS *predictable* rather than merely fast: every data stream carries an explicit contract between the **offered** QoS of the writer and the **requested** QoS of the reader, and the middleware enforces it. Two verified anchor facts from the DDS 1.4 spec text:

- The full policy catalogue of DDS 1.4 (§2.2.3 "Supported QoS") is: **USER_DATA, TOPIC_DATA, GROUP_DATA, DURABILITY, DURABILITY_SERVICE, PRESENTATION, DEADLINE, LATENCY_BUDGET, OWNERSHIP, OWNERSHIP_STRENGTH, LIVELINESS, TIME_BASED_FILTER, PARTITION, RELIABILITY, TRANSPORT_PRIORITY, LIFESPAN, DESTINATION_ORDER, HISTORY, RESOURCE_LIMITS, ENTITY_FACTORY, WRITER_DATA_LIFECYCLE, READER_DATA_LIFECYCLE** ✅ (DDS 1.4 PDF table of contents).
- Reliability levels are ordered: "BEST_EFFORT being lower than RELIABLE. A DataWriter offering a level is implicitly offering all levels below" ✅ — the same "offered ≥ requested" compatibility principle generalizes across policies.

### 5.2 Reliability — BEST_EFFORT vs RELIABLE

The RELIABILITY policy has two kinds (verified in the DDS 1.4 PDF ✅):

- **BEST_EFFORT** — "the service will not re-transmit missing data-samples" ✅ (spec quote). The reader gets what arrives; lost samples stay lost. Right for high-rate data where freshness beats completeness: ticks, telemetry, video.
- **RELIABLE** — the service actively ensures delivery: on the wire, RTPS Heartbeat/ACKNACK drives retransmission of missing samples (⚠-knowledge mechanism), and the reader-side HISTORY policy bounds how much is buffered. Right for state and commands: order state, positions, control messages.

### 5.3 Durability — VOLATILE, TRANSIENT_LOCAL, TRANSIENT, PERSISTENT

Durability answers "what does a *late-joining* reader receive?" The four kinds are ordered in the spec: "**VOLATILE < TRANSIENT_LOCAL < TRANSIENT < PERSISTENT**" ✅ (DDS 1.4 PDF), with PERSISTENT described as "Data is kept on permanent [storage]" ✅ (spec quote):

- **VOLATILE** — nothing is kept for late joiners; only samples written after the reader attaches are delivered.
- **TRANSIENT_LOCAL** — the *writer* keeps samples while it exists; a reader joining while the writer is alive gets the retained history. Lost when the writer leaves.
- **TRANSIENT** — a domain-level *durability service* (the spec: the service behaves "as if" there were a built-in DataReader for each TRANSIENT/PERSISTENT topic, keeping the data available even when no writer is present ✅ (DDS 1.4 PDF)).
- **PERSISTENT** — the durability service persists data to permanent storage, surviving restarts.

The Cymbal Bank worked example (§10.3) uses TRANSIENT_LOCAL for order state: a reconnecting risk engine immediately receives the current state of every live order without asking anyone.

### 5.4 History — KEEP_LAST vs KEEP_ALL

HISTORY bounds how many samples per instance are retained (writer side: for retransmission/durability; reader side: for unread samples). Two kinds (verified ✅ in the DDS 1.4 PDF, "KEEP_LAST is the default kind"):

- **KEEP_LAST with depth N** — keep only the most recent N samples per instance. Default.
- **KEEP_ALL** — keep everything (bounded only by RESOURCE_LIMITS).

For a market-data topic, KEEP_LAST depth 1 on the reader side means "only the latest tick per symbol" — the classic last-value-cache behavior that data-centric middleware is famous for.

### 5.5 Deadline and Latency Budget

- **DEADLINE** (catalogue §2.2.3.7 ✅): a duration contract — the writer must produce a sample for each instance within every deadline period; the reader expects one. Violations surface as the communication statuses **OFFERED_DEADLINE_MISSED** (writer side) and **REQUESTED_DEADLINE_MISSED** (reader side) — both status names verified in the DDS 1.4 spec text ✅. This is how "is the feed still alive and current?" becomes a *measured, actionable* signal rather than a guess.
- **LATENCY_BUDGET** (catalogue §2.2.3.8 ✅): a hint about the acceptable end-to-end latency from writer to reader; implementations use it for scheduling/transport decisions ⚠-knowledge (semantics not re-quoted this pass).

### 5.6 Liveliness

LIVELINESS (catalogue §2.2.3.11 ✅) answers "is the peer still there and still publishing?" The three kinds are ordered in the spec — "**AUTOMATIC < MANUAL_BY_PARTICIPANT < MANUAL_BY_TOPIC**" ✅ — with lease-duration settings:

- **AUTOMATIC** — the middleware asserts liveliness for the application, transparently, as long as the participant is alive and writing.
- **MANUAL_BY_PARTICIPANT** — the application must explicitly assert liveliness (e.g. from a heartbeat thread); the spec notes `assert_liveliness` on the participant "need only be used if the ... LIVELINESS setting is either MANUAL_BY_PARTICIPANT or MANUAL_BY_TOPIC" ✅ (DDS 1.4 PDF).
- **MANUAL_BY_TOPIC** — "requires that at least one instance within the DataWriter is asserted" ✅ (spec quote); the strictest form, used when *producing the data itself* is the proof of life (e.g. a feed handler that must be actively publishing ticks).
- Readers observe the **LIVELINESS_CHANGED** status ✅ (DDS 1.4 PDF) — the event that drives the trading floor's "venue feed lost" alarms in §10.

### 5.7 Ownership

OWNERSHIP (catalogue §2.2.3.9 ✅) controls what happens when *multiple writers* publish the same instance: "There are two kinds of OWNERSHIP selected by the setting of the kind: **SHARED and EXCLUSIVE**" ✅ (spec quote):

- **SHARED** — all writers' samples are delivered (the normal multi-producer case).
- **EXCLUSIVE** — only the writer with the highest **OWNERSHIP_STRENGTH** (catalogue §2.2.3.10 ✅) may publish the instance; the spec's §2.2.3.23 covers "Ownership resolution on redundant systems" ✅ — the active/standby pattern: the standby feed handler holds a lower strength, and when the active dies, the standby automatically takes over ownership without any client-side failover code. The worked example (§10.3) relies on exactly this.

### 5.8 The Rest of the Catalogue

All policy names verified in the DDS 1.4 catalogue ✅; semantics are ⚠-knowledge one-liners unless noted:

| Policy | One-line semantics |
| --- | --- |
| USER_DATA / TOPIC_DATA / GROUP_DATA | Opaque application bytes attached to entities/topics/groups — used for tagging and filtering |
| DURABILITY_SERVICE | Settings for the TRANSIENT/PERSISTENT durability service (cleanup delay, history) |
| PRESENTATION | Coherence/ordering scope for groups of samples (topic vs group) |
| TIME_BASED_FILTER | Reader-side minimum inter-arrival time — the reader says "don't deliver faster than this" |
| PARTITION | Logical subdivision of the domain (§3.4) |
| TRANSPORT_PRIORITY | Hint for transport selection/prioritization |
| LIFESPAN | Expiration time for samples — stale data is dropped automatically |
| DESTINATION_ORDER | Ordering of samples from multiple writers (BY_RECEPTION_TIMESTAMP / BY_SOURCE_TIMESTAMP) |
| RESOURCE_LIMITS | Max samples/instances the middleware may allocate per entity |
| ENTITY_FACTORY | Whether child entities are created enabled or auto-enabled |
| WRITER_DATA_LIFECYCLE / READER_DATA_LIFECYCLE | Auto-dispose/auto-purge behavior for instances on the writer and reader sides |

---

## 6. The DDS Security Model

### 6.1 The Specification and the Plugin Architecture

DDS was designed for environments where "add a broker with TLS" is not an option — so security is specified *inside* the standard, as a pluggable layer. The **DDS-SECURITY** specification (current formal version **1.2**, OMG file id **formal/25-03-06** ✅ omg.org; version history in §2.3) defines:

- The **Service Plugin Interface (SPI)** — the extension points through which security functionality plugs into a DDS implementation. The SPI covers four plugin kinds: **Authentication**, **Access Control**, **Cryptographic** and **Logging** ⚠-knowledge (plugin structure per the spec; the plugin interfaces are standard, well-documented spec knowledge, not re-quoted from the PDF this pass).
- **Three standardized built-in plugin profiles** — the ones every compliant implementation ships, and the ones §10.4 deploys:

| Plugin kind | Built-in profile name | Job |
| --- | --- | --- |
| Authentication | **DDS:Auth:PKI-DH** | Prove who each participant is (X.509 certificates) and derive shared secrets |
| Access Control | **DDS:Access:Permissions** | Decide what each authenticated participant may publish or subscribe to |
| Cryptographic | **DDS:Crypto:AES-GCM** | Protect data in transit (confidentiality + integrity) and protect discovery |

✅ (spec structure and plugin names verified on omg.org; mechanism details ⚠-knowledge).

The central identity concept: **the DomainParticipant is the security principal.** When DDS-Security is enabled for a domain, every participant must authenticate itself before any discovery information or data is exchanged with peers — the handshake happens *before* the first SPDP/SEDP sample is trusted (§4.4, §6.5). Implementations without a plugin still interoperate; the plugins kick in only for participants that declare them — so a secured domain and an unsecured domain can coexist on the same network without leaking data to each other ⚠-knowledge.

### 6.2 Authentication — DDS:Auth:PKI-DH

The built-in authentication profile, **DDS:Auth:PKI-DH**, is a certificate-based mutual-authentication and key-agreement scheme (mechanism details ⚠-knowledge — the profile is public-key based with DH-derived secrets per the standard):

- Every participant holds an **X.509 certificate** (signed by a CA that its peers are configured to trust) plus a private key. The certificate binds the participant's identity — its *principal* — to its public key.
- During the handshake, two participants **mutually authenticate** (each proves possession of its private key) and run a **Diffie-Hellman key agreement** to derive shared secrets. Those secrets are then used to bootstrap the cryptographic plugin's key material (§6.4).
- The handshake itself is carried inside the RTPS discovery exchange — which is why protecting discovery is mandatory in a secured domain (§6.5).
- Net effect: **impersonation is prevented.** A rogue node that spoofs a feed handler's GUID cannot join the domain — it has no valid certificate, and the handshake fails before any endpoint matching happens.

### 6.3 Access Control — DDS:Access:Permissions

The built-in access-control profile, **DDS:Access:Permissions**, governs *what an authenticated participant may do*, through two XML documents (⚠-knowledge — document structure and names are standard, well-documented spec knowledge):

- **The Governance document** — domain-wide settings: which topics require which protections (none / sign-only / encrypt-and-sign), whether discovery traffic must be protected, and the acceptable algorithm and key-length choices. It is the domain's *security policy*.
- **The Permissions document** — per-principal grants: for each participant (identified by its certificate), which topics it may **publish**, **subscribe** to, or **relay**, expressed as topic-name patterns (e.g. `EQ.*`), with optional validity periods and default-deny rules.

Enforcement happens at endpoint creation and matching: a DataWriter whose principal lacks publish permission for its topic fails to enable; a DataReader without subscribe permission is never matched to writers. This is the mechanism behind §10.4's "the risk engine may read everything, trading apps may read only their desks' partitions, nobody else may publish order state."

### 6.4 Cryptographic — DDS:Crypto:AES-GCM

The built-in cryptographic profile, **DDS:Crypto:AES-GCM**, provides the actual wire protection (⚠-knowledge — the profile's algorithm families are standard spec knowledge):

- **AES-GCM** (Galois/Counter Mode — an authenticated-encryption AEAD cipher) for confidentiality *and* integrity in one pass; **AES-GMAC** for integrity-only protection of high-rate streams where encryption cost matters.
- Key material is derived from the authentication handshake's DH secrets (§6.2) — keys never cross the wire in the clear, and different endpoints/topics get distinct keys.
- The profile protects **both discovery metadata and payload samples**, at the granularity the Governance document dictates.

### 6.5 Protecting Discovery

Because discovery *is* DDS traffic (§4.4), an unsecured SPDP/SEDP is an attack surface: an attacker can announce a spoofed participant, flood discovery, or poison the endpoint database. DDS-Security closes this in two ways:

- **Authenticated discovery** — SPDP/SEDP samples are exchanged only after (and inside) the authenticated handshake, so a peer's announcements are trusted only if that peer proved its identity (§6.2).
- **Protected metadata** — discovery samples themselves can be signed or encrypted under the cryptographic plugin, so they cannot be forged or replayed ⚠-knowledge.

Operational rule of thumb for §10: **turn security on at the domain level and let the Governance document decide per-topic protections** — protecting the discovery plane matters as much as protecting the payload, because a poisoned discovery plane can redirect or deny the data plane.

---

## 7. The Implementations

### 7.1 The Landscape Table

| Implementation | Vendor / Origin | License | One-line identity | Status |
| --- | --- | --- | --- | --- |
| RTI Connext | Real-Time Innovations, US | Commercial (proprietary) | The long-standing commercial leader; full product suite with safety certifications | ✅ rti.com |
| Eclipse Cyclone DDS | Eclipse Foundation; originated at ADLINK, stewarded by ZettaScale | EPL-2.0 (open source) | Fast, compact C implementation; a tier-1 ROS 2 middleware | ✅ eclipse.dev / GitHub |
| eProsima Fast DDS | eProsima, Spain | Apache-2.0 (open source) | Default RMW of ROS 2 LTS releases; "always open source" | ✅ eprosima.com |
| OpenDDS | Object Computing Inc (OCI), US | Open source (permissive custom license) | Long-running C++ implementation with Java bindings | ✅ opendds.org |
| GurumDDS | GurumNetworks, South Korea | Commercial | Korean DDS with a ROS 2 RMW, routing service and security products | ✅ gurumdds.com |

Two further names worth knowing: **OpenSplice DDS** — the ADLINK/ZettaScale commercial DDS for the mission-critical market, part of the ZettaScale portfolio since the 2022 ADLINK spin-out (✅ adlinktech.com press release) — and the **DDS Foundation** (dds-foundation.org), the industry body several vendors co-founded to promote DDS adoption ⚠-knowledge.

### 7.2 RTI Connext — Real-Time Innovations

- **Real-Time Innovations, Inc.** — the company that co-led the original DDS effort (RTI is among the DDS 1.0 copyright contributors, §2.1 ✅). Founded 1991, headquartered in Sunnyvale, California ⚠-knowledge (company history well documented; not re-verified this pass).
- **Connext** is the commercial flagship: *"RTI Connext is the only real-time data streaming platform built for intelligent physical systems"* — vendor positioning ✅ (rti.com); RTI explicitly positions it as *"an alternative to broker-based solutions such as MQTT and Kafka"* with a *"decentralized architecture that reduces latency, increases throughput, and eliminates any single point of failure"* ✅ (rti.com).
- **The product suite** (✅ rti.com): **Connext Professional** (time/mission-critical systems), **Connext Micro** (resource-constrained systems), **Connext Cert** (for **ISO 26262 and DO-178C** requirements), **Connext Drive** (software-defined vehicles), **Connext TSS** (FACE-conformant systems — see §8.3/§8.5).
- **Scale claim:** *"over 2,000 designs"* ⚠ (vendor claim, not independently audited).
- **Defense posture:** RTI announced **CMMC Level 2 certification** to support mission-critical defense programs requiring the protection of controlled unclassified information (June 2026 news ✅ rti.com).
- Customers named on rti.com include Aptiv, XPeng Motors, Medtronic, Stryker and REGENT ⚠ (vendor-published logos/announcements — marketing, not an independent census).

### 7.3 Eclipse Cyclone DDS — ZettaScale

- **Eclipse Cyclone DDS** is an open-source DDS implementation hosted by the **Eclipse Foundation** as an Eclipse IoT project, licensed **EPL-2.0** ✅ (GitHub LICENSE file + projects.eclipse.org).
- It was created at **ADLINK Technology**; when **ZettaScale Technologies spun out of ADLINK** (announced May 2022, with TTTech Auto as strategic investor), ZettaScale took stewardship of the Eclipse-hosted **Cyclone DDS and Zenoh** projects, alongside the commercial **OpenSplice DDS** for the mission-critical market ✅ (adlinktech.com press release).
- Written in C for performance and a small footprint; the project describes itself as *"a very performant and robust open-source implementation of the OMG DDS specification"* and a *"tier-1 middleware for ROS 2"* ✅ (GitHub project page — the tier-1 framing is the project's own ⚠).
- Practical significance for this guide: Cyclone DDS is the RMW most often cited in ROS 2 performance work, and Zenoh (§9.5) — the closest thing to a "next-generation DDS" — comes from the same lineage.

### 7.4 eProsima Fast DDS

- **eProsima** is a Spanish company (Tres Cantos, Madrid) focused on DDS, ROS 2 and robotics ✅ (eprosima.com).
- **Fast DDS** (formerly *Fast RTPS* ⚠-knowledge) is its open-source implementation, *"always open source under the Apache 2.0 license"* ✅ (eprosima.com) — Apache-2.0 is the permissive license that made it attractive to robotics and automotive.
- **The ROS 2 default:** eProsima's documentation states Fast DDS *"is the default middleware implementation in ROS 2 in every long term (LTS) releases and most of the non-LTS releases"* ✅ (fast-dds.docs.eprosima.com). This is corroborated by Open Robotics' own choice — Brian Gerkey (Open Robotics CEO): *"we selected the DDS standard as the basis for communication within ROS… we have selected eProsima Fast DDS as the default for ROS because of the permissive open source license, implementation quality, and friendly support"* ✅ (quote published on eprosima.com).
- **Vendor claims, flagged ⚠:** *"the most adopted open-source DDS implementation"*, *"over 1M users"*, *"50,000 downloads/month"* — self-reported marketing metrics.
- **Commercial siblings:** **Fast DDS Pro** (advanced networking + professional support) and **Safe DDS** (claimed **ISO 26262 ASIL D** certified ⚠ vendor claim) ✅ (eprosima.com).

### 7.5 OpenDDS — Object Computing Inc

- **OpenDDS** is an open-source **C++** implementation of DDS, *"developed and open sourced by Object Computing (OCI)"* — an American consultancy headquartered in St. Louis, MO; Java applications are supported through **JNI bindings** ✅ (opendds.org).
- **License:** open source and free of licensing fees; users may use, modify and distribute the code (keeping the copyright notice), and *"can use OpenDDS to build proprietary software… under no obligation to redistribute"* ✅ (opendds.org license page). Note: this is OCI's own permissive license — a relicensing to Apache-2.0 was under community discussion (GitHub discussion #4345 ⚠).
- **Release cadence:** current release **3.34.0 (May 2026)** ✅ (opendds.org); notable features include **RtpsRelay**, a relay service that lets RTPS discovery and data traverse WANs and NATs ✅ (GitHub release notes).
- OpenDDS is a common choice in US aerospace/defense-adjacent integration projects ⚠-knowledge (industry pattern, not verified per-customer this pass).

### 7.6 GurumDDS — GurumNetworks

- **GurumDDS** is the product of **GurumNetworks**, a South Korean company — a **commercial** DDS implementation (the company's site describes it as a commercially productized Korean implementation of the OMG standard) ✅ (gurumdds.com).
- **Conformance claims:** the vendor states conformance to *DCPS 1.4* and *RTPS 2.2* ⚠ (vendor claim; note RTPS 2.2 predates the current 2.5 — see §4.1 — so wire-level conformance to the *current* RTPS revision is not claimed).
- **Product line** ✅ (gurumdds.com): **GurumDDS RMW** (a ROS 2 middleware implementation — GurumDDS is on ROS 2's supported RMW list ✅ docs.ros.org), **GurumDDS RS** (routing service for WAN operation), **GurumDDS Gateway** (LAN-to-LAN global-space merging), **GurumDDS XRCE** (constrained/low-power devices), and **GurumDDS Security**.
- **Markets** ✅/⚠ (vendor solution pages): robotics, autonomous driving (positioned alongside **AUTOSAR Adaptive and ROS 2**), railway safety-management systems, wind-energy plants, and defense **C4I** interoperability. The vendor claims "100+ critical distributed systems" ⚠ (self-reported).
- Significance: GurumDDS shows the DDS ecosystem is genuinely global — Korea's defense/rail/energy sector runs its own commercial DDS, and it interoperates with the rest of the ecosystem through the RTPS wire protocol.

### 7.7 The Market-Share Caveat

There is **no independent, audited market-share study** for DDS implementations — every "leader" claim in this section is the vendor's own ⚠:

- RTI: *"over 2,000 designs"* (rti.com) ⚠.
- eProsima: *"the most adopted open-source DDS implementation"*, 50K downloads/month (eprosima.com) ⚠.
- GurumNetworks: *"100+ critical distributed systems"* (gurumdds.com) ⚠.
- Cyclone DDS: *"tier-1 middleware for ROS 2"* (project README) ⚠.

What *is* independently verifiable: **Fast DDS holds the ROS 2 default-RMW position** in LTS releases (✅ §8.1), **ROS 2 itself defaults to DDS** (✅ docs.ros.org), and **FACE mandates DDS** in its conformance classes for avionics (✅ §8.3/§8.5). Procurement advice: pick by requirements — license, certifications, RMW support, WAN tooling, security features — not by marketing share.

---

## 8. The Ecosystem and Use Cases

### 8.1 ROS 2 — the Default Middleware

- **ROS 2 uses DDS as its default middleware.** The ROS 2 Jazzy documentation states it plainly: *"By default, ROS 2 uses DDS as its middleware"* ✅ (docs.ros.org/en/jazzy, "RMW implementations"). ROS 1's centralized master node was replaced in ROS 2 by a decentralized, DDS-based discovery and transport layer.
- **Why:** Open Robotics evaluated middleware options during ROS 2's design and *"selected the DDS standard as the basis for communication within ROS"* ✅ (Brian Gerkey quote, published on eprosima.com). DDS gave ROS 2 automatic discovery, a rich QoS system (reliability, durability, deadlines, liveliness), and no single point of failure — properties ROS 1's TCPROS/NodeMaster architecture lacked.
- **The RMW abstraction:** ROS 2 talks to DDS through *rmw* (ROS middleware) implementations — `rmw_fastrtps_cpp` (Fast DDS), `rmw_cyclonedds_cpp` (Cyclone DDS), `rmw_connextdds` (RTI Connext), `rmw_gurumdds` (GurumDDS) — plus non-DDS RMWs such as **Zenoh** ✅ (docs.ros.org).
- **Default vendor:** **Fast DDS** is the default RMW in ROS 2 LTS releases ✅ (§7.4).
- Consequences for the industry: every robot, drone and autonomous-vehicle stack built on ROS 2 ships a DDS implementation — this single ecosystem is the largest source of DDS adoption in the world today.

### 8.2 Autonomous Vehicles

- AV stacks need exactly what DDS provides: high-rate sensor-data fan-out (lidar/camera/radar), deterministic QoS, no single point of failure, and security (§6). Both major AV software bases — **ROS 2** (research through production stacks) and **AUTOSAR Adaptive** (production vehicle platforms) — have DDS integrations; GurumDDS, for instance, markets itself against exactly this "AUTOSAR Adaptive + ROS 2" combination ⚠/✅ (gurumdds.com solution page).
- RTI ships **Connext Drive** *"for software-defined vehicles"* and lists AV/automotive customers including Aptiv, XPeng Motors and Inceptio ⚠ (vendor-published; rti.com).
- ZettaScale's automotive play (with TTTech Auto) targets safety-certified DDS for series vehicles — the partnership and the Motionwise Cyclone DDS product are documented ✅ (adlinktech.com press release); certification status is ⚠.
- ⚠ Caveat: there is no independent census of "which middleware runs in which vehicle" — vendor customer lists are marketing. What is solid is the *pattern*: vehicle platforms standardize on DDS-family middleware because of the brokerless failover and the QoS model.

### 8.3 Defense

- DDS's roots are defense: the original DDS consortium included **Thales** (DDS 1.0 copyright contributors, §2.1 ✅), and the standard was built for platform and mission-system integration.
- The strongest verifiable anchor: the **FACE** (Future Airborne Capability Environment) standard from **The Open Group** mandates DDS for its data-transport segment — RTI's **Connext TSS** is a FACE-conformant product ✅ (rti.com). FACE is how the US avionics community buys portable software components, and its data-transport model is DDS-based ⚠-knowledge (the FACE technical standard's DDS requirement is well documented; verified here via RTI's product page).
- RTI's **CMMC Level 2** certification (June 2026) targets *"mission-critical defense programs requiring the protection of Controlled Unclassified Information"* ✅ (rti.com news).
- GurumDDS is deployed for **C4I** interoperability in Korea's defense sector ⚠ (vendor solution page).
- Why defense chose DDS: brokerless survivability (no single point of failure under attack), QoS contracts for time-sensitive data, multicast efficiency, and a security model specified in the standard itself (§6).

### 8.4 Industrial IoT — DDS vs OPC UA

The OT/IIoT companion discussion lives in the sibling [SCADA, ICS and OT Security](scada_guide.md) guide (§5.3 there covers **OPC UA — the OPC Foundation, IEC 62541** ✅ per that guide); this subsection positions DDS against it:

- **OPC UA** is the plant-floor standard: an information-model-rich, client/server (plus PubSub) framework for machine-to-machine and machine-to-supervision data exchange. It dominates PLC/SCADA/MES integration.
- **DDS** is the distributed real-time standard: brokerless, QoS-governed, typed data distribution across many nodes. It dominates robotics, vehicle platforms, and distributed control where latency and failover behaviour are contractual.
- **The overlap:** OPC UA **PubSub** explicitly defines transport mappings, including a mapping **over DDS** ⚠-knowledge (the OPC UA PubSub specification documents a DDS transport mapping; widely referenced, not re-fetched this pass) — the two standards are designed to interoperate rather than only compete.
- **Choosing:** brownfield plant integration with existing PLCs/HMIs → **OPC UA**; a new multi-node real-time data-distribution problem with hard QoS → **DDS**; most real estates run both, bridged by gateways. The SCADA guide covers the security context of putting either on an OT network.

### 8.5 Aerospace

- The FACE mandate (§8.3) makes DDS the data transport of record for much of the US airborne-software world ✅ (rti.com TSS).
- RTI's **Connext Cert** is positioned *"for ISO 26262 and DO-178C requirements"* ✅ (rti.com) — i.e. certifiable against the aviation (DO-178C) and automotive (ISO 26262) safety standards.
- RTI's market copy cites air traffic control among its industries, and its news flow includes aerospace programs (e.g. REGENT's all-electric seaglider using Connext, June 2026 ✅ rti.com).
- ⚠-knowledge: beyond these anchors, specific program names and per-implementation certification levels were not independently verified this pass.

### 8.6 Medical

- DDS appears in medical and surgical-robotics systems where real-time, fault-tolerant device communication is required; RTI lists **Medtronic, Stryker and Levita Magnetics** among customers ⚠ (vendor-published logos).
- eProsima's customer set likewise includes medical-adjacent robotics ⚠ (vendor page).
- ⚠-knowledge: no certification claims (FDA/CE/ISO 13485) are asserted for any implementation in this guide — certification is per-program engineering, not a middleware property.

### 8.7 Financial Market Data

- The fit: a trading floor is a *data-centric* problem — many producers (venue feed handlers), many consumers (trading desks, risk, compliance, algos), hard latency and availability requirements, and a need for efficient fan-out. That is precisely DDS's design point: brokerless multicast fan-out (§1.5), DEADLINE/LIVELINESS contracts (§5.5/§5.6), and ownership-based failover (§5.7).
- **Where DDS sits in the stack:** venue-facing sessions belong to **FIX** (the [FIX Protocol guide](../banking/fix_protocol_guide.md) — session layer §4, FIXP §5.3); durable event storage and analytics belong to **Apache Kafka** (the [Apache Kafka guide](../banking/kafka_guide.md)); the *internal real-time bus* between feed handlers, trading applications and risk engines is where DDS earns its place — see the worked example (§10).
- **Adoption reality** ⚠-knowledge: DDS is a real but niche player in market-data distribution compared with proprietary multicast-feed systems and TIBCO-style bus software; its strongest financial traction is in algo-trading/execution stacks and in firms that also run ROS 2 or defense-style architectures. This guide cites no specific bank deployments — none were verified this pass.
- The boundary in one sentence: FIX owns the venue session, DDS owns the internal real-time bus, Kafka owns the replay/analytics log, OPC UA owns the plant floor (§8.4).

---

## 9. The Head-to-Head Comparison

### 9.1 The Comparison Table

| Dimension | DDS | MQTT | AMQP 1.0 | Apache Kafka | Eclipse Zenoh |
| --- | --- | --- | --- | --- | --- |
| Standard / origin | OMG, 2004 (DDS 1.4 current) | OASIS (3.1.1 also ISO; 5.0 current) | OASIS, 30 Oct 2012 | Apache; LinkedIn, 2011 | Eclipse; ZettaScale (ADLINK lineage) |
| Architecture | **Brokerless** — peer-to-peer, decentralized discovery | **Brokered** — client/server against a broker | **Brokered in practice** — symmetric wire protocol, broker-centric deployments | **Brokered** — partitioned log cluster | **Brokerless-capable** — peer-to-peer by design, optional routers |
| Communication model | Data-centric, typed pub/sub (DCPS) | Message pub/sub, untyped hierarchical topics | Message pub/sub + queues | Log-based streaming: partitioned topics, offsets, replay | Pub/sub + query + geo-distributed storage |
| Data typing | OMG IDL + XTypes type system | None (payload = bytes) | AMQP application-level types | None natively (Schema Registry optional) | Key/value + user-defined encoding |
| QoS / guarantees | 22 policies: reliability, durability, deadline, liveliness, ownership… | QoS 0/1/2 (at most / at least / exactly once) | Delivery guarantees per link and queue | acks + ISR replication, retention, compaction | Reliability levels, fragmentation, batching |
| Discovery | Automatic (SPDP/SEDP over RTPS) | Via broker connection (no peer discovery) | Via broker (URL-based) | Via broker metadata (KRaft) | Dynamic, decentralized |
| Failure model | No single point of failure | Broker is a SPOF (mitigated by clustering) | Broker SPOF (mitigated by clustering) | Broker cluster with replication (ISR) | No SPOF in peer-to-peer mode |
| Typical latency | µs–ms on LAN | Low (TCP) | Low | ms-class (persistence-bound) | Very low (4–6-byte wire overhead) |
| Scale direction | LAN peer-to-peer; WAN via routers/discovery servers | Millions of constrained devices → cloud brokers | Enterprise integration | Massive partitioned throughput | Cloud-to-microcontroller, Internet scale |
| Durability / replay | TRANSIENT/PERSISTENT via durability service; late-joiner catch-up (§5.3) | Broker-retained messages per topic | Durable queues | Full log replay by offset | Geo-distributed storages with sharding and replication |
| Sweet spot | Real-time, embedded, robotics, AV, defense, market data | IoT telemetry, M2M, mobile push | Enterprise messaging, integration | Event streaming, ETL, analytics | Robotics, edge-cloud, constrained devices, IoT |

Cell semantics are ⚠-knowledge one-liners unless quoted elsewhere; the row-level facts (standards, licenses, architecture class) are ✅ per §11.

### 9.2 DDS vs MQTT

- **MQTT** is the lightweight broker-centric IoT protocol: an **OASIS standard** — MQTT 3.1.1 is "an older ISO and OASIS Standard" and MQTT 5.0 is the current OASIS Standard ✅ (mqtt.org); the exact approval dates (3.1.1 October 2014, 5.0 March 2019) are ⚠-knowledge. Topics are hierarchical, untyped byte-stream names; QoS 0/1/2 covers at-most/at-least/exactly-once delivery; retained messages and will messages are broker-side conveniences.
- **The structural contrast:** MQTT is *broker-centric* — every message passes through a broker, which is a single point of failure and a latency hop. Zenoh's designers put it sharply: *"MQTT suffered from a broker paradox: two devices on the same local network still had to route communication through a remote cloud broker"* ✅ (zenoh.io — a competitor's framing, but the architectural point stands).
- **Choosing:** MQTT when the estate is constrained devices + cloud IoT telemetry and a broker is acceptable; DDS when peers need real-time, typed, QoS-governed communication with no single point of failure. Real estates bridge the two (e.g. MQTT bridges/gateways into a DDS domain ⚠-knowledge — vendor tooling exists but was not inventoried this pass).

### 9.3 DDS vs AMQP

- **AMQP 1.0** is the enterprise messaging wire protocol, an **OASIS standard approved 30 October 2012** ✅ (established fact; docs.oasis-open.org). Its lineage is financial: the AMQP working group was seeded by JPMorgan and other firms in the mid-2000s ⚠-knowledge.
- AMQP 1.0 is technically a *symmetric* protocol — any node can act as sender or receiver — but the deployments that made it famous (**RabbitMQ, ActiveMQ, Azure Service Bus, IBM MQ**) are **broker-centric** ⚠-knowledge. Its lane is enterprise integration: queues, routing, transactions, cross-organization messaging.
- **Choosing:** AMQP when the problem is enterprise messaging/integration with queues and routing; DDS when the problem is real-time data distribution with QoS. They coexist: AMQP brokers often sit at integration boundaries while DDS carries the real-time core (the Cymbal Bank estate of §10 uses FIX/Kafka/DDS; an AMQP layer would appear only at third-party integration boundaries ⚠-knowledge).

### 9.4 DDS vs Apache Kafka

- **Kafka** is a distributed, partitioned, replicated **log** — see the [Apache Kafka guide](../banking/kafka_guide.md) for the full treatment (§2 architecture — brokers, topics, partitions; §3 replication and ISR; §5 streaming semantics). Producers append to partition logs; consumers read by offset; retention (time/size) deletes old data; consumer groups rebalance. Kafka originated at LinkedIn in 2011 ✅ (per the sibling guide §1.3).
- **The essential contrast** (echoing §1.5): DDS is a *live-state bus* — data flows directly peer-to-peer, and "durability" means late-joining readers catch up to current state (TRANSIENT/PERSISTENT, §5.3). Kafka is a *durable log* — everything is written to replicated disk logs and consumers replay from arbitrary offsets.
- **What each lacks:** DDS has no equivalent of Kafka's retention-and-replay semantics; Kafka has no equivalent of DDS's automatic discovery, multicast fan-out, deadline/liveliness contracts, or ownership-based failover.
- **They are complements, not rivals.** The Cymbal Bank estate runs both: DDS for the real-time market-data/order-state bus (§10), Kafka for the analytics/audit log — the compliance recorder subscribes on DDS and writes Kafka (cross-ref the [Kafka guide](../banking/kafka_guide.md) §10 worked example for the analytics side).
- Also relevant: the sibling [Kafka Alternatives](kafka_alternatives_guide.md) and [Event Stream Processing](event_stream_processing_guide.md) guides map where each bus/queue/stream pattern fits.

### 9.5 DDS vs Eclipse Zenoh

- **Zenoh** is a pub/sub/query protocol from **Eclipse**, stewarded by **ZettaScale**, designed by **Angelo Corsaro** — at the time CTO of PrismTech and *co-chair of the OMG DDS specification group* ✅ (zenoh.io). It is the closest thing to a "DDS successor-adjacent" design: data-centric and location-transparent like DDS, but built for the cloud-to-microcontroller span DDS was not designed for.
- **Topology is flexible:** *"peer-to-peer, brokered, or any hybrid, decided at runtime"* ✅ (zenoh.io) — so Zenoh is **brokerless-capable** (peer-to-peer mode, no single point of failure) while also supporting **routers** for Internet-scale deployments. The honest answer to "is Zenoh brokerless?" is: yes by default, optional routers when you need scale ✅.
- **Numbers:** minimal wire overhead of **4–6 bytes** (site copy varies between "5 bytes" and "4–6 bytes" — both appear on zenoh.io ✅) and an implementation footprint as small as ~300 bytes on an 8-bit Atmel microcontroller ✅; implemented in Rust with multi-language APIs ✅.
- **Ecosystem:** adopted in robotics/AV initiatives such as CARMA and the Indy Autonomous Challenge ✅ (zenoh.io), and supported as a **non-DDS RMW in ROS 2** ✅ (docs.ros.org).
- **Choosing:** stay on DDS when you need the OMG ecosystem — the QoS catalogue, XTypes typing, DDS-Security, ROS 2 default interop, defense/avionics mandates. Choose Zenoh when the span is cloud↔edge↔device, when queries/storages as first-class abstractions matter, or when wire overhead and constrained transports (Layer 2, LPWAN) dominate (positioning per zenoh.io + this guide's analysis ⚠).

## 10. The Cymbal Bank Worked Example — Market Data on the Trading Floor

### 10.1 The Problem

Cymbal Bank's trading floor runs several desks (equities, FX, fixed income), a central risk engine, algos, and compliance. Market data arrives from venues and aggregators through **FIX feed handlers**; every consumer needs a different slice, and the data must keep flowing when any single node dies. Requirements that rule out the obvious alternatives:

- **Brokerless:** a market-data broker would be a single point of failure and a latency hop at tick rates — the exact argument RTI makes against MQTT/Kafka architectures (§7.2) and the core of §1.5.
- **Typed + QoS:** consumers need contracts — "every symbol updates at least once per second" is a *deadline*, not a hope (§5.5).
- **Security:** order state and some feeds are confidential; only authorized desks may read or write them (§6).
- **FIX stays at the boundary:** FIX is a session protocol for counterparties (see the [FIX Protocol guide](../banking/fix_protocol_guide.md) §4), not a data-distribution bus — nobody wants a FIX session per consumer.

The design: one **DDS domain** (DomainId 100 — the market-data domain), partitioned per asset class and environment, carrying market data, order state and system status; FIX handles venue sessions; Kafka handles the analytics replay log.

### 10.2 The Domain Design

- **One domain, many partitions** (PARTITION policy, §3.4) — partitions subdivide the domain without new domain ids:
  - `PROD/EQ`, `PROD/FX`, `PROD/FI` — per asset class in production;
  - `UAT/*`, `DR/*` — test and disaster-recovery copies of the same topics;
  - writers publish into `PROD/EQ`; readers subscribe to `PROD/EQ` — an equity desk never sees FX traffic even though everything lives in Domain 100.
- **Topics** (IDL types, §3.5):

| Topic | Type (key) | Producers | Consumers |
| --- | --- | --- | --- |
| `MarketDataTick` | struct, key = symbol | Feed-handler cluster (active + standby writers) | Trading apps, algos, risk engine, market-data cache |
| `OrderState` | struct, key = order ID | Order gateways (from FIX ExecutionReports) | Risk engine, compliance, trading apps |
| `PositionUpdate` | struct, key = book | Risk engine (recomputed) | Trading apps, compliance |
| `VenueStatus` | struct, key = venue | Feed-handler cluster | Trading apps, ops dashboard |
| `Heartbeat` | struct, key = node | Every participant | Ops dashboard, risk engine |

- **Participants:** FeedHandler nodes (publish ticks + venue status; own the FIX sessions), OrderGateway nodes (bridge DDS order state ↔ FIX execution, §10.5), Trading apps (subscribe ticks; publish order intents), Risk engine (subscribe to everything relevant; publish positions), Market Data Cache (subscribe ticks, re-publish TRANSIENT for late joiners), Compliance recorder (subscribe order state; write the Kafka audit log).
- **Liveliness wiring:** each feed handler asserts **MANUAL_BY_TOPIC** liveliness on `MarketDataTick` while it is actively publishing — the **LIVELINESS_CHANGED** status (established in §5.6) drives the ops dashboard's "venue feed lost" alarms, and the risk engine widens its VaR assumptions until the feed returns.

### 10.3 The QoS Profile

The QoS profile *is* the contract. (All policy semantics are in §5.)

| Topic | RELIABILITY | DURABILITY | HISTORY | DEADLINE | LIVELINESS | OWNERSHIP |
| --- | --- | --- | --- | --- | --- | --- |
| `MarketDataTick` (L1) | BEST_EFFORT | VOLATILE | KEEP_LAST 1 | 1 s | MANUAL_BY_TOPIC, lease 5 s | EXCLUSIVE — active strength 10, standby 5 |
| `MarketDataTick` (L2 depth) | RELIABLE | VOLATILE | KEEP_LAST 5 | 100 ms | MANUAL_BY_TOPIC, lease 5 s | EXCLUSIVE (as above) |
| `OrderState` | RELIABLE | TRANSIENT_LOCAL | KEEP_LAST | 500 ms | AUTOMATIC, lease 10 s | EXCLUSIVE — the order gateway owns its orders |
| `PositionUpdate` | RELIABLE | TRANSIENT_LOCAL | KEEP_LAST 1 | 1 s | AUTOMATIC, lease 10 s | SHARED |
| `VenueStatus` | RELIABLE | TRANSIENT_LOCAL | KEEP_ALL | — | MANUAL_BY_TOPIC, lease 5 s | SHARED |

Why these choices:

- **Ticks:** BEST_EFFORT + KEEP_LAST 1 = freshest-price-wins — a lost tick is irrelevant if the next arrives 50 ms later; freshness beats completeness (§5.2). DEADLINE 1 s turns "is the feed current?" into a measured signal (§5.5).
- **L2 depth:** RELIABLE + KEEP_LAST 5 — depth rebuilds need the recent book, so loss matters.
- **OrderState:** RELIABLE + **TRANSIENT_LOCAL** — a reconnecting risk engine immediately receives the current state of every live order without asking anyone (the property promised in §5.3); EXCLUSIVE ownership means only the owning gateway may update an order instance (§5.7).
- **Failover:** the feed-handler pair publishes the same tick instances with **OWNERSHIP_STRENGTH 10 vs 5** — when the active dies, the standby automatically takes over the instances with zero client-side failover code (the §5.7 pattern). `VenueStatus` and `Heartbeat` go SHARED so any live node can report.
- **Late joiners:** the Market Data Cache re-publishes ticks with TRANSIENT durability so a joining trading app immediately gets the current last-value snapshot (the data-centric catch-up of §5.3).

### 10.4 The DDS-Security Deployment

All three built-in plugins (§6), enabled at the domain level:

- **Authentication — DDS:Auth:PKI-DH:** every participant receives an X.509 certificate from Cymbal Bank's internal PKI, and the issuing CA is configured on all peers. Feed handlers, gateways, the risk engine, trading apps and the ops dashboard each hold distinct identities — a stolen trading-app credential cannot impersonate a feed handler (§6.2).
- **Access Control — DDS:Access:Permissions** (Governance + Permissions documents, §6.3):
  - *Governance:* discovery and metadata must be protected; payloads on `OrderState` and `PositionUpdate` must be encrypted (AES-GCM); `MarketDataTick` requires integrity (AES-GMAC) — encryption on the highest-rate topic costs latency, so the governance says sign, don't encrypt, unless the feed itself is confidential.
  - *Permissions:* feed handlers may publish `MarketDataTick`/`VenueStatus`; order gateways may publish `OrderState`; trading apps may *subscribe* to their desks' partitions (`PROD/EQ/*`) and publish order *intents* but not `OrderState`; the risk engine may subscribe to everything and publish `PositionUpdate`; the compliance recorder is read-only on `OrderState` and may publish nothing. Default rule: deny.
- **Cryptographic — DDS:Crypto:AES-GCM:** keys derived per handshake from the PKI-DH secrets (§6.4); protected discovery means a rogue node cannot even enumerate the domain's topics (§6.5).
- **Operational notes:** certificates rotate on the bank's PKI cycle; the DR site runs the same governance so failover does not degrade the security posture; the Kafka audit sink (§10.5) receives order state *after* DDS-side authorization — defense in depth, not a bypass.

### 10.5 The Integration Boundary — DDS to FIX

- **Venue-facing sessions stay in FIX.** Feed handlers and order gateways own the FIX sessions to venues — sequence numbers, gap detection and resend (see the [FIX Protocol guide](../banking/fix_protocol_guide.md) §4.2–§4.4), and FIXP for the low-latency venues (§5.3 there). DDS never speaks FIX, and FIX never fans out internally.
- **Inbound (market data):** FIX (or venue-native) messages arrive at the feed handler → normalized into the IDL types (`MarketDataTick`, keyed by symbol) → published onto the DDS domain with the §10.3 QoS. One FIX session in, N consumers out — the fan-out problem FIX was never meant to solve.
- **Outbound (orders):** a trading app publishes an order intent on DDS → the OrderGateway (subscribed to the intent topic) validates, constructs a FIX `NewOrderSingle`, and sends it over the venue session → the venue's `ExecutionReport` returns over FIX → the gateway normalizes it into an `OrderState` update and publishes it on DDS → the risk engine and compliance see the state transition through the same bus as every other consumer.
- **Why the boundary is clean:** DDS's ownership/QoS model guarantees exactly one gateway owns each order instance (§10.3); FIX's session semantics guarantee ordered, gap-checked delivery *to the venue* — each protocol does what it is for, and translation is confined to the gateway processes.
- **The Kafka lane:** the compliance recorder subscribes on DDS and writes the Kafka audit/analytics log — replay, forensics and batch analytics live in Kafka; real-time distribution lives in DDS (§9.4; the [Kafka guide](../banking/kafka_guide.md) §10 covers the sibling event-platform design).

---

## 11. The Claims Audit — Verified, Flagged, Rejected

Every material claim in this guide, with its verdict. ✅ = verified this pass against the cited source; ⚠ = vendor claim / approximate / not re-verified; ❌ = rejected.

| Claim | Verdict | Source |
| --- | --- | --- |
| DDS 1.0 adopted December 2004 | ✅ | omg.org spec history |
| DDS 1.4 is the current formal revision (March 2015, formal/15-04-10) | ✅ | omg.org/spec/DDS/1.4 |
| DDSI-RTPS 2.5 is the current wire protocol (April 2022, formal/22-04-01) | ✅ | omg.org/spec/DDSI-RTPS/2.5 |
| DDS-XTypes 1.3 is the current type-system revision (February 2020, formal/20-02-04) | ✅ | omg.org/spec/DDS-XTypes/1.3 |
| DDS-SECURITY 1.2 is the current security revision (formal/25-03-06) | ✅ | omg.org/spec/DDS-SECURITY/1.2 |
| The DDS 1.4 catalogue contains the 22 QoS policies listed in §5.1 | ✅ | DDS 1.4 PDF (table of contents) |
| RTPS mandates SPDP + SEDP for interoperability | ✅ | RTPS 2.5 PDF |
| DDS-Security built-in plugins: DDS:Auth:PKI-DH, DDS:Access:Permissions, DDS:Crypto:AES-GCM | ✅ | omg.org DDS-SECURITY (spec structure; names as cited in §6) |
| DLRL was deprecated in DDS 1.2 (December 2006) | ⚠ | widely documented; 1.2 PDF not re-fetched this pass |
| "ROS 2 uses DDS as its default middleware" | ✅ | docs.ros.org (Jazzy) |
| Fast DDS is the default RMW in ROS 2 LTS releases | ✅ | fast-dds.docs.eprosima.com + Open Robotics quote |
| ROS 2 supports Fast DDS, Connext, Cyclone DDS, GurumDDS and the non-DDS Zenoh RMW | ✅ | docs.ros.org (Jazzy) |
| Cyclone DDS is EPL-2.0 open source, Eclipse-hosted | ✅ | GitHub LICENSE / projects.eclipse.org |
| ZettaScale spun out of ADLINK (2022) and stewards Cyclone DDS, Zenoh and OpenSplice | ✅ | adlinktech.com press release |
| Fast DDS is Apache-2.0, by eProsima (Tres Cantos, Madrid) | ✅ | eprosima.com |
| OpenDDS is open source by OCI; permissive license; C++ with Java via JNI; 3.34.0 (May 2026) | ✅ | opendds.org |
| GurumDDS is a commercial product of GurumNetworks (South Korea) | ✅ | gurumdds.com |
| GurumDDS conformance: DCPS 1.4, RTPS 2.2 | ⚠ | vendor claim; RTPS 2.2 predates the current 2.5 |
| Zenoh: pub/sub/query; peer-to-peer/brokered/hybrid topology; 4–6-byte overhead; ~300-byte MCU footprint | ✅ | zenoh.io |
| Zenoh was designed by Angelo Corsaro (OMG DDS co-chair) | ✅ | zenoh.io |
| AMQP 1.0 is an OASIS standard, 30 October 2012 | ✅ | docs.oasis-open.org (established) |
| MQTT is an OASIS standard (3.1.1 also ISO; 5.0 current) | ✅ | mqtt.org |
| MQTT 3.1.1 approved October 2014; MQTT 5.0 March 2019 | ⚠ | widely documented; exact dates not re-fetched this pass |
| OPC UA = OPC Foundation, IEC 62541 | ✅ | sibling scada_guide.md §5.3 (verified there) |
| Kafka originated at LinkedIn (2011) as a distributed log | ✅ | sibling kafka_guide.md §1.3 (verified there) |
| RTI Connext: commercial suite (Professional/Micro/Cert/Drive/TSS); CMMC Level 2 (June 2026) | ✅ | rti.com |
| RTI Connext: "over 2,000 designs" | ⚠ | vendor marketing claim |
| eProsima: "most adopted open-source DDS", 50K downloads/month, Safe DDS ISO 26262 ASIL D | ⚠ | vendor marketing claims |
| GurumNetworks: "100+ critical distributed systems" | ⚠ | vendor marketing claim |
| "A formal DDS 1.3 was published" | ❌ | no formal 1.3 appears in the omg.org version history (§2.2) |
| "DDS requires a broker" | ❌ | brokerless by design — RTPS is peer-to-peer (§1.5, §4) |
| "DDS-Security removes the need for PKI and certificate operations" | ❌ | the plugins presuppose certificate management (§6, §10.4) |
| RTI founded 1991, HQ Sunnyvale CA | ⚠ | company history well documented; not re-verified this pass |

## 12. What Could Not Be Verified

- **Market shares.** No independent, audited market-share study of DDS implementations exists; all vendor scale claims (§7.7) are self-reported and remain ⚠.
- **Vendor performance numbers.** Microsecond-latency and throughput figures in vendor benchmarks are self-published and hardware-dependent; this guide deliberately quotes none as fact (§1.2 keeps them ⚠-knowledge).
- **The DLRL deprecation wording.** The exact deprecation language in the DDS 1.2 PDF was not re-fetched this pass (§2.4 — marked ⚠).
- **Named deployments.** No specific bank/financial DDS deployments were verified, so §8.7 cites none; vendor customer lists (Aptiv, XPeng, Medtronic, Stryker…) are marketing ⚠.
- **GurumDDS's current conformance level.** The vendor claims RTPS 2.2; whether the product tracks RTPS 2.5 (the current wire protocol, §4.1) is unverified ⚠.
- **MQTT approval dates** (3.1.1 October 2014, 5.0 March 2019) — widely documented but not re-fetched this pass ⚠.
- **Zenoh's license text** — the EPL-2.0/Apache-2.0 dual licensing is standard knowledge ⚠-knowledge; the LICENSE file was not re-read this pass.
- **Future revisions** DDS15 / DDSIRTP26 (§2.5) — tracker signals only; neither is formal as of this pass ⚠.
- **RTI's founding details** (1991, Sunnyvale) — well-documented company history, not re-verified this pass ⚠.
- **The OPC UA PubSub-over-DDS transport mapping** (§8.4) — documented in the OPC UA PubSub specification but not re-fetched this pass ⚠-knowledge.

## 13. The Glossary

| Term | Meaning |
| --- | --- |
| **DDS** | OMG Data Distribution Service — the data-centric, brokerless publish-subscribe middleware standard (§1) |
| **DCPS** | Data-Centric Publish-Subscribe — the core DDS model: API + communication semantics (§1.4) |
| **DLRL** | Data Local Reconstruction Layer — the optional object-cache layer, deprecated in 1.2, absent from 1.4 as an active clause (§2.4) |
| **Domain** | A virtual communication boundary identified by a DomainId; only same-domain entities interoperate (§3.1) |
| **DomainParticipant** | The entry-point entity of a domain; the security principal when DDS-Security is on (§3.1, §6.1) |
| **Topic** | A named, typed data stream — the rendezvous point of publish-subscribe (§3.2) |
| **Instance / Key** | One state object within a topic, identified by the type's key fields (§3.2) |
| **Sample** | One data value written for an instance, with its SampleInfo (§3.2) |
| **DataWriter / DataReader** | The producer / consumer endpoints (§3.3) |
| **Partition** | A string-tagged subdivision of a domain (§3.4) |
| **OMG IDL** | The Interface Definition Language used to define DDS types (§3.5) |
| **XTypes** | Extensible and Dynamic Topic Types — the type system and type-evolution rules (§3.5) |
| **CDR** | Common Data Representation — the OMG serialization format used on the wire (§3.5) |
| **RTPS (DDSI-RTPS)** | The DDS Interoperability Wire Protocol (§4.1) |
| **SPDP / SEDP** | Simple Participant / Simple Endpoint Discovery Protocols (§4.2, §4.3) |
| **QoS policy** | A contractual parameter of a data stream — offered by writers, requested by readers (§5.1) |
| **BEST_EFFORT / RELIABLE** | Reliability kinds: no retransmission vs guaranteed delivery (§5.2) |
| **Durability kinds** | VOLATILE → TRANSIENT_LOCAL → TRANSIENT → PERSISTENT: what late joiners receive (§5.3) |
| **DEADLINE / LIVELINESS** | Timing contracts: maximum inter-sample time; peer-alive detection (§5.5, §5.6) |
| **OWNERSHIP / OWNERSHIP_STRENGTH** | Multi-writer control: SHARED vs EXCLUSIVE with strength-based takeover (§5.7) |
| **DDS-Security** | The OMG security specification: the SPI plus three built-in plugins (§6) |
| **PKI-DH** | DDS:Auth:PKI-DH — certificate-based mutual authentication + DH key agreement (§6.2) |
| **Governance / Permissions** | The two XML documents of the access-control plugin (§6.3) |
| **AES-GCM / AES-GMAC** | The cryptographic plugin's authenticated-encryption / integrity-only algorithms (§6.4) |
| **RMW** | ROS middleware — the ROS 2 abstraction over DDS implementations (§8.1) |
| **FACE** | The Open Group's Future Airborne Capability Environment; mandates DDS for avionics data transport (§8.3, §8.5) |
| **Brokerless** | Peer-to-peer architecture with no central broker — no single point of failure (§1.5) |
| **OPC UA** | IEC 62541 — the OPC Foundation's industrial communication framework (§8.4; the SCADA guide §5.3) |
| **MQTT** | The lightweight broker-centric IoT pub/sub protocol (OASIS) (§9.2) |
| **AMQP 1.0** | The OASIS enterprise messaging wire protocol (§9.3) |
| **Kafka** | The distributed log / event-streaming platform (Apache) (§9.4) |
| **Zenoh** | The Eclipse pub/sub/query protocol — brokerless-capable, cloud-to-microcontroller (§9.5) |

## 14. Cross-References and Further Reading

### Sibling guides in this repository

- [SCADA, ICS and OT Security](scada_guide.md) — §5.3 OPC UA; the OT security context for §8.4.
- [Apache Kafka Guide](../banking/kafka_guide.md) — §2 architecture, §3 replication/ISR, §10 worked example: the analytics lane of §9.4 and §10.5.
- [FIX Protocol](../banking/fix_protocol_guide.md) — §4 session layer, §5.3 FIXP: the execution boundary of §10.5.
- [Kafka Alternatives](kafka_alternatives_guide.md) — where each bus/queue/stream pattern fits (§9).
- [Event Stream Processing](event_stream_processing_guide.md) — the stream-processing paradigm adjacent to the pub-sub bus (§9).
- [Complex Event Processing](complex_event_processing_guide.md) — the CEP layer that typically sits on top of a DDS bus (§8.7, §10).
- [Zero Downtime System Design](zero_downtime_system_design_guide.md) — the availability discipline for the trading-floor estate (§10).

### Primary sources and official sites

- OMG DDS 1.4 — https://www.omg.org/spec/DDS/1.4 ✅
- OMG DDSI-RTPS 2.5 — https://www.omg.org/spec/DDSI-RTPS/2.5 ✅
- OMG DDS-XTypes 1.3 — https://www.omg.org/spec/DDS-XTypes/1.3 ✅
- OMG DDS-SECURITY 1.2 — https://www.omg.org/spec/DDS-SECURITY/1.2 ✅
- DDS Foundation — https://www.dds-foundation.org ⚠-knowledge
- ROS 2 RMW implementations — https://docs.ros.org/en/jazzy/Installation/RMW-Implementations.html ✅
- RTI Connext — https://www.rti.com ✅
- Eclipse Cyclone DDS — https://cyclonedds.io / https://github.com/eclipse-cyclonedds/cyclonedds ✅
- eProsima Fast DDS — https://www.eprosima.com / https://github.com/eProsima/Fast-DDS ✅
- OpenDDS — https://opendds.org ✅
- GurumDDS — https://www.gurumdds.com ✅
- Eclipse Zenoh — https://zenoh.io ✅
- OASIS MQTT — https://mqtt.org / https://docs.oasis-open.org/mqtt/ ✅
- OASIS AMQP 1.0 — https://docs.oasis-open.org/amqp/core/v1.0/os/amqp-core-complete-v1.0-os.html ✅
- Apache Kafka — https://kafka.apache.org (see the [Apache Kafka Guide](../banking/kafka_guide.md) for the deep dive)

### Closing

DDS is not the newest protocol in this guide, and it is not the simplest — it is the one that treats *data* as the thing being distributed and *predictability* as the contract. Where a message queue routes messages through a broker, DDS lets every node talk directly to every peer, with automatic discovery, typed models, and a QoS system that turns "is the feed alive?" into a measured signal and "which writer owns this state?" into an enforced policy. For the trading floor of §10, for the robot fleets of §8, for the defense platforms that gave DDS its birth — the architecture question keeps coming back to the same answer: the data-centric bus.
