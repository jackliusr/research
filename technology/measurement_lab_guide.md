# Measurement Lab (M-Lab) — The Open Internet-Measurement Platform

Jack Liu Shurui, Solution Architect

> **Author:** Jack Liu Shurui — Solution Architect, Cymbal Bank
> **Topic:** Measurement Lab (M-Lab) — the open internet-measurement platform: the organization and its consortium model; the global server infrastructure (physical sites, Google Cloud virtual sites, host-managed nodes); the NDT test and the rest of the current roster; the open-data model on Google BigQuery and Google Cloud Storage; research, regulatory and consumer usage; the documented limitations; the Singapore server presence; and a Cymbal Bank branch-connectivity monitoring worked example
> **Audience:** Solution Architects, Network and Connectivity Engineers, Data Platform Engineers, Observability Teams, Researchers and Policy Analysts, Banking IT Decision-Makers
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Version:** 1.0 — August 2026

---

# Table of Contents

1. [Executive Summary — The Open Internet-Measurement Platform](#1-executive-summary--the-open-internet-measurement-platform)
2. [The Organization — Origins, Governance, Mission](#2-the-organization--origins-governance-mission)
3. [The Platform — Global Measurement Infrastructure](#3-the-platform--global-measurement-infrastructure)
4. [The Tests — The Current Roster and the Retired Fleet](#4-the-tests--the-current-roster-and-the-retired-fleet)
5. [NDT in Depth — Protocols, Metrics, and Methodology](#5-ndt-in-depth--protocols-metrics-and-methodology)
6. [The Data — The Open-Data Model](#6-the-data--the-open-data-model)
7. [Access — APIs, BigQuery, and Tooling](#7-access--apis-bigquery-and-tooling)
8. [The Usage — Research, Policy, and Consumers](#8-the-usage--research-policy-and-consumers)
9. [Limitations — Bias, Attribution, and Coverage](#9-limitations--bias-attribution-and-coverage)
10. [The Singapore Angle — SIN Sites and Local Context](#10-the-singapore-angle--sin-sites-and-local-context)
11. [Cymbal Bank Worked Example — Branch Connectivity Monitoring](#11-cymbal-bank-worked-example--branch-connectivity-monitoring)
12. [Claims Audit — Verified, Flagged, Unverified](#12-claims-audit--verified-flagged-unverified)
13. [What Could Not Be Verified](#13-what-could-not-be-verified)
14. [Glossary](#14-glossary)
15. [Cross-References and Further Reading](#15-cross-references-and-further-reading)

---

## 1. Executive Summary — The Open Internet-Measurement Platform

### Purpose of This Guide

This guide is the repository's **platform deep-dive on Measurement Lab (M-Lab)**: the open, distributed internet-measurement platform that has, since 2009, collected and published what it calls the largest open dataset of internet performance in the world. It is written for architects who need to understand M-Lab at three levels:

- **What M-Lab is** — the organization (a not-for-profit, community-run consortium fiscally sponsored by Superbloom, founded by New America's Open Technology Institute, the PlanetLab Consortium, Google, and academic researchers in 2009), its mission (open, verifiable internet measurement), and its governance (Sections 2–3). ✅
- **What M-Lab runs and produces** — the measurement tests (NDT first among them, with the ndt7 protocol now dominant, plus Neubot DASH, WeHe, Reverse Traceroute, MSAK, and the retired fleet), the metrics (throughput, RTT, loss), and the open-data pipeline that lands every test result in Google BigQuery and Google Cloud Storage (Sections 4–7).
- **How M-Lab is used and where it breaks** — research and regulatory usage (including the verified 2009–2019 FCC relationship), consumer tools, the well-documented biases (self-selection, last-mile vs middle-mile attribution, coverage gaps), and a Singapore-specific view, since M-Lab operates servers in Singapore today (Sections 8–10).

Section 11 is the **Cymbal Bank worked example**: a Singapore-headquartered bank running scheduled NDT-style tests from branch and office links to M-Lab servers (and, in a later phase, to self-hosted measurement servers), with the metrics collected, the Grafana dashboards, the alerting thresholds, and an honest statement of what public-server measurement can and cannot tell a bank. Sections 12–15 consolidate the verification pass: the claims audit, what could not be verified, the glossary, and cross-references to the rest of this repository.

### The One-Paragraph Answer

M-Lab is an open source, not-for-profit measurement consortium that operates a globally distributed fleet of measurement servers — physical sites of three to four servers at interconnection points, virtual servers in all Google Cloud regions as of April 2023, and since 2024–2025 host-managed nodes run by third-party network providers — and invites anyone to run active measurement tests (chiefly NDT, the Network Diagnostic Tool) against them. Every test produces raw archives plus parsed, annotated rows in the public `measurement-lab` BigQuery project, queryable free of charge, together with raw packet captures in Google Cloud Storage. The design goal is scientific verifiability: all tools are open source, all data is published indefinitely, no PII is stored, and anyone can audit the methodology. The result is the most widely used open record of how the internet actually performs for users, and the same measurement pattern — schedule a test, measure throughput/latency/loss to a well-connected server, store the results, visualize and alert on them — is directly transferable to enterprise connectivity monitoring, which is what the Cymbal Bank worked example demonstrates.

### Integrity Convention

Following repository convention, this guide marks every verifiable claim: ✅ = verified this pass against a primary or cited source (URL in Section 12); ⚠ = flagged, approximate, or dated — the source was retrieved but the figure or status may have moved on, or the claim rests on well-documented knowledge not re-verified live; ❌ = a claim checked and found incorrect, with the correction stated. All verification was performed in August 2026 against live pages at measurementlab.net, the M-Lab GitHub organization, the live M-Lab Locate API, Google Cloud documentation, and archived copies where originals have moved; every URL cited in Section 12 is one that was actually retrieved this session (or is linked directly from a retrieved page). No dates, citations or statistics were invented; anything that could not be confirmed is flagged honestly in Section 13.

---

## 2. The Organization — Origins, Governance, Mission

### 2.1 Founding and Origins

M-Lab's own history page is the primary source for the founding story. ✅ In 2008, Vint Cerf — one of the "fathers of the Internet" — began a series of conversations with internet researchers about the obstacles they faced studying internet performance. Researchers named three problems: a lack of widely deployed servers with ample connectivity to support measurement experiments; an inability to share large datasets easily with one another; and the absence of any public resource providing combined performance data to policymakers or consumers over time. M-Lab was founded in direct response. ✅

The founding year is **2009**, and the founding partners are **New America's Open Technology Institute (OTI)**, the **PlanetLab Consortium**, **Google**, and a group of academic researchers — this is stated verbatim on the M-Lab about page and echoed on the "Who We Are" page, which lists the individual founders: Vint Cerf (Google), Sascha Meinrath (then OTI, now Penn State/X-Lab), Larry Peterson (Princeton University), Derek Slater (Google), Stephen Stuart (Google), and Meredith Whittaker (Google). ✅ M-Lab's 2009 founding vision document, still published on the site, describes the platform as a "joint venture" among OTI, PlanetLab, Google and academic researchers built to remove the barriers researchers had identified. ✅ (The PDF itself was retrieved this pass; its text layer extracts poorly due to font encoding, so the founding facts above are cited from the about and who pages, which are unambiguous.)

The first NDT data in the archive dates to **2009-02-18** (the `web100` protocol era start date published on the NDT page), which anchors the platform's operational start in early 2009. ✅

### 2.2 The Consortium and Non-Profit Model

M-Lab describes itself as "an open source project with contributors from civil society organizations, educational institutions, and private sector companies." ✅ It operates as a **not-for-profit** and is a **fiscally sponsored project of Superbloom** (a New York-based design and technology non-profit), with the privacy policy giving the operating address as 276 Fifth Ave, Suite 704-33, New York, NY 10001. ✅ Fiscal sponsorship means M-Lab does not run its own legal entity; Superbloom holds the sponsorship umbrella, and M-Lab's staff are employed across the sponsor and partner organizations — the maintainer list mixes Google engineers (Christophe Diot, Phillipa Gill, Cristina Leon, Arthur Terry), Superbloom staff (Nathan Kinkade, Melissa Newcomb, Pavlos Sermpezis as Director/Technical Lead), and independent contributors (Matt Mathis, Roberto D'Auria). ✅

Governance is layered:

- **Maintainers** — the working team listed above, responsible for the platform, tools and data. ✅
- **Experiment Review Committee** — academics and industry researchers (Vint Cerf, Nick Feamster of the University of Chicago, Aleksandar Kuzmanovic of Northwestern, Larry Peterson, Sascha Meinrath, Stephen Stuart, Georgia Bullen) who review proposed experiments for the platform. ✅
- **Advisory Committee** — chaired by Georgia Bullen (Superbloom Design), with Lai Yi Ohlsen (Cloudflare) and others. ✅
- **Supporting partners** — a long roster of research networks, regulators and companies that host or sponsor sites and services: Google, Equinix, the Internet Society, Mozilla Foundation, AARNet (Australia), CIRA (Canada), GRNET and EETT (Greece), HEAnet (Ireland), ISC, KENET (Kenya), REANNZ (New Zealand), Sorbonne Université, TOP-IX (Italy), WIDE Project (Japan), Vocus (Australia), and many more. ✅

The consortium model matters for the data: because hosting is donated by dozens of independent organizations, the platform is not owned by any single ISP or government, which underpins M-Lab's claim of neutral, verifiable measurement.

### 2.3 The Mission

M-Lab's mission is stated consistently across its pages: to **"measure the Internet, save the data, and make it universally accessible and useful"** — this is the site's tagline and the opening of the 2022 platform paper. ✅ The about page elaborates: M-Lab "aims to advance Internet research by empowering consumers with useful information about their Internet performance," with data aimed at "researchers, regulators, advocacy groups, and the general public." ✅ Three commitments follow from the mission, all stated on the about page and enforced operationally:

1. **Open data** — all data collected by the platform is openly available; since 2019 (the "M-Lab 2.0" platform) hosting an experiment requires an open source server container that M-Lab can build and publicly archive, enforced through technical means. ✅
2. **Open tools** — all measurement tools are open source, "so anyone with the time and skill can review and improve the underlying methodologies and assumptions." ✅
3. **Verifiable science** — "Real science requires verifiable processes," and M-Lab welcomes scientific collaboration and scrutiny; transparency and review are described as "key to good science, and good science is key to good measurement." ✅

### 2.4 How Tests Get Onto the Platform

Each researcher-developed test is allocated **dedicated resources on the M-Lab platform** to facilitate accurate measurements; server-side tools are openly licensed and operated, and third parties may develop their own client-side measurement software against them. ✅ The platform publishes formal procedures for approving new tests (the "Requirements and Procedure for Approval of New M-Lab tests" document) and roles/responsibilities for researchers. ✅ The open-source requirement is not rhetorical: the SamKnows page documents that in late 2019 M-Lab 2.0 began enforcing the requirement that experiments ship an open source Docker container, and because SamKnows declined to provide one, M-Lab could no longer host its experiment after the close of the 2019 FCC program (Section 8.2). ✅

### 2.5 The Founding Vision Document (2009)

M-Lab still publishes its original governance document — **"M-Lab's Founding Vision"**, described on the publications page as "The founding vision and operational procedures of the M-Lab measurement consortium, as laid out in 2009." ✅ The PDF was retrieved this pass; its text layer extracts badly (font-encoding garbling), so this guide does not quote it and relies on the about/who pages for founding facts. What is safe to say from the retrieval: the document exists, is dated to the founding era, and covers the consortium's objectives, the roles of OTI, PlanetLab and Google (the same partners named on the about page), server-site structure, and the operating principles that still hold today — open data, open source, and a community-based effort. ✅ The existence of a formal 2009 founding document is itself evidence that the consortium model — not a single-company product — was the design from day one.

---

## 3. The Platform — Global Measurement Infrastructure

### 3.1 Architecture at a Glance

The M-Lab platform is a client/server measurement system with four functional layers, all of which this guide verified against M-Lab's own documentation:

1. **Clients** — browser pages (speed.measurementlab.net), mobile apps, Chrome extensions, and third-party integrations that initiate tests. A client first asks the **Locate service** which server to use, then runs the test against that server. ✅
2. **The Locate service** — a GCP-hosted API at `locate.measurementlab.net` that "locates" the best M-Lab server for a request, returning complete, client-usable service URLs signed with an `access_token`. ✅
3. **Measurement servers** — the physical, virtual and host-managed nodes running the server side of each experiment (for NDT, `ndt-server`). Each physical site is **3–4 servers and one switch, connected directly to an upstream provider**; virtual sites consist of one or more virtual servers proportional to demand. ✅
4. **The data pipeline** — test results are packaged, uploaded to **Google Cloud Storage** archives (`archive-measurement-lab`), and parsed/annotated into **BigQuery** views and tables in the `measurement-lab` project, with packet captures retained for NDT. ✅

M-Lab's own description of the flow: someone runs a test; the test communicates a stream of data between the user's machine and a local M-Lab measurement server; the server-side tool analyzes what happened to that stream; the user receives results; and M-Lab packages and stores the results for public access. ✅

### 3.2 Physical Sites

Since 2008 M-Lab has sought hosting in "well connected data centers where ISPs interconnect with one another," so that a small number of locations can serve as many users as possible; in large metro areas M-Lab prefers **multiple sites** to obtain diversity in transit and routes. ✅ The develop page adds that M-Lab sites are typically physically collocated in the same buildings as major network providers' routers, in networks that host internet content or key infrastructure — internet exchanges, transit networks, and prominent hosting companies. ✅ The `mlab-oti` machine-naming convention (e.g. `mlab2-sin01.mlab-oti.measurement-lab.org`) identifies the original OTI-managed physical fleet; site codes follow the airport-style convention (sin01, lga05, bom01, bom02, hnd, gru...). ✅

### 3.3 The Cloud Expansion ("M-Lab to the Cloud")

From launch in 2009 until 2022, M-Lab's server software ran exclusively on physical bare-metal machines. ✅ In **July 2022** M-Lab announced "M-Lab to the Cloud": a pilot deployment of NDT into Google Cloud, directing less than 10% of NDT clients in a few regions to cloud nodes as a canary test against bare-metal results. ✅ By **April 2023**, M-Lab announced a **virtual site running ndt-server in all 35 Google Cloud regions then in existence** — 41 virtual servers in total (three regions — hnd, gru, bom — had three VMs each; the rest one) — with cloud traffic accounting for **13% of platform traffic volume**. ✅ The same post documents the quality-assurance methodology: an experiment in Mumbai (BOM, then the third-busiest metro at ~273K tests/day in September 2022) shifted load between physical sites and a cloud VM and compared throughput, loss and RTT distributions, concluding the cloud VM produced measurements consistent with the physical sites, with slightly lower loss and latency. ✅ It also surfaced a real operational finding: during an extreme-load spike on 2023-01-25, the cloud server in Mumbai (4 GB RAM, versus 16 GB on physical servers) became resource-constrained and the Locate service redirected traffic away from it — the platform behaving as designed. ✅

⚠ The April 2023 numbers are dated: Google Cloud has since expanded well beyond 35 regions, and M-Lab stated it was actively working to expand to more clouds and CDNs. No newer published total was found this pass, so treat "35 regions / 41 virtual servers / 13% cloud traffic" as an April 2023 snapshot, not the current state.

### 3.4 Host-Managed Nodes and Autojoin

In **2024** M-Lab introduced a new deployment model called **"Host-Managed"**: any network provider may stand up a server running M-Lab's software and join the platform; M-Lab does not manage or control those servers. ✅ The status page describes a 2025 pilot of the model. ✅ Host-managed nodes are operationally distinct — their data is kept separate in BigQuery (Section 6.2) because M-Lab "cannot make assertions about the quality of the data" from machines it does not manage. ✅

The host-managed requirements page (retrieved this pass) is detailed and practical ✅:

- Hardware: at least four 2 GHz CPUs with 4 GB RAM, 50 GB+ disk (typical nodes generate ~4 GB/month of measurement data; busy nodes 20 GB/month+).
- Software: a reasonably current Linux with the `tcp_bbr` kernel module and Docker; services run as Docker containers deployed via Docker Compose from the `m-lab/autonode` repository.
- Network: one static IPv4 and one static IPv6 address; a 10 Gb/s uplink (or at least 2× consumer-grade speeds); open ports **80 and 443** (NDT tests) and **9990–9999** (monitoring).
- Operations: hosts must provide their own health monitoring and alerts, respond to M-Lab, and monitor traffic volume, adjusting a `PROBABILITY` setting that controls what share of Locate-routed tests the node receives.
- Eligibility: organizations only (ISPs, hosting providers, universities, non-profits, network operators), with a verifiable public presence and a NOC/support address on their own domain; contributions over ~10 servers may warrant a Memorandum of Understanding.
- Registration flow: organizations register via the Infrastructure Contribution Form, receive an API key, then the node self-registers via the **Autojoin API**, reports health to Locate, serves NDT tests, archives measurements, and publishes them to BigQuery automatically.

The node's Prometheus metrics are exposed on port 9991+ (e.g. `ndt7_client_test_results_total`), with node_exporter on 9995 — the host-managed docs even show a PromQL query for tests-per-minute and point hosts at Grafana for visualization. ✅ This is the same observability pattern the Cymbal Bank worked example generalizes in Section 11.

### 3.5 Server Counts and Coverage — Verified Numbers, Dated Numbers

M-Lab publishes no single authoritative text total on its current status page; the infrastructure map is an interactive Mapbox visualization, which this pass could not reduce to a text count. ✅ What *was* verified in text form, all with dates attached:

- **"Over 125 sites around the world"** — develop page (undated; ⚠ approximate/dated, retrieved August 2026). ✅
- **"Platform of over 130 servers... covers the US, Europe, Australia, and parts of Africa and Asia"** and, elsewhere in the same two-pager, **"100+ servers around the globe"** — the M-Lab two-pager PDF (⚠ dated material; the two-pager is internally inconsistent, also claiming "over 600 terabytes" in one place and "over 750TB" in another). ✅
- **41 virtual servers across 35 GCP regions** — April 2023 blog (⚠ dated, Section 3.3). ✅
- Live machine evidence from the **Locate API v2** (retrieved this pass, August 2026): a query for Singapore returned three healthy NDT machines (Section 10.1), and the API's design returns up to four candidates per request. ✅

The status page states physical sites are 3–4 servers plus a switch, so a site count is not a server count; the older "130 servers" figure is consistent with roughly 40–50 physical sites at that time. ⚠ No current (2026) total site or server count in text was found this pass; the honest statement is that the platform numbers "in the hundreds of servers, in 100+ locations" and is growing, but the exact current figure should be taken from the interactive map or the Locate API rather than from any text page. ⚠

### 3.6 Platform Operations and SLO

- **SLO**: M-Lab states it "has run on a best-effort business-hours-only SLO" since the project's founding; in 2019 the NDT measurement service was globally available between 99% and 99.9% of the time ("slightly more than 8 hours of downtime"). ✅ Developers integrating M-Lab into products are told their service will inherit that SLO and must fail gracefully. ✅
- **Rate limits**: M-Lab imposes a rate limit of **40 tests per client per day**; clients exceeding it receive a 204 No Content response. ✅ Automated and repeated testing is welcomed within that limit, with informed consent. ✅
- **Monitoring**: the platform transitioned to Prometheus monitoring (a 2017 PromCon talk documents it), and per-service metrics are published for hosts. ✅
- **Capacity**: the platform "gets many millions of tests per day" (host-managed docs). ⚠ The two-pager's "200K+ tests run daily" and "over 200 million tests run since launch" figures are dated. ✅
- **Data volume**: the two-pager claims 750 TB+ of open data (⚠ dated; the current figure is not published in text this pass).

---

## 4. The Tests — The Current Roster and the Retired Fleet

### 4.1 How Tests Are Hosted — The Rules of the Platform

The tests page states the platform's ground rules, all verified ✅:

- **Active measurement only** — tests run only when people decide to run them; there is no passive monitoring of connections, and tests do not read emails, web searches, or other traffic. Tests measure how the network responds to a synthetic stream of data generated for measurement purposes.
- **No PII collected by tests** — but the client's public IP address is collected and published with results.
- **Data is public and permanent** — all data is made publicly available, stored and published indefinitely, and M-Lab cannot delete individual tests once collected.
- **All tests are open source**, created and maintained by researchers.

### 4.2 Current Tests (Verified Against measurementlab.net/tests, August 2026)

| Test | What it does (M-Lab's own description) | Developer / origin |
| --- | --- | --- |
| **NDT (Network Diagnostic Tool)** | Tests connection speed and provides "a sophisticated diagnosis of problems limiting speed" | Originally Internet2; maintained with M-Lab since 2009 ✅ |
| **Neubot DASH** | Emulates a video streaming player to measure network quality; the DASH (Dynamic Adaptive Streaming over HTTP) test streams a 30-second video in fifteen 2-second segments | Neubot originally by the Nexa Center for Internet & Society (Politecnico di Torino); original Neubot client retired January 2019, DASH still hosted ✅ |
| **Reverse Traceroute** | Measures the network path back to a user from selected network endpoints | University of Washington ✅ |
| **WeHe** | Replays internet traffic recorded from real apps (e.g. YouTube, Spotify) to detect whether an ISP is giving different performance to an app's traffic (differential treatment) | Northeastern University ✅ |
| **MSAK (Measurement Swiss-Army Knife)** | Configurable WebSocket-based throughput measurement protocol plus a UDP-based latency measurement protocol | M-Lab/newer addition ✅ |

### 4.3 Current Core Services

Core services run on every M-Lab server alongside tests and produce the platform-wide datasets ✅:

- **Packet Headers** — collects packet headers for all incoming TCP flows and saves each stream into a per-stream `.pcap` file.
- **TCP INFO** — collects TCP statistics for platform connections using `tcp-info`.
- **Traceroute** — collects network path information for every connection to the platform.

### 4.4 Retired Tests (Verified Against measurementlab.net/tests, August 2026)

M-Lab lists these as "once hosted but since retired"; the data collected while they ran remains available ✅:

- **Glasnost** — tested for application-specific blocking or throttling; **decommissioned 07/07/2017**; source code still available. ✅ (Glasnost's historical association with the Max Planck Institute for Software Systems is well documented in the literature ⚠ but is not named on the current M-Lab page — treat the attribution as unverified this pass.)
- **NPAD (Network Path & Application Diagnostics)** — diagnosed common problems impacting last-mile broadband networks; retired. ✅
- **Pathload2** — available-bandwidth probing; decommissioned **12/21/2012**. ✅
- **Shaperprobe** — attempted to measure traffic shaping; decommissioned **05/11/2015**. ✅
- **Windrider** — attempted to detect mobile provider application/service differentiation; decommissioned **01/17/2013**. ✅
- **BISmark** — home-router-based performance testing over time. ✅
- **MobiPerf** — mobile network performance measurement. ✅
- **OONI Probe** — measured specific use cases of network interference. ✅ (OONI continues as an independent project today ⚠.)
- **SamKnows** — M-Lab hosted the off-net servers for the FCC's Measuring Broadband America program from **2009–2019**; the relationship ended when SamKnows declined M-Lab 2.0's open source requirement (Section 8.2). ✅

### 4.5 Retired Core Services

- **SideStream** — collected web100 statistics about TCP connections on the "M-Lab 1.0" platform. ✅
- **Paris Traceroute** — collected network path information for every connection on the M-Lab 1.0 platform. ✅

### 4.6 Names Checked and Not Found (❌ and ⚠)

The task brief asked this guide to check several names against the live roster. Results, all checked against measurementlab.net/tests in August 2026:

- **DASU** — ❌ no test or service by this name appears anywhere on the current tests page or in M-Lab's documentation retrieved this pass. It is not part of the current platform.
- **PathNeutral** — ❌ not present on the current tests page. No current M-Lab test by this name was found this pass.
- **Replay** — ⚠ not listed as a standalone test on the tests page; however, the Locate API v2 documentation lists **`wehe/replay`** as a measurement service — "Replay" is the protocol behind the WeHe test (a replay of recorded app traffic). The historical Google "Replay" experiment name does not appear on the current roster.
- **Paris Traceroute, NPAD, SideStream** — present but explicitly retired (Section 4.4–4.5). ✅
- **Internet Health Test** — not a hosted M-Lab test; the consumer tool situation is covered in Section 8.3.

---

## 5. NDT in Depth — Protocols, Metrics, and Methodology

### 5.1 Purpose and History

NDT is "a single stream performance measurement of a connection's capacity for 'bulk transport' (as defined in IETF's RFC 3148)" that reports upload and download speeds and latency metrics. ✅ It was originally developed at **Internet2**; M-Lab has hosted NDT since its founding in 2009 and has helped maintain and develop it for most of its history on the platform. ✅ Three themes have driven NDT's evolution, per M-Lab: standard kernel instrumentation, advances in TCP congestion control, and protocols/ports to support more clients. ✅ The consumer front-end is **speed.measurementlab.net** ("Speed Test by Measurement Lab"), which presents Download, Upload, Latency and Retransmission results and requires consent to the data policy, including IP publication. ✅

### 5.2 The Three Protocol Eras ("Datatypes")

M-Lab names specific protocol versions as NDT "datatypes" ✅:

| Datatype | Details (all verified on the NDT page) | Period |
| --- | --- | --- |
| **web100** | Original NDT server; relied on the web100 kernel module for TCP statistics; Reno congestion control | 2009-02-18 → 2019-11-20 |
| **ndt5** | Backward-compatible with legacy NDT clients; relies on tcp-info for TCP statistics; Cubic congestion control; served by ndt-server | 2019-07-18 → present (ndt5+raw support ended January 2024; ndt5+ws/wss remain) |
| **ndt7** | TCP BBR where available (falling back to Cubic), standard HTTP(S) ports **80/443**; tcp-info statistics; opt-in data-transfer limit; served by ndt-server | 2020-02-18 → present |

The transition facts are verified: the "vast majority of data is ndt7" today (M-Lab's own blog), ndt5+raw protocol support ended **as of January 2024** due to the added support for admission control and virtual servers, while ndt5+ws and ndt5+wss remain available, and new integrations are strongly encouraged to use ndt7. ✅

### 5.3 The ndt7 Protocol Specification (v0.11.0, retrieved this pass)

The ndt7 spec (in `m-lab/ndt-server`, `spec/ndt7-protocol.md`) defines a **non-backwards-compatible redesign** of the original NDT protocol, "based on WebSocket and TLS, and [taking] advantage of TCP BBR, where this flavour of TCP is available." ✅ Key verified details:

- Two independent test types, each over **exactly one TCP connection**: the download test at `/ndt/v7/download` and the upload test at `/ndt/v7/upload`, both upgraded from HTTPS via the WebSocket subprotocol **`net.measurementlab.ndt.v7`**. ✅
- The main metric is **goodput** — application-level speed excluding WebSockets, TLS, TCP/IP and link-layer overhead — supplemented with kernel-level `TCP_INFO` data. ✅
- Tests are short by design: maximum runtime **ten seconds** after the WebSocket handshake, hard-closed at thirteen seconds; the server ends early once performance stabilizes. ✅
- Textual messages carry **JSON-serialized measurements**; binary messages carry random data to generate network load. ✅
- The spec is explicit about what NDT measures and does not measure: "This is not necessarily a measurement of your last mile speed. Rather it is a measurement of what performance is possible with your device, your current internet connection..., the characteristics of your ISP and possibly of other ISPs in the middle, and the server being used." ✅ This is the canonical statement of the last-mile vs middle-mile attribution caveat (Section 9.2).
- `TCP_INFO` fields provide diagnostics: `MinRTT`, `RTT`, `RTTVar` for round-trip statistics; queue buildup signals a bottleneck with a large buffer filling during the test (the bufferbloat signature); `BusyTime`/`RWndLimited`/`SndBufLimited` identify where the bottleneck sits (receiver window, sender buffer, or the network); and the ratio of `BytesRetrans` to `BytesSent` approximates the packet loss rate — a deliberately crude approximation, per the spec, since it assumes uniform drop probability. ✅

### 5.4 What NDT Reports — The Metrics

Across the protocol spec, the BigQuery schema and the speed-test front-end, the metrics NDT produces are verified as ✅:

- **Download and upload throughput** — reported as `MeanThroughputMbps` (average of tcp-info snapshots at the beginning and end of the measurement) and derivable as goodput from `AppInfo.NumBytes` / `ElapsedTime`.
- **Round-trip time** — minimum RTT (`MinRTT`, milliseconds; derived from `TCPInfo.MinRTT` after 2020-06-18 for ndt5 rows) plus RTT variance (`RTTVar`).
- **Packet loss** — `LossRate` ("loss rate from the lifetime of the connection") and retransmission counts/ratios (`BytesRetrans`, `Retransmits`).
- **Congestion control** — `CongestionControl` records which algorithm ran (BBR vs Cubic), important because the algorithm affects measured throughput.
- **Jitter-ish / latency-variation signals** — RTTVar and the queue-buildup diagnostics; M-Lab's published research has separately analyzed "latency variation" from NDT data at scale.
- **Bufferbloat-ish signals** — the spec's discussion of queue buildup and `RWndLimited`/`SndBufLimited` chrono stats is exactly the diagnosis a bufferbloat check needs; M-Lab's blog even carries a bufferbloat category. ✅

### 5.5 The ndt7 Data Format (v0.3.0, retrieved this pass)

Each ndt7 subtest is serialized on disk as a **gzip-compressed JSON file** named `ndt7-<subtest>-<year><month><day>T<hour><minute><second>.<nanoseconds>Z.<uuid>.json.gz`. ✅ The result JSON is a complete record of the test: `GitShortCommit`, `Version`, `ClientIP`, `ClientPort`, `ServerIP`, `ServerPort`, `StartTime`, `EndTime`, and an `Upload` or `Download` object containing `StartTime`, `EndTime`, `UUID`, `ClientMeasurements` (per-interval measurements from the client, e.g. `AppInfo {ElapsedTime, NumBytes}`), `ClientMetadata` (query-string metadata such as client library name/version), `ServerMetadata` (server configuration labels such as `deployment` and `machine-type`), and `ServerMeasurements`. ✅

### 5.6 The Client/Server Model

The flow for an ndt7 test, verified across the locate docs and the ndt7 spec ✅:

1. The client calls the **Locate API v2** (e.g. `https://locate.measurementlab.net/v2/nearest/ndt/ndt7`), which returns up to four candidate machines, each with `machine`, `location` (city/country), and complete `urls` for `ws`/`wss` download and upload endpoints, including an `access_token=` parameter validated by the target server. ✅
2. The client opens a WebSocket (TLS) connection to one candidate and runs the download and upload subtests, exchanging JSON control messages and binary load. ✅
3. The server (and client) record `TCP_INFO` snapshots and application-level counters; results are written as gzip JSON, archived to GCS, parsed and annotated into BigQuery. ✅
4. Reference clients exist in Go (`ndt7-client-go`), JavaScript (`ndt7-js`), plus community-maintained iOS (Swift) and Android (Kotlin/Java) clients. ✅
5. Server-side admission control: target servers may reject a connection if local conditions would degrade measurement quality; clients must fall back through the candidate list and handle total rejection gracefully. ✅

---

## 6. The Data — The Open-Data Model

### 6.1 The Data Policy — Open, Public, Permanent, No PII

M-Lab's data policy, verified across the tests page, the privacy policy and the developer pages ✅:

- **Open and public**: all data collected via M-Lab is made publicly available; the privacy policy states the data is released "into the public domain under an OSI-approved license," with M-Lab waiving "all copyright and related rights in and to the experiment data." ✅
- **Permanent**: "All experiment data is retained indefinitely and published publicly"; M-Lab states it cannot delete data associated with individual tests once collected, and that "storing data forever" is a deliberate consequence of scientific independence. ✅
- **No PII**: "M-Lab does not allow the storage of personally identifiable information (PII) on the platform"; the platform's experiments may collect the client's public-facing IP address "due to its necessity in understanding experiment results," and tests "do not collect information about your other Internet traffic... or any personally identifiable information." ✅
- **Legal basis**: for EU users, M-Lab processes test data under the GDPR "legitimate interest" basis, arguing the dataset's uniqueness and public-good purpose; the current privacy policy version is **[v5], May 3, 2026** (earlier versions v1–v4 archived on the page). ✅
- **Informed consent**: clients must obtain informed consent before tests, and third-party clients that collect additional data may not submit it to M-Lab. ✅

### 6.2 The BigQuery Datasets

M-Lab's parsed and annotated data lives in the public **`measurement-lab`** Google Cloud project on BigQuery, and M-Lab makes **query access free of charge** to users who subscribe to the M-Lab Discuss Google Group (the group acts as the whitelist so that M-Lab pays the query costs). ✅ Verified dataset/view structure (from the NDT page's BigQuery documentation):

- **`measurement-lab.ndt.ndt7_union`** — all ndt7 data from both M-Lab-managed and host-managed servers, with annotations; no filtering; **currently the recommended view for general usage**. ✅
- **`measurement-lab.ndt.ndt7`** — all ndt7 data from M-Lab-managed servers only. ✅
- **`measurement-lab.ndt.ndt7_dynamic`** — all ndt7 data from host-managed servers only. ✅
- **`measurement-lab.ndt.ndt5`** — all ndt5 data (not recommended; most data is ndt7 and ndt5 support may be removed). ✅
- **`measurement-lab.ndt.unified_*`** (e.g. `unified_downloads`, `unified_uploads`) — a standardized schema across all NDT datatypes (web100, ndt5, ndt7), separate views for upload and download, curated to "known good" tests (Section 6.5); largely a transitional tool from the web100/ndt5→ndt7 migration and "currently not recommended" for general use because they are more expensive to query. ✅
- **`measurement-lab.ndt_intermediate.*`** — "extended views" joining raw measurements with annotations and remapping column names across datatypes; schemas not guaranteed stable. ✅
- **`measurement-lab.ndt_raw.*`** — 1-to-1 mappings of GCS archive files to test rows ("faithful views"); provided for completeness, not recommended for general use. ✅
- **Legacy datasets**: `measurement-lab.legacy.ndt` (~2015-01-01 to 2017-05-10), `measurement-lab.legacy.ndt_pre2015` (~2009-02-18 to 2014-12-31), plus `base_tables`, `rc`, `release_v3_1` and `release` (e.g. `release.ndt_all`). ✅

**Years of history**: NDT data in the archive and BigQuery spans **2009-02-18 to the present** — the pre-2015 legacy table starts at the platform's first NDT data, and the changelog documents table versions v1 (2015) through v3 (2017–2018) with all historical data re-parsed and annotated. ✅

### 6.3 The ndt7 BigQuery Schema (Key Fields)

The ndt7 schema (documented on the ndt7 page, retrieved this pass) ✅:

- **`id`** — UUID of the connection. 
- **`a`** (annotations): `UUID`, `TestTime` (UTC), `CongestionControl`, **`MeanThroughputMbps`**, **`MinRTT`** (ms), **`LossRate`** (lifetime of connection).
- **`parser`**: version, time, `ArchiveURL`, `Filename`, git commit, archive/file sizes.
- **`date`** — partition column for query performance.
- **`raw`**: `GitShortCommit`, `Version`, `ServerIP`, `ServerPort`, `ClientIP`, `ClientPort`, `StartTime`, `EndTime`, and `Upload`/`Download` records each containing `UUID`, `StartTime`, `EndTime`, `ServerMeasurements` (with `AppInfo` {NumBytes, ElapsedTime}, `ConnectionInfo`, **`BBRInfo`** {BW, MinRTT, PacingGain, CwndGain}, **`TCPInfo`** — dozens of kernel fields: Retransmits, RTO, ATO, SndMSS, RcvMSS, Unacked, Sacked, Delivered, DeliveredCE, BytesSent, BytesRetrans, DSackDups, ReordSeen, RWndLimited, SndBufLimited, ...), `ClientMeasurements` (client-side TCPInfo), `ClientMetadata`, `ServerMetadata`.

The ndt5 schema follows the same pattern with a `Control` channel record and `S2C` (download) / `C2S` (upload) measurement records. ✅

### 6.4 Raw Data in Google Cloud Storage

- All raw data and log files from the measurement fleet are archived **in their original format** in the public Google Cloud Storage bucket **`archive-measurement-lab`** (NDT raw data under `.../ndt`). ✅
- **TCP packet captures (.pcap)** exist for most NDT tests but are not indexed in BigQuery; they are available in the raw archives for deep analysis. ✅
- M-Lab periodically **reprocesses all archived data** as parsing and analysis improve; BigQuery rows reference the raw archive locations so dedicated users can reconstruct and replicate M-Lab's parsers. ✅
- Historical formats: the web100 era produced kernel snaplog-style records (the BigQuery changelog references "snaplogs" and the `web100` datatype), while today's ndt7 writes gzip-compressed JSON (Section 5.5); the `.tap` file name associated with web100-era NDT results is part of that legacy pipeline ⚠ (the current documentation describes the snaplog format rather than naming `.tap` files directly — treat the specific `.tap` extension as unverified this pass).

### 6.5 Data Curation — The Unified View Criteria

The unified views curate tests against M-Lab's published quality criteria ✅:

- At least **8 KB** of data transferred;
- Test duration between **9 and 60 seconds**;
- For downloads, some form of network congestion detected (tests with only non-network bottlenecks excluded);
- No parser errors and no NULL results;
- Tests from M-Lab operations/monitoring (OAM) infrastructure excluded.

M-Lab's caveat: these criteria encode "our team's current, best understanding of completeness & research quality" — a moving target, and exactly the kind of methodology detail researchers must audit rather than trust blindly.

### 6.6 Research Usage — Verified Publications

The publications page lists dozens of peer-reviewed papers built on M-Lab data; those this pass verified (title, authors, venue, URL) include:

- **"M-Lab: User Initiated Internet Data for the Research Community"** — Phillipa Gill, Christophe Diot, Lai Yi Ohlsen, Matt Mathis, Stephen Soltesz; ACM SIGCOMM Computer Communication Review, January 2022; the definitive platform-update paper ("10+ years after its initial introduction to the research community"). ✅
- **"The Ukrainian Internet under attack: an NDT perspective"** — Jain, Patra, Xu, Sherry, Gill; IMC 2022; used NDT data over 54 days of war, finding average packet loss rates increasing by as much as 500% in some regions. ✅
- **"The importance of contextualization of crowdsourced active speed test measurements"** — Udit Paul et al.; ACM IMC 2022; augments ~1.5M Ookla and M-Lab measurements with a Broadband Subscription Tier methodology and finds "the median throughput reported by Ookla speed tests can be up to two times greater than M-Lab measurements for the same subscription tier, city, and ISP" — a peer-reviewed statement of the measurement-methodology gap between the two major crowdsourced datasets. ✅
- **"Estimating Residential Broadband Capacity using Big Data from M-Lab"** — Deng, Feng, Gharakheili, Sivaraman (UNSW); 2019; 63 million speed tests over 12 months, isolating households behind NAT. ✅
- **"Measuring broadband performance using M-Lab: Why averages tell a poor tale"** — Deng et al.; ATNAC 2015; 96,882 Australian households, four months. ✅
- **"Measuring Latency Variation in the Internet"** — Høiland-Jørgensen et al.; CoNEXT 2016; two-thirds of samples exceeding 100 ms of variation. ✅
- **"M-Lab at IMC"** (2012) and the "M-Lab—Network Science at Scale" (MIT, 2012) presentations. ✅

The task brief mentioned a paper titled "Towards an Open, Verifiable Internet"; ❌ no such title appears on the M-Lab publications page this pass, and no retrieval confirmed it — the verified anchor paper for the platform is the 2022 CCR paper above (see Section 13).

### 6.7 An Illustrative Query (BigQuery)

The following is an **illustrative** query written for this guide against the verified view names and field names documented in Section 6.2–6.3 (it has not been executed this pass ⚠ — it demonstrates the shape of an M-Lab query, using `ndt7_union` and the `a.*` annotation fields):

```sql
-- Median download throughput (Mbps) and p95 loss rate for Singapore, last 90 days.
SELECT
  DATE(a.TestTime) AS day,
  APPROX_QUANTILES(a.MeanThroughputMbps, 100)[OFFSET(50)] AS median_dl_mbps,
  APPROX_QUANTILES(a.LossRate, 100)[OFFSET(95)] AS p95_loss
FROM `measurement-lab.ndt.ndt7_union`
WHERE a.TestTime >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 90 DAY)
  AND a.ClientIP IS NOT NULL
GROUP BY day
ORDER BY day;
```

Real queries in M-Lab's documentation follow this pattern; the recommended view, the field names, and the free-query mechanics are all verified (Sections 6.2–6.3, 7.2), while the SQL above is an original example for this guide and should be validated against the live schema before use. ⚠

---

## 7. Access — APIs, BigQuery, and Tooling

### 7.1 The Locate API v2

The Locate API ("M-Lab's Locate Service and API provides consistent, expected measurement quality for M-Lab clients") is the platform's server-selection and access layer, hosted on GCP ✅:

- **Base**: `https://locate.measurementlab.net/`; v2 endpoint pattern `/v2/nearest/<service>/<type>`, e.g. `https://locate.measurementlab.net/v2/nearest/ndt/ndt7`. ✅
- **Supported services**: `ndt/ndt5`, `ndt/ndt7`, `wehe/replay`, `neubot/dash`. ✅
- **Response**: JSON `results[]` with `machine`, `location` (city/country), and complete `urls` (ws/wss download & upload) including an `access_token` validated by the target server; up to four results. ✅
- **Selection parameters**: geographic proximity via GCP request headers (AppEngine lat/long, with Maxmind DB fallback and a 503 if location is unknown); explicit `country` / `region` (ISO 3166-1 alpha-2 / ISO 3166-2) with a `strict=true` option to force servers in the requested country; `org=` and `site=` filters for multi-organization deployments (e.g. `?org=mlab`, `?site=lga06`). ✅
- v1 is **DEPRECATED**; new integrations must use v2. ✅

### 7.2 BigQuery Access

Verified from the BigQuery QuickStart page ✅:

- Free query access requires subscribing the Google account to the **M-Lab Discuss group** (`groups.google.com/a/measurementlab.net/g/discuss`); members are whitelisted against the `measurement-lab` project so "query charges are paid for by M-Lab."
- Access via the Cloud Console (`console.cloud.google.com/bigquery?project=measurement-lab`) or the `gcloud`/`bq` SDK with `gcloud auth login` and the project set to `measurement-lab`.
- Service accounts can be added by emailing support@measurementlab.net; M-Lab notes it cannot guarantee zero billing for service-account applications (it verified one reference setup incurred no charges).
- Queries from projects you create, saving results to tables, etc. **do incur costs to you** — only in-project queries are free.
- Schemas and changelog: the "M-Lab Dataset Schemas and Changelog" page under data/docs.

### 7.3 Raw Archives and Other Data Access

- **GCS raw data**: `archive-measurement-lab` bucket (Section 6.4); the GCS documentation page linked from the NDT page returned 404 this pass ⚠ (the page has moved or been renamed — use the data docs index instead).
- **DISCO dataset**: M-Lab publishes the DISCO switch dataset (data/docs page). ✅
- **Analyses**: published analyses include the 2018 discard analysis and "Exploring the African Internet" (data/docs page). ✅

### 7.4 M-Lab's Own Tools and Integrations

From the data tools page (retrieved this pass) ✅:

- **M-Lab Measure** — Chrome extension that schedules NDT at regular intervals to track a connection over time.
- **Murakami** (`github.com/m-lab/murakami`) — "a tool for creating an automatic M-Lab testing platform with as little effort as possible"; runs an NDT test roughly every twelve hours and stores results for analysis. This is the closest public analog to the Cymbal Bank worked example in Section 11.
- **speed.measurementlab.net** (`github.com/m-lab/mlab-speedtest`) — the standalone multilingual NDT test site.
- **Community tools**: I3 Connectivity Explorer (combines M-Lab with FCC/Census/EPA/USDA data), Piecewise (aggregation/visualization), SpeedUp America (open maps of M-Lab speeds by census block/ISP).
- **Integrations**: Domotz, Fing, OONI, CIRA's Internet Performance Test, Google OneBox search speed-test widget, Broadband Now, Minnesota Office of Broadband Development, and others.
- M-Lab also publishes **visualizations and broadband maps** (measurementlab.net/visualization, linked from the two-pager).

### 7.5 Cross-Reference — Data Acquisition Patterns

M-Lab data is **fetched, not scraped**: it is obtained through the BigQuery API, the GCS raw archives, and the Locate/measurement APIs — the "API-first" end of the acquisition spectrum documented in this repository's [Large-Scale Web Data Acquisition guide](large_scale_web_data_acquisition_guide.md). The relevant patterns there are the fetch-and-store pipeline design, politeness/rate-limit compliance (M-Lab's 40 tests/client/day limit is the politeness contract), deduplication of test IDs, and governed storage of raw + cleaned forms — M-Lab's own raw/GCS + curated/BigQuery split mirrors the WARC/raw + Parquet/clean pattern recommended in that guide. ✅ (The sibling guide was read this pass for style and cross-reference; it does not cover M-Lab specifically.)

---

## 8. The Usage — Research, Policy, and Consumers

### 8.1 Researchers

M-Lab's research value proposition, verified ✅: an open, verifiable, longitudinal dataset with full provenance — "all of the software that runs our platform, pipeline and measurement services... is open for review on GitHub," giving "an exhaustive view into the provenance of all collected and published data." ✅ The platform is explicitly positioned for measuring "the interconnections between independent networks (the 'inter' part of the Internet)" rather than the last mile (Section 9.2). ✅ Citation guidance is published: "The M-Lab NDT Data Set, <date range used> https://measurementlab.net/tests/ndt" with a BibTeX template. ✅ Section 6.6 lists verified example papers.

### 8.2 Regulators and Policymakers — The FCC Relationship (Verified, Including Its End)

This is the highest-profile regulatory use of M-Lab, and this pass verified both the relationship and its termination from M-Lab's own pages ✅:

- **2009–2019**: "From 2009-2019, M-Lab donated server resources to the FCC Measuring Broadband America program, providing its contractor, SamKnows, with off-net hosting locations for the annual study." ✅ (The FCC's Measuring Broadband America reports — the FCC's own consumer broadband performance studies — therefore rested in part on M-Lab-hosted off-net measurement points.)
- **The end of the relationship**: "In late 2019, we launched M-Lab 2.0, which enforces, through technical means, our long-standing open source requirement... SamKnows has declined to provide an open source server, so we have not been able to host their experiment since the close of the 2019 MBA program." ✅ This is a verified, dated account of the FCC program's M-Lab hosting ending: **the 2019 MBA program was the last one hosted on M-Lab**.
- The M-Lab two-pager (dated) states "The Federal Communications Commission (FCC) and Greece's telecommunications authority have already partnered with M-Lab," and lists regulator partners including the **European Commission, EETT (Greece), RTR (Austria), OCECPR (Cyprus)**, and "4+ national regulators and the European Commission using M-Lab tools, data, and infrastructure." ✅ ⚠ Dated document.
- The policy-advocacy blog quotes the FCC MBA Program Technical Index on on-net/off-net methodology. ✅
- ⚠ **Could not be verified this pass**: fcc.gov pages themselves (including the Measuring Broadband America page referenced by M-Lab) blocked automated retrieval in August 2026, so this guide cannot confirm from FCC's own site what the FCC has done with M-Lab data since 2019 (e.g. in later broadband reports or the FCC speed-test app). Any claim about post-2019 FCC use of M-Lab data is therefore flagged unverified (Section 13).

### 8.3 Consumer-Facing Tools — Speed Test and the Internet Health Test

- **The current consumer front-end is speed.measurementlab.net** — "Speed Test by Measurement Lab," showing Test Server, Download, Upload, Latency and Retransmission, with consent to the data policy (including IP publication) required before testing. ✅
- **The Internet Health Test (internethealthtest.org)**: ❌ checked and found changed. The domain today (retrieved August 2026) serves unrelated SEO/content-farm pages (router-config guides, casino and easter-egg content) — it is no longer the M-Lab internet-health tool. A Wayback Machine capture from 2025-01-09 shows the same unrelated content was already there in early 2025, confirming the takeover predates this pass. The original Internet Health Test — a browser speed/health test M-Lab ran with partner organizations, historically including New America's OTI ⚠ — is gone from that domain; M-Lab's consumer testing now lives at speed.measurementlab.net. ✅ (The historical partner list for the original IHT could not be verified from a primary source this pass — Section 13.)
- Repository cross-reference: this repository has no internet-health-themed guide (a grep for "internet health" across the repo returned zero matches this pass) ⚠; the closest thematic neighbors are the observability guide referenced in Section 11 and the Singapore connectivity context in Section 10.

### 8.4 Industry

M-Lab explicitly welcomes commercial use: "Anyone may use M-Lab's services as long as they follow the AUP and obtain user consent," and clients need not be open source (the majority of NDT measurements come from independently developed clients, including closed source products). ✅ The integrations list (Section 7.4) shows ISPs, CDNs and tool vendors embedding M-Lab tests in consumer products.

---

## 9. Limitations — Bias, Attribution, and Coverage

M-Lab itself documents the major limitations of crowdsourced measurement; this section consolidates the verified statements plus the peer-reviewed critiques.

### 9.1 Self-Selection and Population Drift

M-Lab's policy blog is explicit: researchers "should look at client population changes when comparing statistics across different time periods," because "any change (to the Internet, client population, or service software) may lead to improving or declining performance measurements" — e.g. "improvement in an area may mean the network is getting better, or it may mean a few very fast clients are running a lot of tests." ✅ The blog's analysis recommendations include compensating for "client bias and population drift," using histograms and **logarithmic scales** (geometric means), and not oversimplifying multimodal distributions. ✅ Who runs M-Lab tests is not a random sample of internet users; it is people who chose to run a speed test, and that self-selection biases every aggregate. ✅

### 9.2 Last-Mile vs Middle-Mile Attribution

This is the single most important caveat for interpreting NDT results, and M-Lab states it in two places ✅:

- The ndt7 spec: NDT measures "what performance is possible with your device, your current internet connection (landline, Wi-Fi, 4G, etc.), the characteristics of your ISP and possibly of other ISPs in the middle, and the server being used" — "not necessarily a measurement of your last mile speed."
- The policy blog: M-Lab runs an **exclusively off-net platform** — servers sit at interconnection points, not inside access ISPs — so M-Lab "measures more than the last-mile network by design," and its data is best for representing "the full path from a user to the backbone of the Internet." On-net measurements (server inside the access ISP, e.g. Ookla's typical deployment) assess only that ISP's network; off-net measurements also capture the quality of the peering between the access ISP and transit providers. The FCC MBA Technical Index is quoted approvingly: "The use of both on-net and off-net nodes provides a measure of confidence in the test results." ✅

The practical consequence: a slow NDT result does not tell you whether the problem is the last mile, the ISP's peering, the middle-mile transit, or the test server — it requires triangulation with on-net tests and internal monitoring (Section 11.6).

### 9.3 Coverage Gaps

M-Lab's footprint is uneven: "The majority of sites are located in North America and Europe," with the develop page acknowledging the platform "seeks to build out a presence everywhere" and asking partners for help in under-covered regions. ✅ The two-pager's coverage statement ("US, Europe, Australia, and parts of Africa and Asia") reflects the historical concentration. ⚠ Users far from any site measure longer, more congested paths, and countries with few or no servers are systematically under-represented in the dataset. The host-managed model (Section 3.4) is explicitly a mechanism for providers in underserved regions to join the platform. ✅

### 9.4 Server-Selection and Methodology Debates

- **Server selection**: Locate picks "best" servers by geography/health/load (Section 7.1), and since the cloud expansion a fraction of traffic is deliberately routed to virtual servers — so which server class a test hits is a design decision, not a property of the user's connection. ✅
- **Physical vs cloud**: M-Lab's own QA experiment found cloud VMs produced "measurements consistent with the physical sites" with slightly lower loss/latency, but also documented a cloud node being overwhelmed during a load spike. ⚠ The methodology question — do cloud-hosted measurement servers bias results versus bare-metal sites? — is live and only partially answered in public posts.
- **Crowdsourced-vs-crowdsourced**: the peer-reviewed Paul et al. (IMC 2022) finding that Ookla medians can be ~2× M-Lab medians for the same subscription tier, city and ISP shows how much methodology (on-net vs off-net, multi-stream vs single-stream, congestion control) moves results — a documented, published debate, not speculation. ✅
- **Bias in aggregates**: M-Lab's own guidance (don't oversimplify, use log scales, watch population drift) is an admission that naive averages of its data mislead — the "averages tell a poor tale" paper (Section 6.6) makes the same point academically. ✅

### 9.5 Data-Quality Caveats

- **Switch discards**: M-Lab documented 2015-era performance degradation caused by switch discards from traffic microbursts, re-parsed historical data to add blacklist flags, and published the analysis. ✅
- **App-limited tests**: the ndt7 spec's `AppLimited` flag and the unified-view criterion excluding downloads "with only non-network bottleneck" acknowledge that some tests measure the client's own limits (device, Wi-Fi, browser) rather than the network. ✅
- **NAT and dynamic IPs**: research (the residential-capacity paper, Section 6.6) notes tests are indexed by client IP which "can be dynamic and/or obfuscated by NAT," complicating household-level inference. ✅
- **Host-managed data quality**: M-Lab itself says it "cannot make assertions about the quality of the data" from host-managed nodes and keeps it separate. ✅

---

## 10. The Singapore Angle — SIN Sites and Local Context

### 10.1 Verified Singapore Server Presence (Live Query, August 2026)

This pass verified M-Lab's Singapore presence directly against the **live Locate API v2** with a strict country query — `https://locate.measurementlab.net/v2/nearest/ndt/ndt7?country=SG&strict=true` — which returned three healthy NDT servers in Singapore (retrieved August 2026) ✅:

| Machine | Location (as returned) | Deployment class |
| --- | --- | --- |
| `mlab2-sin01.mlab-oti.measurement-lab.org` | Singapore, SG | Classic M-Lab physical site, code **sin01** (OTI-managed fleet) |
| `ndt-sin15830-931cb329.equinix.autojoin.measurement-lab.org` | Queenstown Estate, SG | **Equinix** host-managed/autojoin node |
| `ndt-sin396982-228eada0.googleoim.autojoin.measurement-lab.org` | Singapore, SG | **Google OIM** (Google Cloud) virtual node |

So the Singapore footprint today is a mix that mirrors the platform's three deployment models (Section 3): a legacy physical site (`sin01`), an Equinix-hosted autojoin node, and a Google-cloud virtual node. ✅ The `sin` site code confirms the historical SIN designation, and the autojoin machine IDs (`sin15830`, `sin396982`) show the newer IATA-based naming. ✅ Whether `sin01` still hosts multiple physical servers (the classic sites run 3–4) is not published in text ⚠ — the locate API returns the healthiest candidates, not the full inventory, and M-Lab's interactive infrastructure map (Section 3.5) is the authoritative per-server view.

Practical implication for Singapore-based testers: an NDT client in Singapore will be directed to one of these three machines (or the fourth candidate slot if one is unhealthy), with the Equinix and Google nodes running in data centers that Singapore's ISPs interconnect with — the off-net placement M-Lab designs for (Section 9.2). ✅

### 10.2 Singapore Context — Cross-References (Not Re-Derived Here)

This guide deliberately does not re-derive Singapore's broadband landscape; it cross-references the repository's Singapore guides, which were read this pass for context:

- **[StarHub: The Software Systems Landscape](../singapore/starhub_software_systems_guide.md)** — the repo's Singapore ISP deep-dive: StarHub Ltd (SGX: CC3) as one of Singapore's full-service operators, the BSS/OSS estate, the 4G/5G RAN, the **NGNBN** national fibre network (via Nucleus Connect) that underpins Singapore's fibre broadband market, and the **IMDA** regulatory context. M-Lab's Singapore servers are exactly the kind of off-net measurement points against which a StarHub (or any SG ISP) connection's real-world performance can be checked. ✅ (Cross-referenced, not re-derived.)
- **[Singapore GDP and Industry Distribution](../singapore/sg_gdp_industry_distribution.md)** — provides the macro context: Singapore's digital economy contributed S$106B (~12% of GDP) in 2023, with the ICT sector alone at ~5.2% of GDP, per the IMDA/MTI Digital Economy Report (2024). A connectivity-obsessed digital economy is precisely why independent performance measurement matters for the city-state. ✅ (Cross-referenced.)
- **[Singapore Government Securities](../singapore/singapore-government-securities-guide.md)** and **[Singaporean Overseas Retirement](../singapore/singaporean_overseas_retirement.md)** — the rest of the singapore/ folder; neither covers internet measurement, so no further cross-reference applies. ✅ (Checked this pass.)
- **IMDA**: the Infocomm Media Development Authority is Singapore's telecoms regulator; its digital-economy reporting and regulatory role are covered in the guides above. ⚠ No evidence was found this pass that IMDA itself uses M-Lab data in published regulatory reporting — do not assume a regulatory relationship exists (Section 13).

### 10.3 What the Singapore Angle Means for Measurement

- **Fibre-rich, dense market**: Singapore's broadband is dominated by NGNBN fibre (per the StarHub guide context), so last-mile links are typically fast; NDT results from Singapore therefore more often reflect middle-mile/peering and international transit quality than access-line speed — exactly the off-net measurement M-Lab is designed to capture (Section 9.2). ✅
- **Three server classes to compare**: a researcher or enterprise in Singapore can compare results across the sin01 physical site, the Equinix node and the Google node — a natural experiment in server-class effects (Section 9.4). ⚠
- **For the bank use case (Section 11)**: Singapore branch links measured against these SIN servers produce low-RTT baselines (a few ms to the Equinix/Google nodes), which makes latency-sensitive alerting meaningful — thresholds must be set against SG baselines, not global defaults. ✅

### 10.4 Running a Test From Singapore

Anyone in Singapore can verify the local footprint directly: the Locate API query used in Section 10.1 (`https://locate.measurementlab.net/v2/nearest/ndt/ndt7?country=SG&strict=true`) returns the current healthy Singapore machines with signed `wss://` URLs; point a browser at speed.measurementlab.net to run the consumer test (which uses the same Locate path), or use the `ndt7-client-go` reference client for scripted runs. ✅ Because the strict-country query returns only Singapore-hosted machines, results isolate the Singapore leg of the path — useful for enterprises comparing branch circuits inside the city-state. ✅

---

## 11. Cymbal Bank Worked Example — Branch Connectivity Monitoring

### 11.1 The Use Case

Cymbal Bank is a Singapore-headquartered bank (the repository's standing persona — see the banking guides) with branch and office locations across Singapore and the region. Branch connectivity is a quiet risk: a branch whose internet link degrades slowly — rising latency, creeping loss, throughput well under the subscribed line rate — produces bad customer experience (teller terminals, ATMs, branch Wi-Fi for staff and customers, video conferencing) long before the circuit fails outright. The bank's network team wanted an **independent, continuous view of branch link quality** that did not depend on the ISP's own portal or on the bank's internal monitoring alone.

The design pattern chosen is M-Lab-style measurement (Section 5.6), applied to the bank's own links, in three phases:

1. **Phase 1 — Public platform**: scheduled NDT-style tests from each branch/office to M-Lab's Singapore servers (the sin01 / Equinix / Google nodes verified in Section 10.1), using the Locate API for server selection. This is essentially the Murakami pattern (Section 7.4: an NDT test roughly every twelve hours, results stored for analysis), industrialized.
2. **Phase 2 — Hybrid**: add tests to **self-hosted measurement servers** (the host-managed model of Section 3.4, or private servers on the bank's own network edge and at a regional IX) to get on-net and controlled off-net references alongside the public ones.
3. **Phase 3 — Internal correlation**: merge M-Lab-style results with the bank's internal telemetry (SD-WAN probes, router/switch SNMP, ISP SLA reports) into one observability platform.

This guide covers Phase 1 in detail; Phases 2–3 are the pairing that the limitations section (11.6) makes non-negotiable.

### 11.2 Architecture

Per branch/office, a small **measurement agent** (a Docker container running an ndt7 client — `ndt7-client-go` is the reference; a browser-less scheduled runner avoids the 40-tests/day platform limit easily, since even an hourly schedule is only 24 tests/day) ✅:

- **Scheduler**: a cron/timed job runs the test every 6 hours per branch (4 tests/day/branch; 40 branches = 160 tests/day across the platform — trivially within M-Lab's rate limits and fair-use expectations, and far below anything that would trip admission control). ✅
- **Server selection**: call `https://locate.measurementlab.net/v2/nearest/ndt/ndt7?country=SG&strict=true` (or nearest, falling back through the candidate list), store the `machine` ID with each result so server effects can be analyzed (Section 9.4). ✅
- **Execution**: download and upload subtests over `wss://`, capturing the full ndt7 JSON result (Section 5.5).
- **Storage**: results land in the bank's data platform — raw ndt7 JSON archived to object storage (mirroring M-Lab's own GCS pattern, Section 6.4), parsed rows in a columnar store (BigQuery or ClickHouse), with the M-Lab-style annotations (site, machine, congestion control) preserved.
- **Observability**: Prometheus metrics from the agents (mirroring the host-managed node metrics, Section 3.4) scraped into the bank's Prometheus, visualized in Grafana, alerting via AlertManager — the same stack the repo's [OpenShift Workload Availability guide](openshift_workload_availability_guide.md) documents for platform health. ✅

### 11.3 Metrics Collected

Per test, the bank stores the full ndt7 metric set (Section 5.4) ✅:

- **Download and upload throughput** — `MeanThroughputMbps` (and goodput derivable from `AppInfo.NumBytes`/`ElapsedTime`).
- **Round-trip time** — `MinRTT` (the noise-robust baseline), plus `RTT`/`RTTVar` for jitter-ish variability.
- **Packet loss** — `LossRate` and the retransmission ratio (`BytesRetrans`/`BytesSent`).
- **Context** — `CongestionControl` (BBR vs Cubic affects throughput readings), test `date`/time, server `machine` and site code, client metadata (agent version).

### 11.4 Dashboards (Grafana Panels)

Per the repository's observability conventions (Grafana-style panels, SLO framing — see the OpenShift guide), the branch-connectivity dashboard set:

- **Daily throughput distribution** — a histogram (log-scaled buckets, per M-Lab's own analysis guidance in Section 9.1) of download and upload Mbps per branch per day, with the subscribed line-rate marker overlaid; multimodal distributions immediately expose "fast normally, slow at peak" branches.
- **Loss spikes panel** — time series of LossRate and retransmission ratio per branch, with a threshold line; a spike-and-recover pattern typically precedes circuit flap or congestion.
- **Latency trends** — MinRTT per branch over 30/90 days, with a moving baseline band; a step-change upward that persists across tests is a path change (ISP reroute) signature, corroborated by traceroute-style path data if available.
- **Server split** — results colored by which M-Lab machine served them (sin01 vs Equinix vs Google node), so the team can see server-class effects and avoid false alarms when a server is the variable.
- **Coverage/health** — tests per branch per day, missed-schedule count (the agent or link was down), and fraction of tests that failed at the platform level (admission control, maintenance).
- **Branch scorecard** — a per-branch summary row (last-7-days median download/upload Mbps, MinRTT, loss, retransmission, tests run) in a Grafana table panel, color-coded against the alert thresholds below, so the network team gets one glanceable view of every site.

### 11.5 Alerting Thresholds

Thresholds below are **design proposals for this guide, not M-Lab-published values** ⚠ — they must be tuned against each branch's baseline:

- **Loss**: sustained `LossRate` > 2% across 2 of 3 consecutive tests in a 24-hour window → warning; > 5% → page. (The ndt7 spec notes some loss is normal and healthy; sustained double-digit loss is a systemic path problem. ✅ for the underlying logic.)
- **Latency**: branch `MinRTT` > 3× its 30-day rolling median for 2 consecutive tests → warning (path change); with accompanying loss → page.
- **Throughput**: median download < 50% of the subscribed line rate for a full day → warning; < 25% → page. (SG fibre lines make the subscribed rate a meaningful reference — Section 10.3.)
- **Retransmission**: retransmission ratio > 5% sustained → warning (congestion or bufferbloat signature).
- **Missed tests**: 0 tests received from a branch for 24 hours → warning (agent down or link down); escalate to the ISP only after internal checks (Section 11.6).

Alert routing follows the repo's SLO practice: warnings to the network team's channel, pages only for sustained, corroborated conditions — never a single-test blip, because a single NDT result is a sample of a path, not a verdict (Section 9).

### 11.6 Limitations — Why Public-Server Measurement Must Be Paired

The bank's measurement program inherits every limitation documented in Section 9, and the design must say so explicitly ✅:

- **Public-server contention**: the Equinix and Google M-Lab nodes are shared, so a slow test may reflect other users' load or the node's own state — the locate API's admission control and the bank's own `machine` tagging mitigate but do not eliminate this. ⚠
- **Last-mile vs middle-mile attribution**: NDT to M-Lab measures the whole path (Section 9.2) — a slow result cannot by itself indict the branch's ISP line; it may be the bank's own Wi-Fi/switch, the ISP's CPE, or regional transit. The bank must pair M-Lab-style results with **internal monitoring** (probe to the branch gateway, SD-WAN path metrics, ISP portal/SLA data) before raising tickets with the ISP — this pairing is the whole point of Phase 3.
- **Rate limits and fair use**: 40 tests/client/day cap (Section 3.6) is generous for this design but must be enforced in the agent to avoid 204s. ✅
- **Platform availability**: M-Lab's best-effort business-hours SLO (Section 3.6) means the bank's own monitoring must not depend on M-Lab availability for alerting on total outage.
- **Coverage**: if a branch is far from any M-Lab server (regional offices outside Singapore), its results measure a longer path — the bank should prefer self-hosted measurement servers for those sites (Phase 2), which is precisely the host-managed model M-Lab now offers organizations (Section 3.4).

### 11.7 Lessons and the Architect's Checklist

- Use the **Locate API** for server selection and record the machine ID in every result — server is a variable, not a constant.
- Schedule modestly (4–6 tests/day/branch) — well under rate limits, enough for daily medians.
- Store **raw JSON + parsed columns** (M-Lab's own two-tier pattern), keep annotations, reprocess later if metrics change.
- Visualize with **log-scale histograms and medians**, not linear averages (M-Lab's analysis guidance, Section 9.1).
- **Alert on corroborated, sustained conditions**, and never on a single test.
- **Pair with internal monitoring** before contacting ISPs; treat M-Lab-style data as the independent second opinion, not the verdict.
- **Reuse the pattern**: the same agent, dashboards and thresholds generalize to office links, DR sites, and cloud egress paths; the bank's network observability estate (Prometheus/Grafana per the repo's OpenShift guide) is the natural home.

---

## 12. Claims Audit — Verified, Flagged, Unverified

All sources below were retrieved (or directly linked from a retrieved page) in **August 2026**. ✅ = verified this pass; ⚠ = flagged/approximate/dated; ❌ = checked and found incorrect/absent.

### Verified (✅)

| Claim | Source (retrieved August 2026) |
| --- | --- |
| M-Lab founded 2009 by New America's OTI, the PlanetLab Consortium, Google, and academic researchers | https://www.measurementlab.net/about/ |
| 2008 Vint Cerf conversations; three researcher problems (servers, data sharing, no public performance resource) | https://www.measurementlab.net/about/ |
| Individual founders: Cerf, Meinrath, Peterson, Slater, Stuart, Whittaker | https://www.measurementlab.net/who/ |
| Not-for-profit; fiscally sponsored project of Superbloom; maintainers/committees | https://www.measurementlab.net/who/ |
| Mission: "measure the Internet, save the data, and make it universally accessible and useful" | https://www.measurementlab.net/ , https://www.measurementlab.net/publications/ |
| All data open, all tools open source, dedicated resources per test | https://www.measurementlab.net/about/ |
| Physical sites = 3–4 servers + switch; virtual sites 1+ VMs; 2022 cloud start; 2025 host-managed pilot | https://www.measurementlab.net/status/ |
| July 2022 cloud pilot (<10% of clients); April 2023: virtual site in all 35 GCP regions, 41 VMs, 13% cloud traffic | https://www.measurementlab.net/blog/2022-mlab-to-the-cloud/ , https://www.measurementlab.net/blog/virtual-sites-gcp/ |
| Host-managed model introduced 2024; hardware/network requirements; ports 80/443 + 9990–9999; autojoin flow | https://www.measurementlab.net/tests/ndt/ , https://www.measurementlab.net/contribute/host-managed/ |
| "Over 125 sites" (develop page) and "over 130 servers"/"100+ servers" (two-pager) — ⚠ dated | https://www.measurementlab.net/develop/ , https://www.measurementlab.net/publications/mlab-brief-overview.pdf |
| Best-effort business-hours SLO; 2019 NDT availability 99–99.9%; 40 tests/client/day limit | https://www.measurementlab.net/develop/ |
| No PII stored on platform; IP addresses collected and published; data retained indefinitely; GDPR legitimate interest; privacy policy v5 (May 3, 2026) | https://www.measurementlab.net/develop/ , https://www.measurementlab.net/privacy , https://www.measurementlab.net/tests/ |
| Current tests: NDT, Neubot DASH, Reverse Traceroute, WeHe, MSAK; core services: Packet Headers, TCP INFO, Traceroute | https://www.measurementlab.net/tests/ |
| Retired: Glasnost (07/07/2017), NPAD, Pathload2 (12/21/2012), Shaperprobe (05/11/2015), Windrider (01/17/2013), BISmark, MobiPerf, OONI Probe, SamKnows; retired core: SideStream, Paris Traceroute | https://www.measurementlab.net/tests/ |
| Neubot by Nexa Center (Politecnico di Torino); client retired Jan 2019; DASH still hosted; Neubot data not in BigQuery | https://www.measurementlab.net/tests/neubot/ |
| NDT purpose (RFC 3148 bulk transport, single stream); Internet2 origin; hosted since 2009; speed.measurementlab.net front-end | https://www.measurementlab.net/tests/ndt/ |
| web100 (2009-02-18→2019-11-20, Reno), ndt5 (2019-07-18→present, Cubic, tcp-info), ndt7 (2020-02-18→present, BBR, ports 80/443) | https://www.measurementlab.net/tests/ndt/ , https://www.measurementlab.net/tests/ndt/ndt7 , https://www.measurementlab.net/tests/ndt/ndt5 |
| ndt5+raw ended January 2024; ndt5+ws/wss remain; ndt7 recommended | https://www.measurementlab.net/tests/ndt/ndt5 |
| ndt7 spec v0.11.0: WebSocket+TLS, single TCP connection, /ndt/v7/{download,upload}, subprotocol net.measurementlab.ndt.v7, ≤10s, goodput, TCP_INFO diagnostics, not last-mile by design | https://raw.githubusercontent.com/m-lab/ndt-server/master/spec/ndt7-protocol.md |
| ndt7 data format v0.3.0: gzip JSON `ndt7-<subtest>-<ts>.<uuid>.json.gz` with full result record | https://raw.githubusercontent.com/m-lab/ndt-server/master/spec/data-format.md |
| ndt7 BigQuery schema fields (MeanThroughputMbps, MinRTT, LossRate, BBRInfo, TCPInfo...) | https://www.measurementlab.net/tests/ndt/ndt7 |
| Free BigQuery access via M-Lab Discuss group whitelist; measurement-lab project; SDK/console access | https://www.measurementlab.net/quickstart/ |
| ndt7_union recommended view; ndt7, ndt7_dynamic, ndt5, unified_*, ndt_intermediate, ndt_raw, legacy datasets (2009-02-18 onward) | https://www.measurementlab.net/tests/ndt/ |
| Raw archives in GCS `archive-measurement-lab`; .pcap per NDT test not indexed in BigQuery; periodic reprocessing | https://www.measurementlab.net/tests/ndt/ , https://www.measurementlab.net/data/docs/ |
| Locate API v2 endpoints, services (ndt/ndt5, ndt/ndt7, wehe/replay, neubot/dash), access_token, country/region/strict/org/site params | https://www.measurementlab.net/develop/locate-v2/ , https://raw.githubusercontent.com/m-lab/locate/main/USAGE.md |
| FCC: M-Lab hosted SamKnows off-net servers for Measuring Broadband America 2009–2019; relationship ended over open-source requirement | https://www.measurementlab.net/tests/samknows/ |
| Regulator partners list (FCC, EC, EETT, RTR, OCECPR) — ⚠ dated two-pager | https://www.measurementlab.net/publications/mlab-brief-overview.pdf |
| On-net vs off-net methodology; M-Lab is exclusively off-net; FCC MBA Technical Index quote | https://www.measurementlab.net/blog/mlab-data-policy-advocacy/ |
| CCR 2022 paper (Gill, Diot, Ohlsen, Mathis, Soltesz); Ukraine IMC 2022; Paul et al. IMC 2022 (Ookla ~2× M-Lab); UNSW 2019 (63M tests); Australia 2015 (96,882 households); CoNEXT 2016 latency variation | https://www.measurementlab.net/publications/ , https://ccronline.sigcomm.org/2022/ccr-january-2022/m-lab-user-initiated-internet-data-for-the-research-community/ |
| M-Lab tools: M-Lab Measure, Murakami (NDT ~12-hourly), speed.measurementlab.net; integrations list | https://www.measurementlab.net/data/tools/ |
| Singapore: 3 live NDT servers (mlab2-sin01, Equinix autojoin, Google OIM autojoin) via strict country query | https://locate.measurementlab.net/v2/nearest/ndt/ndt7?country=SG&strict=true |
| internethealthtest.org no longer the M-Lab tool (content-farm takeover, already present by 2025-01-09 per Wayback) | https://internethealthtest.org/ , https://web.archive.org/web/20250109115149/https://internethealthtest.org/ |

### Flagged (⚠)

| Claim | Status / source |
| --- | --- |
| Current (2026) total site/server count in text | ⚠ No current text total published; interactive map only; "over 125 sites" is undated/dated (https://www.measurementlab.net/develop/) |
| 35 GCP regions / 41 virtual servers / 13% cloud traffic | ⚠ April 2023 snapshot, certainly outdated (https://www.measurementlab.net/blog/virtual-sites-gcp/) |
| 750TB+ open data; 200M+ tests; 200K tests/day; 12 tools; 14+ papers; 4+ regulators | ⚠ Two-pager is dated and internally inconsistent (600TB vs 750TB) (https://www.measurementlab.net/publications/mlab-brief-overview.pdf) |
| "Towards an Open, Verifiable Internet" paper title | ❌ not found on the publications page or elsewhere this pass (https://www.measurementlab.net/publications/) |
| Glasnost developer = Max Planck Institute for Software Systems | ⚠ not named on the current M-Lab page (https://www.measurementlab.net/tests/) |
| Original Internet Health Test partner list | ⚠ not verifiable from a primary source this pass |
| web100-era ".tap" file extension | ⚠ current docs describe snaplogs/web100 datatype, not ".tap" by name (https://www.measurementlab.net/tests/ndt/) |
| DASU / PathNeutral tests | ❌ absent from the current roster (https://www.measurementlab.net/tests/) |
| "Replay" as a standalone test | ⚠ not on tests page; exists as `wehe/replay` service in Locate API v2 |
| IMDA (SG) use of M-Lab data | ⚠ no evidence found this pass |
| Wikipedia "Measurement Lab" article | ❌ no article under that exact name (https://en.wikipedia.org/wiki/Measurement_Lab) |
| fcc.gov pages (MBA program, speed-test app) | ⚠ blocked automated retrieval in August 2026; FCC-side verification not possible this pass |
| Post-2019 FCC use of M-Lab data | ⚠ unverified (see Section 13) |

---

## 13. What Could Not Be Verified

The following could not be verified this session, despite attempts; nothing below should be treated as fact:

- **Current (August 2026) total number of M-Lab sites and servers in text form.** The status page presents an interactive map only; published text figures (125+ sites, 130+ servers, 41 virtual servers) are dated. The live Locate API proves individual machines but not totals. ⚠
- **FCC-side confirmation of anything.** fcc.gov pages — including the Measuring Broadband America page M-Lab links to — blocked automated retrieval this pass. The 2009–2019 M-Lab/SamKnows/MBA relationship and its 2019 end are verified from M-Lab's own pages, but any FCC-published statement could not be fetched. ⚠
- **What the FCC has done with M-Lab data since 2019** (e.g. any use in later broadband reports, rulemakings, or the FCC speed-test app). No primary source was retrievable this pass. ⚠
- **The paper "Towards an Open, Verifiable Internet."** No such title was found on M-Lab's publications page or via retrieval this pass; the verified anchor paper is "M-Lab: User Initiated Internet Data for the Research Community" (CCR, January 2022). The brief's assumed title may be a paraphrase — treat it as unverified. ❌/⚠
- **The original Internet Health Test's partner list and shutdown date.** The domain internethealthtest.org now serves unrelated content (already the case by January 2025 per the Wayback Machine); the historical partnership (M-Lab and partner organizations) could not be confirmed from a primary source this pass. ⚠
- **The specific ".tap" file extension for web100-era NDT data.** The current documentation describes snaplog records and the web100 datatype but does not name ".tap"; the extension is legacy knowledge not re-verified. ⚠
- **Glasnost's developer attribution** (historically the Max Planck Institute for Software Systems): not named on the current M-Lab tests page. ⚠
- **IMDA's use of M-Lab data.** No evidence found this pass; the IMDA references in this guide are cross-references to sibling repo guides, not M-Lab-related claims. ⚠
- **Whether Google Cloud's current region count and M-Lab's current cloud-server total** exceed the April 2023 figures (almost certainly, but unverified). ⚠
- **Wikipedia coverage**: the article "Measurement Lab" does not exist under that exact name; no other encyclopedia entry was verified. ⚠

---

## 14. Glossary

- **NDT (Network Diagnostic Tool)** — the platform's flagship end-user test: single-stream download/upload throughput plus latency and loss diagnostics, per RFC 3148 "bulk transport" semantics. ✅
- **ndt7** — the current NDT protocol: WebSocket+TLS, one TCP connection per subtest, JSON control messages, TCP BBR where available, ports 80/443, ~10-second tests. ✅
- **ndt5** — the backward-compatible NDT protocol (Cubic, tcp-info), still served for legacy clients. ✅
- **web100** — the original NDT datatype (2009–2019) built on the web100 kernel instrumentation; the historical "M-Lab 1.0" era. ✅
- **tcp-info / TCP_INFO** — kernel TCP statistics snapshots that replace web100 as the platform's TCP instrumentation source. ✅
- **BBR / Cubic** — TCP congestion control algorithms; ndt7 uses BBR where available with Cubic fallback; the algorithm is recorded per test. ✅
- **Goodput** — application-level throughput excluding protocol overhead; ndt7's headline metric. ✅
- **MinRTT / RTTVar / LossRate** — minimum round-trip time, RTT variance, and connection-lifetime loss rate; core NDT result fields. ✅
- **Bufferbloat** — excessive queueing in network buffers that inflates latency under load; NDT's queue-buildup and RWndLimited diagnostics speak to it. ✅
- **BigQuery** — Google Cloud's serverless data warehouse; M-Lab's parsed dataset lives in the public `measurement-lab` project. ✅
- **GCS** — Google Cloud Storage; M-Lab's raw archives (`archive-measurement-lab`). ✅
- **Locate API v2** — the GCP-hosted server-selection service that returns signed measurement URLs; formerly the "mlab-ns" name-server era ⚠. ✅
- **mlab-ns** — the legacy M-Lab name-server/lookup service, referenced in M-Lab's retirement posts; superseded by the Locate service. ⚠
- **Autojoin / autonode** — the registration and deployment machinery for host-managed nodes. ✅
- **Host-managed** — M-Lab's 2024+ deployment model where third-party providers run M-Lab software on their own machines; data kept separate. ✅
- **On-net / off-net** — server placement inside vs outside the access ISP's network; M-Lab is exclusively off-net. ✅
- **M-Lab 2.0** — the 2019 platform generation that enforced the open-source container requirement. ✅
- **OTI / PlanetLab / Superbloom** — founding/operating organizations: New America's Open Technology Institute, the PlanetLab Consortium, and the fiscal sponsor Superbloom. ✅
- **AUP** — M-Lab's Acceptable Use Policy, governing client behavior and consent. ✅
- **PII** — personally identifiable information; M-Lab stores none, though public IPs are published. ✅
- **SIN / site codes** — airport-style location codes (sin01, lga05, bom01) used in M-Lab machine names and site identity. ✅
- **IATA code** — the airport code used by host-managed nodes for their location (e.g. SIN). ✅
- **SLO** — service-level objective; M-Lab's is best-effort business-hours only. ✅

## 15. Cross-References and Further Reading

**Repository guides (this repo, read this pass for style and cross-reference):**

- [Large-Scale Web Data Acquisition — the Harvest at Scale](large_scale_web_data_acquisition_guide.md) — the data-acquisition patterns guide; M-Lab data is fetched via BigQuery/APIs and GCS rather than scraped (Section 7.5).
- [StarHub: The Software Systems Landscape](../singapore/starhub_software_systems_guide.md) — Singapore ISP context: NGNBN fibre, Nucleus Connect, StarHub's networks, and the IMDA regulatory setting (Section 10.2).
- [Singapore GDP and Industry Distribution](../singapore/sg_gdp_industry_distribution.md) — IMDA digital-economy context (S$106B, ~12% of GDP in 2023) (Section 10.2).
- [Singapore Government Securities](../singapore/singapore-government-securities-guide.md) and [Singaporean Overseas Retirement](../singapore/singaporean_overseas_retirement.md) — checked; no internet-measurement overlap.
- [OpenShift Workload Availability](openshift_workload_availability_guide.md) — Prometheus/AlertManager/Grafana observability conventions reused in the Cymbal Bank dashboards (Section 11).
- Banking guides (e.g. [Standard Chartered guide](../banking/standard_chartered_guide.md), [Insurance Products guide](../banking/insurance_products_processes_compliance_guide.md)) — the Cymbal Bank persona and worked-example conventions. ⚠ Note: this guide intentionally names no other bank; Cymbal Bank is the only bank persona in this file.

**Primary external sources (all retrieved August 2026 unless noted):**

- M-Lab about/history: https://www.measurementlab.net/about/
- M-Lab who we are (governance, founders, partners): https://www.measurementlab.net/who/
- M-Lab tests and status: https://www.measurementlab.net/tests/ , https://www.measurementlab.net/status/
- NDT documentation and BigQuery views: https://www.measurementlab.net/tests/ndt/ , https://www.measurementlab.net/tests/ndt/ndt7
- ndt7 protocol and data-format specs: https://github.com/m-lab/ndt-server (spec/ndt7-protocol.md, spec/data-format.md)
- BigQuery QuickStart: https://www.measurementlab.net/quickstart/
- Privacy policy (v5, May 2026): https://www.measurementlab.net/privacy
- Locate API v2: https://www.measurementlab.net/develop/locate-v2/ , https://github.com/m-lab/locate (USAGE.md)
- Host-managed docs: https://www.measurementlab.net/contribute/host-managed/
- Publications: https://www.measurementlab.net/publications/ ; platform paper: https://ccronline.sigcomm.org/2022/ccr-january-2022/m-lab-user-initiated-internet-data-for-the-research-community/
- FCC relationship (SamKnows page): https://www.measurementlab.net/tests/samknows/
- Data tools: https://www.measurementlab.net/data/tools/
- Consumer speed test: https://speed.measurementlab.net/

Measurement is a public good, and M-Lab's seventeen-year experiment has proven that a consortium can run it openly: volunteer servers, open-source tools, and a dataset anyone can query — every test archived, every methodology auditable, every number traceable to a packet on the measured internet.
