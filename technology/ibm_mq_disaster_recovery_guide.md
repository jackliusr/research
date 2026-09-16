# IBM MQ Disaster Recovery: HA Is Not DR — A Comprehensive Guide

**Jack Liu Shurui, Solution Architect**

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** Integration / Middleware / Platform (technology/)
> **Audience:** Solution architects, MQ administrators, platform and resilience engineers, integration architects operating an IBM MQ estate
> **Last Updated:** September 2026

*A comprehensive deep-dive on IBM MQ disaster recovery — the design and operational reality of the messaging layer that most banks cannot turn off. It covers the one distinction the whole discipline turns on (HA is not DR), the recovery substrate (persistent versus non-persistent messages, circular versus linear logging, log replay, media images), the damaged-object recovery path, the local resilience family (multi-instance queue managers, RDQM HA, Native HA, the MQ Appliance), the site-loss architectures (RDQM DR, DR/HA RDQM, Native HA Cross-Region Replication and In-Region Replication, appliance DR, storage-level replication), failover and client-side recovery (queue manager identity, automatic client reconnection, the CCDT, in-doubt work), the blunt truth about what a queue manager backup captures and what it does not, cluster and estate-level recovery including certificates and CHLAUTH, the DR testing discipline, the regulated-institution angle, and a fictional Cymbal Bank worked example with a tiered topology, a failover runbook, a test plan, and an honest statement of the single points of failure that remain.*

**Cross-references:** [enterprise_middleware_integration_platform_guide.md](enterprise_middleware_integration_platform_guide.md) (the integration **platform** landscape and the MQ object model — the platform overview this guide assumes and does not re-derive), [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) (the generic cross-platform **reliability** discipline — delivery semantics, idempotency, dedupe keys, the transactional outbox; this guide points there rather than repeating it), [kafka_alternatives_guide.md](kafka_alternatives_guide.md) (broker selection; where IBM MQ sits in the wider comparison), [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) (the integration patterns a surviving MQ estate still carries), [camel_camelk_research.md](camel_camelk_research.md) and [axway_transfer_cft_guide.md](axway_transfer_cft_guide.md) (integration and managed-file-transfer tooling that must inherit the same recovery guarantees — lightly), [ibm_cloud_guide.md](ibm_cloud_guide.md) (IBM's own cloud and platform estate, MQ on OpenShift and the Cloud Paks — lightly), [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) (the **resilience framework** — impact tolerances, regulatory expectations, the evidence model; condensed here, owned there), [../banking/swiftnet_fileact_guide.md](../banking/swiftnet_fileact_guide.md) and [../banking/swift_alliance_access_guide.md](../banking/swift_alliance_access_guide.md) (the SWIFT estate that rides on MQ and file transfer in many banks), [../banking/payment_rails_guide.md](../banking/payment_rails_guide.md) (the payment rails whose availability the messaging layer underwrites), [../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md) (the cores behind the MQ interfaces), [../banking/mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md) (Singapore regulatory context).

**How to read this guide.** §1 is the framing — the one-paragraph answer, the HA-is-not-DR distinction, and the key-facts table. §2 is the recovery substrate — what actually survives what, and the circular/linear logging decision that determines your recovery ceiling. §3 is the damaged-object recovery path — media images, `rcrmqobj`, and the operational sequence. §4 is the local resilience family — multi-instance, RDQM HA, Native HA, and the MQ Appliance, with the comparison table. §5 is the site-loss family — RDQM DR, DR/HA RDQM, Native HA CRR and IRR, appliance DR, and storage-level replication. §6 is failover and client-side recovery. §7 is backup, restore, and configuration recovery. §8 is cluster and estate-level recovery, including the certificate problem. §9 is the DR testing discipline. §10 is the regulated-institution angle. §11 is the fictional Cymbal Bank worked example. §12 is the claims audit (✅ verified / ⚠ flagged / ❌ rejected, each with version gate and source), §13 is "What Could Not Be Verified", §14 is the glossary, §15 is the cross-reference map and further reading, §16 is the closing summary. Cross-reference convention: sibling guides in `technology/` are plain filenames; guides in `banking/` are prefixed `../banking/`; repo-root files are prefixed `../`. **Integrity convention:** ✅ = verified this pass against a primary IBM source (source and version given in §12); ⚠ = flagged / partially verified / vendor-blog-sourced / version-gated and not re-confirmed for every release; ⚠-knowledge = well-documented industry knowledge not re-verified live this pass. **Version note:** IBM MQ's HA/DR feature set is release-gated, so every availability claim in this guide carries a version and, where relevant, a date. No feature names, node counts, or version numbers were invented.

---

## Table of Contents

1. [The Overview: HA Is Not DR](#1-the-overview-ha-is-not-dr)
2. [The Recovery Substrate: Logs, Persistence, and What Survives](#2-the-recovery-substrate-logs-persistence-and-what-survives)
3. [The Media-Image Recovery Path: Rebuilding a Damaged Object](#3-the-media-image-recovery-path-rebuilding-a-damaged-object)
4. [The Local Resilience Family: MIQM, RDQM, Native HA, and the Appliance](#4-the-local-resilience-family-miqm-rdqm-native-ha-and-the-appliance)
5. [The Site-Loss Architectures: RDQM DR, DR/HA RDQM, and Native HA CRR](#5-the-site-loss-architectures-rdqm-dr-drha-rdqm-and-native-ha-crr)
6. [Failover and Client-Side Recovery: Identity, Reconnection, In-Doubt Work](#6-failover-and-client-side-recovery-identity-reconnection-in-doubt-work)
7. [Backup, Restore, and Configuration Recovery](#7-backup-restore-and-configuration-recovery)
8. [The Cluster and Estate-Level Recovery](#8-the-cluster-and-estate-level-recovery)
9. [The DR Testing Discipline](#9-the-dr-testing-discipline)
10. [The Regulated-Institution Angle](#10-the-regulated-institution-angle)
11. [The Worked Example: Cymbal Bank's MQ DR Design](#11-the-worked-example-cymbal-banks-mq-dr-design)
12. [The Claims Audit](#12-the-claims-audit)
13. [What Could Not Be Verified](#13-what-could-not-be-verified)
14. [Glossary](#14-glossary)
15. [Cross-References and Further Reading](#15-cross-references-and-further-reading)
16. [The Closing Summary](#16-the-closing-summary)

---

## 1. The Overview: HA Is Not DR

### 1.1 The One-Paragraph Answer

**IBM MQ disaster recovery is the discipline of keeping the queue manager — and the messages inside it — available across the loss of a site, and it is a different problem from keeping it available across the loss of a process or a node.** IBM ships several distinct mechanisms for this, and they do not overlap as neatly as vendor diagrams suggest. Locally, you have multi-instance queue managers (shared storage, active/standby, from WebSphere MQ 7.0.1), RDQM HA (three Linux nodes, DRBD replication, Pacemaker quorum, IBM MQ Advanced entitlement), Native HA (three nodes, log replication, container-first and on Linux in the 9.4 documentation), and the MQ Appliance HA pair. Across sites, you have RDQM DR and DR/HA RDQM (Linux only, IBM MQ Advanced), MQ Appliance DR (including DR between HA groups from 9.3.2), and — the newest and most significant addition to this list — Native HA Cross-Region Replication (CRR), introduced in IBM MQ 9.4.2 on 27 February 2025, complemented in IBM MQ 10.0 by In-Region Replication (IRR). Under all of it sits the recovery substrate: the queue manager's log, the persistence attribute on every message, and the choice between circular and linear logging that decides whether a damaged object can be rebuilt from a media image at all. The operational reality is that most MQ estates have far better HA than DR, because HA is a product configuration and DR is a programme — an inventory, an architecture per criticality tier, a runbook somebody has actually executed, a certificate and configuration discipline, and a test plan that produces dated evidence. **The failure mode is not that IBM MQ cannot recover a site. It is that nobody ever proved it would.**

### 1.2 The Distinction the Whole Guide Turns On

**HA is not DR.** This is not a slogan; it is the single most consequential architectural error in messaging estates, and it is made constantly because the two words are used interchangeably in status reports.

| | High availability (HA) | Disaster recovery (DR) |
|---|---|---|
| Protects against | Process failure, node failure, storage-path failure, planned maintenance on one host | Loss of an entire site — power, network, data centre, region, or the platform the site runs on |
| Scope | Within one site, usually one storage fabric or one Kubernetes cluster | Across sites or regions |
| Typical mechanism | Active/standby takeover; a surviving copy of the data on another node | A replication link plus an orchestrated role change |
| Data loss on failover | Effectively none for persistent messages in RDQM HA and Native HA (synchronous or quorum-acknowledged replication) | Depends entirely on synchronous versus asynchronous replication |
| Operator involvement | Automatic, or a switchover command | Usually an explicit, authorised promotion decision |
| Typical RTO | Seconds to a couple of minutes | Tens of minutes to hours — dominated by the decision, not the technology |
| What it does not give you | Any protection whatsoever against site loss | Any protection against a single node dying inside a healthy site |

Three consequences follow, and the rest of this guide is largely the elaboration of them:

- **A site-loss event takes out the entire HA group at once.** An RDQM HA group is three nodes in one place with a shared quorum; a Native HA group is three instances fed by block storage in one region; a multi-instance pair is two nodes against one shared file system. If the site goes, the group goes, and HA has contributed nothing to the recovery.
- **HA can hide the absence of DR for years.** A multi-instance queue manager that has survived two real node failures is a *proven* HA solution and simultaneously an estate with no site-loss answer at all. The evidence trail looks impressive right up to the moment it is irrelevant.
- **DR without a rehearsed runbook is a hypothesis.** The technology can be perfectly configured and the recovery still fail, because a certificate expired, a CCDT was never republished to the DR endpoint, CHLAUTH rules were never exported, or the failback was never tested because the last three exercises only ever failed *forward*.

### 1.3 What This Guide Owns, and What It Deliberately Does Not

This is the first dedicated IBM MQ guide in this library, and the first dedicated DR/HA treatment for any messaging middleware in it. It therefore **owns**: MQ's logging and recovery mechanics (§2, §3), its HA and DR product options and their version gates (§4, §5), site-failover architectures (§5), the backup-and-restore reality (§7), cluster and estate-level recovery (§8), and the operational DR discipline for an MQ estate (§9–§11).

It deliberately **does not re-derive**:

- Delivery semantics, producer acks and retries, consumer commit discipline, idempotency and dedupe stores, dead-letter queue theory, exactly-once mechanisms, or the transactional outbox and event-sourcing patterns. Those belong to [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) — go there for idempotency patterns and outbox design; this guide assumes them and only points at where MQ failover creates the duplicate-delivery window that makes them necessary (§6.5).
- The middleware platform landscape, the MQ object model, the vendor taxonomy, and the MQ-on-z/OS boundary. Those belong to [enterprise_middleware_integration_platform_guide.md](enterprise_middleware_integration_platform_guide.md).
- Impact tolerances, business-continuity regulatory expectations, and the evidence model of an operational-resilience programme. Those belong to [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) — §10 here condenses the messaging-layer mapping and points there for the framework.

### 1.4 RPO, RTO, and What They Mean for Messages Specifically

**RPO (recovery point objective)** is how much data you are willing to lose, expressed as time. **RTO (recovery time objective)** is how long you are willing to be unavailable, also expressed as time. Both are business statements, not technology settings. For a messaging layer they have a specific flavour that generic DR discussions miss:

- **Message-layer RPO is a message count as much as a duration.** "Five minutes" is meaningless without a rate. At 3,000 messages per second, five minutes of RPO is 900,000 messages — and for persistent messages on a payment path, every one of them is a financial obligation, not a telemetry sample. State message-layer RPO in both units, and ask which messages were in flight.
- **Message-layer RTO must include consumer replay, not just queue manager availability.** A queue manager that is up in 90 seconds while consuming applications are still retrying a stale connection is not recovered. The end-to-end RTO for a messaging service is: queue manager available + channels re-established + consumers reconnected + the backlog drained + the reversal or confirmation of any in-doubt unit of work.
- **RPO for non-persistent messages is not a number — it is "everything in flight."** See §2.2. If a queue carries non-persistent messages, the honest RPO for that queue on any failover is "all of them," and the honest RTO is "zero, because they never come back."
- **RPO is a property of the replication mode, not of the DR product.** RDQM DR lets you choose synchronous or asynchronous replication, with explicitly different data-loss consequences (§5.1). DR/HA RDQM's inter-site link is *always* asynchronous (§5.2). Native HA CRR is asynchronous (§5.3). Only the local HA mechanisms in §4 give you an effectively-zero-loss guarantee, and they are not DR.

### 1.5 Key Facts at a Glance

| Fact | Value | Version gate | Verdict |
|---|---|---|---|
| Unit of recovery | The queue manager (its data directory, object definitions, and recovery log) | All versions | ✅ |
| Circular logging | Restart recovery only; no media recovery; no damaged-object recovery; default | All versions | ✅ |
| Linear logging | Restart *and* media recovery; enables media images and `rcrmqobj`; required for damaged-object rebuild | All versions | ✅ |
| Log type changeable after creation | Yes, via `migmqlog` | From IBM MQ 9.1.0 | ✅ |
| Automatic log management | `LogManagement` attribute in qm.ini `Log` stanza, Automatic or Manual | From IBM MQ 9.1.0 | ✅ |
| Non-persistent messages on failover | Discarded — they do not survive the queue manager stopping | All versions | ✅ |
| Multi-instance queue manager | One QM, shared network file system, active + standby, max two instances | From WebSphere MQ 7.0.1 | ✅ |
| RDQM HA | Three Linux servers, synchronous DRBD replication, Pacemaker quorum, floating IP | IBM MQ Advanced; RHEL 7.6+/8.2+ (9.2 docs), RHEL 8.8+/9.2+ x86-64 (9.4 docs) | ✅ |
| RDQM DR | Primary/secondary pair, DRBD, manual promotion, sync *or* async | IBM MQ Advanced, Linux | ✅ |
| DR/HA RDQM | HA group at one site failing over to an HA group at another; inter-site replication always asynchronous | New in IBM MQ 9.2.0 | ✅ |
| Native HA | Three nodes (active + two replicas), log replication, quorum ≥ 2 | Documented in 9.2 for containers; 9.4 documentation adds Linux | ✅ (intro release ⚠) |
| Native HA CRR | Cross-region asynchronous log replication; recovery group is itself 3 replicated instances | Introduced in IBM MQ 9.4.2, 27 Feb 2025 | ✅ |
| Native HA IRR | Two-node DR topology, manual switchover to the recovery site | New in IBM MQ 10.0.0 | ✅ |
| Native HA CRR node count | Six nodes — three in each region | IBM MQ 10.0.0 | ✅ |
| MQ Appliance HA | Pair of appliances, automatic failover, close proximity, designed for no data loss | MQ Appliance firmware | ✅ |
| MQ Appliance DR between HA groups | HA group of appliances at both ends of the DR link | New in MQ Appliance 9.3.2 | ✅ |
| Current MQ releases (Sept 2026) | 10.0 LTS (GA 16 June 2026; z/OS 19 June 2026) and 9.4 LTS | — | ✅ |
| Object-definition export | `dmpmqcfg` (AIX/Linux/Windows); `DMPMQMCFG` (IBM i) | Current | ✅ |

---
## 2. The Recovery Substrate: Logs, Persistence, and What Survives

Everything that MQ's HA and DR products do is built on top of one mechanism: the queue manager writes changes to a recovery log before it applies them to the object data files, and any copy of the queue manager that can read that log can reconstruct the state. Understanding the substrate is therefore not academic. The recovery ceiling of your estate is set here, and no amount of DR product above it can raise that ceiling.

### 2.1 Persistent Versus Non-Persistent: What Each Survives

IBM's own wording is unusually direct, and it is worth quoting rather than paraphrasing. From the IBM MQ documentation on message persistence:

> "Persistent messages are written to logs and queue data files. If a queue manager is restarted after a failure, it recovers these persistent messages as necessary from the logged data. Messages that are not persistent are discarded if a queue manager stops, whether the stoppage is as a result of an operator command or because of the failure of some part of your system." — IBM MQ documentation, *Message persistence* ✅

The MQ *System Administration Guide* is equally direct:

> "Nonpersistent messages are discarded; they do not survive when WebSphere MQ stops." ✅

**State it plainly, because the whole DR design depends on it: non-persistent messages are lost whenever the queue manager stops — planned or unplanned, graceful or abrupt, failure or failover.** A failover *is* a queue manager stopping and starting somewhere else, so:

| Message class | Survives a graceful `endmqm -w` and restart | Survives a node failure / crash restart | Survives a failover (MIQM, RDQM HA, Native HA) | Survives a site loss and DR promotion |
|---|---|---|---|---|
| Persistent | Yes | Yes | Yes | Yes, subject to the replication mode |
| Non-persistent | No | No | **No** | **No** |

Three practical corollaries:

- **A queue with `DEFPSIST(NO)` is a queue with an RPO of "everything in it."** No HA or DR product changes this. If the business needs the message across a restart, the message must be persistent, and any queue attribute or `MQPER` setting that quietly defaults to non-persistent is a DR defect, not a performance tuning decision.
- **Non-persistent messages cannot be recovered from a media image.** IBM states this explicitly: "Nonpersistent messages cannot be recovered in this way." ✅ So linear logging does not rescue them either.
- **Persistent cost is a performance trade, and it is real.** Persistence means log writes and an `fsync`-equivalent discipline on the critical path. That is the price of the guarantee. For a discussion of the throughput consequences and of at-most-once versus at-least-once delivery generally, see [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) §2 and §4 — this guide does not re-derive it.

There is a subtlety worth naming without over-claiming: MQ has historically allowed a non-persistent message class with stronger-than-default behaviour on some configurations. Treat any such setting as an optimisation to be verified against your own release's documentation before it is relied on for a DR claim; the baseline, documented, safe statement is the one above — non-persistent messages are discarded when the queue manager stops.

### 2.2 Circular Logging Versus Linear Logging

This is the most important configuration decision in the entire guide, because it decides **whether a damaged or deleted queue file can be rebuilt at all.** IBM MQ supports two log types, and the documentation is emphatic about the consequences.

**Circular logging** — IBM's description:

- "Use circular logging if all we want is restart recovery, using the log to roll back transactions that were in progress when the system stopped." ✅
- The log is a ring of fixed log files. Logging fills the first file, then the next, and so on until all files are full, then returns to the first file and overwrites it.
- The advantage is that "you never run out of log files." ✅
- The advantages IBM lists are **ease of administration** ("once you have configured circular logging correctly for the workload, no further administration is needed") and **performance** ("circular logging performs better than linear logging, because circular logging is able to reuse log extents that have already been formatted"). ✅
- **Circular logging is the IBM MQ default.** ✅

**Linear logging** — IBM's description:

- "Use linear logging if we want both restart recovery and media recovery (re-creating lost or damaged data by replaying the contents of the log)." ✅
- The log is "a continuous sequence of log files." ✅
- Log files may be reused — but *only* when they are no longer needed for either restart recovery or media recovery — or manually archived for longer-term storage.
- "Linear logging (but not circular) enables damaged objects to be recovered. So, linear logging provides protection against queue files being corrupted or deleted, as these damaged queues can be recovered from a linear log." ✅
- The cost is administrative work (media images must be recorded; log extents must be archived or deleted) and performance (linear logging "has to allocate new log extents and format them"). ✅

**What each mode does and does not enable — the decision table:**

| Capability | Circular logging | Linear logging |
|---|---|---|
| Restart recovery (roll back in-flight transactions) | ✅ Yes | ✅ Yes |
| Recovery from power loss | ✅ Yes | ✅ Yes |
| Recovery from communications failure | ✅ Yes | ✅ Yes |
| Media recovery (rebuild a lost or damaged object file) | ❌ **No** — "Media recovery is not available for Circular logging and you cannot recover the damaged objects" ✅ | ✅ Yes |
| Media images (`rcdmqimg`, automatic at shutdown) | ❌ No | ✅ Yes |
| Damaged-object rebuild with `rcrmqobj` | ❌ No — no valid action except delete, or restore a backup | ✅ Yes |
| Requires ongoing media-image discipline | No | ✅ Yes |
| Requires log-extent archival or deletion | No | ✅ Yes |
| Risk of running out of log space | Effectively none | ✅ Real — governed by media-image frequency |
| Performance | Better | Lower (extent allocation and formatting) |
| Protection against a corrupted or deleted **log** | ❌ No | ❌ No |
| Protection against messages or queues deleted by an application or administrator | ❌ No | ❌ No |

Two negatives in that table matter as much as the positives:

- **Neither log type protects against a corrupted or deleted log, or against messages and queues that an application or administrator deleted.** IBM states this explicitly: "Neither circular nor linear logging protect against a corrupted or deleted log, or messages or queues that have been deleted by applications or the administrator." ✅ That is precisely why backup (§7) remains necessary even on a fully linear-logged estate.
- **The default is the weaker option.** Because circular logging is the IBM MQ default and is better-performing and easier to administer, an estate can drift into production with a recovery ceiling of "restart only" without anyone deciding it. IBM's own support guidance is direct: "Linear logging is recommended for production systems." ✅

### 2.3 The Management Regime Around Linear Logging

Choosing linear logging is not a one-off setting; it is an operational commitment, and IBM has progressively automated parts of it.

- **Automatic versus manual log management.** From IBM MQ 9.1.0 you can set the `LogManagement` attribute in the `Log` stanza of `qm.ini` to `Automatic` or `Manual`. ✅
- **Automatic media images.** The queue manager can be configured to take periodic media images automatically, based on time or log usage, or you can schedule them manually. ✅
- **The crucial dependency.** "The frequency of media images determines when linear log files can be reused, and is a major factor in how much disk space must be available for linear log files." ✅ Take media images rarely and your log directory grows without bound; take them frequently and you pay a performance cost and lose the ability to recover to an older point.
- **With automatic log management and automatic media images, linear logging behaves "in a similar way to a very large circular log, but with the improved redundancy against media failure enabled by media recovery."** ✅ This is the configuration most estates should aim for.
- **Only log files required for queue manager restart (the *active log*) need to be online.** ✅ Inactive log files can be copied to archive media and removed from the log directory — with the caveat that if they are still needed for media recovery they must be retrievable, or media recovery will fail.
- **Changing log type after creation is possible** using the `migmqlog` command, from IBM MQ 9.1.0. ✅ Before that, the log type was fixed at queue manager creation. This matters for DR planning: an estate on an old release cannot fix a circular-logging decision in place.
- **Secondary log files are the pressure valve, and they are finite.** If long-running transactions prevent active log files from being freed, secondary log files are formatted and added to the active pool. If the secondary files are exhausted, "requests for most further operations requiring log activity will be refused with an `MQRC_RESOURCE_PROBLEM` return code being returned to the application, and any long running transactions will be considered for asynchronous rollback." ✅ An unbounded transaction — a batch job that opens a syncpoint and runs for an hour — is therefore a genuine availability and data-integrity risk, not merely a design smell.

### 2.4 The Restart and Recovery Path

When a queue manager starts after any failure — crash, power loss, node failure, or failover — it performs recovery from the log. What that recovery does:

- **Roll back in-flight transactions.** Units of work that had not committed when the queue manager stopped are backed out. This is why the log must contain enough information to reverse as well as apply.
- **Roll forward committed work.** Committed persistent changes recorded in the log but not yet reflected in the queue data files are applied.
- **Rebuild the queues in their committed state.** From the *System Administration Guide*: "If you lose power, when the queue manager is restarted, WebSphere MQ restores the queues to their committed state at the time of the failure. This ensures that no persistent messages are lost." ✅
- **Recover affected queues automatically.** "If any transaction was incomplete when the queue manager last shut down, any queue affected is also recovered automatically in order to complete the startup operation." ✅
- **Recover automatically from a damaged object at startup, if it can.** "WebSphere MQ automatically recovers some objects from their media image if it finds that they are corrupt or damaged. In particular, this applies to objects found to be damaged during the normal queue manager startup." ✅

The critical operational fact buried in the failure paths:

- **If the queue manager object itself is damaged, the queue manager cannot start unless that object can be recovered.** With linear logging MQ attempts to re-create it from its media image automatically; **if the log method does not support media recovery, "you can either restore a backup of the queue manager or delete the queue manager."** ✅ That sentence is the entire argument for linear logging in production, and it is the sentence to put in front of anyone who argues for circular logging on performance grounds.
- **On a queue manager without media recovery, a damaged local queue holding uncommitted messages is marked as a damaged object and its uncommitted messages are ignored**, reported by operator message AMQ7472. ✅ This is a silent-looking data-integrity outcome, so it belongs in the runbook: AMQ7472 is not a warning to acknowledge, it is a decision point.
- **If an object is damaged at any time after startup, the queue manager performs a preemptive shutdown.** ✅ A damaged object is therefore not a degraded-mode condition; it is an outage.
- **If a log file required for media recovery is missing, operator message AMQ6767 is issued and the media recovery operation fails.** ✅ "If you do not take regular media images of the objects that you want to re-create, you might have insufficient disk space to hold all the log files required to re-create an object." ✅

### 2.5 Point-in-Time Recovery, Honestly Framed

**IBM MQ does not offer point-in-time recovery as a product feature in the database sense.** There is no "restore the queue manager to 14:32:07" button, no log archive with a replay-to-timestamp command, and no consistent-export utility spanning multiple queue managers. What exists is weaker and needs to be understood as such:

- **Restart recovery gives you "the committed state at the moment the queue manager stopped."** ✅ That is a point in time, but it is the last one, not a chosen one.
- **Media recovery gives you "the latest image of an object, plus all applicable log records up to the moment the rebuild command is issued."** ✅ It reconstructs *current* state, not historical state.
- **A data-directory backup gives you a point in time — the moment the queue manager was stopped for the backup** (§7.1). That is the only honest point-in-time recovery MQ offers, and it costs an outage to manufacture.
- **If you need to recover a message that was *correctly* processed and then deleted by a consuming application, MQ cannot do it.** The message is gone from the log's perspective once the unit of work committed and the queue data was updated. Recovery of that class of loss is an application-level or downstream-reconciliation problem — see [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) for the reconciliation and idempotency discipline, and [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) for how a reconciliation control is expected to be evidenced.

**The practical consequence:** if your DR design assumes "we can always replay from a log archive to a chosen point," that assumption is wrong for IBM MQ on distributed platforms. Design for *state reconstruction to now*, not for *travel to a timestamp*. Where the business genuinely needs historical replay, that requirement belongs to a retained event store or an architected journal, not to the queue manager's recovery log.

---

## 3. The Media-Image Recovery Path: Rebuilding a Damaged Object

This section is the operational heart of MQ's *storage* recovery story — as distinct from its site recovery story. It answers the question: the queue file is corrupt or was deleted; what now?

### 3.1 What a Media Image Is

IBM's definition, from the *System Administration Guide*:

> "Media recovery re-creates objects from information recorded in a linear log. For example, if an object file is inadvertently deleted, or becomes unusable for some other reason, media recovery can re-create it. The information in the log required for media recovery of an object is called a *media image*." ✅

> "A media image is a sequence of log records containing an image of an object from which the object itself can be re-created." ✅

Three properties of media images drive the whole procedure:

- **The media recovery record** — the first log record required to re-create an object — is the start of the object's latest media image, and "the media recovery record of each object is one of the pieces of information recorded during a checkpoint." ✅ Media images are therefore anchored to checkpoints, which is why checkpointing behaviour and recovery requirement are coupled.
- **A media image alone is not enough.** "When an object is re-created from its media image, it is also necessary to replay any log records describing updates performed on the object since the last image was taken." ✅ The image is a floor; the log above it is the rest of the recovery.
- **Objects get a first media image for free.** "When an object is created, the log records written contain enough information to completely re-create the object. These records make up the object's first media image." ✅

### 3.2 Where Media Images Come From

Media images are produced by three routes:

1. **Automatically at shutdown.** "Subsequently, at each shutdown, the queue manager records media images automatically as follows: images of all process objects and queues that are not local; images of empty local queues." ✅ Note the specificity — this is *not* a full set of images for every queue with messages in it.
2. **Automatically on a schedule**, if configured — the queue manager can take periodic media images based on time or log usage. ✅
3. **Manually, with `rcdmqimg`** — "This command writes a media image of the WebSphere MQ object. Once this has been done, only the logs that hold the media image, and all the logs created after this time, are needed to re-create damaged objects." ✅ `rcdmqimg` is how you deliberately create a recovery point and, in doing so, release older log extents for archive or deletion.

**Operational discipline:** `rcdmqimg` is the lever that trades log disk space against recovery granularity, and it is the thing that silently stops happening when a team is busy. If media images are not being recorded, two bad things compound: the log directory grows until it fills, and the age of the recoverable image drifts ever further behind now — so a media recovery rebuild loses more and takes longer than it would have a month earlier.

### 3.3 The Rebuild Mechanism: `rcrmqobj`

`rcrmqobj` (re-create object) reconstructs an object or group of objects from the images contained in the log.

- **It works only with linear logging.** "This command can only be used when using linear logging." ✅
- **It replays the log forward.** "The object is re-created from its latest image found in the log, together with all applicable log events between the time the image was saved and the time the re-create command was issued." ✅
- **It runs against a running queue manager.** IBM's documentation notes the command is used on a running queue manager. ✅
- **Non-persistent messages are not recoverable.** "If a WebSphere MQ object becomes damaged, the only valid actions that can be performed are either to delete it or to re-create it by this method. Nonpersistent messages cannot be recovered in this way." ✅
- **It is the only alternative to deletion.** "When an object has been damaged you can delete it or, if the queue manager is using a linear log, attempt to recover it from its media image using the `rcrmqobj` command." ✅

### 3.4 The Procedure in Outline

The exact commands are release-specific and this outline is deliberately a sequence of decisions rather than a copy-paste script — read your own release's documentation before executing any of it.

1. **Detect and classify.** Establish *what* is damaged: the queue manager object itself, a local queue, a process object. Operator messages AMQ7472 (damage reported) and AMQ6767 (a required log file is missing, so media recovery has failed) are the signals. Determine whether the damage was found at startup or during running — the queue manager's own automatic behaviour differs between the two.
2. **If found at startup on a linear-log queue manager:** MQ may already be attempting automatic recovery from media images. Let it. If it succeeds, the queue manager starts and the work is a verification task (§3.6), not a recovery task.
3. **If the queue manager object is damaged and cannot be recovered:** you are down. The remaining options are a backup restore (§7.1) or rebuilding the queue manager from configuration (§7.2). This is the branch where a DR plan meets reality.
4. **If a non-queue-manager object is damaged while running:** the queue manager performs a preemptive shutdown. ✅ Restart it and let the startup recovery path run.
5. **Verify that the required log files are present.** The log file containing the media recovery record, and every subsequent log file, must be available in the log directory. If any is missing, AMQ6767 is issued and media recovery fails. ✅ This is the step where an over-eager log-cleanup script — or an archive that was never restocked — turns a recoverable incident into a rebuild.
6. **Record a fresh media image if you can, then rebuild.** Issue `rcrmqobj` for the damaged object. Replay proceeds from the latest image plus all applicable log events since.
7. **Accept the recovery ceiling.** The rebuilt object reflects committed persistent state as of *now*. Non-persistent messages are gone. Anything committed and then deleted by a consumer is gone. Both facts are permanent, not fixable by retrying the command.

### 3.5 Prerequisites You Discover Too Late

The failure mode of media recovery is almost never the command; it is a prerequisite that was never established. These are the ones that bite:

- **The estate is on circular logging.** Then no media image exists and there is nothing to rebuild from. The only remaining action for a damaged object is deletion, explicitly. ✅
- **Log extents were archived but not catalogued.** The required extent is on tape in a vault with a four-hour retrieval SLA. Recovery is now a logistics exercise.
- **Log extents were deleted as "superfluous."** Whether a log extent is truly superfluous depends on whether it is still needed for **media** recovery, not just restart recovery. IBM's guidance is explicit that you must keep all log files back to the oldest one required for media recovery (operator message AMQ7468), and that the ones needed only for restart are the ones reported by AMQ7467. ✅ Confusing the two — treating AMQ7467's answer as licence to delete — destroys media recovery.
- **The media recovery LSN has stopped advancing.** IBM documents a defect in this area in older levels (APAR IT30479, "High-CPU issue in amqzmuc0 and media recovery LSN not moving forwards"), fixed in 9.1 CD 9.1.3 and 9.1 LTS 9.1.0.4. ✅ A media recovery LSN that is not moving forward is a silent failure of the whole mechanism, so it belongs on the monitoring list, not in an APAR archive.
- **The disk cannot hold the log.** "If you do not take regular media images of the objects that you want to re-create, you might have insufficient disk space to hold all the log files required to re-create an object." ✅
- **Nobody has ever rehearsed it.** Media recovery is a low-frequency, high-stakes operation performed by an on-call engineer at 03:00. Rehearsing it on a restored copy is the only way it becomes a procedure rather than a research project.

### 3.6 Restore-Testing Discipline for Media Recovery

The test that matters is not "does `rcrmqobj` run." It is "can we demonstrate, on a dated exercise, that a damaged object is rebuilt to a state that reconciles with the source system." A credible media-recovery test:

- **Deliberately damages a populated local queue on a restoration copy** — not on production, and not an empty queue, because empty queues get automatic shutdown images and would pass trivially.
- **Carries a known quantity of persistent messages** so the rebuild can be measured against an expected count, not merely observed to complete.
- **Deletes a log extent to confirm the AMQ6767 failure path** and to prove the team can find and restore an archived extent inside the target RTO.
- **Measures elapsed time end-to-end** and records it against the RTO the business has accepted.
- **Confirms the non-persistent loss empirically** — put both persistent and non-persistent messages on the same queue and show what comes back. A team that has *seen* the non-persistent messages vanish designs differently afterwards.
- **Records evidence**: commands run, operator messages observed, before/after queue depth, elapsed time, operator name, date. This is the artifact an operational-resilience function will ask for, and the section of the runbook that is worth writing down properly.

### 3.7 The Documented Common Failure Modes

Summarised from IBM's own documented behaviour, in the order they typically occur in real incidents:

| Failure mode | Documented signal | Consequence |
|---|---|---|
| Damaged object on a non-media-recovery queue manager at startup | AMQ7472; uncommitted messages ignored | Data loss with no recovery path; object must be deleted or the queue manager restored from backup ✅ |
| Damaged queue manager object on a circular-log queue manager | Queue manager will not start | Only options: restore a backup, or delete the queue manager ✅ |
| Damaged object detected while running | AMQ7472, then preemptive shutdown | Full outage, not degraded service ✅ |
| Required log file missing during media recovery | AMQ6767, recovery fails | Object unrecoverable unless the extent is recovered ✅ |
| Log extents needed for media recovery deleted | Detected only at recovery time | Object unrecoverable ✅ |
| Media recovery LSN frozen | High CPU in `amqzmuc0` (APAR IT30479, older levels) | Media recovery silently unavailable ✅ |
| Secondary log files exhausted by long transactions | `MQRC_RESOURCE_PROBLEM`; long-running transactions considered for asynchronous rollback ✅ | Application failures and unexpected rollbacks |

**The pattern across all seven:** the failure is discovered at the worst possible moment, in an incident, rather than during a planned exercise. That is the argument for §9.

---
## 4. The Local Resilience Family: MIQM, RDQM, Native HA, and the Appliance

IBM MQ has accumulated four distinct local high-availability mechanisms over roughly two decades, each with a different storage assumption, a different node count, and a different set of things it does not protect against. Choosing between them is really a question about what you are willing to have as a single point of failure.

### 4.1 Multi-Instance Queue Managers (Shared Storage, Active/Standby)

**Introduced in WebSphere MQ 7.0.1.** ✅ IBM's documentation: "We can create a multi-instance queue manager on any non-z/OS platform supported by IBM WebSphere MQ Version 7.0.1 and later." ✅

**The design.** "Multi-instance queue managers are instances of the same queue manager configured on different servers. One instance of the queue manager is defined as the active instance and another instance is defined as the standby instance. If the active instance fails, the multi-instance queue manager restarts automatically on the standby server." ✅

- You create the queue manager **once**, with `crtmqm`, placing its queue manager data and logs in **shared network storage**. ✅
- On the other server you do **not** create the queue manager again — you use the **`addmqinf`** command to create a reference to the same data and logs on the network storage. ✅
- "Each of the servers references the same queue manager data and logs; there is only one queue manager, and it is active on only one server at a time." ✅
- **Instance limit: two.** "There is a limit of a total of two instances. We cannot have two standby instances and one active instance." ✅ IBM's HA configuration overview does allow "two or more computers" with "other instances" as standbys in its general description, so treat the two-instance limit as the documented operational constraint for the supported configuration.

**The failover mechanism.** It is a lock, not a heartbeat-vote: "The only control you have over which instance becomes active first is the order in which you start the queue manager on the two servers. **The first instance to acquire read/write locks to the queue manager data becomes the active instance.**" ✅ The active instance "has exclusive access to the shared queue manager data and logs folders when it is running. The standby instance detects when the active instance has failed, and becomes the active instance. It takes over the QM1 data and logs in the state they were left by the active instance, and accepts reconnections from clients and channels." ✅

**Documented failover triggers** — useful, because two of the three are storage- and process-related rather than host-related:

- Failure of the server hosting the active queue manager instance.
- **Failure of connectivity between the server hosting the active instance and the file system.** ✅
- **Unresponsiveness of queue manager processes, detected by IBM MQ, which then shuts down the queue manager.** ✅

**Switchover** is supported deliberately: "We can swap the active instance to the other server, once it has started, by stopping the active instance using the switchover option to transfer control to the standby." ✅ This is how you do planned maintenance without an outage — and, importantly, how you *test* the failover path without inducing a failure.

**The documented caveats — and the one that really bites:**

- **IBM states the storage requirement plainly:** "A multi-instance queue manager is one part of a high availability solution. You need some additional components," the first of which is "a high performance shared network file system (NFS) that manages locks correctly and provides protection against media and file server failure." ✅
- **The storage-maintenance caveat is the one to put on the wall:** "**Important: We must stop all multi-instance queue manager instances that are running in the environment before we can perform maintenance on the NFS drive. Make sure that we have queue manager configuration backups to recover, in the event of an NFS failure.**" ✅ Read that twice. The shared file system that makes failover possible is itself a single point of failure, its maintenance requires a full outage of *every* instance in the environment, and IBM's stated mitigation is *configuration backups* — not a second live copy.
- IBM's own limitations list for multi-instance: "Highly available, high performance networked storage required"; "More complex network configuration because queue manager changes IP address when it fails over." ✅
- **Do not combine it with a cluster manager:** "HA clusters and multi-instance queue managers are alternative ways of making queue managers highly available. **Do not combine them by putting a multi-instance queue manager in an HA cluster.**" ✅
- IBM lists the advantages as: "Basic failover support integrated into IBM MQ," "Faster failover than HA cluster," "Simple configuration and operation," "Integration with IBM MQ Explorer." ✅
- **Automatic client reconnection is not supported by IBM MQ classes for Java** — a documented limitation that matters disproportionately in Java-heavy banking estates. ✅

**What multi-instance does not protect against.** The host-level single point of failure is removed; the storage-level one is not. A storage array loss, an NFS server loss, a fabric partition, or a lost site takes out the queue manager and both instances simultaneously. IBM is explicit that a multi-instance HA solution "must include a mechanism to recover data after a storage failure." ✅

### 4.2 RDQM HA (Replicated Data Queue Managers)

**The design.** "An RDQM configuration consists of three servers configured in a high availability (HA) group, each with an instance of the queue manager. One instance is the running queue manager, which **synchronously replicates** its data to the other two instances. If the server running this queue manager fails, another instance of the queue manager starts and has current data to operate with. The three instances of the queue manager share a **floating IP address**, so clients only need to be configured with a single IP address. Only one instance of the queue manager can run at any one time, even if the HA group becomes partitioned due to network problems." ✅

**Roles and quorum.** The running node is the **primary**; the other two are **secondaries**. ✅ The three-node design is not for capacity — it is for quorum:

- "Three nodes are used to greatly reduce the possibility of a split-brain situation arising." ✅
- "RDQM uses a three node system with quorum to avoid the split-brain situation. Nodes that can communicate with at least one of the other nodes form a quorum. **Queue managers can only run on a node that has quorum.**" ✅
- Single node failure: the queue manager can run on one of the other two nodes. ✅
- **Two nodes lost: the queue manager cannot run on the remaining node, because that node cannot tell whether the other two have failed or whether it has lost connectivity.** ✅ This is the trade-off worth stating out loud to a business sponsor: RDQM HA deliberately *prefers being down over being wrong*. It will not run a queue manager on a partitioned minority.
- All nodes lose connectivity: the queue manager cannot run anywhere. ✅

**The underlying technology — verified.** "The group configuration of the three nodes is handled by **Pacemaker**. The replication between the three nodes is handled by **DRBD**." ✅ IBM's RDQM architecture guidance elaborates: for each RDQM queue manager a DRBD resource is created (`/etc/drbd.d/<qm>.res`), the queue manager is configured to use that DRBD resource for storage, and a set of Pacemaker resources is created to monitor and manage the queue manager. ✅ DRBD and Pacemaker RPM packages are shipped on the IBM MQ media and IBM's instruction is explicit: "We should install the versions supplied with IBM MQ. **Do not download your own versions.**" ✅

**Platform support — verified, and it has narrowed over time.**

- IBM MQ 9.2 documentation: "RDQM is available on RHEL 7.6 and onwards on x86-64, and on RHEL 8.2 and onwards on x86-64." ✅
- IBM MQ 9.4 documentation: "RDQM is available on x86-64 for RHEL 8 (8.8 or later) and RHEL 9 (9.2 or later)." ✅
- **Product entitlement: RDQM is IBM MQ Advanced** — the documentation marks the RDQM topics with the Advanced entitlement icon. ✅
- Because RDQM depends on a DRBD *kernel module*, OS kernel updates and RDQM levels are coupled; IBM maintains a kernel-module compatibility matrix and states that removing and applying maintenance differs between HA, DR, and combined DR/HA configurations. ✅
- Migrating RDQM nodes must be done in a strict sequence: "you must upgrade all nodes in a sequence. **Do not try to operate with the nodes at different levels.**" ✅
- Upgrading RHEL major versions requires a new Pacemaker cluster and migration of the RDQMs to it. ✅

**Known operational limitations:**

- **The IBM MQ Console does not support RDQMs**, and MQ Explorer shows them but without RDQM-specific information. ✅ Tooling investment is required.
- Backing up an RDQM follows the standard queue manager data backup process, and "stopping the queue manager and backing it up has no effect on the node monitoring done by the RDQM configuration." ✅
- Changing the IP addresses of interfaces in an HA configuration breaks HA operation and the queue manager will not run on the node where the addresses were changed. ✅

**What RDQM HA does not protect against.** A three-node RDQM HA group is a three-node group *at one site*. Site loss takes all three. RDQM HA is a genuinely strong local mechanism — synchronous, quorum-protected, with a floating IP that makes client configuration simple — and it is still not DR.

### 4.3 Native HA

**Native HA is the newest of the four and the one whose lineage is most worth getting right**, because its capabilities have expanded substantially across releases.

**The design.** A Native HA configuration consists of **three nodes, each with an instance of the queue manager**. ✅ "One instance is the active queue manager, processing messages and writing to its log. Whenever the log is written, the active queue manager replicates the data to the other two instances, known as replicas. Each replica writes to its own recovery log, acknowledges the data, and then updates its own queue data from the replicated recovery log. If the node running the active queue manager fails, one of the replica instances of the queue manager takes over the active role and has current data to operate with." ✅

**The log type.** Native HA uses a **replicated log**, which is described as "essentially a linear log, with automatic log management and automatic media images enabled." ✅ This is a genuinely important architectural fact: Native HA is built on the *stronger* log type, so media recovery is available to a Native HA queue manager by construction. The same techniques used to manage a linear log apply to the replicated log. (Source: IBM community article by an IBM engineer citing the IBM MQ *Types of logging* topic — see §12 for the source-quality note.)

**Quorum and consensus.**

- **Quorum floor: at least two of the three instances must be available** for the group to operate. ✅
- The mechanism is a consensus algorithm: "Native HA implements a consensus algorithm based closely on raft (elections are based on log content, strong leader model, etc). Native HA uses its own proprietary wire protocol to implement the election process and also replicate between the active and replica pods." ✅ (Same source-quality caveat; see §12.)
- Note the practical implication shared with RDQM: a majority-based system will stop rather than split. Losing two of three nodes means no active queue manager, by design.

**Storage.** Native HA "always uses block storage" and "MQ queue manager does data and log the replication between the AZs. So you can rely on the replication built-in to IBM MQ." ✅ The guidance is explicit and counter-intuitive enough to quote: "**When using Native HA (including CRR) you should use simple RWO block storage that is not replicated — Native HA is responsible for the replication, not the storage layer.**" ✅ This is the opposite of the multi-instance assumption, and it removes the shared-storage single point of failure that defines multi-instance.

**Availability by platform and version.**

- IBM MQ 9.3 documentation: "Native HA is a high availability solution that is available on **container deployments** of IBM MQ." ✅
- IBM MQ 9.4 documentation: "Native HA is a high availability solution that is available on **container deployments of IBM MQ and on Linux**." ✅
- Container deployment is via the IBM MQ Operator (originally for Red Hat OpenShift Container Platform, later also Kubernetes on Amazon EKS) or the sample Helm charts. The operator "watches for new queue manager definitions, and then turns them into necessary low-level resources, such as StatefulSet and Service resources. In the case of Native HA, the operator can also perform the complex rolling update of queue manager instances." ✅
- In a Kubernetes deployment, the three instances are typically three pods spread across three availability zones by the StatefulSet's scheduling, each with its own persistent volume claim. ✅
- **The exact release in which Native HA was first introduced could not be verified this pass** — it is documented for container deployments from 9.2 onward, and the 9.4 documentation extends it to Linux. Treat the introduction release as ⚠ flagged; see §13.

**What Native HA does not cover.** Base Native HA is a **single-region, single-cluster** high-availability mechanism. It does not, on its own, provide any cross-site or cross-region disaster recovery — that is what CRR and IRR add, and they are separate, version-gated features (§5.3). Each replica holds its own storage; the replication is MQ's, over the network; a region-level loss with no CRR deployment is an unrecoverable loss of the queue manager.

### 4.4 The MQ Appliance HA Arrangement

The IBM MQ Appliance is a hardware form factor with HA and DR built in. Its model is a **pair of appliances**:

- "The IBM MQ Appliance includes built-in support for both high availability (HA) and disaster recovery (DR). In HA and DR configurations, multiple appliances are used to provide a resilient solution for running queue managers, which supports rolling maintenance and which can protect against software and hardware failures, or data centre outages. Queue manager data is replicated between the appliances so that queue managers can fail over from one appliance to another. **Fail over is automatic for HA and administratively orchestrated for DR.**" ✅
- The HA pair is designed to be close together: "Both appliances in a high availability pair are typically located in the same data center." ✅ Appliance HA "tend[s] to be in close proximity, and [is] designed for automatic takeover with no data loss." ✅
- The appliance documents **separate network interfaces** for HA primary, HA alternate, HA replication, and DR replication — reflecting that HA traffic and DR traffic have different latency and bandwidth assumptions. ✅
- Operational commands are appliance-specific and include `dsphagrp` (display HA group), `status qmgr`, `sethagrp -r` (resume a suspended appliance), `makehaprimary`, and — for DR — `crtdrprimary` / `crtdrsecondary`. ✅
- **Documented HA failure mode:** "If the two appliances lose the replication interface, the HA status is reported as *Remote appliance(s) unavailable*. The running queue manager might accumulate out-of-sync data. The other queue manager remains in standby with no out-of-sync data. When the connection is remade, replication is resumed." ✅ A split-brain on the appliance is resolved by explicitly choosing a winner (`makehaprimary`) after taking backups of both data sets. ✅
- **Hardware generations matter for lifecycle planning:** the M2004 model was announced with availability from 16 July 2026 and a minimum firmware level of MQ 10.0 LTS; M2003 is supported to 30 September 2031; M2002 to 30 September 2027. ✅

### 4.5 The Comparison Table

| | Multi-instance (MIQM) | RDQM HA | Native HA | MQ Appliance HA |
|---|---|---|---|---|
| Storage dependency | **Shared network file system** (e.g. NFS) that manages locks correctly ✅ | **Local block storage per node**, replicated by DRBD ✅ | **Simple RWO block storage, not replicated** — MQ does the replication ✅ | Appliance-internal storage, replicated between the pair ✅ |
| Storage as a single point of failure | **Yes** — array/file-server loss takes out the QM; IBM requires a data-recovery mechanism alongside ✅ | No — three independent copies ✅ | No — three independent copies ✅ | No — two copies ✅ |
| Node count | 2 (active + standby; max two instances) ✅ | 3 (primary + 2 secondaries), quorum ≥ 2 nodes connected ✅ | 3 (active + 2 replicas), quorum ≥ 2 instances ✅ | 2 appliances ✅ |
| Replication mode | Not applicable — shared data, exclusive locks ✅ | **Synchronous** ✅ | Log replication, acknowledged by replicas ✅ | Replication between the pair ✅ |
| Failover trigger mechanism | Loss of read/write **locks** on the shared file system; standby acquires them ✅ | **Pacemaker** moves the queue manager; **quorum** required ✅ | Consensus election among the replicas ✅ | Appliance HA group, automatic takeover ✅ |
| Failover time | Fast (IBM: "faster failover than HA cluster") ✅ | Automatic, seconds-to-minutes; bounded by log replay | Automatic; promoted replica has current data ✅ | Automatic (HA); designed for no data loss ✅ |
| Operational complexity | Low — simple config, one `crtmqm` + `addmqinf` ✅ | **High** — DRBD kernel modules, Pacemaker, RHEL-only, strict upgrade sequences, no MQ Console support ✅ | Moderate — operator/Helm-driven in containers; block storage per instance | Low — but appliance-specific skills and commands ✅ |
| Version floor | WebSphere MQ **7.0.1** ✅ | RHEL 7.6+/8.2+ on x86-64 in the 9.2 docs; RHEL 8.8+/9.2+ on x86-64 in the 9.4 docs ✅ | Documented for containers from **9.2**; Linux added by the **9.4** documentation (intro release ⚠) ✅/⚠ | MQ Appliance firmware; DR between HA groups from **9.3.2** ✅ |
| Entitlement | Base product | **IBM MQ Advanced** ✅ | Delivered via the operator/container images (Advanced container) | Appliance product |
| Log type it implies | Yours to choose — and a circular-log MIQM has no media recovery ✅ | Yours to choose | **Replicated log = linear log + automatic log management + automatic media images** ✅ | Yours to choose |
| Split-brain protection | Relies on the file system's locking ✅ | **Explicit quorum**; QM cannot run without it ✅ | **Explicit quorum**; ≥ 2 instances required ✅ | Detected and reported; operator chooses a winner ✅ |
| Protects against site loss | ❌ No | ❌ No | ❌ No | ❌ No (needs appliance DR — §5.4) |

**The conclusion of the table, stated plainly: every one of these four mechanisms is a local, single-site mechanism.** Three of them remove the storage single point of failure and one of them does not. None of them answers the question "what happens if we lose the building," and that is the entire subject of the next section.

---

## 5. The Site-Loss Architectures: RDQM DR, DR/HA RDQM, and Native HA CRR

An IBM MQ estate has, as of September 2026, four IBM-native routes to surviving the loss of a site, plus one generic infrastructure route. They differ enormously in platform availability, product entitlement, replication mode, and how much of the failover is automated — and only one of them is a cross-region capability on the modern product line.

### 5.1 RDQM DR (Primary/Secondary Pair)

**The design.** "A queue manager runs on a primary node at one site, with a secondary instance of that queue manager located on a **recovery node at a different site**. Data is replicated between the primary instance and the secondary instance, and if the primary node is lost for some reason, the secondary instance can be made into the primary instance and started." ✅

**The critical constraints, all documented:**

- **"We cannot start a queue manager while it is in the secondary role."** ✅ The secondary is a warm standby, not a read replica. It takes a role change to bring it up.
- **Promotion is manual.** "RDQM (replicated data queue manager) is available on a subset of Linux platforms and can provide a disaster recovery solution… we can **manually** make the secondary instance into the primary instance and start the queue manager, then resume work from the same place." ✅
- **"We cannot add disaster recovery to an existing queue manager."** ✅ You can migrate an existing queue manager to become a DR RDQM, but you cannot bolt DR onto it in place. This is a design-time decision, not a configuration change.
- **"A queue manager cannot be configured with both RDQM disaster recovery and RDQM high availability."** ✅ RDQM DR and RDQM HA are mutually exclusive on the *same* queue manager. (The combined capability is DR/HA RDQM — a different feature, §5.2.)
- **Platform and entitlement:** Linux only, and IBM MQ Advanced. ✅
- **Both nodes must be Linux servers**; replication is handled by DRBD. ✅

**The replication-mode decision — the single most important choice in RDQM DR:**

- "We can choose between **synchronous** and **asynchronous** replication of data between primary and secondary queue managers." ✅
- **Asynchronous:** "operations such as IBM MQ PUT or GET complete and return to the application **before** the event is replicated to the secondary queue manager. Asynchronous replication means that, following a recovery situation, **some messaging data might be lost**. But the secondary queue manager will be in a consistent state, and able to start running immediately, even if it is started at a slightly earlier part of the message stream." ✅ That last clause is the important one: asynchronous RDQM DR guarantees a *consistent* recovery point, just an older one. It does not corrupt.
- **Synchronous:** the PUT or GET does not return until the data is replicated. This is an effectively-zero-loss mode for the replication link, at the cost of write latency tied to inter-site round-trip time — which is why synchronous RDQM DR is only sensible at metro distances.

**Replication, synchronization, and snapshots.** This three-part distinction is where RDQM DR gets subtle:

- **Replication** — normal operation: "any updates to the persistent data for a disaster recovery queue manager are transferred from the primary instance of the queue manager to the secondary instance." ✅
- **Synchronization** — after a network break: "If the network connection between the two nodes is lost, the changes to the persistent data for the primary instance of a queue manager are tracked. When the network connection is restored, a different process is used to get the secondary instance up to speed as quickly as possible. This is known as synchronization." ✅
- **Snapshots** — the safety net during synchronization: "While synchronization is in progress, the data on the secondary instance is in an inconsistent state. A snapshot of the state of the secondary queue manager data is taken. **If a failure of the main node or the network connection occurs during synchronization, the secondary instance reverts to this snapshot and the queue manager can be started. Any of the updates that happened since the original network failure are lost, however.**" ✅

That third bullet is the answer to "how much data do we lose if we fail over at the wrong moment?" — everything since the network break, not everything since the last successful replication. State that in the runbook, because it is a much bigger window than an inexperienced operator will assume.

**Split brain, and who owns the decision.** RDQM DR is explicit that this is a human problem:

- "DR RDQM configurations require user action after loss of the primary instance of a queue manager to promote and run the secondary instance on the recovery node. **It is the responsibility of whoever (or whatever) promotes the secondary instance to ensure that the former primary queue manager is stopped.**" ✅
- If the original primary keeps running, "it might process messages and, when normal operation is restored, the two instances of the queue manager have different views of the data. This is known as a partitioned or split-brain state." ✅
- Where the primary node failed completely, the operator cannot stop it — and the documentation's answer is reassuring: when the original node is repaired, its queue manager "will be initially made the secondary and be synchronized with the primary queue manager on the recovery node. The roles of the two queue managers are then reversed." ✅ The only potential loss is data the primary had not completed replicating.
- Where there is a network failure rather than a node failure, "we must ensure that you stop the original primary before you promote the secondary." ✅ And critically: "**If the replication link is working, we cannot promote a secondary queue manager if the primary instance is still running; the command fails.**" ✅ The tooling enforces safety when it can see the primary, and cannot when it cannot — which is exactly when the operator's judgement is load-bearing.
- **Managed failover avoids the problem:** "A managed failover ends the queue manager on the primary node, then starts the queue manager on the recovery node after data has been fully replicated. A partitioned state is not expected." ✅ Plus: if the queue manager is started on the recovery node while connectivity is lost, "data divergence is likely," and a partitioned state is expected to be reported once connectivity returns. ✅
- **Note the documented inconsistency worth planning around:** "When queue managers fail over to another node they retain the state they had at failover. Queue managers that were running are started, queue managers that were stopped remain stopped." ✅ A queue manager that was *deliberately stopped* will not come back on its own at the recovery site. Your failover runbook must state which queue managers are expected to be started.

### 5.2 DR/HA RDQM (HA Group at One Site, Failing Over to an HA Group at Another)

**This is the combined architecture, and it is new in IBM MQ 9.2.0** — the 9.2.0 "What's new" lists "Disaster recovery for high availability replicated data queue managers" among the features available only with IBM MQ Advanced entitlement. ✅

**The design.** "We can configure a replicated data queue manager (RDQM) that runs on a high availability group on one site, but can fail over to another high availability group at another site if some disaster occurs that makes the first group unavailable. This is known as a DR/HA RDQM." ✅

**The replication-mode fact that must not be missed:** "**The replication between the DR/HA RDQMs on the main site and the disaster recovery site is always asynchronous.** With asynchronous replication, operations such as IBM MQ PUT or GET complete and return to the application before the event is replicated to the secondary queue manager." ✅ Unlike plain RDQM DR, DR/HA RDQM gives you no choice: cross-site replication is asynchronous, so there is always an RPO exposure on site loss. You gain automatic local HA and site survivability; you pay for it with a non-zero data-loss window.

**Other documented characteristics:**

- Both HA groups are configured in the same way as an ordinary HA group; each can have a floating IP address, and "the floating IP address can be the same or different for each HA group." ✅
- **Two active sites are supported** rather than a strict main/recovery split: "if a disaster occurs and one site becomes unavailable, then all DR/HA RDQMs run on the same HA group at the same site." ✅
- **You cannot upgrade an existing RDQM to be a DR/HA RDQM** — you must create it as one. The documented workaround is to back up the data, delete the RDQM, recreate it as DR/HA, and restore. ✅
- Configuration is a substantial sequence: configure an HA group on the main site, configure an HA group on the recovery site, create a primary/primary DR/HA RDQM on one node of the main-site HA group, create primary/secondary DR/HA RDQMs on the other two nodes there, define a floating IP for each site, and mirror the arrangement at the recovery site. ✅ This is not a weekend-evening change.

### 5.3 Native HA Cross-Region Replication (CRR) and In-Region Replication (IRR)

**This is the most significant modern addition to IBM MQ's DR story, and it is the feature most likely to change an architecture.** It is also the one most likely to be mis-stated, so the version history matters.

**Version history — verified:**

- **Native HA CRR was introduced in IBM MQ 9.4.2, released 27 February 2025.** "IBM MQ 9.4.2 was released on 27th February 2025 and introduced our latest feature, **Native HA Cross-region replication**. We have extended Native HA capabilities to permit asynchronous replication of log data supporting unplanned failover and planned switchover." ✅ (IBM community article by an IBM engineer, 28 February 2025.)
- **IBM MQ 10.0.0 adds In-Region Replication (IRR) alongside CRR.** "Native HA adds a new disaster recovery topology option call[ed] In-Region Replication (IRR), alongside Cross-Region Replication (CRR)." ✅ (IBM community article by an IBM engineer, 16 June 2026.)

**The topology facts — verified from the IBM 10.0 release article:**

- "Native HA is a high availability solution requiring **3 nodes within a region**." ✅
- "**Native HA IRR is a 2 node disaster recovery solution** that requires fewer compute nodes, but **manual intervention to switch to the Recovery site**." ✅
- "**Native HA CRR is a high availability and disaster recovery solution, requiring 6 nodes, 3 in each region**." ✅
- IBM's documentation curates a direct comparison: *Comparison of Native HA CRR and Native HA IRR solutions*, in the MQ 10.0.x documentation. ✅

**Mechanism and characteristics:**

- **Replication is asynchronous.** CRR was described on introduction as permitting "asynchronous replication of log data supporting unplanned failover and planned switchover." ✅ As with DR/HA RDQM, there is therefore a non-zero RPO on a site-level event.
- **The recovery group is itself three replicated instances**, so it is highly available in its own right: "Much like the Native HA group that accepts application workload, a Cross-region replication recovery group is also formed of 3 x replicated instances, so is itself highly available." ✅
- **Deployment.** "Deploying a Native HA group with either a live or recovery role couldn't be simpler either using the MQ Operator or using the sample helm charts." ✅
- **No third-party dependencies.** CRR provides "integrated HA & DR capabilities with just simple storage requirements and **without any 3rd party dependencies**" ✅ — a deliberate contrast with RDQM, which rests on DRBD and Pacemaker.
- **Planned switchover, not just failover.** CRR "support[s] unplanned failover and planned switchover" ✅ — and IBM promotes a second use case that has nothing to do with disasters: migrating a queue manager between clusters or cloud providers while retaining message data. The documented sequence is to deploy a new queue manager group with a Recovery role, reconfigure the existing queue manager to replicate to it while continuing to process work, then swap the Live and Recovery roles. ✅
- **MQ Console visibility** of Native HA queue manager status arrived with IBM MQ 10.0. ✅
- **The storage rule still applies:** "When using Native HA (including CRR) you should use simple RWO block storage that is not replicated — Native HA is responsible for the replication, not the storage layer." ✅ Do not double-replicate.

**Platform availability — the honest position.** Native HA is available on container deployments (IBM MQ Operator on OpenShift, and Kubernetes on EKS in the MQ 10.0 timeframe) and, per the 9.4 documentation, on Linux. ✅ CRR deployment is described through the operator and Helm charts. ✅ An estate running MQ classically on AIX or on z/OS has **no** CRR path — for z/OS the DR story is the classic sysplex and Coupling Facility / queue-sharing-group model, which is outside this guide's scope but must not be confused with CRR.

**What CRR does not give you.** It is not synchronous, so it is not a zero-RPO answer. Its RPO is bounded by the asynchronous log-replication lag, which you should measure rather than assume. It also does not replace the need for a configuration and certificate discipline (§7, §8.4) — a queue manager that replicates its log perfectly and comes up with an expired certificate is still down.

### 5.4 MQ Appliance DR

The appliance has had DR longer than most of the Linux options, and its model has matured in a way worth noting.

- **Base DR:** "In a simple scenario, if a 'main' site fails, the recovery process begins to ensure business continuity. The secondary DR Queue Managers on the remote site are converted to Primary QMs and thus become Primary instances on the recovery appliance or appliances, as policy dictates." ✅ First the queue managers were standalone; later the HA pair itself became the DR source.
- **DR is administratively orchestrated, HA is automatic.** ✅
- **Network expectations are different by design:** "Just one network connection is required for DR between the MQ Appliances hosting the Queue Managers. The appliances are designed to tolerate a level of network latency between distant sites — different than High Availability MQ appliances, which tend to be in close proximity, and are designed for automatic takeover with no data loss." ✅
- **9.3.2 added DR between HA groups on the appliance** — "an HA group of appliances on both sides of a DR link, as per the equivalent DR support that is available for RDQM on Linux. Prior to 9.3.2, an HA group of appliances can only have a single appliance at a remote location for disaster recovery." ✅ The pre-9.3.2 approach is now referred to in IBM's documentation as the *legacy* solution, and IBM's stated benefit of the new capability is operational: it "avoids the need to reconfigure HA and DR after each DR fail over." ✅
- **Replication direction:** "Replication of queue manager data is performed from the current HA primary appliance (that is, where the queue manager is running) to the HA primary appliance at the DR site." ✅
- **The client-routing constraint is the one to design around:** "Queue managers can have a floating IP address within each HA group, but **IP addresses cannot float across the DR link from one HA group to another.** Applications can use MQ connectivity options, such as a CCDT or a connection name list to try to connect to each site in turn, or they can be routed to the correct site by using a global load balancer, DNS entry, or an equivalent network routing capability." ✅ Compare that with RDQM, where a floating IP makes the site change largely transparent to a well-configured client.
- **Failback needs explicit planning:** "If your HA queue manager is configured for disaster recovery, and failed over to the recovery appliance when your HA group went out of service, then you might have to resolve data partitioning between the HA group and the recovery appliance. After you have restored your HA group, and resolved data partitioning… you must follow the procedure described in *Switching back to the main appliance*." ✅ Failback on the appliance is a *documented procedure*, not the reverse of failover — a pattern that recurs across every MQ DR mechanism.

### 5.5 Storage-Level Replication, and Why It Is a Trap for a Messaging Layer

The generic infrastructure answer — replicate the LUN, mirror the volume, use array-based replication or a hypervisor-level DR orchestration product — is available to any MQ deployment and is used widely. It is also the route most likely to produce a *plausible but wrong* recovery.

- **Consistency is the problem.** A queue manager's data directory and its log directory must be recovered as a matched pair at a consistent point. Replicate them on different schedules, or let one volume lag the other, and the recovered queue manager has a log that does not describe its data. IBM's own backup guidance makes the requirement explicit: for circular logging, back up the data and log directories *at the same time*; for linear logging, back up both at the same time, and note that it is possible to restore only the data files *if* a corresponding complete sequence of log files is available. ✅ That is the consistent-snapshot requirement, stated by IBM in the backup context and equally true for replication.
- **Crash-consistent is not enough for a bus.** Array-level replication typically gives crash-consistent images. For a single queue manager that is survivable because MQ's restart recovery is designed for exactly that. For a *cluster* or a set of queue managers that exchange messages, a crash-consistent image of each node at a *different* instant produces a set of queue managers whose channels disagree about what they sent and received.
- **The in-flight transaction set is the awkward part.** The recovered queue manager will resolve in-doubt units of work against its channel partners (§6.3). If the partner recovered to a different point in time, the resolution is a human activity, not an automatic one.
- **The storage is usually shared with something else that has a different consistency requirement.** Replicating the MQ LUN with the same schedule and consistency group as a core-banking database volume is convenient and frequently wrong, because the two have different RPOs and different recovery semantics.
- **It hides behind HA.** Storage replication is often implemented *as* the HA mechanism — a mirrored SAN — in which case the "DR site" is a second array in a second room that shares a failure domain with the first. The DR plan then has a single underlying assumption that nobody has documented.

**Guidance:** storage-level replication is a legitimate *supplement* — for example, to protect against an array failure beneath an RDQM or appliance deployment, or to place a cold copy somewhere for a last-resort restore. It is a poor *primary* DR mechanism for a messaging layer unless the implementation guarantees matched, crash-consistent, coordinated snapshots of log and data, and unless the assumption is written down and tested. IBM's own newer mechanisms (RDQM, Native HA CRR) exist precisely because putting MQ's replication in MQ — rather than in the storage layer — makes the consistency problem tractable.

### 5.6 Log Shipping

A note on log shipping, because it is often proposed and is not a native MQ capability:

- MQ has *archival* of linear log extents — "Log files needed for restart recovery must always be available, whereas log files needed only for media recovery can be archived to longer term storage, for example, tape" ✅ — and automatic log management can archive extents as they become inactive. ✅
- That archive is a **recovery** artifact for media recovery, not a **replication** stream. There is no native mechanism to ship live log to a remote node and stand up a queue manager from it. Point-in-time replay from an archive to a chosen moment does not exist as a supported MQ operation (§2.5).
- If a design proposes "log shipping to the DR site," the correct response is to ask which product feature is meant. On the modern product line the answers are RDQM DR/DR-HA (DRBD), Native HA CRR/IRR (MQ log replication), the appliance's DR replication, or storage replication — not log shipping.

### 5.7 Hybrid Designs

Real estates combine these, and the combinations are where competence shows:

- **Native HA (3 nodes, one cluster/region) + CRR (6 nodes, two regions).** The modern, third-party-dependency-free answer for a containerised MQ estate: local automatic failover inside each region, asynchronous log replication and a planned switchover across regions.
- **RDQM HA (three nodes, one site) + RDQM DR (pair to a second site) as a DR/HA RDQM.** The Linux, IBM MQ Advanced answer with synchronous local replication and asynchronous inter-site replication.
- **Multi-instance locally + separate application-level DR.** The weakest but most common pattern: HA against node failure, and DR answered by "the consuming application can be repointed and can re-drive from its own store." This is viable *only* where the application genuinely holds an independent source of truth and a reconciliation control — which is a strong architectural claim, not a default. The reconciliation and idempotency machinery this depends on is the subject of [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md).
- **Native HA + Uniform Cluster + CRR.** IBM explicitly promotes the combination: "For ultimate message and service availability, consider combining the benefits of automatic client reconnect to a Uniform cluster of Native HA queue managers with Cross-region replication." ✅ Uniform clusters rebalance connected applications across queue managers automatically, which converts a failover into a quieter event for the client population.
- **MQ Appliance HA group + DR to an appliance HA group at the second site (9.3.2 and later).** The appliance-shaped equivalent of DR/HA RDQM.

**The pattern in every hybrid:** local replication is synchronous and automatic; inter-site replication is asynchronous and either manual or explicitly orchestrated. That asymmetry is not an implementation shortfall — it is the physics of the speed of light, and a DR design that assumes cross-site synchronous replication for a high-throughput messaging layer should be treated as unverified until it has been measured on the real link.

---
## 6. Failover and Client-Side Recovery: Identity, Reconnection, In-Doubt Work

A queue manager that has failed over successfully is not a recovered messaging service. Recovery completes when clients are reconnected, channels are re-established, in-doubt units of work are resolved, and the backlog is moving. This section is about the half of the problem that lives outside the queue manager.

### 6.1 Queue Manager Identity: What Must Match and What Must Not

The queue manager **name** is the identity that applications, channels, and administrative tooling are bound to. The default discipline is:

- **Keep the queue manager name stable across failover.** Multi-instance queue managers, RDQM, Native HA, and appliance HA all preserve the queue manager name deliberately — the whole point of multi-instance is that "applications reconnect to the familiar name." ✅ Note that IBM's own documentation phrases this as the client connecting to the same queue manager, not a different one.
- **Do not put the host name in the application.** Where a client resolves a hostname, the failure of the active host changes the address, and the client must be told. Where a client uses a queue manager name plus a channel, the address is a separate concern that a connection list, a CCDT, a floating IP, or DNS handles.
- **A floating IP is the cleanest answer** where the platform supports it — RDQM's three instances "share a floating IP address, so clients only need to be configured with a single IP address." ✅
- **A DNS name that follows the active node** is the generic equivalent, and it has a TTL problem: a client with a cached DNS entry survives the failover *logically* and fails *practically*. DNS failover must be paired with short TTLs and a client that re-resolves.
- **Where identity must differ, the mapping must be explicit.** If the DR site necessarily publishes a different queue manager name — which happens in estates that cloned a production queue manager without renaming it, or where a partner's channel definition expects a specific name — then the mapping from production name to DR name must be documented, and every channel's `CONNAME`, every `QMNAME` reference, and every remote queue definition that names it must be included in the change set. **The queue manager name is the contract; a DR plan that changes it silently is a plan that breaks at the worst moment.**
- **Cluster membership is part of identity.** A queue manager in an MQ cluster carries cluster membership and repository roles (§8). A DR queue manager that comes up with the wrong cluster configuration can be worse than one that comes up isolated.

### 6.2 Channel and Client Reconnection

- **Channels reconnect.** "IBM MQ MQI clients and channels reconnect automatically to the standby queue manager when it becomes active." ✅ This is the mechanism that makes multi-instance failover tolerable without application changes — *if* the applications are configured for it.
- **Automatic client reconnection.** Client applications can reconnect automatically without additional code, "by configuring a number of components." It is **inline**: "The connection is automatically restored at any point in the client application program, and the handles to open objects are all restored." ✅ Inline reconnection is the strong form — the application does not need to be written to handle the interruption at a particular call site.
- **Version note on the reconnect hint.** "From IBM MQ 9.2.0 a reconnect request can be sent to a client containing a hint of which queue manager the client should reconnect to." ✅ In a uniform cluster or a balanced pool, this is what keeps reconnection from stampeding all clients onto one survivor.
- **The Java limitation is a real planning constraint.** "Automatic client reconnect is not supported by IBM MQ classes for Java." ✅ Applications using those classes need explicit connection-handling logic, and a DR plan that assumes "MQ clients just reconnect" is wrong for them.
- **Channel reconnection is a configuration property, not a default you can assume.** The channel's reconnect behaviour and the listener's availability on the new active node both have to hold. IBM's own list of "additional components needed to build a high availability solution" puts client and channel reconnection **first** ✅ — ahead of storage, networks, and even monitoring. Treat it as the primary design artifact, not an afterthought.

### 6.3 In-Doubt Units of Work and Requeue

This is the part of MQ failover that decides whether you get duplicates, losses, or neither.

- **A failover stops a queue manager mid-flight.** Messages that were being transmitted sit on their transmission queue until they can be successfully transmitted — a communications failure "is recovered by restarting the channels using the link that failed." ✅ For a *node* failure or a site failover, the same principle applies once channels are re-established.
- **A message caught mid-transmission is delivered twice or not at all, and the design must tolerate both.** At-least-once delivery is the practical guarantee across a failover. The consuming application must therefore be idempotent. **This guide does not re-derive idempotency, dedupe keys, or the dedupe store** — see [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) §6 for that discipline. What this guide adds is *where* the duplicate originates: the window opens at the failover, because the queue manager that had the message and the application that had processed it disagree about the outcome at the instant of the break.
- **In-doubt channels.** A channel with an in-doubt unit of work holds messages whose fate is unresolved: the sender does not know whether the receiver committed. Resolution is a channel resynchronisation between the partners, and where automatic resolution cannot proceed, operator intervention with the `RESET CHANNEL` command is the documented route. **Version note:** the specific resynchronisation behaviour and the exact `RESET CHANNEL` semantics are release- and channel-type-dependent (message channels versus MQ Telemetry or AMQP channels behave differently); confirm against your release's *Channel resynchronization* and *RESET CHANNEL* documentation before writing it into a runbook. This is flagged ⚠ in §12.
- **Backout and backout requeue are the safety valves you configure in advance.** A backout threshold and a backout requeue queue turn "the message keeps failing" into "the message is parked where a human can see it." Their state survives a failover only if the queues are persistent-backed and the definitions exist at the DR site — which means they belong in the object-definition export (§7.2), not in someone's memory.
- **Long-running transactions are a failover liability.** A syncpoint held open across the failover must be rolled back, and its size determines how much log replay the failover costs. This is the same mechanism that can exhaust secondary log files (§2.3) and force asynchronous rollback. Bound your unit-of-work sizes not just for throughput but for recoverability.
- **Browsing applications are a special case.** IBM's HA guidance singles them out: "Applications that tolerate failover. In particular you need to pay close attention to the behavior of transactional applications, and to applications that browse IBM MQ queues." ✅ A browse cursor held across a failover is invalid, and an application that assumes a stable browse position will behave unpredictably.

### 6.4 The CCDT and Connection Name Lists

- **The CCDT (client channel definition table)** is the client-side artifact that lets an application resolve a channel and a connection target without hard-coded hostnames. It is generated by defining `CLNTCONN` channels and is used to distribute connection information to client populations.
- **The connection name list** is the client-side list of `host(port)` pairs the client tries in order. IBM's DR documentation leans on it directly: with appliance DR between HA groups, "**Applications can use MQ connectivity options, such as a CCDT or a connection name list to try to connect to each site in turn**, or they can be routed to the correct site by using a global load balancer, DNS entry, or an equivalent network routing capability." ✅
- **The CCDT is a DR artifact, and it is the one most often stale.** It is distributed to clients out of band, so its freshness depends on a publishing process that nobody owns. A DR test that "passes" because the clients happened to still point at a surviving endpoint is not a DR test. Put CCDT version control, generation, and distribution in the runbook, and include "confirm the CCDT matches the intended topology" as an explicit test step.
- **Behaviour on connection failure is a client configuration decision.** Connection name lists are ordered, so the first entry is preferred; a client that exhausts the list fails. With automatic reconnection enabled, the client retries within the reconnect window rather than failing the call. Both settings should be documented per application, because they determine whether a failover is invisible or an incident.

### 6.5 DNS, Network, and Firewall Failover Mechanics

The unrehearsed items that break a DR failover in practice, in rough order of how often they are the actual cause:

- **DNS TTL and resolution.** Short TTLs, and clients that re-resolve on reconnect. A DNS-based failover with a one-hour TTL has a one-hour RTO floor.
- **Firewall rules at the DR site.** The production firewall change ticket that opened the client-to-queue-manager ports does not automatically exist at the DR site. New source IP ranges, new listener ports, channel pairs to partner organisations, and outbound connectivity to a SWIFT or network service provider all need rules that somebody has to have requested in advance.
- **Partner-side channel definitions.** A counterparty's sender channel points at your `CONNAME`. If your DR site publishes a different address, the partner has to change something, and partner change windows are not yours to schedule. **This is the single most under-planned element of a bank's MQ DR design.**
- **Load balancer and routing configuration at the second site.** The site you never send traffic to has the routing configuration nobody has validated.
- **Listener ports.** The listener on the DR queue manager must exist, be started, and be reachable — and must not clash with something else already running in the DR estate.
- **The `CONNAME` on your own channels.** Sender channels, cluster channels, and client channels all carry addresses that must be valid at the DR site. These live in object definitions, so they are covered by the export in §7.2 — but the *variation* needed at a second site is exactly the kind of manual modification IBM warns about when moving a queue manager between environments. ✅

### 6.6 What Client Recovery Requires of the Application

Stated as a checklist, because this is where architecture meets DR:

- **Idempotency for every consumer on a persistent path.** Owned by [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md).
- **Bounded retry with backoff, not immediate failure.** A client that gives up on the first connection error converts a 90-second failover into a manual restart of every application.
- **Reconnect enabled and configured**, with the Java exemption handled explicitly. ✅
- **No assumption of a stable browse cursor.** ✅
- **A documented handling path for reason codes returned mid-unit-of-work at the moment of failover** — the application must distinguish "my unit of work was rolled back and I can retry" from "my unit of work committed but I did not see the reply." Only idempotency plus a reconciliation control resolves the second case.
- **A reconciliation mechanism against the source of truth**, run after every failover, not only after every DR test.

---

## 7. Backup, Restore, and Configuration Recovery

This is the section where the honest engineering is least flattering, and where the difference between "we back up MQ" and "we can rebuild the MQ estate" is decided. There are two entirely separate backup stories for IBM MQ, they are constantly conflated, and the conflation is the defect.

### 7.1 What a Data-Directory Backup Captures — and Its Price

The **queue manager data backup** copies the queue manager's data and log directories. Done correctly, this captures the queue manager object, the object definitions, and **the message data** — persistent messages live in the queue data files and the log.

IBM's documented procedure and its prerequisites:

- **"Before starting to back up the queue manager, ensure that the queue manager is not running. If you try to take a backup of a running queue manager, the backup might not be consistent because of updates in progress when the files are copied."** ✅
- Stop with `endmqm -w` (a wait shutdown) and only use `endmqm -i` (immediate) if the wait shutdown fails. ✅
- Locate the data and log directories from the queue manager's configuration files, and copy **all** of them including all subdirectories. ✅
- **Do not miss the log control file**, and do not miss the configuration files. "Some of the directories might be empty, but we need them all to restore the backup at a later date." ✅
- For circular logging, back up the data and log directories *at the same time* so that a consistent set is restored. ✅
- For linear logging, back up both at the same time, **and** note that it is possible to restore only the data files *if* a corresponding complete sequence of log files is available. ✅
- Preserve file ownership — on UNIX and Linux, `tar` does this for you, with the caveat that `tar` cannot be used for queues larger than 2 GB. ✅
- Front matter that is easy to lose: take a copy of `qm.ini` as well, because queue manager information is stored there. ✅

**The blunt statement this section owes the reader:**

> **A queue manager data backup is not a disaster-recovery mechanism. It is a stopped-service recovery mechanism, and it should be treated as the last resort beneath DR, not as DR.**
>
> **It captures message data — but only by stopping the queue manager to get a consistent point in time. That means taking it costs an outage, restoring it costs the loss of everything since the backup, and the only honest way to describe its RPO is "the moment we last stopped this queue manager."** On a 24×7 payments queue manager that is not a backup strategy; it is an annual exercise for the audit file.
>
> **And if you take it while the queue manager is running — because the outage was never approved — the backup "might not be consistent because of updates in progress when the files are copied."** ✅ That is IBM's wording. An inconsistent backup of a queue manager is worse than no backup, because it restores to a state that looks plausible and is wrong.

### 7.2 What a Configuration Backup Captures — and What It Absolutely Does Not

The **configuration backup** is the one you actually use, and it captures **no message data at all**. It captures object definitions, configuration, and authorities.

IBM's documented tooling:

- **On AIX, Linux, and Windows: the `dmpmqcfg` command** — "you can use the `dmpmqcfg` command to dump the configuration of an IBM MQ queue manager." ✅
- **On IBM i: the `Dump MQ Configuration` (`DMPMQMCFG`) command** — which dumps "the configuration objects and authorities for a queue manager." ✅
- **`dmpmqcfg` output can be replayed.** Because it emits MQSC, it can be applied to a newly created queue manager to reinstate its object definitions. IBM documents the related use case directly: "When moving a queue manager from one operating system to another, you use `dmpmqcfg` to save the configuration information… and then copy the object definitions across to the new queue manager that you create on the new operating system." ✅
- **IBM warns that this is not a mechanical exercise:** "You must take great care with copying the object definitions, because some manual modification of the definitions…" will be needed. ✅ The cross-platform case is unambiguous; the same warning applies in substance to a production-to-DR copy wherever naming, addressing, or platform differ.
- **Lineage.** `dmpmqcfg` is the current, shipped tool. Its historical predecessor was `saveqmgr`, distributed as an IBM SupportPac, which is how many older runbooks and scripts still describe the process. **Flag: the `saveqmgr` lineage is ⚠-knowledge (community-sourced in this pass), not confirmed against a current IBM documentation page.** What *is* verified is the current name: `dmpmqcfg` on AIX/Linux/Windows, `DMPMQMCFG` on IBM i. ✅

**So the blunt half of the pair, for symmetry:**

> **A configuration backup captures zero messages.** `dmpmqcfg` exports queues, channels, listeners, topics, subscriptions, namelists, and authority records. It does not export the contents of a queue, and it cannot — the message data lives in the queue data files and the log.
>
> **Therefore an estate whose DR plan is "we run `dmpmqcfg` nightly" has a DR plan that rebuilds an empty messaging estate.** It will have every queue and every channel and every authority, and it will have lost every unacked message in flight at the moment of the site loss. Whether that is acceptable is a business decision that must be made explicitly and recorded — because for a payments or settlement path, "empty" is frequently not acceptable, and the answer then has to be one of the replication mechanisms in §5, not a better backup script.

### 7.3 Rebuilding a Queue Manager from Objects

The documented sequence, in outline:

1. **Create the target queue manager** with `crtmqm`. On the DR site, place its data and logs according to the DR architecture (local block storage for RDQM/Native HA, shared storage for multi-instance).
2. **Configure the queue manager's configuration files** — `qm.ini` in particular, including the `Log` stanza (`LogType`, `LogManagement`), channel, and TLS stanzas. This file does not come from `dmpmqcfg`; it is part of the queue manager's filesystem, which is why keeping a copy of it is documented as a backup step. ✅
3. **Apply the object definitions** exported with `dmpmqcfg` (or `DMPMQMCFG`), applying the documented manual modifications for the other environment.
4. **Reconfigure the TLS environment** — key repositories, certificate labels on channels, `SSLCIPH` settings — before anything needs to connect (§8.4).
5. **Recreate authority records** — including CHLAUTH rules, if they were exported, and the OS-level or RACF-level identities the application uses.
6. **Start the queue manager and verify by inspection**: queue depths, channel status, listener status, cluster membership (§8), and a test put/get through each critical channel.
7. **Record the elapsed time.** Step 7 is the one that tells you whether the rebuild is a DR mechanism at all, or merely a recovery-of-last-resort that will exceed the RTO by an order of magnitude.

**The practical finding from doing this an honest number of times:** a configuration-only rebuild of a moderately complex queue manager is a multi-hour activity dominated by the manual modifications, the certificate work, and the verification — not by the MQSC replay, which is fast. Config-only rebuild is a *complement* to a replication-based DR mechanism, never a substitute.

### 7.4 Version and Configuration Drift Between Primary and DR Site

This is the quiet killer, and it is endemic to DR sites because they receive change only when someone remembers.

- **MQ version and maintenance-level drift.** The DR site must be at a level the primary's data can be recovered onto. IBM's own guidance on moving a queue manager between environments warns that object definitions need manual modification; the version dimension is stricter still — a queue manager cannot generally be recovered onto an older release, maintenance levels must be applied coherently, and on the appliance the firmware level and the supported hardware generations are explicitly coupled. ✅
- **RDQM node-level drift is explicitly forbidden:** migrating RDQMs requires all nodes to be upgraded in sequence, and IBM states "Do not try to operate with the nodes at different levels." ✅
- **OS and kernel drift, specifically for RDQM.** RDQM depends on a DRBD kernel module that "must be compatible with the current OS kernel level." ✅ An unattended OS kernel update on one node in the DR group can leave the group unable to start — and it will be discovered during a failover.
- **Configuration drift.** Queue definitions, channel attributes, TLS cipher specifications, CHLAUTH rules, exit programs, and monitoring configuration all drift. `dmpmqcfg` output compared between primary and DR site is a cheap, mechanical drift detector and should be run on a schedule, with differences triaged rather than tolerated.
- **Certificate expiry drift.** §8.4 — the classic.
- **Capacity drift.** The DR site is frequently sized for a fraction of production throughput, on the theory that it only needs to run until the primary returns. If a DR event keeps the DR site in service for weeks — as real events do — the under-sized DR site becomes the incident.
- **Client-side drift.** Stale CCDTs, stale DNS TTLs, stale partner `CONNAME` values (§6.4, §6.5).

### 7.5 The Restoration Test Cadence an Institution Should Require

A defensible cadence, expressed as requirements rather than preferences:

- **Object-definition export: at least daily, automated, version-controlled, and off-site.** This is cheap and it is the one backup that can be run without an outage. Failure to produce today's export should raise an alert.
- **Drift comparison between primary and DR definitions: at least monthly.** Triage every difference.
- **Certificate inventory and expiry forecast: at least monthly, with an alerting threshold measured in months, not days.** For an estate with annual certificates, a 30-day warning is a warning you receive too late to procure, install, test, and evidence the change.
- **A restore of object definitions into a scratch queue manager: at least quarterly.** Proves the export is usable, not merely present.
- **A full queue-manager data restore from backup into an isolated environment: at least annually**, with the elapsed time recorded against the RTO. This is the test that reveals whether the stopped-service backup is a viable last resort.
- **A DR failover exercise: at least annually, with a failback exercise either in the same year or the next** (§9).
- **A media-recovery exercise on a restored copy: at least annually**, with a deliberately populated queue and a deliberately deleted log extent (§3.6).
- **Evidence retention:** every one of the above should produce a dated artifact identifying what was tested, by whom, timed against the objective, with the deviations recorded. This is the material an operational-resilience function and an external auditor will request; see [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) for the evidence model.

---

## 8. The Cluster and Estate-Level Recovery

A bank's MQ estate is rarely a set of independent queue managers. It is a cluster, or several, and the cluster has its own recovery characteristics — including several that are actively hostile to a naive DR design.

### 8.1 Cluster Repository Structure

- A **cluster repository** is the collection of information about the queue managers that are members of a cluster: "queue manager names, their locations, their channels, which queues they host, and other information." It is stored as messages on a queue — `SYSTEM.CLUSTER.REPOSITORY.QUEUE`, one of the default objects created with every queue manager. ✅
- **Full repository versus partial repository.** "A queue manager that hosts a complete set of information about every queue manager in the cluster has a full repository. Other queue managers in the cluster have partial repositories containing a subset of the information in the full repositories." ✅
- **Typically two full repositories.** "Typically, two queue managers in a cluster hold a full repository; the remaining queue managers all hold a partial repository." ✅
- Partial repositories hold information "about only those queue managers with which the queue manager needs to exchange messages," and refresh it by asking a full repository, via `SYSTEM.CLUSTER.COMMAND.QUEUE`. ✅

### 8.2 What Survives the Loss of a Repository Queue Manager

- **Two full repositories is the design intent, and it is a DR control.** With two full repositories, the loss of one leaves the cluster able to refresh repository data from the other. Losing both simultaneously — which happens trivially if both are hosted on the same pair of nodes, in the same data centre, or on the same HA group — leaves new members unable to join and members unable to learn about new cluster queues. **Check where your two full repositories actually live; it is a common finding that they are co-located.**
- **Existing partial-repository information survives for a while.** A queue manager with a populated partial repository continues to route to the queue managers it already knows about. It degrades gracefully, then wrongly — stale repository data means routing to queue managers that no longer exist and not routing to ones that do.
- **Cluster queue resolution requires the full repository.** A member that needs information it does not have must ask a full repository; if none is available, cluster workload distribution to unfamiliar destinations stops. This is why the full-repository pair is a genuine availability dependency, and why it belongs in the DR runbook's start-up order.
- **Loss of a *cluster member* is different from loss of a *repository*.** The surviving cluster continues to route new messages to surviving instances of a cluster queue — but "messages currently queued on an unavailable queue manager are not available until that queue manager is restarted," and therefore "queue manager clusters alone do not provide high availability of all message data or provide automatic detection of queue manager failure and automatic triggering of queue manager restart or failover." ✅ The messages already sitting on the lost member are lost with it unless that member is itself recoverable.

### 8.3 Why a Cluster Is Not a DR Mechanism

This is worth stating as firmly as the HA-is-not-DR distinction, because the reasoning is identical and the mistake is just as common.

- **A cluster redistributes work; it does not replicate data.** Cluster workload balancing routes *new* messages to available instances of a cluster queue. It has no mechanism to copy a message that is already sitting on a queue manager to another one.
- **IBM is explicit:** clusters provide "load balancing of messages across available instances of queue manager cluster queues," which "offers higher availability than a single queue manager," but they do not "provide high availability of all message data." ✅ IBM also notes that "queue manager clusters alone do not provide automatic detection of queue manager failure and automatic triggering of queue manager restart or failover," and that HA clusters (or multi-instance) provide those. ✅
- **All cluster members in one site share a failure domain.** A cluster spanning two nodes in one data centre is not a DR design; it is a load-balancing design with an availability side-effect.
- **Members can span sites, but then the messaging becomes a cross-site synchronous problem** — cluster channels between sites inherit the latency of the link, and the cluster's channel state becomes part of the recovery story.
- **Cluster error recovery is not failover.** "From IBM WebSphere MQ Version 7.1 onwards, cluster error recovery reruns operations that caused problems until the problems are resolved." ✅ That is a retry discipline, not a recovery mechanism, and it can mask a genuinely broken member by retrying forever.

**Correct use of a cluster in a DR design:** as a *service availability* mechanism that keeps some applications working when a member fails, and as a *load-balancing* mechanism that makes the failover of one member a quieter event for clients. It is a complement to HA and DR, never a replacement for either.

### 8.4 Certificate, Keystore, and Truststore Recovery — the Classic DR Failure

This is the most common way a technically correct MQ DR plan fails in practice, and it fails at the worst moment because certificates expire silently while everything else is fine.

**The verified substrate:**

- On AIX, Linux, and Windows, TLS credentials live in a **key repository** (a GSKit CMS keystore, conventionally a `.kdb` file, configured by e.g. `SSLKEYR`). "The key repository is secured with a password as it contains sensitive information." ✅ IBM documents options for storing that password securely, including encrypted key-repository passwords on AIX, Linux, and Windows. ✅
- A key repository can serve as a **truststore**: "When creating outgoing TLS connections you should create a simple 'truststore' which can validate certificates signed by a common set of certificate authorities (CAs)." ✅ The distinction between the queue manager's identity keystore and the truststore that validates partners is one that DR runbooks routinely blur.
- TLS on IBM MQ channels is negotiated by IBM's GSKit with the algorithms specified by `SSLCIPH` on the channel; there is no separate TLS daemon to restart. ✅
- **z/OS uses a different credential store** — RACF key rings rather than GSKit `.kdb` files. **Flagged ⚠:** the z/OS key-ring detail is ⚠-knowledge in this pass, not re-verified against an IBM documentation page. The operational implication is what matters: a DR runbook for a hybrid distributed-plus-z/OS estate has *two* certificate recovery procedures, not one.

**Why certificate recovery breaks DR failovers:**

- **Certificates expire on a calendar, not on a change ticket.** A DR site that receives no changes for eleven months will have certificates that were valid when the site was built and are expired now. The queue manager starts, the channels do not, and the symptom is a security error against a partner — which reads as an authentication problem rather than an expiry problem.
- **The DR key repository is a copy of a production keystore taken at some past point**, so every certificate rotation done in production has to be repeated at the DR site — and the second step is the one that gets dropped from the change record.
- **The key repository password is required at DR time and is often held only by the person who built the site.** Encrypted-password configuration reduces this exposure but must be set up *before* the event.
- **Truststore renewal is a separate cadence from identity renewal.** A partner's CA rotates, your truststore is stale, and channels to that partner fail on a channel that "was working yesterday." At a DR site that "was working" may be months ago.
- **Certificate labels on channels must match certificate labels in the keystore.** A `CERTLABL` that does not exist in the recovered key repository fails exactly like a missing certificate and is far less obvious.

**The certificate recovery checklist a DR runbook needs:**

1. An inventory of every certificate: subject, issuer, expiry date, keystore, and the channels that use it.
2. A documented rotation procedure that includes the DR site as an explicit step, with its own verification.
3. A truststore refresh procedure covering every counterparty CA.
4. A key-repository password recovery path that does not depend on one individual's memory.
5. A **certificate expiry exercise** — deliberately let a test certificate expire in a non-production environment and prove that the failure is detected, diagnosed, and remediated inside the RTO. This is one of the highest-value exercises in the whole DR programme and almost nobody runs it.
6. A key repository—and truststore—restore test as part of every DR exercise, with a channel connection actually established through TLS afterwards, not merely a keystore that opens.

### 8.5 Authority and CHLAUTH Configuration Recovery

- **Authority records** — the object-level authority grants to principals and groups — are part of a `dmpmqcfg` export on AIX/Linux/Windows, and IBM i's `DMPMQMCFG` explicitly dumps "the configuration objects and authorities for a queue manager." ✅ So the modern export tooling does cover authorities, which is an improvement over the era when authorities were frequently a separate, forgotten export.
- **CHLAUTH rules** are channel authentication configuration, expressed as MQSC, and are therefore exportable by the same tooling. The recovery risk is not that they are unexportable but that they are *unreviewed*: a CHLAUTH ruleset that has accreted over years of incident fixes, with block rules that reference IP addresses of systems that no longer exist, will fail at the DR site in a way that presents as "the channel won't start and the reason is 2035/2063-ish."
- **The DR site's addressing is different.** Any CHLAUTH rule that blocks or allows by IP address or hostname must be re-verified against the DR site's addresses — and the §5.4 appliance constraint (floating IPs cannot cross the DR link) is exactly the kind of topology change that invalidates an IP-based ruleset.
- **A test connection through the CHLAUTH ruleset must be part of every DR exercise**, from the *real* client address and with the *real* client identity. A channel that starts but rejects the application is not a successful failover.
- **The TLS and CHLAUTH layers stack**, and failures in either present similarly to an operator under pressure. The runbook should have a decision tree that distinguishes "certificate problem," "CHLAUTH problem," "authority problem," and "network problem" by evidence — the queue manager error log, the channel status, and the actual reason code — rather than by guesswork.

### 8.6 Channel Initiator, Listener, and CCDT Recovery at Estate Level

- **Listeners must exist, be configured, and be started.** A queue manager that comes up without its listener is a queue manager that no client can reach. Listener definitions are part of the object set; the *started* state is not. Start-up automation must include listener start, and the DR runbook must verify it.
- **Channel initiators** are started via the queue manager's `SCHINIT`/`SCMDSERV`/`SCSTAT` disposition settings and by the `START CHINIT` command. If channel initiator startup is not automated, channels will not initiate inbound connections and the failover will look like a network problem.
- **Cluster channels and repository roles must be verified after failover**, not assumed — including which queue managers hold the full repository at the DR site (§8.2).
- **CCDT distribution is an estate-level process with an owner.** It has to be regenerated after any topology change, published to every client, and version-controlled. "The clients still have the old CCDT" is one of the five most common findings in a first real DR test (§9.3).

---

## 9. The DR Testing Discipline

**A DR plan that has never been executed end-to-end is a hypothesis, and it should be labelled as one in every governance report until it has been.** This section is about converting the hypothesis into evidence.

### 9.1 Exercise Design

A credible IBM MQ DR exercise has all six of the following; the absence of any one of them is why exercises produce comfort rather than confidence.

- **Scenario coverage.** At minimum, over a multi-year cycle: (a) unplanned loss of the active node (HA path); (b) unplanned loss of the site (DR path, forward); (c) planned switchover for maintenance (the benign path, which still fails surprisingly often); (d) planned **failback** to the primary site; (e) media recovery on a restored copy (§3.6); (f) a configuration-only rebuild of a queue manager (§7.3); (g) a certificate expiry event (§8.4). Most programmes do (a) and (b) and never (d), (f), or (g) — and those are the three that break.
- **Named participants with real roles.** The queue manager administrator, the storage or Kubernetes engineer, the network engineer with authority to change firewall rules, the application owner who can restart and verify consumers, the partner-connectivity owner for outward channels, the security engineer for certificates, and a business representative who can rule on whether the achieved position is acceptable. An exercise with only MQ administrators present tests only the MQ part.
- **A business representative present.** The point of the exercise is to establish whether the achieved RTO and RPO are *acceptable to the business*, not whether the technology worked. That judgement cannot be delegated to the platform team.
- **Timing, measured end-to-end from the declaration of the incident.** Start the clock when a plausible trigger occurs (not when the team is told "the exercise starts now"), and stop it when the business service is confirmed working — not when the queue manager is up.
- **Evidence capture.** Timeline of actions with timestamps, the commands run, the operator messages observed, queue depths before and after, the measured RPO (what was actually lost), the measured RTO (what it actually took), every deviation and manual step not in the runbook, and the names of those who made the decisions.
- **A post-exercise remediation list with owners and dates**, and a re-test of the items that failed. A finding that is not re-tested is not remediated.

### 9.2 Measurement: Achieved RPO and RTO Versus Target

The measurement is the deliverable. Three disciplines:

- **Measure message-layer RPO as a count, not only a duration.** How many messages existed before the event and how many were recovered? A test that verifies "the queue manager started" without counting messages has measured availability and not integrity. If the DR mechanism is asynchronous, the count of lost messages is the number that matters, and it needs to be reconciled against the source systems to confirm nothing financial was lost silently.
- **Measure RTO in stages**, because aggregated numbers hide the fixable part: time to *decide* to fail over (usually the largest single component), time to promote/replicate-complete, time to start the queue managers in the right order, time to re-establish channels, time for consumers to reconnect and drain the backlog, time to reconcile. The decision time is a governance problem, not a technical one, and it is the one most worth attacking.
- **Record the achieved numbers next to the target**, and escalate the gap as a risk if the achieved RTO or RPO exceeds what the business-service impact tolerance permits. Mapping the messaging layer's numbers onto a business-service impact tolerance is the work described in [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) §4 — this guide does not re-derive the framework, but it supplies the message-layer numbers the framework needs.

**A candid note on measurement:** the first honest DR test of an MQ estate almost always produces a worse number than the documented target, and the temptation is to record the target. Recording the achieved number is the entire point. A documented RTO of 4 hours that has never been measured is a marketing figure; an achieved RTO of 6 hours 20 minutes that has been measured, published, and put on a remediation plan is an engineering fact.

### 9.3 The Failure Modes DR Tests Habitually Uncover

Ranked roughly by frequency across real exercises:

1. **Certificate and truststore drift.** Expired certificates, a certificate label on a channel that does not exist in the recovered keystore, a stale truststore that cannot validate a partner's rotated CA, a key repository password nobody present knows. Presents as an authentication failure and consumes the exercise (§8.4).
2. **Configuration and object-definition drift.** The DR site's queue definitions are months behind. A CHLAUTH rule references an address that has changed. A backout queue that exists in production does not exist at DR. A `MAXMSGL`, a `DEFPSIST`, or a channel attribute differs in a way that changes behaviour under load.
3. **Client-side CCDT staleness and hard-coded hostnames.** An application that was supposed to use the CCDT has a hostname compiled in, or a CCDT was published that never reached one team's servers. This is usually the reason the "queue manager is up but nothing is flowing" period is longer than the RTO.
4. **Untested reverse failover.** Fail-forward has been exercised three times; failback has never been. Failback on the appliance is a documented procedure, RDQM DR reveres the roles after resynchronisation, and the DR/HA and CRR cases require re-establishing replication in the other direction — all of which are distinct procedures with distinct risks (§5).
5. **Storage and quorum behaviour under partial failure.** The exercise "loses the site" cleanly. Real events lose two of three RDQM nodes, or lose connectivity between a node and the file system, or lose the replication link while both sides stay up. Those partial-failure paths — where quorum is lost, where the appliance reports *Remote appliance(s) unavailable*, where RDQM DR reports *partitioned* — are the ones that require human judgement and are almost never exercised. RDQM DR's documented split-brain guidance and the appliance's `makehaprimary` procedure (after taking backups of both data sets) should each be rehearsed (§4.1, §5.1, §4.4).
6. **Undocumented manual steps that only exist in someone's head.** The single most valuable output of a first exercise is the list of things the team did that were not in the runbook. Those are your real runbook; write them down.
7. **Order-of-start dependencies.** Which queue managers must come up first, which full repositories, which channel initiators, which listeners, which applications. A DR site brought up in the wrong order produces a system that is up but not connected, and diagnosing that costs more than the failover.
8. **Storage-layer prerequisites.** NFS or SAN mounts that require a manual step, RDQM kernel-module and OS-level compatibility, block-storage provisioning at the DR site that has never been exercised, DRBD and Pacemaker cluster startup timing.
9. **DNS, firewall, and load-balancer changes with lead times longer than the RTO.** A firewall change with a two-day SLA is not compatible with a four-hour RTO, and finding that out during an exercise is the cheapest possible way to learn it.
10. **Capacity.** The DR site cannot carry production peak even after it is running. Load-test the DR site at least once with production-representative volumes.

### 9.4 Frequency, and the Regulatory-Evidence Angle

- **Frequency.** Annual is the floor for a full site-failover exercise with business representation. Quarterly is defensible for the lighter-weight elements (HA failover, object-definition restore, certificate inventory, drift comparison). The cadence should be justified against the business services that depend on the messaging layer, not chosen because it is the industry norm.
- **The regulatory-evidence angle, kept condensed.** A regulator's interest is not in the DR product you bought but in whether you can demonstrate that the service will stay within its impact tolerance and that you have tested it. That means dated evidence, named accountable owners, a tested plan, and a record of findings and remediation. The framework — impact tolerances, the business-service mapping, the evidence model, the third-party and outsourcing expectations — is owned by [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md); §10 below does only the messaging-layer mapping.
- **The uncomfortable regulatory point.** A DR test that fails, is recorded honestly, and is remediated is a *stronger* control position than a DR test that is never run and reported as "in place." Plan for the first exercise to produce findings, and pre-commit to publishing them.

### 9.5 The Warning

**A DR plan never executed end-to-end is a hypothesis. It is not a control, it is not evidence, and it does not reduce risk — it relocates risk to the moment of the incident, where it is most expensive.** The corollary is uncomfortable and worth writing into the plan itself: the most dangerous state for an MQ estate is a well-documented, well-intentioned, never-rehearsed DR design, because it suppresses the anxiety that would otherwise cause someone to test it. Everything in §7 and §8 is discovered by doing; almost none of it is discovered by reading.

---
## 10. The Regulated-Institution Angle

### 10.1 Why the Messaging Layer Is a Systemic Dependency

IBM MQ is rarely the system a bank talks about, and it is frequently the thing that makes the bank's other systems work. In a typical institutional estate, an MQ backbone sits under:

- **Payment flows.** Interbank and high-value payment traffic between the bank's payment hub, its core systems, and external networks. IBM MQ is a long-standing transport for these interfaces, and in many estates it is the transport for the interfaces into the SWIFT estate as well. The rails themselves are covered in [../banking/payment_rails_guide.md](../banking/payment_rails_guide.md); the SWIFT-specific estate is covered in [../banking/swift_alliance_access_guide.md](../banking/swift_alliance_access_guide.md) and [../banking/swiftnet_fileact_guide.md](../banking/swiftnet_fileact_guide.md). This guide does not re-derive them; what matters here is that the messaging layer is *under* them, so its unavailability is their unavailability.
- **Core banking interfaces.** Postings, account enquiries, limits, and standing instructions move between the cores and the surrounding estate, usually over MQ channels. See [../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md) for the core platforms themselves.
- **File-transfer and batch flows.** Statements, regulatory extracts, bulk payment files, and reconciliation outputs frequently pass through a managed file transfer product riding on MQ, or through MQ channels carrying large messages directly. The Axway CFT-style tooling is discussed in [axway_transfer_cft_guide.md](axway_transfer_cft_guide.md) and [axway_cft_controlm_integration.md](axway_cft_controlm_integration.md).
- **Internal integration.** Master data, risk, fraud, AML, and reporting interfaces. Lower criticality individually, cumulatively significant.
- **Partner and market connectivity.** Counterparty channels, custodian interfaces, clearing and settlement connectivity, and market-data distribution.

**The systemic property is that MQ is a *shared* dependency.** A single queue manager can serve a payment path, a reconciliation path, and a reporting path at the same time. When it fails, it does not fail one business service; it fails a set of business services that happen to have nothing in common except their transport. That is why "we have DR for the payments system" is not the same claim as "we have DR for the messaging layer," and why the estate-level view of §8 and §9 is the one a regulator-facing function needs.

**The corollary risk:** the messaging layer is frequently classified as *infrastructure*, which means it is governed as infrastructure — monitored for uptime, changed under standard change management, and tested for HA. Infrastructure classification is exactly what allows a systemic dependency to go without a DR programme, because infrastructure is assumed to be covered by the platform's BC/DR plan. The queue manager is the platform.
<br>

### 10.2 The Expectations the Institution Will Be Measured Against

The resilience framework — impact tolerances, the mapping from business services to supporting systems, the evidence model, and the third-party dimension — is owned by [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md). This subsection does only what that guide cannot: map those expectations onto the messaging layer, and name the regimes with their verification status.

| Regime | What it is | Why the messaging layer is in scope | Verdict |
|---|---|---|---|
| **DORA** — Regulation (EU) 2022/2554 | EU Digital Operational Resilience Act. Entered into force 16 January 2023; applicable from **17 January 2025**. ✅ | Demands ICT resilience, tested recovery, and oversight of ICT third-party providers — including the software and infrastructure providers the MQ estate depends on (IBM, the cloud/container platform, the storage vendor, the certificate authority) | ✅ verified (ESMA, EIOPA, EBA, and industry summaries) |
| **MAS Business Continuity Management Guidelines** | Revised version issued **6 June 2022** by the Monetary Authority of Singapore ✅ | Directly applicable to a Singapore-headquartered institution's service continuity planning, including recovery objectives and testing expectations for critical systems | ✅ verified (MAS-sourced summaries and legal commentary) |
| **MAS Technology Risk Management Guidelines (TRM)** | MAS guidance on technology risk management, including systems reliability, resiliency, and recoverability ✅ | Sets the technology-risk expectations — availability, recoverability, incident management — that the MQ estate must be able to evidence | ✅ verified as an MAS publication; specific clause numbering not verified this pass |
| **BCBS** — Basel Committee principles on operational resilience / business continuity | International supervisory expectations for operational resilience and continuity planning | Provides the international framing for impact tolerances and continuity planning that national regimes implement | ⚠-knowledge — the specific current BCBS paper titles and dates were not re-verified this pass |
| **FFIEC** — US federal financial institution examination guidance on business continuity | US examination handbook expectations | Relevant to a US presence or a US-facing service path | ⚠-knowledge — not re-verified this pass |
| **PRA / Bank of England** operational resilience requirements | UK regime built around **important business services** and **impact tolerances** | The impact-tolerance construct is the one most directly transferable to a messaging-layer RTO/RPO argument | ⚠-knowledge — not re-verified this pass |

**A caveat that matters more than the table.** Regulatory expectations in this area are in active evolution, the detail is jurisdiction-specific, and secondary summaries of them are frequently out of date. **Do not take the clause-level detail of any regime from this guide.** Verify against the regulator's own text and against the institution's compliance function; the operational-resilience guide in this repository is the appropriate internal reference point.

### 10.3 Mapping the Messaging Layer's RTO/RPO onto Business-Service Impact Tolerance

The construct worth borrowing from the operational-resilience regimes is that the tolerance is set at the level of an **important business service**, and it is expressed as the level of disruption that would cause intolerable harm — not as a system-level uptime target. The messaging layer's job is to demonstrate that it can be recovered within whatever the business service's tolerance permits, and to be honest when it cannot.

The mapping discipline, in five steps:

1. **Enumerate the business services that depend on the MQ estate.** Payments initiation, payments processing, settlement, statements, regulatory reporting, and so on — not "MQ."
2. **For each, record the impact tolerance** as stated by the business: the maximum disruption before intolerable harm. This is a business statement; the platform team's job is to test against it, not to set it.
3. **Decompose the tolerance into a technology budget across the dependency chain.** A business service with a 4-hour tolerance does not hand 4 hours to MQ; the application, database, network, and human decision steps all consume part of it. The message-layer RTO target should be a fraction of the service tolerance, with the budget written down.
4. **Express the message-layer position in the units the business understands:** "the queue managers carrying this service can be failed over in X minutes, and on a region loss we can lose up to Y seconds of message traffic, which is reconciled against the payment hub as follows." The second half of that sentence is the part that usually does not exist and the part that a resilience function will ask for.
5. **Record the gap where the achieved position exceeds the tolerance**, with a dated remediation plan and an accountable owner. An unrecorded gap is the finding.

**The messaging-layer-specific trap in this mapping:** RPO for a messaging service is not one number, because it depends on the message class and the replication mode. A queue manager with Native HA has a local RPO of effectively zero and, with CRR, a region-loss RPO equal to the asynchronous replication lag. A queue manager with synchronous RDQM DR has an RPO of zero on site loss and no local HA at all. An estate with a non-persistent path has an RPO of "everything in flight" for that path and no way to improve it. **The impact-tolerance mapping must be done per queue manager class, not per platform.** A single RPO figure for "MQ" is a fiction.

### 10.4 The Third-Party and Outsourcing Dimension

DORA in particular puts ICT third-party risk squarely in scope, and the MQ estate is unusually exposed to it. A credible control position identifies:

- **The software vendor.** IBM, the product lifecycle of the release in use, and the support window. Which release is in production, when its support ends, and what the upgrade path is. Note that release lifecycle management is itself a resilience control: an estate on an unsupported level cannot receive the defect fixes that recovery sometimes depends on.
- **The underlying platform providers.** The operating system and its kernel level (material for RDQM's DRBD module compatibility ✅), the container platform and its version, the storage provider, and the network provider between sites.
- **The certificate authorities and trust infrastructure.** A third-party CA failure or a rotated root is a channel outage.
- **Concentration risk with counterparties.** If many counterparty channels terminate on the same queue manager or the same site, a single event takes out several third-party relationships at once.
- **Contractual and exit considerations.** Where MQ DR depends on a specific vendor's hardware or firmware — the appliance being the clearest example — the hardware lifecycle is part of the resilience plan (M2003 to 2031, M2002 to 2027 ✅).

### 10.5 The Evidence an Institution Should Hold

The evidence set that a credible MQ DR control position produces. This is a list to be able to hand over, not a list to aspire to:

- **An inventory** of every queue manager, its criticality tier, its HA mechanism, its DR mechanism, its version and maintenance level, its persistence settings, and its connecting applications and counterparties.
- **An architecture decision record per tier** stating why that mechanism was chosen, what its RPO and RTO are, and what residual single points of failure were knowingly accepted.
- **The current RPO and RTO position** — both the designed target and the last measured achieved value, with dates.
- **The impact-tolerance mapping** from business service to messaging component, with the technology budget decomposed.
- **The runbook**, version-controlled, with an owner, and evidence that it was used in an exercise — including the manual steps that were added to it as a result.
- **Dated exercise evidence** for each scenario in §9.1: timeline, participants, commands, measured RPO and RTO, deviations, findings, remediation owners and dates, and re-test results.
- **The configuration export record** showing that object-definition exports were produced, on schedule, and restored successfully into a scratch environment.
- **The certificate inventory and expiry forecast**, with the DR key repositories and truststores included.
- **The version and maintenance-level register** for every queue manager and every underlying platform component, with the upgrade plan.
- **The third-party register** for the components above, with concentration positions.
- **The outstanding risk register entries** where the achieved position falls short of the tolerance, with owners and dates.

**The one-line test of whether this evidence exists:** ask for the last DR exercise's measured RTO and the count of messages not recovered. If either answer requires a search, the evidence is not held.

---

## 11. The Worked Example: Cymbal Bank's MQ DR Design

> **Everything in this section is fictional and illustrative.** Cymbal Bank is a fictional institution. The estate, the tiers, the topology choices, the RTO/RPO figures, the node counts, the costs, and the exercise findings below are constructed examples chosen to demonstrate the reasoning in §1–§10. **No figure in this section is a benchmark, a quotation, a vendor price, or a claim about any real institution.** Where a figure is illustrative it is labelled **[ILLUSTRATIVE]**. The IBM product facts the design rests on (node counts, version gates, platform availability, documentation behaviour) are the verified facts cited in §4, §5, and §12 — the design *decisions* are the fiction.

### 11.1 The Estate Inventory by Criticality Tier

Cymbal Bank is a Singapore-headquartered bank with two data centres — **DC1 (primary, Singapore)** and **DC2 (recovery, Singapore, metro distance)** — plus a small presence in a cloud region. Metro distance between DC1 and DC2 is the design choice that makes synchronous replication feasible for the top tier.

| Tier | Business function | Queue managers | Message paths | Non-persistent traffic | Target RTO | Target RPO |
|---|---|---|---|---|---|---|
| **Tier 0** | Interbank and high-value payments; SWIFT connectivity interfaces | 6 | Payment hub ↔ core; payment hub ↔ SWIFT estate; payment hub ↔ clearing/settlement | **None permitted** | 30 min | 0 for persistent; region-loss lag must be ≤ 5 s and reconciled |
| **Tier 1** | Core banking interfaces (postings, enquiries, limits) | 14 | Core ↔ interfaces; core ↔ channel systems; core ↔ risk | None on financial paths | 2 h | ≤ 30 s, reconciled |
| **Tier 2** | File transfer and batch (statements, extracts, bulk files) | 9 | MFT ↔ core; MFT ↔ SWIFT FileAct-style flows; batch schedulers | Some (telemetry only) | 8 h | N/A — the source file is the system of record and can be re-sent |
| **Tier 3** | Internal integration, reporting, observability | 22 | Reporting, monitoring, internal tooling, non-financial events | Yes, extensively | 72 h | Accepted loss |

**[ILLUSTRATIVE]** The counts are invented. The *shape* — a small number of high-criticality queue managers carrying financial obligations and a much larger tail of low-criticality ones — is the realistic part, and it is what drives a tiered design rather than a uniform one.

### 11.2 Topology Selection and Justification, Per Tier

**Tier 0 — Native HA (3 nodes, DC1) with Cross-Region Replication to a 3-node group at DC2.**

- **Why this over the alternatives:**
  - It is the only option that gives **both** strong local HA and a site-loss mechanism **without a third-party dependency**: Native HA's replicated log needs "simple RWO block storage that is not replicated" ✅ and CRR was introduced as "integrated HA & DR capabilities with just simple storage requirements and without any 3rd party dependencies." ✅ That removes DRBD and Pacemaker from the Tier 0 critical path — a real operational simplification for a payments platform.
  - **Local failover loses no persistent messages**, because the replicas write the log and acknowledge. ✅
  - **CRR gives a planned switchover as well as unplanned failover** ✅, which means the bank can rehearse the DC switch on a Tuesday morning rather than only in a crisis — and can use the same mechanism for a cloud migration later.
  - The recovery group is **itself 3 replicated instances** ✅, so the DR site is not a single node that could fail during the failover.
- **The choice the bank explicitly rejected:** synchronous **RDQM DR**, which would give RPO 0 on site loss — but which cannot coexist with RDQM HA on the same queue manager ✅, so the bank would trade away local HA to buy a zero RPO. For Tier 0 traffic at the bank's volumes, losing local HA (and therefore taking a node failure as a manual event) was judged worse than accepting a small, measured, reconciled region-loss window. **This is the central DR trade-off in the example and it is genuinely arguable in both directions.**
- **Residual exposure accepted in writing:** on a true DC1 region loss, CRR's asynchronous lag means a non-zero number of messages may not have reached DC2. The bank's position is: only persistent messages on Tier 0 paths, a measured lag with a hard alerting threshold, and a payment-hub reconciliation control that identifies and re-drives anything missing. **The reconciliation control is what makes the resulting RPO claim defensible**, and it is the piece a technical DR design alone would omit.

**Tier 1 — RDQM disaster recovery and high availability (DR/HA RDQM): a 3-node HA group at DC1 and a 3-node HA group at DC2.**

- **Why:** Tier 1 queue managers carry financial data but with a weaker immediate-obligation profile than Tier 0, and Cymbal Bank's Tier 1 runs on RHEL virtual machines rather than in containers — so Native HA/CRR is not available to them on the bank's current platform. RDQM DR/HA is the IBM-native option on that platform, and the bank already holds IBM MQ Advanced entitlement.
- **The accepted trade-off, stated plainly:** DR/HA RDQM's inter-site replication "is always asynchronous" ✅, so **the inter-site RPO is non-zero and unavoidable** — the bank cannot buy a synchronous site link at this tier. The bank therefore sets a budget: measure the lag, alert above 30 seconds, and reconcile consumed positions against the core banking source of truth after any real failover.
- **Governance consequence:** this is the tier where the *business* had to accept a non-zero RPO explicitly, in writing, with the reconciliation control as the mitigation. It is the kind of acceptance that a resilience function should be shown once and referenced thereafter.

**Tier 2 — Multi-instance queue managers, local only, with configuration export and business-level re-drive.**

- **Why:** the source of truth for Tier 2 flows is a *file*, held by the sending or receiving system, with an existing re-send procedure. The messaging layer is a transport, not the system of record. A multi-instance pair (one queue manager on shared storage, active plus standby from WebSphere MQ 7.0.1 ✅) plus a nightly `dmpmqcfg` export is proportionate.
- **The honesty requirement:** this tier has **no site-loss DR at the messaging layer at all**. Its DR answer is "the file is re-sent." That is only defensible because the re-send procedure has an owner, a documented SLA, and has been tested — and because §9's exercises include a Tier 2 re-send test. If that procedure is not tested, Tier 2 does not have DR either.

**Tier 3 — Single queue managers with configuration export only.**

- **Why:** internal, largely ephemeral traffic with explicit business acceptance of loss. Local HA is provided by the underlying hypervisor and is not MQ's responsibility at this tier.
- **The honesty requirement:** the bank records the accepted loss in writing with a named business owner, and the acceptance is re-confirmed annually. An undocumented acceptance becomes an undocumented outage.

**Estate-wide, across all tiers:**

- **Linear logging on every production queue manager**, with automatic log management and automatic media images enabled, so media recovery is available. ✅ Validated by configuration drift checks (§7.4) because circular is the default and drift back to it is possible on a newly built queue manager. ✅
- **Persistent messaging on every financial path**, with `DEFPSIST(YES)` verified by drift check rather than assumed.
- **Uniform cluster for the Tier 1 interface queue managers**, so that automatic client reconnection has somewhere sensible to reconnect to. ✅
- **Both cluster full repositories placed in different failure domains** (§8.2) — a specific, cheap control that many estates fail.

### 11.3 The Failover and Failback Runbook, in Outline

**Forward failover (DC1 loss, Tier 0):**

1. **Declare the event.** Named authority (the payments duty manager) decides, with the platform on-call. The clock starts at the *trigger*, not at the decision.
2. **Confirm scope** — is this DC1-only, or has DC2 been affected? Are Tier 0 queue managers intact on the CRR recovery group? Check Native HA status (visible in the MQ Console from IBM MQ 10.0 ✅) and the replication lag.
3. **Promote the recovery group at DC2** to the live role. For a planned switchover, use the supported Live/Recovery role swap ✅; for an unplanned event, use the failover path.
4. **Record the replication lag at the moment of the decision.** This is the RPO evidence, and it cannot be recovered later.
5. **Start queue managers in dependency order:** cluster full repositories first, then Tier 0, then channel initiators, then listeners. Verify each.
6. **Re-establish channels** to the payment hub, the core, and the external network connectivity. For SWIFT connectivity, the SWIFT-estate components on the far side are a separate recovery step owned by a different team — coordinate, do not assume.
7. **Reconnect clients:** confirm the CCDT in use points at DC2; confirm DNS resolution and TTL behaviour; confirm consumers are reconnecting rather than failing. Handle the IBM MQ classes for Java applications explicitly — automatic client reconnect is not supported for them ✅ — by restarting those applications.
8. **Resolve in-doubt work.** Reconcile the payment hub's in-flight positions against the queue managers and decide, message by message where necessary, what must be re-driven.
9. **Run the reconciliation control** against the payment hub and the core. Produce the count of messages not recovered, and the count re-driven.
10. **Declare service restored** only when the business service is confirmed working end-to-end and the backlog is draining — not when the queue managers are up.
11. **Capture evidence** as the exercise progresses, not afterwards.

**Failback (DC1 restored):**

1. **Do not fail back immediately.** A failback is a second planned outage. Schedule it.
2. **Resynchronise** — for CRR, re-establish replication from DC2 back to DC1 as the recovery role and let it catch up; for RDQM, resolve the role situation deliberately. Confirm replication is complete before any role swap.
3. **Resolve data partitioning before promoting DC1.** On RDQM this is a documented procedure; on the appliance it explicitly requires resolving data partitioning between the HA group and the recovery appliance, with backups of both data sets taken first ✅. Treat this as the highest-risk step in the whole runbook, because it is the only step where two divergent copies of the truth exist and a human must choose.
4. **Switch over deliberately**, verify, and then re-establish replication in the original direction.
5. **Confirm DR readiness is restored** — a queue manager that has failed back but whose DR replication is not re-established has *less* resilience than before the event. This state has persisted for months in real estates, and the runbook must have an explicit closing check.

### 11.4 Certificate, Configuration, and CCDT Recovery Steps

**Certificates and key repositories (§8.4):**

1. Maintain the certificate inventory with subject, issuer, expiry, keystore, and the channels using each certificate — including the DR key repositories.
2. Before any DR exercise, verify that every TLS channel can establish a connection at the DR site using the DR keystore and truststore, not merely that the files exist.
3. Confirm the key repository password is available to at least two named people at recovery time, and that encrypted key-repository password storage is configured on AIX/Linux/Windows. ✅
4. Verify every channel's certificate label actually exists in the recovered keystore.
5. On any certificate rotation in production, include the DR site as an explicit, separately verified step in the change record.

**Configuration (§7):**

1. `dmpmqcfg` export of every queue manager on AIX/Linux/Windows, daily, automated, version-controlled, off-site. ✅ (`DMPMQMCFG` for any IBM i components. ✅)
2. Monthly automated diff of primary versus DR object definitions, with triage — the drift detector.
3. `qm.ini` copies held alongside the exports. ✅
4. Quarterly restore of the latest export into a scratch queue manager, to prove usability, not just existence.
5. Apply the documented manual modifications for the DR site (names, addresses, `CONNAME` values) as a version-controlled transformation, **not** as an ad-hoc edit performed at 03:00. ✅

**CCDT and client connectivity (§6.4):**

1. Regenerate the CCDT after every topology change; store it in version control alongside the exports.
2. Publish to every client team, with an acknowledgement from each.
3. Verify at exercise time that the deployed CCDT matches the intended topology — this is the single most common finding in a first real test.
4. Confirm the connection name list ordering and the reconnection window per application, and record them.
5. Maintain the DNS and firewall change set for the DR site with lead times inside the RTO; test that the lead time is achievable.

### 11.5 The DR Test Plan and Evidence

**Annual (with business representative present):**

| Exercise | Objective | Evidence produced |
|---|---|---|
| Tier 0 region failover (DC1 → DC2) | Measure achieved RTO and the CRR lag; prove the payment path works from DC2 | Timeline; measured RTO; measured lag in seconds; count of messages not recovered; reconciliation output |
| Tier 0 planned switchover | Prove the benign path and the failback mechanics | Timeline; deviations; updated runbook |
| Tier 1 site failover (RDQM DR/HA) | Measure achieved RTO; prove asynchronous RPO is within the accepted budget | Timeline; measured lag; business acceptance re-confirmed |
| Certificate expiry event | Let a test certificate expire and prove detection, diagnosis, and remediation inside the RTO | Detection time; diagnosis time; remediation time |
| Configuration-only rebuild of a Tier 1 queue manager | Measure the elapsed time of a `crtmqm`-plus-objects rebuild against the RTO | Elapsed time by stage; deviations |

**Quarterly:**

| Exercise | Objective |
|---|---|
| Tier 0/Tier 1 node failover (HA path, unplanned) | Prove automatic failover, with the clock started by a genuine simulated trigger |
| Object-definition export restore into a scratch queue manager | Prove the export is usable |
| Certificate inventory and expiry forecast review | Prove nothing expires inside the procurement lead time |
| Configuration drift comparison, primary versus DR | Triage every difference |

**Annually, on restored copies, isolated from production:**

| Exercise | Objective |
|---|---|
| Media recovery with a deliberately populated queue and a deliberately deleted log extent | Prove damaged-object rebuild inside the RTO and prove the AMQ6767 path is handled |
| Full queue manager data restore from a stopped-service backup | Prove the last-resort restore works, and measure its elapsed time |
| Tier 2 file re-send procedure with the business | Prove that the tier without messaging DR has a working alternative |

**Every exercise produces:** the timeline, participants and roles, commands and operator messages, measured RPO (in messages and seconds) and RTO (by stage), every deviation and undocumented manual step, findings with owners and dates, and a re-test result for each prior finding. Exercise evidence is retained for the period the compliance function requires and mapped to the business-service impact tolerances.

### 11.6 Cost and Complexity Comparison

> **[ILLUSTRATIVE — FICTIONAL NUMBERS.]** The figures below are invented for the purpose of showing the *shape* of the comparison: that resilience cost is dominated by node count and by operational complexity, and that the cheapest option is cheap because it does not answer the site-loss question. They are not prices, not quotes, and not benchmarks for any real institution.

| Tier / option | Nodes (DC1 + DC2) | [ILLUSTRATIVE] infra cost index | [ILLUSTRATIVE] licensing | [ILLUSTRATIVE] ops complexity (1–5) | Site-loss DR? | [ILLUSTRATIVE] realistic RTO |
|---|---|---|---|---|---|---|
| Tier 0 — Native HA + CRR | 3 + 3 = 6 | 700 | IBM MQ Advanced / container entitlement | 3 | ✅ Yes | 30 min [ILLUSTRATIVE] |
| Tier 0 alternative — synchronous RDQM DR only | 1 + 1 = 2 | 240 | IBM MQ Advanced | 4 | ✅ Yes, RPO 0 | 45 min [ILLUSTRATIVE] — but **no local HA** |
| Tier 0 alternative — RDQM DR/HA | 3 + 3 = 6 | 700 | IBM MQ Advanced | **5** | ✅ Yes, async | 45 min [ILLUSTRATIVE] |
| Tier 0 alternative — multi-instance only | 2 | 200 | Base | 2 | ❌ **No** | N/A |
| Tier 1 — RDQM DR/HA | 3 + 3 = 6 | 640 | IBM MQ Advanced | 5 | ✅ Yes, async | 2 h [ILLUSTRATIVE] |
| Tier 1 alternative — multi-instance + config export | 2 | 200 | Base | 2 | ❌ No | 8 h+ [ILLUSTRATIVE] via rebuild |
| Tier 2 — multi-instance + re-drive | 2 per QM | 180 | Base | 2 | ❌ No | 8 h [ILLUSTRATIVE] via business re-send |
| Tier 3 — single queue manager | 1 | 90 | Base | 1 | ❌ No | 72 h [ILLUSTRATIVE] |
| MQ Appliance HA pair + DR | 2 + 2 = 4 appliances | 900 | Appliance product | 2 | ✅ Yes | 60 min [ILLUSTRATIVE] |

**How to read the table.** The two rows that matter are the ones that look expensive and the ones that look cheap: the Tier 0 options at complexity 3–5 with six nodes are what a site-loss answer costs, and the multi-instance rows at complexity 2 are cheap precisely because they do not answer the question. **The comparison that a governance forum should be shown is not "how much does DR cost" but "what does the business lose if we do not buy it, and has anyone accepted that in writing."**

### 11.7 The Single Points of Failure That Remain

Stated honestly, because a DR design without this section is a sales document:

- **The CRR replication lag on Tier 0.** Asynchronous by design. A region loss loses whatever had not replicated. Mitigated by persistent-only messaging, lag alerting, and the payment-hub reconciliation control — **mitigated, not eliminated.**
- **The asynchronous inter-site link on Tier 1.** Unavoidable on DR/HA RDQM, and accepted in writing by the business.
- **The decision time, not the technology.** The measured RTO will be dominated by how long it takes a human to decide to fail over. No product feature fixes a governance delay, and it is the largest single component of the RTO in most real events.
- **The certificate and key-repository process.** A shared, largely manual, calendar-driven process with a long lead time and no strong verification. This is the most likely cause of a technically successful failover that does not deliver service.
- **The CCDT and DNS publishing process.** Out-of-band, no single owner, no automatic verification that deployed clients match the intended topology.
- **External connectivity to the SWIFT estate and to counterparties.** Outside MQ's control, and outside Cymbal Bank's control in part. The bank can fail its queue managers over perfectly and still not be able to send a payment.
- **Tier 2 and Tier 3 have no messaging-layer DR at all.** They are covered by a business re-drive procedure and an accepted-loss decision respectively. Both are honest positions; both are single points of failure if the procedure rots or the acceptance is forgotten.
- **DC2 capacity.** Sized for sustained operation [ILLUSTRATIVE] at a fraction of DC1 peak. A prolonged event makes the DR site the new incident.
- **The cluster full repositories.** Two, in different failure domains — but they are still two queue managers that must be started in the right order, and there are only two.
- **Key people.** A small number of MQ subject-matter experts who hold undocumented knowledge — the NFS-maintenance outage requirement for multi-instance ✅, the RDQM split-brain resolution ✅, the appliance failback procedure ✅, the keystore passwords. This is the single point of failure that no amount of architecture removes, and the one that §9's evidence requirements are designed to erode.

---
## 12. The Claims Audit

**Legend.** ✅ = verified this pass against a primary IBM source (or, where noted, a clearly attributed IBM-published mirror of the IBM documentation), with the version gate and date recorded. ⚠ = flagged: partially verified, vendor-blog-sourced rather than documentation-sourced, version-gated and not re-confirmed at the latest level, or historical and not confirmed as current behaviour. ⚠-knowledge = well-documented industry or professional knowledge not re-verified live this pass. ❌ = rejected (no claim in this guide was rejected; any that had been would be listed here with the reason).

**Source-quality note on the mirror.** `www.ibm.com/docs/*` returned an error to this pass's fetcher on every attempt, so the IBM Knowledge Center topics below were read through **setgetweb.com**, a public mirror of the IBM MQ Knowledge Center that reproduces the IBM topic text and topic identifiers (e.g. topic `q018440_` = *Types of logging*). IBM Support pages on `ibm.com/support/pages/*` and IBM-published community articles on `community.ibm.com` were read directly. This is a source-quality caveat, not a content caveat: the topic identifiers and text in the mirror correspond to the IBM documentation, but a reader relying on any of these claims for a production decision should confirm them against `www.ibm.com/docs` at their own release level.

### 12.1 The Verification Targets (a)–(j)

| # | Claim | Verdict | Version gate | Source | Date |
|---|---|---|---|---|---|
| **(a)** | **Circular logging enables restart recovery only.** "Use circular logging if all we want is restart recovery, using the log to roll back transactions that were in progress when the system stopped." Ring of log files, never runs out, better performing, easier to administer. | ✅ | All versions | IBM MQ KC, *Types of logging* (topic q018440_); *Choose circular or linear logging* (topic q018445_) | Read Sept 2026 |
| **(a)** | **Linear logging enables restart recovery *and* media recovery.** "Linear logging (but not circular) enables damaged objects to be recovered." | ✅ | All versions | IBM MQ KC, *Choose circular or linear logging* (q018445_) | Read Sept 2026 |
| **(a)** | **Media recovery is not available with circular logging and damaged objects cannot be recovered; linear logging is recommended for production.** "The default for IBM MQ is Circular logging. Media recovery is not available for Circular logging and you cannot recover the damaged objects. Therefore, Linear logging is recommended for production systems." | ✅ | All versions; page modified 13 Mar 2020 | IBM Support, *IBM MQ — Linear and Circular logging*, document 5737737 | Modified 13 Mar 2020 |
| **(a)** | **What you give up with linear logging:** administrative effort (media images, archive/delete extents) and performance ("linear logging has to allocate new log extents and format them"); also the "never run out of log files" property is lost — "The frequency of media images determines when linear log files can be reused, and is a major factor in how much disk space must be available." | ✅ | All versions | IBM MQ KC, *Choose circular or linear logging* (q018445_); *Types of logging* (q018440_) | Read Sept 2026 |
| **(a)** | **Neither mode protects against a corrupted or deleted log, or messages/queues deleted by applications or the administrator.** | ✅ | All versions | IBM MQ KC, *Choose circular or linear logging* (q018445_) | Read Sept 2026 |
| **(a)** | **Both modes protect against power loss and communications failure.** | ✅ | All versions | IBM MQ KC, *Choose circular or linear logging* (q018445_) | Read Sept 2026 |
| **(a)** | **Log type can be changed after creation with `migmqlog`; before 9.1 it could not be changed.** | ✅ | From IBM MQ **9.1.0** | IBM Support doc 5737737; IBM MQ KC, *Types of logging* (q018440_) | Modified 13 Mar 2020 |
| **(a)** | **Automatic versus manual log management via `LogManagement` in the `Log` stanza of qm.ini.** | ✅ | From IBM MQ **9.1.0** | IBM Support doc 5737737; IBM MQ KC, *Types of logging* (q018440_) | Modified 13 Mar 2020 |
| **(a)** | **Exhausting secondary log files returns `MQRC_RESOURCE_PROBLEM` and long-running transactions are considered for asynchronous rollback.** | ✅ | All versions (linear logging) | IBM MQ KC, *Types of logging* (q018440_) | Read Sept 2026 |
| **(b)** | **Multi-instance design:** instances of the same queue manager on different servers; one active, one standby; standby takes over automatically if the active fails. | ✅ | From **WebSphere MQ 7.0.1** | IBM MQ KC, *Multi-instance queue managers* (q018140_) | Read Sept 2026 |
| **(b)** | **Version floor:** "We can create a multi-instance queue manager on any non-z/OS platform supported by IBM WebSphere MQ Version 7.0.1 and later." | ✅ | WebSphere MQ **7.0.1** | IBM MQ KC, *Multi-instance queue managers* (q018140_) | Read Sept 2026 |
| **(b)** | **Creation method:** one queue manager created with `crtmqm` on shared network storage; the second server uses `addmqinf` to reference the same data and logs. | ✅ | All multi-instance versions | IBM MQ KC, *Multi-instance queue managers* (q018140_) | Read Sept 2026 |
| **(b)** | **Failover mechanism is a lock, not a vote:** "The first instance to acquire read/write locks to the queue manager data becomes the active instance." The standby detects failure, takes over the data and logs, and accepts reconnections from clients and channels. | ✅ | All multi-instance versions | IBM MQ KC, *Multi-instance queue managers* (q018140_) | Read Sept 2026 |
| **(b)** | **Instance limit is two:** "There is a limit of a total of two instances. We cannot have two standby instances and one active instance." | ✅ | All multi-instance versions | IBM MQ KC, *Multi-instance queue managers* (q018140_) | Read Sept 2026 |
| **(b)** | **Documented failover triggers** include failure of the server, failure of connectivity between the active server and the file system, and unresponsiveness of queue manager processes detected by IBM MQ. | ✅ | All multi-instance versions | IBM MQ KC, *Multi-instance queue managers* (q018140_) | Read Sept 2026 |
| **(b)** | **THE STORAGE CAVEAT:** "We must stop all multi-instance queue manager instances that are running in the environment before we can perform maintenance on the NFS drive. Make sure that we have queue manager configuration backups to recover, in the event of an NFS failure." | ✅ | All multi-instance versions | IBM MQ KC, *Multi-instance queue managers* (q018140_) | Read Sept 2026 |
| **(b)** | **Storage requirement:** "a high performance shared network file system (NFS) that manages locks correctly and provides protection against media and file server failure." The specific **NFS v4** framing ("that manages locks correctly") appears verbatim in a partner implementation paper, not in the KC sentence above — treat "v4" as ⚠. | ✅ / ⚠ (v4 detail) | All multi-instance versions | IBM MQ KC, *Multi-instance queue managers* (q018140_); **⚠** W3Partnership, *Creating an IBM MQ H/A Cluster using a Multi-Instance Queue Manager* (partner paper) | Read Sept 2026 |
| **(b)** | **Do not combine multi-instance with an HA cluster.** | ✅ | All multi-instance versions | IBM MQ KC, *High availability configurations* (q017820_) | Read Sept 2026 |
| **(b)** | **IBM's stated limitations:** "Highly available, high performance networked storage required"; "More complex network configuration because queue manager changes IP address when it fails over." | ✅ | All multi-instance versions | IBM MQ KC, *High availability configurations* (q017820_) | Read Sept 2026 |
| **(b)** | **Automatic client reconnect is not supported by IBM MQ classes for Java.** | ✅ (as documented at 9.2) | Documented in the 9.2 KC topic | IBM MQ KC, *Multi-instance queue managers* (q018140_) | Read Sept 2026 |
| **(c)** | **RDQM HA architecture:** three servers in an HA group, each with an instance; one running queue manager synchronously replicates data to the other two; the three share a floating IP; only one instance runs at a time. Primary/secondary roles; three nodes used to reduce split-brain; nodes that can communicate with at least one other node form a quorum; the queue manager can only run on a node that has quorum. | ✅ | IBM MQ Advanced, Linux | IBM MQ KC, *RDQM high availability* (q130280_) | Read Sept 2026 |
| **(c)** | **RDQM quorum consequence:** if two nodes fail, the queue manager cannot run on the remaining node; if a single node loses connectivity, the queue manager cannot run on it; if all nodes lose connectivity, it cannot run anywhere. | ✅ | IBM MQ Advanced, Linux | IBM MQ KC, *RDQM high availability* (q130280_) | Read Sept 2026 |
| **(c)** | **Underlying technology — VERIFIED as asked:** "The group configuration of the three nodes is handled by **Pacemaker**. The replication between the three nodes is handled by **DRBD**." Each RDQM gets a DRBD resource (`/etc/drbd.d/<qm>.res`) and a set of Pacemaker resources. | ✅ | IBM MQ Advanced, Linux | IBM MQ KC, *RDQM high availability* (q130280_); IBM-published `ibm-messaging/mq-rdqm` *RDQM Architecture* | Read Sept 2026 |
| **(c)** | **DRBD and Pacemaker RPMs are shipped on the IBM MQ media and must not be replaced with your own versions;** packages are signed with the LINBIT GPG key. | ✅ | IBM MQ Advanced, Linux | IBM MQ KC, *Installing RDQM* (topic q130560_) | Read Sept 2026 |
| **(c)** | **Supported platform (9.2 docs):** "RDQM is available on RHEL 7.6 and onwards on x86-64, and on RHEL 8.2 and onwards on x86-64." | ✅ | IBM MQ 9.2 documentation | IBM MQ KC, *Installing RDQM* (q130560_) | Read Sept 2026 |
| **(c)** | **Supported platform (9.4 docs):** "RDQM is available on x86-64 for RHEL 8 (8.8 or later) and RHEL 9 (9.2 or later)." **The supported footprint narrowed between 9.2 and 9.4 — RHEL 7 support is gone.** | ✅ | IBM MQ 9.4 documentation | IBM MQ 9.4.x documentation, *Installing RDQM* (via IBM docs search result) | Read Sept 2026 |
| **(c)** | **RDQM DR:** primary at one site, secondary on a recovery node at a different site; promotion is **manual**; "We cannot start a queue manager while it is in the secondary role"; "We cannot add disaster recovery to an existing queue manager"; "A queue manager cannot be configured with both RDQM disaster recovery and RDQM high availability." | ✅ | IBM MQ Advanced, Linux | IBM MQ KC, *RDQM disaster recovery* (q131670_) | Read Sept 2026 |
| **(c)** | **RDQM DR replication modes:** **synchronous or asynchronous** is selectable. Asynchronous means PUT/GET return before replication, "some messaging data might be lost," but the secondary is in a consistent state and can start immediately. | ✅ | IBM MQ Advanced, Linux | IBM MQ KC, *RDQM disaster recovery* (q131670_) | Read Sept 2026 |
| **(c)** | **RDQM DR replication / synchronization / snapshot distinction:** on network loss, changes are tracked and a later *synchronization* brings the secondary up to date; during synchronization the secondary data is inconsistent and a **snapshot** is taken; if failure occurs during synchronization, the secondary reverts to the snapshot and updates since the original network failure are lost. | ✅ | IBM MQ Advanced, Linux | IBM MQ KC, *RDQM disaster recovery* (q131670_) | Read Sept 2026 |
| **(c)** | **RDQM DR split brain:** promotion requires user action, and the person promoting must ensure the former primary is stopped; a managed failover avoids a partitioned state; if the replication link is working, a secondary cannot be promoted while the primary still runs — "the command fails." | ✅ | IBM MQ Advanced, Linux | IBM MQ KC, *RDQM disaster recovery* (q131670_) | Read Sept 2026 |
| **(c)** | **DR/HA RDQM:** an RDQM running on an HA group at one site that can fail over to an HA group at another site. **Inter-site replication is ALWAYS asynchronous.** | ✅ | **New in IBM MQ 9.2.0** (listed under Advanced-only new features) | IBM MQ KC, *RDQM disaster recovery and high availability* (q133982_); IBM MQ KC, *What's new in Version 9.2.0* (q134120_) | 9.2.0 GA **23 July 2020** (per IBM fix list) |
| **(c)** | **You cannot upgrade an existing RDQM into a DR/HA RDQM** — it must be created as one (workaround: back up, delete, recreate, restore). | ✅ | IBM MQ 9.2.0+ | IBM MQ KC, *RDQM disaster recovery and high availability* (q133982_) | Read Sept 2026 |
| **(c)** | **RDQM node-level upgrade constraint:** "you must upgrade all nodes in a sequence. **Do not try to operate with the nodes at different levels.**" | ✅ | IBM MQ 9.4 documentation | IBM MQ 9.4.x docs, *Migrating replicated data queue managers* (via IBM docs search result) | Read Sept 2026 |
| **(c)** | **The IBM MQ Console does not support RDQMs;** MQ Explorer shows them but without RDQM-specific information. | ✅ | IBM MQ Advanced, Linux | IBM MQ KC, *RDQM high availability* (q130280_) | Read Sept 2026 |
| **(c)** | **RDQM HA queue managers retain their state across failover:** running queue managers are started, stopped ones remain stopped. | ✅ | IBM MQ Advanced, Linux | IBM MQ KC, *RDQM high availability* (q130280_) | Read Sept 2026 |
| **(d)** | **Native HA node count and design — VERIFIED as asked (it is 3):** "A Native HA configuration consists of three nodes, each with an instance of the queue manager. One instance is the active queue manager, processing messages and writing to its log. Whenever the log is written, the active queue manager replicates the data to the other two instances, known as replicas." Each replica writes to its own recovery log, acknowledges, then updates its own queue data. | ✅ | Documented 9.3.x and 9.4.x; container deployments (9.3), containers **and Linux** (9.4) | IBM MQ KC, *Native HA* (9.4.x and 9.3.x topics, via IBM docs search results); IBM community article (IBM employee) quoting the same text | Read Sept 2026 |
| **(d)** | **Replication mechanism:** Native HA uses a **replicated log**, described as "essentially a linear log, with automatic log management and automatic media images enabled." | ⚠ | Documented behaviour, sourced from an IBM-published community article quoting the IBM *Types of logging* topic rather than read directly from the doc this pass | IBM community article by an IBM engineer, *Design and Architect IBM MQ Native HA…*, 17 Jun 2025, citing IBM MQ *Types of logging* | 17 Jun 2025 |
| **(d)** | **Quorum:** at least two of three instances must be available. | ⚠ | Same source-quality caveat | Same IBM community article, 17 Jun 2025 | 17 Jun 2025 |
| **(d)** | **Consensus:** "Native HA implements a consensus algorithm based closely on raft (elections are based on log content, strong leader model, etc). Native HA uses its own proprietary wire protocol." | ⚠ (no "raft" string located in the KC topic text this pass) | Same source-quality caveat | Same IBM community article, 17 Jun 2025 | 17 Jun 2025 |
| **(d)** | **Storage:** Native HA "always uses block storage"; "When using Native HA (including CRR) you should use **simple RWO block storage that is not replicated** — Native HA is responsible for the replication, not the storage layer." | ⚠ (block-storage/RWO detail from the community article; consistent with IBM's deployment guidance) | Applies to Native HA and CRR | Same IBM community article, 17 Jun 2025 | 17 Jun 2025 |
| **(d)** | **Platform availability:** Native HA is available "on container deployments of IBM MQ" (9.3 documentation) and "on container deployments of IBM MQ **and on Linux**" (9.4 documentation). | ✅ | 9.3 vs 9.4 documentation | IBM MQ KC *Native HA* topics, 9.3.x and 9.4.x (via IBM docs search results) | Read Sept 2026 |
| **(d)** | **What Native HA does NOT cover:** base Native HA is a single-region/single-cluster HA mechanism and provides no cross-site DR on its own; that requires CRR or IRR. | ✅ (inference from the topology descriptions plus the separate availability of CRR/IRR) | 9.4.2+ / 10.0 | IBM MQ KC plus IBM community release articles | 2025–2026 |
| **(e)** | **Cross-region/cross-site replication exists — NAME and VERSION VERIFIED:** **Native HA Cross-Region Replication (CRR)**, introduced in **IBM MQ 9.4.2**, released **27 February 2025**. It extends Native HA "to permit **asynchronous** replication of log data supporting **unplanned failover and planned switchover**." | ✅ | Introduced **IBM MQ 9.4.2** | IBM community article by an IBM engineer, 28 Feb 2025 | 28 Feb 2025 |
| **(e)** | **CRR recovery group is itself three replicated instances** and is therefore highly available; deployment is via the MQ Operator or sample Helm charts. | ✅ | IBM MQ 9.4.2+ | Same IBM community article, 28 Feb 2025 | 28 Feb 2025 |
| **(e)** | **IBM MQ 10.0.0 adds In-Region Replication (IRR) alongside CRR:** "Native HA is a high availability solution requiring 3 nodes within a region"; "Native HA IRR is a 2 node disaster recovery solution… but **manual intervention** to switch to the Recovery site"; "Native HA CRR is a high availability and disaster recovery solution, requiring **6 nodes, 3 in each region**." | ✅ | **IBM MQ 10.0.0** | IBM community release article by an IBM employee, 16 Jun 2026; IBM MQ 10.0.x documentation topic *Comparison of Native HA CRR and Native HA IRR solutions* (referenced) | 16 Jun 2026 |
| **(e)** | **CRR is not limited to disasters:** it supports migrating a queue manager between clusters or cloud providers with a planned switchover and no service interruption to message data. | ✅ | IBM MQ 9.4.2+ | Same IBM community article, 28 Feb 2025 | 28 Feb 2025 |
| **(e)** | **IBM MQ 10.0 GA dates:** announced (announcement article published) 21 April 2026; generally available 16 June 2026, and 19 June 2026 for MQ for z/OS. **⚠ Note a date discrepancy:** an IBM announcement-page summary in this pass's search results gave "14 April 2026" for the announcement, while the announcement article itself is dated 21 April 2026 and community coverage is dated 22 April 2026. Use 21 April 2026 for the published article and treat 14 April 2026 as unverified. | ✅ / ⚠ | IBM MQ 10.0 | IBM announcement *Introducing IBM MQ 10.0*, published 21 Apr 2026; IBM community release article, 16 Jun 2026; Capitalware blog, 22 Apr 2026 | Apr–Jun 2026 |
| **(e)** | **IBM MQ 9.2.0 GA date:** 23 July 2020 (per IBM's fix list for 9.2 LTS). Extended support began 30 Sep 2025 and runs to 30 Sep 2029. | ✅ | IBM MQ 9.2 | IBM Support, *Fix list for IBM MQ Version 9.2 LTS* | Read Sept 2026 |
| **(f)** | **MQ Appliance HA arrangement — VERIFIED as asked (it is an HA pair):** the appliance has built-in HA and DR; multiple appliances run queue managers resiliently; queue manager data is replicated between appliances; **"Fail over is automatic for HA and administratively orchestrated for DR."** Both appliances in an HA pair are typically in the same data centre. | ✅ | MQ Appliance firmware | IBM-published MQ Appliance proof-of-technology lab (ibm-cloudintegration.github.io, IBM); IBM community article by an IBM engineer, 16 Feb 2023 | 16 Feb 2023 / lab |
| **(f)** | **Appliance DR design:** one network connection is required for DR; appliances tolerate latency between distant sites, unlike HA appliances which are designed for automatic takeover with no data loss in close proximity. | ✅ | MQ Appliance | Same IBM-published appliance lab | Lab doc |
| **(f)** | **Appliance DR between HA groups was added in 9.3.2** — an HA group of appliances at both ends of the DR link, as per the equivalent RDQM DR support on Linux; before 9.3.2 an HA group could only have a single appliance at a remote site (now called the *legacy* solution). | ✅ | **New in MQ Appliance 9.3.2** | IBM community article by an IBM engineer, 16 Feb 2023 | 16 Feb 2023 |
| **(f)** | **Appliance client-routing constraint:** queue managers can have a floating IP **within** each HA group, but "IP addresses cannot float across the DR link from one HA group to another" — clients use a CCDT, a connection name list, DNS, a global load balancer, or equivalent routing. | ✅ | MQ Appliance 9.3.2+ | Same IBM community article, 16 Feb 2023 | 16 Feb 2023 |
| **(f)** | **Appliance hardware lifecycle:** M2004 announced, available 16 July 2026, minimum firmware MQ 10.0 LTS; M2003 supported to 30 Sep 2031; M2002 to 30 Sep 2027. | ✅ | MQ Appliance | IBM community release article by an IBM employee, 16 Jun 2026 | 16 Jun 2026 |
| **(g)** | **NON-PERSISTENT MESSAGES DO NOT SURVIVE A FAILOVER — STATED PLAINLY:** "Persistent messages are written to logs and queue data files. If a queue manager is restarted after a failure, it recovers these persistent messages as necessary from the logged data. **Messages that are not persistent are discarded if a queue manager stops, whether the stoppage is as a result of an operator command or because of the failure of some part of your system.**" A failover is a queue manager stopping and starting elsewhere, so non-persistent messages are lost. | ✅ | All versions | IBM MQ documentation, *Message persistence* (via IBM docs search result, 9.4.x) | Read Sept 2026 |
| **(g)** | **Corroboration:** "Nonpersistent messages are discarded; they do not survive when WebSphere MQ stops." And: non-persistent messages "cannot be recovered" by media recovery. | ✅ | All versions | IBM *System Administration Guide* (amqzag0335), Using the log for recovery | Doc © IBM 1994, 2002 |
| **(h)** | **A data-directory backup DOES capture message data — but only from a stopped queue manager.** "ensure that the queue manager is not running. If you try to take a backup of a running queue manager, the backup might not be consistent because of updates in progress when the files are copied." Stop with `endmqm -w`; copy all data and log directories including subdirectories, the log control file, and configuration files; preserve ownership; for circular logging back up data and logs together; for linear logging the same, and restoring only data files is possible only if a complete sequence of log files is available. | ✅ | All versions | IBM MQ KC, *Backing up queue manager data* (q018590_) | Read Sept 2026 |
| **(h)** | **A CONFIGURATION backup captures NO message data at all.** `dmpmqcfg` dumps the configuration of a queue manager (object definitions and authorities); `DMPMQMCFG` on IBM i dumps "the configuration objects and authorities." Neither captures queue contents. | ✅ | Current | IBM MQ 9.4.x documentation, *Backing up queue manager configuration* and *dmpmqcfg (dump queue manager configuration)* (via IBM docs search results) | Read Sept 2026 |
| **(i)** | **Object-definition export tooling and its CURRENT NAME — VERIFIED:** `dmpmqcfg` on AIX, Linux, and Windows; `DMPMQMCFG` on IBM i. Used to save configuration when moving a queue manager between operating systems, with IBM's explicit warning that "some manual modification of the definitions" is required. | ✅ | Current | IBM MQ 9.4.x documentation, *dmpmqcfg (dump queue manager configuration)*; *Backing up queue manager configuration* (via IBM docs search results) | Read Sept 2026 |
| **(i)** | **`saveqmgr` was the predecessor**, historically distributed as an IBM SupportPac, and appears in older runbooks and scripts. | ⚠ | Historical | Community source only this pass (a Stack Overflow question describing `saveqmgr.exe` from a SupportPac) — not confirmed against a current IBM documentation page | Read Sept 2026 |
| **(j)** | **Certificate/keystore:** on AIX, Linux, and Windows the key repository (a GSKit CMS keystore) "is secured with a password as it contains sensitive information"; IBM documents options for securely storing that password, including encrypted key-repository passwords. A key repository can be used as a **truststore** for validating certificates signed by a common set of CAs on outgoing TLS connections. TLS is negotiated by GSKit using the channel's `SSLCIPH`. | ✅ | Current, AIX/Linux/Windows | IBM MQ documentation, *Setting up a key repository on AIX, Linux, and Windows*; *Creating a key repository for use as a TLS truststore*; *Encrypting key repository passwords* (via IBM docs search results) | Read Sept 2026 |
| **(j)** | **z/OS uses RACF key rings rather than GSKit `.kdb` files.** | ⚠-knowledge | z/OS | Not re-verified against an IBM page this pass | — |
| **(j)** | **CCDT recovery consideration:** on the appliance, floating IPs cannot cross the DR link, and "Applications can use MQ connectivity options, such as a CCDT or a connection name list to try to connect to each site in turn, or they can be routed to the correct site by using a global load balancer, DNS entry, or an equivalent network routing capability." CCDT/connection-name-list recovery is therefore an explicit DR design element. | ✅ | MQ Appliance 9.3.2+; general MQ client behaviour | IBM community article by an IBM engineer, 16 Feb 2023 | 16 Feb 2023 |
| **(j)** | **CCDT staleness as a DR failure mode**, and a CCDT being distributed out of band with no automatic verification — the operational claim, not a product fact. | ⚠-knowledge | — | Practitioner judgement; consistent with the appliance/CCDT routing dependency above | — |

### 12.2 Additional Claims Made in This Guide

| Claim | Verdict | Version gate | Source |
|---|---|---|---|
| Queue manager clusters do **not** provide HA of all message data, nor automatic failure detection or automatic failover — HA clusters or multi-instance are needed for those. | ✅ | WebSphere MQ 7.1+ for cluster error recovery | IBM MQ KC, *High availability configurations* (q017820_) |
| Cluster repository: typically two full repositories per cluster, the rest partial; stored on `SYSTEM.CLUSTER.REPOSITORY.QUEUE`; refresh requests via `SYSTEM.CLUSTER.COMMAND.QUEUE`. | ✅ | All cluster versions | W3Partnership partner paper reproducing IBM KC cluster definitions; IBM MQ KC, *High availability configurations* (q017820_) (for the cluster-vs-HA distinction) — **partner-paper sourced for the queue names; treat the queue-name detail as ⚠ if it is load-bearing for you** |
| "From IBM WebSphere MQ Version 7.1 onwards, cluster error recovery reruns operations that caused problems until the problems are resolved." | ✅ | WebSphere MQ **7.1+** | IBM MQ KC, *High availability configurations* (q017820_) |
| Media recovery definition; the media recovery record; media images recorded automatically at shutdown for process objects, non-local queues, and empty local queues; `rcdmqimg` records a media image manually. | ✅ | All versions (linear logging) | IBM *System Administration Guide* (amqzag0335), Recovering damaged objects / media recovery |
| `rcrmqobj` can only be used with linear logging; it re-creates from the latest image plus all applicable log events since; it is used on a running queue manager; non-persistent messages cannot be recovered this way. | ✅ | All versions (linear logging) | IBM *System Administration Guide* (amqzag0335); IBM MQ documentation *rcrmqobj (re-create object)* |
| Automatic recovery from damaged objects occurs at startup where possible; a damaged queue manager object prevents startup unless recoverable; without media recovery the options are restore a backup or delete the queue manager; damaged local queues holding uncommitted messages on a non-media-recovery queue manager are marked damaged and their uncommitted messages ignored (AMQ7472). | ✅ | All versions | IBM *System Administration Guide* (amqzag0335) |
| A damaged object detected after startup triggers a **pre-emptive shutdown**; a missing log file during media recovery raises **AMQ6767** and recovery fails. | ✅ | All versions | IBM *System Administration Guide* (amqzag0335) |
| Media recovery LSN failing to advance was an APAR (IT30479), fixed in 9.1 CD 9.1.3 / 9.1 LTS 9.1.0.4. | ✅ | Fixed in **9.1 CD 9.1.3** and **9.1 LTS 9.1.0.4** | IBM Support doc 5737737 |
| Power-loss recovery behaviour: on restart MQ restores queues to their committed state at the time of the failure, so no persistent messages are lost. | ✅ | All versions | IBM *System Administration Guide* (amqzag0335) |
| Automatic client reconnection is **inline**: "The connection is automatically restored at any point in the client application program, and the handles to open objects are all restored." | ✅ | Documented at 9.3 | IBM MQ KC, *Automatic client reconnection* (9.3.x, via IBM docs search result) |
| "From IBM MQ 9.2.0 a reconnect request can be sent to a client containing a hint of which queue manager the client should reconnect to." | ✅ | From **IBM MQ 9.2.0** | IBM MQ KC, *How automatic balancing uses automatic reconnect* (9.4.x, via IBM docs search result) |
| The readiness of the *consuming application's* recovery is a separate problem from the queue manager's recovery, and it is frequently un-owned: the queue manager recovers the message data, not the consumers that must reconnect, reconcile, and re-drive. | ⚠-knowledge | — | Architectural reasoning, consistent with the cited sources |
| **Switchover** — moving the active instance/role deliberately rather than by failure — is a supported operation for multi-instance queue managers (IBM MQ KC, *Multi-instance queue managers*, q018140_: stopping the active instance using the switchover option transfers control to the standby). | ✅ (capability) / ⚠ (syntax) | Multi-instance versions | IBM MQ KC, *Multi-instance queue managers* (q018140_). **The exact `endmqm`/`strmqm` switchover syntax is version-dependent and was not read from IBM documentation in this pass — do not take command syntax from this guide.** |
| IBM MQ does not offer point-in-time recovery to a chosen timestamp, and log archive is a media-recovery artifact rather than a replay-to-time stream. | ✅ (negative claim, supported by the absence of such a feature across the cited logging, media-recovery, and backup topics, all of which describe current-state reconstruction) | All versions | IBM MQ KC *Types of logging* (q018440_), *Backing up queue manager data* (q018590_), *System Administration Guide* |
| DORA: Regulation (EU) 2022/2554; entered into force 16 January 2023; applicable from **17 January 2025**. | ✅ | — | ESMA; EIOPA; EBA; LSEG summaries |
| MAS Business Continuity Management Guidelines were revised and issued on **6 June 2022**. | ✅ | — | MAS-related legal commentary (Allen & Gledhill, 22 Jun 2022); MAS publication |
| MAS Technology Risk Management Guidelines exist as an MAS guidance publication covering technology risk, reliability, resiliency, and recoverability. | ✅ (publication); clause detail ⚠ | — | MAS materials; not clause-verified this pass |
| A "MAS Technology Risk Management notice" specifies less than four hours of unscheduled downtime for critical systems. | ⚠ | — | **News-media sourced (The Straits Times). Not verified against the MAS notice text this pass. Do not rely on the four-hour figure from this guide.** |

### 12.3 Rejected

**No claim in this guide was rejected during this pass.** Two candidate claims were *downgraded* rather than rejected, and are recorded here so that a reader does not mistake the downgrade for verification:

- **"Native HA uses the Raft consensus protocol."** ❌ as stated, ⚠ as "a consensus algorithm based closely on raft." No occurrence of the bare string "Raft" was located in the IBM Knowledge Center *Native HA* topic text retrieved in this pass. The raft-based description is IBM-published but community-article-sourced. Do not attribute "Raft" to the product documentation.
- **"Multi-instance queue managers support more than one standby."** ❌ as stated. The IBM MQ Knowledge Center topic states a total limit of two instances and explicitly rules out two standbys plus one active, notwithstanding a general KC sentence elsewhere that speaks of "standby instances" plural in the abstract. Use the two-instance limit.

---

## 13. What Could Not Be Verified

Listed explicitly, because an unlisted gap becomes an assumed fact.

1. **The exact IBM MQ release in which Native HA was first introduced.** Native HA is documented for container deployments in the 9.2 documentation set and the IBM MQ 9.4 documentation extends it to Linux. The precise release that first shipped it — and the precise release that first shipped it on Linux — could not be confirmed from a primary IBM source in this pass. The guide therefore states only "documented for containers from 9.2; Linux added by the 9.4 documentation," and flags the introduction release as ⚠.
2. **The Raft attribution for Native HA's consensus algorithm.** See §12.3. The description is IBM-published but community-article-sourced, and no "Raft" string was found in the Knowledge Center topic text retrieved. Treat as ⚠.
3. **The exact node count and quorum rule as written in the product documentation.** The three-node design and the "log written → replicated to the other two instances" behaviour are ✅. The "minimum two instances to maintain quorum" statement is ⚠ (community article). The guide states the quorum rule as documented behaviour with the source-quality caveat rather than as a documentation quote.
4. **CRR's and IRR's platform availability in full.** CRR deployment is described via the MQ Operator and sample Helm charts, and Native HA is available on containers and (per 9.4 documentation) on Linux. Whether CRR and IRR are supported on every Native HA-capable platform, and on which Linux levels, was not verified per-platform. **Verify per-platform before designing.**
5. **CRR's and IRR's operational limits and failure semantics in detail** — the exact failover sequence, the maximum supported network latency between regions, the behaviour when the recovery group is itself degraded, and the documented RPO characteristics. This guide states the topology and the asynchronous-replication property and deliberately stops there.
6. **`RESET CHANNEL` semantics and channel resynchronisation behaviour across failover in current releases.** The existence of in-doubt channels and the operator-intervention route is well-established; the precise behaviour is release-, channel-type-, and platform-dependent, and no primary source was read in full this pass. **Do not take the in-doubt handling procedure from this guide** — read the *Channel resynchronization* documentation for your release.
7. **IBM MQ classes for Java automatic reconnection: whether the limitation stated in the 9.2 documentation still holds at 9.4/10.0.** The guide reports it as documented at the level cited and does not assert it as current.
8. **The z/OS DR model in detail.** z/OS uses a fundamentally different recovery and availability architecture — sysplex, Coupling Facility, queue-sharing groups, RACF key rings. This guide deliberately scopes to distributed platforms plus the appliance and flags its z/OS references as ⚠-knowledge. **A hybrid estate needs a z/OS-specific DR guide, and this is not it.**
9. **`saveqmgr` and its SupportPac lineage.** ⚠ (community-sourced). The current tooling names (`dmpmqcfg`, `DMPMQMCFG`) are ✅.
10. **The precise contents of the IBM MQ 10.0.x topic *Comparison of Native HA CRR and Native HA IRR solutions*.** The topic is referenced by IBM's release article and was not retrieved; the node counts and the manual-intervention property for IRR come from the IBM release article, not from the comparison topic itself.
11. **The IBM MQ Message Persistence FAQ (IBM Support document 346659)** — the answers to its four questions are behind an IBM login, so only its abstract (listing the questions) was readable. The persistence claims in §2.1 come from the *Message persistence* product documentation and the *System Administration Guide*, both ✅.
12. **Regime-level detail for BCBS, FFIEC, and the PRA/Bank of England operational resilience requirements**, and the clause-level text of MAS TRM. Named as ⚠-knowledge and ⚠ respectively; not verified this pass. The **four-hour** unscheduled-downtime figure attributed to a MAS TRM notice is **news-media sourced** and is not verified.
13. **The IBM MQ 10.0 announcement date.** Two dates appeared: an announcement-page summary of 14 April 2026, and a published article dated 21 April 2026 (with community coverage on 22 April 2026). The guide uses 21 April 2026 and flags the discrepancy.
14. **Any per-release confirmation that the specific multi-instance, RDQM, and appliance behaviours described here are unchanged at IBM MQ 10.0.** Most of the cited topics were read at the 9.2/9.3/9.4 levels. **Version-gated behaviour must be re-checked at the release you deploy.**
15. **The direct-fetch of `www.ibm.com/docs` itself failed on every attempt in this pass.** All Knowledge Center content was read via a public mirror carrying the IBM topic identifiers. Where a claim rests on that mirror, the topic identifier is given so the reader can confirm it at `www.ibm.com/docs` directly.

---

## 14. Glossary

- **Active log** — the maximum amount of log space that might be referenced by restart recovery; the log files that must be online. Does not include the space needed for media recovery. §2.3.
- **Asynchronous replication** — replication in which a PUT or GET returns to the application before the update has been replicated. Consistent recovery point, but possible message loss. The default and sometimes mandatory mode for cross-site MQ replication. §5.1, §5.2, §5.3.
- **Backout / backout requeue queue** — the queue to which a message is moved after repeated failed processing (backout threshold); the mechanism that turns a poison message into a visible one. Must exist at the DR site; include it in the object export. §6.3.
- **CCDT (client channel definition table)** — the client-side artifact carrying channel and connection definitions to IBM MQ clients; a DR artifact whose staleness is a classic failure mode. §6.4.
- **CHLAUTH (channel authentication rules)** — MQSC-defined channel access rules; exportable, but must be re-verified against DR-site addressing. §8.5.
- **Circular logging** — ring-of-files logging giving restart recovery only; no media recovery, no damaged-object recovery; better performance and simpler administration; the IBM MQ default. §2.2.
- **`CLNTCONN` channel** — the channel definition type used to build a CCDT. §6.4.
- **DRBD** — the distributed replicated block device technology that provides the data replication beneath RDQM; shipped with IBM MQ and not to be replaced with another version. §4.2.
- **`dmpmqcfg`** — the current queue manager configuration export command on AIX, Linux, and Windows; emits MQSC. **`DMPMQMCFG`** is the IBM i equivalent. Captures object definitions and authorities, **never message data**. §7.2.
- **DEFPSIST / `MQPER`** — the default-persistence queue attribute and the per-message persistence flag; the settings that decide whether a message survives a failover at all. §2.1.
- **DR / disaster recovery** — resilience against the loss of a site, region, or platform. Distinct from HA. §1.2.
- **DR/HA RDQM** — a replicated data queue manager running on an HA group at one site that can fail over to an HA group at another site; inter-site replication is always asynchronous. New in IBM MQ 9.2.0. §5.2.
- **DR RDQM** — a replicated data queue manager in a primary/secondary pair across two sites, with manual promotion and selectable synchronous or asynchronous replication. §5.1.
- **HA / high availability** — resilience against process, node, or storage-path failure within a site. Distinct from DR. §1.2.
- **In-doubt unit of work** — a unit of work whose outcome is unresolved between channel partners, typically after a break; resolved by channel resynchronisation or operator intervention. §6.3.
- **GSKit** — IBM's Global Security Kit; the cryptographic engine that negotiates TLS on MQ channels according to the channel's `SSLCIPH`, and that reads the key repository. §8.4.
- **Impact tolerance** — the maximum disruption to an important business service before intolerable harm; the business-level construct the messaging layer's RTO/RPO must be mapped onto. §10.3; framework owned by [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md).
- **In-region replication (IRR)** — a two-node Native HA disaster-recovery topology requiring manual intervention to switch to the recovery site; new in IBM MQ 10.0.0. §5.3.
- **Key repository** — the store holding a queue manager's TLS credentials on AIX, Linux, and Windows; conventionally a GSKit CMS keystore (`.kdb`), password-protected; may also serve as a truststore. §8.4.
- **Linear logging** — sequential logging giving restart *and* media recovery; enables media images and damaged-object rebuild; costs administration, performance, and the "never run out of log space" property. §2.2.
- **Log control file** — a file within the queue manager's logs that must not be missed when taking a backup. §7.1.
- **Media image** — a sequence of log records containing an image of an object from which the object can be re-created; recorded automatically at shutdown for some objects, on a schedule, or manually with `rcdmqimg`. §3.1, §3.2.
- **Media recovery** — re-creating an object from its media image plus subsequent log records; available only with linear logging. §3.
- **Media recovery record** — the first log record required to re-create an object; the start of its latest media image; recorded during a checkpoint. §3.1.
- **`migmqlog`** — the command that changes a queue manager's log type between linear and circular; available from IBM MQ 9.1.0. §2.2.
- **MQSC** — the IBM MQ scripting command language; the format of `dmpmqcfg` output and the means by which object definitions are replayed. §7.2.
- **Multi-instance queue manager (MIQM)** — one queue manager whose data and logs live on shared network storage, with an active and a standby instance; the standby takes over by acquiring read/write locks. Two-instance limit; storage is a single point of failure. From WebSphere MQ 7.0.1. §4.1.
- **Native HA** — IBM MQ's three-node, log-replicating high-availability mechanism; active instance plus two replicas; quorum-based; uses non-replicated block storage because MQ performs the replication. §4.3.
- **Native HA CRR (cross-region replication)** — asynchronous replication of Native HA log data from a live group to a recovery group, supporting unplanned failover and planned switchover; introduced in IBM MQ 9.4.2 (27 February 2025). §5.3.
- **NFS** — the network file system that multi-instance queue managers depend on; must manage locks correctly and protect against media and file-server failure; requires all instances to be stopped for maintenance. §4.1.
- **Non-persistent message** — a message discarded when the queue manager stops, for any reason. Cannot be recovered from a media image. The plain answer to "does it survive a failover": no. §2.1.
- **Pacemaker** — the cluster resource manager that controls which node of an RDQM HA group runs the queue manager; supplied with IBM MQ. §4.2.
- **Partitioned / split-brain** — the state in which two instances of a queue manager hold divergent data because both ran; requires human judgement to resolve. §5.1, §4.4.
- **Persistent message** — a message written to logs and queue data files and recovered from the log after a failure. The precondition for any DR guarantee. §2.1.
- **Point-in-time recovery** — recovery to a chosen historical moment. **Not a supported IBM MQ operation on distributed platforms**; the closest approximations are restart recovery (the last committed state) and a stopped-service data-directory backup. §2.5.
- **Primary / secondary (RDQM)** — the node running the queue manager and the nodes holding replicated copies; the queue manager cannot run in the secondary role without promotion. §4.2, §5.1.
- **`rcrmqobj`** — recreate object: rebuilds an object from its media image and subsequent log records; linear logging only; runs against a running queue manager. §3.3.
- **`rcdmqimg`** — record media image: writes an object image to the log, creating a recovery point and allowing older log extents to be released. §3.2.
- **RDQM (replicated data queue manager)** — IBM MQ's Linux-only, IBM MQ Advanced replication family, built on DRBD and Pacemaker, in HA, DR, and DR/HA variants. §4.2, §5.1, §5.2.
- **Replicated log** — the log type used by Native HA: essentially a linear log with automatic log management and automatic media images enabled. §4.3.
- **Repository (cluster)** — the collection of cluster membership and routing information; full repositories hold everything, partial repositories a subset; typically two full repositories per cluster; stored on `SYSTEM.CLUSTER.REPOSITORY.QUEUE`. §8.1.
- **RPO / RTO** — recovery point objective (how much data you may lose) and recovery time objective (how long you may be down). For a messaging layer, express RPO in messages *and* time, and include consumer replay in the RTO. §1.4.
- **RWO (ReadWriteOnce) block storage** — the storage class guidance for Native HA and CRR: simple, non-replicated block storage, because MQ performs the replication. §4.3.
- **Secondary log files** — additional log files formatted and added to the active pool when long transactions prevent log files from being freed; exhausting them causes `MQRC_RESOURCE_PROBLEM` and possible asynchronous rollback. §2.3.
- **Synchronous replication** — replication in which the operation does not complete until the update has been replicated; effectively zero RPO, constrained by inter-site latency. Selectable for RDQM DR only. §5.1.
- **Switchover** — a planned transfer of the active role (as opposed to failover, an unplanned one); supported by multi-instance, RDQM, Native HA CRR, and the appliance. §4.1, §5.3, §11.3.
- **Truststore** — a key repository used to validate certificates signed by a common set of certificate authorities on outgoing TLS connections; a separate renewal cadence from the identity keystore. §8.4.
- **Uniform cluster** — a set of queue managers that applications connect to as a pool, with automatic balancing and reconnection; combines usefully with Native HA and CRR to soften the client-side impact of a failover. §5.7.

---

## 15. Cross-References and Further Reading

**Sibling guides (technology/):** [enterprise_middleware_integration_platform_guide.md](enterprise_middleware_integration_platform_guide.md) (the integration platform landscape, the MQ object model, the mainframe boundary — read this first if the MQ estate itself is unfamiliar), [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) (**the reliability discipline this guide assumes** — delivery semantics, idempotency and dedupe keys, DLQ theory, the transactional outbox, and the broker comparison in which IBM MQ is deliberately absent), [kafka_alternatives_guide.md](kafka_alternatives_guide.md) (broker selection), [legacy_integration_patterns_guide.md](legacy_integration_patterns_guide.md) (the patterns a surviving MQ estate carries), [camel_camelk_research.md](camel_camelk_research.md) (integration routing that must inherit the same recovery guarantees), [axway_transfer_cft_guide.md](axway_transfer_cft_guide.md) and [axway_cft_controlm_integration.md](axway_cft_controlm_integration.md) (managed file transfer riding on MQ — the Tier 2 problem in §11), [ibm_cloud_guide.md](ibm_cloud_guide.md) (IBM's platform estate, MQ on OpenShift and the Cloud Paks — the deployment surface for Native HA and CRR).

**Banking guides (../banking/):** [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) (**owns the resilience framework** — impact tolerances, business-service mapping, the evidence model, third-party and outsourcing expectations; §10 here is the messaging-layer mapping onto it and nothing more), [../banking/payment_rails_guide.md](../banking/payment_rails_guide.md) (the rails the messaging layer underwrites), [../banking/swift_alliance_access_guide.md](../banking/swift_alliance_access_guide.md) and [../banking/swiftnet_fileact_guide.md](../banking/swiftnet_fileact_guide.md) (the SWIFT estate above the MQ layer), [../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md) (the cores behind the MQ interfaces), [../banking/mas_regulations_guidelines_guide.md](../banking/mas_regulations_guidelines_guide.md) (Singapore regulatory context).

**Primary sources consulted this pass.** IBM MQ Knowledge Center topics (read via the setgetweb.com mirror of the IBM MQ Knowledge Center, with IBM topic identifiers given in §12 so each can be confirmed at `www.ibm.com/docs`): *Types of logging* (q018440_); *Choose circular or linear logging* (q018445_); *Logging: Making sure that messages are not lost* (q018410_); *Multi-instance queue managers* (q018140_); *High availability configurations* (q017820_); *RDQM high availability* (q130280_); *RDQM disaster recovery* (q131670_); *RDQM disaster recovery and high availability* (q133982_); *Installing RDQM* (q130560_); *Backing up queue manager data* (q018590_); *Backing up and restoring IBM MQ queue manager data* (q018580_); *What's new in Version 9.2.0* (q134120_) and *What's new and changed in IBM MQ Version 9.2.0* (q113110_). Also the IBM *System Administration Guide* (amqzag0335, Using the log for recovery, Recovering damaged objects, media recovery). IBM Support pages: *IBM MQ — Linear and Circular logging* (document 5737737, modified 13 March 2020); *Fix list for IBM MQ Version 9.2 LTS*; *IBM MQ Message persistence FAQs* (document 346659, abstract only); *IBM MQ — High Availability and Disaster Recovery* (document 771965, modified 17 March 2019). IBM-published community articles: Ian Harwood, *IBM MQ, IBM MQ for z/OS and IBM MQ Appliance firmware 10.0 Long Term Support is here! Plus the MQ Appliance M2004* (16 June 2026); Jonathan Rumsey, *Increasing IBM MQ message & service availability* (28 February 2025, on Native HA CRR in 9.4.2); Abhinav Priyadarshi, *Design and Architect IBM MQ Native HA…* (17 June 2025); Jamie Squibb, *IBM MQ Appliance DR between HA groups in 9.3.2* (16 February 2023). IBM announcement: *Introducing IBM MQ 10.0* (published 21 April 2026). IBM-published documentation: `ibm-messaging/mq-rdqm` *RDQM Architecture*; the IBM MQ Appliance proof-of-technology lab (IBM Cloud Integration). IBM MQ 9.4.x and 10.0.x documentation topics retrieved via search: *Message persistence*; *Recovering damaged objects*; *rcrmqobj (re-create object)*; *dmpmqcfg (dump queue manager configuration)*; *Backing up queue manager configuration*; *Native HA* (9.3.x, 9.4.x); *Installing RDQM* (9.4.x); *Migrating replicated data queue managers* (9.4.x); *Automatic client reconnection* (9.3.x); *How automatic balancing uses automatic reconnect* (9.4.x); *Setting up a key repository on AIX, Linux, and Windows*; *Creating a key repository for use as a TLS truststore*. Partner and community sources used only where labelled ⚠: W3Partnership, *Creating an IBM MQ H/A Cluster using a Multi-Instance Queue Manager* (multi-instance design, cluster repository queue names, circular-versus-linear comparison table). Regulatory sources: ESMA, EIOPA, EBA and industry summaries on DORA (Regulation (EU) 2022/2554); MAS Business Continuity Management Guidelines revision of 6 June 2022; MAS Technology Risk Management Guidelines.

**What is deliberately absent from this guide.** Delivery semantics, idempotency and dedupe stores, dead-letter queue theory, consumer commit discipline, the transactional outbox, event sourcing, and the cross-broker guarantees comparison — all owned by [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md). The middleware platform taxonomy and the MQ object model — owned by [enterprise_middleware_integration_platform_guide.md](enterprise_middleware_integration_platform_guide.md). The operational-resilience framework — owned by [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md). The z/OS recovery architecture — out of scope, flagged in §13.

---

## 16. The Closing Summary

**The single sentence.** IBM MQ has no shortage of disaster-recovery mechanisms; it has a shortage of estates that have proved they work.

**What this guide established.**

- **HA is not DR, and the difference is a site.** Multi-instance, RDQM HA, Native HA, and the appliance HA pair all protect against the failure of a process, a node, or a storage path *inside one place*. None of them protects against losing the place. An estate can have excellent, proven, hard-won HA and no DR at all — and the evidence trail will look reassuring right up to the moment it becomes irrelevant.
- **The recovery ceiling is set by the log, not by the DR product.** Circular logging is the IBM MQ default and it permits restart recovery only: no media images, no damaged-object rebuild, and if the queue manager object itself is damaged, no start unless you restore a backup or delete it. Linear logging is what IBM recommends for production, and it is a commitment — media images on a schedule, log extents archived and retrievable, and an understanding that the requirement for media recovery (AMQ7468) is not the same as the requirement for restart (AMQ7467).
- **Non-persistent messages do not survive a failover.** They are discarded whenever the queue manager stops, and media recovery cannot bring them back. Any queue carrying non-persistent messages on a financial path has an RPO of "everything in flight," and no HA or DR mechanism changes that.
- **The site-loss options are real, version-gated, and platform-gated.** RDQM DR and DR/HA RDQM on Linux with IBM MQ Advanced — synchronous or asynchronous for the former, *always* asynchronous for the latter. Native HA Cross-Region Replication, asynchronous, introduced in IBM MQ 9.4.2 on 27 February 2025, with the recovery group itself three replicated instances and no third-party dependencies. In-Region Replication, new in IBM MQ 10.0.0, a two-node DR topology with manual switchover, alongside CRR's six nodes across two regions. Appliance DR, including DR between HA groups since 9.3.2. And the generic route — storage-level replication — which is legitimate as a supplement and dangerous as a primary mechanism for a messaging layer, because crash-consistent copies of each node taken at different instants do not reconstruct a set of queue managers that agree with each other.
- **Backups are two different things and both are honest only if labelled.** A queue manager data backup captures the messages, but it requires stopping the queue manager to be consistent, which makes it a last resort rather than a DR strategy. A configuration export — `dmpmqcfg` on AIX/Linux/Windows, `DMPMQMCFG` on IBM i — captures every object definition and authority record and **zero messages**. An estate whose DR plan is a nightly configuration export has a plan to rebuild an empty messaging estate, and that is a business decision that must be taken explicitly rather than discovered.
- **A cluster is not a DR mechanism.** It redistributes new work to available members and it does not replicate the messages already sitting on a failed one. IBM says so directly.
- **Certificates break failovers.** The key repository and truststore at the recovery site rot on a calendar, the DR rotation step gets dropped from the change record, and the failure presents as an authentication problem at the worst possible moment. The single highest-value exercise in an MQ DR programme is letting a test certificate expire and proving the team can fix it inside the RTO.
- **A DR plan never executed end-to-end is a hypothesis.** Not a control, not evidence, and not risk reduction — risk relocated to the incident. The undiscovered items are almost all discovered by doing: certificate drift, CCDT staleness, untested failback, quorum behaviour under partial failure, undocumented manual steps, start-up order, firewall lead times longer than the RTO, and a DR site nobody load-tested.
- **For a regulated institution, the messaging layer is the platform, not a detail of it.** Payments, interbank connectivity, core-banking interfaces, and file transfer all ride on it, and its unavailability is their unavailability. What the institution will be asked for is not the product name but the measured RTO, the measured RPO in messages and in time, the reconciliation that makes the RPO claim defensible, and dated evidence that all of it was tested.
- **In the fictional Cymbal Bank design, the honest positions were the valuable ones.** Native HA with CRR for the top tier, with the asynchronous region-loss window accepted, alerted on, and reconciled rather than denied. RDQM DR/HA for Tier 1 with its unavoidable asynchronous inter-site RPO accepted in writing. Tier 2 with no messaging-layer DR at all, and a tested business re-send procedure instead of a comforting diagram. Tier 3 with an explicit accepted-loss decision, re-confirmed annually. And a list of the single points of failure that remain — the decision time, the certificate process, the CCDT publisher, the external connectivity, the small number of people who hold the knowledge — because a DR design without that list is a sales document.

**The last thing worth saying.** Every observation in this guide that is worth money came from the same source: the gap between what the architecture says and what an exercise actually does. IBM MQ is unusually good at keeping its promises — the log is written before the change is applied, the replicas acknowledge before the active instance proceeds, the quorum refuses to run a queue manager on a minority rather than split the truth in two. Those are real guarantees, and they hold whether or not anyone is watching. What they cannot do is hold for a plan that has never been executed, a certificate that expired unseen, a CCDT nobody republished, or a business decision nobody wrote down.

*End of guide — HA protects the node, DR protects the site, and only a rehearsed, evidenced restore finally delivers on* the log's promise.
