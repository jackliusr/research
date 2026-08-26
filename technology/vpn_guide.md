# VPN — The Virtual Private Network Deep-Dive

**The Protocol and Remote-Access Deep-Dive — the Definition (Tunnelled Private Connectivity over Public Networks: Encapsulation, Encryption, Authentication), the History (PPTP 1996, IPsec RFC 2401 1998, OpenVPN 2001, WireGuard 2016 → Kernel 5.6 2020), the Protocols Head-to-Head (IPsec/IKEv2, OpenVPN, WireGuard, L2TP, SSTP, PPTP-Legacy), the Architectures (Remote-Access vs Site-to-Site, Full-Tunnel vs Split-Tunnel, the Kill Switch), the Enterprise Patterns (Certificates, AAA, MFA), the Modern Alternatives (ZTNA, SASE/SSE, the Beyond-Zero Lens), the Security Considerations (the Edge-Device CVE Roll Call, Split-Tunnel Risks, No-Logs Claims), the Banking Angle (the Cymbal Bank Estate), a VPN Design Worked Example (a Cymbal Bank), and the One-Page Summary — the Final Word: the Tunnelled Middle Ground**

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Technology / Security — the DEDICATED protocol and remote-access deep-dive of the security cluster. Where [zero_trust_network_architecture_guide.md](zero_trust_network_architecture_guide.md) covers the *paradigm* (identity-first, never-trust-always-verify — the access model that displaces the VPN at the edge), [beyond_zero_enterprise_security_guide.md](beyond_zero_enterprise_security_guide.md) the *beyond-zero* paradigm (the zero-trust-plus layer: the SDP, continuous verification, identity-aware data planes), [distributed_auth_guide.md](distributed_auth_guide.md) the *identity* angle (authentication/authorization — the engine the VPN's AAA layer plugs into), and [cybersecurity_guide.md](cybersecurity_guide.md) the cluster's *umbrella*, this guide is the *tunnel itself*: what a VPN actually is (encapsulation + encryption + authentication over a public network), how it got here (PPTP 1996 → IPsec 1998 → the SSL-VPN 2000s → WireGuard 2016/2020), how the protocols differ at the crypto level, how the architectures are put together (remote-access vs site-to-site, full-tunnel vs split-tunnel, gateways, kill switches), how the enterprise hardens it (certificates, AAA, MFA), why the modern alternatives (ZTNA, SASE/SSE) are displacing it — and why it nevertheless survives (site-to-site, legacy apps, partner extranets), and a full worked design in the Cymbal Bank style. It is written to stand alone and to cross-ref the cluster heavily.
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** the IETF RFC stack, verified this pass directly on rfc-editor.org — **RFC 2401** *Security Architecture for the Internet Protocol* (Kent & Atkinson, **November 1998**, ✅), **RFC 7296** *Internet Key Exchange Protocol Version 2 (IKEv2)* (Kaufman/Hoffman/Nir/Eronen/Kivinen, **October 2014**, Internet Standard STD 79, ✅), **RFC 3948** *UDP Encapsulation of IPsec ESP Packets* (Huttunen et al., **January 2005**, ✅), **RFC 2637** *Point-to-Point Tunneling Protocol (PPTP)* (Hamzeh et al., **July 1999**, Informational, ✅), **RFC 2661** *Layer Two Tunneling Protocol (L2TP)* (Townsley et al., **August 1999**, ✅), **RFC 3193** *Securing L2TP using IPsec* (Patel et al., **November 2001**, ✅), and the 2005 IPsec refresh **RFC 4301 / 4302 / 4303** (Kent & Seo, **December 2005**, ✅); the WireGuard paper — Jason A. Donenfeld, *WireGuard: Next Generation Kernel Network Tunnel*, **NDSS 2017**, 27 February 2017, DOI 10.14722/NDSS.2017.23160 (✅, verified via ndss-symposium.org — NOTE: the arXiv identifier 2012.05071 given in the commissioning brief is INCORRECT: that ID is an unrelated quantitative-biology paper (*The Gossip Paradox*, submitted 8 Dec 2020), verified by direct arXiv extraction this pass; the WireGuard whitepaper's canonical home is the NDSS 2017 proceedings, ⚠-brief-correction); the **Linux kernel 5.6 release** (29 March 2020, WireGuard merged into mainline — ✅ via kernelnewbies.org and the phoronix release announcement); OpenVPN's own history pages (project started **2001**, first open-source release **2002**, ✅ via openvpn.net — the widely-cited May 2001 first release is ⚠ not directly verified); Microsoft's **[MS-SSTP]** Open Specifications page (SSTP tunnels PPP over SSL/TLS on TCP 443, ✅); the NVD entries for **CVE-2021-22893** (Pulse Connect Secure, ✅), **CVE-2022-40684** (Fortinet FortiOS, ✅) and **CVE-2024-21887** (Ivanti Connect Secure, ✅); and the Gartner SASE coinage *The Future of Network Security Is in the Cloud* (Neil MacDonald, Lawrence Orans, Joe Skorupa, **30 August 2019**, ID G00441737 — ✅ metadata via public mirrors; the full Gartner text is paywalled, ⚠). Everything not verifiable this pass is flagged ⚠ honestly in the claims ledger.
> **Last Updated:** August 2026
> **Companion guides (sibling, same folder — plain filenames):** [Zero Trust Network Architecture](zero_trust_network_architecture_guide.md) (**THE paradigm companion — the identity-first, per-app access model that displaces the VPN at the edge; this guide's §6 is the VPN-vs-ZTNA face-off and §9 the coexistence design — cross-ref §6 and §9**), [Beyond Zero Enterprise Security](beyond_zero_enterprise_security_guide.md) (**THE beyond-zero companion — the zero-trust-plus paradigm this guide's §6.4 maps onto; cross-ref §6**), [Distributed Auth](distributed_auth_guide.md) (**THE identity/AAA companion — certificates, OAuth2/OIDC, mTLS, RBAC/ABAC; the enterprise VPN's authentication engine — cross-ref §5**), [Cybersecurity Guide](cybersecurity_guide.md) (the cluster umbrella — the VPN as one control among many — cross-ref §7), [Security by Design](security_by_design_guide.md) (the secure-development discipline the VPN estate must be built under — light cross-ref in §7), [Threat Modeling](threat_modeling_guide.md) (the threat-model angle — STRIDE against the VPN gateway as an edge target — light cross-ref in §7)
> **Companion guides (banking/, prefix `../banking/`):** [Banks in Singapore](../banking/banks_in_singapore_guide.md) (the SG bank estate for §8's regional frame), [Financial Infrastructure](../banking/financial_infrastructure_guide.md) (**the legacy financial-extranet layer — BT Radianz and the financial VPNs this guide's §8.1 maps — cross-ref §8**), [Financial Risk & Compliance Systems](../banking/financial_risk_compliance_systems_guide.md) (the operational-risk angle — a VPN gateway breach is an op-risk event — cross-ref §7 and §8), and the bank software-systems series (HSBC, UOB, OCBC, Standard Chartered…) for the remote-access estate precedents
> **Companion guides (management/, prefix `../management/`):** [Business Case Development](../management/business_case_development_guide.md) (the VPN-modernisation investment angle — the cost of an edge breach vs the cost of ZTNA migration — light cross-ref in §6 and §9)

---

**How to use this guide:** Section 1 is the overview — what a VPN actually is (the "virtual private network"), the definition as tunnelled private connectivity over public networks, the three verbs (encapsulation, encryption, authentication), and the overview table (aspect / description). Section 2 is the history — PPTP and the 1996 Microsoft/Ascend era, IPsec (RFC 2401, November 1998), the SSL-VPN era of the 2000s (OpenVPN, released 2001), WireGuard (introduced 2016, merged into Linux kernel 5.6 in March 2020), the modern remote-access era (cloud gateways, ZTNA displacement), and the history table (year / event / notes). Section 3 is the protocols head-to-head — IPsec/IKEv2 (RFC 7296, 2014), OpenVPN (TLS-based, UDP/TCP, port 443), WireGuard (Noise protocol framework, ChaCha20-Poly1305, Curve25519), L2TP (RFC 2661, 1999 — usually L2TP/IPsec), SSTP (Microsoft, TLS-based), PPTP-legacy (RFC 2637 — broken: MPPE/RC4, CrackPPTP 2012 ⚠), with the comparison table and the crypto table (✅/⚠). Section 4 is the architectures — remote-access vs site-to-site (IPsec tunnel vs GRE/IPsec, DMVPN), full-tunnel vs split-tunnel (definitions, the DNS/routing split, the risks), the kill switch, and the gateways/clients. Section 5 is the enterprise VPN — the workforce remote-access pattern, certificates/AAA (cross-ref distributed_auth_guide.md), and MFA (TOTP, WebAuthn/push, hardware tokens). Section 6 is the VPN vs the modern alternatives — ZTNA (cross-ref zero_trust_network_architecture_guide.md heavily), SASE/SSE (the Gartner coinage), the beyond-zero paradigm (cross-ref beyond_zero_enterprise_security_guide.md), and the "VPN is dead" debate. Section 7 is the security considerations — the known CVEs (Pulse, Fortinet, Ivanti), split-tunnelling risks, the no-logs claims, and the auditing discipline. Section 8 is the banking context — the Cymbal Bank estate's remote-access patterns (traders, branches, third-party/vendor access) with the legacy financial-VPN frame. Section 9 is the worked example — an enterprise VPN design in the Cymbal Bank style: the HA gateway pair in the DMZ, WireGuard for modern clients + IPsec/IKEv2 for legacy and partner site-to-site, and the ZTNA coexistence with a phased migration. Section 10 is the one-page summary — the final word is **the tunnelled middle ground**. The glossary, the claims-status ledger and the cross-references close the file. Cross-references follow the repository convention: sibling guides in `technology/` are plain filenames; guides in `banking/` and `management/` are prefixed `../banking/` and `../management/`. **Integrity convention:** ✅ = verified this pass (live web) or verified in the cross-referenced guide's ledger; ⚠ = flagged/unverified; ⚠-structural = industry-standard practice widely documented but not attributable to a single primary source verified this pass. **The parenthetical (the tunnelled middle ground) is the thesis:** the VPN is neither dead nor sufficient — it is the tunnelled middle ground between the abolished perimeter and the identity-first edge, and this guide is the map of that ground.

---

## Table of Contents

1. [The VPN Overview](#1-the-vpn-overview)
   - 1.1 [The Short Answer](#11-the-short-answer)
   - 1.2 [The Definition — Tunnelled Private Connectivity over Public Networks](#12-the-definition--tunnelled-private-connectivity-over-public-networks)
   - 1.3 [The Three Verbs — Encapsulation, Encryption, Authentication](#13-the-three-verbs--encapsulation-encryption-authentication)
   - 1.4 [The Overview Table — Aspect / Description](#14-the-overview-table--aspect--description)
   - 1.5 [The Vocabulary — VPN, Tunnel, Virtual Interface](#15-the-vocabulary--vpn-tunnel-virtual-interface)
   - 1.6 [The VPN in the Cluster — Where This Guide Sits](#16-the-vpn-in-the-cluster--where-this-guide-sits)
   - 1.7 [The Layered View — Where Each VPN Sits in the Stack](#17-the-layered-view--where-each-vpn-sits-in-the-stack)
2. [The History](#2-the-history)
   - 2.1 [The Dial-Up Ancestry — PPTP and the 1996 Microsoft/Ascend Era](#21-the-dial-up-ancestry--pptp-and-the-1996-microsoftascend-era)
   - 2.2 [The IETF Standardization — IPsec, RFC 2401, November 1998](#22-the-ietf-standardization--ipsec-rfc-2401-november-1998)
   - 2.3 [The SSL-VPN Era — the 2000s and OpenVPN (2001)](#23-the-ssl-vpn-era--the-2000s-and-openvpn-2001)
   - 2.4 [The WireGuard — 2016 Announcement to the Kernel 5.6 Merge (March 2020)](#24-the-wireguard--2016-announcement-to-the-kernel-56-merge-march-2020)
   - 2.5 [The Modern Remote-Access Era — Cloud Gateways and the ZTNA Displacement](#25-the-modern-remote-access-era--cloud-gateways-and-the-ztna-displacement)
   - 2.6 [The History Table — Year / Event / Notes](#26-the-history-table--year--event--notes)
   - 2.7 [Reading the History Table — the Four Eras](#27-reading-the-history-table--the-four-eras)
3. [The Protocols Head-to-Head](#3-the-protocols-head-to-head)
   - 3.1 [IPsec / IKEv2 — RFC 7296, 2014](#31-ipsec--ikev2--rfc-7296-2014)
   - 3.2 [OpenVPN — TLS-Based, UDP/TCP, Port 443](#32-openvpn--tls-based-udptcp-port-443)
   - 3.3 [WireGuard — the Noise Framework, ChaCha20-Poly1305, Curve25519](#33-wireguard--the-noise-framework-chacha20-poly1305-curve25519)
   - 3.4 [L2TP — RFC 2661, 1999, Usually L2TP/IPsec](#34-l2tp--rfc-2661-1999-usually-l2tpipsec)
   - 3.5 [SSTP — Microsoft's TLS-Based Entry](#35-sstp--microsofts-tls-based-entry)
   - 3.6 [PPTP-Legacy — RFC 2637, Broken: MPPE/RC4](#36-pptp-legacy--rfc-2637-broken-mpperc4)
   - 3.7 [The Comparison Table — Protocol / Security / Performance / Ecosystem / Maturity](#37-the-comparison-table--protocol--security--performance--ecosystem--maturity)
   - 3.8 [The Crypto Table — the Primitives per Protocol](#38-the-crypto-table--the-primitives-per-protocol)
   - 3.9 [The Protocol Selection Checklist — a Decision Procedure](#39-the-protocol-selection-checklist--a-decision-procedure)
4. [The VPN Architectures](#4-the-vpn-architectures)
   - 4.1 [Remote-Access vs Site-to-Site](#41-remote-access-vs-site-to-site)
   - 4.2 [The Site-to-Site Patterns — IPsec Tunnel, GRE/IPsec, DMVPN](#42-the-site-to-site-patterns--ipsec-tunnel-greipsec-dmvpn)
   - 4.3 [Full-Tunnel vs Split-Tunnel — the Definitions](#43-full-tunnel-vs-split-tunnel--the-definitions)
   - 4.4 [The DNS and Routing Split — Where Split-Tunnel Bites](#44-the-dns-and-routing-split--where-split-tunnel-bites)
   - 4.5 [The Kill Switch](#45-the-kill-switch)
   - 4.6 [The Gateways and Clients](#46-the-gateways-and-clients)
   - 4.7 [The Gateway Placement Patterns — On-Prem DMZ, Cloud, Provider Edge](#47-the-gateway-placement-patterns--on-prem-dmz-cloud-provider-edge)
5. [The Enterprise VPN](#5-the-enterprise-vpn)
   - 5.1 [The Workforce Remote-Access Pattern](#51-the-workforce-remote-access-pattern)
   - 5.2 [Certificates and AAA — the PKI Tie-In](#52-certificates-and-aaa--the-pki-tie-in)
   - 5.3 [The MFA Layer — TOTP, WebAuthn/Push, Hardware Tokens](#53-the-mfa-layer--totp-webauthnpush-hardware-tokens)
   - 5.4 [The Enterprise Table — Control / Pattern / Notes](#54-the-enterprise-table--control--pattern--notes)
6. [The VPN vs the Modern Alternatives](#6-the-vpn-vs-the-modern-alternatives)
   - 6.1 [The ZTNA — Identity-First, Per-App Access](#61-the-ztna--identity-first-per-app-access)
   - 6.2 [The Comparison — Network-Layer Access vs Identity/Device-Based Per-App Access](#62-the-comparison--network-layer-access-vs-identitydevice-based-per-app-access)
   - 6.3 [SASE and SSE — the Gartner Coinage](#63-sase-and-sse--the-gartner-coinage)
   - 6.4 [The Beyond-Zero Paradigm](#64-the-beyond-zero-paradigm)
   - 6.5 [The "VPN Is Dead" Debate — the Reality](#65-the-vpn-is-dead-debate--the-reality)
   - 6.6 [The Decision Rules — Keep the Tunnel, Broker the Apps](#66-the-decision-rules--keep-the-tunnel-broker-the-apps)
7. [The Security Considerations](#7-the-security-considerations)
   - 7.1 [The Edge-Device Target Pattern — the CVE Roll Call](#71-the-edge-device-target-pattern--the-cve-roll-call)
   - 7.2 [The Split-Tunnelling Risks](#72-the-split-tunnelling-risks)
   - 7.3 [The Logging and Privacy — the No-Logs Claims](#73-the-logging-and-privacy--the-no-logs-claims)
   - 7.4 [The Auditing — Config Reviews, Patch Cadence, Client Posture](#74-the-auditing--config-reviews-patch-cadence-client-posture)
   - 7.5 [The Security Table — Risk / Mitigation / Status](#75-the-security-table--risk--mitigation--status)
8. [The Banking Context — the Cymbal Bank Estate](#8-the-banking-context--the-cymbal-bank-estate)
   - 8.1 [The Legacy Financial-VPN Layer — BT Radianz and the Extranets](#81-the-legacy-financial-vpn-layer--bt-radianz-and-the-extranets)
   - 8.2 [The Traders — Low-Latency Remote Access](#82-the-traders--low-latency-remote-access)
   - 8.3 [The Branch Offices — Site-to-Site](#83-the-branch-offices--site-to-site)
   - 8.4 [The Third-Party and Vendor Access](#84-the-third-party-and-vendor-access)
   - 8.5 [The Banking Table — Pattern / Estate / Control](#85-the-banking-table--pattern--estate--control)
9. [The Worked Example — an Enterprise VPN Design in the Cymbal Bank Style](#9-the-worked-example--an-enterprise-vpn-design-in-the-cymbal-bank-style)
   - 9.1 [The Scenario](#91-the-scenario)
   - 9.2 [The Gateway Topology — the HA Pair in the DMZ](#92-the-gateway-topology--the-ha-pair-in-the-dmz)
   - 9.3 [The Protocol Choice — WireGuard Plus IPsec/IKEv2](#93-the-protocol-choice--wireguard-plus-ipsecikev2)
   - 9.4 [The ZTNA Coexistence — the Phased Migration](#94-the-ztna-coexistence--the-phased-migration)
   - 9.5 [The Design Table — Component / Choice / Rationale](#95-the-design-table--component--choice--rationale)
   - 9.6 [The Lessons](#96-the-lessons)
10. [The Summary — the One-Page VPN](#10-the-summary--the-one-page-vpn)
    - 10.1 [The One-Page Summary](#101-the-one-page-summary)
    - 10.2 [The Final Word — the Tunnelled Middle Ground](#102-the-final-word--the-tunnelled-middle-ground)
11. [The Glossary](#11-the-glossary)

---

## 1. The VPN Overview

### 1.1 The Short Answer

A **VPN** — a *virtual private network* — is a way of carrying private, protected traffic across a public network (the internet, or any untrusted transit) as if the endpoints were on the same private wire. The trick is not magic and not a dedicated circuit: the private traffic is **encapsulated** (wrapped in ordinary packets), **encrypted** (so the transit cannot read it), and **authenticated** (so both ends know exactly who they are talking to and that nothing was altered in flight). The result is a **tunnel**: a logical, point-to-point, cryptographically protected pipe running *through* the public network rather than *around* it.

That single idea — private connectivity over public infrastructure, built from cryptography instead of leased lines — is what made the modern enterprise possible. It is how the trader at home reaches the dealing floor, how the branch office in Jurong reaches the Singapore head office, how a partner firm reaches an extranet without a private circuit, and how the remote worker's laptop becomes, for policy purposes, "inside the network." It is also, in 2026, the technology under sustained attack and sustained displacement: the same tunnel that grants access is the favourite target of attackers (the edge-device CVE roll call in §7.1) and the favourite replacement candidate of the zero-trust movement (§6). The VPN is neither dead nor sufficient. It is the **tunnelled middle ground** — and this guide maps that ground.

### 1.2 The Definition — Tunnelled Private Connectivity over Public Networks

The IETF's own framing in the IPsec architecture — RFC 2401 (November 1998) and its successor RFC 4301 (December 2005) — describes the goal as providing "security services for traffic at the IP layer" (✅ the phrase is verbatim from RFC 4301's abstract, extracted this pass) — confidentiality, integrity, and authentication — so that a private network can be built *over* an insecure one. The general VPN definition decomposes into three properties that must hold simultaneously:

- **Tunnelling (encapsulation).** The original packet (or frame) is wrapped inside another packet whose header addresses the *tunnel endpoints*, not the final destination. To the public network, each VPN packet is just another packet between the gateway IPs; the true source, destination, and payload are hidden inside. Encapsulation is what makes the "virtual" in "virtual private network": the topology of the tunnel need not match the topology of the underlying network at all.
- **Confidentiality (encryption).** The encapsulated payload is encrypted so that anyone tapping the transit path — the ISP, the coffee-shop Wi-Fi operator, a nation-state at the IXP — sees ciphertext only. Encryption is what makes the "private" in "virtual private network": privacy is produced by cryptography rather than by physical possession of the wire.
- **Authentication and integrity.** Both endpoints authenticate each other (mutual authentication — certificate, pre-shared key, or identity), and every packet is integrity-protected and replay-protected so that an on-path attacker cannot inject, reorder, or modify traffic inside the tunnel.

A VPN is therefore best understood as a **cryptographic trust boundary superimposed on an untrusted transport**. The trust boundary travels with the endpoints — it does not depend on the physical network between them. That is the property the whole industry is built on, and it is the property the modern alternatives (§6) challenge: a VPN establishes *where you are allowed to be* (a network); ZTNA establishes *what you are allowed to do* (applications), per identity and per device.

### 1.3 The Three Verbs — Encapsulation, Encryption, Authentication

The three verbs deserve their own subsection because every protocol debate in §3 is a debate about *how* these three are done — and every security failure in §7 is a failure of one of them.

- **Encapsulation — the "how it rides."** Each protocol wraps traffic differently: PPTP and L2TP ride PPP inside GRE (RFC 2637 uses "an enhanced GRE mechanism"; RFC 2661 tunnels PPP packets directly); IPsec ESP wraps IP packets inside ESP (RFC 4303); WireGuard wraps IP packets inside UDP (its own framing); OpenVPN and SSTP ride inside TLS streams (TCP, or UDP for OpenVPN). The encapsulation choice drives the port profile (443 for OpenVPN/SSTP is the deliberate camouflage-and-firewall-bypass choice) and the overhead budget (MTU/MSS effects, §4.6 and the glossary).
- **Encryption — the "how it hides."** The primitives range from the broken (PPTP's MPPE over RC4, §3.6) through the solid-but-heavy (IPsec's AES-CBC/GCM negotiated over IKE) to the modern default (WireGuard's ChaCha20-Poly1305 AEAD, §3.3). The encryption choice drives performance (hardware AES-NI vs ChaCha20 on low-end CPUs) and the audit story (which algorithms the policy allows — and which legacy algorithms the estate still carries).
- **Authentication — the "who it is."** Every tunnel must answer three questions: who is the peer (machine or user), is the key material genuine (certificate chain, key fingerprint), and is this packet from that peer right now (nonce/replay protection)? The answers come from pre-shared keys, X.509 certificates, or public-key identity (WireGuard's Curve25519 keys) — and, at the enterprise layer, from the AAA stack (§5.2) that sits *behind* the tunnel: the VPN authenticates the device, the enterprise authenticates the user.

### 1.4 The Overview Table — Aspect / Description

| Aspect | Description |
|---|---|
| **Definition** | Tunnelled private connectivity over public networks — encapsulation + encryption + authentication (✅ RFC 4301, the current IPsec architecture: "security services for traffic at the IP layer"). |
| **What it provides** | Confidentiality (ciphertext on the wire), integrity (tamper evidence), mutual authentication (both ends verified), replay protection. |
| **What it does NOT provide** | Application-level authorisation — a VPN admits to a *network*; what the user may *do* is left to downstream controls (firewalls, AD, per-app policy). This is precisely the gap ZTNA closes (§6.2). |
| **Trust model** | Implicit-trust perimeter: once inside the tunnel, the user is "inside the network" (the assumption zero trust abolishes — see [Zero Trust Network Architecture](zero_trust_network_architecture_guide.md) §2). |
| **The transport** | Any IP network — the internet, MPLS, LTE/5G, satellite. The tunnel does not care what it rides over. |
| **The classic topology** | Two tunnel endpoints: a client (laptop/phone) or a site gateway (branch router/firewall) at one end; a concentrator/gateway at the other (the head-office edge, a cloud gateway, a provider PoP). |
| **The economic proposition** | Private-network capability at public-network prices — cryptography replaces leased lines and dial infrastructure (the original 1996 PPTP pitch, §2.1). |
| **The modern tension** | Same tunnel, new threat model: edge gateways are prime targets (§7.1), split-tunnel leaks are real (§7.2), and identity-first access is displacing network-first access (§6). |
| **The estate's answer** | Not either/or: VPN for what it is still best at (site-to-site, legacy apps, partner extranets) + ZTNA for the identity-first layer (§9.4). |

### 1.5 The Vocabulary — VPN, Tunnel, Virtual Interface

- **VPN** — the *service*: protected connectivity between endpoints over a public network, delivered by a protocol (or a product).
- **Tunnel** — the *logical channel*: the encapsulated, encrypted, authenticated path between two endpoints. A tunnel is a property of the packets (they carry the tunnel protocol's headers), not of the physical path.
- **Virtual interface** — the *local abstraction*: on the endpoint, the tunnel appears as a network interface (WireGuard's `wg0`, OpenVPN's `tun0`, the IPsec virtual interface) that the OS routes traffic into as if it were a real NIC. The OS does not know (or care) that the "wire" is cryptographic.
- **Concentrator / gateway / head-end** — the server side that terminates many tunnels (the glossary has the full list).

### 1.6 The VPN in the Cluster — Where This Guide Sits

The security cluster reads as a stack: [cybersecurity_guide.md](cybersecurity_guide.md) is the umbrella; [security_by_design_guide.md](security_by_design_guide.md) is the development discipline; [threat_modeling_guide.md](threat_modeling_guide.md) is the adversarial lens; [distributed_auth_guide.md](distributed_auth_guide.md) is the identity engine; [zero_trust_network_architecture_guide.md](zero_trust_network_architecture_guide.md) is the paradigm that replaced the perimeter; [beyond_zero_enterprise_security_guide.md](beyond_zero_enterprise_security_guide.md) is the paradigm's next step. This guide is the *connective tissue*: the thing the paradigm displaced (the VPN) examined honestly — what it is, how it works, why it broke, and where it still earns its keep. It deliberately sits between the protocol RFCs (primary sources) and the Cymbal Bank estate (worked example), because that is exactly where the tunnelled middle ground lives.

### 1.7 The Layered View — Where Each VPN Sits in the Stack

The protocols differ in *which layer of the stack they tunnel*, and the layer determines what can ride:

- **The layer-2 tunnels — PPTP and L2TP** (RFC 2637 and RFC 2661 ✅). They carry **PPP frames** — the dial-up-era layer-2 protocol — inside their encapsulation. Because they tunnel at layer 2, they can in principle carry non-IP payloads; in practice they carried IP inside PPP, and their security came (badly, for PPTP; properly, for L2TP/IPsec) from layers bolted around the PPP session.
- **The layer-3 tunnels — IPsec and WireGuard.** IPsec's ESP wraps **IP packets** (tunnel mode wraps the whole packet; transport mode protects only the payload — see the glossary); WireGuard likewise wraps IP packets, inside UDP datagrams (UDP is the transport that defeats NAT and keeps the design stateless). Layer-3 tunnelling means the tunnel interface looks like a normal network interface to the OS — routing, not streams — which is why these are the site-to-site workhorses.
- **The TLS-stream tunnels — OpenVPN and SSTP.** They ride inside a **TLS byte stream** (TCP 443, or UDP for OpenVPN): the tunnel is a stream protocol over a stream protocol. This buys universal firewall passage (443 is HTTPS) at the cost of stream semantics (and OpenVPN's TCP-over-TCP hazard if the transport is TCP, §3.2).

The practical consequences: layer-3 tunnels give the OS a routable interface (the modern pattern — WireGuard's `wg0`); TLS tunnels give the client a port that looks like HTTPS (the camouflage pattern — the SSL-VPN appliance's pitch); layer-2 tunnels give legacy framing (the retirement tier). The layer also sets the **overhead budget** (§4.6's MTU/MSS tax) and the **firewall profile** (ESP/500/4500 for IPsec, 443 for the TLS family, UDP any-port for WireGuard — which is why WireGuard deployments standardise on a fixed UDP port and why deep packet inspection of UDP-encrypted tunnels is a different problem than inspecting TLS).

---

## 2. The History

### 2.1 The Dial-Up Ancestry — PPTP and the 1996 Microsoft/Ascend Era

The VPN was born from a cost problem. In the mid-1990s, remote access meant dial-up: an employee dialled into a bank of modems at the office (RAS — remote access servers) and paid long-distance or 1-800 charges for the privilege. The internet had already shown that a packet could travel cheaply from anywhere to anywhere — so why not carry the private dial-up session *over* the internet instead of over the phone network?

The answer was the **Point-to-Point Tunneling Protocol (PPTP)**, developed by a vendor consortium led by **Microsoft and Ascend Communications** (RFC 2637, July 1999, lists Hamzeh of Ascend, Pall of Microsoft, Verthein of 3Com, Taarud, Little and Zorn — and the RFC's IESG Note records plainly that "the PPTP protocol was developed by a vendor consortium"). PPTP wrapped PPP sessions (the dial-up protocol) in an "enhanced GRE mechanism" (✅ RFC 2637) so that the PPP session — authentication, IP address assignment, the lot — could run across the internet to a PPTP Network Server at the office. Microsoft shipped PPTP client support in Windows 95 OSR2/Windows NT 4.0-era networking, and "VPN" entered the enterprise vocabulary as a Windows feature before it was a standards conversation. The 1996-era date is the widely recorded origin of the consortium work (⚠-structural — the RFC itself is July 1999, ✅, and the 1996 origin is documented in the protocol's secondary history but was not pinned to a single primary source this pass).

The catch, invisible at the time, was that PPTP's security was bolted on: the tunnel itself carried no encryption, so Microsoft layered **MPPE** (Microsoft Point-to-Point Encryption, built on the **RC4** stream cipher) and **MS-CHAP** authentication on top — a design whose weaknesses would be systematically dismantled later (§3.6).

### 2.2 The IETF Standardization — IPsec, RFC 2401, November 1998

Where PPTP was a vendor consortium's answer, **IPsec** was the IETF's. RFC 2401, *Security Architecture for the Internet Protocol*, by Stephen Kent (BBN) and Randall Atkinson (@Home Network), was published in **November 1998** (✅ verified on rfc-editor.org this pass) and defined a security architecture *at the IP layer itself*: every packet could carry authentication (AH) and/or encryption+authentication (ESP), governed by Security Associations (SAs) and keyed by the Internet Key Exchange (IKE). The ambition was total — IPsec would secure *all* IP traffic, not just remote access — and the architecture split into the three verbs done properly: ESP for confidentiality (RFC 2406, later RFC 4303), AH for authentication without encryption (RFC 2402, later RFC 4302), and IKE for the key management (RFC 2409, later IKEv2 in RFC 7296).

IPsec's standardization story is a saga of its own: it was technically comprehensive and operationally painful (the acronym soup — SA, SPD, SAD, IKE, ISAKMP, ESP, AH — plus the NAT-traversal problem, solved for ESP by **RFC 3948** in January 2005, which encapsulates ESP in UDP for NAT traversal ✅). The December 2005 refresh — RFC 4301 (architecture), RFC 4302 (AH), RFC 4303 (ESP), all by Kent and Seo of BBN (✅) — remains the current architecture, with IKEv2 standardized as **RFC 7296 in October 2014** (Internet Standard, STD 79, ✅). IPsec became the workhorse of the enterprise: site-to-site tunnels between firewalls, remote-access clients on every OS, and the security layer underneath L2TP (§3.4). Its weakness was never the cryptography; it was the complexity budget — dozens of options, algorithms, and modes to negotiate, and therefore dozens of ways to misconfigure.

### 2.3 The SSL-VPN Era — the 2000s and OpenVPN (2001)

The 2000s brought a counter-revolution in simplicity. If IPsec was the Swiss-army knife (many tools, many blades, easy to cut yourself), the **SSL VPN** was the single blade: reuse the one cryptographic protocol the entire web already ran — TLS — and the one port every firewall already opened — 443. No special IPsec client, no NAT-traversal dance, no IKE negotiation; just a TLS connection from a browser or a lightweight client to a gateway.

The open-source standard-bearer was **OpenVPN**, written by **James Yonan**. OpenVPN's own history pages record the project's start in **2001** (the vendor's "About" timeline: "2001 — First private release of OpenVPN Open Source Protocol", ✅ verified this pass) and the first *open-source* release in 2002 (✅ per OpenVPN's as-docs page: "started the OpenVPN open-source project in 2001 with the release of the first open-source version of OpenVPN in 2002"). The widely-cited May 2001 first-release date is ⚠ — it circulates in secondary sources but was not pinned to a primary source this pass. What is not in dispute: OpenVPN brought TLS-grade security to the open-source VPN world, runs over **UDP or TCP** (port 443 TCP being the firewall-camouflage classic), and its OpenSSL underpinnings made it the default choice for the self-managed estate for two decades. The SSL-VPN category also produced the commercial remote-access gateways — the Pulse/ Fortinet/ Ivanti appliances that dominate §7.1's CVE roll call — where the "SSL VPN" branding became the enterprise's default remote-access product line.

### 2.4 The WireGuard — 2016 Announcement to the Kernel 5.6 Merge (March 2020)

The 2010s produced a different kind of revolt: against complexity itself. **WireGuard**, created by **Jason A. Donenfeld**, was announced in 2016 (initial release 30 June 2016 — ✅ in the known-solid ledger; the protocol's design goals are set out in the NDSS 2017 paper, *WireGuard: Next Generation Kernel Network Tunnel*, ✅ verified this pass at ndss-symposium.org). Its thesis: a VPN should be ~4,000 lines of kernel code, not a negotiation protocol; the crypto should be fixed and modern (ChaCha20-Poly1305, Curve25519, BLAKE2s — no algorithm agility to misconfigure); and the configuration should be as simple as exchanging public keys.

The kernel story is the part that made it historic: WireGuard was **merged into the Linux kernel in version 5.6**, released on **29 March 2020** (✅ verified this pass via kernelnewbies.org's Linux 5.6 page — "Linux 5.6 has been released on Sun, 29 Mar 2020… This release adds Wireguard" — and the phoronix release announcement). Kernel-native meant WireGuard ran in the kernel's network stack with wire-speed performance, no userspace round-trips, and no OpenSSL dependency — and its quality bar (the paper describes a formal-verification-adjacent discipline and an audited crypto set) set a new standard for what "secure VPN" should mean. By 2026 it is the default protocol for the modern estate, the built-in VPN of the major Linux distributions, and the protocol of choice in §9's worked example.

### 2.5 The Modern Remote-Access Era — Cloud Gateways and the ZTNA Displacement

The post-2020 era has two currents. The first is **cloudification**: the gateway moved from the head-office DMZ to the cloud (managed VPN gateways, VPN-as-a-service, the gateway as a cloud PoP), because the workforce is no longer at the office and the apps are no longer in one data centre. The second is **displacement**: the zero-trust movement (see [Zero Trust Network Architecture](zero_trust_network_architecture_guide.md) §2 — Kindervag 2010, BeyondCorp 2014, NIST SP 800-207 in 2020) reframed remote access as an *identity and device* problem rather than a *network* problem. ZTNA brokers per-application access (the SDP pattern, §6.1); SASE/SSE folded the gateway's function into cloud-delivered security platforms (§6.3). The industry began saying "the VPN is dead" (the Gartner-adjacent claim, §6.5) — while the VPN quietly continued doing the jobs ZTNA does not do well: site-to-site, legacy protocols, partner extranets. That is the modern era in one sentence: the VPN lost the narrative and kept the workloads.

### 2.6 The History Table — Year / Event / Notes

| Year | Event | Notes |
|---|---|---|
| **1996** | PPTP developed by the Microsoft/Ascend-led vendor consortium | The dial-up-era origin of the enterprise VPN; Windows-native remote access (⚠-structural date; RFC 2637 itself is July 1999 ✅). |
| **1998** | **RFC 2401** — IPsec Security Architecture, November 1998 | Kent & Atkinson; the IETF's IP-layer security architecture (✅ rfc-editor.org). |
| **1999** | **RFC 2637** (PPTP, July 1999, Informational) and **RFC 2661** (L2TP, August 1999) | PPTP documented by its consortium; L2TP published as the IETF's standards-track successor (both ✅). |
| **2001** | **OpenVPN** project started (first open-source release 2002) | James Yonan; the TLS-based SSL-VPN standard-bearer (✅ openvpn.net; the May 2001 exact date ⚠). |
| **2001** | **RFC 3193** — Securing L2TP using IPsec, November 2001 | The fix for L2TP's lack of confidentiality: run it inside IPsec (✅). |
| **2005** | **RFC 3948** (UDP encapsulation of ESP, January 2005); **RFC 4301/4302/4303** (IPsec refresh, December 2005) | NAT traversal for ESP; the current IPsec architecture (✅). |
| **2000s** | The SSL-VPN era | Commercial remote-access gateways on port 443; the appliance category that becomes §7.1's target list. |
| **2012** | CrackPPTP — MS-CHAPv2/MPPE break demonstrated (⚠ — search backend returned nothing this pass; widely documented as Moxie Marlinspike & David Hulton's Defcon 20 talk) | The PPTP-obituary event; see §3.6. |
| **2014** | **RFC 7296** — IKEv2, October 2014 | IKEv2 becomes Internet Standard (STD 79) (✅). |
| **2016** | **WireGuard** introduced by Jason Donenfeld (initial release 30 June 2016) | The minimal, modern-crypto VPN (✅ known-solid; NDSS 2017 paper ✅). |
| **2017** | WireGuard paper at NDSS 2017 (27 February 2017) | *WireGuard: Next Generation Kernel Network Tunnel*, DOI 10.14722/NDSS.2017.23160 (✅). |
| **2019** | Gartner coins **SASE** — 30 August 2019, ID G00441737 | *The Future of Network Security Is in the Cloud*; metadata ✅ via public mirrors, full text paywalled ⚠. |
| **2020** | **WireGuard merged into Linux kernel 5.6**, released 29 March 2020 | Kernel-native VPN; the modern protocol default (✅ kernelnewbies.org). |
| **2021–2024** | The edge-gateway CVE era | Pulse (2021), Fortinet (2022), Ivanti (2024) — all ✅ NVD; the "edge device = prime target" pattern (§7.1). |
| **2020s** | The ZTNA displacement | Identity-first per-app access displaces network-first VPN at the edge (§6); the VPN survives for site-to-site, legacy, extranets (§6.5, §9.4). |

### 2.7 Reading the History Table — the Four Eras

The table reads as four eras, each with a lesson:

- **The dial-up replacement era (1996–1999)** — PPTP and L2TP: tunnels built to carry PPP cheaply over the internet. Lesson: *tunnels without real crypto are highways for attackers* (the PPTP lesson, §3.6).
- **The IETF standardization era (1998–2005)** — the IPsec architecture (RFC 2401), its fixes (RFC 3948's NAT traversal, RFC 4301/4302/4303's refresh), and the L2TP/IPsec combination (RFC 3193). Lesson: *the crypto can be right and the complexity still dangerous* (the misconfiguration surface, §3.1).
- **The SSL-VPN era (2001–2010s)** — OpenVPN and the commercial TLS gateways: simplicity and the 443 camouflage. Lesson: *simplicity wins adoption, then the appliance becomes the target* (the §7.1 roll call is the bill).
- **The modern era (2016–2020s)** — WireGuard's minimalism (fixed crypto, kernel-native, 2016 → kernel 5.6 in March 2020 ✅) and the ZTNA displacement (the paradigm's history is in the zero-trust guide's §2). Lesson: *the tunnel's job is shrinking to transport; the access decision moved to identity* (§6, §9.4).

---

## 3. The Protocols Head-to-Head

### 3.1 IPsec / IKEv2 — RFC 7296, 2014

**What it is.** The IETF's IP-layer security architecture (RFC 2401, November 1998 ✅; current architecture RFC 4301/4302/4303, December 2005 ✅). Traffic is protected per-packet by **ESP** (confidentiality + integrity + replay protection; RFC 4303) or, historically, **AH** (authentication only, no encryption; RFC 4302). Keys and Security Associations (SAs) are negotiated by **IKE** — version 2 standardized in **RFC 7296, October 2014** (Internet Standard STD 79, ✅ verified this pass), which cut IKEv1's complexity (fewer RFCs, built-in NAT traversal, EAP support for remote access, and a leaner exchange: IKE_SA_INIT + IKE_AUTH, four messages total).

**The crypto.** Modern deployments negotiate AES (CBC or GCM), SHA-2, and Diffie-Hellman (or ECDH) groups; the crypto is negotiated, which is both flexibility and attack surface — the classic misconfiguration is an estate still permitting 3DES or SHA-1 or weak DH groups. IKEv2 supports certificate-based mutual authentication, EAP (for user auth behind the machine auth), and **Perfect Forward Secrecy** via fresh DH exchanges.

**The role in 2026.** The enterprise workhorse for **site-to-site** (firewall-to-firewall tunnels — every major firewall vendor speaks it) and the legacy remote-access standard (native clients in Windows, macOS, iOS, Android, and every Linux desktop). Its weaknesses are operational: the option space invites misconfiguration, NAT traversal (RFC 3948 ✅) adds port-4500 complexity, and the IKE/ESP protocol family is heavy relative to WireGuard.

### 3.2 OpenVPN — TLS-Based, UDP/TCP, Port 443

**What it is.** The open-source SSL-VPN (James Yonan, project started 2001 ✅, first open-source release 2002 ✅; the widely-cited May 2001 exact date ⚠). The tunnel rides inside a **TLS** session — the same protocol family as HTTPS — which is exactly the point: TLS is universally implemented, firewall-friendly, and battle-tested. OpenVPN's own crypto layer (OpenSSL) provides the cipher agility; the protocol runs over **UDP** (default; a custom reliable layer over DTLS-like handling) or **TCP** (port 443, the firewall-camouflage classic — the traffic is indistinguishable from HTTPS to a port-based inspector).

**The crypto.** Whatever OpenSSL offers: typically AES-256-GCM for the data channel, TLS 1.2/1.3 for the control channel, certificate-based mutual auth, and optional tls-auth/tls-crypt for an additional HMAC layer. The flexibility is the double-edged sword — a 2018-era audit culture is full of OpenVPN configs with `cipher BF-CBC` and no PFS; modern guidance mandates AES-GCM and TLS 1.2+.

**The role in 2026.** Still the default for self-managed estates and the open-source community (the OpenVPN community edition remains the most-deployed open-source VPN software), and the engine under many commercial products. Its weaknesses: userspace performance (slower than kernel-native WireGuard), TCP-over-TCP meltdown if misconfigured (TCP tunnel inside TCP transport), and the config surface (the `.ovpn` file is a script, and scripts have bugs).

### 3.3 WireGuard — the Noise Framework, ChaCha20-Poly1305, Curve25519

**What it is.** The modern minimal VPN (Jason Donenfeld, announced 2016, initial release 30 June 2016 ✅; NDSS 2017 paper ✅; **merged into Linux kernel 5.6, released 29 March 2020** ✅). WireGuard is built on the **Noise protocol framework** (the IKpsk2 handshake pattern), uses a fixed, modern crypto suite — **ChaCha20-Poly1305** (AEAD), **Curve25519** (key exchange), **BLAKE2s** (hashing) — and has **no algorithm negotiation at all**: the primitives are the protocol. Keys are 32-byte Curve25519 public keys, which double as the peer identity; configuration is a handful of lines (interface + peers); roaming is free (the peer's current IP is just where the encrypted packets currently come from).

**The crypto.** Fixed suite (✅ all three primitives are WireGuard's documented choices in the NDSS paper): ChaCha20-Poly1305 AEAD for encryption+integrity, Curve25519 for the DH, BLAKE2s for hashing, with a 2^60 rekeying guard. The lack of cipher agility is a feature: nothing to downgrade, nothing to misconfigure, a crypto story an auditor can read in one paragraph.

**The role in 2026.** The modern default: kernel-native performance (a few thousand lines in the Linux network stack), built into the major Linux distributions, the protocol of choice for cloud gateways, Kubernetes CNI overlays, and the §9 worked example's modern clients. Its weaknesses are the flip side of minimalism: no built-in user authentication (the key *is* the identity — enterprise deployments bolt on an identity layer or key-distribution service), no native PKI/revocation story (key rotation is the operational discipline), and less mature management-plane tooling than the IPsec/OpenVPN ecosystems.

### 3.4 L2TP — RFC 2661, 1999, Usually L2TP/IPsec

**What it is.** The IETF's standards-track successor to PPTP (RFC 2661, **August 1999** ✅, Townsley/Cisco et al.). L2TP tunnels **PPP** packets across an intervening network (the RFC's abstract: "facilitates the tunneling of PPP packets across an intervening network in a way that is as transparent as possible to both end-users and applications"). Critically, **L2TP has no confidentiality of its own** — like PPTP it carries PPP, and PPP's own security is weak — so the only sane deployment is **L2TP/IPsec**: L2TP for the tunnel, IPsec ESP underneath for the encryption, standardized in **RFC 3193** (*Securing L2TP using IPsec*, November 2001 ✅).

**The crypto.** Whatever IPsec provides (the L2TP/IPsec combo inherits the IKE/ESP stack, §3.1). The L2TP layer itself adds an authentication-only control channel (a shared secret), which is why "L2TP without IPsec" is a historical curiosity and "L2TP/IPsec" is the deployment.

**The role in 2026.** The legacy remote-access option — native Windows/macOS/iOS/Android L2TP/IPsec clients made it the "no extra software" choice of the 2000s–2010s. Its weaknesses: double encapsulation overhead (PPP inside L2TP inside ESP — the MTU/MSS tax, §4.6), UDP 1701 + ESP/4500 port profile (often blocked on hostile networks), and the general complexity of the stack. Modern guidance: retire it in favour of IKEv2 or WireGuard unless a legacy client forces it.

### 3.5 SSTP — Microsoft's TLS-Based Entry

**What it is.** Microsoft's SSL-VPN answer: **Secure Socket Tunneling Protocol**, specified in the [MS-SSTP] Open Specifications (✅ verified this pass on learn.microsoft.com — "This protocol provides an encrypted tunnel (an SSTP tunnel) by means of the SSL/TLS protocol… the client first establishes a TCP connection to the SSTP server over TCP port 443"). SSTP transports **PPP traffic through an SSL/TLS channel** — the same design shape as OpenVPN (TLS on 443) but Windows-native and with the PPP machinery retained. It was introduced with **Windows Vista** (⚠-structural — the Vista-era introduction is widely documented but was not pinned to a primary source this pass).

**The crypto.** TLS (whatever the OS offers — SSL 3.0/TLS 1.0 in the original Vista-era design, TLS 1.2+ in modern Windows), with machine certificates for the server and user credentials (or certificates) for the client. Because it is TLS on 443, it shares OpenVPN's firewall-evasion property — the original commercial pitch was "VPN that works anywhere HTTPS works."

**The role in 2026.** The Windows-shop legacy option: native, zero-install on Windows, and still supported in modern Windows Server RRAS. Its weaknesses: Windows-centric (client support elsewhere is thin), PPP overhead, and TLS-on-TCP performance. It survives as the "we need a Microsoft-native TLS VPN" answer, not as a strategic choice.

### 3.6 PPTP-Legacy — RFC 2637, Broken: MPPE/RC4

**What it is.** The 1996-era pioneer (RFC 2637, July 1999 ✅) — PPP over enhanced GRE, no encryption of its own, with Microsoft's **MPPE** (RC4-based) and **MS-CHAPv2** bolted on for confidentiality and authentication. Both layers are broken:

- **MPPE/RC4** — the RC4 stream cipher with 40/56/128-bit keys and a weak key-scheduling story; RC4 has been publicly demolished (statistical biases, the Bar-Mitzvah/RC4-nom attacks) and is banned from modern TLS for the same reasons.
- **MS-CHAPv2** — the challenge-response authentication whose protocol design (the NT-hash derivation, the DES-based response construction) permits an **offline dictionary attack on the challenge/response pair**, meaning an attacker who captures one authentication exchange can recover the password without touching the server.

The public obituary was **CrackPPTP** — the 2012 demonstration (widely documented as Moxie Marlinspike and David Hulton's Defcon 20 talk, "Divide and Conquer: Cracking MS-CHAPv2 with a 100% success rate", ⚠ — the search backend returned nothing for it this pass, so the attribution is flagged; the underlying MS-CHAPv2 weakness is itself ⚠-structural, documented across the security literature for a decade).

**The role in 2026.** None, except as a liability to retire: every modern audit checklist (see §7.4) begins with "no PPTP anywhere." The protocol's own RFC is Informational (✅ RFC 2637 status), the IESG Note in it points to L2TP as the standards-track replacement — and the replacement's security lesson (§3.4) was "never tunnel without IPsec underneath." PPTP's history is the history of the field: the first generation built tunnels, the second generation learned that tunnels without real crypto are just highways for attackers.

### 3.7 The Comparison Table — Protocol / Security / Performance / Ecosystem / Maturity

| Protocol | Security | Performance | Ecosystem | Maturity |
|---|---|---|---|---|
| **IPsec/IKEv2** (RFC 7296, 2014 ✅) | Strong when correctly configured (AES-GCM, SHA-2, PFS); option space invites misconfiguration ⚠-structural | Hardware-accelerated (AES-NI); kernel or NIC offload; heavier handshake | Every OS, every firewall vendor, every cloud; the site-to-site lingua franca | Internet Standard (STD 79); the 1998→2014 RFC lineage |
| **OpenVPN** (2001 ✅) | Strong (TLS 1.2+/AES-GCM, cert auth); config-dependent | Userspace; slower than kernel-native; TCP-over-TCP hazard if misconfigured | Huge open-source community + commercial products; all OSes | Two decades of production; the SSL-VPN default |
| **WireGuard** (2016/2020 ✅) | Strong and *minimal* — fixed ChaCha20-Poly1305/Curve25519/BLAKE2s, no negotiation to attack | Kernel-native; fastest of the four; free roaming | Linux/macOS/Windows/BSD/Android/iOS clients; cloud and Kubernetes integrations; growing management tooling | Kernel 5.6 since March 2020; the modern default, but a young management plane ⚠-structural |
| **L2TP/IPsec** (RFC 2661/3193 ✅) | As strong as the IPsec underneath; nothing of its own | Double encapsulation overhead (PPP in L2TP in ESP); MTU tax | Native clients on all major OSes (legacy); shrinking | 1999-era standard; in retirement |
| **SSTP** (Vista-era ⚠) | Strong (TLS); Windows-centric | TLS-on-TCP; PPP overhead | Windows-native only, effectively | Microsoft-supported legacy |
| **PPTP-legacy** (RFC 2637 ✅) | **Broken** — MPPE/RC4, MS-CHAPv2 offline crack (⚠-structural; CrackPPTP 2012 ⚠) | Fast, irrelevant | Everywhere historically; nothing modern should enable it | Obsolete; retire-on-sight |

### 3.8 The Crypto Table — the Primitives per Protocol

| Protocol | Key exchange / handshake | Confidentiality | Integrity / auth | Status |
|---|---|---|---|---|
| **IPsec/IKEv2** | IKEv2 DH/ECDH (RFC 7296 ✅) with PFS | AES-GCM/CBC (negotiated) | HMAC-SHA2 or AEAD; ESP (RFC 4303 ✅) | ✅ modern when policy-enforced; ⚠ legacy ciphers (3DES/SHA-1) still configurable |
| **OpenVPN** | TLS handshake over OpenSSL; cert auth | AES-256-GCM (modern config) | AEAD or HMAC; tls-crypt option | ✅ modern when configured; ⚠ config-dependent (legacy ciphers common in old estates) |
| **WireGuard** | Noise IKpsk2; Curve25519 (✅ NDSS 2017) | ChaCha20-Poly1305 (✅) | ChaCha20-Poly1305 AEAD; BLAKE2s (✅) | ✅ fixed suite, nothing to downgrade |
| **L2TP/IPsec** | Inherits IKE from IPsec | Inherits ESP | Inherits ESP | ✅-as-IPsec; L2TP itself adds none ⚠-structural |
| **SSTP** | TLS (server cert; client cert/user) | TLS ciphers | TLS | ✅-as-TLS; ⚠ Windows-centric legacy |
| **PPTP-legacy** | MS-CHAPv2 challenge-response | **MPPE over RC4** | None of substance (PPP checksums only) | ⚠ **broken** — offline-crackable auth, weak cipher (CrackPPTP 2012 ⚠) |

### 3.9 The Protocol Selection Checklist — a Decision Procedure

The head-to-head compresses into a decision procedure the §9 design actually uses:

1. **Is it site-to-site between infrastructure we do not fully control?** (branch hardware, partner gateways, cloud VPN endpoints) → **IPsec/IKEv2**. Interoperability is the requirement; every firewall vendor speaks it (§3.1).
2. **Is it a modern, managed client fleet?** (our laptops, our phones, our engineers) → **WireGuard**. Fixed crypto, kernel performance, roaming, MDM-provisioned keys (§3.3).
3. **Is it a Windows-only legacy estate that demands TLS-on-443?** → **SSTP** — but only as a bridge with a migration plan; it is the retirement tier (§3.5).
4. **Is there a legacy client that only speaks L2TP/IPsec?** → migrate to IKEv2 (native on every modern OS) or WireGuard; do not build new L2TP/IPsec (§3.4).
5. **PPTP?** → **never**. The checklist's first gate is "PPTP count must be zero" (§3.6, §7.4).
6. **Does the choice satisfy the crypto policy?** — IPsec: AES-GCM, SHA-2, PFS, minimum DH group, no 3DES/SHA-1/RC4; WireGuard: fixed suite accepted as-is; TLS family: TLS 1.2+ with modern ciphers (§3.8 as policy).
7. **What is the failure mode?** — every choice must pair with the client kill switch (§4.5) and gateway HA (§9.2); the protocol is not the availability story.

The checklist's real question is *which side of the workload the tunnel is on*: user-facing remote access is migrating to WireGuard-then-ZTNA; network-facing site-to-site stays on IPsec; and the crypto policy is the floor beneath both (§9.3 applies exactly this).

---

## 4. The VPN Architectures

### 4.1 Remote-Access vs Site-to-Site

The two architectural families differ in *who* the endpoints are and *what* the tunnel protects:

- **Remote-access VPN** — many individual clients (laptops, phones, contractor machines) tunnel into a **gateway/concentrator** at the edge of the protected estate. The user's device becomes, for policy purposes, a member of the office network. This is the workforce pattern (§5.1), the ZTNA-displacement zone (§6), and the §9 worked example's modern-client half.
- **Site-to-site VPN** — two (or more) *networks* tunnel to each other through gateways (firewalls, routers, dedicated appliances): branch → head office, office → cloud VPC, partner → extranet. The tunnel endpoints are infrastructure, the traffic is aggregate (every host behind site A can reach every host behind site B per policy), and the tunnel is typically long-lived and always-on. This is the pattern the "VPN is dead" narrative quietly exempts (§6.5) and the pattern banks still run for branches and vendors (§8).

The architectural difference drives everything downstream: remote-access needs user-scale authentication and per-user policy; site-to-site needs routing, redundancy, and throughput; the gateway estate (§4.6) serves both.

### 4.2 The Site-to-Site Patterns — IPsec Tunnel, GRE/IPsec, DMVPN

- **IPsec tunnel mode** — the classic: the whole IP packet is wrapped in ESP (tunnel mode wraps the entire original packet; transport mode protects only the payload — the glossary has both). Firewall-to-firewall IPsec (IKEv2, §3.1) is the default site-to-site pattern for branch offices and cloud VPN gateways, with dead-peer detection (DPD, glossary) and route-based or policy-based selection.
- **GRE/IPsec** — Generic Routing Encapsulation (RFC 2784-era) inside IPsec: GRE provides the multiprotocol, point-to-point tunnel and *its own* encapsulation (so multicast and non-IP protocols can ride), and IPsec provides the crypto. The classic combination for routing protocols (OSPF/BGP) across a WAN tunnel — you cannot easily run dynamic routing over plain IPsec policy tunnels, but you can over a GRE tunnel that happens to be encrypted by IPsec. ⚠-structural: the pattern is textbook (Cisco-era GRE-over-IPsec is the canonical reference design) but was not pinned to a single primary source this pass.
- **DMVPN** — Dynamic Multipoint VPN (Cisco's design, ⚠-structural): a hub-and-spoke *control* plane (the hub is a NHRP/next-hop-resolution server) with **direct spoke-to-spoke tunnels** established on demand — traffic between branches flows directly, not hair-pinned through the hub. The pattern that made large branch meshes affordable in the 2000s, still common in bank branch estates (§8.3).

### 4.3 Full-Tunnel vs Split-Tunnel — the Definitions

- **Full tunnel** — *all* client traffic (including internet-bound traffic) goes through the VPN: the client's default route points into the tunnel, the gateway is the client's exit to the internet. Maximum policy control (all traffic is inspectable, filterable, and logged by the estate), maximum protection against the hostile local network (the coffee-shop Wi-Fi sees only the tunnel), at the cost of latency (everything hair-pins through the office/cloud gateway) and gateway capacity.
- **Split tunnel** — only *estate-destined* traffic goes through the VPN; internet traffic exits via the client's local connection. Lower latency and gateway load, better user experience for bandwidth-heavy consumer traffic — at the cost of control: the client's internet traffic is not inspected by the estate, and (the dangerous variant) a compromised client can be used as a pivot into the estate with no visibility of its other activity (§7.2).

The modern answer is a **policy-driven middle**: split-tunnel *by destination* (estate apps via VPN, everything else local) with DNS handled correctly (§4.4) and posture checks on the client (§7.4) — or the ZTNA pattern where the "tunnel" is per-application anyway (§6.1).

### 4.4 The DNS and Routing Split — Where Split-Tunnel Bites

Split-tunnelling looks simple until DNS is involved. The estate's internal names (intranet, internal app servers) are resolved by internal DNS servers that the tunnel is supposed to protect — but if the client's DNS resolver is split incorrectly, the client leaks: internal names go to the local (attacker-controlled on hostile Wi-Fi) resolver, or internal DNS answers travel in the clear outside the tunnel. The classic failure modes:

- **DNS leak** — the client resolves internal hostnames via the local resolver (or the VPN's DNS split is misconfigured), revealing internal naming and allowing DNS hijacking of internal names.
- **Routing leak** — the split-tunnel route table is incomplete or the VPN drops, and traffic that should have gone through the tunnel exits locally (this is what the kill switch exists for, §4.5).
- **The IPv6 gap** — the tunnel carries IPv4 while the client's IPv6 traffic (often enabled by default) exits locally: a silent full bypass of the tunnel. ⚠-structural but notorious; every VPN hardening checklist includes "disable or tunnel IPv6."

### 4.5 The Kill Switch

The **kill switch** is the client-side guarantee that the tunnel is *the only* path: if the VPN connection drops (or is not yet established), the client blocks the protected traffic entirely instead of letting it fall through to the local network. In implementation terms: the client's firewall rules drop everything not destined for the VPN gateway / not routed through the tunnel interface, so a disconnection fails *closed* rather than *open*. It is the standard mitigation for the routing-leak failure mode above and a mandatory component of the modern-client baseline (the §9 worked example ships it on every managed endpoint). It is also the precise inverse of the enterprise's availability instinct — fail-closed costs productivity when the gateway is down — which is why gateway HA (§9.2) and the kill switch are designed as a pair. ⚠-structural: the term and the pattern are industry-standard client behaviour, documented across vendor docs, not attributable to one primary source verified this pass.

### 4.6 The Gateways and Clients

**The gateway (concentrator / head-end).** The termination point of many tunnels and the enforcement point of the estate's remote-access policy: it authenticates the tunnel (certificates, PSKs, identity), assigns or relays addressing, applies per-user/per-group policy (which networks/apps the tunnel may reach), logs, and integrates with the AAA stack (§5.2). Placement is architectural: the classic design puts the gateway in the **DMZ** (a screened subnet between the internet firewall and the internal network), with the rule set being "the gateway is the only thing that can pass traffic from the tunnel into the interior" — the §9.2 topology. The gateway is also the estate's highest-value attack target (§7.1).

**The clients.** The endpoint software that terminates the tunnel on the user's device:

- **Native/OS clients** — built-in IPsec/IKEv2, L2TP/IPsec, SSTP (Windows), WireGuard (all modern OSes) — zero-install but less policy-rich.
- **Vendor clients** — the gateway vendor's agent (or a universal agent such as the open-source WireGuard tools or OpenVPN Connect), which bundles the config, the kill switch, posture checks, and sometimes the ZTNA brokering (§6.1).
- **The invisible client** — the modern trend: the client disappears into the OS/endpoint-management agent (the same agent that does MDM/EDR also does the VPN/ZTNA), so the user has no VPN UI at all; the tunnel is just "how the laptop reaches the apps."

The **MTU/MSS tax.** Every encapsulation layer costs bytes (PPPoE 8, GRE 24, L2TP 40-ish, ESP ~22+, WireGuard 60, plus UDP/TCP headers), so the tunnel's maximum packet size is smaller than the LAN's 1500 — the classic symptom is "the VPN works but big transfers stall," fixed by clamping the MSS (the TCP maximum segment size) to fit the tunnel overhead. ⚠-structural: the numbers are standard networking knowledge, but no single source was verified this pass.

### 4.7 The Gateway Placement Patterns — On-Prem DMZ, Cloud, Provider Edge

Where the gateway lives is an architectural decision with three standard answers:

- **The on-prem DMZ** (the §9.2 design) — the gateway sits in a screened subnet at the estate's edge, behind the internet firewall, with a filtered path into the interior. Full control (hardware, config, logs, air-gap-able), full lifecycle burden (procurement, patching, capacity), and the bank's classic posture. The rule that makes it safe is the same rule as everywhere: *the gateway is the only thing that may pass tunnel traffic inward, and what it may reach is filtered* (§9.2).
- **The cloud gateway** — the termination point lives in the cloud, next to the cloud workloads (a managed VPN gateway service or a gateway VM in the VPC). The estate's apps moved to the cloud, so the tunnel follows them; the cloud provider's HA/scale replaces the on-prem pair. The DMZ rule becomes the cloud security group / network ACL rule set — same principle, different vocabulary.
- **The provider edge (SASE)** — the tunnel terminates at the security provider's PoP, and the security stack (SWG, CASB, ZTNA, FWaaS — §6.3) applies at that edge. Least operational burden, least control, and the end-state the §9.4 migration evaluates.

The selection rule: **the gateway should sit where the protected resources are**, and the "no lateral path from the tunnel" rule applies identically in all three. The §9 design starts on-prem (the bank's existing estate, the traders' latency requirements) and keeps the cloud/SASE options on the Phase-3 roadmap.

---

## 5. The Enterprise VPN

### 5.1 The Workforce Remote-Access Pattern

The enterprise pattern is a *layered* access decision, and the VPN is only one layer of it:

1. **The device layer** — the endpoint must be a known, managed, compliant device (MDM/UEM enrolment, EDR present, OS patched) before the tunnel is even offered. The VPN authenticates the *machine* (certificate), not just the user.
2. **The user layer** — the human must authenticate with something the estate controls: password + MFA (§5.3), typically validated against the identity provider via the AAA stack (§5.2).
3. **The tunnel layer** — the VPN establishes the protected path (protocol choice per §3, full/split per §4.3).
4. **The application layer** — inside the tunnel, the user reaches the estate's apps, where the *next* set of controls applies (AD/Entra group membership, RBAC — see [Distributed Auth](distributed_auth_guide.md) §4–§5).

The pattern's defining property is **network-first**: the tunnel is established before any application decision, and the user's blast radius is "everything the network allows from the tunnel" — which is exactly what the ZTNA model inverts (§6.1).

### 5.2 Certificates and AAA — the PKI Tie-In

The enterprise VPN is a heavy consumer of the identity estate:

- **Certificate-based machine auth** — each managed endpoint carries a device certificate (issued by the estate's PKI), and the gateway requires it during the tunnel handshake (IKEv2 certificate auth, OpenVPN cert auth, or the client-certificate layer of an SSL-VPN). This is what makes "the VPN knows this is a corporate laptop" true. The PKI mechanics — issuance, renewal, revocation (CRL/OCSP), key storage (TPM) — are the [Distributed Auth](distributed_auth_guide.md) guide's territory; the VPN is one of its biggest consumers.
- **AAA — Authentication, Authorization, Accounting** — the gateway front-ends the estate's identity stack: **RADIUS** (or TACACS+ for device admin, or SAML/OIDC in the modern SSL-VPN/ZTNA gateways) carries the authentication to the identity provider; the authorization (which groups/networks/apps the tunnel may reach) comes back as policy; the accounting (who connected, when, from where, how long, how much traffic) feeds the audit trail. The cross-ref is light by design: [distributed_auth_guide.md](distributed_auth_guide.md) is the deep-dive; this guide just maps the VPN onto it.
- **The practical consequence** — an enterprise VPN without a PKI story is an enterprise VPN with shared passwords and PSKs, which is an enterprise VPN with a breach in it. The certificate layer is not optional in a regulated estate (§8).

### 5.3 The MFA Layer — TOTP, WebAuthn/Push, Hardware Tokens

The tunnel authenticates the machine; **MFA authenticates the human**. The standard second factors, in escalating order of phishing resistance:

- **TOTP** — time-based one-time passwords (RFC 6238-style, the authenticator-app six digits). Widely deployed, cheap, and *phishable* (a real-time phishing page can relay the code — the classic AiTM relay). ✅-structural: RFC 6238 is the standard (not re-verified this pass; the mechanism is common knowledge).
- **WebAuthn/push** — the modern tier: push-approval (the "approve this login?" notification) and, better, **WebAuthn** (FIDO2) — the authenticator signs a challenge bound to the *origin*, so a phishing page cannot relay it. This is the phishing-resistant tier, and it is the tier the [Distributed Auth](distributed_auth_guide.md) guide treats as the modern default.
- **Hardware tokens** — the highest-assurance tier: smart cards / PIV cards and FIDO2 security keys, plus the bank-grade legacy (RSA SecurID-style one-time tokens, ⚠-structural vendor lineage). For traders and privileged users (§8.2) the hardware token is the norm, often combined with the machine certificate so that access requires *possession of the device, the token, and the passphrase*.

The enterprise rule: MFA is mandatory for every VPN login, and the MFA decision belongs to the identity provider (via the AAA path), not to the VPN product — so that the same MFA policy governs VPN, email, and the ZTNA broker (§6.1) alike. (⚠-structural: "MFA mandatory on VPN" is universal industry practice; the regulatory floor behind it — MAS TRM, NIST 800-63-style guidance — is covered in the cluster's guides, notably security_by_design_guide.md's regulatory section.)

### 5.4 The Enterprise Table — Control / Pattern / Notes

| Control | Pattern | Notes |
|---|---|---|
| **Device identity** | Machine certificate (PKI-issued, TPM-stored) required at tunnel handshake | Cross-ref [Distributed Auth](distributed_auth_guide.md) for the PKI mechanics; §5.2 |
| **User identity** | RADIUS/SAML/OIDC to the IdP; MFA mandatory (§5.3) | The AAA path; the same IdP feeds ZTNA later (§6.1) |
| **Posture** | Client posture check pre-connect (OS patch, EDR, disk encryption) | The pre-ZTNA version of continuous verification; §7.4 |
| **Policy** | Per-group network/app access via gateway policy; least-privilege by role | Split-tunnel rules per group (§4.3–§4.4) |
| **Accounting** | Gateway logs → SIEM; session records for audit | The audit trail the bank's op-risk function reads (§8) |
| **Failure mode** | Kill switch on the client (§4.5); HA on the gateway (§9.2) | Fail-closed on the client, fail-over on the gateway |

---

## 6. The VPN vs the Modern Alternatives

### 6.1 The ZTNA — Identity-First, Per-App Access

The zero-trust movement's answer to the remote-access problem is **Zero Trust Network Access**: instead of admitting the user to a *network* (the VPN's move), the ZTNA broker admits the user to *applications*, one at a time, after verifying identity, device posture, and policy on every request — the "never trust, always verify" lens applied to access. The architecture is the **Software-Defined Perimeter (SDP)** pattern from the [Zero Trust Network Architecture](zero_trust_network_architecture_guide.md) guide (§5.1 there): a controller brokers each connection; a connector inside the estate opens outbound-only paths to the application; the application's address is never revealed to the client; unauthorized clients see nothing (the "dark cloud"). The paradigm guide is the deep-dive — this section is the *face-off*.

The heavy cross-ref is deliberate: everything this section says about ZTNA is a compression of [zero_trust_network_architecture_guide.md](zero_trust_network_architecture_guide.md) — its §1 (the never-trust definition), §5 (the SDP architecture), §8 (the banking transformation), and §9 (the worked ZTNA design). The VPN guide's contribution is the *comparison* and the *coexistence* design (§9.4).

### 6.2 The Comparison — Network-Layer Access vs Identity/Device-Based Per-App Access

| Dimension | VPN (network-layer access) | ZTNA (identity/device-based per-app access) |
|---|---|---|
| **What is granted** | Membership of a network ("you are inside") | Access to specific applications ("you may use this app") |
| **When the check happens** | At tunnel establishment (then standing access) | On every request (continuous verification) |
| **What is checked** | Machine cert + user auth (+ MFA) | Identity + device posture + policy, per session and per request |
| **Blast radius** | The whole network reachable from the tunnel | Only the granted applications |
| **The target's visibility** | The apps are reachable (scannable) from inside the tunnel | The apps stay dark to unauthorized clients (the SDP property) |
| **Lateral movement** | Possible once inside the tunnel | Contained by per-app brokering |
| **The 2026 reality** | Still the workhorse for site-to-site, legacy apps, extranets (§6.5) | The default for new remote-access programmes (§9.4) |
| **Primary source** | This guide (§1–§4) | [Zero Trust Network Architecture](zero_trust_network_architecture_guide.md) §1, §5, §8–§9 |

### 6.3 SASE and SSE — the Gartner Coinage

**SASE (Secure Access Service Edge)** — the Gartner coinage of **30 August 2019**: *The Future of Network Security Is in the Cloud*, by Neil MacDonald, Lawrence Orans and Joe Skorupa (ID G00441737 — ✅ metadata verified this pass via public mirrors; the full Gartner text is paywalled, ⚠). The thesis: networking (SD-WAN) and security (SWG, CASB, ZTNA, FWaaS) converge into a single cloud-delivered service edge, delivered from provider PoPs, because the users, devices, and applications are no longer inside the enterprise — the *edge* is wherever the user is. **SSE (Security Service Edge)** is the security-only slice of the SASE stack (the networking/SD-WAN part removed), which is the slice most SSE/ZTNA vendors actually sell.

The structural relevance to this guide: in the SASE model the VPN gateway stops being an appliance in the DMZ and becomes a cloud PoP function — the tunnel terminates at the provider edge, and the security services (including ZTNA) apply at that edge. The §9 worked example treats SASE as the *end-state option* and the phased migration as the realistic path. ⚠-structural: the SASE-vs-SSE product taxonomy (which vendor sells which slice) is vendor-landscape material, not primary-source material.

### 6.4 The Beyond-Zero Paradigm

The [Beyond Zero Enterprise Security](beyond_zero_enterprise_security_guide.md) guide extends the zero-trust frame past the access decision: beyond-zero is the "zero-trust-plus" layer — continuous verification (not just per-session), identity-aware data planes, and the assumption that the perimeter is not merely untrusted but *actively hostile*. The VPN's place in that paradigm is precise: the tunnel survives as a *transport control* (encrypted, authenticated carriage of traffic) while losing its role as an *access control* (the decision moves to the identity/device layer). That is exactly the §9.4 design: WireGuard as the encrypted transport, ZTNA as the access decision, and the VPN's network-level admission restricted to the workloads that genuinely need network access (site-to-site, legacy apps, partners). Cross-ref [beyond_zero_enterprise_security_guide.md](beyond_zero_enterprise_security_guide.md) §6 for the paradigm's own treatment of legacy controls.

### 6.5 The "VPN Is Dead" Debate — the Reality

The claim "the VPN is dead" — in its Gartner-adjacent form (the 2019 SASE paper's framing that network-security design must shift from the data centre to the identity of the user/device) and its zero-trust form (BeyondCorp's "VPN-free access") — contains a true observation and a false conclusion. The true observation: as a *remote-access admission control*, the VPN grants too much (whole-network access after one authentication) and verifies too rarely (standing access after the handshake). The false conclusion: that the VPN as a *technology* disappears. The reality, in 2026:

- **The remote-access VPN is being displaced** at the user-facing edge by ZTNA/SASE — that is the §9.4 migration and it is real.
- **The site-to-site VPN is not going anywhere** — branch offices, cloud VPC interconnects, and partner extranets are still tunnels between gateways; ZTNA brokers *user-to-app* access, not *network-to-network* carriage. The financial industry's own extranet history (§8.1) is the proof that the tunnel-between-networks pattern predates and outlives the remote-access debate.
- **Legacy applications and protocols** (mainframe sessions, market-data feeds, file transfers, anything that is not HTTP) cannot be brokered per-app by the modern ZTNA products without an app-connector retrofit — the VPN (or a tunnel underneath the broker) remains the carriage.
- **The honest synthesis** — the one this guide's final word carries: the VPN is the **tunnelled middle ground**. Not the perimeter (that is gone — see the zero-trust guide's §10), not the decision point (that is identity — see the distributed-auth and beyond-zero guides), but the encrypted carriage underneath — the middle ground where the tunnel still earns its keep.

### 6.6 The Decision Rules — Keep the Tunnel, Broker the Apps

The estate's operational expression of §6.5, in five rules:

1. **User → web/HTTP workload** → the **ZTNA broker** (per-app admission; the SDP pattern — [Zero Trust Network Architecture](zero_trust_network_architecture_guide.md) §5). This is where the displacement is real and the blast-radius win is largest (§6.1).
2. **User → low-latency, legacy, or non-HTTP workload** (trading sessions, mainframe, file transfer, thick clients) → the **VPN as carriage** (WireGuard for the modern fleet, §9.3), because the broker cannot retrofit what is not HTTP without an app-connector programme (§6.5).
3. **Site → site** (branches, cloud VPCs) → **IPsec/IKEv2 site-to-site**, always — this is network-to-network carriage, which ZTNA does not do (§4.2, §8.3).
4. **Partner → network-level integration** → a **segregated IPsec partner tunnel** (own realm, no lateral path, time-boxed, audited — §8.4); **partner → app-level** → the broker.
5. **Anything PPTP** → retire on sight (§3.6).

The rules are the tunnelled middle ground made operational: the tunnel keeps the workloads that need carriage; the broker takes the workloads that need admission; and the crypto policy, the audit trail, and the kill switch are the floor under both (§7.4–§7.5, §9.3).

---

## 7. The Security Considerations

### 7.1 The Edge-Device Target Pattern — the CVE Roll Call

The defining security fact about VPNs in the 2020s: **the gateway is the highest-value, most-exposed device in the estate** — an always-on, internet-reachable appliance whose entire purpose is to admit remote traffic, running vendor firmware with a large attack surface. The pattern is not hypothetical; it is a roll call:

- **CVE-2021-22893 — Pulse Connect Secure** (✅ verified this pass, NVD): an authentication bypass in the Windows File Share Browser / Collaboration features allowing **unauthenticated remote code execution on the gateway**; CVSS 3.1 **10.0 Critical**; NVD records it as **"exploited in the wild"**; published 23 April 2021. The poster child: a VPN appliance compromised before any credentials are involved.
- **CVE-2022-40684 — Fortinet FortiOS / FortiProxy / FortiSwitchManager** (✅ NVD): an authentication bypass using an alternate path (CWE-288) allowing an **unauthenticated attacker to operate the administrative interface** via crafted HTTP/HTTPS requests; CVSS 3.1 **9.8 Critical**; published 18 October 2022. The admin interface of the security device itself, remotely.
- **CVE-2024-21887 — Ivanti Connect Secure / Policy Secure** (✅ NVD): a **command injection in the web components** of the gateway; CVSS 3.1 **9.1 Critical**; published 12 January 2024. The NVD description notes the authenticated-administrator precondition, and Ivanti's own advisory (referenced from the NVD entry) pairs it with **CVE-2023-46805**, an authentication bypass, which is how it was exploited in the wild as an unauthenticated chain (⚠-structural: the in-the-wild chaining is per vendor/CISA commentary, not restated in the NVD description itself).

The pattern across all three: *the VPN gateway is internet-facing by design, runs a large vendor web/management surface, and is breached at the application layer of the appliance — not through the tunnel crypto.* The tunnel cryptography (IPsec, TLS) held; the appliance around it did not. The security consequences for the estate are the ones this section is built around: the gateway must be treated as a hostile-facing service (§7.4's patch cadence), the tunnel's admission must not be the only control (ZTNA coexistence, §9.4), and a gateway breach must not equal a network breach (segmentation behind the gateway — the DMZ topology of §9.2).

**The attack lifecycle on an edge gateway** (⚠-structural — the stages are the standard incident-response framing, applied to this target class): *reconnaissance* (the gateway's TLS certificate and HTTP headers identify the product and version — banner grabbing is trivial), *exploit* (the vulnerable web/management surface — the three CVEs above are the canonical 2021–2024 examples), *persistence* (a webshell or backdoor account on the appliance), *pivot* (credentials harvested from the gateway's stores, or the tunnel itself used as the entry), and *dwell* (weeks to months of quiet presence before detection). Each stage maps to a mitigation: minimise the exposed surface (Pulse's File Share Browser feature *was* the 2021 vector — disable what is not needed), patch on a days-scale SLA (§7.4), watch the gateway's own logs and the SIEM feed (§7.4), and architect so that a gateway compromise is a DMZ incident, not a core-banking incident (§9.2). The pattern's deepest lesson: the gateway is an *edge service*, not a *trust anchor* — the estate's security cannot depend on the appliance that attackers are specifically paid to break.

### 7.2 The Split-Tunnelling Risks

Split-tunnel (defined in §4.3) trades control for experience, and the risk ledger is explicit:

- **The uninspected path** — the client's non-VPN traffic bypasses the estate's inspection stack entirely (no web filter, no DLP, no logging). In a regulated estate that traffic is a governance gap, not just a security gap.
- **The pivot risk** — a compromised client with split-tunnel has two lives: one inside the tunnel (estate access) and one outside (uncontrolled), and the estate sees only half the attacker's activity. The CVE roll call above shows attackers who reach the gateway; split-tunnel is the variant where the attacker *is* the client.
- **The leak mechanics** — DNS leaks, routing leaks, and the IPv6 gap (§4.4) can silently turn "split" into "bypassed."
- **The controls** — kill switch (§4.5), destination-based split policy (estate apps via tunnel only), DNS handling enforced by the client, IPv6 disabled-or-tunnelled, and posture checks before the split policy is applied (§7.4). The §9 design ships exactly this stack.

### 7.3 The Logging and Privacy — the No-Logs Claims

Two distinct logging questions are often conflated:

1. **The enterprise question** — *what does the estate log about its own VPN?* Answer: everything it needs for audit and incident response — who authenticated, from where, when, how long, what the tunnel carried (metadata at minimum; full session data if policy demands). The accounting leg of AAA (§5.2) exists precisely for this.
2. **The consumer question** — *what does a commercial VPN provider log?* The "no-logs" claim is a **commercial claim** ⚠: it is marketing language subject to jurisdiction, warrant-canary history, and independent-audit caveats. A provider's no-logs policy is a *contractual* statement, not a *cryptographic* one — the user cannot verify it from inside the tunnel. This guide's stance: for enterprise purposes, treat no-logs claims as unverifiable marketing (⚠) and engineer the estate's own logging; for consumer purposes, the privacy analysis belongs to a different guide (and a different threat model — the provider is a trusted party either way).

### 7.4 The Auditing — Config Reviews, Patch Cadence, Client Posture

The VPN estate is auditable, and the audit is the control:

- **Config review** — the gateway config is a security document: which protocols are enabled (PPTP must be absent — §3.6), which ciphers are permitted (no 3DES/SHA-1/RC4 — §3.8), which authentication methods are allowed (certificates + MFA only — §5.2–§5.3), which split-tunnel rules exist and why (§4.3), who administers the gateway and with what MFA (the CVE roll call's admin interfaces — §7.1). Review cadence: on every change and at least quarterly (⚠-structural: cadence norms are industry practice, not a cited standard this pass).
- **Patch cadence** — the gateway is the estate's most exposed device; it must be patched *first*, not last. The roll call's dates tell the story: Pulse disclosed April 2021, Fortinet October 2022, Ivanti January 2024 — each a race between the vendor advisory and the exploit-in-the-wild. The control is a **patch SLA measured in days for internet-facing appliances** (⚠-structural practice), with the vendor's security-advisory feed wired into the estate's ticketing, not the human reading it.
- **Client posture** — the endpoint must prove it is still a managed device: EDR present and reporting, OS patched, disk encryption on, no root/jailbreak, and — for the modern tier — the posture check repeated during the session (the continuous-verification idea from [Beyond Zero Enterprise Security](beyond_zero_enterprise_security_guide.md), applied to the VPN client before ZTNA takes over entirely).
- **The audit trail** — gateway logs (auth successes/failures, session metadata, admin actions) into the SIEM, with the op-risk function (cross-ref [Financial Risk & Compliance Systems](../banking/financial_risk_compliance_systems_guide.md)) owning the review. In the bank estate this is not optional hygiene; it is the evidence trail a regulator's examination will ask for (§8).

### 7.5 The Security Table — Risk / Mitigation / Status

| Risk | Mitigation | Status |
|---|---|---|
| **Gateway compromise** (Pulse/Fortinet/Ivanti class) | Patch SLA for internet-facing appliances; admin MFA; gateway in DMZ with no lateral path; treat gateway as hostile-facing (§7.1) | ✅ CVE facts verified (NVD); ⚠ the in-the-wild chain for Ivanti (vendor/CISA commentary) |
| **PPTP/legacy crypto** | Retire PPTP; policy-enforce modern ciphers (§3.6, §3.8) | ✅ RFC 2637 + MPPE/RC4 weakness ⚠-structural; CrackPPTP 2012 ⚠ |
| **Split-tunnel leaks** (DNS/routing/IPv6) | Destination-based split; DNS enforced; IPv6 disabled-or-tunnelled; kill switch (§4.4–§4.5) | ⚠-structural (industry practice) |
| **Standing access after auth** | MFA at connect; posture re-check; ZTNA for per-request verification (§5.3, §6.1) | ⚠-structural |
| **No-logs over-reliance** | Treat provider no-logs as commercial claim; own the estate's logging (§7.3) | ⚠ commercial claims |
| **Un-audited config drift** | Config review cadence; SIEM feed; op-risk ownership (§7.4) | ⚠-structural |

---

## 8. The Banking Context — the Cymbal Bank Estate

### 8.1 The Legacy Financial-VPN Layer — BT Radianz and the Extranets

The financial industry ran VPNs before the term was fashionable — as *extranets*: private networks interconnecting banks, brokers, exchanges, and market-data vendors, delivered historically over dedicated circuits and, since the 2000s, over managed IP-VPN services. The canonical name is **BT Radianz** (the managed financial-extranet platform that consolidated the industry's market-data and trading connectivity), and the pattern it represents — the *community of interest* extranet, where connectivity is by membership, not by individual tunnel — is the financial sector's own version of the tunnelled middle ground. The deep-dive on that layer lives in [Financial Infrastructure](../banking/financial_infrastructure_guide.md) (⚠-structural: the Radianz framing is verified in that guide's ledger, not re-verified this pass — cross-ref §8.1 there); this guide's point is the *shape*: for a bank, "VPN" includes both the commodity remote-access tunnels and the regulated financial extranets that carry market data and trade flows, and the two have different risk appetites, different SLAs, and different regulators looking at them.

### 8.2 The Traders — Low-Latency Remote Access

The trader is the bank's most demanding remote user: low-latency access to trading applications and market data, from home or a contingency site, with regulatory obligations (recording, surveillance, market-abuse controls) that apply to the remote session exactly as they apply on the desk. The estate's answer is the layered pattern of §5.1 at its strictest:

- machine certificate + hardware token MFA (§5.2–§5.3), because the account is worth phishing;
- full-tunnel or tightly destination-split access to the trading environment, because the session is surveilled and recorded;
- the gateway path engineered for latency (protocol choice matters — WireGuard's kernel performance and roaming, §3.3, is the modern answer; the legacy answer was dedicated circuits or QoS'd MPLS, §8.1);
- the ZTNA broker layered on top (§6.1) so that even inside the tunnel the trader reaches *the trading apps*, not the whole estate.

The banking frame for the region: [Banks in Singapore](../banking/banks_in_singapore_guide.md) is the estate context this section assumes — a regional HQ in Singapore with dealing floors, a MAS-regulated perimeter, and the standing remote-access obligations that the [Financial Risk & Compliance Systems](../banking/financial_risk_compliance_systems_guide.md) guide frames as op-risk.

### 8.3 The Branch Offices — Site-to-Site

The branch estate is the site-to-site pattern at scale: dozens of branches (and ATMs, and kiosks) each needing protected connectivity to the head office and to the regional data centres. The classic design is the §4.2 family: IPsec tunnels (or GRE/IPsec, or DMVPN for the mesh) terminated on branch firewalls/routers, with DPD and failover to a backup link, and the traffic carried over the cheapest available transport (broadband/4G) because the tunnel provides the security that the transport lacks. The bank-specific constraints: the branch link carries *banking* traffic (regulatory data, teller systems, cash-management endpoints), so the tunnel is not just encrypted but *audited* (the accounting leg, §5.4), and the branch gateway is a physical asset with its own lifecycle (hardware refresh, config management, remote administration — each of which the §7.4 audit covers).

### 8.4 The Third-Party and Vendor Access

The bank's perimeter is full of guests: core-banking vendors, payment-scheme operators, auditors, outsourced IT partners, market-data vendors. Each gets *some* access, and the VPN question is *how much network* that access implies. The legacy pattern — a vendor VPN account into the estate — is precisely the over-granting the zero-trust movement targets (the [Zero Trust Network Architecture](zero_trust_network_architecture_guide.md) guide's §8 treats this as the flagship transformation). The modern pattern: vendor access via a *brokered* path (ZTNA per-app, or at minimum a segregated VPN realm — separate gateway, separate realm, no lateral path into the banking network), time-boxed, MFA'd, and audited. The §9 design's partner site-to-site (IPsec/IKEv2 to the vendor's gateway) is the network-level variant for the vendors that genuinely need network-level integration; everything else moves to the broker.

### 8.5 The Banking Table — Pattern / Estate / Control

| Pattern | Estate | Control |
|---|---|---|
| **Trader remote access** | Dealing floor + home/contingency | Machine cert + hardware-token MFA; low-latency tunnel (WireGuard, §3.3); session surveillance; ZTNA for app-level admission (§6.1) |
| **Branch site-to-site** | Dozens of branches → HQ/regional DCs | IPsec/GRE-IPsec/DMVPN (§4.2); DPD + failover; audited tunnel traffic (§7.4) |
| **Vendor/third-party** | Core-banking vendors, partners, auditors | Segregated VPN realm or ZTNA broker; time-boxed; MFA; no lateral path (§8.4, §9.4) |
| **Financial extranets** | Market data, payments, exchanges | Managed financial-VPN layer (BT Radianz class) — cross-ref [Financial Infrastructure](../banking/financial_infrastructure_guide.md) §8.1 |
| **Legacy apps** | Mainframe, file transfer, non-HTTP | Tunnel as carriage underneath the broker (§6.5) — the §9.4 reason the VPN survives |

---

## 9. The Worked Example — an Enterprise VPN Design in the Cymbal Bank Style

### 9.1 The Scenario

**Cymbal Bank** (the fictional persona this guide's cluster uses — no real institution is described) runs a regional banking operation out of Singapore with a dealing floor, three regional offices, forty branches across Southeast Asia, and a growing remote workforce (traders at home, engineers, relationship managers). The estate's current remote access is a mix of legacy appliances (including an aged SSL-VPN gateway that the §7.1 roll call keeps the security team awake at night), L2TP/IPsec for the old laptops, and a handful of partner site-to-site tunnels. The mandate: **modernise the VPN estate without a rip-and-replace** — keep the workloads running, retire the legacy protocols, and build the on-ramp to ZTNA. The design below is the architecture this guide has been building toward; it is deliberately the *coexistence* design, not the greenfield one.

### 9.2 The Gateway Topology — the HA Pair in the DMZ

The gateways sit in the **DMZ** (the screened subnet between the internet-facing firewall and the interior), and they sit in **pairs**:

- **The DMZ placement** — internet traffic terminates on the external firewall → the gateway (in the DMZ) terminates the tunnels → the *only* permitted path from the DMZ into the interior is the gateway's interior interface, filtered by the interior firewall to the specific networks/apps the remote estate may reach. A gateway compromise (the §7.1 pattern) therefore lands the attacker in the DMZ with a filtered path, not on the core banking network. The gateway is not, and never is, inside the trusted interior.
- **The HA pair** — two gateways, active/standby (or active/active with sticky sessions), sharing a virtual IP, with health checking and state sync so that a gateway failure (or a patch cycle — §7.4's cadence) does not drop the estate's tunnels. The client-side counterpart is the kill switch (§4.5) failing *closed* on the client while the pair fails *over* on the server side; the two mechanisms are designed as a pair so that a gateway outage is a blip, not a lockout.
- **The split of duties** — the remote-access gateways (user tunnels) and the site-to-site termination (branch/partner tunnels, §8.3–§8.4) are logically separated: same DMZ, different policy domains, so a partner tunnel can never be used to reach the remote-access interior path and vice versa.

### 9.3 The Protocol Choice — WireGuard Plus IPsec/IKEv2

The protocol strategy is two protocols, chosen by workload — the head-to-head of §3 applied:

- **WireGuard for the modern clients** (✅ the §3.3 rationale): the remote workforce's managed laptops and phones run WireGuard — kernel-native performance (the traders' latency budget), fixed modern crypto (the auditors' checklist writes itself), simple key model (each device's Curve25519 key is its identity, distributed via the MDM/endpoint-management plane), free roaming (the trader moving from home Wi-Fi to 4G keeps the tunnel). The key-distribution gap (§3.3) is closed by the estate's MDM: the device keypair is provisioned with the device, and rotation is an MDM action.
- **IPsec/IKEv2 for the legacy and partner site-to-site** (✅ the §3.1 rationale): the branch tunnels and the partner extranets terminate as IPsec/IKEv2 — every firewall vendor speaks it (the branch hardware and the partners' gateways are not ours to choose), the certificate story maps onto the estate PKI (§5.2), and DPD/HA are mature. The legacy *remote-access* protocols — L2TP/IPsec, SSTP, and any PPTP remnants — are on the retirement list: L2TP/IPsec and SSTP clients migrate to WireGuard, and the §7.4 audit's first finding is "PPTP count must be zero."
- **The crypto policy** (the §3.8 table as policy): WireGuard's fixed suite is accepted as-is; the IPsec policy mandates AES-GCM, SHA-2, and PFS with a minimum DH group, and *denies* 3DES/SHA-1/RC4 by policy on both the gateways and the clients. The policy is the config review's ground truth (§7.4).

### 9.4 The ZTNA Coexistence — the Phased Migration

The design is not "VPN or ZTNA"; it is **VPN now, ZTNA layered, migration phased** — the §6.5 synthesis in three phases:

- **Phase 1 — harden the tunnels (months 0–3).** Deploy the HA gateway pair (§9.2), WireGuard for the modern clients (§9.3), retire PPTP, enforce the crypto policy, wire gateway logs into the SIEM (§7.4). This is the security-table (§7.5) as a work plan.
- **Phase 2 — broker the apps (months 3–12).** Stand up the ZTNA broker (the SDP pattern — see [Zero Trust Network Architecture](zero_trust_network_architecture_guide.md) §5 and §9 for the full design) for the web-based and HTTP workloads: the workforce's day-to-day apps (email, intranet, CRM, the portal) move to per-app brokering *first*, because that is where the blast-radius win is largest. The VPN remains the carriage for the rest — the trader's low-latency path, the legacy apps, the file transfers (§6.5's honest list).
- **Phase 3 — the tunnel becomes transport (months 12–24).** The remote-access VPN's *admission* role shrinks to the workloads ZTNA cannot broker; the tunnel becomes the encrypted carriage underneath (the beyond-zero framing of §6.4); the SASE option (the cloud-delivered edge of §6.3) is evaluated as the eventual consolidation of the gateway function into the provider edge. The site-to-site and partner tunnels (§8.3–§8.4) stay on IPsec throughout — they are network-to-network carriage, which is not what ZTNA brokers.

### 9.5 The Design Table — Component / Choice / Rationale

| Component | Choice | Rationale |
|---|---|---|
| **Gateway topology** | HA pair (active/standby, state-synced) in the DMZ; logical separation of remote-access vs site-to-site termination | The §9.2 topology: fail-over for availability, DMZ for blast-radius, separation for policy domains |
| **Modern client protocol** | **WireGuard** (kernel 5.6+ native; managed clients via MDM) | Fixed modern crypto, kernel performance, free roaming, simple key model (§3.3, §9.3) |
| **Legacy/partner protocol** | **IPsec/IKEv2** (RFC 7296) with AES-GCM/SHA-2/PFS policy | Interop with branch and partner hardware; certificate auth onto the estate PKI; mature HA/DPD (§3.1, §9.3) |
| **Retired protocols** | PPTP (zero tolerance), L2TP/IPsec and SSTP (migration path) | §3.6's broken crypto and §7.4's audit ground truth |
| **Authentication** | Machine certificates (PKI/TPM) + user MFA (WebAuthn, hardware tokens for traders) via RADIUS/SAML to the IdP | The §5.2–§5.3 layered pattern; same IdP feeds the ZTNA broker later |
| **Tunnel policy** | Destination-based split-tunnel with DNS enforced, IPv6 disabled-or-tunnelled, kill switch on every managed client | The §4.3–§4.5 risk ledger applied; fail-closed clients |
| **ZTNA coexistence** | SDP broker in Phase 2 for HTTP workloads; VPN admission shrinks to the non-brokerable set in Phase 3 | The §9.4 phased migration; the §6.5 synthesis |
| **Auditing** | Gateway logs + broker logs → SIEM; quarterly config review; patch SLA in days for gateways | The §7.4 discipline; the op-risk evidence trail (§8) |

### 9.6 The Lessons

1. **The gateway is a hostile-facing service.** The DMZ placement, the HA pair, the patch SLA, and the admin MFA are not hygiene — they are the direct response to the §7.1 roll call.
2. **Two protocols beat one.** WireGuard for the modern fleet, IPsec/IKEv2 for the world that must interoperate; the crypto policy is what makes the mix safe (§9.3).
3. **The VPN's future is transport, not admission.** Phase 3's shape — the tunnel as encrypted carriage under the ZTNA broker — is the honest end-state (§9.4, §6.5).
4. **The audit trail is the deliverable.** In a bank, the regulator's question is not "did you use WireGuard?" but "show me who connected, when, and to what" — the accounting leg is the answer (§7.4, §8).

---

## 10. The Summary — the One-Page VPN

### 10.1 The One-Page Summary

A VPN is tunnelled private connectivity over public networks — **encapsulation** (the private packet rides inside a public one), **encryption** (the ride is ciphertext), **authentication** (both ends are verified, every packet is intact). It was born in 1996 to replace dial-up (PPTP, Microsoft/Ascend), standardized by the IETF as IPsec (RFC 2401, November 1998; IKEv2 as RFC 7296 in 2014), democratized by the SSL-VPN era (OpenVPN, 2001; the 443-port gateway appliances), and reinvented by WireGuard (2016; merged into Linux kernel 5.6 on 29 March 2020) — the fixed-crypto, kernel-native tunnel that made the old complexity look indefensible. The protocols sort cleanly: IPsec/IKEv2 is the interop workhorse, OpenVPN the TLS-era default, WireGuard the modern choice, L2TP/IPsec and SSTP the legacy tier, PPTP the broken ancestor (MPPE/RC4, the 2012 CrackPPTP-class break). The architectures sort into remote-access vs site-to-site, full-tunnel vs split-tunnel (with DNS, IPv6, and the kill switch as the leak controls), and the gateway estate (HA pairs in the DMZ, §9.2). The enterprise hardens it with certificates and AAA (§5.2), MFA (§5.3), and audit discipline (§7.4). The modern alternatives — ZTNA (identity-first, per-app — see [Zero Trust Network Architecture](zero_trust_network_architecture_guide.md)), SASE/SSE (the 30 August 2019 Gartner coinage), and the beyond-zero paradigm ([Beyond Zero Enterprise Security](beyond_zero_enterprise_security_guide.md)) — displace the VPN at the user-facing edge, and the security record justifies it: the 2021–2024 gateway CVEs (Pulse 10.0, Fortinet 9.8, Ivanti 9.1 — all ✅ NVD) show the edge device as the prime target. Yet the VPN survives — for site-to-site, legacy apps, partner extranets, and as the encrypted carriage underneath the broker — which is why the Cymbal Bank design (§9) is a coexistence design: WireGuard + IPsec/IKEv2 tunnels, hardened and audited, with ZTNA layered on in phases until the tunnel's role is transport, not admission.

### 10.2 The Final Word — the Tunnelled Middle Ground

The VPN is not the perimeter — the perimeter is gone, and [Zero Trust Network Architecture](zero_trust_network_architecture_guide.md) buried it. The VPN is not the decision point — identity is, and [Distributed Auth](distributed_auth_guide.md) and [Beyond Zero Enterprise Security](beyond_zero_enterprise_security_guide.md) own that ground. What the VPN remains is the **tunnelled middle ground**: the encrypted carriage between the abolished perimeter and the identity-first edge — the place where site-to-site traffic still rides, where legacy applications still breathe, where partners still connect, and where the modern estate's access decisions are *transported* even when they are no longer *made*. Not dead, not sufficient, and — done right, with modern protocols, hardened gateways, and ZTNA layered above — still earning its keep. The tunnelled middle ground is where the Cymbal Bank estate, and every estate like it, actually lives.

---

## 11. The Glossary

- **Tunnel** — the logical, cryptographically protected channel between two endpoints, carried inside ordinary packets across an untrusted network; the core artefact of a VPN (§1.2).
- **Encapsulation** — wrapping one packet/frame inside another whose headers address the tunnel endpoints; the "virtual" in VPN (§1.3).
- **ESP / AH** — the two IPsec protocols: **ESP** (Encapsulating Security Payload, RFC 4303 ✅) provides confidentiality + integrity + replay protection; **AH** (Authentication Header, RFC 4302 ✅) provides authentication/integrity only, no encryption — the reason AH is rare in practice (§3.1).
- **IKE / IKEv2** — Internet Key Exchange: the protocol that negotiates keys and Security Associations for IPsec; version 2 is RFC 7296 (October 2014, STD 79 ✅) (§3.1).
- **SA (Security Association)** — the agreed security context between two IPsec peers (algorithms, keys, SPI, lifetimes); the unit of IPsec state (RFC 2401/4301 ✅).
- **MTU / MSS** — Maximum Transmission Unit (the largest packet a link carries) and Maximum Segment Size (the largest TCP payload); every encapsulation layer shrinks the effective MTU, and MSS clamping is the standard fix (§4.6).
- **Handshake** — the initial exchange that authenticates the peers and establishes key material; the IKE_SA_INIT/IKE_AUTH four-message exchange (IKEv2), the TLS handshake (OpenVPN/SSTP), the Noise IKpsk2 handshake (WireGuard).
- **Key exchange** — the cryptographic establishment of shared secrets (Diffie-Hellman/ECDH in IKE, Curve25519 in WireGuard); distinct from authentication, though the two are interleaved in the handshake.
- **Perfect Forward Secrecy (PFS)** — the property that a compromised long-term key cannot decrypt past sessions, because each session's keys come from an ephemeral exchange; mandated in the §9.3 crypto policy.
- **Kill switch** — the client-side fail-closed mechanism that blocks protected traffic when the tunnel drops, preventing routing leaks (§4.5).
- **Split tunnel** — the configuration where only estate-destined traffic rides the tunnel and internet traffic exits locally; full tunnel is the all-traffic alternative (§4.3–§4.4).
- **Gateway (concentrator, head-end)** — the server-side termination point for many tunnels and the estate's remote-access enforcement point; the §7.1 target and the §9.2 DMZ resident.
- **Client** — the endpoint software/OS component that terminates the tunnel on the user's device; native, vendor, or invisible (management-agent) forms (§4.6).
- **ZTNA** — Zero Trust Network Access: identity/device-based, per-application access brokering, the VPN's displacer at the edge ([Zero Trust Network Architecture](zero_trust_network_architecture_guide.md) §1, §5; this guide §6.1–§6.2).
- **SASE** — Secure Access Service Edge: the Gartner-coined (30 August 2019 ✅ metadata) convergence of networking and security into a cloud-delivered edge; **SSE** is its security-only slice (§6.3).
- **SDP** — Software-Defined Perimeter: the controller-brokered, dark-cloud access architecture ZTNA is built on (§6.1; the zero-trust guide's §5.1).
- **PFS** — see Perfect Forward Secrecy (glossary is alphabetical; PFS is the acronym that wins).
- **DPD** — Dead Peer Detection: the keepalive mechanism by which an IPsec peer detects that its counterpart is gone and triggers failover; the site-to-site HA pattern's pulse (§4.2, §8.3).
- **DMVPN** — Dynamic Multipoint VPN: the hub-controlled, spoke-to-spoke-on-demand mesh pattern (§4.2).
- **GRE** — Generic Routing Encapsulation: the generic tunnelling protocol that PPTP (enhanced GRE, RFC 2637 ✅) and GRE/IPsec site-to-site designs ride on (§4.2).
- **PPP** — Point-to-Point Protocol: the dial-up-era framing/authentication protocol that PPTP and L2TP carry inside their tunnels (RFC 2637, RFC 2661 ✅).
- **MPPE / MS-CHAPv2** — Microsoft Point-to-Point Encryption (RC4-based) and the MS-CHAPv2 challenge-response authentication: the broken security layers of PPTP (§3.6).
- **RADIUS** — the AAA protocol by which gateways hand authentication to the estate's identity provider (§5.2).
- **No-logs** — the commercial claim that a VPN provider retains no session data; treated in this guide as unverifiable marketing for enterprise purposes (§7.3 ⚠).
- **Posture** — the endpoint's compliance state (patched, EDR'd, encrypted) checked before and during access (§7.4).
- **AEAD** — Authenticated Encryption with Associated Data: encryption and integrity in one primitive (AES-GCM, ChaCha20-Poly1305); the modern cipher form every current protocol uses (§3.8).
- **NAT traversal (NAT-T)** — the mechanism (RFC 3948 ✅ for ESP; UDP for WireGuard) by which tunnels survive network-address translation; the reason ESP rides in UDP on port 4500 (§3.1, §1.7).
- **SPI** — Security Parameters Index: the identifier in the ESP/AH header that tells the receiver which Security Association a packet belongs to (RFC 4303 ✅).
- **Cipher agility** — the ability to negotiate algorithms; a strength (IPsec's flexibility) and a weakness (downgrade and misconfiguration) that WireGuard's fixed suite deliberately abolishes (§3.3, §3.8).
- **Roaming** — the tunnel's ability to survive the client changing networks (Wi-Fi → 4G) without re-handshaking; WireGuard's free-roaming property, valuable for the trader on the move (§3.3, §8.2).
- **VTI** — Virtual Tunnel Interface: the routable interface an IPsec or WireGuard tunnel presents to the OS (the modern route-based site-to-site pattern's plumbing, §4.2).
- **tls-crypt / tls-auth** — OpenVPN's optional extra HMAC layer that hides the TLS handshake itself from passive observers (§3.2).

---

## Claims Status and Verification Notes

**Verified this pass (live web, 2026-08-26):**

- ✅ **RFC 2401** — *Security Architecture for the Internet Protocol*, Kent & Atkinson, November 1998, Proposed Standard, obsoleted by RFC 4301 (rfc-editor.org, extracted).
- ✅ **RFC 7296** — *IKEv2*, October 2014, Internet Standard (STD 79), obsoletes RFC 5996; authors Kaufman (Microsoft), Hoffman (VPN Consortium), Nir (Check Point), Eronen, Kivinen (rfc-editor.org).
- ✅ **RFC 3948** — *UDP Encapsulation of IPsec ESP Packets*, January 2005 (rfc-editor.org).
- ✅ **RFC 2637** — *PPTP*, July 1999, Informational; Hamzeh (Ascend), Pall (Microsoft), Verthein (3Com) et al.; IESG note confirms the vendor consortium; enhanced GRE confirmed (rfc-editor.org).
- ✅ **RFC 2661** — *L2TP*, August 1999, Standards Track (rfc-editor.org).
- ✅ **RFC 3193** — *Securing L2TP using IPsec*, November 2001 (rfc-editor.org).
- ✅ **RFC 4301 / 4302 / 4303** — the December 2005 IPsec architecture/AH/ESP refresh, Kent (and Seo) of BBN (rfc-editor.org).
- ✅ **WireGuard paper** — Donenfeld, *WireGuard: Next Generation Kernel Network Tunnel*, NDSS 2017, 27 February 2017, DOI 10.14722/NDSS.2017.23160 (ndss-symposium.org; the PDF is public). **Brief correction: arXiv 2012.05071 is NOT the WireGuard paper** — direct arXiv extraction shows that ID is *The Gossip Paradox* (q-bio, submitted 8 Dec 2020). The whitepaper's canonical home is NDSS 2017.
- ✅ **Linux kernel 5.6** — released 29 March 2020 with WireGuard merged (kernelnewbies.org Linux_5.6 page + phoronix release announcement).
- ✅ **WireGuard primitives** — ChaCha20-Poly1305, Curve25519, BLAKE2s, Noise framework (per the NDSS 2017 paper's design; the known-solid list).
- ✅ **OpenVPN origins** — project started 2001 ("2001 — First private release", openvpn.net About page); first open-source release 2002 (openvpn.net/as-docs).
- ✅ **SSTP** — Microsoft [MS-SSTP] Open Specifications: encrypted tunnel via SSL/TLS over TCP 443, PPP transport (learn.microsoft.com).
- ✅ **CVE-2021-22893** (Pulse Connect Secure) — auth bypass → unauthenticated RCE; CVSS 10.0 Critical; "exploited in the wild"; NVD published 23 Apr 2021 (nvd.nist.gov).
- ✅ **CVE-2022-40684** (Fortinet FortiOS/FortiProxy/FortiSwitchManager) — auth bypass via alternate path (CWE-288), unauthenticated admin-interface operations; CVSS 9.8 Critical; NVD published 18 Oct 2022 (nvd.nist.gov).
- ✅ **CVE-2024-21887** (Ivanti Connect Secure/Policy Secure) — command injection in web components; CVSS 9.1 Critical; NVD published 12 Jan 2024; NVD references Ivanti's advisory pairing it with CVE-2023-46805 (auth bypass) (nvd.nist.gov).
- ✅ **Gartner SASE coinage metadata** — *The Future of Network Security Is in the Cloud*, MacDonald/Orans/Skorupa, 30 August 2019, ID G00441737 (verified via public mirrors — scribd reprint metadata and theregister whitepaper mirrors).
- ✅ **Repo facts** — sibling and banking/management filenames verified by direct file listing.

**Flagged ⚠ / ⚠-structural (honest gaps):**

- ⚠ **arXiv 2012.05071 ≠ WireGuard** — the commissioning brief's identifier is wrong (it is a q-bio paper); corrected to the NDSS 2017 proceedings. No arXiv ID for the WireGuard paper is asserted.
- ⚠ **CrackPPTP / Defcon 20 2012** — the search backend returned empty twice for the Marlinspike/Hulton MS-CHAPv2 break; the claim is written with attribution flagged. The underlying MPPE/RC4 and MS-CHAPv2 weaknesses are ⚠-structural (decade of security literature).
- ⚠ **OpenVPN "May 2001" first release** — the exact date circulates in secondary sources; the vendor's own pages say project 2001, first open-source release 2002. The May 2001 date is not asserted as verified.
- ⚠ **PPTP 1996 origin date** — recorded as ⚠-structural; RFC 2637 (July 1999) itself is ✅.
- ⚠ **SSTP Vista-era introduction** — ⚠-structural (secondary sources); the protocol spec itself is ✅.
- ⚠ **Gartner SASE full text** — paywalled; only metadata (title/date/ID/authors) verified via mirrors. The SASE definitional content in §6.3 is ⚠-structural.
- ⚠ **VPN provider no-logs claims** — commercial claims; not verifiable from inside the tunnel; treated as marketing for enterprise purposes (§7.3).
- ⚠ **Ivanti in-the-wild chain (CVE-2024-21887 + CVE-2023-46805)** — per vendor advisory and CISA commentary referenced from the NVD entry, not restated in the NVD description itself.
- ⚠ **GRE/IPsec and DMVPN descriptions** — ⚠-structural (textbook Cisco-era reference designs, not pinned to a single primary source this pass).
- ⚠ **Kill switch, MTU/MSS figures, patch-cadence norms, MFA-mandatory practice** — ⚠-structural industry practice, no single primary source verified this pass.
- ⚠ **BT Radianz framing** — cross-ref [Financial Infrastructure](../banking/financial_infrastructure_guide.md)'s ledger rather than re-verified this pass.
- ⚠ **Cymbal Bank** — a fictional worked example; no real bank's systems are described.

---

## Cross-References and Further Reading

**Siblings (technology/, plain filenames):** [zero_trust_network_architecture_guide.md](zero_trust_network_architecture_guide.md) — THE paradigm companion: the ZTNA definition (never trust, always verify), the SDP architecture, and the worked ZTNA design this guide's §6 and §9.4 coexist with. [beyond_zero_enterprise_security_guide.md](beyond_zero_enterprise_security_guide.md) — THE beyond-zero companion: continuous verification and the identity-aware layer the VPN's transport role plugs into (§6.4). [distributed_auth_guide.md](distributed_auth_guide.md) — the identity/AAA deep-dive the enterprise VPN consumes (certificates, RADIUS/SAML/OIDC, MFA) (§5). [cybersecurity_guide.md](cybersecurity_guide.md) — the cluster umbrella. [security_by_design_guide.md](security_by_design_guide.md) — the secure-development discipline for the estate around the tunnels (§7). [threat_modeling_guide.md](threat_modeling_guide.md) — STRIDE against the VPN gateway as an edge target (§7.1).

**Banking (../banking/ prefix):** [banks_in_singapore_guide.md](../banking/banks_in_singapore_guide.md) — the SG estate frame for §8. [financial_infrastructure_guide.md](../banking/financial_infrastructure_guide.md) — the legacy financial-extranet/VPN layer (BT Radianz and the financial VPNs) (§8.1). [financial_risk_compliance_systems_guide.md](../banking/financial_risk_compliance_systems_guide.md) — the op-risk framing of a gateway breach and the audit-trail owner (§7.4, §8). The bank software-systems series (HSBC, UOB, OCBC, Standard Chartered…) for remote-access estate precedents.

**Management (../management/ prefix):** [business_case_development_guide.md](../management/business_case_development_guide.md) — the VPN-modernisation investment angle (cost of an edge breach vs cost of the ZTNA migration; §6 and §9).

**Primary sources (all verified this pass unless flagged):** RFC 2401 (Nov 1998) ✅ · RFC 7296 (Oct 2014) ✅ · RFC 3948 (Jan 2005) ✅ · RFC 2637 (Jul 1999) ✅ · RFC 2661 (Aug 1999) ✅ · RFC 3193 (Nov 2001) ✅ · RFC 4301/4302/4303 (Dec 2005) ✅ — all rfc-editor.org · Donenfeld, *WireGuard: Next Generation Kernel Network Tunnel*, NDSS 2017 ✅ · Linux 5.6 release, 29 Mar 2020 (kernelnewbies.org) ✅ · openvpn.net history pages ✅ · Microsoft [MS-SSTP] ✅ · NVD CVE-2021-22893 / CVE-2022-40684 / CVE-2024-21887 ✅ · Gartner SASE metadata (public mirrors) ✅ / full text ⚠ · CrackPPTP 2012 ⚠ (backend empty).

**The canonical document stack (the five to read if you read five):** (1) RFC 2401 (or its 2005 successor RFC 4301) — the IPsec architecture that defined "security services for traffic at the IP layer"; (2) RFC 7296 — IKEv2, the modern key-management standard; (3) the WireGuard NDSS 2017 paper — the minimal-VPN thesis and the crypto rationale, ~20 pages; (4) RFC 2637's IESG Note plus §3.6 of this guide — the PPTP lesson (vendor tunnels without real crypto); (5) the zero-trust cluster's NIST SP 800-207 treatment in [zero_trust_network_architecture_guide.md](zero_trust_network_architecture_guide.md) §3 — the paradigm that decides the VPN's remaining role.

**Series note:** this guide completes the security cluster's remote-access arc: the paradigm ([zero_trust_network_architecture_guide.md](zero_trust_network_architecture_guide.md)), the identity engine ([distributed_auth_guide.md](distributed_auth_guide.md)), the beyond-zero layer ([beyond_zero_enterprise_security_guide.md](beyond_zero_enterprise_security_guide.md)), and now the protocol/transport deep-dive itself. It stands as the cluster's VPN reference: the RFCs to argue with, the crypto tables to audit with, the architecture patterns to build with, and the Cymbal Bank design to adapt — because in 2026, the perimeter is gone, the decision has moved to identity, and the tunnel remains: the tunnelled middle ground where the estate's traffic actually rides.
