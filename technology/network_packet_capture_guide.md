# Network Packet Capture: The Documentation of the Wire

Jack Liu Shurui, Solution Architect

> **Author:** Jack Liu Shurui — Solution Architect, Cymbal Bank
> **Topic:** Network packet capture — where you can capture and what each point costs you; the capture mechanics and the kernel path; loss and timestamps, the two problems that quietly invalidate captures; the file format and storage question; the volume arithmetic; the analysis tooling with dated statuses; the decryption problem; the security and privacy consequences; and the regulated-enterprise angle
> **Audience:** Solution Architects, Network and Security Engineers, Observability/Network Operations Teams, Incident Responders, Banking IT Decision-Makers
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Companions (sibling guides in this folder):** [Zero Trust Network Architecture](zero_trust_network_architecture_guide.md), [OpenShift SCC Comprehensive Guide](openshift_scc_comprehensive_guide.md), [OpenShift SCC Service Account Guide](openshift_scc_service_account_guide.md), [Secure Red Hat OpenShift](secure_red_hat_openshift_guide.md), [Cybersecurity](cybersecurity_guide.md), [Security by Design](security_by_design_guide.md), [Beyond Zero: Enterprise Security](beyond_zero_enterprise_security_guide.md), [Penetration Testing Execution Standard](penetration_testing_execution_standard_guide.md), [L2 Design for Financial Data Centres](l2_design_financial_data_centers_guide.md), [Data Center](data_center_guide.md), [Singapore Data Centres](singapore_data_centres_guide.md), [Axway Transfer CFT](axway_transfer_cft_guide.md), [IBM MQ Disaster Recovery](ibm_mq_disaster_recovery_guide.md)
> **Companions (`../banking/`):** [FIX Protocol](../banking/fix_protocol_guide.md), [SWIFT Alliance Access](../banking/swift_alliance_access_guide.md), [SWIFTNet FileAct](../banking/swiftnet_fileact_guide.md), [MAS Regulations & Guidelines](../banking/mas_regulations_guidelines_guide.md), [Payment Rails](../banking/payment_rails_guide.md), [Market Data Consumption](../banking/market_data_consumption_guide.md)
> **Version:** 1.0 — September 2026
> **Verification convention:** ✅ = verified at source this pass (named source and date in §14); ⚠ = flagged, not asserted as fact; ⚠-unverified = plausible but unsourced and therefore excluded from the body.

---

# Table of Contents

1. [The Overview, the Identity and the Decoder](#1-the-overview-the-identity-and-the-decoder)
2. [Where You Can Capture, and What Each Point Costs You](#2-where-you-can-capture-and-what-each-point-costs-you)
3. [The Capture Mechanics and the Kernel Path](#3-the-capture-mechanics-and-the-kernel-path)
4. [The Loss Problem](#4-the-loss-problem)
5. [The Timestamp Problem](#5-the-timestamp-problem)
6. [The File Format, the Storage and the Indexing Question](#6-the-file-format-the-storage-and-the-indexing-question)
7. [The Volume Arithmetic](#7-the-volume-arithmetic)
8. [The Analysis Tooling, with Dated Statuses](#8-the-analysis-tooling-with-dated-statuses)
9. [The Decryption Problem](#9-the-decryption-problem)
10. [The Security and Privacy Consequences](#10-the-security-and-privacy-consequences)
11. [The Regulated-Enterprise and Banking Angle](#11-the-regulated-enterprise-and-banking-angle)
12. [The Cymbal Bank Worked Example](#12-the-cymbal-bank-worked-example)
13. [The Anti-Patterns](#13-the-anti-patterns)
14. [The Claims Audit](#14-the-claims-audit)
15. [What Could Not Be Verified](#15-what-could-not-be-verified)
16. [The Glossary, the Cross-References and the Closing Summary](#16-the-glossary-the-cross-references-and-the-closing-summary)

---

## 1. The Overview, the Identity and the Decoder

**The thesis in one line: a packet capture is the only evidence that outranks the logs, which is exactly why a capture that is quietly incomplete is more dangerous than no capture at all.**

Logs are written by the software that failed. A capture is written by the network that carried the traffic, before the application interpreted it, and it therefore survives the failure of the thing being investigated. That is the whole reason packet capture persists as a discipline in an era of distributed tracing, structured logging and flow telemetry — and it is also why a captured file carries more authority than it always deserves. A log is obviously partial: everyone knows the application only says what its author thought to say. A capture *looks* complete. Every packet in the file is real, correctly framed, and correctly sequenced. Nothing in the file announces what is missing from it.

### 1.1 What Packet Capture Is

Packet capture is the act of recording some or all of the frames or packets traversing a point in a network, together with metadata — chiefly a timestamp, a length, and the identity of the interface that observed them. The recording is made by a capture library through an operating-system facility, and written to a capture file for later analysis by a protocol analyzer. Every part of that sentence hides a cost, and the costs are what this guide is about.

### 1.2 What It Is For

Six jobs, and it is worth being clear which one you are doing, because they imply different capture points and different retention:

| Job | What the capture is doing | What breaks if the capture is incomplete |
| --- | --- | --- |
| Fault diagnosis | Showing what actually happened on the wire when two systems disagree | You chase a fault that is visible only in the packets you dropped |
| Protocol verification | Proving that a peer implements a specification as documented | You conclude "the peer is fine" from an incomplete sample |
| Performance attribution | Localising delay to a hop, a retransmission, or an application stall | You attribute loss to the application when the capture dropped it |
| Security investigation | Reconstructing an intrusion or exfiltration path at packet level | You assert absence of evidence as evidence of absence |
| Compliance and dispute evidence | Proving a message arrived, or did not, at a point in time | The exhibit is impeachable |
| Capacity and behaviour baselining | Establishing what "normal" looks like for a link or a flow | Your baseline is a measurement of your capture rig, not your network |

### 1.3 The Decoder

The vocabulary is small and each term is load-bearing.

- **Capture point.** The physical or logical place where a copy of the traffic is taken. The whole of §2.
- **Promiscuous mode.** A mode in which a NIC delivers frames not addressed to it. On Linux it is requested through the packet socket (`PACKET_ADD_MEMBERSHIP` with `PACKET_MR_PROMISC`), which the man page glosses as "receiving all packets on a shared medium (often known as 'promiscuous mode')" ✅ (man7.org packet(7), man-pages 6.19). It is not the same as **monitor mode**, which is an 802.11-only mode in which the radio delivers whole frames including 802.11 management and control frames; the Wireshark capture tool documents monitor mode as "supported only on IEEE 802.11 Wi-Fi interfaces, and supported only on some operating systems", with the warning that in monitor mode the adapter "might disassociate from the network with which it's associated" ✅ (dumpcap(1), Wireshark 4.6.8).
- **Capture filter vs display filter.** The capture filter is applied *before* the packet is copied — it is what the kernel runs to decide whether the packet ever reaches your file. The display filter is applied *after* the fact to what you already have. This distinction is the single most consequential one in the guide: a capture filter that is too narrow silently destroys evidence, while a display filter that is too narrow only hides it, and it can be edited and re-run. On Linux the capture filter is compiled to BPF and can be inspected: `tcpdump -d` dumps the compiled packet-matching code in human-readable form ✅ (tcpdump(1), tcpdump 4.99.7 man page dated 31 July 2026).
- **Capture file formats.** Two matter: the classic libpcap format (`pcap`, sometimes "libpcap") and **pcapng**, its extensible successor. Detail in §6.
- **The flow record as the metadata alternative.** Instead of copying packets, a device exports a *record* summarising each flow: five-tuple, byte and packet counts, timestamps, interfaces, ToS. The standards are IPFIX ✅ (RFC 7011 / STD 77, September 2013, obsoletes RFC 5101), Cisco NetFlow v9 ✅ (RFC 3954, October 2004, Informational — the RFC's own IESG note says it "is not a candidate for any level of Internet Standard"), and sFlow version 5 ✅ (sflow.org, July 2004). A flow record is not a packet and never will be; it is a summary produced by a sampler or an exporter that can itself lose data.
- **The tap.** A physical device inserted in the path that copies the signal (or, in an aggregation tap, the frames) to a monitoring port. It sees what it is inserted into; it changes the link it is inserted into. Placement is a facilities question, cross-referenced to [L2 Design for Financial Data Centres](l2_design_financial_data_centers_guide.md).
- **The mirror port.** A switch feature — Cisco calls it SPAN, others port mirroring or port monitoring — that copies selected traffic to a designated port. It is configured, not wired, and it inherits the switch's forwarding behaviour.
- **The broker.** A packet broker (also network packet broker, NPB) aggregates mirrored streams from many capture points, filters and de-duplicates them, and distributes the survivors to a smaller number of analysis tools.
- **The bypass framework.** A kernel-bypass or zero-copy capture framework that moves packets to a userspace process with fewer copies and fewer kernel locks than the default path, at the cost of owning the NIC. §3.
- **The timestamp.** The field that makes a capture evidence rather than anecdote — and the field most often assumed rather than verified. §5.

### 1.4 The Boundary — What This Guide Owns, and What It Does Not

This guide owns **packet capture itself**: the capture points, the mechanics, the loss and timestamp problems, the format and storage question, the volume arithmetic, the analysis tooling with dated statuses, the decryption problem, the security and privacy consequences, and the banking angle.

It deliberately does not re-derive four adjacent subjects:

- **Zero-trust architecture** belongs to [Zero Trust Network Architecture](zero_trust_network_architecture_guide.md). The tension is real and is stated there in its own terms; here it is stated once: **a packet capture is a deliberate visibility exception to a zero-trust posture, and the exception has to be governed** — authorised, scoped, time-boxed, and disposed of. §11 returns to the governance, not to the architecture.
- **The container capability model** belongs to [OpenShift SCC Comprehensive Guide](openshift_scc_comprehensive_guide.md), [OpenShift SCC Service Account Guide](openshift_scc_service_account_guide.md) and [Secure Red Hat OpenShift](secure_red_hat_openshift_guide.md). Capturing inside a container requires holding `NET_RAW` (the packet-socket capability, per packet(7) ✅) and in practice `NET_ADMIN`; a default SCC removes them. §2 uses that fact; it does not re-explain SCCs.
- **The protocols this guide diagnoses** belong to [FIX Protocol](../banking/fix_protocol_guide.md), [SWIFT Alliance Access](../banking/swift_alliance_access_guide.md), [SWIFTNet FileAct](../banking/swiftnet_fileact_guide.md), [Axway Transfer CFT](axway_transfer_cft_guide.md) and [IBM MQ Disaster Recovery](ibm_mq_disaster_recovery_guide.md). Those guides own their wire formats and message semantics. **They are this guide's use cases**: capture is how connectivity and message-flow faults in those systems are actually diagnosed. §11 links to them as applications.
- **Facilities and cabling** belong to [L2 Design for Financial Data Centres](l2_design_financial_data_centers_guide.md), [Data Center](data_center_guide.md) and [Singapore Data Centres](singapore_data_centres_guide.md) — including where a physical tap sits.

One correction for readers arriving from a repository search, and it matters because the filename is actively misleading: **`technology/professional_networking_guide.md` is a career-and-social-networking guide** — relationship-building, personal brand, a 30-day plan — and has nothing to do with computer networking. It is a false friend, not an adjacent guide. No packet-capture content in this repository predates this file; the handful of incidental `pcap` mentions elsewhere are mostly the OpenShift SCC guides noting that capturing inside a container needs capabilities the default SCC removes.

### 1.5 The Two Structural Limits

Everything that follows is a consequence of two facts.

1. **A capture point only sees traffic that traverses it.** Not a limitation of the tool; a limitation of geometry. A capture on a host cannot see traffic that never reaches that host; a mirror of one VLAN cannot see another.
2. **Observing is not free.** The copy costs CPU, memory bandwidth and buffer space — on the switch, if you mirror; on the kernel, if you capture on the endpoint; on the VM, if you mirror in a cloud VPC (Google Cloud states plainly that because mirroring "happens on the virtual machine (VM) instances, not on the network... Packet Mirroring consumes additional bandwidth on the VMs" ✅). The cost is not merely a bill. When the observer falls behind, the observer drops, and the drop is invisible in the output.

---

## 2. Where You Can Capture, and What Each Point Costs You

### 2.1 The Endpoint and Its Own NIC

The cheapest capture point. The traffic is already there; no wiring, no maintenance window, no vendor. On Linux the mechanism is a packet socket (`AF_PACKET`), which requires `CAP_NET_RAW` "in the user namespace that governs its network namespace" ✅ (packet(7)).

It sees: everything the host sends and receives on that interface, plus (in promiscuous mode) frames delivered to the NIC that are not addressed to the host, subject to what the upstream switch forwards. It does **not** see: traffic between two other hosts that the switch does not flood to this port, and — critically for telemetry debates — any traffic that is offloaded to hardware or delivered to a userspace stack that bypasses the kernel receive path. A capture on a host running a kernel-bypass data plane may see almost nothing of that data plane's traffic ⚠ (mechanism verified in §3; the observable consequence is stated as a caution, not a measured fact).

What it changes about the network it observes: nothing on the wire. What it changes about the host: CPU and memory, and it leaves a real file behind, which is a data-protection event in its own right (§10). Inside a container, add the capability question: `NET_RAW`/`NET_ADMIN` are exactly what a hardened SCC removes — see [OpenShift SCC Comprehensive Guide](openshift_scc_comprehensive_guide.md).

### 2.2 The Physical Tap

A tap is a purpose-built appliance inserted in the link, offering a monitoring output that copies traffic. Its defining property — and the reason security teams like it — is that it is not a configuration on a switch and cannot be silently re-scoped by a VLAN change. Its defining cost is that inserting it means taking the link down, which in a bank means a change request, a maintenance window, and a decision about a link you may not be allowed to interrupt.

Tap types differ in a way that matters for capture completeness: a tap that splits optical power or passes the signal through sees everything on the link; an *aggregation* tap that must buffer and forward can be oversubscribed, and oversubscription is a drop condition with the same invisibility problem as everything else in this guide. Placement, redundancy and fail-to-wire behaviour are facilities questions — cross-reference [L2 Design for Financial Data Centres](l2_design_financial_data_centers_guide.md) and [Data Center](data_center_guide.md).

### 2.3 The Switch Mirror / SPAN Port

The convenient capture point, and the one that most often invalidates a capture without anyone noticing.

The mechanism is documented by the vendors. Cisco's SPAN ("Switched Port Analyzer", also called port mirroring or port monitoring) copies traffic from "specified switch source ports or VLANs" to a "specified destination switch port (SPAN port)" ✅ (Cisco Network Management Configuration Guide, Cisco IOS XE 17.17.x for Catalyst 9200, chapter "Configuring SPAN and RSPAN", updated 31 March 2025). Cisco's own older documentation set includes a section explicitly titled "Performance Impact of SPAN on the Different Catalyst Platforms", and Cisco's community knowledge base records the operational consequence: capturing from many busy ports on a line card "can fill up the fabric connection" ✅-structural (Cisco support/community documentation, see §14 — the field-level statement about fabric saturation is documented at platform level, which is why the exact thresholds are not asserted here).

The honest formulation, and the one this guide stands behind:

- A SPAN destination port is a **port with a bandwidth**, and the mirrored traffic is what the switch chooses to send it. If the sources collectively exceed it, the switch has to drop, and depending on platform the drop may not be reported anywhere a packet analyst looks.
- A SPAN session **changes the switch**: the destination port is typically taken out of normal forwarding, source-port behaviour can be affected, and the mirrored traffic consumes internal bandwidth that the production traffic also needs.
- A SPAN session is **a configuration**, so it can be changed, partially applied, or removed by anyone with the privileges — and an incident responder arriving later cannot tell from the capture file that the mirror scope was not what the ticket said.

It sees: whatever the configured source ports/VLANs send through the switch, subject to the above. It does not see: traffic that never traverses the switch, and traffic the switch chose not to mirror when it ran short. It can drop. It changes the network it observes. Use it, but instrument it (§4).

### 2.4 The Packet Broker

A broker aggregates many mirrored inputs into a few filtered outputs. Its value is not bandwidth but **fan-out and re-use**: several tools (an IDS, a protocol analyzer capture rig, a lawful-intercept path, a performance monitor) can be fed from one set of taps, each with its own filter, without a second copy on the wire and without adding an aggregation device per tool. A broker loop that mis-filters is a single point at which all of them lose data simultaneously, which is a governance argument for monitoring the broker as part of the capture estate rather than as a network appliance (§4, §13).

### 2.5 The Virtual Switch and the Container Namespace

East-west traffic inside a virtualisation cluster or a Kubernetes cluster frequently never leaves the host, so nothing on the physical network can mirror it. Capture has to happen in the virtual switch, in the host kernel, or in the namespace. Two consequences:

1. The capability gate. On Linux, capturing inside a container's namespace needs `CAP_NET_RAW` in the governing user namespace ✅ (packet(7)); hardening frameworks remove it. See [OpenShift SCC Comprehensive Guide](openshift_scc_comprehensive_guide.md) and [Secure Red Hat OpenShift](secure_red_hat_openshift_guide.md) for what that means in practice, and [OpenShift SCC Service Account Guide](openshift_scc_service_account_guide.md) for the service-account route to granting it deliberately rather than accidentally.
2. The identity problem. A capture taken in a pod namespace shows addresses and ports that may be meaningless outside the cluster, which is a correlation problem for §5 and a storage problem for §6.

### 2.6 The Cloud Provider's Own Mirroring — Verified Per Provider

Cloud mirroring is a first-class product in each major provider, and each product has its own shape. The three below were verified at the providers' own documentation this pass.

**AWS — Traffic Mirroring.** "An Amazon VPC feature that you can use to copy network traffic from an elastic network interface of type `interface`", sending it to out-of-band security and monitoring appliances for content inspection, threat monitoring and troubleshooting. Key concepts are Source (the network interface to monitor), Filter, Target, and Session (source + filter + target). It "supports filters and packet truncation". It is available in all Regions, supported only for a documented list of instance families (virtualised, and Nitro v2 bare-metal only for those bare-metal types listed), and billed **hourly per active traffic mirror session** with data-transfer charges applying ✅ (AWS docs, "What is Traffic Mirroring?", retrieved 2026-09-22). The billing subtlety is documented and catches people: you are still charged after detaching the source, stopping it, or changing its instance type to an unsupported one, until all active sessions are deleted.

**Azure — Network Watcher packet capture.** "Azure Network Watcher packet capture allows you to create packet capture sessions to track traffic to and from a virtual machine (VM) or a scale set", triggered through the portal, PowerShell, CLI or REST API, or by a VM alert, saving either to the VM's local disk or to a storage blob. It requires the `AzureNetworkWatcherExtension` VM extension. Configuration is explicit: maximum bytes per packet (leave blank for all; 34 captures "only... the IPv4 header"), a time limit per session (default 18,000 seconds = 5 hours), filters by 5-tuple (protocol, local/remote IP, local/remote port). **Continuous packet capture is in PREVIEW**: it uses a ring-buffer mechanism with a rotating set of files, configurable file count (default 10, maximum 10,000) and file size (default 100 MB, maximum 4 GB), running for a user-specified duration up to a maximum of seven days, deleting the oldest file when the file-count limit is reached. Limits and constraints are documented: 10,000 parallel capture sessions per region per subscription; storage access uses SAS tokens and "key access must be enabled on the storage account", otherwise captures can only be saved to the VM's local disk ✅ (Microsoft Learn, "Packet capture overview", last updated 24 February 2026).

**Google Cloud — Packet Mirroring.** "Packet Mirroring clones the traffic of specified instances in your VPC network and forwards it for examination", capturing "all traffic and packet data, including payloads and headers", configurable for egress, ingress or both. Two facts deserve emphasis because they are the live trade-offs: mirroring "happens on the virtual machine (VM) instances, not on the network. Consequently, Packet Mirroring consumes additional bandwidth on the VMs"; and the collector destination is "an instance group that is behind an internal load balancer" whose forwarding rule must be configured for Packet Mirroring, with "any non-mirrored traffic that is sent to the load balancer... dropped". By default it collects all IPv4 traffic; filters can widen to IPv6 or narrow by protocol, CIDR, and direction. Multiple policies can apply, policy priority "is always 1000 and cannot be changed", and pricing is by data processed ✅ (Google Cloud VPC documentation, "Packet Mirroring", page last updated 2026-09-18).

The general lesson across all three: **cloud mirroring is a per-resource product whose filters, truncation options, session limits and billing model are documented, and whose completeness depends on how you set them.** The Azure default of a 5-hour session and the AWS per-session hourly charge are the two settings most likely to surprise a project plan.

### 2.7 The Trade-Off Table

| Capture point | Can see | Cannot see | Can drop under load? | What it changes about the network |
| --- | --- | --- | --- | --- |
| Endpoint NIC | That host's send/receive, plus frames the NIC accepts | Other hosts' traffic not forwarded to it; traffic bypassing the kernel path | Yes — kernel socket buffer and softnet backlog (Linux); the OS reports what it reports (§4) | Nothing on the wire; CPU/memory on the host; creates a data store |
| Physical tap | Everything on the tapped link | Anything on other links; encapsulated traffic it has no licence for | Aggregation taps can be oversubscribed | Requires an outage to insert; adds a powered device in the path |
| Switch mirror / SPAN | What the configured sources send to the destination port | What the switch did not forward; other VLANs | Yes — destination port and internal fabric are finite; documented platform-level performance impact | Destination port out of normal forwarding; internal bandwidth consumed |
| Packet broker | Aggregated view of many points; filtered fan-out to many tools | Anything upstream of its inputs | Yes; and a broker failure loses all downstream tools at once | Adds an appliance and a policy surface to govern |
| Virtual switch / namespace | East-west traffic that never reaches the physical network | Traffic outside the namespace; identities that do not map outside it | Yes — the vSwitch/host is a finite path | Container needs `NET_RAW`/`NET_ADMIN` (see SCC guides) |
| Cloud mirroring (AWS/Azure/GCP) | Per the documented scope of the resource (ENI, VM/NIC, instance) | Traffic not on the mirrored resource; and per documented filter/truncation settings | Provider-documented limits and, in GCP's case, VM bandwidth consumption | Consumes VM/host capability and bandwidth; billed per data or per session |

### 2.8 The Summary of the Section

Every point in that table is a trade, and no point is free of the observer's paradox: the more you mirror, the more load you place on the thing you are trying to observe accurately. §3 explains why the default software path cannot keep up beyond modest rates; §4 explains how you tell, after the fact, whether it did.

---

## 3. The Capture Mechanics and the Kernel Path

### 3.1 How a Packet Actually Reaches a Capturing Process (Linux)

The capture path is a copy, not a read. The sequence, as documented:

1. The NIC receives a frame and, through its driver and the NAPI poll path, hands it to the kernel receive stack.
2. The frame traverses the network stack. A packet socket bound to the interface receives a **copy** of matching frames — packet(7) is explicit that SOCK_RAW packets are "passed to and from the device driver without any changes in the packet data" — before those frames reach the protocol handlers implemented in the kernel.
3. The bytes land in a kernel buffer, and a userspace process (tcpdump, dumpcap, Zeek, Suricata) reads them out.
4. The analysis tool either prints them, writes them to a file, or feeds them to a protocol state machine.

The copy in step 2 and the transfer in step 3 are the costs. The capture library, historically libpcap, sets this up for every one of those tools. The kernel documentation of the `PACKET_MMAP` facility is unusually candid about the naive version's cost: without `PACKET_MMAP` "the capture process is very inefficient. It uses very limited buffers and requires one system call to capture each packet, it requires two if you want to get packet's timestamp (like libpcap always does)" ✅ (Linux kernel documentation, `networking/packet_mmap`). With `PACKET_MMAP`, the kernel hands userspace "a size configurable circular buffer mapped in user space", so that "reading packets just needs to wait for them, most of the time there is no need to issue a single system call", and packet copies are minimised ✅ (same source).

Programmatically, on Linux that facility is the packet receive ring: `PACKET_RX_RING` creates "a memory-mapped ring buffer for asynchronous packet reception"; the kernel fills slots and hands ownership to the application by changing `tp_status` from `TP_STATUS_KERNEL` to a value with the `TP_STATUS_USER` bit set, and the application hands slots back by setting `TP_STATUS_KERNEL` ✅ (packet(7)). The ring has variants; `TPACKET_V1` is the default for `PACKET_RX_RING`, `PACKET_VERSION` selects another, and `TPACKET_V3` support arrived in libpcap 1.5.0 ✅ (packet(7); kernel packet_mmap documentation).

The kernel documentation also names the two other things that must be right before any of this scales: interrupt load mitigation (or better, NAPI) on the NIC, and CPU IRQ pinning ✅ (packet_mmap). Those are the difference between a capture rig that keeps up and one that quietly does not.

### 3.2 The Facilities Involved, and Their Behaviour Under Load

| Facility | What it provides | Load behaviour |
| --- | --- | --- |
| `AF_PACKET` socket (Linux) | The raw capture interface; `SOCK_RAW` (with link header) or `SOCK_DGRAM` (cooked); requires `CAP_NET_RAW` | The per-packet syscall path (`recvmsg` per packet, historically two syscalls when a timestamp is wanted) is the bottleneck; the ring removes most syscalls |
| `PACKET_RX_RING` + `PACKET_MMAP` | Shared kernel/userspace circular buffer; block/frame geometry set by `tpacket_req` | Ring overrun is the drop condition; the socket's own counters report it (§4) |
| `PACKET_FANOUT` (since Linux 3.1) | Spreads packets across sockets for multi-threaded capture — hash (per-flow ordering), load-balance, CPU, rollover, random, queue-mapping modes | Designed for exactly this problem; mode choice changes whether flow ordering is preserved |
| `PACKET_AUXDATA` (since Linux 2.6.21) | Per-packet metadata (length, captured length, VLAN tag) alongside the frame | Enables VLAN-aware capture without re-parsing |
| Softnet/NAPI path and backlog | Kernel's mechanism for absorbing bursts between interrupts | Backlog exhaustion is a drop condition that happens *before* your capture socket sees the packet (§4) |
| NIC ring (RX descriptors) | Device-level buffering before the kernel sees the frame | Ring exhaustion is a drop condition counted by the device, often as `rx_missed_errors` (§4) |

`PACKET_FANOUT` deserves one line more: its modes exist because the naive answer to "capture is too slow" is "run more capture threads", and the naive answer to that is reordering. The default `PACKET_FANOUT_HASH` "sends packets from the same flow to the same socket to maintain per-flow ordering", while `PACKET_FANOUT_LB` "implements a round-robin algorithm" — the former preserves flow coherence, the latter scales more evenly ✅ (packet(7)).

### 3.3 Zero-Copy and Kernel-Bypass: Current Names and Statuses

The claim "kernel bypass solves capture loss" is true in a specific, narrow sense and false as a general statement. Here is the verified current state of each named approach. **Every status below was checked at the project's or the kernel's own documentation this pass; versions and dates are in §14.**

**AF_XDP — in the mainline kernel, not a patch.** "AF_XDP is an address family that is optimized for high performance packet processing" ✅ (Linux kernel documentation, `networking/af_xdp`). Mechanism: an XDP program attached to a device can redirect frames to a userspace memory buffer using `bpf_redirect_map()`; an AF_XDP socket (XSK) has RX and TX rings (`XDP_RX_RING`, `XDP_TX_RING`) pointing into a registered UMEM region divided into equal-sized chunks, with FILL and COMPLETION rings transferring ownership between kernel and userspace. A UMEM "can be shared between processes" via `XDP_SHARED_UMEM`. Two operating modes are documented: `XDP_SKB` (generic, "a fallback mode that works for any network device", which "uses SKBs together with the generic XDP support and copies out the data to user space") and `XDP_DRV` (used when the driver supports XDP, "but there is still a copy of the data into user space" in copy mode) ✅. That last clause is the honest limit: **AF_XDP removes the skb and the copy-to-ring path of `AF_PACKET`, but zero-copy is a mode and a driver-capability question, not a property of the interface.** The documentation also warns that XDP-to-XSK routing is by queue: with the default one-queue-per-core layout, "if you bind to queue 0, you are NOT going to get any traffic that is distributed to queues 1 through 7", and it gives `ethtool -L`/`ethtool -N` as the remedy ✅ — a queue-mapping mistake that looks exactly like packet loss.

**DPDK — actively released, no GitHub releases.** The Data Plane Development Kit publishes on its own infrastructure, not as GitHub releases: latest **26.07.0 (22 July 2026)** and latest Long Term Stable **25.11.3 (20 August 2026)**, with additional LTS lines maintained (24.11.7, 11 August 2026) ✅ (core.dpdk.org/download, retrieved 2026-09-22). DPDK is a full userspace packet-processing framework that takes over a NIC through a poll-mode driver — very fast, and a commitment: the port you give it is no longer the kernel's port. Note the coupling that surprises people: **libpcap 1.11.0 (a "testing" release, 13 September 2026) removes any remaining support for DPDK** among other capture modules ✅ (tcpdump.org release announcement), so "capture with libpcap on DPDK" is not a path the library still offers at that version.

**PF_RING — actively maintained, current release 9.4.0.** ntop's "Linux kernel module and user-space framework" for high-rate packet processing, which its own README describes in terms of packets per second ("It can range from 80k pkt/sec on a 1,2GHz ARM to more than 20M pkt/sec per core on a low-end 2,5GHz Xeon" — a vendor performance claim, quoted as a vendor claim) ✅ (github.com/ntop/PF_RING README). Activity is verifiable and recent: tag **9.4.0 on 29 August 2026**, and a repository commit on **21 September 2026** ✅ (GitHub tags and repository page, retrieved 2026-09-22). It is a third-party kernel module, not mainline — which is the whole trade: best-in-class rates, and a module you must maintain against kernel changes.

**netmap — maintained, low-velocity, no release tags.** A framework for fast packet I/O from userspace, with the VALE in-kernel L2 switch, from the University of Pisa; it has shipped in FreeBSD since 10.x and supports Linux via out-of-tree drivers ✅ (github.com/luigirizzo/netmap README). Status check: repository commits on 8 August 2026, 10 July 2026, 26 May 2026 and 18 April 2026, with recent work being kernel-compatibility fixes, FreeBSD metadata clean-up and CodeQL workflow addition; the repository publishes no GitHub releases ✅ (GitHub commits view, retrieved 2026-09-22). **Characterise it accurately: alive, maintained by a small group, and paced by kernel compatibility rather than feature development.** It is not the default recommendation in 2026 for a bank that needs a vendor to call.

**The eBPF route.** The mainline-kernel capture route that uses eBPF is not a separate product: it is XDP plus AF_XDP, i.e. an XDP program that selects frames (`bpf_redirect_map()` into an `XSKMAP`, with a validated XSK bound to the same device and queue, or the packet is dropped) and a userspace AF_XDP socket that consumes them ✅ (kernel `networking/af_xdp`). Its attraction is that filtering happens early, in the driver/BPF path, before an skb is built. Its unresolved cost is the same as everything else in this section: someone has to write and maintain the filter, and an incorrect filter is a data-loss event that no counter on the capture side will report.

### 3.4 The Practical Consequence

**The default path is fine at modest rates and quietly inadequate at high ones, and the transition point is neither documented nor uniform.** It depends on packet size and rate, on the NIC and its driver, on whether NAPI, interrupt mitigation and IRQ affinity are configured, on the ring size chosen, on the number of capture sockets and their fanout mode, on how much work the analysis process does per packet, and on whether the write path can keep up. The kernel documentation says as much in one sentence, and it is the sentence to remember: performance work is "not everything"; "at least, if you are capturing at high speeds (this is relative to the cpu speed), you should check if the device driver of your network interface card supports some sort of interrupt load mitigation or (even better) if it supports NAPI, also make sure it is enabled" ✅ (packet_mmap). No source verified this pass states a universal pps threshold at which the default path fails, and this guide does not invent one. What it does instead is §4: instrument the capture so that the instance in front of you tells you whether it is failing.

---

## 4. The Loss Problem

This is the central technical finding of the guide.

**A capture that drops packets looks complete.** The file opens, the packets dissect, the timestamps ascend, the stream reassembles for long stretches. Nothing in the artefact says "I am a sample". The analyst quiets a symptom, closes a ticket, and files a conclusion built on an incomplete record — which is strictly worse than having no capture, because the absence of a capture at least forces honesty about evidence.

### 4.1 Why a Capture Drops

Five distinct drop locations exist between the wire and your file. They are frequently conflated, and they have different counters.

1. **At the device.** The NIC's receive ring runs out of descriptors because the host is not draining it fast enough. The kernel's statistics documentation describes the corresponding statistic, `rx_missed_errors`, as counting packets dropped "due to lack of buffer space", adding that this "usually indicates that the host interface is slower than the network interface, or host is not keeping up with the receive packet rate", and that it "corresponds to hardware events and is not used on software devices" ✅ (Linux kernel documentation, `networking/statistics`).
2. **In the kernel receive path.** The softnet backlog or the per-CPU processing budget is exhausted before any capture socket gets a look. This is the least visible class, because the traffic never enters the capture mechanism at all.
3. **In the capture socket's ring.** The application is not draining the `PACKET_RX_RING` fast enough and the kernel overwrites or discards slots.
4. **In the capture library's user-space buffer**, where the process asks the kernel for packets in batches and the batch is lost on a signal or an exit path.
5. **On the write path.** Disk latency, a full filesystem, or an analysis process that cannot keep up with the rate at which it is being handed packets. This one produces a capture that is *late* rather than absent, and it is what a "capture looks fine but the timing is nonsense" incident usually turns out to be.

The switch, tap and broker add a sixth location *upstream of all of these*: the mirror itself may never have sent the packet (§2).

### 4.2 Which Counters Report It, and Where a Practitioner Reads Them

Every row below was verified at the named source this pass. Where a name could not be verified at source, it is in §15 rather than here.

**The capture tool's own report — check this first.**

- **libpcap's statistics API.** `pcap_stats()` fills a `struct pcap_stat` with three fields whose definitions matter: `ps_recv` — "number of packets received"; `ps_drop` — "number of packets dropped because there was no room in the operating system's buffer when they arrived, because packets weren't being read fast enough"; `ps_ifdrop` — "number of packets dropped by the network interface or its driver" ✅ (tcpdump.org man page `pcap_stats(3PCAP)`, documenting libpcap 1.11.1-PRE-GIT, page updated 5 March 2022). The same page documents the limits you must respect: the API "is supported only on live captures, not on `savefiles`" (so **there is no drop counter in a capture file**); the semantics differ across platforms; `ps_drop` "is not available on all platforms; it is zero on platforms where it's not available"; and `ps_ifdrop` being zero "might mean that no packets were dropped by the interface, or it might mean that the statistic is unavailable", so it "should not be treated as an indication that the interface did not drop any packets" ✅. Anyone who has ever cited a zero drop count as proof of completeness has been defeated by that sentence.
- **tcpdump at exit.** tcpdump "will report counts of" packets "captured" (received and processed), packets "received by filter" (whose meaning "depends on the OS" and the OS's configuration), and packets "dropped by kernel" — defined as packets "dropped, due to a lack of buffer space, by the packet capture mechanism in the OS on which tcpdump is running, **if the OS reports that information to applications; if not, it will be reported as 0**" ✅ (tcpdump(1), tcpdump 4.99.7 man page dated 31 July 2026). The same page documents that `-B buffer_size` sets the OS capture buffer in KiB and that the counts can be requested mid-capture with SIGINFO or SIGUSR1, and that SIGUSR2 with `-w` will "forcibly flush the packet buffer into the output file" ✅.
- **dumpcap, Wireshark's capture engine.** `-S` "Print statistics for each interface once every second"; `-B` sets the capture buffer size in MiB with a default of 2 MiB, and the man page says plainly: "If you encounter packet drops while capturing, try to increase this size", while warning that the system or interface "might silently limit the capture buffer size to a lower value or raise it to a higher value" ✅ (dumpcap(1), Wireshark 4.6.8). That last warning is the loss problem wearing a friendly face.

**The kernel's account of what it did with the traffic.**

- **Interface statistics, three ways.** The kernel documents three families: standard interface statistics based on `struct rtnl_link_stats64`; protocol-specific statistics; and driver-defined statistics available via ethtool ✅ (Linux kernel documentation, `networking/statistics`). The standard statistics are reachable via `ip -s -s link show <dev>` (whose output labels the receive columns "bytes, packets, errors, dropped, overrun, mcast" and the detailed receive errors "length, crc, frame, fifo, missed"), via the historical `/proc/net/dev`, via sysfs at `/sys/class/net/<dev>/statistics/`, and via netlink — with rtnetlink documented as "the preferred method" (RTM_GETLINK, and RTM_GETSTATS "when IFLA_STATS_LINK_64 bit is set") ✅.
- **The field definitions that matter.** From the same kernel page: `rx_dropped` is the "number of packets received but not processed, e.g. due to lack of resources or unsupported protocol", and for hardware interfaces "this counter may include packets discarded due to L2 address filtering but should not include packets dropped by the device due to buffer exhaustion which are counted separately in **rx_missed_errors** (since procfs folds those two counters together)"; `rx_fifo_errors` is described as counting packets dropped by the device due to lack of buffer space "where this statistic corresponds to hardware events and is not used on software devices"; `rx_errors` must aggregate the detailed error counters ✅. That "procfs folds those two counters together" note is the practical reason to read the detailed view rather than the one-line one.
- **Driver and hardware counters.** `ethtool -S <iface>` dumps driver-defined statistics; standard groups (Ethernet PHY/MAC/control and RMON) are exposed via the `--groups` parameter, e.g. `ethtool -S eth0 --groups eth-phy eth-mac eth-ctrl rmon`; and the kernel warns that on devices containing unmanaged switches "the events counted may not pertain exclusively to the packets destined to the local host interface" ✅ (kernel `networking/statistics`). The named counters here are driver- and hardware-specific by construction — there is no universal "nic_dropped" — which is exactly why a capture rig needs its counters recorded alongside the capture.
- **The per-CPU softnet counters.** `/proc/net/softnet_stat` is the file practitioners watch for drops that happen before the capture socket: one row per CPU, and by the widely used reading, a processed-packet column and a drop column, with further columns for times the softirq ran out of budget and times `net_rx_action` was rescheduled ✅-secondary (Netdata's collector documentation for `/proc/net/softnet_stat` lists the dimensions `processed`, `dropped`, `squeezed`, `received_rps`, `flow_limit_count`, and its shipped alerts reference `net.core.netdev_max_backlog` and `net.core.netdev_budget`; retrieved 2026-09-22). **Flagged deliberately: this file is not documented in the kernel's own Documentation tree as far as this pass could establish (§15).** Read it, but treat the column meanings as secondary-source knowledge and validate them against your kernel version.
- **The capture socket's own counters.** `PACKET_STATISTICS` retrieves `struct tpacket_stats { unsigned int tp_packets; unsigned int tp_drops; }` — "Total packet count" and "Dropped packet count" — and the man page adds the detail that most teams get wrong when they poll it: "Receiving statistics resets the internal counters", and "the statistics structure differs when using a ring of variant TPACKET_V3" ✅ (packet(7)). A monitoring loop that samples `PACKET_STATISTICS` and does not account for the reset-on-read will mis-total everything.

**The analysis tool's own loss accounting.**

- **Zeek infers loss from the traffic itself.** `capture_loss.log` "reports analysis of missing traffic. Zeek bases its conclusions on analysis of TCP sequence numbers. When it detects a 'gap,' it assumes that the missing traffic corresponds to traffic loss" ✅ (Zeek documentation, `capture_loss.log` and `reporter.log`; the log carries per-interval fields `ts`, `ts_delta`, `peer`, `gaps`, `acks` and `percent_lost`, and the docs give a worked example where `0.412308930008045` means "0.4123% capture loss, not 41.23% capture loss"). This is the single most useful detection mechanism in the guide, and it is worth being precise about why: it does not ask the capture rig whether it dropped anything. It reconstructs what the *peers* saw and reports the difference. A capture that is missing packets but whose counters read zero is caught here.
- **The security engines' statistics streams.** Suricata logs engine statistics both to a text `stats.log` and as an EVE record type `stats`, both "enabled by default", with a configurable interval (the documented example uses 8 seconds) and counters such as `detect.alert` and `detect.alert_queue_overflow` ✅ (Suricata documentation, `suricata.yaml` reference, from the project's own documentation source). ⚠ The specific capture-drop counter names in that stream were **not** verified at source this pass (§15); the mechanism is verified, the field names are not asserted here.

### 4.3 Loss at the Capture Point vs Loss on the Wire

These are different problems with the same symptom, and the counters separate them.

- If the **wire** is losing traffic, the endpoints know: retransmissions, TCP sequence gaps visible in *both* directions, ICMP or counter evidence on the path, and — the discriminators — the loss appears in directions and at points that do not correspond to the capture rig, and it appears in captures taken at different points simultaneously.
- If the **capture point** is losing traffic, the loss is a property of the observation, not the network: it correlates with capture rate, buffer settings and CPU load; it disappears when the capture is moved, rate-limited, or given a bigger ring; and the interface/device counters or the per-CPU softnet counters move when the capture traffic is what saturates the host.
- If the **capture is fine but the observation point is wrong**, nothing is lost and nothing is reported, and you conclude the network is healthy. This is §2's structural limit, and it is not detectable from counters at all — only from a topology diagram and a sceptical reviewer.

A workable discriminator, stated as a practice rather than a measurement: run two captures at two different points, with independent instrumentation, over the same window, and reconcile. Agreement is weak evidence of completeness; disagreement is proof that at least one is wrong, and usually identifies which.

### 4.4 What the Bypass Frameworks Change, and What They Do Not

They change the *mechanism* of the copy: fewer copies, fewer syscalls, fewer context switches, a path that can be steered before the kernel builds an skb (AF_XDP with an XDP program, DPDK's poll-mode driver, PF_RING's ring, netmap's rings). They raise the rate at which a well-configured rig can keep up.

They do **not** change five things, and every one of them has caused a production surprise:

1. **They do not create buffer where there is none.** The ring is still finite; the write path is still finite; the analysis process is still the slowest stage in most rigs.
2. **They do not report loss better.** Some of them report it worse, because the counters a packet-socket user has come to trust do not exist on a bypass path.
3. **They do not see traffic they are not given.** DPDK owns a port; AF_XDP binds a queue (and the kernel documentation's own example shows how easily you bind the wrong one); PF_RING and netmap need their module and their drivers ✅ (sources in §3.3).
4. **They do not compose freely with the rest of the system.** libpcap 1.11.0's removal of remaining DPDK support ✅ is the concrete example of a bypass path ceasing to be a library option.
5. **They do not survive an OS or driver upgrade by themselves.** Out-of-tree modules are the maintenance liability; netmap's recent commit history is precisely kernel-compatibility and metadata work ✅.

### 4.5 The Detection Discipline — Establishing That a Capture Was Complete

You cannot verify a capture after the fact from the capture file alone. pcapng does define an Interface Statistics Block that "contains the capture statistics for a given interface", whose purpose the specification gives as storing "some statistical data (e.g. packet dropped, etc) which can be useful to understand the conditions in which the capture has been made", normally placed at the end of the file but with no assumption about position, and possibly appearing multiple times ✅ (pcapng draft-ietf-opsawg-pcapng-06, 19 September 2026, §4.6). That is the mechanism by which a capture file *can* carry its own completeness evidence — and it only carries it if the capture rig wrote it. If your capture pipeline does not write an ISB, or does not log `pcap_stats()` alongside the file, then the file is unverifiable by construction.

So the discipline is a pipeline property, established before the incident:

1. **Record the environment with the capture.** Tool and version, interface, ring/buffer settings, filter expression as executed, and the counter readings at start and end.
2. **Watch the counters during capture, not after.** tcpdump's counters on SIGINFO/SIGUSR1, `dumpcap -S` (statistics per interface every second), `PACKET_STATISTICS`, `/proc/net/softnet_stat` deltas per CPU, and the interface/device counters ✅.
3. **Infer loss from the traffic.** Run Zeek's `capture_loss.log` over the window, or an equivalent sequence-gap analysis, and read `percent_lost` ✅.
4. **Sanity-check rate against expectation.** If the observed packet rate is materially below what the link and the business activity imply, the capture is the first suspect, not the network.
5. **Reconcile two independent points** for any capture that will be used as evidence.
6. **Refuse to testify to completeness you did not measure.** In a dispute or a regulatory question, the honest statement "we hold a partial recording of the window, with the following measured completeness" is defensible; the confident statement "the packets show no such message arrived" is not, unless the counters and the sequence-gap analysis back it.

### 4.6 The Operational Rule

**Instrument the capture, not just the capture's output — an unmonitored capture is an unverified one.**

A capture rig without loss telemetry is a measurement instrument with no calibration, deployed by people who will read its output as fact. Every subsequent section of this guide assumes the rig in question has been instrumented; where that assumption fails, the analysis in §8, the arithmetic in §7 and the evidence claims in §11 are all built on sand.

---

## 5. The Timestamp Problem

Loss makes a capture incomplete. Timestamps make it *wrong* — and wrong in a way that survives every sanity check, because the file still looks perfect.

### 5.1 What the Default Timestamp Actually Is

By default, on Linux, the timestamp attached to a captured packet is taken when the kernel copies the packet into the capture ring — not when the frame arrived at the NIC. The packet-socket man page states it directly: the packet receive ring "always stores a timestamp in the metadata header. **By default, this is a software generated timestamp generated when the packet is copied into the ring**", and `PACKET_TIMESTAMP` "selects the type of timestamp", supporting the hardware formats described in the kernel's timestamping documentation ✅ (packet(7)). The kernel's own packet_mmap documentation adds the precision nuance: if `PACKET_TIMESTAMP` is not set for the RX ring, "a software fallback was invoked *within* PF_PACKET's processing code (less precise)" ✅.

What that software timestamp therefore *is*: a measurement of when the kernel got around to copying the frame, in the kernel's notion of system time. Everything between the wire and that moment — PHY, MAC, descriptor ring, interrupt coalescing, NAPI polling, softirq scheduling, per-CPU backlog — is outside the measurement.

The tooling is explicit that it is a consumer, not a source. Wireshark's documentation: "While capturing, Wireshark gets the time stamps from the libpcap (Npcap) library, which in turn gets them from the operating system kernel", and on accuracy, "Wireshark doesn't create any time stamps itself but simply gets them from 'somewhere else' and displays them. So accuracy will depend on the capture system (operating system, performance, etc.) that you use", with a concrete warning: "USB connected network adapters often provide a very bad time stamp accuracy. The incoming packets have to take 'a long and winding road' to travel through the USB cable until they actually reach the kernel... this time stamping mechanism becomes very inaccurate. Don't use USB connected NICs when you need precise time stamp accuracy" ✅ (Wireshark User's Guide §7.6, "Time Stamps").

### 5.2 Precision and Accuracy Are Different Properties

The distinction is not pedantry; each fails independently and each is fixed differently.

- **Resolution / precision** is how finely the timestamp is represented and what the capture file can *hold*. Wireshark's guide documents that the common libpcap format "supports two possible fixed resolutions, microsecond or nanosecond, indicated by a magic number at the start of the file", whereas pcapng "supports a wide range of time stamp resolutions, which can be different for each interface in the file, as well as records without time stamps", and warns that writing into a format that cannot store the actual precision "will lead to loss of information" ✅.
- **Accuracy** is whether the number corresponds to the moment the frame was on the wire. It is bounded below by the capture mechanism (software timestamp at copy time, per §5.1) and by the synchronisation of the clock that stamps it. A capture can have nanosecond resolution and millisecond accuracy, and it will look more authoritative than a microsecond-resolution capture that is far better aligned to real time.

**The practical consequence: a capture file's timestamp field tells you nothing about its accuracy, and the resolution may even be a property of the file format rather than the measurement.**

### 5.3 Hardware Timestamping: What It Requires End to End

Hardware timestamping moves the measurement to the NIC — the point where the frame meets the medium — and it is the only way to get a capture timestamp that is meaningfully a *wire* timestamp. It is not a checkbox. Every link in the chain has to be present.

1. **A NIC (or PHY) that can timestamp.** Mainline Linux has a "PTP hardware clock infrastructure": the kernel documents that this "patch set introduces support for IEEE 1588 PTP clocks in Linux", that "together with the SO_TIMESTAMPING socket options, this presents a standardized method for developing PTP user space programs, synchronizing Linux with external clocks, and using the ancillary features of PTP hardware clocks", and that supported hardware is a specific list (e.g. NVIDIA Mellanox ConnectX with free-running counters, "ConnectX-6 Dx and later devices have a UTC format counter"; Renesas/IDT ClockMatrix with up to four PHC channels) ✅ (kernel documentation, `driver-api/ptp`). **Not every NIC timestamps; not every NIC that timestamps does so on receive.**
2. **A clock device per NIC.** "The class driver also creates a character device for each registered clock. User space can use an open file descriptor from the character device as a POSIX clock id and may call clock_gettime, clock_settime, and clock_adjtime" ✅ (same page). ⚠ The conventional device-node naming (`/dev/ptpN`) was **not verified at source this pass** (§15); the kernel page verifies the character device, not its name.
3. **Driver-level enablement of RX/TX timestamping, by an administrator.** The kernel documentation is unambiguous that "hardware time stamping must also be initialized for each device driver that is expected to do hardware time stamping", configured through the tsconfig netlink command `ETHTOOL_MSG_TSCONFIG_SET` (readable via `ETHTOOL_MSG_TSCONFIG_GET`) or, as the legacy path, the `SIOCSHWTSTAMP` ioctl carrying a `struct hwtstamp_config` with `tx_type` and `rx_filter` ✅. It further states: "Only a process with admin rights may change the configuration" and "if the requested packets cannot be time stamped, then nothing should be changed and ERANGE shall be returned" ✅. A silently ERANGE'd request is a silently software-timestamped capture.
4. **A request from the capturing application for hardware timestamps**, expressed as SO_TIMESTAMPING flags. The socket-option documentation lists `SOF_TIMESTAMPING_RX_HARDWARE` ("Request rx timestamps generated by the network adapter"), `SOF_TIMESTAMPING_TX_HARDWARE`, `SOF_TIMESTAMPING_TX_SOFTWARE`, `SOF_TIMESTAMPING_RX_SOFTWARE` ("generated just after a device driver hands a packet to the kernel receive stack"), `SOF_TIMESTAMPING_TX_SCHED` and `SOF_TIMESTAMPING_TX_ACK` ✅ (kernel documentation, `networking/timestamping`). For packet sockets, the equivalent is setting `PACKET_TIMESTAMP` to a bit field such as `SOF_TIMESTAMPING_RAW_HARDWARE`, with the ring's status bits telling you afterwards what you actually got: the `tpacket` header's `tp_status` is OR'd with `TP_STATUS_TS_RAW_HARDWARE` or `TP_STATUS_TS_SOFTWARE`, "that are equivalent to its SOF_TIMESTAMPING_* counterparts" ✅ (kernel packet_mmap documentation).
5. **Delivery to the application.** For receive timestamps, `struct scm_timestamping` (or `scm_timestamping64`) carries up to three timestamps in a `SCM_TIMESTAMPING` control message, and the kernel says which is which: "Most timestamps are passed in ts[0]. Hardware timestamps are passed in ts[2]." And then the sentence that saves a lot of debugging: "ts[1] used to hold hardware timestamps converted to system time. Instead, expose the hardware clock device on the NIC directly as a HW PTP clock source, to allow time conversion in userspace and optionally synchronize system time with a userspace PTP stack such as linuxptp" ✅ (kernel `networking/timestamping`, §2.1). Note also the warning that a **false** software timestamp can be generated and passed in ts[0] "when a real software timestamp is missing" if SO_TIMESTAMP/SO_TIMESTAMPNS is enabled alongside SO_TIMESTAMPING — "This happens also on hardware transmit timestamps" ✅.
6. **A synchronised clock at both ends.** A hardware timestamp is only an absolute time if the NIC's clock is disciplined. That is the PTP stack's job (§5.4), and if it is not running or not locked, a hardware-timestamped capture is *precisely* stamped and *wrongly* positioned in absolute time — arguably worse than a loose software timestamp, because it invites trust.

And the tooling exposes the choice: tcpdump documents `-j tstamp_type` and `--time-stamp-precision=tstamp_precision` (with `--micro`/`--nano`) ✅; dumpcap documents `--list-time-stamp-types` and `--time-stamp-type <type>` ✅. If the tool lists no time stamp types for the interface, there is nothing to choose from.

### 5.4 The Network Timing Standards, Related But Not Interchangeable

- **PTP, IEEE 1588.** The kernel's PTP hardware clock infrastructure exists to support it ("support for IEEE 1588 PTP clocks in Linux"; the Renesas ClockMatrix driver notes an integrated low-pass filter compliant with ITU-T G.8273.2, the telecom PTP profile) ✅ (kernel `driver-api/ptp`). PTP is the standard that pairs with hardware timestamping, because it disciplines the NIC's clock rather than only the host's, and because it is designed for the sub-microsecond territory in which hardware timestamps are meaningful.
- **NTP.** A separate protocol lineage and a separate ecosystem; it disciplines the host clock over the network. **⚠ The relationship between the two — including the specific NTP specification and how a PTP-disciplined NIC clock interacts with host NTP — was not verified at source this pass (§15).** The one relation that *is* sourced is narrow and useful: the kernel suggests using a userspace PTP stack such as **linuxptp** to "synchronize system time with" the NIC's PTP clock, i.e. the NIC clock can be the reference that disciplines host time, which is the opposite of the common assumption that host NTP is always the reference ✅.
- **The consequence for a capture estate.** If capture timestamps come from the NIC clock while application logs come from host time disciplined by NTP, then two clocks are in play and the offset between them is a systematic error in every cross-correlation. Nobody documents that offset when they hand over a capture file. It has to be measured and recorded.

### 5.5 The Correlation Trap — When the Capture Is the One That Is Wrong

The dangerous case, and the one this section exists for, is not "the timestamps were obviously broken". It is this: **a capture and an application log disagree about the order or the timing of events, and the analyst trusts the capture, and the capture is the one that is wrong.**

Three documented mechanisms produce exactly that:

1. **The software timestamp measures a later moment than arrival, by an unmeasured amount.** If the host was under load, the delay between wire and copy varies packet by packet — so the capture's *inter-packet spacing* is distorted, not just offset. A capture that shows a 40 ms gap between a request and a response may be showing the kernel's scheduling, not the network's latency. ✅ mechanism sourced (packet(7), packet_mmap).
2. **Resolution loss in the file format.** Wireshark's documentation gives the case: load a nanosecond-resolution capture and store it in a format with microsecond resolution and the precision must be reduced ✅. A downstream consequence worth stating plainly: microsecond-resolution files cannot support microsecond-and-below latency claims, however good the underlying measurement was.
3. **A silent fallback to a software timestamp.** If the hardware timestamp request was not honoured — no admin-enabled driver config, no NIC capability, or the adapter is USB, or a filter fallback on the receive path without `SOF_TIMESTAMPING_OPT_RX_FILTER` ✅ (kernel `networking/timestamping`) — the capture may still carry timestamps, still look plausible, and simply be software-stamped. Nothing in the file says which it was unless the pipeline recorded the ring's `tp_status` bits or the tool's chosen timestamp type.

The mitigations are cheap and almost never done:

- Record, per capture, **which timestamp source was used** (tool flag, requested and granted `hwtstamp_config`, or `dumpcap --time-stamp-type`), and the clock-synchronisation state at the time.
- Prefer **on-wire reasoning that is timestamp-independent** when the question is protocol correctness rather than timing: sequence numbers, TCP flags, retransmission counts, field values. These survive timestamp error.
- When the question *is* timing, **measure the measurement**: place two capture points with known synchronisation and measure the fixed offset, the way you would calibrate an instrument.
- Treat any latency claim below the file format's stated resolution as unsupported.

### 5.6 The Rules for This Section

1. State the timestamp source in every capture handover; "timestamps are present" is not a statement about accuracy.
2. Distinguish resolution from accuracy in every requirement, and refuse requirements that name only the first.
3. Hardware timestamping is an end-to-end configuration — NIC, driver, admin enablement, application request, delivery, and a disciplined clock — and any missing link degrades it silently.
4. Record the offset between the capture clock and the application-log clock, or accept that cross-correlation has an unquantified error term.

---

## 6. The File Format, the Storage and the Indexing Question

### 6.1 The Two Formats

Two capture file formats matter in 2026, and the difference is not cosmetic.

- **The classic libpcap format** ("pcap", "libpcap"). Simple: one file header, then a flat sequence of packet records, all assuming one interface and one link type and one timestamp resolution. It is the interchange format every tool can read.
- **pcapng** ("PCAP Next Generation", now documented as "PCAP Now Generic"). Block-structured and extensible, supporting multiple interfaces per file, per-interface timestamp resolution, per-file statistics, comments, name-resolution records, and vendor extensions. Wireshark's capture engine writes it by default: dumpcap's man page states "Dumpcap's default capture file format is pcapng format. The -F option can be specified to write the output file in the pcap format instead", and the pcap format option is overridden where pcapng is required (for example when capturing from multiple interfaces) ✅ (dumpcap(1), Wireshark 4.6.8).

### 6.2 The Classic Format, at Field Level

Per the IETF's OPSAWG draft specifying the format (draft-ietf-opsawg-pcap-04, published 4 August 2024) ✅:

- "A capture file begins with a File Header, followed by zero or more Packet Records, one per packet."
- The File Header is **24 octets**: Magic Number (32 bits), Major Version (16), Minor Version (16), Reserved1 (32), Reserved2 (32), SnapLen (32), and "LinkType and additional information" (32).
- The Magic Number "is either the hexadecimal number 0xA1B2C3D4 or the hexadecimal number 0xA1B23C4D", and the draft states the operational meaning: "If the value is 0xA1B2C3D4, timestamps in Packet Records are in seconds and microseconds; if it is 0xA1B23C4D, timestamps in Packet Records are in seconds and nanoseconds"; the value also distinguishes files written on little-endian from big-endian machines.
- "All fields in the File Header and in the headers of Packet Records will always be written according to the characteristics (little endian / big endian) of the machine that is writing the file."

The fields a practitioner actually uses: **SnapLen** (how much of each packet the file claims to store — the truncation contract) and **LinkType** (how to interpret the bytes; the link-type registry is a separate document, `draft-ietf-opsawg-pcaplinktype`).

⚠ Note on the specification's status: the revision verified this pass (‑04) carries an expiry date of 5 February 2025, and no later revision or RFC status could be confirmed from this environment (§15). The field-level facts above are correct for the format as deployed; the document-number citation may be superseded.

### 6.3 pcapng, at Block Level

Per draft-ietf-opsawg-pcapng-06, published 19 September 2026, Intended Status Informational, expiring 23 March 2027 ✅ — the most current specification-level source in this guide.

- **Every block has the same envelope**: Block Type (32 bits), Block Total Length (32 bits), Block Body (variable, padded to 32 bits), and a repeated Block Total Length at the end. The duplicated length exists deliberately: "This field is duplicated to permit backward file navigation", and the spec requires that future versions preserve it so tools can skip blocks in either direction.
- **The Section Header Block (SHB) is the only mandatory block**, has Block Type `0x0A0D0D0A`, and identifies the beginning of a section; a file may contain several SHBs (typically the result of concatenation), each defining endianness for the data that follows. A tool that cannot read a section "MUST skip everything until the next Section Header Block".
- **Interface Description Block (IDB)** defines "the most important characteristics of the interface(s) used for capturing traffic". Each IDB is assigned an incrementing 32-bit **Interface ID** starting from 0, "unique within each Section", and referenced by other blocks. The IDB carries the interface's **SnapLen** and, per the ISB text's cross-reference, the `if_tsresol` and `if_tsoffset` values that give per-interface timestamp resolution and offset ✅. This is the pcapng feature with the most practical consequence: a file can legitimately contain packets from several interfaces whose timestamps have different resolutions and different offsets.
- **Enhanced Packet Block (EPB)** contains a single captured packet with: Interface ID (32 bits), Timestamp High and Timestamp Low (32 bits each — a 64-bit value "represented as 2 32-bit values... each written as 32-bit integers in the endianness of the machine that wrote the file", and the spec warns that 64-bit values are "not aligned to 64-bit boundaries"), **Captured Packet Length** ("the number of octets captured from the packet", being "the minimum value among the Original Packet Length and the snapshot length for the interface"), **Original Packet Length** ("the number of octets of packet data that would have been provided had the packet not been truncated... If no truncation was done, it will be the same as the Captured Packet Length, but it will be different... if the packet has been truncated by the capture process"), and Packet Data ✅.
- **Simple Packet Block (SPB)** is the minimal form, with no Interface ID — all SPBs implicitly belong to the interface of the first IDB — and in a Section with more than one interface, packets from other interfaces "MUST be contained in an Enhanced Packet Block" ✅.
- **Interface Statistics Block (ISB)** stores capture statistics for an interface, including dropped packets, "useful to understand the conditions in which the capture has been made", normally at the end of the file "but no assumptions can be taken about its position — it can even appear multiple times for the same interface" ✅. This is §4.5's completeness evidence, in the file.
- **Name Resolution Block (NRB)** maps numeric addresses to names; **Custom Block** carries vendor data "in a portable fashion", with paired type codes to express whether a tool may safely copy it into a rewritten file ✅.
- The old **Packet Block** is obsolete and "SHOULD NOT appear in newly written files"; it is documented only in an appendix — and it is worth knowing that its header contained a **Drops Count** field ✅, a per-packet-attached drop counter that the modern EPB does not carry.

### 6.4 The Fields That Matter to a Practitioner

| Field / property | Where it lives | Why it decides your incident |
| --- | --- | --- |
| SnapLen | pcap File Header; pcapng IDB | Defines the truncation contract. A 96-byte snap is fine for headers and useless for payload forensics |
| Captured vs Original Packet Length | pcapng EPB (and SPB's Original length) | The only in-file evidence that a packet was truncated, and by how much |
| Timestamp resolution | pcap magic number; pcapng IDB (`if_tsresol`) | Bounds every latency claim (see §5) |
| Link type | pcap File Header; pcapng IDB | Misidentified, every offset in the file is wrong |
| Interface identity | pcapng IDB (Interface ID, name/description options) | In a multi-interface file, the difference between "the firewall dropped it" and "the server never sent it" |
| Capture comments | pcapng `opt_comment` and capture-level comment support | The human context — "packet 17 is the beginning of all our problems" is the specification's own example ✅ |
| Interface statistics | pcapng ISB | Completeness evidence (§4.5) |
| Decryption secrets | pcapng can embed decryption secrets (per Wireshark's TLS documentation, "Embedding decryption secrets in a pcapng file") ✅ | Convenient, and a secret-handling decision with consequences (§9, §10) |

### 6.5 The Indexing and Retrieval Problem

Neither format defines an index. In pcap, a reader finds the last record by walking every record; in pcapng, the duplicated Block Total Length exists explicitly so a reader can walk blocks in either direction, which is navigation, not indexing ✅ (both per the specifications above).

So the question "find the five minutes I need in a month of traffic" is answered entirely by infrastructure *you* build around the files, and the viable answers are structural, not clever:

1. **Time-partitioned filenames and directories.** Every capture tool supports rotation: dumpcap's ring buffer and autostop options (`-b duration|files|filesize|interval|packets`, `-a duration|files|filesize|packets`, with the filenames carrying a number and a creation timestamp) ✅, and tcpdump's `-C`/`-G` rotation ✅. Rotation is the index: the filename is a queryable key, and a search is a directory listing plus a filename filter.
2. **A catalogue written at capture time.** One row per file: path, start/end time, capture point, interface, filter expression, tool and version, packet count, byte count, and the drop counters for the window. This is the artefact that makes retrospective loss analysis possible at all; without it, the counters are gone.
3. **Consistent NTP/PTP-aligned time in filenames and catalogue**, so that a query by time does not require reading files.
4. **A retention policy expressed in time, not in size**, because the legal and regulatory questions are about windows ("what happened on the 14th between 02:00 and 02:15"), not about bytes. §7 gives the arithmetic that makes the policy affordable; §11 gives the obligations that shape it.

### 6.6 Compression, and Its Trade Against Retrieval

Compression is the obvious lever on §7's arithmetic and it has a real cost. Wireshark's capture engine supports it — Wireshark 4.6.x introduced compressing live captures while writing, with the release notes stating that "Live captures can be compressed while writing. (Previously there was support for compressing when performing multiple file capture, at file rotation time.) The `--compress` option in TShark works on live captures as well" ✅ (Wireshark 4.6.8 release notes, issue 9311).

The trade, stated without inventing ratios (no compression ratio was verified at source this pass — §15):

- Compressed files cannot be **read in place**. A tool must decompress, and random access within a compressed stream costs more than random access within a raw file.
- Compression adds **CPU to the capture path**, which is the path already at risk of falling behind (§4.1, location 5).
- Compression is not encryption. A compressed capture of an authenticated session still contains the credentials.

The defensible design is therefore: keep the recent window uncompressed for fast retrieval, compress older windows, and never let the compression on the write path be the reason a capture dropped.

### 6.7 The Write-While-Capture Durability Question

A capture that is being written when the host crashes is truncated, and truncated capture files are usually *readable* — which is worse, because a truncated file has all the appearance of a complete short one. The mechanisms that exist:

- **Rotation with bounded files.** dumpcap's ring buffer (`-b filesize` and `-b files`, with the files limit documented as required to be under 100,000, and a note that "some filesystems do not handle many files in a single directory well") means at most one file is at risk at a time, and its bounded size bounds the loss ✅.
- **Flushing.** tcpdump documents that SIGUSR2 along with `-w` "will forcibly flush the packet buffer into the output file" ✅ — the operational meaning being that buffered packets are otherwise an unquantified hole at the tail of the file.
- **Writing to a filesystem that tolerates it.** Where the capture writes matters as much as what it writes; this is the one part of this section that is a facilities question (cross-reference [L2 Design for Financial Data Centres](l2_design_financial_data_centers_guide.md) for the deployment context and [Data Center](data_center_guide.md)).
- **Treating the tail as suspect.** For any evidence-grade capture, validate the file (walk it to the last block) rather than assuming it ends where it should. pcapng's duplicated block length at each block's end is what makes this check cheap for a tool that bothers to perform it ✅.

---

## 7. The Volume Arithmetic

**Everything in this section is illustrative arithmetic from stated assumptions.** None of these figures is a measurement of any system, and none should be quoted as a benchmark. The point of the section is to show the *method*, so a reader can substitute their own three numbers and get their own answer. The unit chain is shown step by step, with the numbers verified by computation.

### 7.1 The Assumptions, Stated Plainly

| Assumption | Value | Note |
| --- | --- | --- |
| Average packet size on the wire | **700 bytes** | Illustrative mixed-traffic average. A capture is a distribution, not an average; a real estimate should be built from the link's packet-size histogram |
| Sustained packet rate | **50,000 packets/s** | Illustrative. This is the number a capture rig has to survive, not the number on the business case |
| Retention | **30 days** | Illustrative window; a bank's real number comes from the obligations in §11 |
| Units | **Decimal throughout** (1 KB = 1,000 B; 1 GB = 1,000,000,000 B; 1 TB = 1,000,000,000,000 B) | Label the units you use; TiB-based figures are about 9% smaller than TB-based ones for the same bytes |

### 7.2 The Unit Chain, Computed

**Step 1 — bytes per second.**

> 700 bytes/packet × 50,000 packets/s = **35,000,000 bytes/s = 35 MB/s**

(For contrast in the other unit system: 35,000,000 / 1,048,576 = **33.4 MiB/s**.)

**Step 2 — per hour.**

> 35,000,000 B/s × 3,600 s = 126,000,000,000 B = **126 GB/h**

**Step 3 — per day.**

> 126 GB/h × 24 h = 3,024 GB/day = **3.024 TB/day**

**Step 4 — per week.**

> 3.024 TB/day × 7 = **21.168 TB/week**

**Step 5 — per 30-day month.**

> 3.024 TB/day × 30 days = **90.72 TB**

**Step 6 — per year, for the impatient.**

> 3.024 TB/day × 365 days = **1,103.76 TB ≈ 1.10 PB/year**

**Step 7 — the unit chain, written as a reader can substitute it.**

> bytes/s = packet_size × packets/s
> GB/h = bytes/s × 3,600 / 10⁹
> TB/retention = GB/h × 24 × days / 1,000

The chain is the deliverable. Anyone can replace 700 and 50,000 with their own two numbers and get a defensible estimate in three lines of arithmetic.

### 7.3 The Retention Table and Why It Matters More Than the Total

| Window | Volume at the stated assumptions | What it buys |
| --- | --- | --- |
| 1 hour | 126 GB | The incident, if you arrive within the hour |
| 1 day | 3.024 TB | The incident, plus the day's context |
| 1 week | 21.168 TB | The intermittent fault that recurs weekly (see §12) |
| 30 days | 90.72 TB | The monthly close, the dispute window, the investigating auditor |
| 1 year | 1.10 PB | Almost never justified in full-rate packet form; see §7.6 |

**The useful conclusion is not "90 TB is a lot". It is that the retention window, not the capture rate, is what makes full-rate capture unaffordable — and retention is the parameter that policy, not engineering, decides.** Cutting the window from 30 days to 7 cuts the cost by ~76%; halving the packet rate halves it; narrowing the capture filter to one protocol cuts it by whatever fraction that protocol is.

### 7.4 The Honest Conclusion About Full-Rate Capture for a Given Link Speed

A capture rig must handle the link's line rate, not the average. Pure wire-byte arithmetic, again illustrative:

| Link speed | Bytes/s at line rate | Per day | Per 30 days |
| --- | --- | --- | --- |
| 1 Gbit/s | 125,000,000 | 10.8 TB | 324 TB |
| 10 Gbit/s | 1,250,000,000 | 108 TB | 3,240 TB (3.24 PB) |
| 100 Gbit/s | 12,500,000,000 | 1,080 TB | 32,400 TB (32.4 PB) |

Read that as an order-of-magnitude statement about *storage*, and then add the parts it omits: the capture rig must sustain the **packet rate** implied by that link speed at plausible packet sizes (which is where §3's default path becomes inadequate), and the analysis estate must be able to open the result. The honest conclusion:

> **Full-rate, full-retention capture of a busy link is not a storage purchase; it is a business decision to fund a data estate, and it is almost never the right first answer.** The right first answer is a narrow filter on a known problem, a short window, and measured completeness — with full-rate capture reserved for the windows where it is genuinely required and for point evidence in disputes.

### 7.5 The Metadata Alternative, Computed

Flow records are the summary, and the arithmetic is different by orders of magnitude. Illustrative again:

- Assume **2,000 new flows/s** and **200 bytes per flow record**.
- 2,000 × 200 = **400,000 B/s = 0.4 MB/s**
- Per day: 400,000 × 86,400 = 34,560,000,000 B = **34.56 GB/day**
- Per 30 days: **1.037 TB** — call it **~1.04 TB**

- Now assume the exporter **samples 1 in 1,000** flows (the sFlow specification defines the sampling rate exactly this way: "a sampling rate of 100 specifies that, on average, 1 sample will be generated for every 100 packets observed" ✅ sFlow v5, July 2004).
- 2 flows/s × 200 B = **400 B/s** = 34.56 MB/day ≈ **1.04 GB per 30 days**

**So the same 30-day window costs ~90.72 TB as full-rate packets, ~1.04 TB as unsampled flow records, and ~0.001 TB as 1:1,000-sampled flow records.** Three different instruments with three different evidentiary properties, and the point of the table is that the choice between them is a choice about what questions you will be able to answer later — not a linear cost trade.

What each buys: a flow record answers "who talked to whom, how much, when, on which ports" over a long window at negligible cost. A packet capture answers "what exactly was said", and only over the window you can afford to store. A bank needs both, and they are complements, not substitutes.

### 7.6 The Sampling Question and What It Costs in Evidentiary Value

Sampling is the mechanism that makes long-window metadata affordable, and it has a precise definition worth quoting rather than paraphrasing: sFlow's sampling rate is the "ratio of packets observed at the Data Source to the samples generated", and the sampler's randomness requirement is only that "the mean value of the numbers it generates converges to the required sampling rate" ✅ (sFlow v5 specification, July 2004, including Appendix B on random number generation). IPFIX, the IETF's own flow-export standard, took the reliability question seriously enough to specify **Metering Process Reliability Statistics** and **Exporting Process Reliability Statistics** options templates in the protocol itself ✅ (RFC 7011 / STD 77, September 2013) — i.e. the standard expects an exporter to tell the collector how trustworthy its own metering was.

In evidentiary terms:

- **Sampling cannot prove absence.** A 1:1,000 sample that contains no message of type X is consistent with no such message, and equally consistent with a thousand such messages.
- **Sampling does not preserve sequence.** It answers volume and participant questions, not protocol-correctness questions.
- **Sampling is invisible in the output** unless the tool that produced it says so, which is why the IPFIX reliability templates and the sFlow sampling-rate field exist, and why a derived metric must carry its provenance.
- **Full-rate capture is not automatically more trustworthy than sampled metadata** — a full-rate capture with unmeasured drops (§4) is evidence of unknown completeness, while a sampled flow record with a declared sampling rate and reliability statistics is evidence of *known and stated* completeness. **Declared sampling beats undeclared loss.**

### 7.7 The Rule for This Section

Compute before you buy, state the three assumptions out loud, and label every figure illustrative. A capture design that starts with the retention window and the filter — rather than with the link speed and a "capture everything" instinct — is one that survives its first finance review.

---

## 8. The Analysis Tooling, with Dated Statuses

Every tool below was checked this pass, and every entry states what it is for, what it does **not** do, its current release, and whether it is actively maintained. This is a field with celebrated abandoned classics, so **activity is established, not assumed** — and where a tool is dormant or legacy, it is labelled as such rather than presented as the recommendation.

### 8.1 tcpdump / libpcap

- **What it is for.** tcpdump is the command-line packet analyzer; libpcap is "a portable C/C++ library for network traffic capture" that underpins it and most everything else ✅ (tcpdump.org). Its capture-filter syntax (`pcap-filter`) is the de facto language of BPF filtering and is consumed by other tools, including Wireshark's capture filter field ✅ (dumpcap(1) points at `pcap-filter(7)`).
- **What it does not do.** It is not a protocol state machine, not an IDS, and not an indexed store. It prints or writes, and the analysis is yours. Its drop accounting at exit has the platform caveats set out in §4.2, and `pcap_stats()` is unavailable on saved files ✅.
- **Current release and maintenance status.** ✅ **Actively maintained, and released this month.** Stable **tcpdump 4.99.7 (2026-09-21)**; stable **libpcap 1.10.7 (2026-09-05)**; a **testing libpcap 1.11.0 (2026-09-13)** described as the result of "nearly six years of development" and as improving support for Endace DAG, Myricom SNF, netmap and RDMA while **removing any remaining support for AirPcap, DPDK, Riverbed TurboCap, Septel and SITA ACN** ✅ (tcpdump.org, "Latest Releases", retrieved 2026-09-22). The tcpdump man page verified this pass is dated 31 July 2026. Note the release-channel quirk: the project publishes tarballs on tcpdump.org and has no GitHub releases at all ✅.
- **Why it matters to this guide.** It is the reference implementation of the loss and timestamp accounting in §4 and §5 — the man pages quoted there are its man pages.

### 8.2 Wireshark, tshark, dumpcap (and the Wireshark tool family)

- **What each is for.** Wireshark is the graphical protocol analyzer (the project's own phrasing: "the world's most popular network protocol analyzer. It is used for troubleshooting, analysis, development and education" ✅). **tshark** is its command-line sibling, scriptable and the right tool for automated dissection. **dumpcap** is the capture engine: "a network traffic dump tool. It lets you capture packet data from a live network and write the packets to a file" ✅ (dumpcap(1)). The same family provides `editcap`, `mergecap` and `capinfos` ✅.
- **What it does not do.** Wireshark does not create timestamps (§5.1) and does not guarantee decryption (§9). It is also not a flow-analysis or long-term-storage tool; its purpose is deep single-trace analysis, and it is the wrong place to keep a month of traffic.
- **Current release and maintenance status.** ✅ **Actively maintained, and released this month.** Stable **4.6.8** and old-stable **4.4.18**, both released **12 August 2026** (the release announcement describes 4.6.8 as fixing "quite a few vulnerabilities... due to the recent trend in AI-assisted vulnerability reports"), plus a development release 4.7.3 ✅ (wireshark.org/download, retrieved 2026-09-22; wireshark.org/news). On Windows, live capture requires **Npcap**, and "each Windows package comes with the latest stable release of Npcap" ✅. New in the 4.6 line: live captures can be compressed while writing, with `tshark --compress` working on live captures ✅.

### 8.3 Zeek

- **What it is for.** A scriptable network analysis framework — "a framework for network traffic analysis and security monitoring" per the project's own README, "The Zeek Network Security Monitor", with key features including in-depth protocol analysis, a domain-specific scripting language for site-specific monitoring policy, high-performance operation, and extensive application-layer state ✅ (Zeek README). Its distinguishing property for this guide is that it turns traffic into typed logs, and one of those logs (`capture_loss.log`) is a completeness measurement ✅.
- **What it does not do.** It is not a packet-store: dumpcap/tcpdump does that, and Zeek can consume their output or capture directly. It is not a signature IDS in the Snort/Suricata sense — its detection model is scripted and stateful, not rule-matcher-first. And it cannot report loss it cannot infer: `percent_lost` is derived from TCP sequence gaps ✅, so it is a measure that requires observable TCP.
- **Current release and maintenance status.** ✅ **Actively maintained.** Latest **v9.0.0, released 21 August 2026**, with v8.2.2 and v8.0.10 also in the visible release list ✅ (github.com/zeek/zeek releases, retrieved 2026-09-22). Breaking changes in 9.0 are documented and worth knowing about before an upgrade: Broker's legacy embedded WebSocket server is removed; and **"Beginning with Zeek 9.0, the Zeek team is sunsetting our official support of FreeBSD"** ✅. Notable new capability relevant to §2: the **zeek-packet-source-udp** plugin is now included and enabled by default on Linux and FreeBSD, turning Zeek into a UDP server that processes "UDP-based VXLAN or GENEVE mirror traffic directly" (e.g. `zeek -i udp::0.0.0.0:4789:vxlan`), using `SO_REUSEPORT` so the kernel balances packets across workers by outermost UDP header hash ✅ — that is the cloud-mirror ingestion path in product form.

### 8.4 Suricata

- **What it is for.** An open-source IDS/IPS and network security monitoring engine (project: Open Information Security Foundation). Its configuration model is documented in detail, including runmodes, `max-pending-packets`, `default-packet-size`, rule action ordering (pass, drop, reject, alert, with pass processed first) and its statistics output ✅ (Suricata `suricata.yaml` documentation).
- **What it does not do.** It is not a packet archive, and its statistics — while useful for engine health — are not a substitute for the capture-point instrumentation of §4.5.
- **Current release and maintenance status.** ✅ **Actively maintained.** Latest **8.0.7, released 15 September 2026**; the previous major line is also maintained, with **7.0.17 released 7 July 2026** ✅ (github.com/OISF/suricata releases, retrieved 2026-09-22). Relevant version-specific fact: **"Suricata 8.0 sandboxes Lua rules by default"**, with the restrictions configurable and Lua rules defeatable entirely ✅ (Suricata documentation). Statistics are produced by default in two forms: a text `stats.log` and an EVE record type `stats`, both enabled by default with a configurable interval (documented example: 8 seconds) ✅.

### 8.5 Snort

- **What it is for.** The long-established signature-based IDS, now Snort 3, with a large rule ecosystem operated by Cisco Talos.
- **What it does not do.** Again, not a capture store, and not a passive traffic-analytics framework. Also relevant: the rule feed and the engine have distinct lifecycle and licensing considerations.
- **Current release and maintenance status.** ✅ **Actively maintained (Snort 3), with a legacy line still published.** The download page lists **snort3 3.12.2.0** with **libdaq 3.0.27** and **libml 2.0.0**, and a rule advisory dated **17 September 2026**; the same page still hosts the **Snort 2.9** line (2.9.20, with daq 2.0.7) ✅ (snort.org/downloads, retrieved 2026-09-22). **Read that correctly: Snort 2.9.x is a legacy line kept available for existing deployments, not a current recommendation** — new deployments are Snort 3.

### 8.6 The Abandoned Classics — Why This Section Is Written the Way It Is

The packet-analysis field is littered with tools that every guide still cites and nobody still ships. Two verifiable signals from this pass, offered as method rather than as accusation:

1. **Capability removal is a status signal.** libpcap 1.11.0 removed remaining support for several capture modules (AirPcap, DPDK, Riverbed TurboCap, Septel, SITA ACN) alongside platform removals ✅. A capture framework that has been discontinued by libpcap is not where a new project should start, whatever the older documentation says.
2. **A repository's recent commit subjects are a status signal.** netmap's 2026 commits are kernel-compatibility fixes, FreeBSD metadata clean-up and a CodeQL workflow ✅ — the signature of a maintained-but-not-developing project, which is a perfectly good thing to be if you know it. By contrast, PF_RING's tag 9.4.0 (29 August 2026) and DPDK's 26.07.0 (22 July 2026) and 25.11.3 LTS (20 August 2026) are releases ✅.

**The rule: name no tool in a design without a dated status, and never inherit a tool's reputation from a document that has not been re-checked.**

### 8.7 The Status Table

| Tool / framework | Role | Current version (as verified 2026-09-22) | Status |
| --- | --- | --- | --- |
| tcpdump | CLI capture + analysis | 4.99.7 (2026-09-21) | ✅ Actively maintained, released this month |
| libpcap | Capture library | 1.10.7 stable (2026-09-05); 1.11.0 testing (2026-09-13) | ✅ Actively maintained; 1.11.0 removed several legacy capture modules |
| Wireshark | GUI protocol analyzer | 4.6.8 stable; 4.4.18 old-stable (both 2026-08-12); 4.7.3 dev | ✅ Actively maintained |
| tshark / dumpcap / editcap / mergecap / capinfos | CLI dissection, capture engine, file surgery, file stats | Shipped with Wireshark 4.6.8 | ✅ Actively maintained |
| Npcap | Windows capture driver required by Wireshark for live capture | Shipped with Wireshark packages (version not captured this pass — §15) | ✅ In use; version flagged |
| Zeek | Scriptable network analysis framework; produces typed logs incl. capture-loss | 9.0.0 (2026-08-21); 8.2.x/8.0.x maintained | ✅ Actively maintained; FreeBSD support sunset at 9.0 |
| Suricata | IDS/IPS + NSM engine | 8.0.7 (2026-09-15); 7.0.17 (2026-07-07) | ✅ Actively maintained |
| Snort 3 | Signature IDS | snort3 3.12.2.0 (page retrieved 2026-09-22); libdaq 3.0.27; rules advisory 2026-09-17 | ✅ Actively maintained |
| Snort 2.9 | Legacy signature IDS | 2.9.20 with daq 2.0.7 still published | ⚠ Legacy line — do not choose for new deployments |
| DPDK | Userspace packet processing (kernel bypass) | 26.07.0 (2026-07-22); LTS 25.11.3 (2026-08-20) | ✅ Actively released |
| PF_RING | High-rate packet processing framework (out-of-tree module) | Tag 9.4.0 (2026-08-29); commit 2026-09-21 | ✅ Actively maintained |
| netmap / VALE | Fast packet I/O framework | No release tags; commits 2026-08-08, 2026-07-10, 2026-05-26, 2026-04-18 | ✅ Maintained, low-velocity, maintenance-paced |
| AF_XDP | Kernel address family for high-performance packet processing | In mainline kernel (docs rendered for 7.3.0-rc4) | ✅ Part of the kernel, not an add-on |

§14 carries the source and date for every row above.

---

## 9. The Decryption Problem

### 9.1 What Encryption Does to a Capture

A capture of an encrypted session is a capture of the traffic, not of the conversation. The packets are real, complete, correctly ordered and correctly timed — and the payload is opaque. Everything this guide has said about loss (§4) and timestamps (§5) still applies, and then the analytical value is bounded by what can be inferred from behaviour rather than content.

**No verified statistic on the share of enterprise traffic that is encrypted was obtained this pass, so none is asserted here** (§15). What can be stated from verified sources is directional and sufficient for design: TLS is the transport for web, API and control-plane traffic; the SSLKEYLOGFILE key-log format's own specification covers TLS 1.2, TLS 1.3, DTLS, QUIC and "other protocols that use the TLS key schedule", and extends to Encrypted Client Hello ✅ (RFC 9850, December 2025). Software that knows what is on the wire in 2026 should assume a large and increasing fraction of it is unreadable by default.

### 9.2 The Documented Mechanisms to Make a Session Readable

Wireshark's TLS documentation is the canonical practitioner reference and states the three methods plainly: "Wireshark supports TLS decryption when appropriate secrets are provided. The three available methods are: Key log file using per-session secrets; Decryption using an RSA private key; Decryption using a Pre-Shared-Key (PSK)" ✅ (Wireshark wiki, TLS). Each has a mechanism and a boundary.

**The key-log mechanism — what it actually is.** The key log file "is a text file generated by applications such as Firefox, Chrome and curl when the `SSLKEYLOGFILE` environment variable is set. To be precise, their underlying library (NSS, OpenSSL or boringssl) writes the required per-session secrets to a file" ✅ (Wireshark wiki, TLS). The format is now a published IETF document: **RFC 9850, "The SSLKEYLOGFILE Format for TLS", Informational, December 2025** ✅ (authors Thomson/Mozilla, Rosomakho/Zscaler, Tschofenig). Field-level facts a practitioner needs:

- A key log file is a UTF-8 text file; each secret is one line of three space-separated values: the **label** (secret type), the **`client_random`** — "the 32-byte value of the Random field from the ClientHello... encoded as 64 hexadecimal characters", which "can be used to identify a connection" in a multi-connection log — and the **secret** value in hexadecimal ✅ (RFC 9850 §2).
- Labels include the TLS 1.3 traffic-secret family (`CLIENT_HANDSHAKE_TRAFFIC_SECRET`, `SERVER_HANDSHAKE_TRAFFIC_SECRET`, `CLIENT_TRAFFIC_SECRET_0`, `SERVER_TRAFFIC_SECRET_0`), `EXPORTER_SECRET`, the TLS 1.2-era `CLIENT_RANDOM`, and ECH-specific labels (`ECH_SECRET`, `ECH_CONFIG`) ✅ (RFC 9850 §2 and Appendix A).
- Lines are ignored if empty or beginning with `#`; comments are permitted, but the file "MUST NOT contain a Unicode byte order mark"; tools "MUST accept CRLF... CR... or LF" as line terminators ✅.
- Consumers: Wireshark (the "(Pre)-Master-Secret log filename (`tls.keylog_file`)" preference) and, importantly, **pcapng files themselves can carry embedded decryption secrets** — the Wireshark wiki documents "Embedding decryption secrets in a pcapng file", and the test-capture convention `tls12-dsb.pcapng` is described as a TLS 1.2 trace "with decryption keys embedded" ✅. (Storing keys and traffic together in one object is a data-protection decision, not a convenience: see §10.)

**The RSA-private-key route, and its boundaries.** The wiki is unusually clear that the server private key "only works in a limited number of cases", listing them: the cipher suite selected must not use (EC)DHE; the protocol must be SSLv3 or (D)TLS 1.0–1.2, because it "does *not* work with TLS 1.3"; the private key must match the **server** certificate (not the client's, not the CA's); and "the session has not been resumed. The handshake must include the **ClientKeyExchange** handshake message" ✅ (Wireshark wiki, TLS). For a 2026 estate, the practical reading is blunt: **for TLS 1.3, the private key is useless for decryption**, because the session keys derive from an ephemeral exchange the private key does not enter.

**The PSK route.** Where a TLS session uses a pre-shared key, supplying it in the analyzer's PSK preference decrypts the session; the wiki notes the property that makes this powerful and dangerous: "Since the PSK doesn't usually change, the same PSK will decrypt all traffic, past and future, that uses that key" ✅.

**Non-TLS cases with keys in hand.** tcpdump can decrypt IPv4 IPsec ESP when given the SPI, the address, the algorithm and the secret (`-E spi@ipaddr algo:secret`, with `des-cbc`, `3des-cbc`, `blowfish-cbc`, `rc3-cbc`, `cast128-cbc` or `none` as algorithms, defaulting to `des-cbc`), available only if compiled with cryptography enabled — and the man page itself warns the option "is only for debugging purposes, and the use of this option with a true 'secret' key is discouraged. By presenting IPsec secret key onto command line you make it visible to others, via ps(1) and other occasions" ✅ (tcpdump(1)).

### 9.3 The Honest Limits — Where None of This Helps

**Capture-based decryption is not a general capability, and the specification that defines the key-log format says so in its own applicability statement.** RFC 9850: "The artifact that this document describes — if made available to entities other than endpoints — completely undermines the core guarantees that TLS provides. This format is intended for use in systems where TLS only protects test data. While the access that this information provides to TLS connections can be useful for diagnosing problems while developing systems, **this mechanism MUST NOT be used in a production system.** For software that is compiled, use of conditional compilation is the best way to ensure that deployed binaries cannot be configured to enable key logging." ✅ (RFC 9850 §1.1).

Read that sentence for what it establishes, because it is the strongest sourced statement in this section. It is not merely "don't do this". It establishes that **the ecosystem's documented way of making TLS readable is architecturally unavailable in production by design** — and that compiled production software can be built so that key logging is *impossible to enable*. The cases in which decryption therefore does not work, with the primary source for each:

| Case | Why the documented mechanisms do not help | Source |
| --- | --- | --- |
| TLS 1.3 session with no key log | Static private keys do not derive session keys; the RSA-key route "does *not* work with TLS 1.3" | ✅ Wireshark wiki, TLS |
| Any (EC)DHE session | The RSA-key route requires that the cipher suite "is not using (EC)DHE" | ✅ Wireshark wiki, TLS |
| Resumed session | The RSA-key route requires the handshake to include ClientKeyExchange; a resumed session does not | ✅ Wireshark wiki, TLS |
| Client-certificate-authenticated or mutual-TLS links where only the counterparty holds its key | The RSA-key route requires the private key matching the **server** certificate, "not the client certificate, nor the Certificate Authority (CA) certificate" | ✅ Wireshark wiki, TLS |
| Production software built without key-log capability | "For software that is compiled, use of conditional compilation is the best way to ensure that deployed binaries cannot be configured to enable key logging" | ✅ RFC 9850 §1.1 |
| Appliances, HSMs and third-party stacks that never expose per-session secrets | The key-log mechanism presupposes an application that writes the secrets to a file | ✅ mechanism per RFC 9850 / Wireshark wiki; the appliance consequence is a reasoning step, flagged ⚠ |
| Sessions inside a pinned or proprietary tunnel whose endpoints you do not control | Decryption requires a secret; capture alone supplies none | ⚠ reasoning from the above; not separately sourced this pass |
| PSK-protected sessions whose key you were never given | PSK decryption requires knowledge of the PSK | ✅ Wireshark wiki, TLS |

**The point of that table for a bank:** the traffic that matters most — counterparty links, SWIFT connectivity, MQ-over-TLS, payment APIs, cloud control planes — is precisely the traffic least likely to have an exportable key log, because it runs in hardened, often vendor-supplied stacks, sometimes on HSMs, and often over TLS 1.3 with mutual authentication. A capture programme designed on the assumption that it can read content will discover this at the worst possible moment.

### 9.4 What an Undecryptable Capture Is Still Worth

Which is the answer most teams actually need, because it is the answer that works today:

1. **Topology and participants, exactly.** Which addresses and ports talked, who initiated, whether the session was carried end-to-end or through a sequence of hops.
2. **Volumes and rates per direction.** Bytes and packets each way, which the TCP headers give without any payload access.
3. **TCP behaviour.** Retransmission patterns, window behaviour, resets, sequence gaps, acknowledgement timing, connection teardown. This is the empirical basis of the loss inference in §4 (Zeek's `percent_lost` is derived from TCP sequence numbers ✅) and it survives full encryption.
4. **Latency and jitter decomposition, at the granularity §5 permits.** Where the capture timestamps were verified, request-to-response timing per hop; where they were not, the timing is a lower bound at best.
5. **Handshake metadata.** Observed TLS handshake traffic — including the client's hello and its parameters, and, for TLS 1.2 and earlier, certificates in the clear. **Do not assume the same for TLS 1.3**, where the certificate exchange is protected by the handshake keys, and do not assume ClientHello visibility survives Encrypted Client Hello ✅-partial (RFC 9850 documents ECH secret labels, which presupposes ECH in the field; the specific visibility consequences are not separately sourced here).
6. **Fault localisation without content.** For back-pressure, connectivity, message-flow and throughput faults — the majority of the banking use cases in §11 — the packet-level behaviour is frequently sufficient: a session that is being reset by a middle device, a window that collapses to zero, a path that starts retransmitting at a specific moment, an MTU discovery failure visible as consistent black-holing of large segments. None of these require reading the payload.

### 9.5 The Rule for This Section

Design capture programmes for metadata value first and treat content as a bonus that must be separately authorised, separately justified and separately protected. Where content is genuinely required — an investigation, a dispute, a development or test qualification — the successful cases will be ones where the endpoint's own software was built or configured to export secrets, in a non-production context, with the specification's own prohibition understood rather than ignored.

---

## 10. The Security and Privacy Consequences

A capture file is not a log. It is a recording of other people's data, other people's credentials and other people's behaviour, made by a capability that is trivially available and trivially abusable. This section states the consequences from named sources and cross-references the repo's governance material rather than re-deriving it.

### 10.1 What a Capture File Actually Contains

- **Authentication material.** Any session whose credentials or tokens are not protected by an unbroken end-to-end encryption layer has them in the file, in the clear, at the byte level. This includes legacy protocols inside an estate that are still in use precisely because they are legacy: an internal LDAP bind, an FTP control channel, an HTTP management interface, an SNMP community string, a database login, a basic-auth API call. Wireshark's TLS documentation notes that a capture with embedded decryption secrets is a supported artefact ✅ — which is the same risk stated in the optimising direction.
- **Personal data.** Addresses, identifiers, customer references, email addresses, names in protocol payloads, session cookies and content of communications. Even a **flow record** is treated by its own standard as disclosure-sensitive: RFC 3954's security considerations include a section on "Disclosure of Flow Information Data" ✅ (RFC 3954, October 2004). If a summary record requires a disclosure warning, a full capture requires more.
- **The ordinary business content of the traffic.** Trade instructions, payment messages, contract text, file transfers, market data, database results.
- **The capture's own metadata.** Which interfaces were observed, which filter was applied, what the analysis tool is, and — where the capture rig wrote it — its statistics blocks. In an investigation, the capture's metadata is often as sensitive as its payload, because it reveals what the monitoring estate can see.

### 10.2 The Authorisation Question

The technical position is uncomfortable and worth stating precisely: capturing traffic on a network segment you have access to requires no special tooling — `CAP_NET_RAW` on a host ✅ (packet(7)), administrative access to a switch to create a mirror ✅ (Cisco SPAN documentation, §2.3), or the cloud mirroring permission in a project. **The tooling makes it trivially easy to capture traffic you are not permitted to capture.**

⚠ **No specific legal instrument was verified at source this pass**, so this guide does not cite one (§15). What it does instead is state the governance requirement that follows from the technical facts, and route the legal framework to the guides that own it: [MAS Regulations & Guidelines](../banking/mas_regulations_guidelines_guide.md) (MAS and PDPA obligations), [Payment Rails](../banking/payment_rails_guide.md) (payment-data handling), [Market Data Consumption](../banking/market_data_consumption_guide.md) (market-data licensing and handling), and — for the security-programme context — [Cybersecurity](cybersecurity_guide.md), [Security by Design](security_by_design_guide.md) and [Beyond Zero: Enterprise Security](beyond_zero_enterprise_security_guide.md). For internal testing that looks exactly like an attack, [Penetration Testing Execution Standard](penetration_testing_execution_standard_guide.md) owns the authorisation discipline; a capture during a red-team engagement must be covered by the same authorisation as the test.

The minimum bar that follows, and it is deliberately boring:

1. **Named authority.** A named role authorises captures; the authorisation names the segment, the interface, the filter, the purpose and the end date.
2. **Positive scope.** The default is no capture. Authorisation is a positive act with a written scope, not the absence of a prohibition.
3. **A register.** Every capture instance is recorded: who, when, why, where, scope, and where the output went.
4. **Technical guardrails where they exist.** Capture filters that implement the authorisation, not filters that the operator chose for convenience; role separation so that whoever can create a mirror is not automatically whoever can read the output.
5. **Expiry.** Every standing capture has a review date, and the review removes it if nobody can justify it.

### 10.3 Data at Rest: Protecting the Capture Store

The capture store is a database of other people's data, and it needs the controls a database of that data would need:

- **Encryption at rest**, including the indexes, the catalogue and any extracted artefacts — and a key-management story for the medium (a NAS share mounted by an analysis workstation is not a protected store).
- **Encryption in transit** when captures move between a sensor and the store.
- **Separation of secrets from traffic.** Where a key log exists — in a test environment, per RFC 9850's applicability statement ✅ — it must not live in the same object as the traffic it decrypts. (pcapng's ability to embed decryption secrets ✅ is a feature with exactly one safe use: disposable test captures.)
- **Integrity and provenance.** Hashes, an immutable write path where the capture is potential evidence, and a record of every read.
- **A defined deletion path** that actually deletes: rotation that overwrites, storage that honours it, backups included in scope.

### 10.4 Access Control: Who May Read It

Read access to packet data is the highest-value access in the estate: a single capture file can contain credentials that unlock other systems. The controls that matter:

- **Least privilege, by case.** Access granted per investigation, not per team, and revoked at closure.
- **Two-person handling for anything evidence-grade**, mirroring the discipline the repo's financial-crime and audit material already applies to evidence.
- **Content minimisation at capture time.** SnapLen is an access-control control: capturing headers only (the Azure configuration documents 34 bytes for "only... the IPv4 header" ✅ as an explicit option, and dumpcap's default snapshot length of 262144 bytes is the opposite extreme ✅) reduces both volume and exposure. **The cheapest data-protection control ever invented is not recording the payload you will never read.**
- **Filter scoping.** The same argument applies to capture filters: a filter that excludes HR, legal and personal-banking segments is an access control implemented in a BPF expression (§13, anti-pattern 5).

### 10.5 The Regulatory Consequence for a Bank

The single most important regulatory statement in this guide is a consequence, not a citation: **a capture store that may hold cardholder or customer data is itself in scope for the obligations attached to that data.** A capture is not exempt because it is a diagnostic artefact; it is a copy of the data, with an additional characteristic that makes it worse — it is an unstructured, high-volume copy with no schema, no retention metadata and often no owner. Where a payment or customer-data obligation applies to the traffic, it applies to the file, and that means:

- retention limits apply to the capture store (**and a legal hold is an exception to, not a suspension of, the retention rule — §11.6**);
- subject-access, dispute and discovery obligations reach into it;
- cross-border transfer restrictions reach it, because a capture of a Singapore-hosted customer's session is that customer's data in whatever country the capture store sits in — cross-reference [MAS Regulations & Guidelines](../banking/mas_regulations_guidelines_guide.md) and [Singapore Data Centres](singapore_data_centres_guide.md) for the residency context;
- data-minimisation applies at design time, which is exactly the argument for header-only captures and tight filters;
- breach-notification logic applies, because a capture store is a place customer data lives.

**State it in the design document, not in the post-incident review:** classify the capture store as a data store, assign it an owner, and apply the obligations of the most sensitive traffic it can hold. If that traffic class cannot be justified, filter it out rather than protecting it.

---

## 11. The Regulated-Enterprise and Banking Angle

Banking is where capture earns its keep, because banking is where the failures are expensive, intermittent and disputed. This section is deliberately application-focused: the protocols belong to their own guides in this repository, and this guide's contribution is *how the wire answers the question you have about them*.

### 11.1 Diagnosing Connectivity and Message-Flow Faults in the Messaging and File-Transfer Estate

| Estate | The guide that owns the protocol | What the capture answers (which no log can) |
| --- | --- | --- |
| FIX session connectivity and sequencing | [FIX Protocol](../banking/fix_protocol_guide.md) | Whether a message left a host and whether it arrived; whether a gap is a counterparty-sent gap or a network failure; whether the connection was reset by a middle device. A FIX engine's log records what the engine did, not what survived the network |
| SWIFT connectivity (Alliance Access, SWIFTNet FileAct) | [SWIFT Alliance Access](../banking/swift_alliance_access_guide.md), [SWIFTNet FileAct](../banking/swiftnet_fileact_guide.md) | Whether a file or message reached the network boundary and when, with byte-level evidence of the transfer's completion; the timing of a session's establishment and teardown. Constraints: the connectivity is hardened and the traffic may be opaque (§9) |
| Managed file transfer | [Axway Transfer CFT](axway_transfer_cft_guide.md) | Whether an outbound transfer's bytes were pushed at the rate the MFT product claims; whether a retry was a network event or a product event; the exact point in a file's transfer where a stall began |
| Message queues (MQ over TLS) | [IBM MQ Disaster Recovery](ibm_mq_disaster_recovery_guide.md) | Channel-level behaviour: connection establishment, heartbeats, session drops, retransmission storms after a network event, and the difference between "the channel died" and "the peer stopped reading" |
| Payment paths | [Payment Rails](../banking/payment_rails_guide.md) | Latency attribution across hops for time-critical payment flows, and per-direction byte counts that corroborate (or contradict) a counterparty's volume claim. Note the §10.5 consequence: payment traffic makes the capture store a regulated data store |
| Market data | [Market Data Consumption](../banking/market_data_consumption_guide.md) | Whether a multicast or TCP feed's gaps are gaps in the sender's stream or gaps in your reception; feed latency measurement; the evidence base for a vendor performance dispute |

Across all six: **the capture is the arbiter between two parties' logs, and it is the only artefact that both parties will accept as neutral.** That is the business case, and it is a stronger one than "we might need to look at packets one day".

### 11.2 The Dispute and Investigation Use Case, Where the Capture Is the Evidence

When the capture is the evidence, three things change:

1. **The completeness question becomes legal, not technical.** §4.5's discipline is not optional: the counter readings, the capture-loss analysis and the environment record are what make the exhibit defensible, and the absence of them is what a competent opposing party attacks.
2. **The chain of custody becomes part of the capture pipeline.** Who wrote the file, on which sensor, into which store, read by whom and when — recorded as data, not remembered.
3. **Retention stops being a storage decision.** A capture that is the only evidence of a disputed message is subject to legal hold, which overrides the rotation policy that would otherwise have deleted it within days — and the rotation policy has to have an exception mechanism *before* the dispute, not after.

### 11.3 The Performance-Attribution Use Case

Attributing latency and loss to a specific component is where packet-level evidence is uniquely strong, provided the timestamps survive scrutiny (§5). The workable pattern: measure at two points, not one, so that the difference localises the delay to the segment between them; reason from protocol behaviour (retransmissions, window sizes, acknowledgement spacing) rather than from captured payload; and refuse to publish sub-millisecond conclusions from microsecond-resolution files ✅ (§5.2, §6.4).

### 11.4 The Security-Detection Use Case

Capture is the food supply of detection: Zeek for high-level behavioural logs and completeness measurement ✅, Suricata and Snort 3 for signature and protocol-anomaly detection ✅ (§8). The banking-specific considerations are that detection capture runs continuously (so §7's arithmetic applies in full), that it consumes the same mirrored stream the troubleshooting capture wants (so §2's broker question is a real design decision), and that the output of a detection capture is an alert stream whose underlying packets must be retrievable under the retention and access rules of §10. Cross-reference [Cybersecurity](cybersecurity_guide.md) and [Beyond Zero: Enterprise Security](beyond_zero_enterprise_security_guide.md) for the detection architecture, and [Penetration Testing Execution Standard](penetration_testing_execution_standard_guide.md) for the authority under which internal attack simulation traffic may be captured at all.

### 11.5 The Tension With a Zero-Trust Posture

Stated once, without re-deriving the architecture that [Zero Trust Network Architecture](zero_trust_network_architecture_guide.md) owns: a zero-trust posture reduces implicit trust in the network and increases the value of identity and policy evidence. A packet capture is a **deliberate visibility exception** to that posture: it reads traffic that the architecture's encrypted, authenticated, segmented design deliberately made hard to read, at a point the design deliberately tried to make uninteresting. The exception is legitimate — diagnosis, dispute and detection are real needs — but it is an exception, and exceptions are exactly what a zero-trust programme is built to inventory and govern. **The capture is not a contradiction of the posture; an ungoverned, unexpiring, unrecorded capture is.**

### 11.6 The Governance Question That Ties It Together

| Question | The answer that survives scrutiny |
| --- | --- |
| Who authorises a capture? | A named role, by positive written scope covering segment, interface, filter, purpose and end date (§10.2) |
| Who may hold the output? | The case team, by name, for the duration of the case; access as a grant, not as a team membership (§10.4) |
| How long is it retained? | A stated window per capture class, expressed in time, reviewed, and enforced by rotation rather than by hope (§6.5, §10.5) |
| How does a legal hold interact with a retention rule? | The hold is an **exception mechanism that must exist in advance**: flagged captures are exempted from rotation, moved to an evidence store, and recorded as exempted. A retention rule without a hold mechanism will delete evidence |
| How is it disposed of? | A defined deletion path — overwrite or secure erase, backups included, with a destruction record for evidence-grade captures (§10.3) |
| Who can prove all of the above happened? | The capture register and the catalogue (§6.5): the artefact that turns a capture programme into a governed one |

**And do not name a real institution as the user of any of this.** No bank in this repository, or anywhere else, is asserted to run these tools or these designs; the worked example that follows is explicitly fictional.

---

## 12. The Cymbal Bank Worked Example

> **This scenario is entirely fictional and illustrative.** Cymbal Bank is the repository's fictional persona; the counterparty, the volumes, the figures and the costs are invented for the purpose of showing the method. Nothing here describes a real institution's network.

### 12.1 The Situation

Cymbal Bank runs its core banking estate in its primary data centre and exchanges payment instructions and status messages with a payments counterparty over a dedicated cross-connect. Twice in eleven weeks, on a Sunday evening batch, a subset of instruction acknowledgements failed to arrive inside the counterparty's expected window. Both times the counterparty's logs showed the requests sent and the responses issued; both times Cymbal's application log showed nothing received. Both times the incident cleared within twenty minutes and the retry logic covered it. Neither time was any packet evidence retained.

Cymbal also has a regulatory and internal-audit expectation that, when something is disputed, it can produce wire evidence of what happened. Today it cannot: captures are taken ad hoc, by whoever is on shift, with no instrumentation and no retention.

The architecture team is asked to do two things: **find out what happened on that link**, and **stand up a standing capture capability that can be trusted when it matters.**

### 12.2 Choosing the Capture Point — and Why the Obvious Answer Is Wrong

The obvious answer is "put a SPAN on the switch port that faces the counterparty router, mirror everything, analyse it later". Cymbal rejects it, for four reasons that are all documented behaviours rather than opinions:

1. **The mirror destination port is finite and can drop** (§2.3), and the mirrored traffic would include every other flow to that router — so the drop risk is bought for no analytical benefit.
2. **The fault is intermittent and time-bounded.** A continuous mirror answers a question nobody asked (what does the link look like at all times) and creates a §7-sized storage problem for a fourteen-hour-a-week window of relevance.
3. **The direction question.** The dispute with the counterparty is about whether acknowledgements left *their* environment into the cross-connect and whether they arrived at Cymbal's gateway. That requires evidence at the demarcation on Cymbal's side — a point that shows the wire, not the switch's opinion of it.
4. **The trust question.** A capture that is going to be shown to a counterparty or an auditor should not depend on a switch configuration that a third party can change, and cannot be shown to have been correct after the fact.

Cymbal's decision: **a physical tap at the cross-connect demarcation** feeding a **broker**, which filters and fans out to two consumers — a continuous header-only capture, and a narrower session-scoped trigger capture — plus an independent capture on the payments gateway host itself at the far end, so that any claim can be checked at both ends. (Placement, redundancy and fail-to-wire behaviour are facilities questions: cross-reference [L2 Design for Financial Data Centres](l2_design_financial_data_centers_guide.md) and [Data Center](data_center_guide.md).)

### 12.3 The Verification That Must Happen Before Anything Is Trusted

This is the step Cymbal's original ad-hoc captures skipped, and the step that makes the difference between evidence and anecdote (§4.5, §5.5). The rig is not allowed to produce a finding until all six of these pass:

| Check | How Cymbal does it | Pass condition |
| --- | --- | --- |
| Capture-point drop | Read the capture tool's own counters at start and end (tcpdump's count report, `dumpcap -S`), the packet-socket counters where applicable, and `/proc/net/softnet_stat` deltas per CPU on the capture host | Counter deltas are zero across the reference window; non-zero triggers a re-run, not a footnote |
| Interface/device drop | Interface statistics on the capture NIC (among them the receive-dropped and receive-missed families) read before and after | No movement in the relevant counters |
| Kernel-path drop preceding the socket | Per-CPU softnet drop deltas | No movement during the baseline |
| Traffic-derived loss | Zeek's capture-loss analysis over the same window, reading `percent_lost` | Zero or near-zero with the small tolerance the Zeek documentation accepts for a healthy sensor ✅ |
| Timestamp source | Record which timestamp type the tool actually used, and whether a hardware timestamp was requested and granted; verify the NIC's timestamping capability is enabled at driver level rather than assumed | The design document states the source and the achievable accuracy, and no latency claim is made below it |
| Timestamp agreement between the two points | Compare both captures over a window of known traffic and measure the offset between them | A stable, measured offset; a drifting offset is an incident in its own right |

Cymbal also records, per capture file, the environment: tool and version, interface, filter expression as executed, ring/buffer settings, packet and byte counts, and the counter readings (§6.5). This is what turns the next dispute from an investigation into a lookup.

### 12.4 The Volume and Retention Arithmetic, With Labelled Illustrative Cost

Cymbal does the arithmetic (§7) rather than buying storage and hoping.

- **Continuous header-only capture at the demarcation.** Assumed 3,000 packets/s of relevant traffic on that link, snap length set to capture headers only, and an illustrative 120 bytes written per packet: 3,000 × 120 = 360,000 B/s = 0.36 MB/s = **31.1 GB/day**; a 30-day rolling window is **~933 GB**. (Unit chain as §7.2; all figures illustrative.)
- **On-demand full-payload capture, filtered to the specific FIX session and the payment endpoints.** At 3,000 packets/s and an illustrative 700 bytes per packet: 3,000 × 700 = 2,100,000 B/s = **2.1 MB/s** = **181.44 GB/day**; 30 days would be **~5.44 TB**, which is why this capture is *triggered and time-boxed to 72 hours* by default (≈**0.54 TB** per event) rather than left running.
- **The counterfactual.** Full-rate, full-payload capture of the link without a filter, at an illustrative 700 bytes and 30,000 packets/s on the busiest hours, lands in the tens of terabytes per month by the §7.2 chain. Cymbal declines it, and records the decline.
- **Cost, labelled illustrative.** Monthly cost = stored TB-months × the storage platform's blended unit rate. Presenting it this way is deliberate: the deliverable is the volume and the chain, and the unit rate is the finance function's parameter, not an engineering invention. **No unit rate is asserted in this guide because none was verified at source (§15).**

### 12.5 The Decryption Reality for This Traffic

Cymbal's cross-connect carries the payment exchange over TLS with mutual certificate authentication, terminated in a vendor-supplied gateway with the private key held in an HSM. On the evidence of §9, that means:

- There is **no key log to be had in production** — the gateway does not write one, and RFC 9850's own applicability statement says the mechanism "MUST NOT be used in a production system" ✅.
- The **private key is useless for TLS 1.3** even if Cymbal could extract it from the HSM, because the RSA-key route "does *not* work with TLS 1.3" and requires a non-(EC)DHE cipher suite ✅ (Wireshark wiki, TLS).
- Therefore **the design must not promise content**. The capture's value is metadata and behaviour: whether the session was established, whether acknowledgements crossed the demarcation, per-direction byte counts, retransmission and reset patterns, and where in time the stall began (§9.4).

Cymbal's design takes the honest route and documents it: the packet capture answers *whether and when*; the application's own audit trail answers *what*. Where Cymbal's own software is under its control — internal test environments — key logging is available under the specification's applicability constraints ✅, and the test cases are built there, with the key log handled separately from the traffic (§10.3).

### 12.6 Access Control, Authorisation and Retention Design

- **Authorisation.** A named network-security role authorises every capture, with a written scope; the standing header-only capture is authorised as a class, reviewed quarterly, with a named owner.
- **Access.** Capture output is granted per case, by name, with a defined expiry; the store is encrypted at rest and access is logged.
- **Minimisation.** The continuous capture is **header-only by default**; payload is captured only on a case-by-case basis and only for the filtered session. Payload capture of segments that cannot be in scope (personal-banking and HR segments, and any link carrying cardholder data outside the payments scope) is excluded by filter, not by policy statement.
- **Retention.** 30 days rolling for the continuous header capture; 72 hours by default for triggered payload captures, extended only on a documented case; legal hold as an explicit exception that moves the capture to an evidence store with a recorded exemption (§11.6).
- **Classification.** The capture store is classified as a data store that may hold payment and customer data, and therefore inherits the obligations attached to that data — cross-reference [MAS Regulations & Guidelines](../banking/mas_regulations_guidelines_guide.md), [Payment Rails](../banking/payment_rails_guide.md) and [Market Data Consumption](../banking/market_data_consumption_guide.md) for the frameworks, and [Cybersecurity](cybersecurity_guide.md) and [Security by Design](security_by_design_guide.md) for the control expectations.
- **Governance tension recorded.** The design document states the zero-trust tension in the words §11.5 recommends, and names [Zero Trust Network Architecture](zero_trust_network_architecture_guide.md) as the owner of the posture that the capture is an exception to.

### 12.7 The Recommendation — Including What Cymbal Decides NOT to Capture

**Cymbal builds:** one tap at the demarcation feeding one broker; a continuous header-only capture with instrumentation and a written certificate of completeness; a triggered, filtered, full-payload capture capability with a default 72-hour window; an independent capture at the gateway host for cross-checking; a rotating store with a catalogue written at capture time; Zeek's capture-loss analysis as a standing control; and a governance register that names the authoriser, the owner, the retention class and the hold mechanism.

**Cymbal explicitly decides not to capture:**

1. **Not full-rate, full-payload on the data-centre core links.** The arithmetic (§7.4, §12.4) does not justify it, and the analytical need is satisfied by flows plus targeted captures.
2. **Not the payload of segments that cannot be in scope** — personal-banking, HR, legal and unrelated customer segments are excluded by capture filter, because a store that cannot hold the data cannot leak it (§10.5).
3. **Not continuous payload on the counterparty link.** It would be unreadable anyway (§12.5) and would create a regulated store of counterparty traffic for no diagnostic gain.
4. **Not traffic the authorisation does not cover**, including internal red-team activity outside an authorised engagement — the discipline that [Penetration Testing Execution Standard](penetration_testing_execution_standard_guide.md) owns.
5. **Not an unmonitored capture on a production link.** No capture runs without its counters watched and recorded (§4.6). This is the single guardrail that makes every other item on this list meaningful.

### 12.8 The Lessons the Example Is Meant to Carry

1. The capture point is chosen for the **question** — who can prove the acknowledgement crossed the demarcation — not for the convenience of the switch.
2. **Verification precedes analysis.** A capture program that cannot state its own completeness is not evidence.
3. **The arithmetic drives the design**, and the design that survives is the narrow, instrumented, time-boxed one.
4. **Decryption reality shapes the promise.** Design for metadata value; treat content as an exception with a separate authority.
5. **What you decline to capture is part of the design**, and it should be written down next to what you capture.

---

## 13. The Anti-Patterns

Nine failure modes, each of which has a symptom that looks like success.

| # | Anti-pattern | Symptom | Root cause | Guardrail |
| --- | --- | --- | --- | --- |
| 1 | **The capture trusted without a loss check** | A clean, complete-looking trace becomes the basis of a confident conclusion | Nobody asked whether the rig was keeping up; the artefact conceals its own incompleteness (§4) | Counters recorded at start and end; traffic-derived loss analysis; a certificate of completeness attached to every capture that will be used for a decision |
| 2 | **The mirror port that silently drops and looks healthy** | Dashboard green, packets missing | The destination port and the internal path are finite; the switch may not report what it could not mirror (§2.3) | Mirror scope reduced to what is needed; an independent capture point for any evidence use; mirror statistics treated as a monitored signal, not an assumption |
| 3 | **The capture started with timestamps unverified** | A latency conclusion that no one can reproduce from another tool | Software timestamps at copy time; silent fallback from hardware timestamping; resolution loss on write (§5) | Record the timestamp source with the capture; state resolution and accuracy separately; refuse claims below the format's resolution; measure the offset to the application-log clock |
| 4 | **The full-rate capture nobody can afford to retain** | A storage escalation two quarters after go-live, or a silent 48-hour retention nobody documented | Capture designed from the link speed instead of the question (§7.4) | Start from filter, retention and question; compute the unit chain before purchase; keep full-rate for windows that justify it |
| 5 | **The verbose capture left running on a production link with no owner** | A multi-terabyte store nobody can explain, containing traffic nobody meant to record | Standing capture with no review date and no named owner (§10.2) | Every standing capture has an owner, a review date and a filter that implements its authorisation |
| 6 | **The archived capture with no index** | "We have the traffic" followed by three days of searching by hand | Filenames that carry no time; no catalogue; retention by size rather than by window (§6.5) | Time-partitioned filenames from day one; a catalogue row per file written at capture time; retention expressed in time |
| 7 | **The analyst who captures traffic they were not authorised to capture** | An internal incident, a disciplinary process, or a regulatory finding | The capability is trivially available and the guardrails are procedural rather than technical (§10.2) | Positive written scope; capture filters that implement the scope; role separation between who can mirror and who can read; a register |
| 8 | **The capture store holding regulated data under ordinary backup rules** | A retention or disclosure obligation discovered after the fact | The store was treated as engineering artefacts rather than as a data store (§10.5) | Classify the store; apply the obligations of the most sensitive traffic it can hold; bring backups and replicas into the retention design; exclude in-scope-prohibited segments by filter |
| 9 | **The undecryptable capture used to claim content-level absence** | "The packets show no such message was sent" — about a TLS 1.3 session | A design assumption that capture equals content (§9.3) | State the decryption reality in the design; diagnose from behaviour; use the application audit trail for content questions; never assert payload-level absence from a capture you cannot read |

The pattern across all nine: **the artefact looks fine, and the failure is in a property that was never specified.** Every guardrail above is a requirement statement, not a tool purchase — which is why the fixes are cheap and why they are so often skipped.

---

## 14. The Claims Audit

Status key: **Verified** = checked at the named source, with the source's own date supplied. **Verified (arithmetic)** = computed this pass from stated assumptions; illustrative, not measured. **Flagged** = asserted only with a caveat, or omitted from the body pending verification. **Rejected** = considered and deliberately excluded because no source supports it.

| Claim | Status | Source | Source date | Source quality |
| --- | --- | --- | --- | --- |
| tcpdump stable release 4.99.7 | Verified | tcpdump.org, "Latest Releases" (retrieved 2026-09-22) | 2026-09-21 | Project primary |
| libpcap stable release 1.10.7 | Verified | tcpdump.org, "Latest Releases" | 2026-09-05 | Project primary |
| libpcap 1.11.0 testing; removes remaining AirPcap, DPDK, TurboCap, Septel, SITA ACN support | Verified | tcpdump.org, "Latest Releases" | 2026-09-13 | Project primary |
| tcpdump end-of-run counts "captured", "received by filter", "dropped by kernel"; reports 0 if the OS does not report drops | Verified | tcpdump(1) man page (`tcpdump.1.in`, master) | Man page dated 31 July 2026 | Project primary |
| tcpdump `-B` buffer in KiB; SIGINFO/SIGUSR1 counts; SIGUSR2 flushes with `-w` | Verified | tcpdump(1) | 31 July 2026 | Project primary |
| tcpdump `-E spi@ipaddr algo:secret` ESP decryption, algorithms, and its own "debugging purposes" warning | Verified | tcpdump(1) | 31 July 2026 | Project primary |
| Linux packet socket requires `CAP_NET_RAW`; `PACKET_MR_PROMISC` = promiscuous mode | Verified | packet(7), man-pages 6.19 | man page 2026-02-08; repo tarball fetched 2026-09-09 | Upstream man page |
| `PACKET_RX_RING`, `TP_STATUS_KERNEL`/`TP_STATUS_USER`, `TPACKET_V1` default, `PACKET_VERSION` | Verified | packet(7) | as above | Upstream man page |
| `PACKET_STATISTICS` returns `struct tpacket_stats {tp_packets, tp_drops}`; reading resets counters; TPACKET_V3 structure differs | Verified | packet(7) | as above | Upstream man page |
| `PACKET_FANOUT` (since Linux 3.1) and its hash / load-balance / CPU / rollover / random / queue-mapping modes | Verified | packet(7) | as above | Upstream man page |
| `PACKET_MMAP` removes the per-packet syscall cost; ring is a configurable circular buffer; NAPI and IRQ pinning advice | Verified | kernel documentation, `networking/packet_mmap` | page retrieved 2026-09-22 (legacy text, 2.4/2.6/3.x wording) | Kernel primary |
| `PACKET_TIMESTAMP` default is a software timestamp "generated when the packet is copied into the ring"; `TP_STATUS_TS_RAW_HARDWARE`/`TP_STATUS_TS_SOFTWARE` | Verified | packet(7); kernel `networking/packet_mmap` | as above | Upstream primary |
| `pcap_stats()` fields `ps_recv`, `ps_drop`, `ps_ifdrop` and their documented caveats (unavailable on savefiles; `ps_drop` zero where unsupported; `ps_ifdrop` zero ambiguous) | Verified | tcpdump.org man page `pcap_stats(3PCAP)` | Page updated 5 March 2022, documenting 1.11.1-PRE-GIT | Project primary |
| dumpcap: default output format pcapng, `-F` for pcap, `-B` default 2 MiB with "try to increase this size" advice, `-S` statistics per interface each second, `-s` default snapshot length 262144, `-I` monitor mode 802.11-only with disassociation warning | Verified | dumpcap(1), Wireshark 4.6.8 | Retrieved 2026-09-22 | Project primary |
| Wireshark stable 4.6.8 and old-stable 4.4.18 released 12 August 2026 | Verified | wireshark.org/news | 2026-08-12 | Project primary |
| Wireshark gets timestamps from libpcap/Npcap/kernel; USB NICs give very bad timestamp accuracy; pcap supports two fixed resolutions by magic number; pcapng supports per-interface resolutions | Verified | Wireshark User's Guide §7.6 "Time Stamps" | Retrieved 2026-09-22 | Project primary |
| Live capture compression while writing; `tshark --compress` on live captures | Verified | Wireshark 4.6.8 release notes (issue 9311) | Retrieved 2026-09-22 | Project primary |
| Npcap is required for live capture on Windows and ships with Wireshark packages | Verified | wireshark.org/download | Retrieved 2026-09-22 | Project primary |
| Npcap's current version number | **Flagged** | Not captured this pass | — | Omitted from the body |
| Three Linux interface-statistics families; `ip -s -s link show` output labels; sysfs `/sys/class/net/<dev>/statistics/`; netlink RTM_GETLINK/RTM_GETSTATS with IFLA_STATS_LINK_64 as preferred | Verified | kernel documentation, `networking/statistics` | Retrieved 2026-09-22 | Kernel primary |
| `rx_dropped` vs `rx_missed_errors` definitions; procfs folds them; `rx_fifo_errors` as device buffer-space drops; `rx_errors` aggregation requirement | Verified | kernel `networking/statistics` | Retrieved 2026-09-22 | Kernel primary |
| `ethtool -S <ifc>` for driver stats; `--groups eth-phy eth-mac eth-ctrl rmon`; unmanaged-switch ports may count other hosts' events | Verified | kernel `networking/statistics` | Retrieved 2026-09-22 | Kernel primary |
| `/proc/net/softnet_stat` per-CPU processed and dropped dimensions, and alerts on `net.core.netdev_max_backlog` / `net.core.netdev_budget` | **Flagged** (secondary sources; not in the kernel's own Documentation tree as far as established this pass) | Netdata collector documentation for `/proc/net/softnet_stat` | Retrieved 2026-09-22 | Vendor secondary |
| Softnet backlog is where pre-socket drops occur at high rate | **Flagged** (mechanism plausible and widely documented, no primary source verified) | — | — | Stated as a caution, not as a sourced mechanism |
| Zeek `capture_loss.log` infers loss from TCP sequence-number gaps; fields `ts`, `ts_delta`, `peer`, `gaps`, `acks`, `percent_lost`; worked example where 0.4123 = 0.4123% | Verified | Zeek documentation, `logs/capture-loss-and-reporter.rst` (zeek-docs master) | Retrieved 2026-09-22 | Project primary |
| Zeek v9.0.0 released 21 August 2026; FreeBSD support sunset; Broker WebSocket server removed; `zeek-packet-source-udp` for VXLAN/GENEVE with `SO_REUSEPORT` | Verified | github.com/zeek/zeek releases | Retrieved 2026-09-22 | Project primary |
| Suricata 8.0.7 (15 Sep 2026) and 7.0.17 (7 Jul 2026) | Verified | github.com/OISF/suricata releases | Retrieved 2026-09-22 | Project primary |
| Suricata `stats.log` and EVE `stats` record enabled by default; interval configurable (8 s in the documented example); Suricata 8.0 sandboxes Lua rules by default | Verified | Suricata `suricata.yaml` documentation (project repo) | Retrieved 2026-09-22 | Project primary |
| Suricata capture-drop counter name (`capture.kernel_drops`) | **Rejected** — no source verified | Searches returned empty | — | Excluded from the body |
| Snort 3: snort3 3.12.2.0, libdaq 3.0.27, libml 2.0.0; rule advisory 2026-09-17; Snort 2.9.20 still published | Verified | snort.org/downloads | Retrieved 2026-09-22 (advisory 2026-09-17) | Vendor primary |
| DPDK 26.07.0 (2026-07-22) and LTS 25.11.3 (2026-08-20); additional LTS lines (24.11.7, 2026-08-11) | Verified | core.dpdk.org/download | Retrieved 2026-09-22 | Project primary |
| PF_RING tag 9.4.0 (2026-08-29); repository commit 2026-09-21; vendor pps figures quoted **as vendor claims** | Verified (as vendor claim) | github.com/ntop/PF_RING (tags + repo page) | Retrieved 2026-09-22 | Project primary |
| netmap: commits 2026-08-08, 2026-07-10, 2026-05-26, 2026-04-18; no GitHub releases; shipped in FreeBSD since 10.x | Verified | github.com/luigirizzo/netmap (commits, README) | Retrieved 2026-09-22 | Project primary |
| AF_XDP: XSK, UMEM, FILL/COMPLETION/RX/TX rings, `XDP_UMEM_REG`, `XDP_SHARED_UMEM`, `XSKMAP`, XDP_SKB vs XDP_DRV modes, copy in copy mode, queue-binding caveat with `ethtool -L`/`-N` remedies | Verified | kernel documentation, `networking/af_xdp` | Retrieved 2026-09-22 (docs rendered for 7.3.0-rc4) | Kernel primary |
| Timestamping: `SO_TIMESTAMPING` flags (`SOF_TIMESTAMPING_RX_HARDWARE`, `_RX_SOFTWARE`, `_TX_*`); `SCM_TIMESTAMPING` with ts[0] software / ts[2] hardware / ts[1] deprecated → PHC + linuxptp; cmsg per-`sendmsg` requests; `SOF_TIMESTAMPING_OPT_RX_FILTER` | Verified | kernel documentation, `networking/timestamping` | Retrieved 2026-09-22 | Kernel primary |
| Hardware timestamping configuration via `ETHTOOL_MSG_TSCONFIG_SET`/`GET` or legacy `SIOCSHWTSTAMP`; `struct hwtstamp_config` (`tx_type`, `rx_filter`); admin rights required; ERANGE on failure; `HWTSTAMP_TX_OFF/ON`, `HWTSTAMP_FILTER_*` | Verified | kernel `networking/timestamping` §3 | Retrieved 2026-09-22 | Kernel primary |
| PTP hardware clock infrastructure in Linux; IEEE 1588; one character device per registered clock usable as a POSIX clock id; supported-hardware list (e.g. ConnectX UTC counter) | Verified | kernel documentation, `driver-api/ptp` | Retrieved 2026-09-22 | Kernel primary |
| `/dev/ptpN` device-node naming | **Flagged** — the kernel page verifies a character device, not the node name | — | — | Stated as uncertain in §5.3 |
| `ethtool -T` output field names (e.g. a "PTP Hardware Clock" line) | **Flagged** — two searches returned empty (tool/backend limitation) | — | — | Excluded from the body |
| NTP specification reference and the NTP↔PTP relationship details | **Flagged** — not verified this pass | — | — | Excluded from the body; the sourced linuxptp/PHC relation is given instead |
| Classic pcap: 24-octet File Header (Magic, Major/Minor version, Reserved1/2, SnapLen, LinkType); magic `0xA1B2C3D4` = µs and `0xA1B23C4D` = ns; fields written in the writing machine's byte order | Verified | draft-ietf-opsawg-pcap-04 | Published 4 August 2024 | IETF Internet-Draft (not an RFC) |
| Current status/later revision of the pcap specification | **Flagged** — the ‑04 revision expired 5 February 2025; a later revision or RFC could not be confirmed from this environment | — | — | Flagged in §6.2 |
| pcapng: block envelope with duplicated Block Total Length for backward navigation; SHB mandatory at `0x0A0D0D0A`; IDB with per-section Interface ID; EPB fields (Interface ID, Timestamp High/Low, Captured Packet Length, Original Packet Length, Packet Data); SPB without Interface ID; ISB for per-interface statistics including drops; NRB; Custom Block; obsolete Packet Block's Drops Count field; `if_tsresol`/`if_tsoffset` referenced by the ISB text | Verified | draft-ietf-opsawg-pcapng-06 | Published 19 September 2026; expires 23 March 2027 | IETF Internet-Draft (Informational; current) |
| RFC 9850 "The SSLKEYLOGFILE Format for TLS": Informational, December 2025; three-value line format; `client_random` 32 bytes / 64 hex characters; TLS 1.3 traffic-secret and ECH labels; UTF-8, no BOM; CRLF/CR/LF accepted; **applicability statement "MUST NOT be used in a production system"** and conditional-compilation advice | Verified | RFC 9850 (tlswg.org rendering; rfc-editor number) | December 2025 | IETF RFC (Informational) |
| Wireshark TLS decryption: three methods; key log from `SSLKEYLOGFILE` written by NSS/OpenSSL/boringssl; RSA-key limitations (no (EC)DHE, not TLS 1.3, server certificate only, no resumption); PSK decrypts past and future traffic; embedded decryption secrets in pcapng | Verified | Wireshark wiki, "TLS" | Retrieved 2026-09-22 | Project primary |
| AWS Traffic Mirroring: copies traffic from an elastic network interface of type `interface`; Source/Filter/Target/Session; filters and packet truncation; all Regions; supported instance families; billed hourly per session plus data transfer; charged until sessions deleted | Verified | AWS documentation, "What is Traffic Mirroring?" | Retrieved 2026-09-22 | Vendor primary |
| Azure Network Watcher packet capture: `AzureNetworkWatcherExtension` required; 5-tuple filters; max bytes per packet (34 = IPv4 header only); default 18,000 s time limit; continuous capture **in PREVIEW** with ring-buffer files (default 10, max 10,000 files; default 100 MB, max 4 GB per file; max 7 days); 10,000 parallel sessions per region per subscription; SAS tokens with storage key access required | Verified | Microsoft Learn, "Packet capture overview" | Page last updated 24 February 2026 | Vendor primary |
| Google Cloud Packet Mirroring: clones instance traffic incl. payloads; configurable ingress/egress; mirroring happens on VMs and consumes VM bandwidth; collector destination is an instance group behind an internal passthrough load balancer; policy priority always 1000; all IPv4 by default; charged by data processed | Verified | Google Cloud VPC documentation, "Packet Mirroring" | Page last updated 2026-09-18 | Vendor primary |
| Cisco SPAN: copies traffic from specified source ports or VLANs to a destination port; sometimes called port mirroring/port monitoring; documented performance impact at platform level | Verified (definition) | Cisco Network Management Configuration Guide, IOS XE 17.17.x, Catalyst 9200, "Configuring SPAN and RSPAN" | Page updated 31 March 2025 | Vendor primary |
| Statement that capturing from many busy ports can saturate a line card's fabric connection | **Flagged** (⚠-structural) | Cisco community knowledge-base article "Performance issues occur due to SPAN or RSPAN configuration" | Undated on retrieval | Vendor community secondary |
| IPFIX is RFC 7011 / STD 77, September 2013, obsoleting RFC 5101; specifies Metering/Exporting Process Reliability Statistics options templates | Verified | rfc-editor.org RFC 7011 | September 2013 | IETF RFC (Standards Track / STD) |
| NetFlow v9 is RFC 3954, October 2004, Informational, "not a candidate for any level of Internet Standard"; includes a disclosure-of-flow-information security consideration | Verified | rfc-editor.org RFC 3954 | October 2004 | IETF RFC (Informational) |
| sFlow version 5, July 2004; sampling-rate definition (1 sample per N packets observed); random-number requirements; replaces RFC 3176 | Verified | sflow.org, `sflow_version_5.txt` | July 2004 | Specification body (sFlow.org) |
| Volume arithmetic: 700 bytes × 50,000 packets/s = 35 MB/s → 126 GB/h → 3.024 TB/day → 90.72 TB per 30 days → ~1.10 PB/year | Verified (arithmetic) | Computed this pass from the stated assumptions | 2026-09-22 | **Illustrative** — not a measurement |
| Link-speed storage table (1/10/100 Gbit/s → 10.8/108/1,080 TB per day) | Verified (arithmetic) | Computed this pass | 2026-09-22 | **Illustrative**; note it assumes perfect utilisation and excludes framing overhead |
| Metadata arithmetic: 2,000 flows/s × 200 bytes = 34.56 GB/day (≈1.04 TB/30 days); 1:1,000 sampled ≈ 1.04 GB/30 days | Verified (arithmetic) | Computed this pass | 2026-09-22 | **Illustrative** |
| Cymbal Bank illustrative volumes (31.1 GB/day header-only; 181.44 GB/day full payload; ≈0.54 TB per 72-hour event) | Verified (arithmetic) | Computed this pass from the scenario's stated assumptions | 2026-09-22 | **Fictional scenario, illustrative** |
| A specific compression ratio for capture files | **Rejected** — no source verified | — | — | Deliberately not asserted (§6.6) |
| A specific percentage of traffic that is encrypted | **Rejected** — no source verified | — | — | Deliberately not asserted (§9.1) |
| A universal packet-per-second threshold at which the default capture path fails | **Rejected** — no source states one | — | — | §3.4 states the transition is undocumented and non-uniform |
| Specific legal instruments for unauthorised capture (statutes, regulations) | **Flagged** — no legal source verified this pass | — | — | §10.2 states the governance requirement and routes the framework to the repo's banking guides |
| PCI DSS / cardholder-data obligations as applied to a capture store | **Flagged** — stated as a consequence and cross-referenced by name to repo guides, not re-derived | — | — | §10.5 |

---

## 15. What Could Not Be Verified

Listed honestly, because a guide that hides its gaps is a guide that manufactures false confidence. Each item is either excluded from the body or explicitly caveated there.

**Tool limitations encountered this pass**

1. **`ethtool -T` output field names.** Two web searches for the exact output format returned empty results (a search-backend limitation, not evidence of absence). The kernel's own hardware-timestamping configuration interface (`ETHTOOL_MSG_TSCONFIG_SET`/`GET`, `SIOCSHWTSTAMP`, `hwtstamp_config`) **is** verified and is what §5.3 relies on; the *command-line tool output* a reader would see is not asserted.
2. **`ethtool` man page** was not retrieved, so the `--groups` behaviour is cited from the kernel's statistics documentation rather than from the tool's own documentation.
3. **Suricata's capture-drop counter names.** The project's documentation site failed to fetch on two attempts and a search returned empty. Suricata's statistics mechanism (`stats.log` and EVE `stats` records, enabled by default) **is** verified; the specific counter names for capture-path drops are **not**, and are therefore absent from §4.2.
4. **zeek.org/docs hosted pages** failed to fetch (the documentation source in the project's own repository was used instead, which is the same content but a different URL).
5. **The Datatracker HTML pages** for the pcap/pcapng drafts failed to fetch; the IETF's `archive/id/` renderings were used instead and are the same documents.
6. **tcpdump's GitHub releases page is empty** by design (the project releases tarballs on tcpdump.org) — a search that starts at GitHub will conclude, wrongly, that the tool has not released in years. Recorded because it is exactly the kind of false status signal §8.6 warns about.

**Claims left unverified or excluded**

7. **`/dev/ptpN` naming.** The kernel documentation verifies that a character device is created per registered PTP clock and that it can be used as a POSIX clock id; it does not, on the page retrieved, give the device-node name. The conventional `/dev/ptpN` is therefore flagged in §5.3 rather than asserted.
8. **The NTP specification and the precise NTP↔PTP interaction.** No NTP standard was verified this pass, so §5.4 gives only the sourced relation (the kernel's suggestion to synchronise system time from the NIC's PTP clock via a userspace stack such as linuxptp) and declines to describe the standards' relationship further.
9. **`/proc/net/softnet_stat` column semantics as primary fact.** The file is widely documented in vendor and community documentation and is used operationally, but the kernel's own Documentation tree did not yield a specification of the columns in this pass. Its dimensions are cited from Netdata's collector documentation and marked secondary.
10. **The softnet-backlog drop mechanism as a citable primary fact.** The mechanism is consistent with the kernel's NAPI/backlog documentation and with operational practice, but no primary source verified this pass states that backlog overflow is a drop location preceding the capture socket. §4.1 lists it as a location with a caution rather than a citation.
11. **A later revision of the classic pcap specification, or an RFC number for it.** The ‑04 revision (4 August 2024) carries an expiry of 5 February 2025; a later revision could not be confirmed. The field-level facts in §6.2 are correct for the deployed format.
12. **Npcap's current version number.** Wireshark's download page states that Windows packages ship "the latest stable release of Npcap" without naming it, and no Npcap release page was retrieved.
13. **Fabric-saturation thresholds for switch mirroring.** Cisco's documentation set includes a platform-level performance-impact discussion, and its community material states that capturing from many busy ports "can fill up the fabric connection" on older line cards; **no numeric threshold was verified**, and §2.3 therefore asserts the mechanism and not a figure.
14. **Any compression ratio for capture files.** No ratio verified → none asserted (§6.6).
15. **Any figure for the share of traffic that is encrypted.** No figure verified → none asserted (§9.1).
16. **Any throughput or drop-rate figure for the capture path or for any bypass framework.** The vendor's own packets-per-second figures for PF_RING are quoted **as vendor claims with attribution**, and no independent measurement was verified. This guide states no universal pps threshold (see §3.4).
17. **Specific legal instruments** (statutes, data-protection acts, sectoral rules) governing unauthorised packet capture, and their interaction with employee monitoring. No legal source was verified this pass; §10.2 states the governance requirement and cross-references the repository's regulatory guides by name.
18. **PCI DSS requirements as applied to a capture store.** The consequence in §10.5 is stated as a governance consequence and cross-referenced to the repo's MAS/PDPA, payment-rails and market-data guides rather than re-derived from the standard.
19. **Cloud-provider mirroring behaviour that is not in the providers' own documentation** — e.g. whether a specific provider's mirror drops under load, how it handles encapsulated traffic, or what its internal aggregation limits are. §2.6 states only what the providers document.
20. **Windows/macOS capture-path specifics** beyond the documented requirement for Npcap. The kernel-path detail in §3 is Linux, and the guide says so.

---

## 16. The Glossary, the Cross-References and the Closing Summary

### 16.1 Glossary

| Term | Meaning as used in this guide |
| --- | --- |
| **AF_PACKET** | The Linux packet-socket address family used to receive raw frames; requires `CAP_NET_RAW` in the governing user namespace |
| **AF_XDP** | A Linux address family for high-performance packet processing, redirecting frames from an XDP program to a userspace socket (XSK) backed by a registered UMEM |
| **BPF** | Berkeley Packet Filter; the instruction set capture filters compile to, and (as eBPF) the in-kernel programmable path used by XDP |
| **Broker (packet broker)** | An appliance that aggregates mirrored streams, filters and de-duplicates them, and fans them out to multiple analysis tools |
| **Bypass framework** | A kernel-bypass or zero-copy capture framework (AF_XDP, DPDK, PF_RING, netmap) that moves packets to userspace with fewer copies and fewer kernel locks, at the cost of owning a NIC, a queue or a driver |
| **Capture filter** | A filter applied before the packet is copied; determines what enters the file, and is therefore a data-loss decision |
| **Capture point** | The physical or logical place where a copy of the traffic is taken |
| **Capture register / catalogue** | The record of what was captured, by whom, under what authority, with what settings, and with what counter readings |
| **Display filter** | A filter applied after the fact to data already captured; reversible, and therefore not a data-loss decision |
| **Drop** | A packet that traversed the capture point but is not in the file, because the device, the kernel path, the socket ring, the userspace buffer or the write path could not keep up |
| **EPB / SHB / IDB / ISB / SPB / NRB / CB** | pcapng block types: Enhanced Packet Block, Section Header Block, Interface Description Block, Interface Statistics Block, Simple Packet Block, Name Resolution Block, Custom Block |
| **Flow record** | A summary of a flow (five-tuple, counts, timestamps, interfaces) exported instead of the packets themselves; standardised as IPFIX (RFC 7011), NetFlow v9 (RFC 3954) and sFlow |
| **Hardware timestamping** | Timestamping by the NIC/PHY at the point of transmission or reception, requiring NIC capability, driver enablement by an administrator, an application request, and a disciplined clock |
| **Header-only capture** | A capture in which SnapLen is set small enough that only protocol headers are stored — the cheapest data-minimisation control available |
| **Interface ID** | In pcapng, the per-section integer identifying which interface a block's packets or statistics refer to |
| **Loss (capture loss)** | See *Drop*; distinct from loss on the wire, which the endpoints observe independently |
| **Npcap** | The Windows capture driver required by Wireshark for live capture |
| **pcap (classic / libpcap format)** | The original capture file format: a 24-octet file header then a flat sequence of packet records, with one interface, one link type and one timestamp resolution |
| **pcapng** | The extensible block-structured capture file format; supports multiple interfaces, per-interface timestamp resolution, statistics blocks, comments and vendor extensions |
| **PHC (PTP hardware clock)** | The clock inside a NIC/PHY exposed by Linux as a character device usable as a POSIX clock id |
| **Promiscuous mode** | A NIC mode in which frames not addressed to the host are also delivered (`PACKET_MR_PROMISC` on Linux) |
| **Monitor mode** | An IEEE 802.11-only mode in which the radio delivers whole 802.11 frames; may disassociate the adapter |
| **PTP (IEEE 1588)** | The precision time protocol lineage whose hardware-clock infrastructure Linux exposes; pairs with hardware timestamping |
| **SnapLen** | Snapshot length: the maximum number of bytes stored per packet, declared in the pcap file header or the pcapng Interface Description Block |
| **SPAN / port mirroring** | A switch feature that copies selected traffic to a designated port; Cisco's name for it is Switched Port Analyzer |
| **Tap** | A physical device inserted in a link that copies traffic to a monitoring output |
| **TRUNCATION** | Storage of fewer bytes per packet than were on the wire; visible in pcapng as Captured Packet Length < Original Packet Length |
| **XSK / UMEM** | In AF_XDP, the socket and the registered userspace memory region it receives frames into |
| **Zero-copy** | A capture path in which the packet is not copied into a kernel ring before reaching userspace; a mode and a driver-capability question, not a property of an interface |

### 16.2 Cross-References

**Governance and security disciplines (sibling guides, this folder)**

- [Zero Trust Network Architecture](zero_trust_network_architecture_guide.md) — owns the posture that a capture is a governed exception to (§1.4, §11.5)
- [Cybersecurity](cybersecurity_guide.md), [Security by Design](security_by_design_guide.md), [Beyond Zero: Enterprise Security](beyond_zero_enterprise_security_guide.md) — owns the security-monitoring and control-programme disciplines (§10.2, §11.4)
- [Penetration Testing Execution Standard](penetration_testing_execution_standard_guide.md) — owns the authorisation under which test traffic may be captured (§10.2, §11.4, §12.7)

**Container capability model (sibling guides, this folder)**

- [OpenShift SCC Comprehensive Guide](openshift_scc_comprehensive_guide.md), [OpenShift SCC Service Account Guide](openshift_scc_service_account_guide.md), [Secure Red Hat OpenShift](secure_red_hat_openshift_guide.md) — own `NET_RAW`/`NET_ADMIN` and the SCC model that governs in-cluster capture (§1.4, §2.5)

**Facilities, cabling and placement (sibling guides, this folder)**

- [L2 Design for Financial Data Centres](l2_design_financial_data_centers_guide.md), [Data Center](data_center_guide.md), [Singapore Data Centres](singapore_data_centres_guide.md) — own where a tap sits and where the capture store lives (§2.2, §6.7, §12.2)

**The use cases this guide serves (`../banking/` and sibling guides)**

- [FIX Protocol](../banking/fix_protocol_guide.md) — session and sequencing faults diagnosed from the wire (§11.1)
- [SWIFT Alliance Access](../banking/swift_alliance_access_guide.md), [SWIFTNet FileAct](../banking/swiftnet_fileact_guide.md) — connectivity and file-transfer evidence (§11.1)
- [Axway Transfer CFT](axway_transfer_cft_guide.md) — managed file transfer stalls and retries (§11.1)
- [IBM MQ Disaster Recovery](ibm_mq_disaster_recovery_guide.md) — channel-level behaviour over TLS (§11.1)
- [Payment Rails](../banking/payment_rails_guide.md) — payment latency attribution and payment-data handling (§11.1, §10.5)
- [Market Data Consumption](../banking/market_data_consumption_guide.md) — feed gaps, latency and licensing (§11.1, §10.5)
- [MAS Regulations & Guidelines](../banking/mas_regulations_guidelines_guide.md) — MAS/PDPA obligations that reach into a capture store (§10.2, §10.5, §12.6)

**A false friend, noted for readers searching the repository**

- `technology/professional_networking_guide.md` — a **career and social networking guide** (relationship-building, personal brand, a 30-day plan). It has nothing to do with computer networking. Recorded here so that no reader mistakes it for an adjacent technical guide (§1.4).

### 16.3 The Closing Summary

Packet capture is the last resort of the honest engineer — the evidence that survives the failure of everything that was supposed to record it. Its power is that it outranks the logs. Its hazard is the same fact viewed from the other side: because it looks complete, it is believed, and a capture that is quietly incomplete will be believed just as firmly as a complete one.

Everything in this guide reduces to five disciplines:

1. **Choose the capture point for the question, and pay its cost knowingly.** No point sees everything, and every point changes something — a switch's mirror, a tap in the path, a VM's bandwidth, a container's capabilities.
2. **Instrument the capture, not just the capture's output.** Counters, recorded; loss inferred from the traffic itself; completeness stated rather than assumed. An unmonitored capture is an unverified one, and an unverified capture is a liability with a filename.
3. **Verify the timestamps, and separate resolution from accuracy.** A capture and a log will eventually disagree, and the capture is not automatically the one that is right.
4. **Compute before you buy, and design for the retention you can defend.** Narrow filters, short windows, measured completeness; full rate only where a specific question requires it.
5. **Govern the exception.** A capture is a deliberate visibility exception to a zero-trust, encrypted, segmented estate; it contains credentials and personal data; and for a bank, the store that holds it inherits the obligations of the data it holds. Who authorised it, who may hold it, how long it lives, how a legal hold overrides that, and how it is destroyed — those are the questions that decide whether a capture programme is an asset or a liability.

And one thing to remember above all the arithmetic: when a capture and an application log disagree, the naive instinct is to trust the capture, because a packet is a fact. It is a fact — about the observation point, at the observation time, with the observation's completeness. Know the point, the clock and the counters, or you are not reading evidence at all.

the wire does not lie.
