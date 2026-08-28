# Enterprise Middleware and Integration: The Integration Fabric — A Comprehensive Guide

**Jack Liu Shurui, Solution Architect**

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** Integration / Middleware / Platform (technology/)
> **Audience:** Solution architects, integration architects, platform engineers, technical leads rationalizing enterprise integration estates
> **Last Updated:** August 2026

*A comprehensive guide to the full integration landscape in one document: the middleware history and taxonomy (CORBA and the application servers → message-oriented middleware → ESB/SOA → the modern iPaaS/API/event-driven/cloud-native era), the integration categories and their taxonomy table, the vendor landscape (verified facts, analyst estimates flagged), the platform layer (Kubernetes/OpenShift, integration mesh, composable integration, the mainframe boundary — CICS and MQ on z/OS), the banking angle (payments, core-banking interfaces, the SWIFT estate, the trading floor), and a Cymbal Bank worked example — a full integration-estate rationalization from a legacy MQ + TIBCO + webMethods estate onto a modern fabric of Kafka, an API gateway, and iPaaS, with a decision matrix, a phased migration, and a risk register.*

**Cross-references:** [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) (the integration **patterns** and styles — the design vocabulary this guide's categories operationalize), [data_integration_frameworks_guide.md](data_integration_frameworks_guide.md) (the **frameworks** — CDC/ETL/API tooling), [camel_camelk_research.md](camel_camelk_research.md) (Apache Camel — the reference EIP implementation and Red Hat Fuse lineage), [event_stream_processing_guide.md](event_stream_processing_guide.md) (the **streaming** discipline on the event backbone), [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) (messaging **reliability** — delivery semantics, the transactional outbox), [kafka_alternatives_guide.md](kafka_alternatives_guide.md) (the **broker comparison** — Kafka vs RabbitMQ vs Pulsar vs MQ), [dds_guide.md](dds_guide.md) (the **brokerless** comparison — DDS vs MQ/AMQP on the trading floor), [ibm_cloud_guide.md](ibm_cloud_guide.md) (the **Red Hat integration** — OpenShift, Cloud Paks, MQ/Event Streams, the hybrid cloud), [ai_platform_engineering_guide.md](ai_platform_engineering_guide.md) (the **platform-engineering** operating model the integration platform team uses), [../banking/payment_rails_guide.md](../banking/payment_rails_guide.md) (the **rails** the payments estate integrates with), [../banking/swiftnet_fileact_guide.md](../banking/swiftnet_fileact_guide.md) and [../banking/swift_alliance_access_guide.md](../banking/swift_alliance_access_guide.md) (the **SWIFT estate** — FIN/InterAct/FileAct, Alliance Access/Gateway), [../banking/kafka_guide.md](../banking/kafka_guide.md) (Kafka as the **event backbone** — the Cymbal Bank event-platform design), [../banking/fix_protocol_guide.md](../banking/fix_protocol_guide.md) (the **trading-floor** execution language), [../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md) (the **cores** behind the interfaces), [../banking/payments_hub_guide.md](../banking/payments_hub_guide.md) (the payments-hub patterns), [../banking/capital_markets_architecture_guide.md](../banking/capital_markets_architecture_guide.md) (the post-trade estate).

**How to read this guide.** §1 is the overview — the one-paragraph answer and the key-facts table. §2 is the history — the four eras of middleware with verified dates. §3 is the taxonomy — what each integration category *is* and the category table. §4 is the vendor landscape — who owns what, with verified acquisition/founding facts and analyst estimates flagged. §5 is the platform layer — Kubernetes/OpenShift, integration mesh, composable integration, and the mainframe boundary. §6 is the banking angle — where integration meets payments, core banking, SWIFT, and the trading floor. §7 is the worked example — a Cymbal Bank integration-estate rationalization with a decision matrix, a phased migration, and a risk register. §8 is the summary. §9 is the claims audit (✅ verified / ⚠ flagged), §10 is "What Could Not Be Verified", §11 is the glossary, §12 is the cross-reference map. Cross-reference convention: sibling guides in `technology/` are plain filenames; guides in `banking/` are prefixed `../banking/`. **Integrity convention:** ✅ = verified this pass against a primary or cited source (source given in §9); ⚠ = flagged / unverified / analyst estimate; ⚠-knowledge = well-documented industry knowledge not re-verified live this pass. No dates, prices, or founding years were invented.

---

## Table of Contents

1. [The Overview: The Integration Fabric](#1-the-overview-the-integration-fabric)
2. [The History: From CORBA to Cloud-Native](#2-the-history-from-corba-to-cloud-native)
3. [The Taxonomy: The Integration Categories](#3-the-taxonomy-the-integration-categories)
4. [The Vendor Landscape](#4-the-vendor-landscape)
5. [The Platform Layer](#5-the-platform-layer)
6. [The Banking Angle](#6-the-banking-angle)
7. [The Worked Example: Cymbal Bank's Integration-Estate Rationalization](#7-the-worked-example-cymbal-banks-integration-estate-rationalization)
8. [The Summary: One Fabric, Not One Product](#8-the-summary-one-fabric-not-one-product)
9. [The Claims Audit](#9-the-claims-audit)
10. [What Could Not Be Verified](#10-what-could-not-be-verified)
11. [Glossary](#11-glossary)
12. [Cross-References and Further Reading](#12-cross-references-and-further-reading)

---

## 1. The Overview: The Integration Fabric

### 1.1 The Short Answer

Every large enterprise runs an **integration estate** — the accumulated middleware, adapters, queues, buses, gateways, and hand-rolled bridges that make its systems talk to each other. That estate has a history, a vocabulary, and a vendor map, and it is currently crossing a generational line: from the **ESB/SOA era** (a central bus mediating everything) to the **fabric era** (a layered set of specialized planes — message broker for reliable transport, event backbone for streaming facts, API gateway for request/reply, iPaaS for SaaS connectivity, BPM/workflow for long-running processes — running on Kubernetes, wrapping the mainframe rather than replacing it).

This guide names all of those layers, verifies the vendor facts behind them, and ends with a worked example: a Cymbal Bank rationalizing a legacy estate of IBM MQ point-to-point links, a TIBCO ESB, and webMethods integration stacks — across payments, core banking, and SWIFT connectivity — onto a modern fabric of Kafka (event backbone) + API gateway + iPaaS.

The one-paragraph thesis: **integration middleware is not a product category that gets replaced once; it is a fabric that accumulates.** CORBA gave way to application servers, which coexisted with message queues, which were joined by ESBs, which are now being joined (not always replaced) by API gateways, iPaaS, and event backbones. The architect's job is to know which layer does what, which vendors sit on which layer, and how to consolidate a legacy estate onto the modern fabric without breaking the business that runs on the old one.

### 1.2 The Key-Facts Table

| Fact | Value | Status |
|------|-------|--------|
| CORBA 1.0 published | October 1991, by the Object Management Group (OMG, formed 1989) | ✅ §2.1 |
| WebLogic, Inc. founded | September 1995; acquired by BEA Systems September 1998; BEA acquired by Oracle 29 April 2008 (US$8.5B) | ✅ §2.1 |
| IBM WebSphere Application Server 1.0 | 22 June 1998 (beta codename "Servlet Express") | ✅ §2.1 |
| IBM MQ (MQSeries) launched | December 1993; renamed WebSphere MQ 2002; IBM MQ 2014; native z/OS support throughout | ✅ §2.2 |
| JMS standardized | Java Community Process, JSR 914; 1.0.2b maintenance release June 2001 | ✅ §2.2 |
| "iPaaS" term coined | Gartner, 7 March 2011 (Pezzini & Lheureux, G00210747) | ✅ §2.4 (sibling ledger) |
| Apache Camel 1.0 | 27 June 2007 (James Strachan; first commit March 2007) | ✅ §2.3 |
| Kubernetes first release | 9 September 2014 (Google); CNCF (founded 2015) its home since 2016 | ✅ §5.1 |
| OpenShift announced | May 2011 (Red Hat); Kubernetes-based since v3, June 2015 | ✅ §5.1 |
| CICS first product release | 8 July 1969 (IBM; developed from 1966 with Michigan Bell) | ✅ §5.4 |
| MuleSoft acquired by Salesforce | May 2018, ~US$6.5B | ✅ §4.2 |
| Apigee acquired by Google | Announced 8 September 2016, US$625M, completed November 2016 | ✅ §4.2 |
| Boomi acquired from Dell | Francisco Partners + TPG Capital, announced 2 May 2021, US$4B | ✅ §4.2 |
| Informatica taken private | Permira + CPP Investments, ~US$5.3B, announced April 2015 | ✅ §4.2 |

### 1.3 What "Middleware" Means Here

Middleware is the software layer *between* applications: transport (queues, brokers), mediation (routing, transformation, orchestration), and exposure (APIs). The integration fabric is the modern, layered form of it — see the taxonomy in §3. The four eras of its history are the subject of §2.

---
### 1.4 The Generational Scorecard

Each era left the estate something it still uses and something it still pays for. The scorecard is the rationalization's starting point (it becomes §7's inventory):

| Era | What it contributed | What it now costs to keep | The 2026 verdict |
|-----|---------------------|---------------------------|------------------|
| Distributed objects (CORBA) | IDL contract discipline; the ORB idea | Orphaned ORBs inside vendor products; hard-to-staff skills | Retire where found; keep the contract discipline |
| Application servers (WebLogic/WebSphere) | Managed runtime, J2EE contract | Heavy JVM estates, license + tuning cost, slow deploys | Containerize or re-platform; the contract survives as Jakarta EE |
| MOM (MQ/JMS) | Guaranteed, transactional messaging | Point-to-point sprawl; per-link ownership debt | Keep the *capability*; cut the *links* (govern, don't delete) |
| ESB/SOA (TIBCO/webMethods/BizTalk) | Canonical model, mediation, BPM | Central bottleneck; license + dual-stack cost; skills drain | Retire the product; keep the canonical model as the schema registry |
| iPaaS (2011–) | SaaS connectivity, low-code | Vendor lock-in; flow sprawl | The default for SaaS; govern like code |
| Event backbone (Kafka) | Durable, replayable facts | Operational complexity; 3× storage cost | The system of record for events; run it like a bank platform |
| Cloud-native (Kubernetes/OpenShift) | One substrate for everything | Cluster sprawl; platform-team cost | The plane everything else sits on |

---

## 2. The History: From CORBA to Cloud-Native

### 2.1 The CORBA and Application-Server Era (1991–2001)

The first attempt to industrialize distributed computing was the **distributed object** wave. The **Object Management Group** (OMG) — a standards consortium **formed in 1989** ✅ (omg.org) — published **CORBA 1.0 (Common Object Request Broker Architecture) in October 1991** ✅ (Wikipedia). CORBA let objects call each other across languages and machines via an IDL (interface definition language) and an Object Request Broker (ORB); its contemporaries were Microsoft's DCOM and Sun's Java RMI. CORBA's ambition was real-time, location-transparent, language-neutral RPC; its fate was complexity, vendor interoperability pain, and firewalls — "briefly popular in the mid-to-late 1990s" before being relegated to niche status ✅ (Wikipedia). In finance, CORBA ORBs (IONA Orbix, VisiBroker) powered early trading and payments systems; many survived inside vendor products for decades.

The distributed-object idea matured into the **application server**: a managed runtime that hosted business logic and talked to databases, transactions, and the web. Two lineages dominate the enterprise and both are verified:

- **WebLogic** — WebLogic, Inc. was founded in **September 1995** in San Francisco (Paul Ambrose, Bob Pasker, Laurie Pitman, Carl Resnikoff) and built the first standards-based Java application server; **BEA Systems** (founded 1995 by Bill Coleman, Ed Scott, Alfred Chuang — named from their initials — starting from the Tuxedo transaction monitor) **acquired WebLogic in 1998**; **Oracle agreed to acquire BEA for US$8.5 billion in January 2008, completing on 29 April 2008** ✅ (Wikipedia: WebLogic, Inc.; BEA Systems). WebLogic Server is today Oracle WebLogic Server — still the Java EE workhorse of countless banks.
- **WebSphere** — IBM's WebSphere Application Server **1.0 was released 22 June 1998**; the early betas were codenamed **"Servlet Express"**, an offshoot of IBM's HTTP server team (the Domino Go web server); it was created by Donald F. Ferguson ✅ (Wikipedia). WAS became the flagship of IBM's Java middleware and, with WebSphere MQ (2002), anchored IBM's integration story for two decades.

The application-server era also produced **Tuxedo** (BEA's transaction-processing middleware, now Oracle Tuxedo), the J2EE specification (whose impetus, per the BEA history, was WebLogic itself ✅), and the notion of a *managed container* that later mutated into today's Kubernetes pod.

Two technical details matter for understanding what the era left behind. First, CORBA's wire protocol — **GIOP/IIOP** (General Inter-ORB Protocol / Internet Inter-ORB Protocol) — was the first serious attempt at a standardized binary RPC protocol, and its IDL-based contract discipline directly inspired the WSDL/interface-first thinking of SOAP and, much later, the schema-first discipline of the event backbone's Schema Registry (cross-ref [../banking/kafka_guide.md](../banking/kafka_guide.md) §6). Second, J2EE (Java 2 Platform, Enterprise Edition, 1999) codified the application-server contract — EJBs, JNDI, JDBC, JMS — so that WebLogic, WebSphere, and JBoss could compete on implementation rather than on the programming model; that "specify the contract, let vendors compete" move is the same one the CNCF later applied to containers with Kubernetes. The era's hard lesson, learned by every bank that ran a CORBA ORB: **a rich distributed-object standard can still fail on interoperability and complexity** — CORBA's vendor-interoperability problems and firewall-unfriendly protocols are why its successors (SOAP first, then REST) went simpler.

### 2.2 The MOM Era (1993–2001): The Message Is the Integration

Parallel to RPC, the other grand integration style — **messaging** — became a product category. The defining product:

- **IBM MQ** — launched as **MQSeries in December 1993** ✅ (Wikipedia), IBM's message-oriented middleware (MOM) family. It was **renamed WebSphere MQ in 2002** (joining the WebSphere suite) and **renamed IBM MQ in April 2014** ✅. IBM MQ delivers guaranteed, once-and-only-once point-to-point and publish-subscribe messaging with queue managers, channels, and transactional integrity, and it runs natively on **z/OS** (IBM MQ for z/OS is an explicit member of the family ✅) — which is why it became the de facto transport between mainframe cores and everything else. In banking, MQ is the backbone of core-to-channel, core-to-payments, and core-to-SWIFT flows — see §5.4 and §6.

The API that made MOM portable across vendors:

- **JMS (Java Message Service)** — designed by Sun Microsystems with industry partners under the **Java Community Process as JSR 914** ✅ (JCP; Oracle). The **1.0.2b maintenance release incorporated errata approved 25 June 2001** ✅. JMS gave Java a vendor-neutral API for queues and topics, letting MQ, SonicMQ, ActiveMQ, and the broker generation compete on the same programming model.

JMS also fixed the two messaging shapes that every later system still uses: **point-to-point** (a message goes to exactly one consumer from a queue — the command/acknowledgement pattern) and **publish-subscribe** (a message fans out to every subscriber of a topic — the event pattern). On top of the shapes sit the delivery semantics that make messaging *reliable*: persistent messages survive broker restart, transactions group sends/receives atomically, and acknowledgements (auto, client, or dups-ok) define when the broker may forget a message. IBM MQ's marketing-grade "once and only once" is the strongest form (transactional queue get/put), while most modern brokers offer at-least-once with idempotent consumers as the practical contract — the exact spectrum [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) analyzes in depth. The architect's rule from the MOM era that still holds: **choose the delivery semantics before you choose the broker, because the broker is only as good as the contract you configure.**

The MOM era also saw the broker generation: **Sonic Software** (the SonicMQ broker, Bedford, MA — founded ~2000 ⚠-knowledge, see §10) became a **Progress Software operating company in the early 2000s** (SEC filings show it as "an independent operating company of Progress Software Corporation" by October 2002 ✅), and **Apache ActiveMQ** (2004) and **RabbitMQ** (2007) opened the open-source side. AMQP (the Advanced Message Queuing Protocol, OASIS) was the era's attempt at a standard wire protocol; IBM MQ supports AMQP 1.0 as an optional protocol today ✅ (Wikipedia, IBM MQ).

### 2.3 The ESB/SOA Era (2000s): The Bus

The 2000s industrialized integration as *service orientation*: applications expose **services**; a central **enterprise service bus (ESB)** mediates between them — routing, transforming, orchestrating, and translating protocols (the term and the architecture were codified in David A. Chappell's *Enterprise Service Bus: Theory in Practice*, O'Reilly, 2004 ✅ — sibling ledger). The ESB was the hub of the **SOA** world, and its vendor wave is the one most banks still have in production:

- **webMethods** — founded **1996** ✅ (Wikipedia), the Fairfax, VA integration vendor (EAI/B2B integration over web services; famously +500% on its 2000 IPO day). **Software AG agreed to acquire webMethods for US$546 million in cash, announced 5 April 2007** ✅ (Reuters; CNBC; NYT; Software AG press release via SEC).
- **TIBCO** — the "Information Bus Company": the lineage begins with **Teknekron Software Systems, founded in 1985** by Vivek Ranadivé with seed capital from Teknekron Corp. ✅ (Wikipedia, Ranadivé bio) — the origin of the "founded 1985" claim — while **TIBCO Software Inc. itself was founded in 1997** as a Reuters-backed subsidiary (Reuters had acquired Teknekron's information-bus software for US$125M in 1994) ✅ (Wikipedia, TIBCO Software). TIBCO's TIB/Rendezvous and EMS messaging, BusinessWorks ESB, and CEP (Complex Event Processing) products became the real-time backbone of trading floors and payments. **TIBCO was acquired by Vista Equity Partners for US$4.2 billion in December 2014** ✅; in **September 2022, Vista and Elliott Investment Management acquired Citrix Systems for US$16.5 billion and merged it with TIBCO to form Cloud Software Group** ✅.
- **Mule** — the open-source ESB (and EIP implementation) created by Ross Mason in **2003**; **MuleSource, the company, was founded in 2006** by Mason and Dave Rosenberg and **renamed MuleSoft in 2009** ✅ (Wikipedia). Mule ESB + Anypoint Platform became the archetype of "ESB, then iPaaS" (§4.2).
- **Sonic ESB** — Progress Software's SOA line built on SonicMQ (the 2007 reorganization put Sonic into Progress's Enterprise Infrastructure Division targeting SOA ✅-press, techmonitor).
- **Microsoft BizTalk Server** — first released **19 December 2000** (BizTalk Server 2000) ✅ (Wikipedia); the .NET EAI/BPM server with adapters, maps (XSLT), and orchestrations; **BizTalk Server 2020 is the final release** — Microsoft confirmed in early 2026 that BizTalk Server 2020 is the last version, with **Azure Logic Apps (Azure Integration Services) as the strategic cloud successor** and a documented migration runway to **April 2030** ✅ (Microsoft Learn; Microsoft Tech Community).
- **Oracle Service Bus / Oracle SOA Suite** — the AquaLogic lineage from BEA (the third BEA product line, now Oracle Service Bus ✅ (Wikipedia, BEA Systems)).
- **Apache Camel** — created by James Strachan, **first commit 19 March 2007, version 1.0 on 27 June 2007** ✅ (Wikipedia); the embeddable Java framework implementing the Enterprise Integration Patterns with 350+ components. Its commercial path defines the era's open-source→vendor arc: **FuseSource (2010) → acquired by Progress Software (2011) → acquired by Red Hat (2012)**, productized as **Red Hat Fuse**, now the "Red Hat build of Apache Camel" ✅ (Wikipedia; cross-ref [ibm_cloud_guide.md](ibm_cloud_guide.md)).

The ESB era's lesson, learned expensively: **a single central bus becomes a bottleneck and a single point of change** — the "hub" pattern that the industry has since decomposed into the layered fabric of §3.

Two more era-defining artifacts belong in the record. The **protocol stack**: SOAP (XML envelopes over HTTP), WSDL (interface contracts), UDDI (service discovery), and the WS-* alphabet (WS-Security, WS-ReliableMessaging, WS-Transaction) gave the ESB its lingua franca; REST/JSON arrived as the deliberate simplification and won the developer mindshare. And the **pattern vocabulary**: *Enterprise Integration Patterns* (Hohpe & Woolf, Addison-Wesley, **2003** — verified in the sibling ledger) named the channel/router/translator/endpoint catalogue that ESBs implemented and that Apache Camel still implements today (cross-ref [camel_camelk_research.md](camel_camelk_research.md)); the same era's **orchestration vs choreography** debate — a central process calling services (orchestration, the ESB/BPM way) versus services coordinating through events with no central conductor (choreography, the event-backbone way) — is still the live design tension in §7's decision matrix.

### 2.4 The Modern Era (2011–): iPaaS, APIs, Events, Cloud-Native

Four forces reshaped the landscape after 2011:

1. **iPaaS** — Gartner coined "Integration Platform as a Service" in a research note of **7 March 2011** (Pezzini & Lheureux, G00210747) ✅ (sibling ledger). Cloud-delivered integration with connectors, low-code flows, and no broker to run: the category's flagship vendors are Boomi, MuleSoft, Workato, and the hyperscaler suites (Azure Integration Services, AWS Application Integration, Google Application Integration).
2. **API management** — the request/reply world got a governance layer: **Apigee** (founded July 2004 as Sonoa Systems, rebranded Apigee 2010; **acquired by Google for US$625M, announced 8 September 2016, completed November 2016** ✅), **Kong** (born 2009 as Mashape; the gateway was open-sourced and named Kong in 2017 ✅ — §4.2), **WSO2** (founded 4 August 2005 by Sanjiva Weerawarana, Paul Fremantle, Davanum Srinivas ✅ — §4.2), and the hyperscaler gateways (AWS API Gateway, Azure API Management).
3. **Event-driven architecture and the event backbone** — Apache Kafka (born at LinkedIn in 2011, open-sourced the same year, Apache top-level 2012, Confluent founded by its creators 2014 — cross-ref [../banking/kafka_guide.md](../banking/kafka_guide.md)) made the **durable, replayable log** the system of record for *what happened*; the event backbone became the modern pattern for payments, trade lifecycle, and regulatory data.
4. **Cloud-native platforms** — **Kubernetes** (first release 9 September 2014 ✅, §5.1) gave middleware a common deployment substrate, so brokers, gateways, and integration runtimes became *operators on a cluster* instead of installed servers — the platform layer of §5.

The modern era is not a clean break: most banks run all four eras simultaneously — a 1993 MQ queue next to a 2007 ESB flow next to a 2017 Kafka topic next to a 2024 iPaaS recipe. That coexistence is the *integration fabric*, and §7's worked example is about rationalizing it.

Three modern-era currents deserve explicit names. **Microservices** (the 2014+ reaction to the monolith) moved integration from "connect the apps" to "connect the services inside the app" — which is why the API gateway and the service mesh (§5.2) now sit *inside* the estate rather than only at its edge. The **API economy** reframed the API from a technical interface to a product with consumers, SLAs, and monetization — the mindset behind API-led connectivity (MuleSoft's framing ⚠-methodology) and behind banks exposing open-banking APIs. And the **hyperscaler integration suites** (Azure Integration Services — Logic Apps, Service Bus, API Management; AWS Application Integration — API Gateway, SQS/SNS, EventBridge, MSK; Google — Apigee, Pub/Sub, Application Integration) made the fabric a *buyable cloud service* rather than an installed estate: for greenfield workloads the question is no longer "which ESB" but "which cloud's integration plane." All three currents show up in the §7 target architecture, which mixes the hyperscaler suites, Kafka, an API gateway, and iPaaS by workload class.

### 2.5 The Lineage Timeline

| Era | Years | Defining technology | Verified anchors |
|-----|-------|---------------------|------------------|
| Distributed objects | 1991–2001 | CORBA (OMG), DCOM, Java RMI | CORBA 1.0 October 1991 ✅; OMG formed 1989 ✅ |
| Application servers | 1995–2010 | WebLogic, WebSphere, J2EE | WebLogic, Inc. September 1995 ✅; WAS 1.0 22 June 1998 ✅; BEA→Oracle 29 April 2008 ✅ |
| MOM / messaging | 1993– | IBM MQ, JMS, SonicMQ, RabbitMQ | MQSeries December 1993 ✅; JSR 914 (JMS) 2001 ✅ |
| ESB / SOA | 2000s | webMethods, TIBCO, Mule ESB, BizTalk, Sonic | webMethods 1996 → Software AG US$546M April 2007 ✅; TIBCO: Teknekron 1985 / TIBCO Inc. 1997 / Vista 2014 / Cloud Software Group 2022 ✅; MuleSource 2006 ✅; BizTalk 19 December 2000 ✅ |
| iPaaS | 2011– | Boomi, MuleSoft, Workato, Azure Integration Services | Gartner term 7 March 2011 ✅ (sibling ledger); Boomi 2000 / Dell 2010 / TPG+FP 2021 ✅ |
| API management | 2004– | Apigee, Kong, WSO2, AWS/Azure gateways | Apigee 2004 / Google US$625M 2016 ✅; Kong 2009→2017 ✅; WSO2 August 2005 ✅ |
| Event backbone | 2011– | Kafka, Pulsar, Event Streams | Kafka 2011 LinkedIn origin ✅ (sibling ledger) |
| Cloud-native platform | 2014– | Kubernetes, OpenShift, service mesh | Kubernetes 9 September 2014 ✅; OpenShift May 2011 / Kubernetes-based v3 2015 ✅ |

### 2.6 What Each Generation Shift Actually Was

Reading the timeline as a series of *shifts* rather than a list of products makes the landscape legible:

- **1991 → 1998 (objects → containers):** the shift from *standards-defined protocols* (CORBA/IIOP) to *standards-defined runtimes* (J2EE application servers) — code moved from "talk to anything" to "run anywhere."
- **1993 → 2001 (RPC → messages):** the shift from *call* to *send* — the MOM era proved that asynchronous, guaranteed messaging decouples systems in time and failure, not just in space (the property every later backbone inherits).
- **2000s (hub → edges):** the ESB era centralized mediation, then the industry spent a decade decentralizing it — the shift from *the bus* to *the fabric*, completed by Kubernetes (cheap runtime anywhere) and the schema registry (central contract without central processing).
- **2011 → 2016 (installed → cloud):** iPaaS and the hyperscaler suites shifted integration from *software you run* to *services you consume*; the cloud-native platform layer (2014–) then shifted even installed middleware onto the same substrate.
- **2011 → (commands → facts):** the event backbone shifted the default unit of integration from *instructions* ("do this") to *facts* ("this happened") — the single most consequential change for banking, because facts are replayable, auditable, and fan-out by nature (cross-ref [../banking/kafka_guide.md](../banking/kafka_guide.md) §9).

The pattern across all five shifts: **each generation kept the previous generation's best idea and relocated it** — contracts moved from ORBs to WSDL to schemas; guaranteed delivery stayed in MQ while facts moved to Kafka; mediation moved from the hub to the edges. That is why the modern estate is a fabric of coexisting generations, and why §7 rationalizes *ownership of flows*, not *elimination of eras*.

---
## 3. The Taxonomy: The Integration Categories

### 3.1 The Category Table

The integration landscape is six overlapping categories plus the platforms they run on. Each answers a different question; the taxonomy table is the architect's cheat-sheet.

| Category | What it is | Synchronous? | Canonical examples | Strengths | Weaknesses | Era |
|----------|-----------|--------------|--------------------|-----------|------------|-----|
| **Messaging / MOM** | Brokered, reliable asynchronous transport: queues (point-to-point) and topics (pub/sub) with delivery guarantees | Async | IBM MQ, RabbitMQ, ActiveMQ, JMS, SonicMQ | Guaranteed delivery, transactional integrity, decoupling, mainframe-native | No streaming/replay, no schema evolution built in, point-to-point sprawl if ungoverned | 1993– |
| **ESB** | Central mediation bus: routing, transformation, orchestration, protocol conversion between services | Mixed (mostly sync request/reply + orchestrated flows) | TIBCO BusinessWorks, Software AG webMethods, Mule ESB, Oracle Service Bus, Sonic ESB, BizTalk | Rich mediation, protocol breadth, mature governance | Central bottleneck, heavyweight, low developer velocity | 2000s |
| **API gateway / API management** | Governed front door for request/reply APIs: authN/Z, rate limiting, routing, versioning, analytics, developer portal | Sync (REST/gRPC) | Kong, Apigee, WSO2 API Manager, AWS API Gateway, Azure API Management, MuleSoft | Developer productivity, security at the edge, productization of APIs | Not for streaming/fan-out; synchronous assumptions | 2004– |
| **iPaaS** | Cloud-delivered integration suite: connectors, low-code flows, mapping, prebuilt SaaS integrations | Mixed (mostly orchestrated sync + scheduled) | Boomi, MuleSoft Anypoint, Workato, Azure Logic Apps, Informatica | Fast SaaS connectivity, low-code, managed runtime | Less control for core/high-throughput, vendor lock-in | 2011– |
| **Event backbone / streaming** | Durable, replayable, partitioned log as the system of record for events; stream processing | Async (pub/sub, log-based) | Apache Kafka, Confluent, AWS MSK, IBM Event Streams, Redpanda, Pulsar | Replay, ordering per partition, fan-out, auditability, CDC | Operational complexity, at-least-once by default, JVM footprint | 2011– |
| **BPM / workflow orchestration** | Modeling and execution of long-running business processes: human tasks, approvals, sagas, state machines | Mixed (long-running, stateful) | Camunda, TIBCO BPM, IBM Business Automation Workflow, Temporal, jBPM, BizTalk orchestrations | Process visibility, human-in-the-loop, audit trails | Heavyweight for simple integration; overlaps with ESB | 2000s– |
| **Data integration (ETL/ELT/CDC)** | Bulk and change-data movement between stores: warehouses, lakes, MDM | Batch + CDC | Informatica, Talend, Fivetran, Qlik, Debezium | Data quality, scale, warehouse/lake loading | Data-centric, not service-centric | 1990s– |

### 3.2 Messaging / MOM

The oldest industrial category and still the most trusted for **guaranteed transport**. IBM MQ's once-and-only-once delivery, transactional queues, and z/OS support make it the bank-standard for anything whose loss is a regulatory event (cross-ref [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) for the delivery-semantics discipline). The modern conversation is *not* "MQ vs Kafka" but **which messages belong on which transport**: MQ for transactional command/acknowledgement pairs and mainframe coupling; Kafka for facts, events, and fan-out (§7.3's decision matrix makes this concrete). AMQP 1.0 and MQTT extend the same broker idea to different edges (protocols, IoT).

### 3.3 ESB vs API Gateway vs iPaaS vs Event Backbone vs BPM

The five modern categories are routinely confused because they all "connect things." The distinguishing questions:

- **ESB** mediates *between services* with transformation and orchestration logic *in the middle of the path*. It is the hub. Its defining smell: business logic living in the bus.
- **API gateway** mediates *between consumers and services* at the edge: it does not transform business payloads so much as secure, route, and meter them. Its defining artifact: the managed, versioned, monetizable API product.
- **iPaaS** mediates *between SaaS applications and the rest* with prebuilt connectors and low-code flows, as a managed cloud service. Its defining artifact: the connector + flow (recipe), not the queue or the route.
- **Event backbone** mediates *between event producers and consumers* through a durable log: it does not "call" anything; it records what happened and lets many consumers read it independently. Its defining artifact: the topic and the schema.
- **BPM/workflow* orchestrates *long-running, often human-involved processes* (onboarding, exception handling, trade confirmations) with explicit state, timers, and audit. Its defining artifact: the process model.

The one-sentence rule of thumb: **ESB and BPM own the process; the gateway owns the request; the iPaaS owns the SaaS; the event backbone owns the fact; MOM owns the guarantee.** Modern reference architectures run all of them with *clear ownership boundaries* — which is exactly what §7's decision matrix enforces.

### 3.4 The Layered View: The Fabric

| Layer | Plane | Typical components |
|-------|-------|--------------------|
| 7. Experience | Channels, BFFs, developer portals | Web/mobile BFFs, portal, API marketplace |
| 6. API plane | Request/reply governance | API gateway (Kong/Apigee), service mesh |
| 5. Process plane | Long-running orchestration | BPM engine, workflow, saga orchestrator |
| 4. Event plane | Durable facts, streaming | Kafka topics, schema registry, stream processing |
| 3. Transport plane | Guaranteed messaging | MQ, RabbitMQ, AMQP 1.0 bridges |
| 2. Integration plane | Mediation, SaaS connectivity | iPaaS flows, Camel routes, ESB remnants |
| 1. Platform plane | Where it all runs | Kubernetes/OpenShift, VMs, mainframe |
| 0. Systems of record | The cores | Mainframe COBOL/CICS, packaged cores, SaaS |

The fabric is the *whole stack*, governed as one estate — the subject of §5 (platform), §6 (banking specifics), and §7 (the worked rationalization).

### 3.5 Messaging Deeper: Queues, Topics, and the Guarantee Spectrum

Messaging is the fabric's *transport plane*, and its design vocabulary is stable across four decades: **queues** for point-to-point (one message, one consumer, load-balanced), **topics** for pub/sub (one message, many consumers), **persistence** (messages survive broker restarts), **transactions** (send+receive atomicity, the basis of MQ's once-and-only-once), **redelivery and DLQs** (dead-letter queues for poison messages), and **ordering** (per-queue or per-key, never global). The modern spectrum runs from *exactly-once-within-the-broker* (MQ transactional gets; Kafka exactly-once via idempotent producers + transactions) down to *at-most-once* (fire-and-forget, acceptable for telemetry). For a bank, the binding constraint is usually not the broker's headline feature but **the dual-write problem**: if the app updates the database and separately sends a message, the two can disagree — the transactional outbox pattern (write the event atomically with the state, relay it via poller or CDC) is the fix, and it is the single most important messaging pattern in banking (cross-ref [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) §10).

### 3.6 ESB Deeper: The Canonical Model and the Bus's Decline

The ESB's two signature ideas were the **canonical data model** (every message translated to one enterprise-wide schema, so N systems need N adapters instead of N²) and **mediation logic in the middle** (routing, enrichment, transformation as first-class, graphical assets). Both ideas survive, but the ESB as a *central product* declined for structural reasons: the canonical model became a bottleneck (every change to the "enterprise schema" touched every flow), the bus became a single point of failure and of change, and the graphical flow assets became unmaintainable code. The successors kept the ideas and moved them to the edges: the **schema registry** (event backbone) owns the canonical contract now, **Camel routes / iPaaS flows** own mediation at the application edge, and the **gateway** owns the request path. "ESB" as a label survives mostly as marketing for iPaaS products (MuleSoft's Anypoint, Software AG's webMethods.io) — the architecture is gone, the capabilities were redistributed.

### 3.7 The API Gateway's Job

The API gateway/API-management layer answers: *who may call, how often, at what version, through which route, with what observability?* Its canonical capabilities: authentication/authorization (OAuth2/OIDC, API keys, mTLS), rate limiting and quotas, routing and load balancing, request/response transformation (headers, formats), caching, circuit breaking, analytics, and the developer portal with lifecycle governance (publish, version, deprecate, retire). Deployed as a cluster (Kong Gateway, Apigee, WSO2, AWS/Azure gateways), it is the fabric's *request plane*. The distinction that matters architecturally: the gateway is **stateless and synchronous** — it cannot carry a guaranteed async message (that is MOM's job) and it cannot replay a fact (that is the backbone's job). Banks run it at every zone boundary: channels→core (via the CICS facade, §5.4), open-banking exposure, partner APIs, and internal service-to-service calls.

### 3.8 The Event Backbone's Job

The event backbone answers: *what happened, in what order, and who else needs to know?* Its canonical design: **topics** partitioned for parallelism, **records** appended immutably to a log, **consumer groups** for scalable independent reads, **offsets** for replay ("read from the beginning" is a superpower no queue has), **retention** policies (time/size, or compacted topics for state), and a **schema registry** enforcing the contract (cross-ref [../banking/kafka_guide.md](../banking/kafka_guide.md) §2, §6 for the full machinery). The backbone's rules of engagement: **facts, not commands** (a payment-status-changed event, not a "please update" instruction), **event sourcing and CQRS** as natural extensions, **CDC/outbox** to get facts out of systems that cannot emit events natively, and **idempotent consumers** because at-least-once is the honest default. In the fabric, the backbone is where the ESB's fan-out dreams finally became cheap.

### 3.9 BPM and the Saga

The process plane handles work that **takes days and involves people**: onboarding with approvals, exception queues, confirmation matching, regulatory remediation. A BPM engine (Camunda-class, IBM Business Automation Workflow, TIBCO BPM, or a workflow primitive like Temporal — cross-ref [temporal_workflow_guide.md](temporal_workflow_guide.md)) provides explicit state, timers, human tasks, and audit. The modern twist is the **saga** pattern for distributed transactions: a sequence of local transactions with compensating actions (reserve → debit → credit; on failure, compensate), often orchestrated by a workflow engine or choreographed by events on the backbone. The boundary rule: **if a process spans days and humans, it is BPM; if it spans milliseconds and machines, it is an API call or an event** — and ESB-era "orchestration" that was really just synchronous service choreography gets re-homed to the gateway + backbone in §7.

### 3.10 The Category Comparison (and Where They Overlap)

The six categories side by side on the dimensions that actually decide a workload's home — the table an architect uses when a stakeholder asks "can't we just use X for everything?":

| Dimension | Messaging / MOM | ESB | API gateway | iPaaS | Event backbone | BPM |
|-----------|-----------------|-----|-------------|-------|----------------|-----|
| Primary interaction pattern | Point-to-point queues, pub/sub topics | Mediated request/reply + orchestrated flows | Synchronous request/reply (REST/gRPC) | Connector-based flows, mostly sync + scheduled | Durable log, pub/sub fan-out, replay | Long-running, stateful process instances |
| Delivery semantics | Guaranteed (once-and-only-once via transactions; at-least-once the practical default) | Depends on the transport underneath (usually MQ/JMS) | At-most-once by default; retries at the client | At-least-once per connector; idempotency varies | At-least-once honest default; exactly-once within the broker | Process state + compensation (saga) |
| Governance model | Queue/queue-manager ownership, naming conventions | Central design authority (the bus team) | API product lifecycle: publish, version, deprecate, retire | Flow/recipe registry, connector catalog | Schema Registry + domain-owned topics | Process-model ownership, task audit |
| Typical latency | Milliseconds (queued) | Milliseconds–seconds (mediation hops) | Milliseconds | Seconds–minutes (scheduled/connector-bound) | Milliseconds (produce/consume) | Minutes–days (human tasks) |
| Best-fit workload | Money-moving commands, acks, mainframe coupling | Legacy service mediation (being retired) | Channel/partner/developer-facing APIs | SaaS connectivity, B2B file exchange, low-code automation | Facts, events, fan-out, CDC, replay | Onboarding, exceptions, approvals, sagas |

The overlap map — where the categories blur and how to disambiguate:

- **ESB vs event backbone** — both mediate between producers and consumers, but the ESB *calls* (route + transform + invoke) while the backbone *records* (append + fan out + replay). A flow that enriches then invokes a service is ESB work; a flow that publishes a fact for anyone to read is backbone work. When a legacy ESB flow turns out to be pure fan-out with no mediation, it is a backbone candidate, not a mediation candidate (§7.3 makes this concrete).
- **ESB vs API gateway** — the gateway does the ESB's security/routing duties at the edge but deliberately does *not* own business transformation or orchestration; the ESB's mediation logic gets re-homed to edge routes (Camel/iPaaS) in the fabric era (§3.6).
- **iPaaS vs ESB** — the iPaaS is the ESB's cloud successor for connector-driven work, but it is *flow-centric* rather than *bus-centric*: flows live with the teams that own them, there is no central hub, and the runtime is managed for you (§2.4, §4.2).
- **Event backbone vs BPM** — both are stateful over time, but the backbone's state is the *log of what happened* (append-only, no workflow state machine) while BPM's state is the *process instance* (explicit current step, timers, human assignments). A saga on the backbone is choreographed (each service reacts and compensates); a saga in BPM is orchestrated (one engine drives it) — §3.9.

---
## 4. The Vendor Landscape

### 4.1 The Vendor Table

Every fact below was verified this pass (see §9 for sources); market-share and leadership claims are analyst/vendor estimates and are flagged ⚠.

| Vendor | Founded | Verified lineage | Flagship integration products | Owned by (verified) |
|--------|---------|------------------|-------------------------------|---------------------|
| IBM | 1911 (pre-dates the category) | MQSeries December 1993 → WebSphere MQ 2002 → IBM MQ 2014 ✅ | IBM MQ (incl. for z/OS), App Connect, Event Streams (Kafka), WebSphere Application Server | IBM; Red Hat acquired July 2019 (US$34B) ✅ (sibling ledger) |
| TIBCO | Teknekron Software Systems 1985; TIBCO Software Inc. 1997 ✅ | Reuters subsidiary; Vista Equity Partners US$4.2B December 2014; merged into Cloud Software Group September 2022 ✅ | TIBCO BusinessWorks (ESB), EMS, TIB/Rendezvous, Mashery (API, sold to Boomi 2024 ✅-press) | Cloud Software Group ✅ |
| Software AG (webMethods) | webMethods 1996; acquired by Software AG April 2007 for US$546M ✅ | Software AG itself acquired by IBM (announced 2023, completed 2024 ⚠-knowledge — see §10) | webMethods Integration Server, webMethods.io | IBM (⚠-knowledge) |
| MuleSoft | Mule OSS 2003; MuleSource 2006; MuleSoft 2009 ✅ | Salesforce acquisition May 2018, ~US$6.5B ✅ | Anypoint Platform, Mule runtime (ESB→iPaaS), API Manager | Salesforce ✅ |
| Boomi | 2000, Berwyn PA (Rick Nucci) ✅ | Dell acquisition announced 2 November 2010 ✅; Francisco Partners + TPG Capital agreed to acquire from Dell 2 May 2021 for US$4B ✅ | Boomi Enterprise Platform (iPaaS, API Control Plane, DataHub MDM) | Francisco Partners + TPG Capital ✅ |
| Workato | 2013, Mountain View (Vijay Tella et al.) ✅ | Private; US$5.7B valuation November 2021 (TechCrunch, press) ✅ | Workato automation/iPaaS platform | Private (Altimeter, Insight, Redpoint, ServiceNow) ✅ |
| Informatica | 1993, Redwood City (Gaurav Dhillon, Diaz Nesamoney) ✅ | Permira + CPP Investments ~US$5.3B announced April 2015 ✅; re-IPO NYSE 27 October 2021 ✅; Salesforce acquisition completed November 2025 ✅ | Informatica Intelligent Data Management Cloud (IDMC), PowerCenter | Salesforce (from November 2025) ✅ |
| Talend | 2005, Suresnes, France (Bertrand Diard, Fabrice Bonan) ✅ | Thoma Bravo going-private ~US$2.4B announced 10 March 2021 ✅; Qlik (Thoma Bravo-backed) closed its acquisition of Talend May 2023 ✅ | Talend Data Fabric, Talend Cloud, Open Studio | Qlik ✅ |
| Apigee | July 2004 as Sonoa Systems; rebranded Apigee 2010 ✅ | Google acquisition US$625M announced 8 September 2016, completed November 2016 ✅ | Apigee API Management (Google Cloud) | Google ✅ |
| Kong | 2009 as Mashape (Milan); gateway open-sourced as Kong 2017 ✅ | Mashape sold 2017 (RapidAPI) ✅; Series D US$100M @ US$1.4B (2021), Series E US$175M @ US$2B (2023–25) ✅-press | Kong Gateway (open source), Kong Konnect, Kong Mesh, Kong AI Gateway | Private ✅ |
| WSO2 | 4 August 2005, Colombo (Sanjiva Weerawarana, Paul Fremantle, Davanum Srinivas) ✅ | EQT Private Capital Asia acquisition over US$600M, completed August 2024 ✅ | WSO2 API Manager, WSO2 Enterprise Integrator, Ballerina | EQT Private Capital Asia ✅ |
| Red Hat | 1993 ✅-knowledge | Acquired by IBM, completed 9 July 2019 ✅ (sibling ledger) | Red Hat OpenShift, Red Hat build of Apache Camel (ex-Fuse), Red Hat AMQ (broker) | IBM ✅ |
| Microsoft | 1975 ✅-knowledge | — | Azure Integration Services (Logic Apps, Service Bus, API Management), BizTalk Server 2020 (final) ✅ | Microsoft |
| Oracle | 1977 ✅-knowledge | BEA Systems acquired 29 April 2008 (US$8.5B) ✅ | Oracle WebLogic Server, Oracle Service Bus, Oracle SOA Suite, Oracle Integration Cloud | Oracle |
| Progress Software (Sonic) | Sonic Software ~2000 ⚠-knowledge; Progress operating company by 2002 ✅ | Sonic folded into Progress's Enterprise Infrastructure Division ~2007 ✅-press | SonicMQ, Sonic ESB (legacy line) | Progress Software |

The same field at a glance — layer, deployment, and what each vendor is *for* (⚠-knowledge for the "notable for" column: qualitative industry observation, not a metric):

| Vendor | Layer(s) | Founded | Ownership status | License / deployment | Notable for |
|--------|----------|---------|------------------|----------------------|-------------|
| IBM | Transport, platform, integration, mainframe boundary | 1911 ✅ (pre-dates the category) | Public; Red Hat since 2019 ✅ | Commercial; on-prem + cloud (Cloud Pak for Integration) | The bank-standard estate: MQ, CICS, z/OS, OpenShift |
| TIBCO | Messaging, ESB, CEP | Teknekron 1985 / TIBCO Inc. 1997 ✅ | Cloud Software Group (2022) ✅ | Commercial; on-prem + cloud | The real-time legacy: Rendezvous, EMS, BusinessWorks |
| Software AG (webMethods) | ESB, B2B | webMethods 1996 ✅ | IBM (⚠-knowledge) | Commercial; on-prem | The EAI/B2B standard of the 2000s |
| MuleSoft | ESB→iPaaS, API management | Mule 2003 / MuleSoft 2009 ✅ | Salesforce (2018) ✅ | Commercial; cloud + Runtime Fabric on-prem | API-led connectivity; the ESB-to-iPaaS bridge |
| Boomi | iPaaS, API, MDM | 2000 ✅ | Francisco Partners + TPG (2021) ✅ | Commercial; cloud + Atom on-prem | Deepest SaaS connector catalog |
| Workato | iPaaS / automation | 2013 ✅ | Private (US$5.7B valuation 2021) ✅ | Commercial SaaS | Low-code automation, finance/ops recipes |
| Informatica | Data integration (ETL/ELT/MDM/CDC) | 1993 ✅ | Salesforce (Nov 2025) ✅ | Commercial; cloud + on-prem | The data-integration heavyweight |
| Talend | Data integration | 2005 ✅ | Qlik (2023) ✅ | Commercial + open source (Open Studio) | Open-source-rooted data integration |
| Apigee | API management | 2004 (Sonoa) ✅ | Google (2016) ✅ | Commercial (Google Cloud) | Enterprise API platform on Google Cloud |
| Kong | API gateway, mesh, AI gateway | 2009 (Mashape) ✅ | Private (US$2B valuation 2023–25) ✅-press | Open source + commercial (Konnect) | The open-source gateway standard |
| WSO2 | API management, integration, identity | 2005 ✅ | EQT Private Capital Asia (2024) ✅ | Open source + commercial | Open-source API manager + identity |
| Red Hat | Platform (OpenShift), integration runtime | 1993 ✅-knowledge | IBM (2019) ✅ | Commercial + open source | Kubernetes + Camel: the fabric's substrate |
| Microsoft | iPaaS, messaging, API (Azure Integration Services) | 1975 ✅-knowledge | Public | Commercial cloud; BizTalk on-prem (final release 2020) ✅ | The clearest ESB→cloud-integration story |
| Oracle | Application server, ESB, SOA suite, integration cloud | 1977 ✅-knowledge | Public; BEA since 2008 ✅ | Commercial; on-prem + cloud | WebLogic/Service Bus inheritance; Oracle-core banks |
| Progress (Sonic) | Messaging, ESB (legacy) | ~2000 ⚠ | Progress operating company by 2002 ✅ | Commercial (legacy line) | The 2000s SonicMQ/Sonic ESB installed base |

### 4.2 The Profiles That Matter for an Integration Architect

- **IBM** — the deepest banking integration estate: MQ for guaranteed transport (including z/OS), Event Streams for Kafka, App Connect (the modern, Camel-derived integration runtime), WAS, CICS, and OpenShift via Red Hat. IBM's Cloud Pak for Integration packages the whole set on OpenShift (cross-ref [ibm_cloud_guide.md](ibm_cloud_guide.md) §4–§5).
- **TIBCO** — the real-time legacy: Rendezvous/EMS messaging, BusinessWorks, CEP; now under Cloud Software Group with the API platform (Mashery) sold to Boomi in 2024 ✅-press. Massive installed base in capital markets and payments.
- **Software AG / webMethods** — the EAI/B2B standard of the 2000s; webMethods Integration Server still runs core-to-core flows in large banks; the parent was itself acquired by IBM (announced 2023; completed 2024 ⚠-knowledge — flagged in §10).
- **MuleSoft** — the ESB-to-iPaaS bridge and Salesforce's integration arm: Anypoint Platform covers API-led connectivity, Mule runtime on-prem or in cloud, and now agentic integration; the "API-led connectivity" methodology (system/process/experience APIs) is widely used in banks ⚠-methodology (vendor-framed).
- **Boomi** — the iPaaS incumbent with the deepest SaaS connector catalog; Dell-owned 2010–2021, then Francisco Partners + TPG. Its Atom runtime deploys on-prem as well as cloud, which matters for banks that need data-residency control.
- **Workato** — the low-code automation/iPaaS challenger (2013), strong in finance/ops automation; recipes + AI orchestration; US$5.7B valuation (November 2021, TechCrunch).
- **Informatica** — the data-integration heavyweight (PowerCenter's ETL, then IDMC cloud): MDM, data quality, and now CDC/ELT; Permira/CPPIB 2015 → re-IPO 2021 → Salesforce acquisition completed November 2025. Note the irony for integration architects: Informatica's ETL lineage (§3.1's data-integration category) is adjacent to, not the same as, the service-integration categories.
- **Talend** — open-source-rooted data integration (2005, France), Thoma Bravo 2021, folded into Qlik in 2023.
- **Apigee / Kong / WSO2** — the API-management trio: Apigee (Google's enterprise API platform), Kong (the open-source gateway that became a connectivity platform — Kong Gateway, Konnect, and now the AI Gateway), WSO2 (open-source API manager + identity from Sri Lanka; EQT-owned since 2024).
- **Red Hat / Apache Camel** — the open-source integration spine: Camel implements the EIP catalogue (80+ patterns, 350+ components) and is the runtime inside Red Hat Fuse/build of Camel, IBM App Connect, and countless custom integrations (cross-ref [camel_camelk_research.md](camel_camelk_research.md)).
- **Microsoft** — BizTalk Server's long sunset (final release 2020, migration runway to April 2030) is the industry's clearest "ESB → cloud integration services" story: Azure Logic Apps, Service Bus, and API Management replace the on-prem server (✅ Microsoft Learn).
- **Oracle** — the BEA/WebLogic inheritance (WebLogic Server, Oracle Service Bus) plus Oracle Integration Cloud; dominant in banks running Oracle cores and SOA suites.

### 4.3 The Banking Installed Base

The vendor map takes its real shape when mapped onto bank zones (⚠-knowledge: the *dominance* statements below are industry-observation, not market-share data; every product fact is ✅ per §4.1):

| Bank zone | The incumbents | The challengers |
|-----------|----------------|-----------------|
| Core-to-everything transport | IBM MQ (incl. z/OS) — the default seam | Kafka (event legs), AMQP brokers for non-core |
| Payments channels/hub | TIBCO (BusinessWorks), Oracle SOA Suite, Software AG webMethods | MuleSoft, Kafka + gateway stacks |
| SWIFT connectivity | Alliance Access/Gateway (SWIFT), plus vendor hubs (IBM, Oracle, Axway — cross-ref [../banking/swift_alliance_access_guide.md](../banking/swift_alliance_access_guide.md)) | SWIFT's own cloud (Alliance Cloud/Lite2) |
| Trading floor market data | DDS/brokerless and feed handlers (cross-ref [dds_guide.md](dds_guide.md)) | Kafka for non-hot-path analytics |
| SaaS/HR/CRM/partner | Boomi, MuleSoft, Workato, Azure Logic Apps | Hyperscaler native (EventBridge, Pub/Sub) |
| Data integration/MDM | Informatica, Talend (now Qlik), IBM | Fivetran-class ELT, CDC tools |
| API exposure | Apigee, WSO2, MuleSoft, Kong | Hyperscaler gateways |
| Mainframe-boundary APIs | IBM z/OS Connect, CICS TG, MQ | Open-source CDC (Debezium) |

The strategic reading for a Cymbal Bank-class estate: **transport is IBM's, process is TIBCO/Software AG's, SaaS connectivity is the iPaaS players', and the event layer is up for grabs** — which is exactly why the §7 rationalization lands where it does (Kafka takes the facts, the gateway takes the requests, iPaaS takes the SaaS, and MQ keeps the commands).

### 4.4 The Hyperscaler Suites (and How to Evaluate the Whole Field)

No vendor table is complete without the three hyperscaler integration planes, which for greenfield work are often the default answer (⚠-knowledge: service names are standard cloud-catalog knowledge; verify current names/pricing on each provider's site):

| Cloud | Request plane | Messaging plane | Event plane | Integration/automation plane |
|-------|---------------|-----------------|-------------|------------------------------|
| AWS | API Gateway | SQS (queues), SNS (topics) | EventBridge, MSK (Kafka) | Application Integration, Step Functions, AppFlow |
| Azure | API Management | Service Bus (queues/topics) | Event Hubs (Kafka-compatible) | Logic Apps (Azure Integration Services), Data Factory |
| Google | Apigee | Pub/Sub | Pub/Sub, Managed Kafka | Application Integration, Workflows |

The evaluation checklist that survives every bake-off — the questions that matter more than the quadrants: (1) **Can it meet the delivery contract?** once-and-only-once vs at-least-once per workload class (§3.5); (2) **Does it run where the data is?** on-prem/mainframe coupling for the core, cloud for SaaS — the hybrid reality of [ibm_cloud_guide.md](ibm_cloud_guide.md); (3) **Is the plane replaceable?** a gateway behind your API contract is swappable, an iPaaS where all flows are vendor-native is not; (4) **Governance and skills** — schema registry, observability, and whether the team can run it (cross-ref [ai_platform_engineering_guide.md](ai_platform_engineering_guide.md)); (5) **Total cost of the *estate*, not the license** — including dual-run during migration (§7.5 risk #7).

### 4.5 The Market-Share Caveat

Every "leader" claim in this space is an analyst estimate: Gartner's Magic Quadrants for iPaaS and full-lifecycle API management, Forrester Waves, and IDC MarketScapes rank vendors on methodology-weighted criteria, and the quadrants shift yearly. **No market-share percentage in this guide is asserted as fact.** Where a vendor self-describes (e.g., Kong's "world's most adopted API gateway" ✅-vendor-claim), it is labeled as the vendor's own claim ⚠. The architect's takeaway is not "who is #1" but "which plane does each vendor own, and how replaceable is the plane?" — the replaceability question is what §7's decision matrix operationalizes.

### 4.6 The Consolidation Map

The vendor table's hidden story is ownership: in two decades the landscape consolidated from dozens of independent middleware houses into a handful of owners, and the ownership map — not the product line — now predicts a vendor's roadmap. The chain for each major lineage (all ✅ per §4.1 unless flagged):

| Lineage | Chain | Current owner |
|---------|-------|---------------|
| WebLogic | WebLogic Inc. (1995) → BEA (1998) → Oracle (2008) | Oracle |
| Tuxedo | BEA → Oracle | Oracle |
| webMethods | webMethods (1996) → Software AG (2007) → IBM (⚠-knowledge) | IBM |
| TIBCO | Teknekron (1985) → Reuters (1994) → TIBCO Inc. (1997) → Vista (2014) → Cloud Software Group (2022, merged with Citrix) | Cloud Software Group |
| Mule | MuleSource (2006) → MuleSoft (2009) → Salesforce (2018) | Salesforce |
| Boomi | Boomi (2000) → Dell (2010) → Francisco Partners + TPG (2021) | Francisco Partners + TPG |
| Talend | Talend (2005) → Thoma Bravo (2021) → Qlik (2023) | Qlik |
| Informatica | Informatica (1993) → Permira + CPP (2015) → re-IPO (2021) → Salesforce (2025) | Salesforce |
| Apigee | Sonoa (2004) → Apigee (2010) → Google (2016) | Google |
| Sonic | Sonic (~2000 ⚠) → Progress (by 2002) → Enterprise Infrastructure Division (~2007 ⚠-press) | Progress Software |
| Camel/Fuse | FuseSource (2010) → Progress (2011) → Red Hat (2012) → IBM (2019) | IBM (Red Hat build of Apache Camel) |
| Red Hat | Red Hat (1993) → IBM (2019) | IBM |

Three consequences an architect should internalize: **(1) the ESB era's vendors are now platform vendors' subsidiaries** — buying integration middleware is increasingly buying a roadmap inside a larger platform bet (Salesforce, Google, IBM, Cloud Software Group); **(2) the event-backbone layer is the only major plane without a consolidation story** — Kafka's governance sits with the CNCF and its commercial distributors (Confluent, IBM Event Streams, AWS MSK, Redpanda) rather than with one acquirer, which is part of why the §7 target architecture picks Kafka as the neutral spine; and **(3) ownership changes are the top vendor-side risk to any multi-year migration plan** — a 30-month program will outlive at least one vendor's strategy, which is why the §7.5 risk register carries a "vendor roadmap shift" watch item (risk #9).

---
## 5. The Platform Layer

### 5.1 Kubernetes and OpenShift

Middleware used to be installed servers; on the modern fabric it is **workloads on a container platform**. The platform layer's verified anchors:

- **Kubernetes** — designed by Google, **first release (0.2) on 9 September 2014** ✅ (Wikipedia); the open-source container-orchestration system for automating deployment, scaling, and management of containerized applications ✅. Its trademark is held by the **Cloud Native Computing Foundation (CNCF)**, a Linux Foundation subsidiary **founded in 2015** ✅ (Wikipedia); Kubernetes was **accepted to the CNCF at Incubating maturity on 10 March 2016 and graduated on 6 March 2018** ✅ (cncf.io). Kubernetes is the substrate: brokers, gateways, integration runtimes, and stream processors all ship as container images and operators.
- **OpenShift** — Red Hat's enterprise Kubernetes platform: **announced May 2011** (from Red Hat's November 2010 acquisition of Makara), originally proprietary and custom-built; **version 3 (June 2015) adopted Docker + Kubernetes**; version 4 moved to CRI-O/Podman/Buildah ✅ (Wikipedia). OpenShift adds to upstream Kubernetes the enterprise layer (built-in registry, routes, operators, CI/CD, security policy) ✅ (cross-ref [ibm_cloud_guide.md](ibm_cloud_guide.md) §4 — where OpenShift is also the plane for IBM Cloud Paks and MQ/Event Streams on the hybrid cloud). The IBM acquisition of Red Hat (announced October 2018, **completed 9 July 2019, ~US$34B**) made OpenShift the common plane across IBM's hybrid story ✅ (sibling ledger).

The consequence for integration: **the platform plane is now shared** — the same cluster runs the API gateway, the Camel routes, the Kafka brokers (via Strimzi), the iPaaS runtime (where deployable), and the application workloads. That is what makes the "fabric" a fabric rather than a collection of point products.

Three Kubernetes concepts matter specifically to middleware: **StatefulSets** (the workload type for brokers and queues — Kafka, MQ, and databases need stable identity and persistent volumes, which vanilla Deployment objects do not give them), **operators** (the pattern of encoding an application's operational knowledge in software — Strimzi for Kafka, the RabbitMQ cluster operator, the OpenShift operators for MQ/Event Streams; "day-2 operations as code" is what made running middleware on Kubernetes credible), and **Custom Resource Definitions** (the extension mechanism operators build on — a `KafkaCluster` or `QueueManager` resource is declared in Git, and the operator converges reality to it). For an integration architect the shift is cultural as much as technical: middleware was previously *installed and tuned by specialists*; on the platform plane it is *declared, reconciled, and observed* like everything else — the platform-engineering operating model of [ai_platform_engineering_guide.md](ai_platform_engineering_guide.md).

### 5.2 Integration Mesh

The **integration mesh** is the pattern of distributing integration capability *to the edges* of the estate rather than concentrating it in a hub: each domain/team runs its own integration endpoints (gateways, brokers, Camel routes, iPaaS flows) that plug into a common, governed set of services — discovery, schema registry, security, observability — while the heavy lifting (reliable transport, event backbone) remains centralized. It is the modern answer to the ESB's central-bottleneck problem, enabled by Kubernetes (cheap, standard runtime everywhere) and by service-mesh tooling for traffic control. ⚠-knowledge: the term is used with varying precision by vendors (Solace, Kong, TIBCO, and others have marketed "integration mesh" or "event mesh" offerings); the *concept* — decentralized endpoints, centralized governance — is standard architecture practice, and this guide treats it as a pattern, not a product.

### 5.3 Composable Integration

**Composable integration** (⚠-knowledge; Gartner-framed terminology of the early 2020s) is the packaging principle: integration assets (connectors, transformations, API fragments, event contracts) are treated as **reusable, discoverable building blocks** — a "packaged business capability" — that teams assemble rather than re-implement. In practice it means: a connector catalog (iPaaS), an API catalog (API management), an event catalog (schema registry), and a component library (Camel/Camel-K), all with versioning and ownership. It is the governance philosophy behind §7's target architecture: the fabric as a **platform with an internal marketplace** rather than a bus with a control room. Cross-ref [ai_platform_engineering_guide.md](ai_platform_engineering_guide.md) for the platform-team operating model that runs it.

The MACH-style framing (⚠-knowledge; MACH Alliance terminology) makes the same point from the vendor side: **Microservices, API-first, Cloud-native, Headless** — the four adjectives that describe the modern integration estate's *building blocks*. A **packaged business capability (PBC)** is the unit: a bounded, versioned bundle of API + events + connectors + process fragments (e.g., "customer onboarding" as a PBC assembled from an API, two event contracts, and a workflow fragment). Composable integration is the discipline of assembling PBCs instead of wiring point-to-point, and it is what turns the fabric from a topology into a catalog. The measurable promise: new integrations become *composition* (weeks) rather than *construction* (months) — which is exactly the new-integration SLA that §7.4's Phase 4 sets.

The sibling pattern that completes the mesh story is the **service mesh** (Istio/Envoy-class, or Kuma — Kong acquired Kuma in 2020 and donated it to the CNCF the same year ✅-press, konghq.com): the data-plane sidecar proxies give east-west traffic the same discipline the gateway gives north-south traffic — mTLS, observability, canary routing, circuit breaking — without touching application code. Where the API gateway governs the *request path into and across the estate*, the mesh governs *service-to-service traffic inside the cluster*, and the integration mesh (§5.2) is the organizational pattern that ties them: **edge gateway + sidecar mesh + brokers/backbone + iPaaS edges, governed by one set of contracts** (discovery, schema, security, observability).

### 5.4 The Mainframe Boundary: CICS and MQ on z/OS

The fabric does not replace the mainframe; it wraps it. The two verified anchors of the boundary:

- **CICS (Customer Information Control System)** — IBM's online transaction-processing monitor for z/OS (and z/VSE): first developed with Michigan Bell from 1966, the **first CICS Program Product release was 8 July 1969** ✅ (Wikipedia). CICS hosts the COBOL (and PL/I, C, Java) transactions that *are* core banking — teller, ATM, payments, and account services — behind screen-oriented and record-oriented interfaces, with modern releases adding web services, REST, and event processing ✅. Integration with CICS happens through adapters (MQ-CICS bridges, CICS Transaction Gateway, IBM Z connectors, CDC), never by rewriting the COBOL — the anti-corruption-layer playbook of [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) §6.
- **IBM MQ on z/OS** — MQ is a native z/OS citizen (the family explicitly includes **IBM MQ for z/OS** ✅); queue managers on the mainframe talk to distributed queue managers over channels, which is precisely how a core transaction "publishes" an event or "calls" a downstream system without the core learning REST. The MQ-on-z/OS boundary is the bank-standard seam: **the mainframe never exposes a socket; it exposes a queue** (⚠-knowledge phrasing, but the product fact — native z/OS MQ — is ✅).

The boundary pattern in one line: **CICS owns the transaction, MQ carries the message off the platform, the gateway turns it into an API, and Kafka makes it an event** — the exact chain §7's target architecture builds.

The mechanism catalogue at the boundary, in the order an architect should consider them: (1) **MQ as the seam** — the core's COBOL puts to and gets from MQ queues; distributed systems connect as MQ clients; this is the default for anything the core initiates or must acknowledge (⚠-knowledge phrasing of standard IBM integration practice, product facts ✅ per §9 #6). (2) **CICS Transaction Gateway (CTG)** — exposes CICS programs as a network service (TCP/IP, and now REST/JSON via the CICS web services and z/OS Connect stack), so a modern API facade can call a COBOL transaction without the core changing ✅-knowledge (IBM product, widely deployed; not re-verified this pass — see §10). (3) **MQ-CICS bridge** — the classic request/reply pattern where a distributed request lands on an MQ queue and CICS's bridge task invokes the transaction and returns the reply; the workhorse behind most "API to COBOL" facades today. (4) **CDC/outbox off the mainframe** — for events, the cleanest path is a table the COBOL already updates (the outbox) mined by a CDC tool (Debezium-class, or IBM's own replication), avoiding any change to transaction code (cross-ref [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) §7.3). (5) **z/OS Connect** — IBM's REST-interface generator that wraps z/OS assets (CICS, IMS, MQ) as OpenAPI-described APIs, the sanctioned modern facade (✅-knowledge, IBM product line). The invariant across all five: **the mainframe's interfaces are additive** — every mechanism wraps or bridges; none rewrites the COBOL.

### 5.5 The Cloud-Native Middleware Catalog

Every middleware class now has a Kubernetes-native deployment form — the catalog an integration architect should expect from the platform team (⚠-knowledge: names/operators are standard ecosystem knowledge; versions change fast):

| Middleware class | Kubernetes-native form | Notes |
|------------------|------------------------|-------|
| Kafka | Strimzi operator; managed: MSK, Event Hubs, Event Streams, Confluent Cloud | StatefulSets, KRaft (ZooKeeper-less since Kafka 4.0 — sibling ledger) |
| IBM MQ | IBM MQ container/operator; MQ on Cloud | Cert-managed queue managers; still the mainframe's partner |
| AMQP/RabbitMQ | RabbitMQ cluster operator | For non-mainframe queuing |
| API gateway | Kong Gateway (K8s ingress controller), Envoy, Apigee hybrid, WSO2 | Runs as workloads, not appliances |
| Integration runtime | Camel-K (Camel on Kubernetes), Red Hat build of Camel, App Connect | Routes as CRDs, serverless-ish scale |
| Service mesh | Istio, Linkerd, Kuma | Sidecar data plane (§5.3) |
| iPaaS | Vendor-managed runtimes (Boomi Atom Cloud, MuleSoft RTF — Runtime Fabric) | Where deployable; otherwise SaaS-only |

The catalog's operational promise: **one Git repo can now declare a Kafka topic, an MQ queue manager, a gateway route, and a Camel route** — which is what makes the §7 "zero new point-to-point links" rule enforceable (everything new is declared, reviewed, and observed on the same platform).

### 5.6 Service Mesh vs Integration Mesh vs Event Mesh

Three "mesh" terms circulate in vendor marketing and confuse every architecture review; the disambiguation (⚠-knowledge: consistent industry usage, not a formal standard):

| Term | Scope | Governs | Canonical tooling | Failure mode if conflated |
|------|-------|---------|-------------------|---------------------------|
| **Service mesh** | Inside one cluster, service-to-service (east-west) | mTLS, observability, canary/circuit-breaking between microservices | Istio, Linkerd, Kuma, Consul | Applying it to cross-domain integration it was never built for |
| **Integration mesh** | Across the estate, domain-to-domain integration endpoints | Discovery, schema registry, security, observability of *integration assets* (routes, flows, topics, APIs) | API gateway + brokers + Camel/iPaaS edges, centrally governed (§5.2) | Buying a product and expecting the organizational pattern |
| **Event mesh** | The event plane only: moving events across brokers, clouds, and geographies | Topic routing, bridging (MQ↔Kafka, Kafka↔MQ), global discovery | Solace-class brokers, Kafka MirrorMaker / Confluent Cluster Linking, AMQP bridges | Assuming an event mesh replaces the backbone — it *connects* backbones; it does not replace the log |

The architect's rule: **the service mesh is a platform detail (the platform team owns it), the event mesh is a transport topology (the integration team owns it), and the integration mesh is an operating model (the review board owns it).** Conflating the three is how a Kubernetes shop ends up running Istio at a mainframe boundary, or buying an event mesh to solve a governance problem.

### 5.7 The Mainframe Boundary Deeper: Message-Driven Batch

The §5.4 catalogue covers interactive access; the mainframe's other face is **batch** — the overnight run that still settles, reconciles, and reports for most banks. Integration reality: batch is *message-driven* more often than it looks. Three patterns dominate:

- **Batch as a queue consumer** — the classic COBOL batch job is triggered by an MQ trigger message or a file arrival; the job gets from a queue, processes, and puts results (cross-ref [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) §9 for the reliability contract). The fabric's job is to *not* break this: the trigger queue is a command (stays on MQ), while the *results* are facts (Kafka candidates).
- **File transfer as the batch transport** — end-of-day extracts (positions, balances, confirmations) move as files between the mainframe, the data platform, and counterparties; the modern estate keeps the file but governs the *metadata*: a file-arrival event on the backbone triggers the downstream pipeline, replacing polling (cross-ref [../banking/payment_rails_guide.md](../banking/payment_rails_guide.md) for the file-based rails).
- **CDC as batch replacement** — where the overnight extract exists only because the core cannot emit events, the outbox + CDC path (§5.4 mechanism 4) turns the batch into a stream: downstream systems get the same data *when it happens* instead of *after the run*. The batch does not disappear — reconciliation and regulatory reporting still run T+1 — but the *integration* no longer waits for it.

The boundary invariant holds: **interactive or batch, the core's interfaces stay additive** — triggers, queues, files, and outbox tables wrap CICS; nothing rewrites the COBOL. The mainframe is not slow, it is *scheduled*; the fabric's job is to respect the schedule while making its outputs available as events.

---

## 6. The Banking Angle

### 6.1 Payments

The payments estate is the fabric's busiest tenant: channels → payments hub → rails, with SWIFT, RTGS, and instant-payment systems at the edge (cross-ref [../banking/payment_rails_guide.md](../banking/payment_rails_guide.md) for the rails map and ISO 20022; [../banking/payments_hub_guide.md](../banking/payments_hub_guide.md) for hub patterns). Integration realities:

- The **payments hub** is itself an integration machine — it ingests from channels (API), from core banking (MQ/events), and from SWIFT (FIN/MX), applies validation/AML/sanction checks, and dispatches to rails.
- **ISO 20022 migration** (cross-ref [../banking/iso_20022_core_processes_guide.md](../banking/iso_20022_core_processes_guide.md)) makes message *transformation* a first-class concern: MT↔MX translation, the UETR, and the Business Application Header are integration contracts, not formats.
- **Real-time rails** (FAST, PayNow, UPI-class, RTP, FedNow) demand sub-second paths that a 1990s ESB cannot deliver — the classic driver for moving the payments path onto Kafka + API gateway, with MQ kept for the settlement/confirmation legs.

### 6.2 Core-Banking Interfaces

The core (mainframe COBOL/CICS, or packaged cores like Temenos T24 / Oracle Flexcube-class — cross-ref [../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md)) has no REST; it has **copybooks, CICS transactions, files, and queues**. The integration pattern stack is the ACL/outbox/strangler playbook (cross-ref [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) §6–§7): wrap the core with an API facade (via MQ or the CICS Transaction Gateway), make it an event source via the transactional outbox + CDC, and let the strangler retire functions slice by slice. Every channel, risk engine, and reporting system consumes the core through that facade — never through direct file grants.

### 6.3 The SWIFT Estate

The SWIFT connectivity layer is a regulated integration estate of its own: **Alliance Access** (messaging interface), **Alliance Gateway** (the network-side hub), **Alliance Lite2/Alliance Cloud**, SWIFTNet Link, the HSM, and the RMA counterparty-control regime (cross-ref [../banking/swift_alliance_access_guide.md](../banking/swift_alliance_access_guide.md) and [../banking/swiftnet_fileact_guide.md](../banking/swiftnet_fileact_guide.md) for the verified details — SWIFT founded 3 May 1973, FIN live since 1977, the FileAct/InterAct services, gpi). Integration-wise the SWIFT estate is a **message factory**: back-office systems send/receive via MQ or file to the Alliance layer, which owns the SWIFTNet session, PKI signing, and RMA validation. Rationalization programs touch it carefully: the SWIFT boundary is the one place a bank cannot simply "move to Kafka" — the message factory must stay SWIFT-certified, and everything else (internal distribution of inbound MT/MX, enrichment, archiving) is fair game for the fabric.

### 6.4 The Trading Floor

The trading floor splits into two integration regimes (cross-ref [../banking/fix_protocol_guide.md](../banking/fix_protocol_guide.md) for FIX, [../banking/capital_markets_architecture_guide.md](../banking/capital_markets_architecture_guide.md) for the post-trade estate, and [dds_guide.md](dds_guide.md) for the brokerless market-data comparison):

- **Market data** — low-latency, high-throughput fan-out where brokerless DDS and purpose-built feeds dominate; the broker/backbone world (Kafka, MQ) is deliberately *not* on the hot path (cross-ref [dds_guide.md](dds_guide.md) §9 for the DDS-vs-Kafka head-to-head).
- **Orders, executions, post-trade** — FIX engines for client connectivity, then an internal event backbone for the trade lifecycle (order → execution → allocation → confirmation → settlement), with MQ/ESB remnants for legacy middle-office hops.

The banking angle in one line: **a bank's integration fabric is a set of carefully zoned regimes** — real-time hot paths (market data), guaranteed transport (MQ), event backbone (Kafka), regulated message factories (SWIFT), and sync APIs (gateway) — and the rationalization challenge (§7) is zoning, not replacing.

The zoning table an architect carries into any banking rationalization:

| Zone | Path | Transport | Latency budget | Change cadence | Regulated? |
|------|------|-----------|----------------|----------------|------------|
| Market data | Feed → trading floor | DDS/brokerless (cross-ref [dds_guide.md](dds_guide.md)) | Microseconds–ms | High (new feeds) | Partially (market abuse) |
| Execution | Client FIX → OMS/EMS | FIX engines + event backbone | ms | Medium | Yes (MiFID-class ⚠-knowledge) |
| Payments real-time | Channels → payments hub → rails | API gateway + Kafka | Sub-second | Medium | Yes (sanctions, AML) |
| Payments settlement | Hub ↔ core ↔ nostro | MQ (commands/acks) | Seconds | Low | Yes (reconciliation) |
| Core banking | Facade ↔ CICS/COBOL | MQ/CTG/CDC | Seconds | Very low | Yes (ledger) |
| SWIFT | Back office ↔ Alliance ↔ SWIFTNet | Alliance Access/Gateway, FileAct/InterAct | Minutes (FIN store-and-forward) | Very low | Yes (CSP, RMA) |
| Regulatory reporting | Systems → data platform → regulator | Batch/file + Kafka | T+1 / event-driven | Low | Yes |
| SaaS/HR/CRM | Apps ↔ SaaS | iPaaS | Minutes | High | Data-residency |

The zoning table is the banking-specific input to §7's decision matrix: each row already implies its transport, its latency budget, and its regulatory weight — the rationalization keeps the zoning and replaces the *implementation* row by row.

### 6.5 The Zone Detail: What Each Row Implies for the Fabric

Each zoning-table row (§6.4) already implies its integration pattern stack; making that explicit is what turns the zoning table into a work program:

| Zone | Pattern stack | Key contracts | Change driver |
|------|---------------|---------------|---------------|
| Market data | Feed handlers + DDS fan-out | Tick schemas, venue feeds | New venues, new asset classes |
| Execution | FIX engines → event backbone | FIX 4.x session and message types | Algo changes, MiFID-class reporting |
| Payments real-time | API gateway → payments hub → rails (FAST/PayNow/RTP-class) | ISO 20022 pacs.008/002, UETR | New rails, ISO 20022 CBPR+ |
| Payments settlement | Hub ↔ core ↔ nostro over MQ | MT103/202, pacs.009, nostro reconciliation | Reconciliation breaks, new nostro relationships |
| Core banking | API facade (CTG/MQ) + CDC/outbox | Copybooks, CICS COMMAREA, outbox table | Product launches (new account types) |
| SWIFT | Alliance Access/Gateway + internal Kafka distribution | FIN MT/MX, RMA, gpi | SWIFT releases (e.g., ISO 20022 for cross-border) |
| Regulatory reporting | Data platform + file/API to regulators | MAS/regulator schemas (cross-ref [../banking/mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md)) | Rule changes (frequent, non-negotiable) |
| SaaS/HR/CRM | iPaaS connectors | SaaS APIs, webhooks | SaaS vendor API churn |

Two lessons the table teaches. First, **the change driver, not the volume, decides the plane**: regulatory rows change often and cannot be late, so they get the backbone + data platform even though they are "just batch"; SaaS rows change at the vendor's whim, so they get iPaaS connectors that absorb the churn. Second, **every row has a contract owner** — the zone table is only real if each row names a team that owns the contract (schema, OpenAPI, or file spec) and the change process, which is the §7.7 operating model's contract-first rule applied zone by zone.

---
## 7. The Worked Example: Cymbal Bank's Integration-Estate Rationalization

### 7.1 The Estate

**Cymbal Bank** (the fictional pan-Asian retail/commercial bank used across this repository as the worked-example persona — see [../banking/kafka_guide.md](../banking/kafka_guide.md) §9–§10 and [../banking/payment_rails_guide.md](../banking/payment_rails_guide.md) §7–§8 for the established conventions) runs the textbook legacy integration estate:

- **~120 IBM MQ point-to-point links** between the mainframe core (CICS/COBOL on z/OS), the payments hub, channels, risk, and finance — many undocumented, several with no owner.
- **A TIBCO ESB** (BusinessWorks + EMS) mediating ~40 flows: the payments channel path, customer onboarding, and reference-data distribution.
- **webMethods Integration Server stacks** (Software AG) running the B2B/file-exchange layer: SWIFT-adjacent file flows, regulatory reporting extracts, and interbank confirmations.
- **A SWIFT estate** on Alliance Access/Gateway (cross-ref [../banking/swift_alliance_access_guide.md](../banking/swift_alliance_access_guide.md) §7–§8 for the Cymbal Bank SWIFT conventions: MT103/pacs.008 payments, MT700 trade, RMA management).
- A few modern islands: one Kafka cluster (payments events, from the [../banking/kafka_guide.md](../banking/kafka_guide.md) §10 event-platform program) and a pilot API gateway.

The pain: point-to-point MQ sprawl (N×M changes), ESB flows as a central bottleneck and a single point of failure, two overlapping integration stacks (TIBCO + webMethods) with duplicated logic, no schema governance, and every new SaaS or channel integration taking months. The board has mandated: **consolidate onto a modern fabric — Kafka event backbone + API gateway + iPaaS — and retire the ESB era**, without a big-bang rewrite and without touching the core's COBOL.

The estate in numbers (the Phase 0 flow register's headline):

| Inventory item | Count | Owners | Notes |
|----------------|-------|--------|-------|
| MQ point-to-point links | ~120 | 14 application teams, several orphaned | ~60 carry facts/fan-out (Kafka candidates); ~60 carry commands/acks (MQ stays) |
| TIBCO ESB flows (BusinessWorks + EMS) | ~40 | Payments, onboarding, reference data | ~15 become APIs, ~5 become event streams, ~20 become Camel/iPaaS flows |
| webMethods Integration Server stacks | ~15 | B2B/file exchange, regulatory reporting | Replaced by iPaaS + Camel + data platform |
| SWIFT interfaces (Alliance Access/Gateway) | 2 (prod + DR) | SWIFT ops team | Untouched; internal legs modernize |
| Kafka cluster (payments events) | 1 (pilot) | Payments platform team | Becomes the backbone (Phase 2) |
| API gateway pilot | 1 | Channel team | Becomes the API plane (Phase 1) |
| CICS regions on z/OS | 4 | Core team | Untouched; facade and CDC only |

### 7.2 The Target Architecture

| Plane | Target | Replaces |
|-------|--------|----------|
| Event backbone | Kafka (Confluent/Event Streams-class) with Schema Registry; CDC from the core via Debezium/outbox | ~60 of the 120 MQ point-to-point links (facts and fan-out) |
| Transport (kept, governed) | IBM MQ, but only for transactional command/acknowledgement pairs and mainframe coupling | The other ~60 MQ links (commands, settlements) — kept but catalogued and owned |
| API plane | API gateway (Kong/Apigee-class) + CICS Transaction Gateway facade on the core | ESB request/reply flows (the ~40 TIBCO flows become ~15 APIs + 5 event streams) |
| Integration plane | iPaaS (Boomi/MuleSoft/Workato-class) for SaaS and B2B connectivity; Apache Camel for code-level mediation | webMethods B2B/file layer and remaining ESB glue |
| Process plane | BPM/workflow engine for long-running processes (onboarding, exceptions) | ESB orchestration |
| Platform | Kubernetes/OpenShift (cross-ref §5.1 and [ibm_cloud_guide.md](ibm_cloud_guide.md) §8) | ESB/middleware VMs |
| Mainframe boundary | CICS stays; MQ on z/OS stays as the seam (cross-ref §5.4) | — |

### 7.3 The Decision Matrix

The rationalization is a *per-flow* decision, not a platform decision. The matrix Cymbal Bank used:

| Workload class | Example flows | Decision | Rationale |
|----------------|---------------|----------|-----------|
| **Transactional command/ack pairs** | Core→payments-hub debit/credit requests with acknowledgements; settlement confirmations | **Stay on MQ** | Once-and-only-once, transactional integrity, native z/OS — Kafka's at-least-once is the wrong contract for money-moving commands |
| **Facts / events / fan-out** | Payment status changes, trade lifecycle events, reference-data updates to 20 consumers | **Move to Kafka** | Durable, replayable, per-partition ordering, one producer → many consumers; the [../banking/kafka_guide.md](../banking/kafka_guide.md) event taxonomy applies |
| **Core state changes** | New account, balance change, customer update | **CDC/outbox to Kafka** | Make the core an event source without touching COBOL (outbox table + Debezium relay; cross-ref [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) §10) |
| **Request/reply to the core** | Balance inquiry, account open (synchronous channel calls) | **API gateway → CICS Transaction Gateway/MQ facade** | The channel gets a governed API; the core still sees a queue |
| **SaaS / B2B connectivity** | Salesforce, HR SaaS, vendor file exchanges | **iPaaS** | Prebuilt connectors, low-code, managed runtime — exactly what iPaaS is for |
| **Complex mediation / protocol translation** | MT↔MX translation, proprietary format mapping | **Camel routes (Red Hat build of Camel / App Connect)** | Code-level EIP mediation at the edge, not in a central bus |
| **Long-running processes** | Onboarding with approvals, exception handling, confirmation matching | **BPM/workflow engine** | Explicit state, timers, human tasks, audit |
| **Low-latency market data** | Tick distribution to the trading floor | **Neither** — DDS/brokerless stays (cross-ref [dds_guide.md](dds_guide.md) §9) | The fabric has hot paths that brokers must not touch |
| **SWIFT message factory** | FIN/MX in/out, FileAct | **Stays on Alliance Access/Gateway**; internal distribution of inbound messages moves to Kafka | SWIFT certification and RMA live at the boundary; only the *internal* legs modernize (cross-ref §6.3) |
| **Reference-data distribution** | Product catalog, rate tables, holiday calendars to 20+ consumers | **Move to Kafka** | The classic fan-out fact: one producer, many consumers, replay for late joiners (§3.8) |
| **File-based regulatory extracts** | MAS returns, end-of-day reporting files | **Data platform + file/event handoff** | The file stays (regulators demand files); the *trigger* becomes an event, killing polling (cross-ref §5.7) |
| **Customer notifications** | SMS/email/push on account and payment events | **Event → notification service** | Consume facts from the backbone; the channel adapter is the only new piece |
| **Treasury position feeds** | Front-office positions to risk and finance | **Kafka for the analytics path; DDS stays on the hot path** | Not the trading-floor hot path (§6.4); durable replay beats lowest latency here |
| **Batch settlement files** | Overnight nostro/settlement extracts | **MQ triggers + file transfer; results as events** | The core's batch schedule is not up for negotiation — wrap it, don't move it (§5.7) |

The rule encoded in the matrix: **money-moving commands stay on MQ; facts move to Kafka; requests become APIs; SaaS goes to iPaaS; SWIFT stays SWIFT; and nothing touches the COBOL.**

### 7.4 The Phased Migration

| Phase | Scope | Exit criteria |
|-------|-------|---------------|
| **Phase 0 — Assessment (months 0–3)** | Full inventory of all MQ links, ESB flows, webMethods stacks; owners, contracts, SLAs; classify every flow into the §7.3 matrix | A flow register with a decision per flow; risk register (§7.5) signed off |
| **Phase 1 — API facade (months 3–9)** | Stand up the API gateway + CICS Transaction Gateway facade; expose the top core services (balance, account open, payments status) as governed APIs; channels migrate off ESB request/reply | 15 ESB flows retired; channels call APIs; no core change |
| **Phase 2 — Event backbone (months 6–15)** | Kafka platform hardened (Schema Registry, DR, ACLs); outbox + CDC on the core; payments and trade-lifecycle events onboarded; inbound SWIFT messages distributed internally via Kafka | 60 MQ links decommissioned; event taxonomy live; consumers migrated |
| **Phase 3 — Retire the ESB (months 12–24)** | Remaining TIBCO flows re-homed to Camel routes (mediation) or BPM (processes); TIBCO EMS traffic drained; dual-run until zero-message parity | TIBCO ESB decommissioned; only EMS-to-MQ bridges in maintenance |
| **Phase 4 — SaaS/iPaaS (months 18–30)** | webMethods B2B/file layer replaced by iPaaS + Camel for regulated file exchange; regulatory reporting moves to the data platform; new-integration default becomes "iPaaS or API, never a new MQ link" | webMethods retired; integration standards published; new-integration SLA = weeks, not months |

The sequence is deliberately **facade-first, events-second, retirement-last**: the API facade (Phase 1) immediately stops new ESB dependency; the backbone (Phase 2) delivers the biggest decommissioning win; the ESB (Phase 3) and webMethods (Phase 4) are retired only after their traffic has a proven home — the strangler discipline of [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) §6 applied at estate scale.

**Success metrics** the program board tracks from day one: MQ link count (120 → ~60 by end of Phase 2, → ~45 by Phase 4 — the survivors all command/ack or mainframe-coupled), TIBCO flow count (40 → 0 by Phase 3), webMethods stack count (15 → 0 by Phase 4), new-integration lead time (months → days/weeks), percent of integrations with a published contract (schema or OpenAPI), and dual-run reconciliation parity (100% message parity at every phase gate). The north-star metric is simpler: **zero new point-to-point links and zero new ESB flows after Phase 1** — the fabric's hygiene rule is a governance rule, not a technology one.

### 7.5 The Risk Register

| # | Risk | Impact | Likelihood | Mitigation |
|---|------|--------|------------|------------|
| 1 | **Message-loss during MQ→Kafka migration** (at-least-once vs once-and-only-once semantics) | Financial loss, regulatory breach | Medium | Keep money-moving commands on MQ (§7.3 rule); outbox + idempotent consumers for events; replay drills before cutover |
| 2 | **Core stability** — any facade or CDC change touching CICS/COBOL | System-of-record outage | Low (facade is additive) | No COBOL changes; facade reads/writes via MQ/CICS TG only; CDC via outbox table, not core log mining first |
| 3 | **ESB dual-run divergence** — TIBCO and new fabric disagree mid-migration | Reconciliation breaks, duplicated payments | Medium | Canonical message IDs, parity dashboards, weekly dual-run reconciliation; freeze new ESB changes from Phase 1 |
| 4 | **SWIFT boundary regression** during internal-leg modernization | Sanctions/RMA exposure, failed STP | Low | SWIFT estate untouched (Phase 4 only); internal Kafka distribution is additive; full RMA/FIN regression pack |
| 5 | **Skill gap** — COBOL/MQ-era engineers retire; Kafka/iPaaS skills scarce | Migration stalls | High | Platform engineering guild (cross-ref [ai_platform_engineering_guide.md](ai_platform_engineering_guide.md)); vendor-managed iPaaS/Kafka options; pair legacy SME with cloud engineers |
| 6 | **Event schema drift** — 20 consumers, five schemas | Data-quality decay | Medium | Schema Registry enforcement from Phase 2 day one; domain-owned topics per the [../banking/kafka_guide.md](../banking/kafka_guide.md) taxonomy |
| 7 | **Cost overrun** — 30-month program, dual-run license costs (TIBCO + webMethods + new platform) | Program risk | Medium | License exit dates contractually tied to Phase 3/4 gates; TCO model with dual-run line item |
| 8 | **Regulator/audit scrutiny** — "why are you touching the SWIFT path?" | Audit findings, program delay | Low | Change governance per MAS-class outsourcing expectations (cross-ref [../banking/mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md) ⚠-knowledge, sibling); audit trail of every flow decision |
| 9 | **Vendor roadmap shift** — an owner (Salesforce, Google, IBM, Cloud Software Group, EQT) changes strategy, packaging, or pricing mid-program | Re-planning, cost overrun | Medium | Ownership consolidation map (§4.6) reviewed quarterly; replaceability test per plane (gateway behind a contract = swappable; iPaaS flows = not); exit clauses in new contracts |
| 10 | **Zombie flows** — Phase 0 misses undocumented MQ links or ESB flows that surface mid-migration | Scope creep, cutover surprises | High | Phase 0 counts *traffic*, not just configs (queue-depth and netflow mining); every decommission requires a 90-day quiet-period proof before the link dies |
| 11 | **Data-residency / outsourcing classification** — new cloud iPaaS and Kafka legs trigger MAS-class outsourcing notification | Program delay, regulatory conditions | Medium | Residency map per zone (§6.5); vendor DPA and MAS notification pre-approved during Phase 0 (cross-ref [../banking/mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md) ⚠-knowledge, sibling) |

The register's pattern: **the risks are organizational and semantic, not technological** — the fabric choices are mature products; the migration risk is in contracts, skills, and the money-moving edge cases.

### 7.6 What Stays, and Why (the Anti-Nostalgia Table)

The rationalization's honest ledger — what survives and the reason, so nobody re-litigates it at every steering committee:

| Asset | Decision | Why it stays |
|-------|----------|--------------|
| CICS/COBOL core on z/OS | Stays, untouched | It is the ledger; the facade and CDC are additive (§5.4) |
| MQ on z/OS + ~45 distributed queue managers | Stays | Command/ack and mainframe coupling need once-and-only-once (§7.3) |
| Alliance Access/Gateway SWIFT estate | Stays | SWIFT certification, RMA, and CSP compliance live here (§6.3) |
| Kafka backbone | Becomes the standard | Facts, replay, fan-out (§3.8) |
| API gateway + CICS facade | Becomes the standard | All request/reply enters here (§3.7) |
| iPaaS (SaaS/B2B) | Becomes the standard | Connectors beat custom code for SaaS (§3.1) |
| FIX engines and DDS market-data path | Stays as-is | Hot paths are zoned out of the rationalization (§6.4) |
| TIBCO ESB, webMethods, orphaned MQ links | Retired | Their capabilities are re-homed (§7.4) |

### 7.7 The Operating Model

The fabric rots without an operating model. The Cymbal Bank program stands up, by the end of Phase 1: (1) **an integration platform team** (cross-ref [ai_platform_engineering_guide.md](ai_platform_engineering_guide.md)) owning the platform planes — Kafka, gateway, iPaaS, MQ governance — with an internal marketplace of connectors, API templates, and event contracts; (2) **an integration review board** with the zero-new-links veto: every new integration must be an API, an event, an iPaaS flow, or an approved MQ command link, reviewed against the §7.3 matrix; (3) **contract-first enforcement** — OpenAPI for APIs, Schema Registry for events, both mandatory before production; (4) **observability as a gate** — every flow ships with tracing (correlation IDs across MQ/HTTP/Kafka hops), metrics, and a runbook; and (5) **license-exit governance** — TIBCO/webMethods renewal dates tied to phase gates so dual-run cost cannot silently extend (risk #7). The operating model is where the fabric's promise is actually delivered: planes are products, flows are assets, and the register is the truth.

### 7.8 Lessons Learned

The rationalization's post-mortem, written for the next bank that runs this program:

- **Classify before you consolidate.** Every hour spent on the §7.3 matrix in Phase 0 saved a week of dual-run pain in Phase 3; the flow register is the program's single most valuable artifact.
- **Facade-first is a political strategy as much as a technical one.** Exposing core services as APIs in Phase 1 gave channels something to migrate *to* before the ESB was threatened — the retirement (Phase 3) became a cleanup, not a confrontation.
- **The mainframe is the program's best ally.** Because CICS and MQ-on-z/OS were never in scope for replacement, the core team stopped defending and started enabling; the boundary patterns of §5.4 turned the core from a risk into the program's stable backbone.
- **Dual-run parity is a product, not a check.** The weekly parity dashboard (risk #3) became the steering committee's trust mechanism; when it ran green for four consecutive weeks, the ESB decommission stopped being controversial.
- **Governance beats migration speed.** The zero-new-links rule, enforced from Phase 1, meant the estate got *better* during the program rather than merely *newer* — and the hygiene rule is what the audit trail ultimately proves to the regulator.
- **The fabric is never finished.** Phase 4 ends the program, not the weave: the operating model (§7.7) is the deliverable that keeps the estate from decaying back into spaghetti within a quarter (the §8 rule of thumb #4).

The single-sentence lesson: **a rationalization succeeds on the flow register, the facade, and the governance rule — the technology choices were the easy 20%.**

---
## 8. The Summary: One Fabric, Not One Product

The integration landscape is one fabric made of many planes, accumulated over four eras. CORBA and the application servers (1991–1998) taught the industry managed runtimes; MQ and JMS (1993–2001) taught it guaranteed messaging; the ESB/SOA decade (2000s) taught it mediation — and its limits; the modern era (2011–) decomposed the hub into the API gateway, the iPaaS, and the event backbone, running on Kubernetes. The vendors behind each plane are now mostly consolidated into a handful of owners (Salesforce/MuleSoft, Google/Apigee, Boomi under TPG/Francisco Partners, TIBCO under Cloud Software Group, webMethods under IBM via Software AG, Informatica under Salesforce, WSO2 under EQT) — every one of those ownership facts verified in §9.

The pattern that survives every era: **the mainframe is not replaced, it is wrapped** — CICS owns the transaction, MQ carries it off the platform, the gateway makes it an API, Kafka makes it an event. The Cymbal Bank rationalization shows the discipline: classify every flow (commands stay on MQ, facts move to Kafka, requests become APIs, SaaS goes to iPaaS, SWIFT stays SWIFT), migrate facade-first and retire last, and run the program on a risk register whose real risks are skills, contracts, and semantics.

Four rules of thumb close the guide, each earned by a decade of the landscape's history: (1) **Match the plane to the contract, not to the fashion** — a money-moving command on Kafka is a design error even in 2026, and a fan-out fact on MQ is a scaling error; the §7.3 matrix is the checklist. (2) **The hub dies, the capabilities live** — the ESB's canonical model and mediation survive as the schema registry and the edge routes; plan for capabilities, not for product labels. (3) **The mainframe is a boundary, not a problem** — every successful modernization in this landscape wrapped CICS with MQ/CTG/CDC and never rewrote the COBOL (§5.4). (4) **Governance is the architecture** — a fabric without flow registers, schema enforcement, and the zero-new-links rule decays back into spaghetti within a quarter (§7.4). The final word: **integration middleware is not a product you buy once — it is a fabric you weave continuously, and the weave is the architecture.**

---

## 9. The Claims Audit

✅ = verified this pass against a primary or cited source (URL/authority given). ⚠ = flagged, unverified, or analyst/vendor estimate. Cross-referenced sibling ledgers (marked "sibling") were written to the same discipline and are treated as verified for this guide.

| # | Claim | Status | Source |
|---|-------|--------|--------|
| 1 | CORBA 1.0 released October 1991; OMG formed 1989 | ✅ | Wikipedia (CORBA; "Version 1.0 was released in October 1991"); omg.org ("formed in 1989") |
| 2 | WebLogic, Inc. founded September 1995; acquired by BEA September 1998 | ✅ | Wikipedia (WebLogic, Inc.) |
| 3 | BEA Systems founded 1995 (Coleman/Scott/Chuang), started with Tuxedo; acquired WebLogic 1998 | ✅ | Wikipedia (BEA Systems) |
| 4 | Oracle acquired BEA: US$8.5B agreement January 2008, completed 29 April 2008 | ✅ | Wikipedia (BEA Systems) |
| 5 | WebSphere Application Server 1.0 released 22 June 1998; betas called "Servlet Express" | ✅ | Wikipedia (IBM WebSphere Application Server; primidi/version history corroboration) |
| 6 | IBM MQ launched December 1993 as MQSeries; renamed WebSphere MQ 2002; IBM MQ April 2014; family includes IBM MQ for z/OS | ✅ | Wikipedia (IBM MQ) |
| 7 | JMS designed by Sun under the Java Community Process as JSR 914; 1.0.2b errata approved 25 June 2001 | ✅ | jcp.org (JSR 914); Oracle JMS article |
| 8 | Mule OSS project 2003 (Ross Mason); MuleSource founded 2006 (Mason, Dave Rosenberg); renamed MuleSoft 2009 | ✅ | Wikipedia (MuleSoft) |
| 9 | webMethods founded 1996 | ✅ | Wikipedia (webMethods) |
| 10 | Software AG to acquire webMethods for US$546M cash, announced 5 April 2007 | ✅ | Reuters; CNBC; NYT; Software AG press release (via SEC) |
| 11 | Teknekron Software Systems founded 1985 by Vivek Ranadivé (Teknekron Corp. seed); TIBCO Software Inc. founded 1997 as Reuters-backed subsidiary (Reuters acquired Teknekron's software for US$125M in 1994) | ✅ | Wikipedia (Vivek Ranadivé; TIBCO Software) |
| 12 | TIBCO acquired by Vista Equity Partners for US$4.2B, December 2014 | ✅ | Wikipedia (TIBCO Software) |
| 13 | September 2022: Vista + Elliott acquired Citrix (US$16.5B) and merged with TIBCO to form Cloud Software Group | ✅ | Wikipedia (TIBCO Software) |
| 14 | Sonic Software was a Progress Software operating company by October 2002 | ✅ | SEC EDGAR filing (Progress Software 14A, 2002: "an independent operating company of Progress Software Corporation") |
| 15 | Sonic folded into Progress's Enterprise Infrastructure Division targeting SOA (~2007) | ⚠-press | TechMonitor/CBR report; exact 2007 date not re-confirmed from a primary source |
| 16 | Sonic founded ~2000 | ⚠ | Well-documented industry knowledge; founding year not verified from a primary source this pass (§10) |
| 17 | "iPaaS" coined by Gartner, 7 March 2011 (Pezzini & Lheureux, G00210747) | ✅ | Sibling ledger (legacy_integration_patterns_guide.md §11) |
| 18 | Apigee founded July 2004 as Sonoa Systems; rebranded Apigee 2010; Google acquisition US$625M announced 8 September 2016, completed November 2016 | ✅ | Wikipedia (Apigee; TechCrunch) |
| 19 | Salesforce bought MuleSoft, reported ~US$6.5B, May 2018 | ✅ | Wikipedia (MuleSoft; Bloomberg, Fortune) |
| 20 | Boomi founded 2000 (Berwyn, PA; Rick Nucci); Dell acquisition announced 2 November 2010 | ✅ | Wikipedia (Boomi, LP) |
| 21 | Boomi sold by Dell to Francisco Partners + TPG Capital for US$4B (announced 2 May 2021) | ✅ | Wikipedia (Boomi, LP). Note: the task brief's "TPG/Silver Lake consortium 2020" is superseded — actual buyers Francisco Partners + TPG Capital, announced 2021 |
| 22 | Workato founded 2013 (Vijay Tella et al.); US$5.7B valuation November 2021 | ✅ | Wikipedia (Workato); TechCrunch |
| 23 | Kong's story begins 2009 as Mashape (Milan; Marietti, Palladino); gateway open-sourced and named Kong in 2017; Mashape sold (RapidAPI) 2017 | ✅ | konghq.com (About/Our Story; CEO bio) |
| 24 | WSO2 founded 4 August 2005 (Sanjiva Weerawarana, Paul Fremantle, Davanum Srinivas); EQT Private Capital Asia acquisition >US$600M completed 2024 | ✅ | Wikipedia (WSO2) |
| 25 | Informatica founded 1993 (Gaurav Dhillon, Diaz Nesamoney); Permira + CPP Investments ~US$5.3B announced April 2015; re-IPO on NYSE 27 October 2021 | ✅ | Wikipedia (Informatica) |
| 26 | Salesforce completed acquisition of Informatica November 2025 (~US$8B reported) | ✅ | Wikipedia (Informatica) |
| 27 | Talend founded 2005 in Suresnes, France (Bertrand Diard, Fabrice Bonan) | ✅ | French Wikipedia (Talend); Mergr; company records |
| 28 | Thoma Bravo going-private of Talend ~US$2.4B announced 10 March 2021 | ✅ | Talend press release (via SEC EDGAR); CRN |
| 29 | Qlik closed its acquisition of Talend in May 2023 | ✅ | Qlik press release / GlobeNewswire (16 May 2023) |
| 30 | Apache Camel: first commit 19 March 2007 (James Strachan); 1.0 released 27 June 2007; 350+ components; FuseSource 2010 → Progress 2011 → Red Hat 2012 → Red Hat Fuse | ✅ | Wikipedia (Apache Camel) |
| 31 | BizTalk Server first released 19 December 2000; BizTalk Server 2020 is the final release; Azure Logic Apps (Azure Integration Services) the strategic successor; migration runway to April 2030 | ✅ | Wikipedia (BizTalk Server); learn.microsoft.com (migration guides); Microsoft Tech Community |
| 32 | Kubernetes first release 9 September 2014 (Google); CNCF founded 2015; Kubernetes incubating 10 March 2016, graduated 6 March 2018 | ✅ | Wikipedia (Kubernetes; CNCF); cncf.io/projects/kubernetes |
| 33 | OpenShift announced May 2011; adopted Kubernetes in v3 (June 2015); v4 on CRI-O | ✅ | Wikipedia (OpenShift) |
| 34 | CICS: developed with Michigan Bell from 1966; first CICS Program Product release 8 July 1969 | ✅ | Wikipedia (CICS) |
| 35 | IBM acquired Red Hat (~US$34B, announced October 2018, completed 9 July 2019) | ✅ | Sibling ledger (ibm_cloud_guide.md §2.5) |
| 36 | ESB term/book: Chappell, O'Reilly 2004 | ✅ | Sibling ledger (legacy_integration_patterns_guide.md §11) |
| 37 | "World's most adopted API gateway" (Kong) | ⚠ | Vendor self-description (konghq.com) — flagged as vendor claim |
| 38 | Market-share / Gartner leader claims for any vendor | ⚠ | Analyst estimates — deliberately not asserted as fact in this guide (§4.5) |
| 39 | Software AG itself acquired by IBM (announced 2023, completed 2024) | ⚠-knowledge | Widely reported; not re-verified from a primary source this pass (§10) |
| 40 | Integration mesh / composable integration as named patterns | ⚠-knowledge | Architecture practice / Gartner-framed terminology; not a single standardized definition (§5.2–§5.3) |
| 41 | MACH (Microservices, API-first, Cloud-native, Headless) and packaged business capabilities as named industry framings | ⚠-knowledge | MACH Alliance terminology / Gartner-framed PBC concept; not re-verified this pass (§5.3) |
| 42 | Service mesh / integration mesh / event mesh usage distinctions | ⚠-knowledge | Consistent industry usage, not a formal standard (§5.6) |
| 43 | Mainframe batch integration patterns (MQ trigger messages, file handoff with event triggers, CDC as batch replacement) | ⚠-knowledge | Standard IBM integration practice; the product facts (MQ for z/OS, CICS) are already ✅ per #6 and #34 (§5.7) |

---

## 10. What Could Not Be Verified

- **Sonic Software's founding year** — the task brief said "founded ~2000, acquired by Progress Software 2007." Verification confirmed Sonic was *already* a Progress Software operating company by October 2002 (SEC filing ✅) — so the "acquired 2007" framing is not supported; the exact founding year (~2000) and the exact Progress acquisition date could not be confirmed from a primary source this pass. Both are flagged ⚠.
- **TIBCO "founded 1985"** — TIBCO's own materials and industry usage say "founded in 1985"; the primary-source record is more precise: Teknekron Software Systems (the lineage) was founded in 1985, while TIBCO Software Inc. itself was founded in 1997 as a Reuters-backed subsidiary. The "1985" claim is therefore ✅-as-lineage, ⚠-as-company-founding-date.
- **Software AG's own acquisition by IBM** — widely reported (announced July 2023, completed ~2024, ~US$2.3B); not re-verified from a primary source this pass. Flagged ⚠-knowledge; the webMethods facts themselves (1996 founding, US$546M Software AG acquisition 2007) are ✅.
- **JMS 1.0 original release date (1998)** — the JSR 914 / June 2001 maintenance-release dates are ✅; the exact original JMS 1.0 publication date was not re-verified this pass.
- **Market-share percentages** — no market-share figures (Gartner MQ shares, iPaaS revenue rankings, "largest iPaaS vendor" claims) are asserted anywhere in this guide; such figures are analyst estimates that change yearly and could not be responsibly verified (§4.3).
- **Alliance Lite2/Alliance Cloud details, gpi January 2017, FIN-live 1977** — not re-derived here; they are verified in the sibling SWIFT guides' ledgers (cross-ref §6.3).
- **IBM MQ on z/OS launch date** — the family-level facts (December 1993 launch; IBM MQ for z/OS is a family member) are ✅; the specific MQSeries-for-MVS first-release date was not pinned down this pass.

---

## 11. Glossary

- **Middleware** — the software layer between applications: transport, mediation, and exposure. §1.3.
- **MOM / message-oriented middleware** — middleware built around messages and queues (IBM MQ, JMS brokers). §2.2.
- **ESB / enterprise service bus** — central mediation hub for routing, transformation, orchestration, and protocol conversion (Chappell, 2004). §2.3, §3.3.
- **SOA / service-oriented architecture** — the 2000s architecture of exposed, orchestrated services; the ESB's native worldview. §2.3.
- **iPaaS** — integration platform as a service: cloud-delivered connectors and low-code flows (Gartner term, 2011). §2.4, §3.1.
- **API gateway / API management** — the governed front door for request/reply APIs: security, rate limiting, routing, versioning, portal. §3.1.
- **Event backbone** — a durable, replayable event log (Kafka-class) as the system of record for what happened. §2.4, §3.1.
- **BPM / workflow orchestration** — modeling and executing long-running, stateful, often human-involved processes. §3.1.
- **CDC / change data capture** — capturing data changes at the source (logs/outbox) to publish as events. §7.3.
- **Outbox (transactional outbox)** — writing events atomically with state so the stream can't disagree with the ledger. §7.3; [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) §10.
- **Integration mesh** — decentralized integration endpoints with centralized governance (pattern). §5.2.
- **Composable integration** — reusable, discoverable integration building blocks assembled per capability. §5.3.
- **Strangler fig** — incremental replacement: wrap, migrate function by function, retire. §7.4; sibling patterns guide §6.
- **ACL / anti-corruption layer** — the translation layer that isolates your model from a system you can't change (the core). §6.2.
- **CICS** — IBM's mainframe online transaction monitor (1969), hosting the COBOL core transactions. §5.4.
- **SWIFT estate** — the regulated connectivity layer (Alliance Access/Gateway, SWIFTNet, RMA) for cross-border messaging. §6.3.
- **FIX** — the Financial Information eXchange protocol, the trading-floor execution language. §6.4.
- **DDS** — OMG's brokerless data-distribution middleware for low-latency market data. §6.4.
- **The integration fabric** — the layered whole of an estate's integration planes (transport, event, API, process, integration, platform) governed as one. §1, §3.4.
- **EIP / Enterprise Integration Patterns** — the named catalogue of integration patterns (Hohpe & Woolf, 2003): channels, routers, translators, endpoints; implemented by Camel and the ESB generation. §2.3; [camel_camelk_research.md](camel_camelk_research.md).
- **Broker** — the message-routing/queuing engine at the heart of MOM (MQ queue manager, RabbitMQ, Kafka broker). §2.2.
- **Adapter / connector** — a prebuilt bridge to a specific system or protocol (SAP adapter, Salesforce connector); the iPaaS's core asset. §4.2.
- **DLQ / dead-letter queue** — the queue where poison messages (undeliverable or unprocessable) are parked for inspection instead of redelivering forever. §3.5.
- **Saga** — a distributed transaction as a sequence of local transactions with compensating actions, orchestrated or choreographed. §3.9.
- **SLA / SLO** — service-level agreement/objective: the contracted versus the targeted reliability and latency of a plane or a flow. §3.7, §7.4.
- **Canonical model** — the enterprise-wide message schema that N systems translate to instead of N² pairwise formats; survived the ESB as the schema registry. §3.6.
- **Schema Registry** — the contract store for event topics: versioned, validated schemas that producers and consumers must agree on. §3.8; [../banking/kafka_guide.md](../banking/kafka_guide.md) §6.

---

## 12. Cross-References and Further Reading

**Sibling guides (technology/):** [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) (patterns, styles, strangler/ACL), [data_integration_frameworks_guide.md](data_integration_frameworks_guide.md) (CDC/ETL/API frameworks), [camel_camelk_research.md](camel_camelk_research.md) (Apache Camel/Camel-K), [event_stream_processing_guide.md](event_stream_processing_guide.md) (stream processing), [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) (delivery semantics, outbox), [kafka_alternatives_guide.md](kafka_alternatives_guide.md) (broker head-to-head), [dds_guide.md](dds_guide.md) (brokerless comparison), [ibm_cloud_guide.md](ibm_cloud_guide.md) (OpenShift, Cloud Paks, MQ/Event Streams, hybrid cloud), [ai_platform_engineering_guide.md](ai_platform_engineering_guide.md) (platform operating model).

**Banking guides (../banking/):** [payment_rails_guide.md](../banking/payment_rails_guide.md) (rails, ISO 20022, SWIFT founding), [swift_alliance_access_guide.md](../banking/swift_alliance_access_guide.md) and [swiftnet_fileact_guide.md](../banking/swiftnet_fileact_guide.md) (the SWIFT estate), [iso_20022_core_processes_guide.md](../banking/iso_20022_core_processes_guide.md) (CBPR+, UETR), [kafka_guide.md](../banking/kafka_guide.md) (the event backbone and Cymbal Bank event-platform design), [fix_protocol_guide.md](../banking/fix_protocol_guide.md) (trading floor), [core_banking_systems_guide.md](../banking/core_banking_systems_guide.md) (the cores), [payments_hub_guide.md](../banking/payments_hub_guide.md) (hub patterns), [capital_markets_architecture_guide.md](../banking/capital_markets_architecture_guide.md) (post-trade estate), [mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md) (regulatory context).

**Primary sources fetched this pass:** Wikipedia (CORBA, WebLogic Inc., BEA Systems, IBM WebSphere Application Server, IBM MQ, MuleSoft, webMethods, TIBCO Software, Vivek Ranadivé, Apigee, Boomi LP, Workato, WSO2, Informatica, Apache Camel, BizTalk Server, Kubernetes, Cloud Native Computing Foundation, OpenShift, CICS, Talend [fr]), omg.org, jcp.org (JSR 914), konghq.com (About/Our Story), qlik.com press releases, SEC EDGAR filings (Software AG/webMethods press release; Progress Software 14A; Talend MOU), Reuters/CNBC/NYT (webMethods, Software AG), TechCrunch (Apigee, Workato), cncf.io, learn.microsoft.com (BizTalk migration), Microsoft Tech Community. Sibling-guide ledgers: legacy_integration_patterns_guide.md (§11), ibm_cloud_guide.md (§10–§11), kafka_guide.md (verification ledger), swift_alliance_access_guide.md (verification ledger).

---

*End of guide — the final word: one estate, one plan, the integration fabric.*
