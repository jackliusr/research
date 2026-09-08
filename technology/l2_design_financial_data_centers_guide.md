# L2 Design in Financial Data Centers: The Connected Floor — A Comprehensive Guide

**Jack Liu Shurui, Solution Architect**

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank
> **Context:** Technology / Capital-Markets Infrastructure — the network-infrastructure deep-dive of the repository's capital-markets cluster: the Layer-2 design of the financial data centre — the mission (latency, determinism, market-data fanout, HA/DR), the leaf-spine/CLOS topology practice, the L2 technology stack from VLANs and the spanning-tree family through VXLAN and EVPN (verified against RFC 7348, RFC 7432, RFC 8365 and the IEEE 802.1 standards), multicast for market data and PTP timing (condensed from the trading and market-data siblings, never re-derived), the low-latency L2 controversies (cut-through, micro-buffers, lossless Ethernet), exchange connectivity and colocation, the stretched-L2 vs L3-exit HA debate, the compliance-capture layer, and a Cymbal Bank worked example — a full L2 design review for a trading floor
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** RFC Editor pages for [RFC 7348](https://www.rfc-editor.org/rfc/rfc7348.html) (VXLAN — Informational, August 2014), [RFC 7432](https://www.rfc-editor.org/rfc/rfc7432.html) (BGP MPLS-Based EVPN — Proposed Standard, February 2015, updates per the RFC Editor: RFC 8584/9161/9572/9573/9746), [RFC 8365](https://www.rfc-editor.org/info/rfc8365) (EVPN NVO — Proposed Standard, March 2018), [RFC 7761](https://www.rfc-editor.org/info/rfc7761) (PIM-SM, STD 83, obsoletes RFC 4601), [RFC 3376](https://www.rfc-editor.org/info/rfc3376) (IGMPv3), [RFC 4607](https://www.rfc-editor.org/info/rfc4607) (SSM), [RFC 4604](https://www.rfc-editor.org/info/rfc4604) (IGMPv3/MLDv2 for SSM) — all extracted directly this pass; IEEE 802.1Q/802.1 standards history (Wikipedia as secondary, cross-checked against the 802.1 working-group record); IEEE 1588 editions (2002/2008/2019) and 802.1AS gPTP (Wikipedia as secondary); cut-through switching and Ethernet flow control mechanics (Wikipedia, citing Cisco and IEEE 802.3x/802.1Qbb); in-repo verified facts credited to their sibling guides (SGX Reach and colocation — market-making sibling §7.2; multicast venue feeds and tick capture — market-data sibling §5.4; the low-latency stack — trading sibling §9). Where a source could not be reached this pass (EUR-Lex, ESMA, datatracker.ietf.org, the web-search backend all returned errors/empty), the claim is flagged ⚠ and listed in §13.4. No RFC number, standard edition, date or statistic was invented.
> **Last Updated:** September 2026

*A network-infrastructure deep-dive on how the Layer-2 fabric of a financial data centre is actually designed: the mission that shapes it (microsecond latency, deterministic forwarding, multicast market-data fanout, HA/DR), the leaf-spine/CLOS topology that replaced the spanning-tree core, the L2 technology stack — VLANs (IEEE 802.1Q) and the STP family (802.1D/802.1w/802.1s) and why they were displaced, VXLAN (RFC 7348) and EVPN (RFC 7432, RFC 8365) and the BGP control-plane fabric, multicast for market data and PTP/IEEE 1588 timing (both condensed from the sibling guides), the low-latency L2 controversies (cut-through vs store-and-forward, micro-buffers, the lossless-Ethernet debate), exchange connectivity and colocation economics, the stretched-L2 vs L3-exit HA debate, the compliance tap/capture layer, and a full Cymbal Bank worked example — an L2 design review for a trading floor, ending at the fabric that binds every desk to the market.*

**Cross-references (repository convention — `banking/` siblings are `../banking/…`, `technology/` siblings are plain filenames):** [Trading-System Software Architecture](trading_system_software_architecture_guide.md) (the order/execution stack this fabric carries — **cross-ref, condense**) · [Market Data Consumption](../banking/market_data_consumption_guide.md) (the market-data value chain — §5.4 multicast distribution and co-location — **cross-ref, do not re-derive**) · [Market Making in Singapore](../banking/market_making_singapore_guide.md) (§7.2 SGX Reach and the SGX data centre — the colocation context — **cross-ref, condense**) · [FIX Protocol: The Trade Conversation](../banking/fix_protocol_guide.md) (§8 the binary/multicast venue protocols — **cross-ref**) · [Capital Markets Architecture](../banking/capital_markets_architecture_guide.md) (the umbrella) · [Enterprise Risk Management](../banking/enterprise_risk_management_guide.md) (the op-resilience overlay for §10) · [DDS Guide](dds_guide.md) (the market-data bus that rides this fabric) · [Zero Downtime System Design](zero_downtime_system_design_guide.md) (session estates and HA discipline — §10) · [Low-Latency C/C++ Development](low_latency_cpp_development_guide.md) (the endpoint engineering §8 assumes) · [Zero Trust Network Architecture](zero_trust_network_architecture_guide.md) (the security overlay on the same fabric).

**How to read this guide.** §1 is the overview — the one-paragraph answer and the key-facts table. §2 is the mission — why a financial data centre's network is a different animal from an enterprise or cloud fabric: latency, determinism, market-data fanout, HA/DR. §3 is the topology — leaf-spine/CLOS, ECMP and oversubscription practice. §4 is classic L2 — VLANs (IEEE 802.1Q) and the spanning-tree family (802.1D STP, 802.1w RSTP, 802.1s MSTP), verified against the standards, with their limits stated plainly. §5 is modern L2 — VXLAN (RFC 7348), EVPN (RFC 7432, RFC 8365), the MP-BGP EVPN control plane and the VXLAN-EVPN combination that displaced STP-based fabrics. §6 is multicast for market data — IGMP, PIM-SM, SSM and rendezvous points — condensed from the market-data and trading siblings, with only the fabric-design angle derived here. §7 is timing — PTP/IEEE 1588-2008 and 1588-2019 on the L2 fabric, boundary clocks, transparent clocks, grandmasters, and the regulatory clock context (MiFID II clock-sync rules flagged ⚠ at §13.4) — condensed from the trading sibling. §8 is low-latency L2 — cut-through vs store-and-forward, micro-buffers and microbursts, lossless Ethernet (PFC 802.1Qbb, ECN 802.1Qau) and the controversy over whether lossless L2 belongs in a trading path. §9 is exchange connectivity and the colocation edge — condensed from the market-making sibling. §10 is the HA/DR debate — stretched L2 vs the routed L3 exit. §11 is compliance and recording — taps, SPAN/ERSPAN and packet capture, cross-referenced to the surveillance content in the trading sibling. §12 is the Cymbal Bank worked example — an L2 design review of a trading floor: fabric sizing, multicast design, PTP deployment, HA path. §13 is the claims status (✅/⚠/❌) with §13.4 "What Could Not Be Verified". §14 is the glossary, the cross-references and the closing summary. **Integrity convention:** ✅ = verified this pass against a primary or cited source (source named in §13); ⚠ = flagged (vendor-internal, marketing, single-source, or not re-verified this pass); ⚠-structural = industry-standard practice widely documented but not attributable to a single primary source verified this pass; ❌ = disputed claims found in the wild and rejected. No RFC number, standard edition, date or statistic was invented.

---

## Table of Contents

1. [The Overview](#1-the-overview)
2. [The Financial-Data-Centre Network Mission](#2-the-financial-data-centre-network-mission)
3. [The Topology — Leaf-Spine and the CLOS Fabric](#3-the-topology--leaf-spine-and-the-clos-fabric)
4. [Classic L2 — VLANs and the Spanning-Tree Family](#4-classic-l2--vlans-and-the-spanning-tree-family)
5. [Modern L2 — VXLAN, EVPN and the Control-Plane Fabric](#5-modern-l2--vxlan-evpn-and-the-control-plane-fabric)
6. [Multicast for Market Data](#6-multicast-for-market-data)
7. [Timing on the Fabric — PTP and the Regulatory Clock](#7-timing-on-the-fabric--ptp-and-the-regulatory-clock)
8. [Low-Latency L2 — Cut-Through, Buffers and the Lossless Debate](#8-low-latency-l2--cut-through-buffers-and-the-lossless-debate)
9. [Exchange Connectivity and the Colocation Edge](#9-exchange-connectivity-and-the-colocation-edge)
10. [HA/DR — Stretched L2 vs the Routed L3 Exit](#10-hadr--stretched-l2-vs-the-routed-l3-exit)
11. [Compliance and Recording — Taps, SPAN and Packet Capture](#11-compliance-and-recording--taps-span-and-packet-capture)
12. [The Cymbal Bank Worked Example — an L2 Design Review](#12-the-cymbal-bank-worked-example--an-l2-design-review)
13. [The Claims Status — Verified, Flagged, Rejected](#13-the-claims-status--verified-flagged-rejected)
14. [Glossary, Cross-References and the Closing Summary](#14-glossary-cross-references-and-the-closing-summary)

---

## 1. The Overview

### 1.1 The Short Answer

A financial data centre's network exists to do one thing the generic enterprise or cloud fabric is never asked to do: **move price information and orders between a firm and the markets with the least, most predictable delay — and never, ever lose the record of what moved.** Everything else — capacity, cost, manageability, security — is negotiated against that mission. The Layer-2 (L2) design is where the negotiation is won or lost, because L2 is the layer that carries the *raw Ethernet frames*: the exchange's binary multicast feed as it lands on a colocated server's NIC, the FIX session between an order gateway and the venue, the PTP packets that discipline every timestamp, and the east-west traffic between a feed handler and the algos that consume it.

The industry's answer, converged over a decade and a half, is a fabric with three layers of technology:

- **A physical topology with no blocked links** — leaf-spine (a two-stage CLOS network) in which every leaf switch connects to every spine switch, so traffic can use *all* links via ECMP instead of disabling half of them to keep a spanning tree loop-free (§3).
- **An overlay for the L2 services that still need them** — VXLAN (RFC 7348) encapsulation terminated at VTEPs, with the MAC-learning and loop-prevention control plane supplied by BGP EVPN (RFC 7432, RFC 8365) rather than by flooding and spanning tree (§5). Classic VLANs and spanning tree survive in the access layer and in small legacy domains, but no modern low-latency or HA fabric is built on STP.
- **A deterministic timing and multicast plane** — PTP/IEEE 1588 for sub-microsecond clock distribution (§7), and IGMP/PIM-managed multicast groups carrying market data to exactly the servers that asked for it (§6).

The worked example (§12) shows all three layers designed together for one trading floor — this guide's convention is that a floor is not a collection of VLANs but *a single fabric binding every desk to the market*: the connected floor.

### 1.2 The Key-Facts Table

| Aspect | Fact | Status |
| --- | --- | --- |
| The mission | Latency, determinism, market-data fanout, HA/DR — the network is a revenue path, not plumbing (§2) | ✅/⚠-structural §2 |
| The topology | Leaf-spine / two-stage CLOS; ECMP over all spines; oversubscription ratios 3:1–4:1 typical in financial estates, 1:1 for the hot paths | ✅-structural / ⚠ ratios §3 |
| VLANs | IEEE 802.1Q: 32-bit tag (TPID 0x8100; 3-bit PCP, 1-bit DEI, 12-bit VID); 12-bit VID ⇒ 4,094 usable VLANs | ✅ §4 (802.1Q) |
| STP family | 802.1D STP (now consolidated into the 802.1Q bridging standard with 802.1D-2004), 802.1w RSTP, 802.1s MSTP — loop prevention by blocking links; convergence of tens of seconds (STP) down to ~seconds (RSTP/MSTP) | ✅ standards; ⚠-structural timings §4 |
| VXLAN | RFC 7348 (Informational, Aug 2014): MAC-in-UDP, 24-bit VNI, 8-byte VXLAN header, IANA UDP port 4789, VTEP = tunnel endpoint | ✅ §5 (rfc-editor.org) |
| EVPN | RFC 7432 (Proposed Standard, Feb 2015): BGP MPLS-based EVPN; MP-BGP EVPN address family; ESI, split-horizon, aliasing, DF election; updated by RFC 8584/9161/9572/9573/9746 | ✅ §5 (rfc-editor.org) |
| EVPN over VXLAN | RFC 8365 (Proposed Standard, Mar 2018): EVPN as the NVO control plane for VXLAN/NVGRE/MPLS-over-GRE/GENEVE | ✅ §5 (rfc-editor.org) |
| Multicast | IGMPv3 RFC 3376 (Oct 2002; RFC Editor now lists it obsolete, replaced by RFC 9776), PIM-SM RFC 7761 (STD 83), SSM RFC 4607 (232/8), RFC 4604; exchange market data is binary multicast (CME MDP 3.0-style) | ✅ §6 (rfc-editor.org + in-repo) |
| Timing | IEEE 1588-2002 → 1588-2008 (PTPv2) → 1588-2019 (PTPv2.1); UDP 319/320; boundary/transparent clocks; 802.1AS gPTP | ✅ §7 (secondary, cross-checked) |
| Lossless L2 | PFC = IEEE 802.1Qbb (per-CoS pause); ECN-style congestion notification = 802.1Qau; pause frames from 802.3x (1997) | ✅ designations §8 |
| Lossless for trading | Controversial: PFC head-of-line blocking and congestion-spreading risks vs microburst loss; practice favours pacing at the app layer | ⚠/❌ §8 |
| Colocation | Metres of fibre at ≈5 ns/m decide the venue-side budget; SGX Reach sub-90 µs engine, co-location at the SGX data centre | ✅ in-repo §9 |

---

## 2. The Financial-Data-Centre Network Mission

### 2.1 What the Network Is For

Four properties distinguish the financial data centre's L2 estate from any other. Each one is a *design constraint*, and each shows up repeatedly in the sections that follow:

- **Latency — the network is on the critical path of a revenue process.** In electronic markets, an order that reacts a microsecond later can be an order that does not fill, or fills at a worse price. The framing is documented across the industry's low-latency literature: venue connectivity is measured in single-digit microseconds, tick-to-quote budgets in the low microseconds, and the colocation decision exists precisely to remove the milliseconds of WAN distance (cross-ref the [trading guide](trading_system_software_architecture_guide.md) §7/§9 and the [market-making sibling](../banking/market_making_singapore_guide.md) §7.3 for the full latency-budget analysis — this guide owns the *fabric* side of that budget). The L2 consequence: switches must forward in cut-through, buffers must be sized to absorb bursts without adding store-and-forward delay, and every encapsulation or filtering hop on the hot path is a tax (§3, §8).
- **Determinism — latency is only useful if it is predictable.** A trading system can engineer around a constant delay; it cannot engineer around jitter. This drives the preference for fixed forwarding paths, for pacing rather than loss + retransmit, for PTP-disciplined clocks (§7), and for the avoidance of anything that makes forwarding contingent — queue build-up, head-of-line blocking, control-plane events mid-session, STP re-convergence (the historical nightmare: a topology change that blocks the trading path for tens of seconds, §4).
- **Market-data fanout — one source, many consumers, at line rate.** An exchange feed (CME MDP 3.0-style binary multicast, Nasdaq multicast ITCH, the SGX feeds) is published *once* and must reach every consuming server simultaneously and identically — which is exactly what IP multicast on an L2 fabric does (§6). The alternative — unicast copies per consumer — multiplies bandwidth and, worse, multiplies the fanout latency differences between consumers. The market-data value chain (venue → feed → vendor → platform → decision) is owned by the [market-data sibling](../banking/market_data_consumption_guide.md); §6 here owns the multicast plumbing underneath it.
- **HA/DR — the floor must survive a switch, a rack, a room.** Trading hours are not the moment for a fabric to be "mostly" available. This drives dual-homed servers, N+1 spine designs, and the most contested architectural decision in this guide: whether to stretch L2 across data centres at all (§10).

⚠-structural: the four-part mission framing is this guide's synthesis from the cited industry literature (Cisco/Arista low-latency data-centre material, the low-latency trading stack literature cross-referenced in the trading sibling, and the colocation economics of the market-making sibling); the individual parts are well documented, the exact four-way framing is this guide's structure, not a quotation from any single source.

### 2.2 The L2 Estate on One Diagram

```text
  EXCHANGE / VENUE (colocation cage in the same building or metro)
        │  binary multicast market data (UDP) + order sessions (FIX/native, TCP)
        ▼
  ┌──────────────────── meet-me room / exchange cross-connect ───────────────────┐
  │  FW / EDGE ROUTERS ──(L3: eBGP / static / extranet VPN)                       │
  └───────────────────────────────────┬──────────────────────────────────────────┘
                                      ▼
  ┌────────────────────  LEAF-SPINE L2/L3 FABRIC  ────────────────────┐
  │  spine  ──  spine  ──  spine  ──  spine  (L3 ECMP underlay)        │
  │    │  ╲   │   ╲  │    ╲  │      ╲   │                              │
  │  leaf ── leaf ── leaf ── leaf ── leaf  (ToR / services)            │
  │   │         │        │        │       │                            │
  │  MD/order  feed     algo     colo-     PTP/ops                     │
  │  gateways handlers  hosts    servers   capture                      │
  └───────────────────────────────────┬──────────────────────────────────┘
                                      ▼
                MPLS/DWDM ──► DR site (routed L3 exit, §10)
```

The picture to hold: **the underlay is L3** (every leaf and spine has an IP address, ECMP across spines), **the overlay is L2-where-needed** (VXLAN-EVPN for the VLAN services that survive: legacy appliances, clustering, some HA patterns), and **the true L2 domains are small, local and deliberately bounded** — the access ports to a server, the multicast domain inside a leaf's broadcast domain, the PTP domain.

### 2.3 Where L2 Still Matters

For a senior architect the useful mental model is not "L2 vs L3" but **"broadcast domain vs routed domain."** L2 survives in a financial estate exactly where a broadcast domain is wanted:

1. **Server access** — a host's NIC talks Ethernet to its leaf; VLANs partition the port.
2. **Multicast** — market-data groups are L2-adjacent to their receivers at the leaf, with IGMP snooping pruning the groups (§6).
3. **Timing** — PTP's best performance comes from an L2 (or L3 multicast) domain with boundary clocks at every hop (§7).
4. **Legacy clustering and appliance semantics** — some HA/clustering software, load balancers in transparent mode, and older appliances assume L2 adjacency or gratuitous-ARP behaviour; these are the *raison d'être* of the VXLAN-EVPN overlay (§5, §10).
5. **The compliance capture layer** — SPAN/tap delivery is an L2 construct (§11).

Everything else should be routed. The rest of this guide verifies the technologies that make each of those five survivable without letting a spanning tree anywhere near the trading path.

### 2.4 The Non-Goals — What the Fabric Must Not Do

Equally important as the mission is what a disciplined design refuses:

1. **Do not make the trading path conditional on a control plane.** No STP reconvergence, no routing-protocol flap, no VTEP state churn on the hot path — control-plane events belong off the data path (§4, §5).
2. **Do not add losslessness to hide a burst problem** — fix the burst at the source (pacing, §8.4); PFC on a trading path trades a rare drop for a guaranteed head-of-line stall (§8.3).
3. **Do not stretch what can reconnect.** Order sessions and market-data streams have recovery machinery designed for exactly this; stretching L2 to avoid reconnecting is how failure domains grow (§10).
4. **Do not let security or compliance machinery sit inline on the HFT path** — filter and record passively (taps), not by adding hops (§9.3, §11).
5. **Do not confuse the overlay with the fabric** — the VXLAN-EVPN services layer is for the workloads that *need* L2 semantics; the moment an overlay VLAN starts carrying the hot path is the moment to re-architect (§5.4).

⚠-structural: these five are engineering-judgement rules distilled from the verified sections, presented as this guide's design doctrine rather than as quotable external claims.

---

## 3. The Topology — Leaf-Spine and the CLOS Fabric

### 3.1 Why CLOS Won

The defining topology decision of the modern data centre is **leaf-spine: a two-stage Clos network** in which every lower-stage switch ("leaf", the top-of-rack or end-of-row device servers plug into) connects to every upper-stage switch ("spine"), and no leaf ever connects directly to another leaf. Traffic between any two leaves crosses exactly one spine — **every flow crosses exactly two switches and at most one spine hop**, which makes latency both low and *uniform* (the east-west traffic pattern of a trading estate — feed handler to algo to gateway — is the reason the topology exists).

Two structural properties do the work:

- **No blocked links.** A spanning-tree network (or a traditional core/distribution/access tree) must disable links to stay loop-free, wasting half its bandwidth and forcing traffic up and down a tree. In leaf-spine, *all* links carry traffic. The motivation is documented in the primary source for this guide's overlay section — RFC 7348's problem statement explicitly names spanning tree's disabled links among the limitations the overlay era was built to escape (§5). ✅ (RFC 7348 §1, extracted this pass)
- **ECMP scale-out.** With the underlay running L3 (each leaf and spine is an IP router), traffic between any leaf pair is hashed across the parallel spine uplinks by equal-cost multipath (ECMP). Adding capacity = adding spines and (re)wiring each leaf to them. There is no spanning tree to reconverge and no active/standby waste.

⚠-structural: leaf-spine as *the* data-centre topology and ECMP as its forwarding engine are universal industry practice, documented across every major vendor's data-centre design literature (Cisco, Arista, Juniper, NVIDIA) — but the specific market share / "industry standard" phrasing is vendor-ecosystem consensus rather than one neutral primary source, so the strongest claims here are carried ⚠-structural rather than ✅. The RFC 7348 motivation is the ✅ anchor.

### 3.2 Oversubscription — Sizing the Fabric

Oversubscription is the ratio of *server-facing* bandwidth to *uplink* bandwidth at a leaf. The design vocabulary, with the ⚠ that every number is practice, not standard:

| Ratio | Meaning | Where used in a financial estate |
| --- | --- | --- |
| 1:1 (no oversubscription) | Uplinks = server ports in aggregate bandwidth | The hot path: market-data and order leaves where a microburst must never contend (§8) |
| 3:1 to 4:1 | Typical general-purpose leaf | Corporate/office VLANs, dev/test, back-office — traffic that tolerates contention |
| Higher (10:1+) | Bulk, non-real-time | Never on a trading path |

⚠-structural: specific ratio norms (3:1, 4:1) are commonly cited industry practice but vary by vendor and workload; there is no IEEE/ISO number to verify. The *direction* — hot paths at or near 1:1, everything else oversubscribed — is this guide's synthesis of that practice, not a quotable source.

For a trading floor the practical rule, exercised in §12, is: **size the multicast and order leaves at 1:1 to the fanout rate, size the general leaves at 2:1–3:1, and never oversubscribe the spine-to-leaf direction of a leaf that carries exchange-facing traffic.**

### 3.3 Spine Count and Failure Domain

- Minimum production spine count is **two** (a leaf survives one spine failure); the worked example in §12 uses four.
- A leaf with N spine uplinks loses 1/N of its capacity per spine failure — with N=4, a single spine loss costs 25% of every leaf's uplink capacity, which is why hot leaves are sized with headroom (§12).
- The **failure domain** of a leaf-spine fabric is small and local: a failed leaf takes down only its own servers' *network* path (servers remain up); a failed spine degrades all leaves equally. Contrast with the spanning-tree era, where a topology change could block an entire core (§4).

✅-structural: spine redundancy math is elementary (N uplinks ⇒ 1/N capacity loss) — stated as derived arithmetic, not as a vendor claim.

### 3.4 The Underlay/Overlay Split

Because the underlay is L3 and ECMP-hashed, the fabric *as a whole* behaves like one giant distributed switch for the overlays that run on it: VXLAN tunnels (VTEP-to-VTEP, §5) ride the ECMP underlay invisibly, and each tunnel's traffic gets the same deterministic two-hop treatment as any other flow. This is the key architectural sentence of the modern design: **L3 gives you the multipath and the failure handling; L2 gives you the services; the overlay decouples the two so each can evolve alone** (RFC 7368-style "L3 everywhere" is the cloud-era underlay norm — cross-ref the [cloud-regions sibling](cloud_providers_guide.md) for the hyperscaler version of the same idea).

### 3.5 The Physical-Layer Notes — Optics, Cables and Ports

A fabric-design guide would be incomplete without the physical layer the L2 runs on, and a financial estate has specific rules of thumb (all ⚠-structural practice unless marked):

- **Speed tiers on the floor:** 25 G to the trading servers (feed handlers and gateways at 25/100 G), 100 G or 400 G on the spine links. ⚠-structural — speed tiers are procurement reality, not standard.
- **Fibre vs copper:** access links inside a rack are often copper (DAC) for cost and power; anything longer is fibre. Latency per metre is the same physics (§9.2) — the choice is density, reach and reliability, not speed-of-light. ⚠-structural.
- **Link loss budgets:** each mated pair and splice costs a fraction of a dB; a "short" cross-connect that traverses three patch panels can eat more loss budget than a clean 100 m run. The meet-me-room rule from §9.2 applies to optics too: count the connections, not just the metres. ⚠-structural.
- **Forward-error correction (FEC)** on 100 G+ links adds a small fixed latency and can add jitter under correction; the low-latency estate should know which links have FEC enabled and why, because "enable FEC by default" is a cloud-era default that a trading floor may consciously override on short clean links. ⚠ (FEC latency trade-offs are vendor- and link-length-dependent; this pass verified no specific numbers).

---

## 4. Classic L2 — VLANs and the Spanning-Tree Family

### 4.1 VLANs — IEEE 802.1Q

A VLAN is a broadcast domain carved out of a physical switched network by tagging. The standard is **IEEE 802.1Q**, which defines a 32-bit tag inserted between the source MAC address and the EtherType/length field of an Ethernet frame ✅ (IEEE 802.1Q; tag structure verified via the 802.1Q record — Wikipedia as secondary):

| Field | Width | Meaning |
| --- | --- | --- |
| TPID | 16 bits | Tag protocol identifier, value **0x8100** — marks the frame as 802.1Q-tagged |
| PCP | 3 bits | Priority code point — the 802.1p class-of-service (8 levels) that PFC and QoS policies key off (§8) |
| DEI | 1 bit | Drop-eligible indicator (formerly CFI) |
| VID | 12 bits | VLAN identifier — 4,096 values, 0 and 4095 reserved ⇒ **4,094 usable VLANs** ✅ (derived from the 12-bit VID) |

The 4,094-VLAN ceiling and the single-tag broadcast-domain model are precisely the scaling limits RFC 7348's problem statement calls out (thousands of tenants/segments needed; VLAN count inadequate; MAC-table sizes at the ToR inadequate) ✅ (RFC 7348 §1, §3.1–§3.3, extracted this pass). Note that 802.1Q also extended the maximum frame size from 1,518 to 1,522 bytes to carry the tag ✅ (802.1Q frame format).

### 4.2 The Spanning-Tree Family — 802.1D, 802.1w, 802.1s

Bridged Ethernet needs loop prevention: a redundant topology with a broadcast frame circulating is a broadcast storm. The classic answer is the Spanning Tree Protocol — one logical tree of forwarding ports, all other links blocked:

| Protocol | Standard | What it adds | Status this pass |
| --- | --- | --- | --- |
| STP | IEEE 802.1D (originally 1990; the 802.1D-2004 revision of MAC bridges was incorporated into the 802.1Q-2014 consolidation) | The original: one spanning tree per bridged network; blocked ports for redundancy; convergence via timers (max-age 20 s, forward delay 15 s defaults ⇒ reconvergence on the order of tens of seconds) | ✅ designations and consolidation (802.1Q-2014 incorporated 802.1D-2004 — verified); ⚠-structural timer/convergence behaviour |
| RSTP | IEEE 802.1w (2001) | Rapid Spanning Tree: explicit proposal/agreement handshakes on point-to-point links collapse convergence to well under the timer regime; still one tree, still blocks ports | ✅ designation; ⚠-structural "sub-second on point-to-point" behaviour |
| MSTP | IEEE 802.1s (2002) | Multiple Spanning Trees: map VLANs to *instances*, each with its own tree, so different VLANs can use different paths; the last mainstream STP evolution | ✅ designation |

All three now live in the consolidated **802.1Q bridging standard** (802.1Q-2014 and its successors absorbed 802.1D-2004's bridging and spanning-tree content — the STP family is no longer a set of separate 802.1D/802.1w/802.1s documents) ✅ (802.1Q-2014 consolidation, verified via the 802.1 record).

### 4.3 Why the STP Family Lost the Data Centre

The limits, stated plainly:

1. **Blocked links are wasted money.** Half the redundancy you paid for is administratively dead. In a low-latency estate you also cannot *afford* the failover path being cold.
2. **Convergence is a control-plane event on the data path.** STP reconvergence of tens of seconds (or even RSTP's seconds) is a trading-session killer; the deterministic mission of §2 forbids topology recomputation mid-session.
3. **One active tree per instance underutilises the mesh** and makes latency path-dependent (traffic between two leaves may hairpin through the root).
4. **The 4,094-VLAN ceiling and flat L2 scale limits** (§4.1) cap multi-tenant and multi-service growth.

⚠-structural: the *judgement* that the STP family is unfit for modern data-centre fabrics is industry consensus (every major vendor's fabric architecture since ~2013 is STP-free in the core), but it is consensus-with-a-commercial-interest, so it is flagged rather than ✅. The factual anchors — 802.1Q's incorporation of 802.1D-2004, the amendment designations — are ✅. RFC 7348's own problem statement (written by Arista/Cisco/VMware/Broadcom authors) documents the disabled-links and VLAN-scale limitations from the standards side ✅.

### 4.4 Where Spanning Tree Still Belongs

STP-family protocols remain appropriate at the *edge* of a financial estate — the small switched domain of an office floor, a legacy access switch pair, management networks — where the failure modes are acceptable and the cost of replacement is not. The design rule carried into §12: **STP is allowed only where no trading or market-data path crosses it, and RSTP (not classic STP) is the minimum there.**

---

## 5. Modern L2 — VXLAN, EVPN and the Control-Plane Fabric

### 5.1 VXLAN — RFC 7348 (Verified)

**Virtual eXtensible Local Area Network (VXLAN)** is the encapsulation that carries Ethernet frames over the L3 underlay. Verified against the primary source this pass (RFC 7348, extracted from rfc-editor.org):

| Fact | Verified content |
| --- | --- |
| Document status | RFC 7348, *Virtual eXtensible Local Area Network (VXLAN): A Framework for Overlaying Virtualized Layer 2 Networks over Layer 3 Networks* — **Informational** (not standards track; documents the deployed protocol), **August 2014**, Mahalingam et al. (Storvisor, Cumulus, Arista, Broadcom, Cisco, VMware, Intel, Red Hat) |
| What it is | MAC-in-UDP encapsulation: the inner Ethernet frame is carried inside an outer IP/UDP packet between **VTEPs** |
| VTEP | VXLAN Tunnel End Point — "an entity that originates and/or terminates VXLAN tunnels" (RFC 7348 §1.1, quoted) |
| VNI | VXLAN Network Identifier — a **24-bit** segment ID (16M segments vs 4,094 VLANs); the VNI identifies the scope of the inner MAC frame; isolation between tenants is by VNI |
| Frame format (§5) | Outer Ethernet header → outer IP header → outer UDP header → **VXLAN header (8 bytes: 8 flag bits, 24-bit VNI, reserved bits that MUST be zero on transmission)** → inner Ethernet frame. Parsed "from the bottom": above the outer FCS sits the inner MAC frame with its own Ethernet header |
| UDP port | **IANA-assigned destination port 4789** (RFC 7348 §5) |
| Learning model | Data-plane learning: unicast via MAC learning; BUM (broadcast/unknown-unicast/multicast) via IP multicast groups in the underlay or head-end replication — RFC 7348's own model, later superseded for control by EVPN (§5.3) |
| Problem statement | Explicitly: STP's disabled links; the 4,094-VLAN ceiling; ToR MAC-table scale; multi-tenant isolation; L2 scale across DCs |

✅ all of the above (RFC 7348, extracted this pass).

### 5.2 EVPN — RFC 7432 (Verified)

Where VXLAN supplies the *data plane*, **Ethernet VPN (EVPN)** supplies the *control plane*: MAC reachability learned and distributed by BGP instead of by flooding. Verified against the primary source this pass:

| Fact | Verified content |
| --- | --- |
| Document status | RFC 7432, *BGP MPLS-Based Ethernet VPN* — **Standards Track, Proposed Standard**, **February 2015**, Sajassi (ed., Cisco), Aggarwal, Bitar, Isaac (Bloomberg), Uttaro, Drake (Juniper), Henderickx (Alcatel-Lucent) |
| Current status / evolution | Per the RFC Editor info page this pass: **updated by RFC 8584, RFC 9161, RFC 9572, RFC 9573, RFC 9746** — the EVPN RFC family continues to evolve; RFC 7432 itself remains Proposed Standard |
| Control plane | **MP-BGP with a new address family**: EVPN routes (NLRI) carry MAC/IP reachability — the BGP control plane replaces both data-plane MAC learning and the flooding-based Ethernet emulation of VPLS |
| EVPN route types | Ethernet Auto-Discovery route; MAC/IP Advertisement route; Inclusive Multicast Ethernet Tag route; Ethernet Segment route; (plus the extended-community machinery: ESI label, ES-import route target, MAC mobility, default gateway) — verified from the RFC 7432 table of contents |
| Multihoming / ESI | An **Ethernet Segment** is the set of links attaching a customer site to two or more PEs; the **Ethernet Segment Identifier (ESI)** names it. EVPN's multihoming functions (RFC 7432 §8) deliver: per-segment auto-discovery, fast convergence (mass withdrawal via Ethernet A-D routes), **split-horizon** filtering (ESI labels / ingress replication), **aliasing and backup path**, and **Designated Forwarder (DF) election** for BUM traffic |
| Why it displaced STP fabrics | The combination — VXLAN data plane + EVPN control plane — gives a loop-free, multipath, all-links-active L2 service with fast failover, no blocked ports, no STP anywhere in the fabric, and MAC mobility (the MAC Mobility extended community) |

✅ all of the above (RFC 7432 + RFC Editor info page, extracted this pass).

### 5.3 The Combination: EVPN as the NVO Control Plane — RFC 8365 (Verified)

RFC 7432 was written for MPLS transport. The overlay data centre needed the same control plane over VXLAN/NVGRE/GENEVE. That is **RFC 8365, *A Network Virtualization Overlay Solution Using Ethernet VPN (EVPN)* — Standards Track, Proposed Standard, March 2018**, Sajassi (ed., Cisco) and Drake (ed., Juniper) et al. ✅ (RFC Editor info page, extracted this pass). It specifies how EVPN serves as the NVO control plane, analyses VXLAN, NVGRE and MPLS-over-GRE encapsulations (with GENEVE noted as needing separate work), and — critically for this guide — specifies **new multihoming procedures for split-horizon filtering and mass withdrawal, and EVPN route constructions for VXLAN/NVGRE**, i.e., the anycast-VTEP / multi-homed-NVE behaviour a dual-homed trading server relies on. Per the RFC Editor, RFC 8365 is itself updated by RFC 9746.

So the modern financial fabric's L2 story in one paragraph: **VXLAN (RFC 7348) carries the frames; EVPN (RFC 7432 as extended by RFC 8365) carries the MAC routes over MP-BGP; the spanning tree is gone; every uplink carries traffic; a dual-homed server's failover is a BGP-convergence event in the EVPN control plane, not an STP topology change.** ⚠-structural: the *"displaced STP"* historical claim is industry-consensus framing (see §4.3); the RFC facts above are ✅.

### 5.4 What the Overlay Costs a Trading Path

Honest engineering requires stating the taxes the overlay imposes on a low-latency path:

- **50 bytes of encapsulation overhead** for IPv4 transport (14 outer Ethernet + 20 outer IP + 8 outer UDP + 8 VXLAN) — derived from the RFC 7348 frame format ✅-derived; hence 9,000-byte jumbo MTUs on the underlay so a 1,500-byte inner frame survives with room to spare ⚠-structural (MTU engineering practice).
- **VTEP hop latency** — the encapsulation/decapsulation point is usually the leaf (or the host NIC/vSwitch), adding a small per-packet cost. On the *trading hot path* the industry preference is to **avoid the overlay entirely**: the order gateway's session and the feed handler's multicast are delivered on plain L2/L3 access at the leaf, and the VXLAN-EVPN overlay carries the *services* layer (legacy appliances, clustering, DR mobility, management) — the "routed fast path, overlaid slow path" split exercised in §12. ⚠-structural: the split itself is practice; per-device VTEP latency figures are vendor-internal and not cited here.

### 5.5 The Control-Plane Alternative for the Overlay: Notes

For completeness: VXLAN per RFC 7348 originally contemplated multicast- or head-end-replication-based BUM handling with data-plane learning; some early fabrics ran "VXLAN without EVPN" (flood-and-learn over an anycast VTEP). The industry outcome — EVPN as the control plane — is what the vendor ecosystem standardised on, and RFC 8365 documents it ✅. A fabric *without* a real control plane for the overlay (pure flood-and-learn VXLAN) does not get the split-horizon, aliasing and fast-failover properties of §5.2, so it is not an HA design; treat "VXLAN-only" vendor talk as a ⚠ simplification where HA L2 is claimed.

---

## 6. Multicast for Market Data

> **Division of labour with the siblings:** the [market-data guide](../banking/market_data_consumption_guide.md) owns the market-data value chain — feeds, tapes, vendors, tick capture — and its §5.4 covers multicast distribution and co-location consumption; the [FIX sibling](../banking/fix_protocol_guide.md) §8 owns the binary multicast venue protocols (CME MDP 3.0-style designs, sequence numbers, gap recovery); the [trading guide](trading_system_software_architecture_guide.md) §5/§7 frames the latency tiers. **This section condenses only what an L2 fabric designer must know: the protocols, the groups, the snooping, the RP placement.** Do not re-derive the sibling content.

### 6.1 Why Multicast

An exchange feed is one stream to N consumers. **IP multicast** delivers one copy per link, replicated only where receivers exist — the only distribution method that keeps per-consumer latency equal (every receiver sees the same packet at the same fabric distance) and bandwidth independent of consumer count ✅-structural (multicast's defining property; the venue-feed practice is documented in the market-data sibling §5.4). The host-side protocol machinery (IGMP membership, gap detection and snapshot recovery) is owned by the market-data sibling; here is the L2/fabric layer:

| Layer | Protocol | Role on the fabric | Verified |
| --- | --- | --- | --- |
| Host membership | **IGMP** (v2 RFC 2236; **v3 RFC 3376**, Oct 2002 — adds source filtering, i.e., "only this source"; the RFC Editor now lists RFC 3376 as obsolete, replaced by **RFC 9776**; updated by RFC 4604) | Receivers tell the first-hop switch/router which groups they want | ✅ rfc-editor.org |
| L2 pruning | **IGMP snooping** | The leaf watches IGMP reports and forwards multicast only to ports with interested receivers — without it, multicast would flood the broadcast domain (RFC 4541 documents snooping switch considerations) | ✅-structural practice; RFC 4541 designation |
| Routing protocol | **PIM-SM** — Protocol Independent Multicast, Sparse Mode — **RFC 7761, Internet Standard (STD 83), March 2016, obsoletes RFC 4601** | Builds a shared tree rooted at a **Rendezvous Point (RP)** per group, then optionally shortest-path trees per source | ✅ rfc-editor.org |
| Source-specific | **SSM** — RFC 4607 (Aug 2006): **232/8** (232.0.0.0/8) designated for SSM; host signalling via **RFC 4604** (IGMPv3/MLDv2 for SSM) | (S,G) joins only — no shared tree, no RP needed, no cross-source interference | ✅ rfc-editor.org |

### 6.2 The Fabric-Design Consequences

1. **IGMP snooping must be on, everywhere a market-data group flows** — a leaf without snooping treats the group as broadcast and floods every access port, which is both a security leak (a server that never subscribed receives the tape) and a load problem. ✅-structural.
2. **Snooping needs the querier and the RP reachable.** In a pure L2 access domain the leaf must run the IGMP querier role when no router is upstream; in the leaf-spine L3 design the leaf is the first-hop router and owns the membership state ✅-structural (standard switch behaviour; per-vendor CLI is ⚠ vendor-internal).
3. **RP placement for PIM-SM** — when market data is re-originated *inside* the firm (ticker plant → internal multicast distribution, e.g. the DDS-bus estates the [DDS sibling](dds_guide.md) describes), the RP should sit on the distribution spine/service leaf, and anycast-RP (multiple RPs sharing one IP) removes the RP as a single point of failure. ⚠-structural: anycast-RP practice is standard multicast engineering; the specific "financial firms run internal multicast for ticker-plant fanout" framing is cross-referenced from the DDS/market-data siblings rather than re-verified here.
4. **Source-specific multicast is the clean pattern for internal re-distribution** (SSM: receivers join (S,G) — the ticker plant is the single source): no RP, no shared tree, no (*,G) state, and a misconfigured second source cannot inject into the group (RFC 4607's design intent ✅). Many venue *external* feeds still use ASM-style groups from the exchange's own infrastructure, which is the venue's choice — the firm consumes whatever (S,G)/(*,G) the exchange publishes ✅-in-repo (venue feed practice, market-data sibling §5.4).
5. **Multicast and the lossless debate intersect here** (§8): UDP multicast is lossy by design; the recovery path is sequence-gap detection plus snapshot/refresh services, *not* L2 retransmission — which is precisely why lossless-Ethernet lobbying has not taken over the market-data path.

### 6.3 The Worked Numbers (Cross-Referenced, Condensed)

- Venue feeds of the CME MDP 3.0 family publish to multicast groups with per-instrument or per-book segmentation and sequence numbers per group ✅-in-repo (market-data sibling §4.3/§5.4 naming MDP 3.0-style binary multicast; FIX sibling §8 owns the mechanics).
- A colocated consumer joins the exchange's multicast groups **directly at the colocation leaf** — no routing, no firewalling, minimal hops; this is why the colo rack's leaf is the most latency-critical switch in the estate (§9, §12).
- Internal fanout from feed handler to algos is typically one group per instrument set, joined by every algo host, with IGMP snooping at each leaf confining the stream to interested racks ✅-in-repo practice (DDS sibling §10's Cymbal worked example is the bus-level twin of this design).

### 6.4 The Group-Design Quick Rules

The fabric designer's multicast checklist, distilled from §6.1–§6.3:

1. **One group per logical stream** — per instrument set or per feed, never one group for "all market data"; group membership is the only pruning the fabric has, so the granularity *is* the security and the load control.
2. **Sources are few and known.** SSM makes the source explicit in the join ((S,G)); the group range and the permitted sources should be in the change-control records, because a new source on an ASM group is invisible until it transmits.
3. **Snooping is a default, not a feature** — every leaf on a market-data path has IGMP snooping on, and the *absence* of a querier is a monitored condition, not a surprise.
4. **Capture members are silent** — the recording farm joins groups but must never answer queries or become the querier (§11.2, §12.2.6).
5. **Gap recovery is the contract, not the network** — the fabric may drop under a true overload; every consumer's feed handler must detect and recover (sequence numbers + snapshot), which the market-data/FIX siblings own. A fabric that "never drops" is a fabric that has added latency to hide a design flaw (§8.3).

⚠-structural: the checklist is this guide's condensation of the verified multicast material into operating rules.

---

## 7. Timing on the Fabric — PTP and the Regulatory Clock

> **Division of labour with the siblings:** the [trading guide](trading_system_software_architecture_guide.md) §9 frames hardware timestamping in the low-latency stack (its claims ledger records PTP/IEEE 1588 NIC timestamping as in-repo verified); the [market-data sibling](../banking/market_data_consumption_guide.md) covers timestamp quality in tick capture; the regulatory overlay (MiFID II RTS 6 algorithmic-trading controls, MAS conduct rules) lives in the banking siblings. **This section condenses the fabric side: PTP editions, clock types, deployment on L2, and why a bank's floor runs PTP at all.** Do not re-derive the siblings.

### 7.1 The Standard — IEEE 1588 Editions (Verified)

| Edition | What it is | Verified |
| --- | --- | --- |
| IEEE 1588-2002 | The first Precision Time Protocol (PTP v1), published 2002 | ✅ (IEEE 1588 history — Wikipedia secondary) |
| IEEE 1588-2008 | **PTP Version 2** — the revision that made PTP practical: new message types and clock types, *not backward compatible* with 2002; carries the profile concept (parameter sets for telecom, power, finance…) | ✅ |
| IEEE 1588-2019 | The third edition, published November 2019 — backward-compatible improvements to 2008 (colloquially **PTPv2.1**): security, better management, unicast negotiation refinements | ✅ |
| IEEE 802.1AS | The AVB/TSN adaptation of PTP — **gPTP** — for bridged networks; the L2-profile cousin used in time-sensitive Ethernet | ✅ |

PTP runs over UDP ports **319 (events) and 320 (general)** ✅ (PTP transport). The financial relevance of the 2008→2019 jump: 1588-2019 adds the integrity/security machinery that a *regulated, auditable* clock chain wants, and the industry's PTP deployments for trading floors are overwhelmingly PTPv2-lineage with hardware timestamping at the NIC — the trading sibling's verified stack records the endpoint practice ✅-in-repo.

### 7.2 The Clock Types and How Time Flows

The PTP model, condensed to fabric-design essentials:

- **Grandmaster (GM)** — the domain's time source, disciplined by GNSS (and/or atomic holdover). There should be *two* (primary + backup) with the Best Master Clock Algorithm (BMCA) arbitrating; a financial floor typically runs a primary GM per data centre with GNSS, plus a backup. ⚠-structural (redundant-GM practice; BMCA is standard PTP ✅).
- **Boundary clock (BC)** — a switch that terminates PTP on every port and re-originates it: it is a slave on its upstream port and a master on downstream ports, so *per-hop delay is measured and removed*. Every switch on the timing path should be a BC — this is the L2-fabric design rule.
- **Transparent clock (TC)** — a switch that forwards PTP but corrects the residence time (and link delay) in the message. End-to-end transparent clocks are simpler (no per-port master/slave state) but do not isolate the domain the way BCs do.
- **Ordinary clock (OC)** — the endpoint (the trading server), disciplined by the protocol; with **hardware timestamping** in the NIC the achievable sync is sub-microsecond; software-only timestamping is an order of magnitude worse and is unacceptable on a trading path. ✅-in-repo (trading sibling §9: "PTP/IEEE 1588 NIC timestamps for true path measurement").

⚠-structural: the specific nanosecond/microsecond accuracy claims per clock type and the BC-vs-TC accuracy comparisons are vendor-measured and network-dependent — the *mechanisms* (BC terminates and re-originates; TC corrects in transit) are standard definitions ✅; the accuracy numbers are practice, flagged, and the trading sibling's verified claims ledger is the reference for the endpoint stack.

### 7.3 Deploying PTP on the Trading Floor's L2

The fabric design rules, condensed:

1. **Run PTP in its own domain, over multicast or unicast, on the management/HA VLAN — never fight the market-data path.** PTP sync traffic is small and periodic; it must not queue behind a feed burst, so it gets a dedicated priority class (802.1p PCP) — and if PFC is in use anywhere (§8), PTP's class must be pause-protected *first*. ⚠-structural (QoS practice).
2. **Every hop is a boundary clock.** The leaf-spine path (server → leaf → spine → leaf → peer) should be all-BC switches so each link's delay is measured, not estimated. This is the single biggest accuracy lever after NIC hardware timestamping. ⚠-structural (industry deployment practice — Cisco/Arista/Juniper PTP deployment guidance is uniform here, but vendor-documented rather than neutral-source-verified).
3. **Colocation changes the clock topology.** In the exchange data centre the firm cannot install its own GNSS antenna — the *exchange* publishes time, and the colocated rack synchronises to the venue's PTP domain (or to the firm's GM reachable over the metro), whichever the venue's colocation rules permit. ⚠ (per-venue colocation timing rules vary and were not re-verified per venue this pass).
4. **Timestamp everything at the NIC, traceably** — the endpoint practice belongs to the trading/market-data siblings; the fabric obligation is to deliver time of the required *quality* to every port that needs it (which is every port with a feed handler or order gateway behind it).

### 7.4 The Regulatory Clock Context (Condensed, ⚠ Where Not Re-Verified)

Why a bank runs all of this: regulators now treat *clock quality* as a market-integrity property. The EU regime — **MiFID II's clock-synchronisation RTS (the delegated regulation on the level of accuracy of business clocks, Commission Delegated Regulation (EU) 2017/574 ⚠ number/articulation not re-verified this pass — EUR-Lex was unreachable from this session; cross-ref the trading sibling's RTS 6 coverage for the surrounding apparatus)** — requires trading clocks to be synchronised to UTC at microsecond-level accuracy for timestamping and latency measurement, with the widely documented HFT threshold at **100 µs** (⚠ vendor/secondary-sourced; see §13.4). The US side (FINRA, the SEC's consolidated audit trail clock requirements) and MAS expectations push the same direction ✅-in-repo (MAS overlay cross-ref). The architectural upshot, which *is* verified: exchange timestamps (venue-side) and firm timestamps (RTS-side) are only comparable if both chains are UTC-traceable — which is why PTP discipline is a *network* requirement, not a server setting, and why the fabric's timing design (§12) is an audit artifact as much as an engineering one.

### 7.5 The Timing Design in One Table

| Design question | The financial-floor answer | Why |
| --- | --- | --- |
| Time source | Dual grandmasters (GNSS + backup), BMCA-selected; in the colo cage, the venue's domain is the reference of record | A single GM is a single point of failure for every timestamp in the firm (§7.2) |
| Switch role | Boundary clock on every hop of the timing path | Per-hop delay measured, not estimated — the accuracy lever after NIC timestamping (§7.3) |
| Endpoint | Hardware NIC timestamping (PTP), never software | Software timestamping is an order of magnitude worse (§7.2) |
| Transport | Dedicated PTP class/VLAN at top 802.1p priority | Sync must never queue behind a feed burst (§7.3) |
| Audit | UTC-traceability evidence pack produced by the network team | The clock chain is a regulatory artifact (§7.4) |
| Worst case | Documented holdover behaviour per device; alarms on clock-quality degradation | A silent clock drift is worse than a failure — it corrupts the record without anyone noticing |

---

## 8. Low-Latency L2 — Cut-Through, Buffers and the Lossless Debate

### 8.1 Cut-Through vs Store-and-Forward (Verified Mechanics)

A switch can forward a frame two ways:

- **Store-and-forward** — receive the whole frame, verify the FCS, then forward. Zero corrupted frames escape, but every frame pays the full serialisation delay of the ingress link.
- **Cut-through** — start forwarding as soon as the destination address and egress port are known, *before* the frame is fully received; corrupted frames can be forwarded (error checking is the receiver's job). Fragment-free is the middle variant (hold until the first 64 bytes, killing collision runts).

✅ (cut-through mechanics verified via the switching literature — Wikipedia secondary, citing Cisco's cut-through/store-and-forward material). The low-latency relevance: on a 10/25/40/100 G path, store-and-forward adds the ingress serialisation of a full frame per hop (microseconds at the large-frame end, sub-microsecond for typical market-data frames ⚠-derived), so a multi-hop trading path built on store-and-forward devices pays that tax *per switch*. Hence: **cut-through is the norm for financial-fabric switches; store-and-forward is confined to devices where error-checking matters more than delay** (some WAN/security appliances). ⚠-structural: the *"cut-through everywhere on the hot path"* norm is industry practice; the per-frame arithmetic is derived from serialisation physics ✅-derived. (The cut-through switching standardisation effort — the 802.1 TSN "cut-through forwarding bridges" project 802.1DU — is noted in the 802.1 record ⚠ name-level.)

### 8.2 Micro-Buffers and Microbursts

The buffer story that actually bites in trading: a switch with **small per-port buffers** ("micro-buffers") handles steady rate fine but drops frames when N servers' traffic arrives at one egress in the same instant — a **microburst** of a few hundred microseconds. Market-data fanout is *bursty by construction* (an exchange event publishes a wave of updates), so microburst loss on the feed path is a real, measured failure mode ⚠-structural (documented across vendor buffer-architecture literature — Arista/Cisco/Mellanox buffer write-ups disagree on the right buffer size precisely because the workloads differ; treat any vendor's "right-size buffer" claim as marketing-adjacent ⚠).

Design responses, in order of preference for a trading floor:

1. **Remove the burst** — pace or shape at the source (the app-layer pacing of §8.4), keep the fanout leaves at 1:1 (§3.2).
2. **Absorb the burst** — deeper buffers on the *multicast replication* leaves, which is where a fanout wave converges.
3. **Signal instead of dropping** — ECN-style congestion notification (IEEE 802.1Qau, the congestion-notification amendment ✅ designation) marks rather than drops — but 802.1Qau deployment is rare; TCP's own ECN (RFC 3168) is the mainstream cousin, and *market data is UDP*, so L2/ECN signalling is of limited use on the feed path ⚠-structural.

### 8.3 Lossless Ethernet — PFC and the Controversy

**Priority Flow Control (PFC, IEEE 802.1Qbb)** extends the classic Ethernet pause (IEEE 802.3x, 1997: a MAC-control pause frame, EtherType 0x8808, opcode 0x0001, destination 01-80-C2-00-00-01, halting the sender for a specified time ✅) from whole-link to **per-CoS-class**: each of the 8 802.1p priorities can be paused independently, so a lossless *class* can run over a lossy link — the mechanism behind "lossless Ethernet"/DCB fabrics (the Data Center Bridging amendment family: 802.1Qbb PFC, 802.1Qaz enhanced transmission selection, 802.1Qau congestion notification ✅ designations).

The controversy, stated plainly: **lossless L2 is loved by storage/RDMA (where a dropped frame is catastrophic and retry is expensive) and distrusted by the low-latency trading community** for three documented-in-practice reasons:

1. **Head-of-line blocking** — a paused class halts the queue behind it, so one slow consumer of the pause class delays *every* class behind it on that link, including the unpaused trading traffic. This is the classic "pause storm" failure mode ⚠-structural.
2. **Congestion spreading** — PFC back-pressure propagates hop by hop; a congestion point at one leaf can pause *upstream* leaves and, in a loop-free-but-meshed fabric, spread to neighbours that had nothing to do with the congestion — the "victim flows" problem ⚠-structural.
3. **Deadlock and misconfiguration risk** — PFC domains must be configured consistently across every hop (buffer thresholds, class maps); a mismatch creates a silent absorption point that turns into drops *with* added latency — the worst of both worlds ⚠-structural.

And the trading-specific verdict, which this guide carries as ⚠/❌-adjacent practice: **lossless L2 for low-latency trading paths is usually avoided.** The market-data path is UDP multicast with application-level sequence-gap recovery (§6.2) — it does not need losslessness and it cannot afford PFC's blocking behaviour. The order path is TCP with retransmit — a rare loss recovered by TCP is cheaper than a PFC storm. What the industry does instead is **pace at the application layer** (rate-limit and shape the fanout so the fabric never congests), which the low-latency messaging literature has recommended for two decades (the classic market-data messaging guidance — 29West's *Topics in High Performance Messaging* — argued against relying on link-level flow control for market data ⚠ name-level, not re-fetched this pass). ❌ rejected: the vendor claim that "lossless Ethernet is required for high-performance financial networking" — no neutral benchmark supports it, and the PFC failure modes above are documented in the RDMA/storage community's own literature (where lossless is justified by *different* workloads — RDMA's no-retry semantics). ⚠: the counter-argument — that a *well-tuned* PFC domain with tiny pause thresholds can protect a specific priority class on a specific link without catastrophe — is credible in niche designs (e.g., protecting PTP or a single order link) but is precisely the kind of tuning that fails in production. §12 keeps PFC off the trading path and confines any lossless class to the storage/HA VLAN.

### 8.4 The App-Layer Pacing Discipline (Cross-Referenced)

The endpoint half of "don't make the fabric lossless, make the sources polite" — pacing, rate-shaping of outbound multicast, NIC-level timestamping and interrupt coalescing control — is owned by the [low-latency C++ sibling](low_latency_cpp_development_guide.md) and the [trading guide](trading_system_software_architecture_guide.md) §9 (kernel bypass, busy-polling, ring buffers). The fabric-side contract with those endpoints: **if the sources pace, the fabric can stay shallow-buffered, cut-through and lossy-but-fast; if they do not, no amount of L2 machinery saves the path.** ⚠-structural (the pacing-vs-lossless trade as engineering culture rather than a single citable source).

---

## 9. Exchange Connectivity and the Colocation Edge

> **Division of labour with the siblings:** the [market-making sibling](../banking/market_making_singapore_guide.md) §7 owns the SGX story (Reach, the SGX data centre, the S$250M initiative — verified in-repo: Reach live 15 August 2011, sub-90 µs engine, co-location at the SGX data centre); the [trading guide](trading_system_software_architecture_guide.md) §7 owns venue connectivity, FIX vs native protocols and the colocation decision; the [market-data sibling](../banking/market_data_consumption_guide.md) §5.4 owns multicast + co-location consumption. **This section condenses the L2-relevant essence: what terminates where, and what the fabric does about it.** Do not re-derive the siblings.

### 9.1 The Connectivity Menu

A firm reaches an exchange four ways, in increasing order of latency cost and decreasing order of operational convenience:

| Path | What it is | L2 relevance |
| --- | --- | --- |
| Direct cross-connect (colo) | A fibre pair from the firm's cage to the exchange's matching-engine/market-data infrastructure inside the same data centre (or via the building's meet-me room) | The hot path: terminates at the colo leaf, plain L2/L3 access, no extranet in the middle |
| Exchange extranet / VPN | The venue's managed connectivity product (or a financial extranet such as the BT Radianz-style community the zero-trust sibling's banking cross-refs describe — cross-ref [ZTNA guide](zero_trust_network_architecture_guide.md) §8 for the extranet/VPN estate angle) | Terminates at the edge routers; adds the extranet's own switching — fine for non-HFT, wrong for the colo hot path |
| Metro / WAN to a non-colo venue | The firm's own or leased line to a venue it does not colocate in | L3/MPLS; the L2 story ends at the edge |
| Vendor/aggregator | A broker or market-data vendor's feed | The vendor's network, not the firm's |

The architectural rule, consistent with §2 and verified-in-repo practice: **the HFT-adjacent paths (direct feeds, order entry to the primary venues) run over direct cross-connects at the colocation leaf; everything else runs over the routed edge.** The extranet/VPN path is a *routed* product — its presence is why the edge is L3 and why the "stretched L2 to the venue" pattern does not exist (nobody bridges their floor VLAN into an exchange).

### 9.2 Colocation and the Physics of the Meet-Me Room

Colocation exists to buy metres, not bandwidth. The physics: light in silica fibre travels at ≈**5 nanoseconds per metre** (speed of light in vacuum, 299,792,458 m/s, divided by a refractive index of ≈1.47) ✅-derived (physical constants; no source needed beyond the arithmetic). At that rate:

- **1 metre of fibre ≈ 5 ns** one way — and every metre, every patch panel, every switch hop is on the bid/offer path twice (order out, fill back).
- A **100-metre difference** between two colocation options is ≈1 µs of round-trip latency — a lifetime in HFT terms.
- The **meet-me room / cross-connect** is where the exchange's demarcation meets the firm's: cross-connect fees, patch-panel counts and physical cable routes are latency and reliability decisions, not procurement trivia. ⚠-structural (the per-metre numbers are derived physics ✅; the *commercial* framing — "colo decisions are latency decisions" — is the market-making sibling's verified §7 story condensed).

The SGX instance, condensed from the [market-making sibling](../banking/market_making_singapore_guide.md) §7.2: SGX's low-latency story is the **Reach** engine (live 15 August 2011; sub-90 µs; part of the S$250M Reach initiative) running in a purpose-built SGX data centre with co-location — the venue-side budget that a firm's colo cage sits inside ✅-in-repo. The firm-side budget (tick-to-quote through the firm's own fabric) is this guide's §2–§8 content.

### 9.3 What the Colo Rack's L2 Looks Like

The colocation cage's fabric, in one paragraph (designed in §12): **two leaves (A/B), each cross-connected to the exchange infrastructure; the market-data NICs join the venue's multicast groups at those leaves with IGMP snooping on; the order gateways terminate their FIX/native sessions on the same leaves; PTP arrives from the venue's timing domain (§7.3); and nothing — no firewall, no overlay VTEP, no SPAN tax — sits in the unicast hot path** unless the capture layer taps it passively (§11). ⚠-structural: the *passive-only* capture on the hot path is practice (active inline security appliances on the HFT path are the exception, used by firms that must filter but cannot afford the hop); per-venue rules differ and are cross-referenced, not re-verified, here.

---

## 10. HA/DR — Stretched L2 vs the Routed L3 Exit

### 10.1 The Question

Every bank running a trading estate eventually asks it: **do we stretch L2 between the primary data centre and the DR site (so a workload can fail over with its IP and MAC — or migrate live — as if nothing happened), or do we make every site a routed island and fail over at L3?** The debate is one of the most contested in data-centre networking, and the honest answer is *it depends on what you are failing over* — which is why this section separates the claims from the engineering.

### 10.2 The Stretched-L2 Case (VXLAN-EVPN Across Sites)

What stretching actually means today is not a giant VLAN bridged by dark fibre (the 2000s pattern, with its broadcast domains, STP across sites and failure storms) but **VXLAN-EVPN between the sites' fabrics: the same §5 overlay, extended over the inter-site transport.** The documented attractions:

- **Workload mobility with identity preserved** — a VM/container (or a stateful appliance pair) keeps its IP/MAC and moves or fails over without re-addressing; the historical driver was hypervisor live migration (vSphere vMotion-class mobility, which classically required L2 adjacency at the VM's network — ⚠-structural: modern hypervisors support routed/L3 live migration, which has weakened this driver; the "vMotion needs stretched L2" claim is *dated* and is ❌ as a universal truth in 2026).
- **Active/active clusters with L2 semantics** — some HA/clustering software still assumes it can see peer heartbeats and gratuitous ARPs on the same broadcast domain.
- **Seamless DR for appliances that cannot be re-addressed** — the legacy-appliance argument (§2.3).

The costs, equally documented:

- **The failure domain becomes the WAN.** A stretched L2 domain that spans sites makes a WAN partition look like a bridging loop/black-hole event; split-horizon and DF-election machinery (RFC 7432 §8, RFC 8365 — verified ✅) handle the *protocol* correctly, but the *operational* blast radius of one misconfigured overlay now covers both sites.
- **East-west hairpinning and latency** — traffic between a workload and its peer on the other site crosses the inter-site link; on a trading path that is microseconds you do not have.
- **Inter-site transport quality** — stretched L2 is only as good as the transport's MTU, loss and latency; VXLAN's 50-byte overhead (§5.4) and any loss mean the overlay's BUM/learning behaviour degrades exactly when you need it.

### 10.3 The Routed L3 Exit — the Default for Trading

The counter-design, and the one this guide recommends for trading paths: **every site is a routed island; the L2 domain dies at the site edge; workloads fail over by IP (DNS/LB/anycast or BGP), and stateful sessions fail over by design (session replication, active/standby gateways) rather than by MAC continuity.** The op-resilience overlay for this — session estates, graceful drain, the discipline of making every component restartable — is owned by the [zero-downtime sibling](zero_downtime_system_design_guide.md) and the [ERM sibling](../banking/enterprise_risk_management_guide.md)'s operational-risk frame. Why routed wins for a trading floor:

1. **Fast, deterministic failover** — L3 convergence (or better, BGP/anycast re-advertisement) is bounded and testable; stretched-L2 failover depends on the overlay's MAC-mobility convergence and the WAN's behaviour.
2. **Failure containment** — a DR test, a WAN partition or a broadcast storm cannot cross into the production site's L2 domain.
3. **No hairpinning** — each site's traffic stays local until it must leave.
4. **The trading hot path never needed L2 across sites anyway** — order sessions reconnect (FIX session recovery is the trading sibling's §2/§4 machinery ✅-in-repo); market-data streams rejoin (multicast gap recovery, §6); what *looks* like it needs stretch is almost always a stateful middleware problem that session replication solves better.

### 10.4 The Synthesis — Where Each Belongs

| Workload class | Design | Why |
| --- | --- | --- |
| Trading hot path (feed handlers, gateways, algos) | Routed islands; no stretch | Latency, determinism, containment (§10.3) |
| Clustering/HA pairs that need L2 | Stretch *selectively* via VXLAN-EVPN, one VNI per cluster, over the DCI — or re-architect to L3 clustering | The overlay's split-horizon/DF machinery handles it safely (§10.2); keep the stretched set tiny |
| Legacy appliances (cannot re-address) | Stretched VXLAN-EVPN VLAN, isolated | Cheaper than the re-platform; contain it |
| VM live-migration mobility | Routed live migration where supported; stretch only the migration VLAN if genuinely required | The dated vMotion-needs-L2 claim is ❌ as a universal |

✅-structural: the *debate framing* and the workload-class synthesis are this guide's engineering position, consistent with the vendor-neutral direction of the zero-downtime and ERM siblings; the protocol capabilities it relies on (EVPN multihoming, split-horizon, aliasing — RFC 7432/8365) are ✅-verified above. The specific recommendation "do not stretch the trading path" is a design judgement, flagged ⚠-structural, not a standard.

---

## 11. Compliance and Recording — Taps, SPAN and Packet Capture

> **Division of labour with the siblings:** the [trading guide](trading_system_software_architecture_guide.md) §10 owns surveillance and reconstructability (T+0 monitoring, order-path reconstruction — in-repo verified); the regulatory *content* (15c3-5, RTS 6, MAS) lives in the banking siblings. **This section covers only the L2 capture mechanics** — how a firm gets a trustworthy copy of every frame without disturbing the path. Do not re-derive the siblings.

### 11.1 The Capture Options

| Mechanism | What it is | Use in a financial estate |
| --- | --- | --- |
| **Network tap** | A passive optical/electrical split on the link — the copy is physics, not switch state; the tap cannot be "forgotten" by a config change | The hot path and any regulated session path: order entry, direct feeds, gateway-to-exchange links |
| **SPAN (port mirroring)** | The switch copies frames from a source port/VLAN to a destination port — configured, therefore reconfigurable/fallible, and on some switches it taxes forwarding | Non-hot-path capture, ad-hoc troubleshooting, lower-assurance recording |
| **ERSPAN / RSPAN** | Remote/encapsulated mirroring: SPAN traffic tunneled (IP-encapsulated) to a collector elsewhere | Centralising capture from many leaves to a recorder cluster; the L2/L3 detail is vendor-implementation-specific ⚠ |
| **Inline capture appliance** | A device physically in the path copying as it forwards | Only where the latency budget tolerates the hop (rarely on the HFT path — §9.3) |

⚠-structural: tap/SPAN/ERSPAN are standard switch/network practice, not codified in one IEEE standard — SPAN/ERSPAN are Cisco-originated terms that became industry generics, and per-vendor mirroring behaviour is vendor-internal; treat the terminology as practice-verified, the per-platform semantics as ⚠ vendor-documented.

### 11.2 The Fabric-Design Rules for Capture

1. **Passive on the hot path, SPAN elsewhere.** The regulated session paths (order entry, direct market data) get physical taps; the general estate makes do with SPAN. The taps feed a **capture/recording farm** on its own leaf — the recorder NICs are *receivers only*, and the capture VLAN is a normal L2 domain with the recorders as the only members. ⚠-structural.
2. **Timestamp at the tap, not at the recorder.** Capture appliances timestamp frames at the tap point (PTP-disciplined, §7) so the recording's timebase is the network's, not the collector's queue's. ✅-in-repo direction (the trading sibling §10's reconstructability requirement: replay any order's full life + market context from stored events).
3. **Capture is not optional plumbing** — it is the raw material of regulatory reconstruction (the trading sibling's T+0 surveillance and reconstructability content) and of the latency forensics that the Corvil-class monitoring tools perform. ⚠ name-level: Corvil-style network monitoring for trading (per-message latency analytics, feed-vs-feed skew measurement) is an established product category in the low-latency industry, but this pass did not re-verify any vendor's product specifics — the *category* is named only as industry context.
4. **Never let capture tax the path.** SPAN on a hot leaf can drop or delay under load on some platforms; the rule is: hot paths are tapped, not spanned; if a platform must SPAN a hot port, the mirror goes to a dedicated monitoring destination and the platform's SPAN behaviour is load-tested. ⚠-structural (platform-specific mirroring behaviour).

---

## 12. The Cymbal Bank Worked Example — an L2 Design Review

> **Persona convention:** the only bank persona used in this repository's worked examples is **Cymbal Bank** (see the banking siblings' conventions). This example follows the format of the market-data sibling's §10 worked example: scenario → design → review findings → lessons. All *numbers* here are worked-example sizing assumptions for one fictional floor, not market facts; every *technology* claim they rest on is carried from the verified sections above.

### 12.1 The Scenario

Cymbal Bank's institutional equities/derivatives desk in Singapore is being re-homed. The estate: **two rooms** — a colocation cage inside the SGX data centre (cross-connect to the Reach-based venue infrastructure, §9.2's venue-side budget) and the firm's own trading floor in the bank's Singapore DC a metro hop away — plus a **DR site** in a second Singapore-region DC. The desk runs: two venue feed handlers (SGX Reach market data + one regional venue), an order gateway pair to SGX, ~40 algo/execution servers, the tick-capture/recording farm, and the usual back-office/corporate VLANs. Requirements from the business: deterministic sub-10 µs fabric transit floor-to-venue-leaf; zero STP anywhere on a trading path; multicast fanout to all algos; PTP discipline traceable to UTC for the regulatory clock record; RTO on the order path measured in seconds, not minutes.

### 12.2 The Design Review — What Was Proposed and How It Was Judged

The design under review (an architecture-review artefact, not a vendor RFP):

**1. Fabric topology and sizing (per §3).**

- **Colo room:** 2 leaves (A/B) cross-connected to the SGX infrastructure; both leaves uplink to the same 2-spine pair. Judged **correct** — two spines is the floor for one-site survival (§3.3); four was not required at colo scale. Server-facing ports: 2 × feed-handler NICs (primary + backup feed), 2 × order-gateway pairs, 4 × capture-tap destinations ⇒ **1:1 oversubscription** on the uplinks (§3.2). Finding: sizing holds only if the capture destinations are *receiver-only* (they are — §11.2) and the venue multicast groups are snooped at the leaf, not flooded (they are — §6.2).
- **Trading floor:** 6 leaves (4 application + 2 services/capture) over 4 spines; application leaves at **2:1**, the multicast-fanout leaf at **1:1** (a fanout wave must never contend — §8.2). Four spines chosen so a single spine failure costs 25% of uplink capacity, absorbed by the headroom on the 1:1 leaf and the 2:1 ratio on the rest (§3.3). Finding: **adequate; add one spine if the algo count grows past ~60** — the review's one forward-looking sizing note.

**2. L2 services — VLANs and the overlay (per §4–§5).**

- Access VLANs at the leaves: `MD-VENUE` (exchange multicast ingress), `MD-INTERNAL` (ticker-plant fanout), `ORDER`, `CAPTURE`, `PTP`, `MGMT`, `STORAGE-HA`, `CORP`. VLANs are *local to a leaf's domain*; nothing L2 spans the fabric.
- The only stretched/overlay need: the **gateway HA pair and the tick-capture cluster** want L2 between their two hosts; and DR-site legacy appliances cannot be re-addressed. Design: **VXLAN-EVPN overlay** (RFC 7348 + RFC 7432/8365 mechanics, §5) carrying exactly three VNIs — gateway-HA, capture-cluster, legacy-DR — terminated at anycast VTEPs on the service leaves, with the MP-BGP EVPN control plane on the spine pair. Judged **correct and minimal**: the overlay is the *services* layer, the trading hot path stays off it (§5.4), and the stretched set is tiny and contained (§10.4).
- **Rejected in review:** (a) stretching `MD-VENUE` or `ORDER` to the DR site — the trading path is a routed island (§10.3); (b) a flood-and-learn VXLAN without EVPN — no split-horizon/aliasing, not an HA design (§5.5); (c) any STP in the fabric — RSTP permitted only on `CORP`'s office-leaf stub, and even there the review recommended eventual elimination (§4.4).

**3. Multicast design (per §6).**

- Venue feeds terminate at the colo feed handlers **on the venue's own multicast groups**; the handlers re-originate **internal** fanout as SSM (S,G) groups — source = the active feed handler, group range 232/8 (RFC 4607 ✅) — one group per instrument set.
- **IGMP snooping on every leaf**; the leaves are the first-hop routers for their access domains and run the querier role; a *capture-only* member (the recorder joins every group silently, never the querier) is on the capture leaf. 
- **RP placement**: the internal fanout is SSM and needs no RP (§6.2). The *legacy* ASM groups (a vendor appliance that only speaks (*,G)) are served by an **anycast-RP pair on the two service leaves** — no single RP point of failure. Finding: correct; the review's note — if the firm later re-originates *venue-ASM* groups internally (some regional venue feeds are ASM), those must be mapped to the anycast RP *before* go-live, not discovered in a fire drill.
- **Pacing**: the feed handlers rate-shape the internal fanout to a published pps ceiling; the fabric's leaves stay shallow-buffered and cut-through (§8.4). Finding: the pacing contract must be an *SLO with alarms*, because the entire shallow-buffer design depends on it.

**4. PTP deployment (per §7).**

- **Primary GM** (GNSS) on the trading floor; **backup GM** in the colo cage (where the venue's timing domain is the reference of record); BMCA arbitrates.
- **Every leaf and spine on the timing path is a boundary clock**; the fabric carries PTP on the `PTP` VLAN at the top 802.1p class, so sync never queues behind a feed burst (§7.3).
- Endpoints: feed handlers, gateways, algos and the capture farm all timestamp at the **NIC in hardware** (the trading sibling's verified stack, §7.2) — software timestamping rejected on every trading-path host.
- Finding: sound; the review's compliance note — the **UTC-traceability evidence pack** (GM logs, per-hop clock-quality telemetry, drift alarms) must be produced *by the network team* as an audit artifact, not assembled ad hoc for each regulatory ask (§7.4).

**5. HA path (per §10).**

- **Within the room:** servers dual-homed to leaves A/B (two NICs, active/standby or LACP); a leaf failure costs nothing; a spine failure costs 25% capacity (§3.3). The gateway pair runs active/standby with the EVPN overlay carrying the HA VLAN.
- **Across sites:** the colo cage and the trading floor are **two routed islands** joined by the metro link — no L2 stretch between them. Feed-handler failover = the standby joins the venue groups and takes over the SSM source role; order-gateway failover = FIX session re-establishment (the trading sibling's session machinery). 
- **To DR:** the trading path fails over by **IP/BGP re-advertisement**, not MAC continuity; only the three overlay VNIs stretch (and the legacy-DR VNI is dark until a DR test). RTO on the order path is bounded by session re-establishment + market-data rejoin (gap recovery, §6), which the firm rehearses quarterly. Finding: the DR design was judged the strongest part of the proposal; the review's only demand — **prove the overlay's split-horizon/DF behaviour in a WAN-partition test**, because that is the exact moment stretched-L2 misconfiguration shows up (§10.2).

**6. Capture and compliance (per §11).**

- Physical **taps** on the venue cross-connects and the gateway uplinks; **SPAN** for the internal estate; everything lands on the capture farm (PTP-disciplined NICs, receiver-only). Finding: correct; the review added one rule — the taps are *inventory-controlled physical assets* (a "forgotten" tap is a compliance gap, and SPAN config can be wiped in a change window; a tap cannot).

### 12.3 The Review Verdict

The design was approved with three conditions: (1) the pacing SLO gains alarm-and-page coverage before go-live; (2) the WAN-partition/split-horizon DR test is scheduled in the first quarter post-migration; (3) the UTC-traceability evidence pack is produced and walked through with compliance once, on the record. The design's centre of gravity — **routed fast paths, a minimal and contained overlay, SSM fanout with snooping everywhere, boundary-clock PTP, passive capture, and no stretched trading L2** — is the direct application of §2's mission to one floor: a single fabric binding every desk to the market.

### 12.4 The Findings at a Glance

| # | Design element | Judgement | Condition / note |
| --- | --- | --- | --- |
| 1 | Colo leaves 1:1, 2-spine | ✅ Adequate | Holds only with snooping on and receiver-only capture |
| 2 | Floor leaves 2:1 (app) / 1:1 (fanout), 4-spine | ✅ Adequate | Add a spine past ~60 algo hosts |
| 3 | VLANs local to leaf domains; no fabric-wide L2 | ✅ Correct | Nothing L2 spans the fabric |
| 4 | VXLAN-EVPN overlay, 3 VNIs only, anycast VTEP on service leaves | ✅ Correct and minimal | Hot path stays off the overlay |
| 5 | SSM internal fanout; anycast-RP for legacy ASM | ✅ Correct | Map future venue-ASM re-origination to the RP before go-live |
| 6 | Pacing SLO on feed handlers | ✅ Correct | Must be alarm-and-page covered (condition 1) |
| 7 | Dual GMs; all-BC timing path; NIC hardware timestamping | ✅ Sound | UTC-traceability evidence pack is an audit deliverable (condition 3) |
| 8 | Routed islands; overlay stretch for HA/capture/legacy only | ✅ Strongest part | WAN-partition split-horizon/DF test required (condition 2) |
| 9 | Physical taps on hot paths; SPAN elsewhere | ✅ Correct | Taps are inventory-controlled physical assets |
| 10 | Rejected: stretch `MD-VENUE`/`ORDER`; flood-and-learn VXLAN; STP in fabric | ❌ Not approved | Trading path is a routed island; overlay needs EVPN; RSTP only on the `CORP` stub |

---

## 13. The Claims Status — Verified, Flagged, Rejected

**Integrity convention:** ✅ = verified this pass against a primary or cited source; ✅-in-repo = verified in the cross-referenced sibling's own claims ledger; ⚠ = flagged (vendor-internal, marketing, single-source, or not re-verified this pass); ⚠-structural = industry-standard practice widely documented but not attributable to a single primary source verified this pass; ❌ = disputed claims found in the wild and rejected. Sources are named inline in §13.1.

### 13.1 Verified This Pass (✅)

| # | Claim | Source |
| --- | --- | --- |
| 1 | RFC 7348 (VXLAN): Informational, Aug 2014; MAC-in-UDP; 24-bit VNI; 8-byte VXLAN header; VTEP definition; IANA UDP port 4789; STP-disabled-links and 4,094-VLAN limits in the problem statement | rfc-editor.org (RFC 7348, extracted this pass) |
| 2 | RFC 7432 (EVPN): Standards Track/Proposed Standard, Feb 2015; Sajassi (Cisco) et al.; route types (Ethernet A-D, MAC/IP Advertisement, Inclusive Multicast Ethernet Tag, Ethernet Segment); ESI multihoming functions: split-horizon, aliasing, mass withdrawal, DF election | rfc-editor.org (RFC 7432 + info page, extracted this pass) |
| 3 | RFC 7432 current status/evolution: updated by RFC 8584, 9161, 9572, 9573, 9746 | RFC Editor info page (rfc7432), extracted this pass |
| 4 | RFC 8365 (EVPN NVO): Proposed Standard, Mar 2018; EVPN control plane for VXLAN/NVGRE/MPLS-over-GRE (+GENEVE noted); NVO multihoming split-horizon/mass-withdrawal procedures; updated by RFC 9746 | rfc-editor.org (RFC 8365 info page, extracted this pass) |
| 5 | RFC 7761 (PIM-SM): Internet Standard (STD 83), Mar 2016, obsoletes RFC 4601; shared tree rooted at an RP per group + optional SPTs per source; removed (*,*,RP)/PMBR/IPsec-auth | rfc-editor.org (RFC 7761 info page, extracted this pass) |
| 6 | IGMP: v2 = RFC 2236; v3 = RFC 3376 (Oct 2002, source filtering; RFC Editor now lists it obsolete — replaced by RFC 9776); RFC 4604 updates RFC 3376/3810 for SSM | rfc-editor.org info pages (extracted this pass) |
| 7 | SSM: RFC 4607 (Aug 2006), 232/8 (IPv4) and FF3x::/32 (IPv6) reserved for SSM | rfc-editor.org (RFC 4607, extracted this pass) |
| 8 | 802.1Q VLAN tag: TPID 0x8100, 32-bit tag, 3-bit PCP + 1-bit DEI + 12-bit VID; max frame 1,518→1,522 bytes; 802.1Q-2014 incorporated 802.1D-2004 (bridging and STP content) | 802.1Q record (Wikipedia secondary, cross-checked) |
| 9 | IEEE 1588 editions: 2002 (v1), 2008 (PTPv2, not backward compatible), 2019 (Nov 2019, backward-compatible = PTPv2.1); UDP 319/320; profile concept; 802.1AS gPTP | IEEE 1588 history (Wikipedia secondary) |
| 10 | PFC/pause: 802.3x pause frame (1997, EtherType 0x8808, opcode 0x0001, dest 01-80-C2-00-00-01); 802.1Qbb PFC per-CoS; DCB family incl. 802.1Qaz and 802.1Qau designations | Ethernet flow-control record (Wikipedia secondary) |
| 11 | Cut-through vs store-and-forward mechanics; fragment-free variant | Switching literature (Wikipedia secondary, citing Cisco) |
| 12 | SGX Reach: live 15 Aug 2011, sub-90 µs, S$250M initiative, co-location at the SGX data centre | ✅-in-repo — market-making sibling §7.2 |
| 13 | Venue feeds are binary multicast (CME MDP 3.0-style, sequence-numbered, gap recovery) | ✅-in-repo — market-data sibling §4.3/§5.4, FIX sibling §8 |
| 14 | NIC hardware timestamping (PTP/1588) as the endpoint practice; reconstructability as a regulatory requirement | ✅-in-repo — trading sibling §9/§10 |
| 15 | Fibre propagation ≈5 ns/m (c ÷ refractive index ≈1.47) | ✅-derived — physical constants |

### 13.2 Flagged (⚠ / ⚠-structural)

| # | Claim | Why flagged |
| --- | --- | --- |
| 1 | The four-part mission framing of §2 | This guide's synthesis from cited industry literature — not a quotation from one source (⚠-structural) |
| 2 | Leaf-spine as *the* DC topology; ECMP everywhere | Universal vendor-ecosystem consensus, no neutral single source (⚠-structural); RFC 7348's problem statement is the ✅ anchor |
| 3 | Oversubscription ratio norms (1:1 hot, 3:1–4:1 general) | Practice norms, no standard to verify; the direction is this guide's synthesis (⚠-structural) |
| 4 | STP reconvergence "tens of seconds"; RSTP "sub-second on point-to-point" | Timer behaviour consistent with the standard's defaults but the specific convergence figures are practice-measured (⚠-structural) |
| 5 | VXLAN 50-byte overhead arithmetic is ✅-derived; jumbo-MTU 9,000-byte underlay practice | Practice (⚠-structural) |
| 6 | "VXLAN-EVPN displaced STP-based fabrics" | Industry consensus with vendor commercial interest (⚠-structural); RFC facts are ✅ |
| 7 | RP placement/anycast-RP and snooping-querier roles | Standard multicast engineering practice (⚠-structural) |
| 8 | PTP accuracy figures; BC/TC accuracy comparisons; all-BC deployment rule | Vendor-measured, network-dependent (⚠-structural); mechanisms are ✅ standard definitions |
| 9 | "Cut-through everywhere on the hot path" norm | Industry practice (⚠-structural) |
| 10 | Micro-buffer microburst loss as a trading failure mode; buffer-size disagreements | Vendor literature documents the phenomenon and disagrees on the fix — marketing-adjacent (⚠) |
| 11 | Lossless-L2-avoided-in-trading verdict (PFC HOL blocking, congestion spreading, deadlock risk) | Documented practice in RDMA/storage and low-latency communities but no neutral benchmark; the counter-position (niche tuned PFC) is credible-but-fragile (⚠/❌-adjacent, §8.3) |
| 12 | Colocation commercial framing ("metres not bandwidth"); passive-only hot-path capture | ✅-in-repo direction / practice (⚠-structural) |
| 13 | vMotion-class mobility as the historical stretched-L2 driver, now weakened by routed live migration | Vendor-history, dated as a universal claim (⚠-structural, ❌ as universal) |
| 14 | SPAN/ERSPAN semantics; Corvil-class monitoring category | Cisco-originated generics / industry category — name-level only (⚠) |
| 15 | MiFID II clock-sync RTS specifics (2017/574, 100 µs HFT threshold) | EUR-Lex unreachable this pass — see §13.4 |

### 13.3 Rejected (❌)

| # | Claim | Why rejected |
| --- | --- | --- |
| 1 | "Lossless Ethernet (PFC) is required for high-performance financial networking" | No neutral benchmark supports it; PFC's HOL-blocking/congestion-spreading failure modes are documented in the very communities that use lossless for *different* workloads (RDMA); the market-data path is UDP-with-recovery and needs no losslessness (§8.3) |
| 2 | "vMotion/live migration requires stretched L2, therefore every estate must stretch L2" | Dated as a universal truth — modern hypervisors support routed live migration; stretch only where genuinely required (§10.2) |
| 3 | "A flood-and-learn VXLAN fabric is an HA design" | Without the EVPN control plane there is no split-horizon, aliasing or fast-failover machinery (RFC 7432/8365 capabilities) — it is a scalability hack, not HA (§5.5) |

### 13.4 What Could Not Be Verified

This section records, honestly, everything this pass could not confirm. Each item is flagged ⚠ in the body where it appears:

1. **The MiFID II clock-synchronisation delegated regulation's exact text and article-level accuracy figures** — the widely documented regulatory frame is Commission Delegated Regulation (EU) 2017/574 (RTS 25 on the level of accuracy of business clocks), with microsecond-level UTC sync and a widely cited 100 µs HFT timestamping threshold. EUR-Lex was unreachable from this session (all scraping engines failed) and ESMA's 2015 final-report PDF returned 404; the regulation number and the 100 µs figure are therefore carried ⚠ from secondary/vendor sources (the PTP-timing vendor literature, e.g. FSMLabs' regulatory-compliance material, names MiFID II clock-sync compliance generically ✅-name-level but does not state article-level numbers). Cross-ref the trading sibling's RTS 6 coverage for the surrounding EU apparatus; the precise RTS 25 articulation remains ⚠ (§7.4). ⚠
2. **IEEE standard publication dates and current revisions beyond what the 802.1 record confirmed** — amendment *designations* (802.1w, 802.1s, 802.1Qbb, 802.1Qau, 802.1Qaz, 802.1AS) are ✅; exact ratification years for several (Qbb/Qau/Qaz) and the current consolidated 802.1Q revision number were not re-verified against IEEE SA this pass (IEEE SA store pages were not fetched; the 802.1 Wikipedia record was the source). ⚠
3. **The 802.1DU cut-through-forwarding project status** — noted name-level in the 802.1 record; not verified as published. ⚠
4. **Per-venue colocation timing rules** (which exchange data centres publish PTP domains, and the terms of joining them) — venue-specific, not re-verified per venue this pass. ⚠
5. **Corvil's and Exablaze's current corporate/ownership state and product specifics** — the *categories* (network latency analytics for trading; ultra-low-latency FPGA switching/NICs) are established industry context, but this pass verified no vendor page; names carried name-level only, ownership claims deliberately omitted. ⚠
6. **29West's *Topics in High Performance Messaging* content** — referenced name-level for its anti-link-level-flow-control market-data guidance; the archived document was not re-fetched this pass. ⚠
7. **RFC 9776's full title and content** (the RFC Editor lists it as replacing RFC 3376) — the replacement relationship is ✅ per the RFC Editor info page; the document itself was not fetched. ⚠
8. **The exact "4094 usable VLANs" convention** — derived from the 12-bit VID (4,096 − 2 reserved) ✅-derived; the specific reserved values (0 and 4095) are standard 802.1Q usage carried ⚠-name-level rather than quoted from the standard this pass.
9. **Any vendor's latency, buffer or PTP accuracy specifications** — deliberately not cited; vendor-internal numbers are marketing until measured in-rack. ⚠
10. **The full EVPN RFC family's individual scope** (what each of RFC 8584/9161/9572/9573/9746 specifically changes) — the update list is ✅ per the RFC Editor; per-RFC content was not fetched. ⚠

---

## 14. Glossary, Cross-References and the Closing Summary

### 14.1 The Glossary

| Term | Meaning |
| --- | --- |
| L2 / Layer 2 | The data-link layer — Ethernet bridging, VLANs, MAC addressing; the layer that carries frames within a broadcast domain |
| Broadcast domain | The set of ports that receive a broadcast frame; a VLAN is one (§4) |
| VLAN | Virtual LAN — an 802.1Q-tagged broadcast domain (§4.1) |
| STP / RSTP / MSTP | Spanning Tree Protocol family (802.1D / 802.1w / 802.1s) — loop prevention by blocking ports (§4.2) |
| Leaf / spine | The two stages of a CLOS data-centre fabric: access switches (leaves) fully meshed to core switches (spines) (§3) |
| ECMP | Equal-Cost Multipath — hashing flows across equal-cost L3 paths (§3.1) |
| Oversubscription | Ratio of server-facing to uplink bandwidth at a leaf (§3.2) |
| VXLAN | Virtual eXtensible LAN — RFC 7348 MAC-in-UDP encapsulation (§5.1) |
| VTEP | VXLAN Tunnel End Point — originator/terminator of VXLAN tunnels (§5.1) |
| VNI | VXLAN Network Identifier — the 24-bit overlay segment ID (§5.1) |
| EVPN | Ethernet VPN — RFC 7432 BGP control plane for Ethernet services; MAC routes over MP-BGP (§5.2) |
| ESI | Ethernet Segment Identifier — names the multihomed attachment of a site to two+ PEs (§5.2) |
| NVO | Network Virtualization Overlay — RFC 8365's EVPN-over-VXLAN model (§5.3) |
| MP-BGP | Multiprotocol BGP — BGP carrying multiple address families, incl. EVPN (§5.2) |
| IGMP | Internet Group Management Protocol — host multicast membership (v3 = RFC 3376) (§6.1) |
| IGMP snooping | Switch inspection of IGMP to prune multicast to interested ports only (§6.1) |
| PIM-SM | Protocol Independent Multicast, Sparse Mode — RFC 7761 (STD 83); shared tree per group via an RP (§6.1) |
| RP | Rendezvous Point — the PIM-SM shared-tree root (§6.1) |
| SSM | Source-Specific Multicast — RFC 4607, 232/8, (S,G) joins without an RP (§6.1) |
| ASM | Any-Source Multicast — the classic (*,G) model (§6.2) |
| PTP | Precision Time Protocol — IEEE 1588 (v2 = 2008, v2.1 = 2019) network time sync (§7.1) |
| GM | Grandmaster — the PTP domain's time source (§7.2) |
| BC / TC / OC | Boundary clock / transparent clock / ordinary clock — PTP clock roles (§7.2) |
| gPTP | IEEE 802.1AS — the TSN adaptation of PTP (§7.1) |
| PFC | Priority Flow Control — IEEE 802.1Qbb per-class pause; the lossless-Ethernet mechanism (§8.3) |
| DCB | Data Center Bridging — the 802.1Qbb/Qaz/Qau amendment family (§8.3) |
| Cut-through | Forwarding a frame before it is fully received, after the destination is known (§8.1) |
| Microburst | A sub-millisecond traffic burst exceeding a port's buffer (§8.2) |
| Colocation | Running the firm's servers inside/near the exchange's data centre (§9) |
| Cross-connect | The physical fibre/patch between the firm's cage and the venue's demarcation (§9.1) |
| DCI | Data-Centre Interconnect — the link(s) between sites (§10) |
| Tap / SPAN / ERSPAN | Passive link split / port mirroring / encapsulated remote mirroring (§11.1) |
| RTO | Recovery Time Objective — the target time to restore a function after failure (§10) |

---

### 14.2 The Cross-References

**Sibling guides in this repository** (convention: `banking/` siblings are `../banking/…`; `technology/` siblings are plain filenames):

- [Trading-System Software Architecture](trading_system_software_architecture_guide.md) — the order/execution stack, venue connectivity (§7), the low-latency stack and PTP NIC timestamping (§9), surveillance and reconstructability (§10) — the estate this fabric carries
- [Market Data Consumption](../banking/market_data_consumption_guide.md) — the market-data value chain; §5.4 multicast distribution and co-location; §10 the market-data worked example this guide's §12 extends — cross-ref, never re-derived
- [Market Making in Singapore](../banking/market_making_singapore_guide.md) — §7.2 SGX Reach and the SGX data centre (co-location, sub-90 µs engine) — the venue-side budget
- [FIX Protocol: The Trade Conversation](../banking/fix_protocol_guide.md) — §8 the binary multicast venue protocols (MDP 3.0-style mechanics, sequence numbers)
- [Capital Markets Architecture](../banking/capital_markets_architecture_guide.md) — the front/middle/back umbrella over the estate
- [Enterprise Risk Management](../banking/enterprise_risk_management_guide.md) — the operational-risk and resilience overlay for §10
- [Zero Downtime System Design](zero_downtime_system_design_guide.md) — session estates, HA discipline, the routed-failover machinery of §10.3
- [DDS Guide](dds_guide.md) — the market-data bus (its §10 Cymbal worked example is the bus-level twin of §12's fabric design)
- [Low-Latency C/C++ Development](low_latency_cpp_development_guide.md) — the endpoint pacing/timestamping discipline §8.4 contracts with
- [Zero Trust Network Architecture](zero_trust_network_architecture_guide.md) — the security overlay; its §8 banking frame covers the extranet/VPN estate §9 replaces on the hot path
- [Singapore Data Centres](singapore_data_centres_guide.md) — the physical DC landscape §12's rooms sit in

**Primary sources used in this pass:** the RFC Editor (rfc-editor.org) — [RFC 7348](https://www.rfc-editor.org/rfc/rfc7348.html) (VXLAN, full text extracted), [RFC 7432](https://www.rfc-editor.org/rfc/rfc7432.html) (EVPN) and the [RFC 7432 info page](https://www.rfc-editor.org/info/rfc7432) (status/updates), the [RFC 8365 info page](https://www.rfc-editor.org/info/rfc8365) (EVPN NVO), the [RFC 7761 info page](https://www.rfc-editor.org/info/rfc7761) (PIM-SM), the [RFC 3376 info page](https://www.rfc-editor.org/info/rfc3376) (IGMPv3), [RFC 4607](https://www.rfc-editor.org/info/rfc4607) (SSM), [RFC 4604](https://www.rfc-editor.org/info/rfc4604) (IGMPv3 for SSM) · IEEE 802.1 standards record and IEEE 1588 history (Wikipedia as secondary, cross-checked) · the switching and flow-control literature (Wikipedia secondary, citing Cisco and the IEEE 802.3x/802.1Qbb material) · in-repo verified facts credited to their sibling guides (SGX Reach — market-making sibling §7.2; multicast venue feeds — market-data sibling §4.3/§5.4; PTP NIC timestamping and reconstructability — trading sibling §9/§10). Sources that could not be reached this pass and the claims affected are recorded in §13.4.

### 14.3 The Closing Summary

**Closing note.** Layer 2 is where a financial network's promises are kept or broken: the VLAN that marks a feed, the snooping that keeps it private, the overlay that carries a legacy heartbeat across a data centre without letting a spanning tree anywhere near the trading path, the boundary clock that makes every timestamp an audit artifact, the tap that records what the regulators will one day ask to replay. Architect it with routed fast paths and a contained overlay, prune every broadcast domain down to the servers that need it, pace the sources instead of lossless-ing the fabric, discipline the clocks at the NIC, and capture passively at the edge — and the estate becomes what every desk already assumes it is: one deterministic fabric under the whole floor, binding every order, every tick and every timestamp to the market — the connected floor.
