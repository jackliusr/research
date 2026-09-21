# Redfish: The Standard for Out-of-Band Infrastructure Management

**Jack Liu Shurui, Solution Architect**

*Deep-research guide — DMTF Redfish. Research pass completed 21 September 2026. Current specification version described throughout: Redfish Specification DSP0266 v1.25.0 (published 14 September 2026); schema and registries as of Redfish Release 2026.2. All document numbers, versions and dates in this guide were read from the DMTF document register or from DMTF's own publications, not from secondary summaries.*

---

## Table of Contents

1. [Overview, identity and decoder](#1-overview-identity-and-decoder)
2. [The problem it was built to replace](#2-the-problem-it-was-built-to-replace)
3. [The REST resource model](#3-the-rest-resource-model)
4. [Schema, conformance and the interoperability problem](#4-schema-conformance-and-the-interoperability-problem)
5. [The security model](#5-the-security-model)
6. [Versioning and the state of the standard](#6-versioning-and-the-state-of-the-standard)
7. [The implementations](#7-the-implementations)
8. [The management-plane practice this enables](#8-the-management-plane-practice-this-enables)
9. [The architecture of the management plane itself](#9-the-architecture-of-the-management-plane-itself)
10. [The regulated-enterprise and banking angle](#10-the-regulated-enterprise-and-banking-angle)
11. [The Cymbal Bank worked example](#11-the-cymbal-bank-worked-example)
12. [Anti-patterns](#12-anti-patterns)
13. [Claims audit](#13-claims-audit)
14. [What Could Not Be Verified](#14-what-could-not-be-verified)
15. [Glossary, cross-references and closing summary](#15-glossary-cross-references-and-closing-summary)

---

## 1. Overview, identity and decoder

**Redfish is a standardised, REST-based, vendor-neutral interface for managing servers and their components out of band.** A client speaks HTTPS, follows hyperlinks from a service root, and reads and writes a JSON data model whose shapes are published as schemas by a standards body. That is the whole of it. The interface is *out of band* in the precise sense the specification uses: it is served by an embedded management controller that runs independently of the host operating system, so the control path exists when the OS does not.

The standards body is the **DMTF** (Distributed Management Task Force), a not-for-profit industry consortium — a real consortium with named member companies, a document register, numbered publications and formal review cycles, not a marketing alliance. The Redfish work is done in the **Redfish Forum**, one of DMTF's forums. DMTF's own material describes the organisation as "a not-for-profit association of industry members dedicated to promoting enterprise and systems management and interoperability", and its specifications carry the standard clause that "as DMTF specifications may be revised from time to time, the particular version and release date should always be noted." That clause is not legal boilerplate here; it is the single most important operational fact about the standard, and section 4 builds directly on it.

**The framing thesis.** Redfish did not replace one command set with a better command set. It replaced a *command protocol* with a *resource model* — and that shift is the entire point. Under the predecessor, the protocol defined verbs, and the meaning of any given verb was fixed by the protocol author and then extended, per vendor, by whatever the vendor's firmware team decided to add. Under Redfish, you do not ask a device to perform an operation; you read and write the *representation of the device's state*. A power-on is not a command with a vendor-specific byte layout — it is a `POST` to an action URI that the service advertised in the resource you were already reading. The verbs are HTTP; the nouns are the standard's; the discovery mechanism is the links in the payload. This is why the same client code can, in principle, talk to a Dell PowerEdge, an HPE ProLiant and a Lenovo ThinkSystem without three adapters — and also why "in principle" is doing real work in that sentence, which is the subject of section 4.

### Decoder table

| Term | What it actually means | Where it is defined |
| --- | --- | --- |
| **DMTF** | Distributed Management Task Force. The not-for-profit consortium that owns and publishes Redfish. Develops it through the Redfish Forum. | dmtf.org; stated in the Foreword of DSP0266 |
| **Out-of-band management** | Management of a system through a path that does not traverse the host operating system, CPU and usually not the host network stack. Redfish's Scope clause defines the specification as defining "an interoperable, multivendor, remote, and out-of-band capable interface". | DSP0266, Scope |
| **Baseboard management controller (BMC)** | The embedded device that serves the out-of-band interface. DSP0266 §3.1.1 defines it as an "embedded device or service… typically, an independent microprocessor or system-on-chip with associated firmware in a computer system that completes out-of-band systems monitoring and management-related tasks". | DSP0266 §3.1.1 |
| **Service root** | The single well-known entry point of a Redfish service, conventionally `/redfish/v1/`. A hypermedia API: everything else is reached by following links from here. DSP0266 §3.3.17 defines it; the specification refers to the "hypermedia API service root" as a design property. | DSP0266 §3.3.17, §5.4.4 |
| **Resource and collection** | A resource is an addressable object with a type and properties. A collection is a resource whose members are other resources, exposed via a `Members` array of `@odata.id` references, with optional `Members@odata.count` and `Members@odata.nextLink` for paging. | DSP0266 §3.3.13, §3.3.14, §9.6.11–§9.6.12 |
| **Schema and profile** | A schema defines the shape of a resource type and its properties (published in DSP8010). A profile defines a *minimum set of functionality* an implementation must expose to be considered interoperable for a purpose (published under DSP8013, structured by DSP0272). Schema = "is this legal?"; profile = "is this sufficient?" | DSP8010, DSP8013, DSP0272 |
| **OEM extension** | The sanctioned escape hatch. Standard resources carry an `Oem` property into which vendors place vendor-namespaced data, plus OEM resource types, OEM registries, OEM URIs and OEM actions. Conformant implementations use it; the consequence is covered at length in section 4. | DSP0266 §9.6.15, §9.8 |
| **Session authentication** | Creating a login session via `POST /redfish/v1/SessionService/Sessions`, receiving an `X-Auth-Token` opaque token and a `Location` header, and using the token on subsequent requests. One of the two authentication mechanisms DSP0266 requires services to support. | DSP0266 §13.3.4 |
| **The specification document** | **DSP0266, _Redfish Specification_** — the normative protocol document. Currently v1.25.0, published 14 September 2026. The data model lives in DSP0268; the schemas in DSP8010; the registries in DSP8011. | dmtf.org document register |
| **IPMI** | Intelligent Platform Management Interface — the predecessor out-of-band management interface that Redfish was built to displace. DSP0266 §3.1.2 gives only the expansion; the specification does not describe it. | DSP0266 §3.1.2 |
| **Vendor implementations** | The vendor-specific firmwares that expose a Redfish service: HPE iLO, Dell iDRAC, Lenovo XClarity Controller, Supermicro BMC, Cisco, Fujitsu iRMC, Huawei iBMC, Inspur, and — underneath many brand names — AMI MegaRAC. Section 7. | Section 7 of this guide |

### Boundary of this guide

This guide owns **the Redfish standard itself**: the problem it replaces, its resource model, its schema and conformance machinery, its security model, its versioning, its implementations with dated statuses, the management-plane practice it enables, and the regulated-enterprise reading of that practice.

It does **not** re-derive adjacent material. Specifically:

- The nearest *use case* is out-of-band management of GPU servers — see [on_prem_llm_deployment_guide.md](on_prem_llm_deployment_guide.md), which names the out-of-band management plane as a required layer. This guide explains the standard that plane is built on; that guide explains what the workload needs.
- Facilities, power, cooling, racks and data-centre design belong to [data_center_guide.md](data_center_guide.md), [singapore_data_centres_guide.md](singapore_data_centres_guide.md) and [l2_design_financial_data_centers_guide.md](l2_design_financial_data_centers_guide.md). The management network described in section 9 is a logical network in their physical plant.
- The compute — GPU, accelerator, HAMi — is owned by the GPU and NVIDIA guides.
- Vendor firms are owned by their vendor guides (for example [nec_asia_pacific_guide.md](nec_asia_pacific_guide.md)); this guide names vendors only as implementers of a standard.
- Platform guides such as [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) own their platforms. Where such a platform consumes Redfish (OpenStack Ironic does, via Sushy), that is noted here and detailed there.
- The security *discipline* — supply-chain provenance, secure development, defensive architecture — is owned by [cybersecurity_guide.md](cybersecurity_guide.md), [security_by_design_guide.md](security_by_design_guide.md), [secops_guide.md](secops_guide.md), [beyond_zero_enterprise_security_guide.md](beyond_zero_enterprise_security_guide.md) and [zero_trust_network_architecture_guide.md](zero_trust_network_architecture_guide.md). Section 5 and section 10 of this guide apply that discipline to one interface.

---

## 2. The problem it was built to replace

### Why out-of-band management exists at all

The requirement is not exotic. It is the requirement for **a control path that works when the OS does not**. A production server can fail in ways that leave the operating system unable to help: the OS will not boot; the OS booted but the network stack is misconfigured; the OS booted but is thrashing so hard that no in-band agent will answer; the machine is powered off and needs to be powered on. In every one of those cases the in-band management tools — SSH, agents, configuration management — are unavailable precisely when they are most wanted, because they are the thing that is broken. Out-of-band management moves the control path into a small independent computer on the motherboard with its own processor, its own firmware, its own network interface and (usually) its own power state. That is the BMC of the decoder table, and every serious server platform has shipped something in that role for decades.

Redfish's own Scope clause states the ambition directly: the specification defines "the required protocols, data model, behaviors, and other architectural components for an interoperable, multivendor, remote, and out-of-band capable interface", aimed at "cloud-based and web-based IT professionals' expectations for scalable platform management."

### The predecessor's architecture

The predecessor is **IPMI**, the Intelligent Platform Management Interface — an out-of-band interface designed at the turn of the century and described, in DMTF's own glossary, only by its expansion (DSP0266 §3.1.2). Its architecture was:

- A **binary, message-based protocol** over a defined byte-level transport, with commands identified by numeric NetFn/command pairs and fixed or semi-fixed payload layouts.
- A companion specification, **DCMI** (Data Center Management Interface), layered on top to standardise a subset of server-oriented telemetry and control that base IPMI left to the vendor. OpenBMC's own feature list advertises "Full IPMI 2.0 Compliance with DCMI" — which confirms the existence and pairing of the two specifications, and confirms that the ecosystem treats compliance with both as a meaningful, checkable property.
- **"IPMI over LAN"** as the common remote carrier: the same binary protocol tunnelled in UDP datagrams, so that a BMC reachable on an Ethernet segment could be driven from a management station.

The architectural consequence is the one that matters. Because the protocol was a *command set*, everything the standard authors had not anticipated had to be added as **vendor extensions** — vendor-specific commands, vendor-specific payloads, and in practice vendor-specific client libraries to interpret them. The specification could not describe what a device *is*; it could only describe messages it might accept.

### What was wrong with a command-based, vendor-extended, network-exposed protocol

Setting aside for a moment the question of what is documented versus what is folklore (see the honesty note below), four structural objections follow from the architecture itself:

1. **No data model, therefore no discovery.** A client could not ask a device "what do you contain?" and receive a typed answer. Discovery meant a catalogue of tests, each knowing which vendor's command returned which layout. Every new hardware generation was a client-library update.
2. **The extension mechanism was the protocol.** Where Redfish confines vendor divergence to a namespaced `Oem` property inside an otherwise standard shape, the predecessor had no such containment: divergence was the norm in the protocol itself. Two "IPMI-compliant" servers could differ in everything a management tool actually needed.
3. **The protocol was designed for a different threat environment.** A UDP, unauthenticated-by-default, request/response protocol with weak integrity options, exposed on a network interface that in practice was often reachable from the same segments as production traffic, is a hard thing to secure retroactively. The protocol's own design assumptions belong to a period when a management LAN was assumed to be a trusted island.
4. **It was not built for scale.** Fleet-sized, API-driven automation wants stateless request/response over a normal web stack, tooling that already exists (HTTP clients, TLS, OIDC-adjacent token flows, JSON parsers), and payloads that can be validated against published schemas. A binary command protocol gives you none of that.

### The industry's stated requirements for a successor

DMTF's design properties for Redfish, read from DSP0266 §5.4, read as a specification of what was wrong with the predecessor — each line answers a deficiency above:

- **REST-based interface** (§5.4.1) — use the web stack that already exists.
- **Data-oriented** (§5.4.2) — describe state, not commands.
- **Separation of protocol from data model** (§5.4.3) — the protocol (DSP0266) and the model (DSP8010/DSP0268) are separately versioned documents, so the model can grow without forking the protocol.
- **Hypermedia API service root** (§5.4.4) — discovery is following links, not consulting a compatibility table.
- **OpenAPI v3.0 support** (§5.4.5) and **OData conventions** (§5.4.6) — interoperate with existing generic client tooling.
- Plus, from §5.5: synchronised and **asynchronous operation support** (§5.5.1) for operations that take minutes, an **eventing mechanism** (§5.5.2) instead of polling-only, **actions** (§5.5.3) for the legitimate non-CRUD operations, and **service discovery** (§5.5.4).

DMTF also publishes a white paper whose title states the thesis plainly: _End-to-End Interoperable Management: The Standards Requirement for Modern IT_ — a direct argument that the requirement is interoperability, and that a vendor-extended command protocol cannot deliver it.

### An honesty note on the predecessor's failures

This is where most write-ups become useless, because the predecessor's weaknesses are repeated far more often than they are sourced. Separating what is documented from what is merely widely said:

**Documented and citable:**

- **The predecessor's promoters put its end-of-line on the record — and named the successor.** DMTF's own IPMI standards page carries "A Joint Message from the IPMI Promoters (Dell, Hewlett Packard Enterprise, NEC, Intel Corporation)", **issued June 2020**: "No further updates to the IPMI specification are planned or should be expected. The IPMI promoters encourage equipment vendors and IT managers to consider a more modern systems management interface which can provide better security, scalability, and features for existing datacenters and be supported on the requisite platforms and devices." The message then names **DMTF's Redfish standard** as "an example of one such interface", and carries two notes: the statement "applies only to the IPMI specification, and should have no impact on existing IPMI implementations", and that Redfish uses the IPMI Group ID value of **0x52** (ASCII 'R'). Source: DMTF's IPMI standards page (`dmtf.org/standards/ipmi`), read 21 September 2026 — primary, quality **A**. This is the strongest sourced fact available for this section, and it is not folklore: the predecessor's own promoters told the industry to move to a modern interface and pointed at Redfish by name, while explicitly declining to declare the incumbent dead.
- The predecessor's identity and pairing are documented: DSP0266 §3.1.2 defines IPMI; OpenBMC's own project documentation claims "Full IPMI 2.0 Compliance with DCMI", establishing both the version line and the companion DCMI specification as real, named artefacts.
- That IPMI remains present and maintained inside modern BMC stacks is documented: OpenBMC supports it as a first-class feature alongside Redfish, and Lenovo's XClarity Controller documentation contains a chapter describing "the IPMI interface supported by the XClarity Controller". The predecessor did not vanish; it persists beside its successor.
- That the industry wanted a data-model-based replacement is documented by DMTF's own design-property list (§5.4) and by its interoperability white paper.

**Widely repeated, but which this pass could NOT source to a named authority:**

- Specific numeric claims about the predecessor's protocol overhead, packet sizes or comparative performance versus Redfish. No primary measurement source was located in this pass.
- Specific claims that the predecessor's message specification was *abandoned* or formally *withdrawn*. There is no withdrawal notice, and the June 2020 promoters' statement explicitly says it "should have no impact on existing IPMI implementations" — so an abandonment claim remains unsupported. What *is* documented is the weaker and more precise claim above: no further **updates** to the specification are planned. Say that; do not say "abandoned".
- Enumerations of the "well-known" predecessor vulnerabilities as a class. Named CVEs exist in BMC firmware generally (section 5), but attributing any specific published vulnerability to the predecessor protocol itself, rather than to a particular firmware implementation of it, is not something this pass substantiated.

The honest version: the predecessor's problems that matter *for architecture* follow from its command-based, vendor-extended design, and are visible in the shape of its successor's design goals — while the transition itself is documented at source by the promoters' June 2020 statement rather than by folklore. What remains genuinely unsourced in this pass is the *measurement* folklore: comparative overhead and performance numbers, and any attempt to attribute a specific published vulnerability to the protocol itself rather than to a particular firmware implementation built on it.

---

## 3. The REST resource model

### Resources and relationships rather than commands

A Redfish service is a tree of JSON documents. Every document has a type (`@odata.type`), an identity (`@odata.id`), and properties. Where a property refers to another resource, it holds a link — an object containing an `@odata.id` — and usually a convenience `@odata.navigationLink`. Nothing is addressed by constructing a path from a specification table; you *discover* the path by reading a parent. This is what "hypermedia API" means in §5.4.4, and it is why a client that hard-codes paths is fragile by design.

DSP0266 §9.6 fixes the properties every resource carries, and a client author should treat this list as the contract:

- `@odata.id` (§9.6.2) — the resource's URI, unique within the service.
- `@odata.type` (§9.6.3) — the resource type, in the form `#Type.vMajor_vMinor_vErrata.Type`.
- `@odata.etag` (§9.6.4) — for optimistic concurrency on writes.
- `@odata.context` (§9.6.5) — the metadata context.
- `Id` (§9.6.6), `Name` (§9.6.7), `Description` (§9.6.8), `MemberId` (§9.6.9) — the human and machine identification fields. `Id` and `Name` are required; empty read-only strings are an anti-pattern the DMTF validator explicitly warns about (an absent `SerialNumber` should be *omitted*, not `""`).
- `Members` (§9.6.11) with `Members@odata.count` (§9.6.10) and `Members@odata.nextLink` (§9.6.12) — collections.
- `Links` (§9.6.13) — relationships expressed as references to other resources.
- `Actions` (§9.6.14) — the non-CRUD operations the resource supports.
- `Oem` (§9.6.15) — the vendor extension point.
- `Status` (§9.6.16) — `State` and `Health`, the health vocabulary the whole model shares.

### The service root and how a client traverses it

The client begins at the service root, conventionally `/redfish/v1/`. This is the only path a client is entitled to assume. Everything else is a link in a document the client has already read. The service root identifies the device and the locations of each service, and — critically for section 4 — **the service root and the metadata document are among the handful of resources a service may expose without authentication** (§13.3.2.1).

### Principal resource types a practitioner meets

| Resource type | Typical URI (as it appears in the specification's own examples) | What it is for |
| --- | --- | --- |
| `ServiceRoot` | `/redfish/v1/` | Entry point; holds links to every service. `ProtocolFeaturesSupported` advertises optional protocol capabilities. |
| `ComputerSystem` (`Systems`) | `/redfish/v1/Systems/1` | The host. Power state, boot configuration, `Bios`, `Processors`, `Memory`, `EthernetInterfaces`, `Storage`, `LogServices`, `VirtualMedia`, graphical console. |
| `Chassis` | `/redfish/v1/Chassis/A88` | The physical enclosure. Thermal, power, sensors, assemblies. Parent of subordinate resources such as `/redfish/v1/Chassis/A88/Assembly`. |
| `Manager` (`Managers`) | reached from the service root's `Managers` collection | The BMC itself as a manageable object: its network protocol settings, its own firmware, the systems and chassis it manages. |
| `AccountService` | `/redfish/v1/AccountService` | Accounts, roles (`/redfish/v1/AccountService/Roles/Administrator`), multi-factor and authentication configuration (`HTTPBasicAuth`, `MFA`). |
| `SessionService` | `/redfish/v1/SessionService`, sessions at `/redfish/v1/SessionService/Sessions` | Login sessions and their lifecycle. |
| `EventService` | `/redfish/v1/EventService`, subscriptions at `/redfish/v1/EventService/Subscriptions` | Event subscription management; push delivery to a client-supplied destination. |
| `UpdateService` | `/redfish/v1/UpdateService` | Firmware update; `FirmwareInventory` for what is installed. Multipart push targets appear as a `MultipartHttpPushUri` property. |
| `TaskService` / task monitors | `/redfish/v1/TaskService/TaskMonitors/<TaskMonitorId>` | Long-running operation bookkeeping. The task monitor URI is how a client follows an asynchronous operation to completion. (`JobService` is sometimes referenced as a parallel concept, but **no separate `JobService` resource type was confirmed** in the specification or schema index — see section 13.) |
| `CertificateService` / `Certificate` | certificates reachable via `/redfish/v1/CertificateService` and `CertificateLocations` | TLS and client certificates, their replacement and renewal. |
| `Storage` and its children | `/redfish/v1/Systems/1/Storage/SATAEmbedded/Volumes/2` | Controllers, drives, volumes as first-class resources. |
| `NetworkAdapter`, `NetworkDeviceFunction`, `EthernetInterface` | subordinate to the system or adapter | The NICs, their functions and their interfaces. |
| `LogService` and entries | `/redfish/v1/Systems/1/LogServices/FaultLog/Entries` | On-board logs, readable and expandable. |
| `TelemetryService`, `PowerSubsystem`, `ThermalSubsystem`, `Sensor` | subordinate to chassis or system | Metric and sensor exposure (see the Redfish Release 2026.2 additions in section 6). |

### Collections and members

Every collection answers `GET` with an envelope containing `Members` (links), and — where the service supports it — `Members@odata.count` and `Members@odata.nextLink` so a client can page a large fleet's worth of components without `GET`-ing an unbounded array. A `POST` to a collection URI creates a member; a `DELETE` on a member removes it. The specification permits a client to `POST` to either the collection URI or to a `.../Members` subordinate URI where the service supports it (for example, adding a subscription at `/redfish/v1/EventService/Subscriptions`).

### The ACTION pattern for non-CRUD operations

Not everything is create/read/update/delete. "Power cycle this system" and "set this boot override" are operations, not states. Redfish's answer is the **action**: an object inside the resource's `Actions` property, carrying a `target` URI, which the client invokes with `POST`. The specification's own example shows the pattern in the payload it returns from `GET /redfish/v1/Systems/1`:

```json
"Actions": {
  "#ComputerSystem.Reset": {
    "target": "/redfish/v1/Systems/1/Actions/ComputerSystem.Reset",
    "ResetType@Redfish.AllowableValues": [ "On", "ForceOff", "GracefulRestart" ]
  }
}
```

and the invocation is a `POST` to `/redfish/v1/Systems/1/Actions/ComputerSystem.Reset` with a body carrying the parameter:

```json
{ "ResetType": "GracefulRestart" }
```

The important structural point is `@Redfish.ActionInfo`. Because "supported parameters can differ among vendors and even among resource instances" — DMTF's own words in the `ActionInfo` schema description — the specification does not ask a client to guess. A resource may advertise an action-info document that enumerates the parameters, their types and their allowable values, so a generic client can discover how to invoke an action it has never seen. This is the resource model doing for operations what `Members` does for containment. Where a service does not provide it, the `...@Redfish.AllowableValues` annotation on the action parameter is the fallback.

A related pattern is the **deep operation**: a `POST` to a `.Deep` form of a collection URI, such as `POST /redfish/v1/AccountService/Roles.Deep`, which applies a change across many members in one request rather than iterating. It is the standard's answer to "configure a hundred things atomically".

### Eventing: push versus poll

Two facilities, and the difference matters operationally.

- **`EventService` and subscriptions.** The client `POST`s a subscription to `/redfish/v1/EventService/Subscriptions`, supplying a destination URI, and the service `POST`s event payloads to that destination as things happen. This is push. It is the only sane way to learn about a failing fan in a timely fashion across a large fleet, and it is also a liability: the BMC now holds a credential-bearing outbound destination, which is a section 9 and section 12 topic.
- **Polling.** Any telemetry or sensor resource can simply be `GET`-ed on an interval, optionally with `$expand` to collapse a navigation hop into one response. The specification's own example is `GET /redfish/v1/UpdateService/FirmwareInventory?$expand=.` — get the whole inventory, with members inlined, in one request.

The specification's own frame: §5.5.2 lists an "eventing mechanism" as a design property, distinct from §5.5.1's synchronous and asynchronous operation support. Eventing is how the standard intends a client to learn about change; polling is the fallback when a subscriber is unreliable or when the security posture forbids inbound delivery.

### A walked traversal: from service root to a concrete component

*Every URI below appears in DSP0266 v1.25.0 or in the schema index; none is invented.*

1. `GET /redfish/v1/` — the service root. The response carries `Systems`, `Chassis`, `Managers`, `AccountService`, `SessionService`, `EventService`, `UpdateService`, `TaskService` and the rest as link properties. (The specification's own session example shows the root carrying `"SessionService": { "@odata.id": "/redfish/v1/SessionService" }` and a `Links.Sessions` reference — both `@odata.id` values, and both required to be the same session collection.)
2. `GET /redfish/v1/Systems` — the collection envelope. Read `Members[]`, or `Members@odata.count` plus `Members@odata.nextLink` if paging.
3. `GET /redfish/v1/Systems/1` — a `ComputerSystem`. Now the tree opens: `Processors`, `Memory`, `EthernetInterfaces`, `Storage`, `Bios`, `LogServices`, `VirtualMedia`, `Actions`.
4. Branch to a component. For hardware components, follow the sub-collection, e.g. `GET /redfish/v1/Systems/1/Processors` and then the member URI it returns; for storage, follow `Storage` to the collection and then a concrete member the service names, e.g. `/redfish/v1/Systems/1/Storage/SATAEmbedded/Volumes/2`; for firmware, `GET /redfish/v1/UpdateService/FirmwareInventory`.
5. Branch by relationship instead of containment. A `Chassis` may hold `Links` to the systems it contains and to the managers that manage it; a `ComputerSystem` holds a `Links` reference back to its `Chassis` and `ManagedBy`. `Links.ManagedBy` is the property that tells you *which BMC* is authoritative for a system — the first question any fleet tool must answer, and one it can only answer correctly by reading links, not by assuming.
6. Act. Power and boot control are actions on the system, e.g. `POST /redfish/v1/Systems/1/Actions/ComputerSystem.Reset` with a `ResetType`. Read-only operations use `GET` with optional `$select` and `$expand`, e.g. the specification's `GET /redfish/v1/Systems/1?$select=Name,SystemType,Status/State`.

New in v1.25.0 (Release 2026.2): **resource aliases**. A client that does not want to discover `@odata.id` values on every run can reference a resource by a user-defined alias prefixed with a tilde — the specification's example is `/redfish/v1/Systems/~MySystem/Processors/~MyGPU`. Aliases are set through a writable `@Redfish.Alias` annotation, must be unique within a collection (and across collections of the same schema type sharing a parent), and a reassigned alias emits the `AliasValueReassigned` message from the Base Message Registry. This is the standard responding to a genuine operational demand — stable names for components whose `@odata.id` is vendor- and re-enumeration-dependent — and it is brand new, so assuming it exists in a given BMC version would be exactly the unverified-claim error section 4 warns against.

---

## 4. Schema, conformance and the interoperability problem

This is the most practically useful section in the guide. It explains why "supports Redfish" is not an answer to anything, and what to ask instead.

### How DMTF publishes schema

The model is published as a set of documents with distinct jobs:

| Document | Title (exact) | What it contains |
| --- | --- | --- |
| **DSP8010** | _Redfish Schema Bundle_ | The schema definition files themselves. Normative, and normatively referenced by the Redfish Specification. |
| **DSP0268** | _Redfish Data Model Specification_ | The normative descriptions and example payloads for every schema, in one reference. Formerly titled _Redfish Schema Supplement_. |
| **DSP8011** | _Redfish Standard Registries Bundle_ | Message registries: the machine-readable catalogue of the messages a service can emit — errors, events, privilege maps. |
| **DSP2046** | _Redfish Resource and Schema Guide_ | The schema contents in a table format readable by a human. Informative. |
| **DSP2053** | _Redfish Property Guide_ | An alphabetical index of every property defined across all schema. Informative. |
| **DSP2065** | _Redfish Message Registry Guide_ | The registry definitions in human-readable form. Informative. |
| **DSP0272** | _Redfish Interoperability Profiles Specification_ | The structure and JSON document format used to define a profile — i.e. how to write a profile, not the profiles themselves. |
| **DSP8013** | _Redfish Interoperability Profiles Bundle_ | The published profiles themselves, plus the supporting schema and sample documents for authoring them. |

The schema comes in **three functionally equivalent languages**, and this is a real practical fact rather than a curiosity: DSP8010's own readme states the bundle is organised into `csdl` (OData Common Schema Definition Language, XML), `json-schema` (JSON Schema), and `openapi` (OpenAPI YAML) folders, plus `dictionaries` (Redfish Device Enablement, in Binary-encoded JSON / BEJ format). DMTF's words: "These schema definitions should be functionally equivalent, thus specifying the schema in three different languages." Practical effect: a Python or Go client can validate against JSON Schema; an enterprise toolchain that speaks OData can consume the CSDL; an API-gateway or code-generator flow can consume the OpenAPI service document. The `csdl` folder also carries `RedfishExtensions_v1.xml` and `Resource_v1.xml`, described by DMTF as containing "core definitions of Redfish".

Registries matter more than newcomers expect. A service's error and event payloads are `MessageId`s referencing a registry and version — for example `Base.1.24.0`. The **message registry** is what makes an error machine-actionable: without it, a client can only log an opaque string. DMTF's release materials emphasise the registry work as a first-class part of each release (section 6 lists the new registries added in 2026.2).

### How a client discovers what an implementation supports

Two mechanisms, layered:

1. **The metadata document.** The OData-style `$metadata` document describes the resource types the service exposes. DSP0266 §8.4 governs "OData metadata responses", and §8.4.2 covers `$metadata` specifically, including §8.4.2.1 "Referencing other schemas" and §8.4.2.2 "Referencing OEM extensions" — the latter is the standard mechanism by which a client learns that a service carries vendor extensions and where they are described. A client that reads `$metadata` can generate its own type model at runtime.
2. **The schema files served by the service.** A service can expose its schema on the service itself (a `JsonSchemaFile` resource and `JsonSchemaFile` collection), and the specification permits unauthenticated access to the metadata document for exactly this purpose (§13.3.2.1). Schema *files* published under DSP8010 are also readable offline; a tool can validate a live service's payloads against the published schema without asking the service anything.

**The advanced move** — and this is where real interoperability work happens — is to pull the service's own schema and validate against *that*, rather than against the public bundle. A conformant implementation is permitted to *modify* standard resources in defined ways (the "Schema modification rules" clause referenced by DMTF's own validator documentation), so the published schema is an upper bound on what you might see, not a guarantee of what you will see.

### What the conformance and interoperability programme actually requires

DMTF's Redfish Forum provides **open-source tools** to validate conformance, and its own release materials describe three, each with a distinct job:

| Tool | Repository | What it validates | Notes |
| --- | --- | --- | --- |
| **Redfish Protocol Validator** | `github.com/DMTF/Redfish-Protocol-Validator` | HTTP protocol behaviour against the Redfish Specification: response headers and status codes. | Requires Python 3. Its copyright header reads "Copyright 2020-2022 DMTF" — the earliest copyright of the three, and see the maintenance note in section 7. |
| **Redfish Service Validator** | `github.com/DMTF/Redfish-Service-Validator` | Payload conformance against Redfish CSDL schema. Scope is **`GET` requests only** — it does not test writes. | DMTF's release deck recommends running this **first**: "errors in payload are more likely to cause issues for end users and interoperability". |
| **Redfish Interop Validator** | `github.com/DMTF/Redfish-Interop-Validator` | A service against a **specific interoperability profile**. | Copyright 2017-2026; latest release **3.0.0 on 18 September 2026** — actively maintained. |

The important structural observation about the programme: **it is a tooling programme, not a certification mark.** A vendor runs these tools against its own service; the tools are public and the reports are the vendor's to publish or not. This pass found no DMTF-operated certification registry analogous to, say, a plug-fest certification database, and attempts to reach a formal "Redfish Conformance Program" landing page returned no content (see section 14). What that means for a buyer is important and slightly uncomfortable: **"conformant" is a vendor's claim about a self-run test suite, and the evidence is a validator report you should ask to see.** The tools being open source is what makes this checkable — you can run them yourself, and you should.

Profiles are the other half. A profile defines a minimum set of functionality: which resources must exist, which properties must be present, which actions must be supported, and to what depth. The Interop Validator consumes a profile and reports pass/fail per requirement. This is the mechanism by which "supports Redfish" could be turned into something contractual — *"supports profile X at version Y"* — and it is the single most useful question to put to a vendor.

One honest signal worth recording: **`redfish.dmtf.org/profiles` returned the literal text "Profiles coming soon."** on 21 September 2026. The profiles bundle (DSP8013) is published and the profile *specification* (DSP0272) is at v1.10.0, but the Developer Hub's profile-browsing page is not serving content. Practically: obtain profiles from the DSP8013 bundle or DMTF's document register, not from that URL.

### The OEM extension mechanism, and its consequence

DSP0266 devotes a whole clause — §9.8, "Extending the standard" — to how a vendor may diverge lawfully. The toolkit:

- **§9.8.1 Extending standard resources overview** — the permitted ways to add to a standard resource.
- **§9.8.2 OEM property format and content** — the `Oem` object: a namespaced container inside a standard resource. The `Oem` property itself is defined at §9.6.15, and the `Resource` schema's first errata note records the addition of a **"Pattern Redfish annotation for `Oem` property names"** — i.e. the naming rule is schema-enforced.
- **§9.8.3 OEM-specified object naming**, **§9.8.4 OEM resource types**, **§9.8.5 OEM registries**, **§9.8.6 OEM URIs**, **§9.8.8 OEM actions**.

Now the consequence, stated plainly. **A conformant implementation may still be non-interoperable in exactly the parts a customer depends on.** Conformance is a statement about the *shape* of the payload — does it validate against schema. Interoperability is a statement about the *content and coverage* — does it do the thing. A BMC can pass the Service Validator cleanly while:

- returning a standard property as `null` because the vendor chose not to populate it;
- implementing a capability only through an OEM action, so that a client must write a vendor branch;
- exercising its legal right to modify standard resources per the schema modification rules, so your validated parser sees a shape your schema did not anticipate;
- supporting a profile's mandatory resource but omitting an optional property that your automation relies on — legal, and fatal to your runbook.

This is not a defect in the standard. It is a deliberate design choice: a standard that forbids extension would be ignored, and the OEM mechanism at least makes divergence *discoverable* (via `$metadata` §8.4.2.2 and the `Oem` pattern) and *contained* (inside a named namespace rather than scattered through the protocol). But it means the naive reading — "it's standardised, so one client works everywhere" — is wrong in a way that will be discovered in production if not in procurement. The mitigation is not to demand zero OEM usage; it is to demand named profiles and to test on the actual hardware you are buying.

### THE THREE QUESTIONS for a vendor

Ask these, in this order, and treat a vague answer to any of them as a finding.

**Question 1 — Which specification version does your implementation support?**
The answer looks like "Redfish Specification DSP0266 v1.21.0" (current is v1.25.0, published 14 September 2026). What it guarantees: the protocol semantics of that version are implemented — authentication behaviour, query parameters, status codes, the session model. What it does **not** guarantee: anything about which *resources* exist, and nothing at all about the data model version (that is a different document, DSP8010/DSP0268, on a completely different version line). It also does not guarantee forward compatibility: a client written to v1.25.0 features (resource aliases, policies) will fail against a service that implements v1.21.0.

**Question 2 — Which schema bundle does it implement?**
The answer looks like "DSP8010 2025.2" or "2026.2" (2026.2 published 14 September 2026). What it guarantees: the resource types and properties in that bundle are the ones the service's payloads can be validated against, and the corresponding message registries from DSP8011 are what its errors and events reference. What it does **not** guarantee: that any *particular* property within those types is populated. A service implementing a 2026.2 schema can still leave a given property absent or `null`. Schema version tells you the vocabulary; it says nothing about whether the vendor used the words you need.

**Question 3 — Which interoperability profile does it conform to, and can I see the Interop Validator report?**
The answer looks like a profile name and version from the DSP8013 bundle. This is the only one of the three questions that speaks to *coverage* rather than shape. What it guarantees: the profile's mandatory resources and properties are present, so automation written against that profile can rely on them. What it does **not** guarantee: anything outside the profile. And it guarantees nothing about the vendor's *reliability* in implementing its own claim — hence the second half of the question. Ask for the report, and re-run the validator yourself on a unit you own. DMTF's own release materials rank the tools: run the Service Validator first, because payload errors break end users, before spending time on protocol conformance.

The three questions together are also the framework for the boundary negotiation in section 11 and the anti-pattern in section 12 — buying on an unqualified "supports Redfish" claim is exactly the failure these three questions prevent.

---

## 5. The security model

The specification's security content is concentrated in clause 13, which splits into transport and certificates (13.1), sensitive data (13.2), authentication (13.3) and authorization (13.4). What follows is what the specification actually requires. The framing is deliberately flat: the standard's security model is coherent, its documentation is candid about its weakest authentication mode, and the *operational* risk lives in how the interface is deployed rather than in what the specification says.

### Authentication modes the specification provides

DSP0266 §13.3.1 requires services to support **both** of the following, and permits others:

**1. HTTP Basic authentication (§13.3.3).** Defined by reference to IETF RFC 7617. "All requests that use HTTP Basic authentication shall require HTTPS." When multi-factor authentication is enabled, services **shall** reject Basic authentication for accounts not configured to bypass MFA; session-based authentication is required in that case. The service can restrict or disable Basic entirely via the `HTTPBasicAuth` property in the `AccountService` resource, and the `WWW-Authenticate` response header is only required when Basic is configured to advertise itself. Critically, the specification does not pretend this is the good option. It carries an explicit note: *"The IETF has highlighted security concerns with HTTP Basic authentication. While HTTPS is required for the usage of HTTP Basic authentication, there are other concerns implementers need to be aware of that RFC7617 documents."* **Documented design risk, stated by the specification itself: credentials in a header are replayable by anything that captures them, and the credential is the user's long-lived password.**

**2. Redfish session login authentication (§13.3.4).** The client `POST`s credentials to the session collection — `POST /redfish/v1/SessionService/Sessions` — and the response carries an `X-Auth-Token` header containing an opaque session token, a `Location` header pointing at the new `Session` resource, and the session body. Subsequent requests set `X-Auth-Token`. The specification is explicit that the token "shall be indistinguishable from random" and that the session token is distinct from the session ID: the session ID identifies the resource and is visible to an administrator; the token is known only to the client that logged in. The `POST` creating a session "shall only be supported with HTTPS"; if both HTTP and HTTPS are enabled, the redirect should go to HTTPS. The specification also recommends the service save the `Origin` header at session creation and compare it on subsequent requests — a CSRF mitigation. **This is the mode clients should use.** It replaces a reusable password with a revocable, expiring token, and it is what enables an administrator to kill a session server-side without rotating a password.

**3. Client certificate authentication (§13.3.5).** mTLS as part of the RFC 8446 TLS handshake, or CAC/PIV per NIST FIPS 201-3. If enabled, the service "shall send a client certificate request during the TLS handshake", then verify the presented certificate against the certificates in the `CertificateCollection` referenced by the `ClientCertificate` property within the `AccountService` resource's `MFA` property — and "shall check for certificate revocation before processing the request with any configured Online Certificate Status Protocol (OCSP) servers." The `RespondToUnauthenticatedClients` property inside that same `ClientCertificate` object controls whether an invalid certificate fails the TLS handshake (`false`) or is allowed through so the service can return an error or unauthenticated resources (`true`, or unsupported). **This is the strongest mode the standard defines, and it is the one that scales: a fleet with client certificates issued from a known CA does not need a per-device password inventory at all.**

### Transport

- **HTTPS/TLS is required, not optional, wherever credentials move.** Basic auth "shall require HTTPS"; session creation "shall only be supported with HTTPS".
- Services "shall use only connections that conform to TLS to transport the data between any third-party authentication service and clients" — i.e. the hop to your LDAP/AD/IdP is also in scope.
- **Certificates** (§13.1.3): "Redfish implementations shall support replacement of the default certificate if one is provided," and shall use X.509 v3 certificates per RFC 5280. Read that as the specification acknowledging, in normative language, that **the default certificate is a problem to be replaced** — a self-signed, per-unit, unmanaged certificate is the expected out-of-box state, and management is the operator's job.
- **Sensitive data** (§13.2): operations carrying sensitive data "should use HTTPS only" — the specification's example is a `SimpleUpdate` action carrying a username and password. Properties representing sensitive data "shall be `null`" in responses. Where the URI itself contains sensitive data, the service may return `404` rather than `401`/`403` "to prevent attackers from obtaining the sensitive data in the URI".

### Roles, privileges and the privilege model

§13.4 defines authorization as two parts: the **privilege model** (users → roles → privileges) and the **operation-to-privilege mapping** (which privilege an operation requires). A user is assigned **exactly one** role via the `RoleId` property on the `ManagerAccount` resource; the role is a resource in the `RoleCollection`, and its `AssignedPrivileges` (and `OemPrivileges`) arrays define what it permits.

Services **shall** support all of these standard roles (§13.4.2.1, Table 43 of the specification):

| Role name | Assigned privileges |
| --- | --- |
| `Administrator` | `Login`, `ConfigureManager`, `ConfigureUsers`, `ConfigureComponents`, `ConfigureSelf` |
| `Operator` | `Login`, `ConfigureComponents`, `ConfigureSelf` |
| `ReadOnly` | `Login`, `ConfigureSelf` |

Services **may** support additional standard roles (the specification's Table 44), and may support client-defined custom roles. Vendors may extend the model with OEM privileges and OEM roles, which "participate in the privilege model the same as Redfish standard privileges and roles"; restricted roles and restricted privileges let a vendor refine authority further. Two rules worth quoting because they are the kind of thing that gets missed: **"Two roles with the same privileges shall behave equivalently"**, and services "shall enforce the same privilege model for ETag-related activity as is enforced for the data being represented by the ETag". The `Administrator` role's `ConfigureManager` privilege is the one that matters most in a fleet context — it is the privilege that permits reconfiguring the manager itself, including its network settings, which is the privilege that can strand a device.

### Session handling, timeouts and expiry

§13.3.4.3 is worth reading verbatim in spirit, because it is unlike most token systems: **Redfish sessions are idle-timeout based, not token-expiry based.** "Unlike some token-based methods that use token expiration times, Redfish sessions time out. As long as a client continues to send requests more frequently than the session timeout period, the session remains open and the session authentication token remains valid. If the session times out, it is automatically terminated." The session-management specifics — the orphaned-session timeout value and the limit on simultaneous open sessions — are determined by the implementation, not fixed by the specification. §13.3.4.4 covers termination: a `DELETE` on the `Session` resource logs out; an administrator with sufficient privilege can delete another user's session, and can force termination of all of an account's sessions by disabling the account. Removing an `AccountType` (for example `KVMIP` or `HostConsole`) from the `ManagerAccount` should terminate the affected sessions.

**Operational consequence:** a client that holds a session open indefinitely and treats the token as permanent will eventually fail, and a client that opens a new session per request will exhaust the service's session limit. Fleet tooling must handle session re-establishment on `401` as a normal, expected event — not as an error condition.

### Account and certificate facilities

- **`AccountService`** holds the account, role and authentication configuration: `HTTPBasicAuth` to restrict Basic, `MFA` for multi-factor and client-certificate settings, and the role collection. The `AccountService` schema's `Authentication` object gained `PasswordSet`, `TokenSet` and `KerberosKeytabSet` in 2026.2 (v1.19.0).
- **`CertificateService`** and **`CertificateLocations`** exist so that an administrator can "locate all certificates installed on a given service" and install or replace them. `Certificate` is a first-class resource type (v1.11.0 in 2026.2).
- **External authentication.** The specification requires TLS on the path to any third-party authentication service, and supports client-certificate authentication plus MFA flows with time-based one-time passwords (including detailed rules on secret-key handling and the `Token` property in session creation). LDAP/AD integration is a service-side capability expressed through these facilities rather than mandated by name in the specification.
- **New in 2026.2:** the `AccountSecurity` message registry has moved to v1.2.0 with `InsufficientPrivilegeWithAddress` and `InvalidCredentialsWithAddress` messages — i.e. the registries now carry address information on authentication failure events, which is exactly what a SIEM needs to detect credential-based attacks against a management plane.

### The management-plane exposure question

The specification defines a secure *model*. It does not, and cannot, control deployment. The concerns below are separated by evidentiary status, deliberately.

**DOCUMENTED DESIGN RISK — what the specification itself warns about:**
- Basic authentication's inherent weakness, flagged by the specification's own normative note (above).
- The default certificate requiring replacement (normative requirement in §13.1.3, implying the un-replaced state is expected).
- The inescapable fact that a BMC is reachable on a network interface and holds credentials sufficient to control the host (section 9).

**PUBLISHED VULNERABILITIES AND AUTHORITATIVE GUIDANCE — named authorities:**
- **CISA and NSA, joint Cybersecurity Information Sheet, "Harden Baseboard Management Controllers", released 14 June 2023.** This is the authoritative, named source for the management-plane-as-attack-surface concern, and its own framing is the one to use: BMCs "are trusted components designed into a computer's hardware that operate separately from the operating system (OS) and firmware to allow for remote management and control, even when the system is shut down." The advisory states that "Hardened credentials, firmware updates, and network segmentation options are often overlooked, leading to a vulnerable BMC", and that "A vulnerable BMC broadens the attack vector by providing malicious actors the opportunity to employ tactics such as establishing a beachhead with pre-boot execution potential." NSA's own press release for the same product is titled "NSA and CISA Release Guide To Protect Baseboard Management Controllers". **Note precisely what the advisory does and does not say:** it is a hardening-guidance document about threats to BMC *implementations* and recommended actions. It does not assert a specific exploited incident, and this guide makes no such assertion on its behalf. The three mitigations the advisory names — hardened credentials, firmware updates, network segmentation — map exactly onto sections 9, 12 and 10 of this guide.
- **Vendor advisories and CVEs.** Named CVEs exist against BMC firmware implementations, and AMI's MegaRAC in particular is a widely-deployed BMC firmware product (section 7) with a public advisory history. This pass did **not** enumerate specific CVE identifiers, and none are asserted here: asserting a CVE number that has not been verified against an authoritative advisory would violate the standard this guide holds itself to. Any reader making a procurement or risk decision should query a CVE database and the vendor's own security advisory page directly.

**SPECULATION — no source found in this pass:**
- Any claim that a specific named financial institution, or any specific named enterprise, was compromised via its management plane. No source located. Not asserted.
- Any claim about the *prevalence* of BMC compromise in the wild. No source located. Not asserted.
- Any quantitative claim that most BMCs on the internet are misconfigured or default-credentialed. Widely repeated; no named authority located in this pass. Not asserted.

**What not to do with this section:** do not turn the incumbent protocol's widely-repeated weaknesses into a claim about all BMCs. The CISA/NSA advisory concerns BMC implementations generally — Redfish-serving ones included. Redfish improves the *management interface*; it does not make the BMC a hardened system, and the advisory's existence is evidence that the industry treats the BMC itself as a soft target regardless of which management protocol it speaks.

---

## 6. Versioning and the state of the standard

**Which version this guide describes, and when it was checked.** The specification version used throughout is **DSP0266 v1.25.0**, published **14 September 2026** (the document's own internal date is 2026-08-07; it supersedes v1.24.1). The schema, registry and data-model references are to **Redfish Release 2026.2**, published **14 September 2026**. All version numbers and publication dates in this section were read from `dmtf.org/dsp/DSP<number>` register pages on **21 September 2026**. A later release may exist by the time you read this; the register is the check.

### The specification version line

The version line is dense and worth understanding, because "which version" is one of the three questions in section 4 and vendors answer it inconsistently.

| Version | Published | Title as recorded in the register |
| --- | --- | --- |
| 1.0.0 | 04 Aug 2015 | Redfish Scalable Platforms Management API Specification |
| 1.0.1 | 16 Dec 2015 | Redfish Scalable Platforms Management API Specification |
| 1.0.2 | 02 May 2016 | Redfish Scalable Platforms Management API Specification |
| 1.0.3 | 14 Jul 2016 | Redfish Scalable Platforms Management API Specification |
| 1.0.4 | 30 Aug 2016 | Redfish Scalable Platforms Management API Specification |
| 1.0.5 | 03 Jan 2017 | Redfish Scalable Platforms Management API Specification |
| 1.1.0 | 08 Jan 2017 | Redfish Scalable Platforms Management API Specification |
| 1.2.0 | 22 May 2017 | Redfish Scalable Platforms Management API Specification |
| 1.3.0 | 02 Oct 2017 | Redfish Scalable Platforms Management API Specification |
| 1.4.0 | 31 Dec 2017 | Redfish Scalable Platforms API Specification |
| 1.5.0 | 14 May 2018 | Redfish Scalable Platforms Management API Specification |
| 1.6.0 | 20 Sep 2018 | Redfish API Specification |
| 1.11.0 | 15 Jun 2020 | Redfish Specification |
| 1.16.0 | 21 Sep 2022 | Redfish Specification |
| 1.19.0 | 21 Sep 2023 | Redfish Specification |
| 1.20.0 | 17 Jan 2024 | Redfish Specification |
| 1.21.0 | 16 Sep 2024 | Redfish Specification |
| 1.22.0 | 21 Mar 2025 | Redfish Specification |
| 1.23.0 | 12 Oct 2025 | Redfish Specification |
| 1.24.0 | 17 May 2026 | Redfish Specification |
| **1.25.0** | **14 Sep 2026** | **Redfish Specification (current)** |

Two observations from the register that are useful in practice. First, **the document was renamed repeatedly early on** — "Redfish Scalable Platforms Management API Specification" through v1.5.0, "Redfish API Specification" at v1.6.0, and stabilising as "Redfish Specification" from the v1.11 era. A procurement document asking for "Redfish API Specification version 1.6" is asking for a 2018 document; ask for a modern version instead. Second, the register's own titling is not perfectly consistent (v1.4.0 reads "Scalable Platforms API" without "Management"; v1.5.1 appears as "Redfish API Specification" in the same release window as v1.5.0's longer title). Read the version number, not the title, when comparing.

### Eras in the line

Reading the register and DMTF's own release overviews together, the line breaks into recognisable eras. This is interpretation, but each element is anchored in a dated publication:

- **2015–2017 — protocol first.** v1.0.0 (August 2015) establishes the REST protocol and the first schema bundle (DSP8010 1.0.0, same day). The work of this period is the protocol/data-model separation (§5.4.3) and the schema languages.
- **2018 — the model gets a reference document and the profile concept appears.** DSP0272, the _Redfish Interoperability Profiles Specification_, reaches v1.0.0 on **08 January 2018** — the mechanism that turns "conformant" into "sufficient" arrives within three years of the protocol. DSP2043's mockups bundle (the basis of the interactive public mockups at `redfish.dmtf.org/redfish/v1`) dates from this era as the schema development aid.
- **2019–2020 — data model formalised and the spec renamed.** DSP0268 appears as _Redfish Schema Supplement_ from its first release (2019.3, 01 November 2019). The Redfish Host Interface Specification (DSP0270, "in-band" / OS-based access) reaches v1.3.0 on 04 August 2020, and the specification title settles as "Redfish Specification" from the v1.11 line (15 June 2020).
- **2021–2022 — release cadence and the multi-language schema bundle.** The schema bundle moves to a regular quarterly-style release naming (2021.1, 2021.2, … 2026.2), and the bundle's readme establishes the three-language equivalence (CSDL / JSON Schema / OpenAPI).
- **2023–2024 — command-and-control features mature.** Profiles specification at v1.6.0 (23 January 2023) then v1.7.0 (17 January 2024). DSP0268 is retitled from _Redfish Schema Supplement_ to _Redfish Data Model Specification_ at 2022.2 into 2023 (register shows the 2022.1 edition — 24 May 2022 — still titled _Redfish Schema Supplement_, with the new title in place from 2022.2 onward). Host Interface Specification's last published edition, v1.3.1, dates to 21 September 2023.
- **2025–2026 — policy, aliases, and registry sophistication.** Current era; see below.

### What has changed in the current era

Redfish Release 2026.2 (September 2026) is unusually substantive, and its contents are a good read on where the standard is going. From DMTF's own release overview:

**Specification 1.25.0 additions:**
- **Resource aliases** — user-defined, implementation-independent names for collection members, addressed with a tilde prefix (`/redfish/v1/Systems/~MySystem/Processors/~MyGPU`), configured through the new writable `@Redfish.Alias` annotation. This directly addresses the operational nuisance of unstable `@odata.id` values.
- **Message referencing in registries** — a message registry can now import and reuse messages from another registry via a new `ReferencedRegistry` property, so a registry no longer has to duplicate a message. A client subscribing to either source or referencing registry sees the message as coming from the source, exactly once.
- **Asynchronous-operation enhancements** for clients that go offline during long-running operations — an acknowledgement that firmware updates and similar tasks outlive a client session.

**Registry additions (DSP8011 2026.2):**
- **New `Device` registry v1.0.0** — common events for any device or component, with messages migrated out of the Platform registry. Examples from DMTF's own slide: `PresenceMismatch`, `AssemblyDataMalformed`, `ManagementInterfaceError` ("sideband communication failures").
- **New `EventService` registry v1.0.0** — lifecycle events for subscriptions themselves: `EventSubscriptionCreated`, `EventDeliveryFailed` ("exhausted retries when sending an event"), `EventSubscriptionResumed`. **This is a notable maturity signal:** the standard now emits events about its own eventing, which is the kind of gap that only becomes visible at fleet scale.
- `ManagerEvent` v1.2.0 (redundancy messages), `Platform` v1.5.0 (memory, PCIe, thermal, power messages), `AccountSecurity` v1.2.0, `Base` v1.24.0, `Power` v1.4.0, `Update` v1.5.0 (update lifecycle messages including automatic-recovery outcomes).

**Schema additions (DSP8010 2026.2):**
- **New `Policy` and `PolicyService` schemas** — "reactions that occur when a set of conditions are met", with predefined or client-created policies. DMTF's own example is a rack power cap policy that, on a summed power reading crossing a threshold, issues a `GracefulShutdown` reset to systems and chassis. This is automation moving *into* the BMC — a strategic shift worth noting, and a security-relevant one, since it means the management plane now contains logic that can power off equipment without a client in the loop.
- **Graphical console support in `ComputerSystem` v1.29.0** — `VNC`, `RDP`, `HTML5` and `KVMIP` properties plus `GraphicalConsoleGetOneTimeURL`, `GraphicalConsoleSetOneTimePassword` and `GraphicalConsoleSetPassword` actions. Remote console had long been the least standardised part of the out-of-band surface; this begins to close it.
- **New `UpdateServiceCapabilities` v1.1.0** — properties such as `TargetsParameterSupported`, `ExcludeTargetsParameterSupported`, `ForceUpdateParameterSupported`, `StageParameterSupported` and `LocalImageParameterSupported`, i.e. the service can *declare* which update semantics it supports. Read this as the standard's answer to the "conformance is not interoperability" problem of section 4, applied specifically to the update workflow.
- 48 updated schemas in the release in total, including `PrivilegeRegistry` v1.3.0 (added `ActionPrivileges` to `OperationMap`), `Session` v1.9.0 (added `UserAuthenticationSource`), `ServiceRoot` v1.22.0 (added the `PolicyService` property and an `Alias` entry to `ProtocolFeaturesSupported`), `SoftwareInventory` v1.15.0, `Processor` v1.24.0 and `PowerSubsystem`/`PowerSupplyMetrics` (OCP high-power rack power-brake support).

### Where coverage is thin or commonly extended by OEMs

Stated as findings from the primary material, with the caveat that absence of a schema property is not proof of absence of capability (the vendor may simply have put it in `Oem`):

- **The graphical console / KVM surface** has only just been standardised (2026.2), which means essentially every deployed implementation today handles remote console through vendor-specific means. Expect divergence here for years.
- **BIOS/attribute configuration** is modelled as an *attribute registry* whose contents the vendor defines. DMTF's `AttributeRegistry` schema description says it plainly: "The attribute registry is specific to an implementation or product. The attributes and property names are not standardized." This is the single largest standardised-shaped-but-not-standardised area a practitioner will meet, and it is where configuration-baseline tooling (section 8) always ends up with vendor branches.
- **Firmware update semantics** were partially standardised but vary in practice between `SimpleUpdate`, multipart push (`MultipartHttpPushUri`), and OEM update paths; DSP0266 §13.2 mentions `SimpleUpdate` carrying credentials, indicating the action's shape is standard while its behaviours are not.
- **Redfish Device Enablement (RDE, DSP0218)** covers mapping Redfish into PLDM for device-level enablement — relevant for components on a sideband fabric (the `Device` registry's `ManagementInterfaceError` message is a sideband-failure event), and a domain where standard coverage is newer than for the server-level model.
- **The profile-browsing developer experience is thin.** `redfish.dmtf.org/profiles` served "Profiles coming soon." on 21 September 2026. The documents are published; the hub's profile navigation is not.

### Adoption picture, with sources

Honest assessment, distinguishing what is documented from what is asserted:

**Documented:** The standard is developed by a consortium whose member companies appear in the specification's own acknowledgements (DSP0266 v1.25.0's contributor list includes named individuals from Dell Technologies, Google, NVIDIA, SuperMicro and Hewlett Packard Enterprise). The vendor implementations exist and are documented by their vendors (section 7). OpenBMC — the Linux Foundation project — implements Redfish through bmcweb. OpenStack implements a Redfish client in Sushy and sushy-tools. DMTF maintains client libraries and validators. Tools that consume Redfish exist across the ecosystem.

**Not documented in this pass, and therefore not asserted:** any market-share figure, any claim that a given percentage of servers ship with Redfish enabled, any claim that Redfish has "replaced" IPMI in the field (OpenBMC's and Lenovo's continued IPMI support argue against it), or any claim about Redfish adoption outside servers (Swordfish, the storage-profile sibling built on the same schema machinery, is referenced in DMTF's own schema index page text but is a separate standard owned elsewhere and is out of this guide's boundary).

---

## 7. The implementations

Two rules govern this section. First, every support statement is **dated and attributed** to the vendor's own documentation, with the source named — and where the vendor's public statement is only "supports Redfish", this guide says exactly that rather than inferring a specification version. Second, **dormant projects are flagged as dormant**; nothing here is presented as a live option without evidence of current activity.

### Server-vendor BMC implementations

| Vendor / product | What the vendor's own documentation says | Source and date checked |
| --- | --- | --- |
| **HPE — iLO (Integrated Lights-Out)** | HPE publishes a dedicated server-management portal with Redfish services documentation for iLO generations. **This pass could not retrieve the specific page**: the URL attempted (`servermanagementportal.ext.hpe.com/docs/redfishservices/ilos/`) returned a 404. The existence of HPE's Redfish documentation portal is evidenced by DMTF's specification acknowledgement list naming HPE contributors, but **no HPE support statement with a carrier, generation or firmware version is asserted here.** | Not verified — 21 Sep 2026 (see section 14) |
| **Dell — iDRAC** | Dell publishes Redfish API documentation and tooling, and Dell staff are prominent in DMTF's validator and specification work (the Interop Validator's commit history shows Dell-authored commits in 2026; Dell contributors appear in DSP0266's acknowledgements). **The specific Dell developer-portal page attempted returned no content** ("No data found / Api version not found"), so no iDRAC generation, firmware version or specification version is claimed here. | Not verified — 21 Sep 2026 (see section 14) |
| **Lenovo — XClarity Controller (XCC)** | Lenovo's own product documentation states: "The Lenovo XClarity Controller provides a Redfish compliant set of easy-to-use REST APIs that can be used to access Lenovo XClarity Controller data and services from applications running outside of the Lenovo XClarity Controller framework." Lenovo documents XCC as the successor to the Integrated Management Module II (IMM2), and separately documents that XCC supports an **IPMI interface** alongside Redfish. | `pubs.lenovo.com/xcc/rest_api`; checked 21 Sep 2026 |
| **Supermicro** | Supermicro publishes a Redfish API resource page hosting a **Redfish User Guide v6.1** (resource thumbnail dated 2026-04) and a **Redfish Reference Guide v4.0**, plus a Redfish white paper. The generic landing page documents the existence of the guides; the guides themselves are behind a manual-download path. | `supermicro.com/en/solutions/management-software/redfish`; checked 21 Sep 2026 |
| **Cisco** | Not verified in this pass. Cisco's out-of-band management (CIMC and related) is documented by Cisco, but no Cisco Redfish support statement was retrieved, so none is asserted. | Not verified — 21 Sep 2026 |
| **Fujitsu — iRMC** | Fujitsu publishes an **"iRMC Redfish API" specification document** through its technical support site, listed under Server Management Controller with products including PRIMERGY CX400 M4 and PRIMEQUEST 3400E; the register entry shows an original version dated 2017/07 with a revision dated 03/05/2020. Fujitsu's out-of-band management documentation describes the iRMC S5 and points to that Redfish API specification for "the commands and parameters of the Fujitsu Redfish API". | `support.ts.fujitsu.com` document entries SWP1236312 and SWP1249205; checked 21 Sep 2026. The 03/05/2020 date is a **revision date on the document entry**, not a claim about current firmware. |
| **Huawei — iBMC** | Indirect dated evidence only: the community `redfish_exporter` project's tested-device list includes a Huawei iBMC at **firmware 6.22**. This is a third-party tool's list, not a Huawei statement, and is recorded as such. No Huawei support statement with a dated specification version is asserted. | `github.com/jenningsloy318/redfish_exporter` README; checked 21 Sep 2026 |
| **Inspur** | Not verified in this pass. No Inspur Redfish support statement retrieved; none asserted. | Not verified |
| **AMI — MegaRAC (BMC firmware vendor)** | AMI describes **MegaRAC SP-X** as "AMI's mature and widely deployed BMC firmware solution, trusted by top-tier ODMs, OEMs and hyperscalers", providing "comprehensive out-of-band management for servers, accelerators and storage systems". AMI announced **MegaRAC SP-X LTS 13.5** as a firmware release supporting customer reference boards including Intel 4th Gen Xeon ("Archer City") and AMD 4th Gen EPYC ("Quartz"). AMI also maintains a public GitHub organisation (`github.com/ami-megarac`) containing "the Modified, Reconfigured, or Added open source modules in MegaRAC SP-X stack". | `ami.com/products/megarac/` and AMI's release announcement for SP-X LTS 13.5; checked 21 Sep 2026. **Why this row matters:** MegaRAC is firmware, not a brand — it appears *underneath* other vendors' products, so "whose Redfish implementation is this?" is a question that must be answered by firmware, not by the logo on the bezel. |

**The general statement this table supports, and the one it does not.** Every major server vendor ships an out-of-band controller with a Redfish service, and each vendor documents it. What this pass could *not* establish for any vendor is a **dated mapping from a product generation and firmware release to a specification version** — the "iLO N, firmware X, DSP0266 vY" triple that Question 1 of section 4 asks for. That mapping exists in vendor release notes, which are per-generation and per-firmware documents; it is the specific thing a buyer must request. Do not let a vendor's "supports Redfish" slide substitute for it.

### The open-source BMC stack: OpenBMC

**OpenBMC** is "a Linux Foundation project whose goal is to produce a customizable, open-source firmware stack for Baseboard Management Controllers (BMCs)" — DMTF-external, vendor-neutral, and the reference implementation for several large operators. Governance is documented: a **Technical Oversight Forum (TOF)** "guides the technical direction of the project", with members "voted on by community members" per published membership-and-voting guidelines; the project publishes a charter, a code of conduct, formal security policy and contribution process; contact is via an OzLabs mailing list and the project Discord.

**Dated activity status: ACTIVE.** Checked 21 September 2026: the `openbmc/openbmc` distribution repository shows ~24,565 commits, 2.6k stars, 1.2k forks and per-vendor layers for a long list of silicon and system vendors (Ampere, AMD, Aspeed, ASRock, ByteDance, Delta, Facebook/Meta, FII, Google, HPE, IBM, IEI, Ingrasys, Intel, Inventec, Nuvoton, NVIDIA, NXP, OpenPOWER, Qualcomm, Quanta, Supermicro, Tyan, UfiSpace, Wistron, Yadro). The `openbmc/docs` repository shows 841 commits and a maintained documentation set including `REDFISH-cheatsheet.md`. The `openbmc/bmcweb` component — described as "a protocol compliant, DBus to Redfish translator" — shows 3,653 commits.

**Two candid caveats, both from OpenBMC's own documentation:**

1. **Platform maturity is uneven.** OpenBMC's own README warns that "not all ports are equally mature — some platforms are better supported than others, and functionality on some 'supported' boards may be fairly limited", and that board support is not system support: "fan control is critically dependent on not just the motherboard but also the fans connected to it and the chassis", so "rough edges (such as your cooling fans running continuously at full throttle) are likely."
2. **The project's own README lists "OpenCompute Redfish Compliance" as a _Feature In Progress_, not a completed feature.** This is a significant, self-reported data point for section 4's thesis: even the flagship open-source BMC stack describes its Redfish compliance work as ongoing.

**Security posture, as documented:** `bmcweb` supports multiple authentication protocols — Basic per RFC 7617, cookie-based authentication for the web UI, mutual TLS based on OpenSSL, session authentication, and **"XToken based authentication conformant to Redfish DSP0266"** — each enable-able or disable-able at runtime through the relevant Redfish APIs or at configure time; username/password mechanisms route to libpam for pluggable authentication. Authorization is "determined at routing time, and per route, and conforms to the Redfish PrivilegeRegistry", with non-Redfish functions mapped to the closest equivalent Redfish privilege. TLS is via OpenSSL with HTTP/1 and HTTP/2 over ALPN; **if SSL is enabled and no usable certificate is found, bmcweb "will generate a self-signed certificate before launching the server"** — precisely the default-certificate condition DSP0266 §13.1.3 normatively requires an implementation to be able to replace.

**A note on the project website.** `openbmc.org` presented content referencing "release 2.10" and founding organisations, and read as stale relative to the repositories. Treat the **repositories as the authoritative activity signal** and the marketing site with caution when dating the project.

### Client libraries and automation tooling — with dated maintenance status

| Project | Maintainer | What it does | Status as checked 21 Sep 2026 |
| --- | --- | --- | --- |
| **`python-redfish-library`** (`pip install redfish`) | DMTF | Python 3 client performing GET/POST/PUT/PATCH/DELETE against a Redfish service; BSD-3-Clause; supports session keys, proxies (including SOCKS), and an optional async client. Python 2 dropped at v3.0.0. | **Active/maintained by DMTF.** 216 stars, 366 commits. Note the documented naming collision warning: an unrelated PyPI package `python-redfish` (an OpenStack-era module) uses a conflicting name — the DMTF library installs as `redfish`. |
| **`Redfish-Service-Validator`** | DMTF | GET-only payload validation against CSDL schema. Requires `redfish>=3.1.5`. Publishes to PyPI as `redfish_service_validator`. | **Active.** 971 commits; copyright "2016-2026 DMTF" (current year present). |
| **`Redfish-Protocol-Validator`** | DMTF | HTTP protocol conformance testing (headers, status codes). Publishes as `redfish_protocol_validator`. | **Maintained, but the least active of the three.** Only 241 commits and the smallest following (14 stars); its copyright header reads "Copyright 2020-2022 DMTF". Use it, but expect the Service and Interop validators to be the more actively developed instruments. |
| **`Redfish-Interop-Validator`** | DMTF | Validates a live service against a named interoperability profile. Publishes as `redfish_interop_validator`. | **Actively developed.** Copyright "2017-2026 DMTF"; latest release **3.0.0 on 18 September 2026** — three days before this check. |
| **`Redfish-Mockup-Creator`** | DMTF | Walks a live service with GETs and writes a directory tree of responses, for use with the Redfish Mockup Server. Docker image published as `dmtf/redfish-mockup-creator`. | **Maintained.** 162 commits. Its practical function is important: it lets you capture a *real* device's Redfish surface as a fixture, which is how you build a regression test for a fleet without keeping hardware on a bench. |
| **`sushy`** | OpenStack (mirror of opendev.org) | Small Python library to talk to Redfish systems, used by OpenStack Ironic for bare-metal provisioning. Apache-2.0. | **Active.** 741 commits. Its README is unusually candid and operationally useful: the goal is to "be extremely simple, small, have as few dependencies as possible and be very conservative when dealing with BMCs by issuing just enough requests to it (**BMCs are very flaky**)", and the scope "has been limited to what is supported by the OpenStack Ironic project." Treat it as a well-maintained but deliberately narrow client, not a general-purpose Redfish SDK. |
| **`sushy-tools`** | OpenStack | Redfish simulators: a static responder and a virtual Redfish BMC backed by libvirt or a cloud. | **Active** (423 commits) but **explicitly not production tooling**: "It is not designed for use outside of development and testing environments. Please do not run sushy-tools in a production environment of any kind." This is a documented dormancy-avoidance warning in the *opposite* direction — a live project you must still not put in the architecture. |
| **`redfish_exporter`** (Prometheus) | Community (`jenningsloy318`) | Prometheus exporter for hardware metrics over Redfish; exposes `/redfish?target=<host>` on port 9610 (a Prometheus-reserved default port); built on the `gofish` library. | **Community-maintained; treat as a dependency you must own.** 96 stars, 151 commits, 87 forks. Its README carries a **dated tested-device list with firmware versions**, which is the most useful thing in it: Enginetech EG520R-G20 (Supermicro firmware revision 1.76.39), Enginetech EG920A-G20 (Huawei iBMC 6.22), Lenovo ThinkSystem SR850 (BMC 2.1/2.42), Lenovo ThinkSystem SR650 (BMC 2.50), Dell PowerEdge R440/R640/R650/R6515/C6420, GIGABYTE G292-Z20/G292-Z40/G482-Z54. **This pass did not verify a last-commit date**, so its current maintenance level is unestablished (section 14). |
| **Ansible Redfish module / collection** | Community | Declarative Redfish task execution in Ansible playbooks. | **Not verified in this pass.** No specific module or collection was retrieved and date-checked, so none is named or recommended here. For Ansible mechanics generally see [ansible_vs_ansible_tower_guide.md](ansible_vs_ansible_tower_guide.md). |
| **Terraform provider** | Community / vendor | Declaring Redfish-managed state through Terraform. | **Not verified in this pass.** No specific provider was retrieved and date-checked; none is named. |

**The maintenance rule this table is meant to enforce.** A Redfish integration is a *dependency on somebody's continued maintenance of a mapping between a standard and a vendor's divergences*. The mapping breaks when the standard adds a resource, when the vendor changes firmware, or when the maintainer stops. So the question is never "does this project exist" — all of the above exist — but "when was it last released, and who owns it when it stops." Two of the entries above (the protocol validator's 2022-era copyright and the community exporter's unverified commit date) are the ones to check before depending on them.

---

## 8. The management-plane practice this enables

For each capability below: what the **standard** provides, and what **tooling must add**. The distinction is the section's point — Redfish gives you a machine-checkable interface; it does not give you a fleet management product.

### Automated discovery and inventory

**Standard provides:** the service root and the hypermedia tree; `Systems`, `Chassis` and `Managers` collections at the root; `Links.ManagedBy` and `Links.ManagerForSystem` / `ManagerForChassis` relationships; `SerialNumber`, `PartNumber`, `Model`, `Manufacturer` and `SKU` on inventory resources; `AssetTag`-style identification where the vendor populates it; the new **resource alias** mechanism (v1.25.0) for giving components stable names; `$expand` to collapse a navigation hop into one response (`GET /redfish/v1/UpdateService/FirmwareInventory?$expand=.`) and `$select` to trim payloads (`GET /redfish/v1/Systems/1?$select=Name,SystemType,Status/State`). DMTF's own `Redfish-Mockup-Creator` walks a service with GETs and captures the whole surface as a file tree — the cheapest possible way to learn what a given firmware exposes.

**Tooling must add:** the *seed* (which addresses to probe), credentials and certificate trust, the scheduling, the normalisation of vendor-populated fields across models, the reconciliation against an authoritative CMDB, and the deduplication of a device that appears in several tools' inventories. Redfish has no "discover all devices on the subnet" mechanism; discovery is a discovery-protocol problem, not a Redfish problem.

### Firmware lifecycle management at fleet scale

**Standard provides:** `UpdateService` with `FirmwareInventory`; the `SimpleUpdate` action for a straightforward push; multipart HTTP push via a service-advertised `MultipartHttpPushUri` (the specification's own example shows `/redfish/v1/UpdateService/update-multipart`); asynchronous execution with a **task resource** and a **task monitor** URI (`/redfish/v1/TaskService/TaskMonitors/<TaskMonitorId>`) so a client can follow a long update to completion; `Update` messages in the standard registry for lifecycle events; and — new in 2026.2 — `UpdateServiceCapabilities` v1.1.0 declaring which semantics the service supports (`TargetsParameterSupported`, `ExcludeTargetsParameterSupported`, `ForceUpdateParameterSupported`, `StageParameterSupported`, `LocalImageParameterSupported`, `AllowableTargetReferenceTypes`).

**Tooling must add:** the firmware **provenance and decision** problem — which image, signed by whom, verified how, approved by whom (section 10); the ordering and blast-radius logic (never update a whole cluster at once); the pre/post health comparison; the rollback path and its test; the durable record of which unit got which firmware when; and the handling of the fact that an update can reboot the BMC and drop the session mid-task. The `Update` registry's automatic-recovery messages (`AutomaticRecoveryInitiated`, `AutomaticRecoveryFailed`, `AutomaticRecoverySuccessful`, added in 2026.2) exist precisely because updates fail — the standard now tells you when a service recovered itself, which is only useful if something is subscribed and listening.

### Configuration baselines and drift detection

**Standard provides:** resources whose state is readable and writable — `Bios` attributes through the **attribute registry**, boot configuration, network settings on `ManagerNetworkProtocol`, `EthernetInterface` and `NetworkAdapter` resources, `Storage` and volume configuration; the `@Redfish.Settings` mechanism (a resource advertises a settings object to modify, so a client changes the *pending* configuration and the service applies it at a defined point such as reset — DSP0266's `Bios` schema description states that "it is likely that a client finds the `@Redfish.Settings` term in this resource" and that changes are made by modifying the resource identified by that term); `ETag` and `If-Match` for optimistic concurrency on writes; and the `.Deep` operation for changing many members in one request.

**Tooling must add:** the baseline itself. The standard has no notion of "desired state" — there is no `DesiredConfig` resource. A baseline is a tool-side artefact compared against read state, and the comparison runs straight into the section 6 finding that **BIOS attribute registries are vendor-specific by design** ("the attributes and property names are not standardized"). Expect a vendor branch per platform family, and budget for it explicitly rather than discovering it mid-project.

### Hardware health and eventing into monitoring

**Standard provides:** the shared `Status` object with `State` and `Health` on resources (§9.6.16 of the specification); `LogService` and entries (`/redfish/v1/Systems/1/LogServices/FaultLog/Entries`); `TelemetryService`, `Sensor`, `PowerSubsystem` and `ThermalSubsystem`; `EventService` with client-supplied subscriptions for push delivery; and standard message registries that make events machine-interpretable — including, as of 2026.2, the new `Device` registry (`PresenceMismatch`, `AssemblyDataMalformed`, `ManagementInterfaceError`) and the new `EventService` registry (`EventSubscriptionCreated`, `EventDeliveryFailed`, `EventSubscriptionResumed`).

**Tooling must add:** the receiving endpoint and its authentication; the mapping from Redfish `MessageId`s to alert routing and severity; the de-duplication and flapping suppression that hardware telemetry always needs; the correlation from a component to the service it affects (a `Links`-walk, and the place where a good CMDB earns its keep); and the observability stack itself. The repository's observability and monitoring practice is owned elsewhere — this guide specifies the interface and the event vocabulary, not the pipeline. Note the `EventSubscriptionResumed` and `EventDeliveryFailed` messages particularly: **the standard now tells you when event delivery is broken, which means a management plane that is silently not receiving events is a detectable condition rather than an invisible one.**

### Power and boot control, including virtual media

**Standard provides:** the `ComputerSystem.Reset` action with a `ResetType` parameter whose allowable values the resource advertises (`On`, `ForceOff`, `GracefulRestart`, and so on); boot override configuration through the system's `Boot` object including `UefiTargetBootSourceOverride`; `VirtualMedia` resources for attaching remote media, with `InsertMedia` / `EjectMedia` actions; and — as of 2026.2 — `VirtualMedia` v1.7.0 adding an `HTTPHeaders` property and an `HTTPHeaders` parameter to `InsertMedia`, plus a broader graphical-console story (`VNC`/`RDP`/`HTML5`/`KVMIP` properties and one-time-URL and password actions on `ComputerSystem`).

**Tooling must add:** the **change-gating**. A graceful restart action is one `POST` away from a production node, and the standard is neutral about whether you meant it. Approval workflows, maintenance windows, the "is this node in service?" check against the load balancer or cluster state, and an audit trail of who invoked the action and why are all tool-side. Virtual media, in particular, is the quintessential "powerful and easy to misuse" facility: it lets a client boot a host from an image it controls, which is legitimate recovery tooling and also a complete host takeover primitive if the management plane is reachable by the wrong party (section 9).

### Certificate and account lifecycle

**Standard provides:** `AccountService` for accounts, roles and authentication configuration (`HTTPBasicAuth`, `MFA`); `AccountService` role resources including the three required standard roles and their privilege sets; `CertificateService` and `CertificateLocations` to find every certificate on the service; `Certificate` resources for install, replacement and renewal, with `CertificateEnrollment` for automatic enrolment; the normative requirement to support replacing the default certificate and to use X.509 v3 (RFC 5280); client-certificate authentication with OCSP revocation checking for mTLS flows; session termination and forced logout semantics; and the new `AccountSecurity` registry messages that carry addresses on authentication failure.

**Tooling must add:** the **certificate authority for a fleet**. Issuing, distributing, rotating and revoking certificates across hundreds of BMCs, attaching them to a trust chain the automation clients also trust, and discovering the self-signed defaults that were never replaced. The standard explicitly requires replaceability — the `bmcweb` behaviour of auto-generating a self-signed certificate when none is present shows why: out of the box, every unit is serving a certificate nobody chose and nobody tracks. Closing that gap is an operational programme, not an API call.

---

## 9. The architecture of the management plane itself

This section is for the network and security architect. Everything in it follows from one property of the BMC that no standard can change: **the BMC is a computer on your network that can control other computers without their operating system's cooperation.** Redfish makes that computer's interface well-defined. It does not make it safe to expose, and a well-defined interface to a powerful control plane is arguably a sharper weapon than an obscure one.

### Segmentation of the out-of-band network

**The requirement, plainly stated:** the management interface must not be reachable from the production network or the general corporate network. Not filtered by a rule that some team can loosen; architecturally unreachable, with any exception a deliberate, recorded, reviewed decision.

The architectural case does not rest on Redfish at all. It rests on three facts, all documented:

1. **The control path bypasses the OS.** The CISA/NSA advisory states the point directly: BMCs "operate separately from the operating system (OS) and firmware to allow for remote management and control, even when the system is shut down." Host-based controls — endpoint agents, host firewalls, OS-level logging — are structurally blind to this path.
2. **The advisory names network segmentation as one of three commonly-overlooked mitigations**, alongside hardened credentials and firmware updates: "Hardened credentials, firmware updates, and network segmentation options are often overlooked, leading to a vulnerable BMC."
3. **The compromise consequence is described by the advisory as pre-OS.** A vulnerable BMC "broadens the attack vector by providing malicious actors the opportunity to employ tactics such as establishing a beachhead with pre-boot execution potential." A beachhead with pre-boot execution potential is, for a payments system, a control position underneath the entire software stack.

**Reference architecture, in layers:**

- **A dedicated out-of-band management VLAN/VXLAN per data centre**, carried on dedicated physical ports of the servers (the BMC's own NIC, separate from the host's data NICs). No VLAN trunking that puts the management VLAN and a production VLAN on the same host-facing interface where the host can be made to bridge or tag.
- **No default gateway to anywhere interesting.** If the management network's default route leads to the internet, an unattended BMC is an internet-facing BMC. Either no default route, or a default route to a broker that permits only the specific flows required.
- **Explicit allow-list flows only:** automation client → BMC (HTTPS/443); BMC → event collector (if push eventing is used); BMC → internal CA (OCSP/certificate services); BMC → internal firmware repository (if the BMC pulls updates rather than being pushed to); BMC → directory service (if LDAP/AD is used, over TLS as §13.3.1 requires). Everything else denied, and denied by default.
- **Direction matters.** Inbound from production is denied. *Outbound from the BMC to production is also a risk:* a compromised BMC initiating a connection into a production subnet is an egress path, so egress control belongs in the design, not the remediation plan.
- **The management network is a management network.** Where the physical facilities sit is owned by [data_center_guide.md](data_center_guide.md), [l2_design_financial_data_centers_guide.md](l2_design_financial_data_centers_guide.md) and, for Singapore specifically, [singapore_data_centres_guide.md](singapore_data_centres_guide.md).

### Jump-host and broker patterns

Direct client-to-BMC connectivity does not scale as a security model, because it distributes credentials to clients. Three patterns, in increasing order of control:

**1. Jump host (bastion).** Operators reach BMCs only by first authenticating to a hardened, logged jump host that sits inside the management segment. Cheap, familiar, auditable at the session level. Weaknesses: it concentrates access, its credential becomes the crown jewel, and it does nothing for automation clients.

**2. Broker / API gateway.** Automation clients never hold BMC credentials. They authenticate to a broker service, which authenticates to each BMC, executes the request, and returns the result. The broker holds the fleet credentials (or, better, the fleet client certificates), enforces policy on which calls are permitted, and produces the audit record. This is the pattern that makes section 10's segregation-of-duties requirement tractable, because a single chokepoint can enforce who may invoke `ComputerSystem.Reset` and under what conditions.

**3. Certificate-based authentication at scale (the pattern to aim for).** Rather than a per-device password inventory, use the client-certificate authentication DSP0266 §13.3.5 defines: the broker presents a certificate during the TLS handshake, the BMC verifies it against a `CertificateCollection` it trusts, and checks revocation via OCSP. **What this buys:** no static password on any BMC for automation use; a single revocation point; and an authentication event that is cryptographically attributable to the broker rather than to a shared string. **What it costs:** a PKI programme — the CA, the trust distribution to every BMC, the rotation, and the OCSP infrastructure the specification's revocation requirement implies. That cost is the honest reason this pattern is not universal; it is also the reason a fleet-scale management plane without it eventually accumulates a shared password (see section 12).

**On eventing destinations:** if you use push eventing, the BMC holds an outbound destination you configure, and a credential for it where the destination requires authentication. That destination is now part of the BMC's attack surface — a compromised BMC can send crafted events to your monitoring pipeline, and a misconfigured destination can leak inventory data. Terminate events at a collector inside the management segment, validate and re-emit outward; do not point a BMC directly at a production SIEM endpoint. The `EventDeliveryFailed` message added in the 2026.2 `EventService` registry is your signal that this path is broken.

### The authentication and certificate-authority question for a fleet

The single most underestimated task. For N BMCs you must solve: **who is the CA, and how does each BMC come to trust the right things?**

- **Server certificates:** every BMC needs a certificate its clients will trust. The out-of-box state is a self-signed, per-unit certificate (`bmcweb` generates one automatically when none is present), which trains every client to disable verification — the anti-pattern in section 12. The specification normatively requires that the default certificate be replaceable and that certificates be X.509 v3 (§13.1.3). So: issue from an internal CA, distribute, rotate before expiry, and have an inventory of *which BMC has which certificate and when it expires*. Without that inventory, certificate expiry becomes an outage that arrives on a date nobody recorded.
- **Client certificates / trust anchors:** the `CertificateService` and `CertificateLocations` resources exist to make this enumerable on a live device. The fleet problem is *knowing* what each device trusts, which the standard lets you read but does not track for you.
- **Trust on the client side:** your automation must verify BMC certificates against your CA bundle. Note that DMTF's own Protocol Validator ships both a `--no-cert-check` flag and a `--ca-bundle` option — a reminder that the discipline is to pass the CA bundle, not to skip verification.
- **Directory integration:** if accounts come from LDAP/AD, the BMC-to-directory path is in scope for the specification's TLS requirement and needs its own monitoring, because authentication failure against a directory looks identical to a wrong password from the outside.

### Inventory and firmware-provenance implications

The management plane is where hardware identity and firmware state become *knowable*. That creates two obligations:

- **Inventory is a security artefact, not just an asset record.** `SerialNumber`, `PartNumber`, `Model`, `Manufacturer`, `SKU` and the firmware inventory are the inputs to vulnerability management and to supply-chain provenance. If your CMDB's hardware records come from a spreadsheet, they are wrong.
- **Firmware provenance must survive the update pipeline.** A firmware image should be authenticated as it enters the repository, its hash recorded, its approval recorded, and its delivery verified — and the `UpdateService` path should be the *only* path (see section 10 and section 12). `SoftwareInventory` v1.15.0 added an `ImageState` property and an `Armed` property with `ArmedTargets` links in 2026.2 — the model can now describe staged and pending update state, which is what a provenance-aware pipeline needs to distinguish "the image is on the device" from "the image is running".

### Failure modes of a badly-built management plane

Named, because each has a symptom that is mistaken for something else:

| Failure mode | What it looks like | What it actually is |
| --- | --- | --- |
| **The flat management network** | "Our BMCs are on the server VLAN, it's fine, only admins can reach them" | The BMC is on the same segment as the workloads it controls. One compromised workload, or one misconfigured host bridge, is adjacent to the control plane. |
| **The shared credential** | One `svc_bmc` account with the same password on every device, in a config file, rotated never | Compromise of one device compromises the fleet; revocation requires touching every device; nobody can attribute an action to a person. |
| **The unmanaged certificate** | Every client sets "ignore certificate errors" because each BMC's cert is self-signed and different | TLS becomes encryption without authentication — you cannot detect an impostor. The fix is a CA and a distribution process, and it is usually deferred indefinitely. |
| **The firmware nobody tracks** | A spreadsheet, updated when someone remembers, listing versions for the models someone surveyed | You cannot answer "which units are exposed to advisory X" in time to act. The CISA/NSA advisory names firmware updates as a routinely overlooked mitigation for exactly this reason. |
| **The unmonitored management plane** | Monitoring covers production; the BMC segment is a blind spot because "it's just management" | Loss of the management plane is invisible until the day you need it — and then it is discovered during an incident, at the worst possible moment (section 10). |

---

## 10. The regulated-enterprise and banking angle

The banking angle is not that Redfish is a banking technology. It is that **a regulated institution running payments or holding customer data operates a control plane underneath every control it has.** This section states what that implies, in terms the institution's existing frameworks already use.

### The management plane as an attack surface

What a compromised BMC gives an attacker, in the CISA/NSA advisory's own language: a position that operates independently of the OS, persists when the system is shut down, and offers "pre-boot execution potential" as a beachhead. Translated to a payments environment:

- **Code execution beneath the measured stack.** Anything the OS reports about itself becomes unreliable; attestation that depends on the host is worthless.
- **Persistence across the standard remediation.** Wiping the OS and reimaging does not clear the BMC. An incident-response runbook that stops at the operating system leaves the attacker in place.
- **Complete host takeover potential.** Virtual media and boot override (section 8) let a party who controls the BMC boot the host from an image they choose.
- **Lateral movement from a trusted position.** Management networks are often trusted by construction — the BMC segment may be permitted to reach management, orchestration and backup systems that production cannot.

That is the risk statement. It is why the mitigation list the advisory gives — hardened credentials, firmware updates, segmentation — is a control set, not a checklist.

### Firmware and supply-chain provenance

The question is: **do you know what code is running in the management controller, who signed it, and how it got there?** For an institution that must evidence control over its technology stack, this is the least mature area of most hardware estates, because firmware sits below the layers that change management usually reaches.

Cross-reference the repository's security-discipline content for the framing: [cybersecurity_guide.md](cybersecurity_guide.md) and [beyond_zero_enterprise_security_guide.md](beyond_zero_enterprise_security_guide.md) for the threat model, [security_by_design_guide.md](security_by_design_guide.md) for designing the controls in rather than bolting them on, [secops_guide.md](secops_guide.md) for the detection and response side, and [zero_trust_network_architecture_guide.md](zero_trust_network_architecture_guide.md) for the network posture that section 9 applies to a segment.

Concretely, provenance for a BMC fleet means: an approved firmware baseline per model; images obtained from the vendor through a verified channel; signature or hash verification before deployment; the `UpdateService` path as the only update path; a durable record of what was applied to which serial number and when; and the ability to answer an advisory question ("which of our units run the affected firmware?") from records rather than from a survey.

One procurement-relevant fact from section 7: **your BMC firmware may not be your server vendor's work.** AMI's MegaRAC SP-X is "trusted by top-tier ODMs, OEMs and hyperscalers" and appears across many brands. So the provenance chain has a third party in it that the logo on the bezel does not reveal. Ask which BMC firmware stack is under the product, and track advisories for it by name.

### Change control and audit expectations for firmware and BMC configuration

An examiner's questions are predictable, and they map onto ordinary change-management evidence:

| Expectation | The evidence that satisfies it |
| --- | --- |
| Firmware changes are controlled | A change record per firmware change, with approval, window, affected unit list, and outcome — including failures and automatic recoveries (the 2026.2 `Update` registry's `AutomaticRecoveryInitiated`/`Failed`/`Successful` messages are exactly what a runbook should capture) |
| BMC configuration changes are controlled | Configuration baselines under version control, with diffs and approval; the `@Redfish.Settings` pending-vs-applied distinction used deliberately so the change lands at a known point |
| Changes are attributable | Per-operator credentials, or broker-mediated access with the broker's identity in the audit trail; not a shared service account (section 12) |
| **Segregation of duties** | The person who proposes a change is not the only person who approves and executes it. Architecturally, this means the broker pattern from section 9, because a flat credential model cannot express it |
| Access is least-privilege | Use the standard roles rather than everyone as `Administrator`: `ReadOnly` for inventory and monitoring, `Operator` for `ConfigureComponents` operations, `Administrator` restricted to a small number. The specification's role table (§13.4.2.1, Table 43) gives you the privilege sets to map onto job functions |
| Access is reviewed | Periodic account and certificate review, including the automation credentials and certificates, with the fleet CMS/CMDB as the population list |
| The evidence is durable | Audit records retained for the institution's retention period, in a system whose own integrity the BMC cannot touch — i.e. not the BMC's own log service as the sole record |

The last row deserves emphasis: the BMC's `LogService` is evidence of what the BMC saw, and it is under the BMC's control. A party who has compromised the BMC can erase it. Audit records must leave the device, in real time, to a collector they cannot reach.

### Operational resilience: the honest reading

Here is the point most management-plane discussions get wrong, in both directions.

**Loss of out-of-band management does not take a service down.** If every BMC in a data centre becomes unreachable, the servers keep running, the applications keep serving, the transactions keep clearing. Out-of-band management is not in the transaction path. Nobody's payment fails.

**Loss of out-of-band management does take away the ability to recover the service promptly.** This is the real exposure, and it is a resilience exposure rather than an availability one. When out-of-band management is unavailable and something *else* fails — a node hangs on a driver, a host needs reimaging, a BIOS setting must change, a firmware vulnerability must be patched within a mandated window, a crashed node needs a power cycle at 03:00 — the recovery path is gone. Someone drives to the data centre. Mean time to recovery is measured in hours and in human presence. And the moment you discover the management plane is broken is exactly the moment you need it, so the discovery is always at the worst time.

This is why the management plane belongs in the institution's resilience analysis as a **dependency of the recovery capability**, not as a service in its own right. The right place to record it is the operational-resilience framework — see [operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) — as an *important business service* input: a component whose loss does not breach impact tolerances directly but which removes the institution's ability to stay within them after an unrelated failure. The corollary is that the management plane needs its own availability monitoring (section 12): if you do not know it is down, you will find out during the incident, and by then the clock is already running.

Related framework material in the repository: [enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md) and [risk_management_models_guide.md](../banking/risk_management_models_guide.md) for how to score a dependency of this shape, and [secops_guide.md](secops_guide.md) for detecting and responding to management-plane events.

### The secret-management question for a fleet of credentials

Stated as a design requirement with the failure it prevents: **a fleet of BMC credentials must be a secret-management problem, not a config-file problem.**

The failure mode is well known and entirely empirical: the credentials start in a documented place, get copied into a script, get copied into a playbook, get copied into a monitoring configuration, are rotated in one of those four places, and are never rotated again because rotation would break the other three. The endpoint is a single password for several hundred devices, held by an unknown number of systems and people, which cannot be revoked without an outage. Section 12 records it as an anti-pattern.

The design that avoids it:

- **Prefer client certificates over shared passwords for automation** (section 9): no static secret to distribute, and revocation is a CA operation rather than a fleet-wide change.
- **Where passwords are unavoidable, make them per-device and machine-retrievable** from a vault, with the retrieval itself audited. Per-device means a compromise is contained; vault-retrieved means rotation does not require editing scripts.
- **No credential in a device-side destination.** The eventing destination and the update source should not be places where a compromised BMC finds a credential for something else.
- **Rotate on a schedule and on events.** Scheduled rotation is table stakes; **rotation on personnel change and on suspected compromise** is what makes the schedule meaningful.
- **Monitor authentication failures across the fleet.** The 2026.2 `AccountSecurity` registry's `InvalidCredentialsWithAddress` and `InsufficientPrivilegeWithAddress` messages exist so that a fleet-wide credential-stuffing attempt against BMCs is a detectable pattern. Subscribe to them (section 8).

---

## 11. The Cymbal Bank worked example

> **Cymbal Bank is a fictional institution. It is the only bank persona used in this repository, and it exists solely to make this design discussion concrete.** All figures in this section are **illustrative** — they are order-of-magnitude planning numbers chosen to expose the shape of the trade-off, not measurements, quotes, benchmarks or commitments. Nothing in this section should be read as a real price, a real deployment, or a statement about any real institution.

### The scenario

Cymbal Bank is standardising out-of-band management of **several hundred servers across two data centres** (call it 600 units, split 350/250 for the purposes of the exercise). The estate is multi-vendor: two primary server vendors plus a storage platform and a growing GPU-accelerated inference estate. The platform team owns the servers; the network team owns the management network; security owns the policy; a small automation team owns the tooling. There is no single owner of "the management plane," which is the first finding.

### Current-state findings

| Finding | Evidence a reviewer would accept | Why it matters |
| --- | --- | --- |
| **Flat management network** | BMC addresses drawn from the same RFC1918 ranges as server management interfaces, routed on the general data-centre network; the "management VLAN" is a naming convention, not a segmentation control | Directly contradicts the CISA/NSA advisory's segmentation recommendation. Any workload on the adjacent segment is one step from the control plane. |
| **Per-vendor tooling** | Three vendor portals, three CLI tools, one vendor-specific update utility, and a fourth tool for the GPU nodes; no shared inventory view | No fleet-level answer to "what firmware is running where." Operators learn one vendor's tool and are helpless on the other. |
| **Untracked firmware** | A spreadsheet listing BMC firmware versions, last updated 14 months ago, covering roughly a third of the estate | Cannot answer an advisory question inside a mandated patch window. The advisory names firmware updates as routinely overlooked. |
| **Shared credentials** | One `bmcadmin` account, same password on all 600 units, stored in three places (a script, a playbook, a monitoring config), rotated once | One compromise is a fleet compromise, and revocation is an outage. Cannot attribute an action. |
| **Unmanaged certificates** | Self-signed per-unit certificates; automation and operator scripts invoke clients with verification disabled | TLS without authentication. Cannot detect impostor devices. |
| **No management-plane monitoring** | Monitoring covers the applications and the server OS agents; the BMC segment has no availability check and no event subscription | Out-of-band management failure is invisible until an incident needs it. |
| **Unowned lifecycle** | Nobody has the job "BMC firmware and certificates" | Every finding above is a symptom of this one. |

### Target architecture

**Segmentation.** A dedicated out-of-band management segment per data centre, on the BMC's own physical ports, with no trunk carrying production VLANs to a host-facing interface. No default route to the internet; an explicit allow-list of flows only: automation/broker → BMC over HTTPS; BMC → internal event collector; BMC → internal CA/OCSP; BMC → internal firmware repository; BMC → directory service over TLS. Egress from the management segment is controlled, on the basis that a compromised BMC is an outbound path. The management segment's own design sits inside the physical and cabling decisions owned by [data_center_guide.md](data_center_guide.md) and [l2_design_financial_data_centers_guide.md](l2_design_financial_data_centers_guide.md).

**Brokering.** A single broker service inside the management segment is the only thing that talks to BMCs. Interactive operators reach the segment through a logged jump host. Automation and operators both go through the broker, which authenticates to BMCs, enforces policy ("may this identity invoke `ComputerSystem.Reset` on a node that is in service?"), and produces the audit record. Segregation of duties becomes enforceable at one chokepoint rather than hoped for across 600 devices.

**Credential and certificate model.** Prefer client certificates from an internal CA for broker-to-BMC authentication (DSP0266 §13.3.5), with per-device accounts as the fallback during migration. Any remaining password lives in a vault, is per-device, and is retrieved with an audit record. A fleet certificate and account inventory is a deliverable, not a by-product: **you cannot rotate what you cannot enumerate.**

**Tooling decisions.** Standardise on the DMTF validator suite as the conformance instrument (section 4) and on a Redfish client that the team owns. Reject any plan that depends on a vendor's own portal for fleet operations, and reject any plan whose client library is dormant.

### The vendor-interoperability questions, drawn from section 4

This is where Cymbal's standardisation effort either succeeds or becomes three parallel efforts. For each platform family, the team puts **the three questions** in writing to the vendor, and requires answers before purchase:

1. **Which Redfish specification version** (DSP0266 version) does this product generation, at this BMC firmware release, implement? Request the mapping per generation and firmware release, not a generic "supports Redfish".
2. **Which schema bundle** (DSP8010 release) and **which registry bundle** (DSP8011 release) does that firmware implement? This determines what a client may assume exists.
3. **Which interoperability profile** (from DSP8013) does it conform to, **and can we see the Interop Validator report** — and may we re-run the validators ourselves on a unit we own?

Then the follow-through that matters more than the answers: **Cymbal runs the validators itself**, on one unit of each platform family, before signing. The DMTF suite is public; the Service Validator is GET-only and safe to run against production, the Protocol Validator tests HTTP behaviour, and the Interop Validator checks a named profile. A vendor that refuses a validator run is a finding.

The known gaps to plan around, from section 6: **BIOS attribute configuration is vendor-specific by design** (the `AttributeRegistry` schema says so explicitly), so the configuration-baseline tooling will carry a vendor branch per platform family; **graphical console/KVM** only became standard in the 2026.2 release, so every currently-deployed unit handles it vendor-specifically; and **firmware update semantics** vary between `SimpleUpdate`, multipart push and vendor paths. Budget for these three branches explicitly rather than treating them as implementation details.

### Firmware lifecycle design

1. **Baseline.** An approved firmware version per model family, recorded with its source, signature/hash and approval. Baseline changes are changes, with owners.
2. **Provenance.** Images enter through one repository; authenticity verified on entry; delivery to devices through the `UpdateService` path only (`SimpleUpdate` or the service-advertised `MultipartHttpPushUri`). No vendor utility, no USB stick, no exception without a written one.
3. **Rollout.** Asynchronous execution monitored through the **task resource and task monitor** (`/redfish/v1/TaskService/TaskMonitors/<TaskMonitorId>`); never a whole fleet at once; a pilot ring, then a wave, with a health check before and after; a defined abort condition tied to the update registry's messages — and note that an update can restart the BMC and drop the session, so the client must re-establish and resume (section 5).
4. **Record.** Which serial number, which image, which version, when, approved by whom, outcome — in a system the BMC cannot reach.
5. **Answerability.** A rehearsed test: given a hypothetical advisory naming a firmware version, produce the affected-unit list within the mandated window. If the team cannot do this on a Tuesday afternoon, the design is not finished.

### Cost comparison

**All figures below are illustrative, order-of-magnitude planning numbers for a 600-unit, two-site estate, chosen to show the shape of the trade-off. They are not quotes, not benchmarks, and not derived from any real deployment.**

| Cost element | Do-minimum (status quo plus fixes) | Brokered and segmented (recommended target) |
| --- | --- | --- |
| Management network segmentation (switching, cabling, firewall policy, design) | Low — the segment exists in name; incremental hardening only | Medium — new VLANs/segments per site, allow-list policy, design and change work |
| Broker / gateway service (build or buy, plus run) | None — direct client-to-BMC access continues | Medium — the largest single new run-cost item |
| PKI for a fleet (CA integration, issuance, distribution, OCSP, rotation) | Low initially — self-signed, verification disabled | Medium–High — genuine programme cost, and the item most often under-budgeted |
| Automation tooling and client library | Low — per-vendor scripts, manual portal work | Medium — one owned client, validator integration, CI |
| Firmware lifecycle (baseline, repository, approval, records) | Low — the spreadsheet continues | Medium — process plus repository plus record-keeping |
| Fleet credential/secrets management (vault, per-device, audit) | Low — the shared account continues | Low–Medium — vault is likely already licensed for other uses |
| Monitoring and eventing for the management plane itself | Negligible — no coverage | Low–Medium — collector, subscriptions, dashboards, alerting |
| **The cost that does not appear in the table** | **Recovery time during an incident without out-of-band access; a fleet-wide credential rotation forced by one compromise; an unanswerable advisory question during a mandated patch window; a compromise that survives OS reimaging** | **Design and run cost above** |

The comparison is deliberately asymmetric in the way real comparisons are: the do-minimum column looks cheaper because the expensive items in its column are deferred and unbudgeted, and they arrive as incident costs at the worst moment. The honest framing for a business case is not "which is cheaper" but "which costs are we choosing to pay on a schedule, and which are we choosing to pay during an incident."

### Recommendation, and what was deliberately *not* automated

**Recommended:** build the brokered, segmented architecture; adopt certificate-based authentication for automation and per-device vaulted credentials for the remainder; run the DMTF validators as the conformance gate for every platform family; make firmware lifecycle an owned process with a single delivery path and a durable record; monitor the management plane's own availability and subscribe to its security and event registries.

**Deliberately NOT automated — and why.** This list is as important as the recommendation, because the discipline of not automating is what makes the rest defensible:

- **Power control (`ComputerSystem.Reset`) is not automated on in-service nodes.** The standard makes it a single `POST` (section 8); the correct guardrail is a human approval for production nodes and automation only for nodes confirmed out of service. Cymbal accepts a slower MTTR for power actions in exchange for not having a script that can power off the payments estate on a bad condition.
- **Automatic firmware remediation is not enabled — no "detect drift, auto-update" loop.** Firmware changes are changes: approved, windowed, piloted. An automatic loop against a fleet this size contains a propagating-failure mode that no amount of tooling confidence justifies.
- **BIOS/attribute baselines are reported, not enforced.** Given the vendor-specific attribute registries (section 6), enforcement would mean a per-vendor enforcement engine whose failure modes are not yet understood. Report drift, alert on it, change it through the normal process.
- **Virtual media is not exposed to general automation.** Boot-from-controlled-image is a recovery facility and a host-takeover primitive. It stays available to a small, privileged, logged set of operations, not to a generic pipeline.
- **Push event destinations are not pointed directly at production SIEM endpoints.** Events terminate at a collector inside the management segment and are re-emitted outward, so a compromised BMC cannot reach into the production monitoring path.
- **No OEM-only capability is a dependency.** Where a vendor exposes a needed function only through `Oem`, that is recorded as an accepted interoperability risk with a named owner and a review date, rather than silently becoming load-bearing in the tooling.

---

## 12. Anti-patterns

Each entry: **symptom** (what you observe) / **cause** (what actually produced it) / **guardrail** (the control that prevents it).

### 12.1 The management interface reachable from the production network

- **Symptom:** BMC addresses respond to a request from a production workload; the "management VLAN" is a naming convention; the firewall rule that prevents it exists but has a broad exception.
- **Cause:** the management network was designed for operator convenience, in an era when the BMC segment was assumed trusted; segmentation was never scoped as a security control.
- **Guardrail:** dedicated out-of-band segments on the BMC's own physical ports; default-deny allow-lists; no default route to the internet; and a **periodic, automated test** that a request originating from a production segment fails. A control that is not tested is a control that has already silently lapsed. This is the CISA/NSA advisory's own recommended mitigation, named first in its list of commonly-overlooked options.

### 12.2 One shared credential across a fleet

- **Symptom:** a single service account on every BMC; the same password in a script, a playbook and a monitoring config; rotation attempted once and abandoned.
- **Cause:** credentials are treated as configuration, not secrets; there is no vault-integrated retrieval path and no per-device distinctness, so rotation has a fleet-wide blast radius.
- **Guardrail:** client certificates for automation (no static secret to share); per-device vaulted credentials where passwords remain; rotation on schedule *and* on personnel change; and monitoring of authentication-failure messages across the fleet so credential abuse is detectable. Segregation of duties is impossible under a shared credential, which makes this an audit finding and not only a security one.

### 12.3 Firmware tracked in a spreadsheet

- **Symptom:** a spreadsheet, a survey that covered part of the estate, and a last-modified date measured in years; the inability to answer an advisory question inside a mandated window.
- **Cause:** firmware was treated as a property of the hardware purchase rather than as continually changing software; no inventory pipeline, no update record.
- **Guardrail:** read firmware inventory from the devices on a schedule (the `UpdateService` firmware inventory, and `SoftwareInventory`), route updates through one path, and record per-serial-number application. Then **rehearse the advisory question** — measure how long the answer takes. If it takes days, the guardrail has not been built.

### 12.4 Buying on an unqualified "supports Redfish" claim

- **Symptom:** the procurement document says "must support Redfish"; the vendor's response says "yes"; six months later the automation works on one platform and needs a rewrite for the other two.
- **Cause:** "supports Redfish" answers none of the three questions in section 4. It does not name a specification version, a schema bundle, or a profile — so it specifies nothing testable, and conformance is not interoperability.
- **Guardrail:** require the **three answers in writing** — which DSP0266 specification version, which DSP8010 schema bundle (and DSP8011 registry bundle), which DSP8013 interoperability profile — and require the **Interop Validator report**, plus permission to re-run the validators on a unit you own. Make the answers acceptance criteria, not informational.

### 12.5 Relying on an OEM extension that only one vendor implements

- **Symptom:** the tooling has a `if vendor == X` branch that is the only way to accomplish a required operation; a platform refresh breaks the runbook; the same operation has no standard path.
- **Cause:** the standard's `Oem` mechanism (DSP0266 §9.6.15, §9.8) is deliberately permissive — a vendor may legitimately implement a capability only through a vendor namespace. Conformance is a statement about payload shape, not coverage, so this is legal and still fatal to portability.
- **Guardrail:** treat every OEM-only capability as a **recorded, owned, time-bounded risk**, not an implementation detail. Ask for the standard mechanism, and where none exists, log it against the vendor and review it — the standard adds capabilities over time, and a new release may close the gap. Never let an OEM-only path become the sole path for a control you depend on in an incident.

### 12.6 A dormant open-source BMC dependency left in the architecture

- **Symptom:** a component in the management path whose last release predates the current schema bundle; a Python 2-era client; a simulator or test tool in a production pipeline; nobody knows who maintains it.
- **Cause:** the project was active when it was adopted, and its maintenance status was never re-checked. Redfish integrations are mappings between a moving standard and moving vendor firmwares; the mapping rots, and it does so silently.
- **Guardrail:** record, for every Redfish dependency, **who maintains it and when it was last released**, and re-check on a schedule. Two specific cautions from section 7: the DMTF Protocol Validator is the least active of the three validators (2022-era copyright, fewest commits) — use it, but do not build a gate on it alone; and OpenStack's **`sushy-tools` states in its own README that it "is not designed for use outside of development and testing environments"** — a live project that must still never appear in the production architecture. Dormant (or explicitly non-production) dependencies must be replaced, not documented.

### 12.7 A management plane with no monitoring of its own availability

- **Symptom:** no BMC availability check; no event subscription; the first indication that out-of-band management is broken arrives during an incident, when someone attempts to power-cycle a node and cannot.
- **Cause:** monitoring was scoped to the services that carry traffic, and the management plane carries none — so it was omitted. And because losing it takes no service down (section 10), nothing ever pages anyone.
- **Guardrail:** monitor the management plane as a service in its own right: BMC reachability and session-establishment success from the broker, certificate expiry dates, event-delivery health (the `EventSubscriptionResumed` and `EventDeliveryFailed` messages from the 2026.2 `EventService` registry exist for this), authentication-failure patterns (`AccountSecurity` registry), and firmware-drift status. **Then record the management plane in the resilience analysis as a dependency of the recovery capability**, per [operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) — because its loss does not break the service, it removes the ability to fix it.

---

## 13. Claims audit

Every DMTF document number and version, every resource path and property, every vendor support statement and every security claim is audited below. Source, date, and a quality rating where **A** = primary (DMTF register, the specification itself, or the vendor's/project's own documentation), **B** = primary-adjacent (DMTF's own release materials or repository metadata), **C** = third-party or indirect, **U** = unverified.

### DMTF documents — verified at the register on 21 September 2026

| Identifier | Exact title | Version | Publication date | Source | Quality |
| --- | --- | --- | --- | --- | --- |
| **DSP0266** | Redfish Specification | **1.25.0** (current) | 14 Sep 2026 | dmtf.org/dsp/DSP0266 register; DSP0266_1.25.0 HTML (internal date 2026-08-07, supersedes 1.24.1) | A |
| **DSP0266** | Redfish Scalable Platforms Management API Specification | 1.0.0 (first edition) | 04 Aug 2015 | dmtf.org/dsp/DSP0266 register | A |
| **DSP0266** | Redfish API Specification (retitled edition) | 1.6.0 | 20 Sep 2018 | dmtf.org/dsp/DSP0266 register | A |
| **DSP0266** | Redfish Specification (title stabilised) | 1.11.0 | 15 Jun 2020 | dmtf.org/dsp/DSP0266 register | A |
| **DSP0268** | Redfish Data Model Specification | **2026.2** (current) | 14 Sep 2026 | dmtf.org/dsp/DSP0268 register | A |
| **DSP0268** | Redfish Schema Supplement (former title) | 2022.1 | 24 May 2022 | dmtf.org/dsp/DSP0268 register | A |
| **DSP8010** | Redfish Schema Bundle | **2026.2** (current) | 14 Sep 2026 | dmtf.org/dsp/DSP8010 register | A |
| **DSP8010** | Redfish Schema | 1.0.0 (first edition) | 04 Aug 2015 | dmtf.org/dsp/DSP8010 register | A |
| **DSP8011** | Redfish Standard Registries Bundle | **2026.2** (current) | 14 Sep 2026 | dmtf.org/dsp/DSP8011 register | A |
| **DSP8013** | Redfish Interoperability Profiles Bundle | current version **not confirmed** | — | Title and contents confirmed from DMTF's own `Redfish-Publications` README and the DSP8010 readme; the register page (dmtf.org/dsp/DSP8013) returned HTTP 520 on 21 Sep 2026 | B (title) / U (version) |
| **DSP0272** | Redfish Interoperability Profiles Specification | **1.10.0** (current) | 17 May 2026 | dmtf.org/dsp/DSP0272 register | A |
| **DSP0272** | Redfish Interoperability Profiles Specification | 1.0.0 (first edition) | 08 Jan 2018 | dmtf.org/dsp/DSP0272 register | A |
| **DSP0270** | Redfish Host Interface Specification | **1.3.1** (current) | 21 Sep 2023 | dmtf.org/dsp/DSP0270 register | A |
| **DSP0218** | Platform Level Data Model (PLDM) for Redfish Device Enablement | version not confirmed | — | DMTF `Redfish-Publications` README and DSP0266 normative references name the document and title | A (title) / U (version) |
| **DSP2046** | Redfish Resource and Schema Guide | **2026.2** | 14 Sep 2026 | redfish.dmtf.org/essentials download links; Redfish Release 2026.2 overview | A |
| **DSP2053** | Redfish Property Guide | 2026.2 (per bundle convention) | — | Named in DMTF's DSP8010 readme and Release 2026.2 overview | A (title) / B (version) |
| **DSP2065** | Redfish Message Registry Guide | 2026.2 (per bundle convention) | — | Named in DMTF's DSP8010 readme and Release 2026.2 overview | A (title) / B (version) |
| **DSP2043** | Redfish Mockups Bundle | version not confirmed | — | DMTF `Redfish-Publications` README; DSP8010 readme | A (title) / U (version) |
| **DSP2044** | Redfish White Paper | version not confirmed | — | DMTF `Redfish-Publications` README; DSP8010 readme | A (title) / U (version) |
| **DSP2050** | Redfish Composability (white paper) | 1.0.0 | — | redfish.dmtf.org/education links `DSP2050_1.0.0.pdf` | A |

**Rejected / dropped hypotheses.** The following candidate identifiers were treated as hypotheses and **could not be confirmed, so they are dropped and must not be cited**: a "Redfish Conformance Program" document identifier; **DSP2032 / BIND**; **DSP2052**. No evidence for any of these was found at the DMTF register or in DMTF's own publications. They are recorded as dropped rather than as unverified-and-plausible, because a plausible-looking document number that does not exist is worse than no number.

### Predecessor status — verified at the source on 21 September 2026

| Claim | Source | Date of the source | Quality |
| --- | --- | --- | --- |
| "No further updates to the IPMI specification are planned or should be expected"; promoters encourage "a more modern systems management interface"; DMTF's Redfish named as an example; Redfish uses the IPMI Group ID value 0x52 (ASCII 'R') | DMTF's IPMI standards page, carrying the joint message of the IPMI Promoters — Dell, Hewlett Packard Enterprise, NEC Corporation, Intel Corporation | Joint message issued **June 2020**; page read 21 Sep 2026 | A |
| The statement "applies only to the IPMI specification, and should have no impact on existing IPMI implementations" | as above | June 2020 | A |
| BMCs "operate separately from the operating system (OS) and firmware to allow for remote management and control, even when the system is shut down"; "Hardened credentials, firmware updates, and network segmentation options are often overlooked, leading to a vulnerable BMC" | CISA alert page (2023/06/14) announcing the joint CISA + NSA Cybersecurity Information Sheet, "Harden Baseboard Management Controllers" | Released **14 Jun 2023**; read 21 Sep 2026 | A |

### Specification content — verified against DSP0266 v1.25.0

| Claim | Where in the source | Quality |
| --- | --- | --- |
| BMC defined as an embedded device completing out-of-band monitoring/management tasks | DSP0266 §3.1.1 | A |
| Scope: "interoperable, multivendor, remote, and out-of-band capable interface" | DSP0266 §1 Scope | A |
| Design properties: REST-based interface, data-oriented, protocol/model separation, hypermedia service root, OpenAPI v3.0, OData conventions | DSP0266 §5.4.1–§5.4.6 | A |
| Asynchronous operations, eventing mechanism, actions, service discovery, remote access | DSP0266 §5.5.1–§5.5.5 | A |
| Services shall support both HTTP Basic and Redfish session login authentication | DSP0266 §13.3.1 | A |
| "All requests that use HTTP Basic authentication shall require HTTPS" | DSP0266 §13.3.3 | A |
| Specification's own note on IETF-documented concerns with Basic auth; `HTTPBasicAuth` can restrict/disable | DSP0266 §13.3.3 | A |
| Session creation at `/redfish/v1/SessionService/Sessions`; response carries `X-Auth-Token` and `Location`; token "indistinguishable from random" | DSP0266 §13.3.4.1–§13.3.4.2 | A |
| Sessions time out on idleness rather than by token expiry | DSP0266 §13.3.4.3 | A |
| Logout by `DELETE` on the `Session` resource; forced logout and account-disable semantics | DSP0266 §13.3.4.4 | A |
| Client certificate authentication (mTLS per RFC 8446; CAC/PIV per NIST FIPS 201-3); OCSP revocation check; `RespondToUnauthenticatedClients` | DSP0266 §13.3.5 | A |
| Required standard roles and privileges (`Administrator`, `Operator`, `ReadOnly` with listed privileges) | DSP0266 §13.4.2.1, Table 43 | A |
| "Two roles with the same privileges shall behave equivalently"; ETag privilege parity | DSP0266 §13.4.1–§13.4.2 | A |
| Certificates: shall support replacing the default certificate; shall use X.509 v3 (RFC 5280) | DSP0266 §13.1.3 | A |
| Sensitive data: HTTPS-only for sensitive operations; sensitive properties `null`; `404` may replace `401`/`403` where the URI carries sensitive data | DSP0266 §13.2 | A |
| Unauthenticated resources limited to service root, metadata document, OData service document, OpenAPI YAML, and the `/redfish` version object | DSP0266 §13.3.2.1 | A |
| Services "shall not use HTTP cookies to authenticate any activity" | DSP0266 §13.3.2.2 | A |
| `$expand` and `$select` semantics and examples | DSP0266 §7.3.2–§7.3.3 | A |
| Actions: representation, `target`, invocation by `POST` | DSP0266 §9.6.14, §9.6.14.1–§9.6.14.2 | A |
| `Oem` property and OEM extension clauses | DSP0266 §9.6.15, §9.8.1–§9.8.8 | A |
| `Links`, `Members`, `Members@odata.count`, `Members@odata.nextLink`, `Id`, `Name`, `Status` | DSP0266 §9.6.10–§9.6.16 | A |
| Deep operations (`.Deep` `POST`/`PATCH` forms) | DSP0266 §7 examples (`Roles.Deep`, `EventService/Subscriptions.Deep`, `Systems/47832.Deep`) | A |
| Resource aliases with tilde prefix and `@Redfish.Alias` | Redfish Release 2026.2 overview; DSP0266 v1.25.0 change log | A |
| Task monitor URI form `/redfish/v1/TaskService/TaskMonitors/<TaskMonitorId>` | DSP0266 §7 table of URIs | A |

**Resource paths and properties used in this guide, all appearing in DSP0266 v1.25.0 or the schema index (no path is constructed):** `/redfish/v1/`, `/redfish/v1/Systems`, `/redfish/v1/Systems/1`, `/redfish/v1/Systems/1/Actions/ComputerSystem.Reset`, `/redfish/v1/Systems/1/Storage/SATAEmbedded/Volumes/2`, `/redfish/v1/Systems/1/LogServices/FaultLog/Entries`, `/redfish/v1/Chassis/A88`, `/redfish/v1/Chassis/A88/Assembly`, `/redfish/v1/Chassis/MultiBladeEncl/Thermal`, `/redfish/v1/AccountService`, `/redfish/v1/AccountService/Roles/Administrator`, `/redfish/v1/SessionService`, `/redfish/v1/SessionService/Sessions/1`, `/redfish/v1/EventService/Subscriptions`, `/redfish/v1/UpdateService`, `/redfish/v1/UpdateService/FirmwareInventory`, `/redfish/v1/UpdateService/update-multipart`, `/redfish/v1/TaskService/TaskMonitors/<TaskMonitorId>`, `/redfish/v1/Systems/~MySystem/Processors/~MyGPU` (alias example, v1.25.0 only). Properties: `@odata.id`, `@odata.type`, `@odata.etag`, `@odata.context`, `@odata.navigationLink`, `Id`, `Name`, `Description`, `MemberId`, `Members`, `Members@odata.count`, `Members@odata.nextLink`, `Links`, `Links.ManagedBy`, `Actions`, `Oem`, `Status`, `HTTPBasicAuth`, `MFA`, `ClientCertificate`, `RespondToUnauthenticatedClients`, `RoleId`, `AssignedPrivileges`, `OemPrivileges`, `IsPredefined`, `AccountTypes`, `MultipartHttpPushUri`, `ProtocolFeaturesSupported`, `@Redfish.Settings`, `@Redfish.ActionInfo`, `@Redfish.Alias`, `TargetsParameterSupported` (and sibling `UpdateServiceCapabilities` properties), `ImageState`, `Armed`, `ArmedTargets`. **All verified in the specification or the 2026.2 release overview.** No property name in this guide was invented.

**Schema versions cited from the 2026.2 release overview (quality A, DMTF's own release deck):** `AccountService` v1.19.0, `ComputerSystem` v1.29.0, `Chassis` v1.29.0, `PrivilegeRegistry` v1.3.0, `ServiceRoot` v1.22.0, `Session` v1.9.0, `SoftwareInventory` v1.15.0, `Storage` v1.23.0, `UpdateServiceCapabilities` v1.1.0, `VirtualMedia` v1.7.0, `Certificate` v1.11.0, `ActionInfo` v1.6.0, `Resource` v1.25.0. **Registries:** `Device` v1.0.0 (new), `EventService` v1.0.0 (new), `Base` v1.24.0, `ManagerEvent` v1.2.0, `Platform` v1.5.0, `AccountSecurity` v1.2.0, `Power` v1.4.0, `Update` v1.5.0, `StorageDevice` v1.7.0, `ServiceCommunications` v1.2.0.

### Security claims

| Claim | Source and date | Status |
| --- | --- | --- |
| BMCs are trusted components operating independently of the OS, allowing remote management even when the system is shut down | **CISA + NSA joint Cybersecurity Information Sheet "Harden Baseboard Management Controllers", released 14 June 2023** | **Verified** — named authority |
| "Hardened credentials, firmware updates, and network segmentation options are often overlooked, leading to a vulnerable BMC" | Same CSI, 14 June 2023 | **Verified** — named authority |
| A vulnerable BMC "broadens the attack vector", offering "a beachhead with pre-boot execution potential" | Same CSI, 14 June 2023 | **Verified** — named authority |
| The specification itself flags Basic authentication's weaknesses and allows disabling it | DSP0266 v1.25.0 §13.3.3 | **Verified** — primary |
| Self-signed certificates are the expected out-of-box state (a BMC generates one when none is present) | OpenBMC `bmcweb` README, checked 21 Sep 2026 | **Verified** (for bmcweb specifically; generalising to all vendors would be an inference) |
| Named CVEs exist against BMC firmware (e.g. the MegaRAC family) | Directionally supported by AMI's public product/advisory context, but **no CVE identifier is asserted in this guide** | **Flagged** — deliberately not enumerated |
| Any specific institution compromised via its management plane | — | **Rejected** — no source found; not asserted |
| Prevalence/market-share figures for BMC compromise or Redfish adoption | — | **Rejected** — no source found; not asserted |
| The predecessor (IPMI) is formally abandoned by its owner | — | **Rejected** — no withdrawal notice found; OpenBMC and Lenovo both continue to document IPMI support |
| Specific performance or protocol-overhead comparisons between IPMI and Redfish | — | **Rejected** — no measurement source found; not asserted |

### Vendor and project statements

| Claim | Source and date | Quality |
| --- | --- | --- |
| Lenovo XCC "provides a Redfish compliant set of easy-to-use REST APIs" | `pubs.lenovo.com/xcc/rest_api`, checked 21 Sep 2026 | A — vendor's own docs (no specification version stated, and none inferred) |
| Lenovo XCC is the successor to IMM2; XCC documents an IPMI interface | `pubs.lenovo.com/xcc/`, checked 21 Sep 2026 | A |
| Supermicro publishes Redfish User Guide v6.1 and Reference Guide v4.0 | `supermicro.com/en/solutions/management-software/redfish`, checked 21 Sep 2026 | A |
| Fujitsu publishes an "iRMC Redfish API" specification; products include PRIMERGY CX400 M4, PRIMEQUEST 3400E | `support.ts.fujitsu.com` SWP1236312 / SWP1249205, checked 21 Sep 2026 | A (document entry dated 2017/07 with a 03/05/2020 revision) |
| AMI MegaRAC SP-X is "mature and widely deployed", "trusted by top-tier ODMs, OEMs and hyperscalers"; SP-X LTS 13.5 released with Intel 4th Gen Xeon / AMD 4th Gen EPYC reference-board support | `ami.com/products/megarac/` + AMI release announcement, checked 21 Sep 2026 | A |
| HPE iLO supports Redfish, with a documented specification version | Attempted URL returned 404 | **U — not verified; nothing asserted** |
| Dell iDRAC supports Redfish, with a dated generation/firmware mapping | Attempted developer-portal URL returned no content | **U — not verified; nothing asserted** |
| Cisco CIMC Redfish support statement | Not retrieved | **U — not verified; nothing asserted** |
| Huawei iBMC Redfish support (firmware 6.22) | Third-party `redfish_exporter` tested-device list, checked 21 Sep 2026 | **C — indirect, recorded as such** |
| Inspur BMC Redfish support statement | Not retrieved | **U — not verified; nothing asserted** |
| OpenBMC is a Linux Foundation project with a TOF; active development | `openbmc.org`, `github.com/openbmc/*`, checked 21 Sep 2026 (repo commit counts and per-vendor layers) | A/B |
| OpenBMC lists "OpenCompute Redfish Compliance" as a *feature in progress* | `github.com/openbmc/openbmc` README, checked 21 Sep 2026 | A — project's own statement |
| `bmcweb` supports Basic (RFC 7617), cookie, mutual TLS, session, and "XToken based authentication conformant to Redfish DSP0266"; authorization conforms to the Redfish PrivilegeRegistry | `github.com/openbmc/bmcweb` README, checked 21 Sep 2026 | A |
| DMTF `python-redfish-library` is maintained; publishes as `redfish` | `github.com/DMTF/python-redfish-library`, checked 21 Sep 2026 | A |
| `Redfish-Service-Validator` is GET-only and validates against CSDL | `github.com/DMTF/Redfish-Service-Validator` README, checked 21 Sep 2026 | A |
| `Redfish-Interop-Validator` release 3.0.0, 18 Sep 2026 | `github.com/DMTF/Redfish-Interop-Validator` releases, checked 21 Sep 2026 | A |
| `Redfish-Protocol-Validator` is the least active of the three (copyright 2020-2022, 241 commits) | `github.com/DMTF/Redfish-Protocol-Validator`, checked 21 Sep 2026 | A |
| `sushy` scope limited to OpenStack Ironic; "BMCs are very flaky" | `github.com/OpenStack/sushy` README, checked 21 Sep 2026 | A |
| `sushy-tools` "is not designed for use outside of development and testing environments. Please do not run sushy-tools in a production environment of any kind." | `github.com/OpenStack/sushy-tools` README, checked 21 Sep 2026 | A |
| `redfish_exporter` tested-device list with firmware versions | `github.com/jenningsloy318/redfish_exporter` README, checked 21 Sep 2026 | C — community project; last-commit date **U** |
| Ansible Redfish collection; Terraform provider | — | **U — not verified in this pass; not named** |
| Version of the predecessor specification (IPMI) and its owning body | Not verified at a primary register in this pass | **U — recorded, not asserted** |

### Structural and programme claims

| Claim | Source and date | Quality |
| --- | --- | --- |
| DSP8010 ships schema in three functionally equivalent languages: CSDL (XML), JSON Schema, OpenAPI YAML, plus RDE BEJ dictionaries | DSP8010 2024.1 readme (DMTF), verified 21 Sep 2026 | A |
| DMTF provides three open-source conformance tools, and recommends running the Service Validator first | Redfish Release 2026.2 Overview (DMTF's own deck), Sep 2026 | A |
| The conformance tools are a public tooling programme rather than a DMTF-operated certification registry | Inference from DMTF's own materials, which describe tools and repositories, not a certification database; **no formal conformance-programme page could be retrieved** | **Flagged** — inference, labelled as such |
| `redfish.dmtf.org/profiles` served "Profiles coming soon." | Direct fetch, 21 Sep 2026 | A — observed |
| BIOS attribute registries are implementation-specific and not standardised | `AttributeRegistry` schema description quoted via the DMTF schema index, checked 21 Sep 2026 | A |
| `ActionInfo` exists because supported action parameters "can differ among vendors and even among resource instances" | `ActionInfo` schema description, DMTF schema index, checked 21 Sep 2026 | A |
| `TaskService`/`JobService` as a distinct resource type | `TaskService` and task monitors verified in DSP0266; a separate `JobService` type was **not** confirmed | **TaskService: verified. JobService: U — not confirmed, not used as a claim** |

---

## 14. What Could Not Be Verified

Stated plainly, because each of these is a place where a less careful guide would have invented something plausible.

**Vendor support statements that could not be sourced.** The HPE iLO Redfish documentation URL attempted (`servermanagementportal.ext.hpe.com/docs/redfishservices/ilos/`) returned **HTTP 404**, and the Dell developer-portal page attempted (`developer.dell.com/apis/2378/...`) returned **"No data found / Api version not found"**. Consequently **no HPE and no Dell support statement appears in this guide** — not their specification version, not their firmware mapping, not their schema bundle. Cisco and Inspur were not retrieved either. This is a real gap in a section that is supposed to be dated and attributed, and it is left as a gap rather than filled with a guess. The specific artefact that remains missing for *every* vendor surveyed is the triple a buyer actually needs: **product generation + BMC firmware release → supported DSP0266 specification version and DSP8010 schema bundle.** Vendors publish this per-generation in release notes and support matrices; it is not on a single consolidated page, and a buyer must request it.

**DSP8013's current version.** The *title* and *contents* of the Redfish Interoperability Profiles Bundle are confirmed from DMTF's own `Redfish-Publications` README and from the DSP8010 readme's annex, both of which describe it as "a bundle of published Redfish interoperability profile documents as well as supporting schema and sample documents used for creating profiles." The register page for DSP8013 returned **HTTP 520 (Cloudflare origin error)** on 21 September 2026, so its current version number and publication date are **unconfirmed**. Do not cite a DSP8013 version from this guide.

**The formal conformance programme.** DMTF's release materials describe *conformance testing tools* (three validators, each with a named repository). This pass could not retrieve a formal "Redfish Conformance Program" landing page, a certification registry, or a document identifier for such a programme — several search queries returned **empty results** (see the tool-limitation note below). The guide therefore describes what is evidenced — a public open-source validator programme, with the Service Validator recommended first — and labels as *inference* any statement that this constitutes a tooling programme rather than a certification mark. A reader who needs certainty here should contact DMTF or the Redfish Forum directly rather than rely on this guide's inference.

**The predecessor's specification identity.** The expansion of IPMI is normative in DSP0266 §3.1.2 (Intelligent Platform Management Interface), and the pairing with DCMI at version 2.0 is evidenced by OpenBMC's own claim of "Full IPMI 2.0 Compliance with DCMI" and by Lenovo's IPMI documentation chapter. **What is not verified is the specification document's own identifier, its current version, and its revision history.** A claim that it is at a particular revision was not confirmed at any primary register in this pass, so no such claim is made. Partially resolved since the first pass: the *owner group* is now evidenced — DMTF's IPMI standards page carries the joint message of the IPMI Promoters (Dell, Hewlett Packard Enterprise, NEC Corporation, Intel Corporation), issued June 2020, which confirms both the promoter set and the fact that Redfish uses the IPMI Group ID value 0x52 (see section 13). What remains unretrieved is the predecessor's own specification text, so any statement in this guide about its *architecture* still relies on the shape of its successor's stated design goals — which are documented — rather than on the predecessor's specification document itself.

**Maintenance status of two tooling projects.** The community Prometheus exporter `redfish_exporter` exists, is 87-forks deep, and carries an extremely useful dated tested-device list — but **its last-commit date was not verified**, so its current level of maintenance is unestablished. Separately, while the DMTF Protocol Validator is clearly maintained (it has CI workflows, a release process and unit tests), its copyright header reads "2020-2022" and it has the fewest commits of the three validators; the guide describes it as *less actively developed* on that evidence, which is a characterisation rather than a measured dormancy assessment.

**Ansible and Terraform integrations.** No specific Ansible Redfish module or collection and no Terraform Redfish provider were retrieved and date-checked. Both categories exist in the ecosystem; **none is named or recommended here**, because naming a project whose maintenance status was not checked is exactly the error this guide warns against in section 12.

**CVE enumeration.** The guide asserts that named CVEs exist against BMC firmware implementations, and names no identifier. That is deliberate: enumerating CVE numbers without verifying each against an authoritative advisory would breach the standard this guide holds itself to, and a wrong CVE identifier in a procurement or risk document is actively harmful. Use a CVE database and the vendor's advisory page.

**Tool limitation — searches that returned empty.** Multiple `web_search` queries returned **zero results** in this pass, including queries for a DMTF conformance programme, for DSP8011 details, for Redfish interoperability-profile material, and for vendor documentation (HPE iLO, Dell iDRAC). **An empty search result is a tool limitation, not evidence of absence**, and nothing in this guide is concluded from a blank result. Where a search came back empty, the guide fell back to `web_extract` against known primary URLs (the DMTF register pages, the specification's own HTML, DMTF's release overview PDF, the schema index, the registries index, and project repositories) — and each such fallback is noted at the point of use. Two DMTF URLs additionally failed for infrastructure reasons rather than absence: `www.dmtf.org/documents/redfish` returned a 404 page and `www.dmtf.org/dsp/DSP8013` returned a Cloudflare 520.

---

## 15. Glossary, cross-references and closing summary

### Glossary

| Term | Definition |
| --- | --- |
| **AccountService** | The Redfish service resource holding accounts, roles and authentication configuration (including `HTTPBasicAuth` and `MFA`). |
| **Action** | A non-CRUD operation exposed on a resource through its `Actions` property, with a `target` URI, invoked by `POST`. |
| **`@Redfish.ActionInfo`** | An annotation mechanism by which a resource advertises an action's supported parameters, types and allowable values, so a client need not guess. |
| **`@Redfish.Alias`** | New in v1.25.0. A writable annotation carrying a user-defined alias for a collection member, addressable with a tilde prefix (`~name`). |
| **BMC — baseboard management controller** | The embedded device or service that completes out-of-band monitoring and management tasks, typically an independent microprocessor or SoC with associated firmware (DSP0266 §3.1.1). |
| **CertificateService / CertificateLocations** | Resources for finding all certificates on a service and installing, replacing or enrolling them. |
| **Collection** | A resource whose members are other resources, exposed through `Members`, `Members@odata.count` and `Members@odata.nextLink`. |
| **CSDL** | OData Common Schema Definition Language; the XML form of the Redfish schema in DSP8010. |
| **DCMI** | Data Center Management Interface; the companion specification standardising a server-oriented subset over the predecessor protocol. |
| **Deep operation** | A `POST`/`PATCH` to a `.Deep` form of a collection URI, applying a change across many members in one request. |
| **DMTF** | Distributed Management Task Force; the not-for-profit consortium that publishes Redfish through its Redfish Forum. |
| **DSP** | DMTF Specification/document prefix. DSP0266 = Redfish Specification; DSP8010 = schema; DSP8011 = registries; DSP0268 = data model; DSP0272 = profiles structure; DSP8013 = profiles bundle. |
| **EventService** | The service that manages event subscriptions and pushes events to client-supplied destinations. A new `EventService` message registry (v1.0.0) arrived in 2026.2 for subscription lifecycle events. |
| **Firmware inventory** | The list of installed firmware versions, readable through `UpdateService/FirmwareInventory` (and modelled further by `SoftwareInventory`). |
| **Interoperability profile** | A document defining a minimum set of functionality a Redfish implementation must expose for a given purpose; structured per DSP0272, published in DSP8013. |
| **IPMI** | Intelligent Platform Management Interface; the predecessor out-of-band management interface. |
| **KVM-IP** | Keyboard, video, mouse redirection over IP; the remote console facility (DSP0266 §3.1.3). |
| **`Members@odata.nextLink`** | The paging link on a collection, so a client can traverse members beyond the first page. |
| **Message registry** | The machine-readable catalogue of messages a service can emit (errors, events), so a client can interpret a `MessageId` rather than log an opaque string. Published in DSP8011. |
| **OData** | The Open Data Protocol conventions Redfish follows for query options (`$select`, `$expand`, `$filter`) and metadata (`$metadata`). |
| **`Oem`** | The vendor extension container inside a standard resource, with a schema-enforced naming pattern; the sanctioned place for vendor-specific data. |
| **Out-of-band management** | Management over a path that does not traverse the host OS or CPU, and typically not the host network stack. |
| **RDE — Redfish Device Enablement** | Mapping of Redfish schemas and properties into PLDM messages (DSP0218); relevant to component-level/device management over a sideband fabric. |
| **Resource** | An addressable object with a type and properties, identified by `@odata.id`. |
| **Resource tree** | The hierarchy reachable by following links from the service root. |
| **`@Redfish.Settings`** | The annotation pointing at a settings object a client modifies to change pending configuration, applied at a defined point (such as reset) rather than immediately. |
| **Service root** | The single entry point of a Redfish service, conventionally `/redfish/v1/`. |
| **Session** | A login session created by `POST` to the session collection, authenticated thereafter with an `X-Auth-Token` header; idle-timeout based. |
| **SessionService** | The resource managing login sessions and their lifecycle. |
| **`SimpleUpdate`** | The standard firmware update action on `UpdateService`; the alternative is a multipart HTTP push to the service-advertised `MultipartHttpPushUri`. |
| **Status** | The shared object carrying `State` and `Health` on resources. |
| **Task / task monitor** | The representation of a long-running operation and the URI (`/redfish/v1/TaskService/TaskMonitors/<TaskMonitorId>`) a client polls to follow it. |
| **TLS / mTLS** | Transport Layer Security; mutual TLS is the client-certificate authentication flow DSP0266 §13.3.5 defines. |
| **UpdateService** | The service for firmware update and firmware inventory. |
| **`X-Auth-Token`** | The response header carrying a session authentication token, "indistinguishable from random", used on subsequent requests. |

### Cross-references

Same-directory references are plain filenames; `../` is the repository root.

- **Nearest use case:** [on_prem_llm_deployment_guide.md](on_prem_llm_deployment_guide.md) — names the out-of-band management plane for GPU server estates.
- **Facilities and data-centre design:** [data_center_guide.md](data_center_guide.md), [singapore_data_centres_guide.md](singapore_data_centres_guide.md), [l2_design_financial_data_centers_guide.md](l2_design_financial_data_centers_guide.md).
- **Platforms that consume Redfish:** [nutanix_vs_openstack_guide.md](nutanix_vs_openstack_guide.md) — OpenStack Ironic manages bare metal through Sushy.
- **Automation mechanics:** [ansible_vs_ansible_tower_guide.md](ansible_vs_ansible_tower_guide.md).
- **Security discipline and supply chain:** [cybersecurity_guide.md](cybersecurity_guide.md), [security_by_design_guide.md](security_by_design_guide.md), [secops_guide.md](secops_guide.md), [beyond_zero_enterprise_security_guide.md](beyond_zero_enterprise_security_guide.md), [zero_trust_network_architecture_guide.md](zero_trust_network_architecture_guide.md).
- **Banking and resilience:** [operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md), [enterprise_risk_management_guide.md](../banking/enterprise_risk_management_guide.md), [risk_management_models_guide.md](../banking/risk_management_models_guide.md).
- **Vendor firms:** the vendor guides own their firms — for example [nec_asia_pacific_guide.md](nec_asia_pacific_guide.md).

### Closing summary

Redfish is the rare standard that changed an industry by changing the *shape* of an interface rather than any single feature of it. Out-of-band management exists because a control path that works when the operating system does not is not a convenience but a requirement — and for two decades that requirement was met by a command protocol whose extension mechanism *was* its protocol, so that interoperability was a per-vendor, per-generation, per-tool negotiation. Redfish replaced that with a hypermedia resource model: a well-known service root, resources that carry their own identity and relationships, collections that page, actions that advertise their own parameters, events driven by registries that make a message machine-interpretable, and a data model published as schemas in three languages with separately versioned registries beside it. The interface stopped describing messages a device might accept and started describing what a device *is*.

The practical consequences are equally clear, and they are the parts that get left out. "Supports Redfish" specifies nothing, because conformance is a statement about payload shape while interoperability is a statement about coverage — so the only adequate questions are which specification version, which schema bundle, and which interoperability profile, with the validator report to show for the third. That is the finding this guide most wants a reader to carry into a procurement conversation, and the reason section 4 gets the space it does. The security model is coherent and candid — session authentication over token-free Basic, TLS mandatory where credentials move, a privilege model with three required roles, client-certificate authentication with revocation checking, and the specification itself flagging Basic's weaknesses and normatively requiring the default certificate to be replaceable — and it still does not make the management plane safe, because a BMC is a computer that controls other computers without their operating system's cooperation. Hence the CISA and NSA guidance, hence segmentation, hardened credentials and firmware provenance, hence the brokering and certificate patterns that make segregation of duties expressible at all. And hence the resilience point that is easy to say and easy to skip: losing out-of-band management takes no service down, and takes away the ability to bring one back.

Build the fence, replace the default certificate, know what firmware is running and who signed it, put a broker in the path, and remember that the standard's achievement is not a better set of commands. It is the model, not the command.
