# Clusterware in Banking and Financial Services: The Decision About Who Survives

**Jack Liu Shurui, Solution Architect**

> **Author:** Jack Liu Shurui - Solution Architect at Cymbal Bank, Singapore
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** Technology / Platform Infrastructure (technology/)
> **Audience:** Solution architects, database and platform engineers, resilience and change-management leads operating tier-0 clustered systems
> **Last Updated:** November 2026

*A deep-research guide to clusterware - the layer that decides which node of a cluster is allowed to keep running - and to why banks still run it under their most consequential systems. It covers the four jobs clusterware performs (membership, quorum, fencing, resource management), the split-brain problem that is the whole reason the discipline exists, the five product families that dominate financial services (Oracle Clusterware/Grid Infrastructure, Pacemaker with Corosync, IBM PowerHA SystemMirror, Veritas InfoScale, Microsoft Failover Clustering), the determinants of failover time as a chain rather than a number, the operational reality of patching a live cluster, what containers did and did not replace, how an institution proves the clustering works, and a fictional Cymbal Bank tier-0 example that puts the central trade-off on the table with its cost. Product facts are version-gated and traced to vendor documentation; where a vendor document could not be reached, the guide says so instead of filling the gap.*

**Cross-references:** [oracle_database_guide.md](oracle_database_guide.md) (the database itself - this guide owns the cluster layer beneath it and explains RAC and ASM only as cluster substrate), [ibm_mq_disaster_recovery_guide.md](ibm_mq_disaster_recovery_guide.md) (messaging-layer HA and DR - section 4, "The Local Resilience Family: MIQM, RDQM, Native HA, and the Appliance", owns the MQ-level mechanisms that are *not* clusterware), [openshift_workload_availability_guide.md](openshift_workload_availability_guide.md) (pod-level availability, scheduling, PodDisruptionBudgets, topology spread, etcd quorum, stateful workload patterns - the orchestration layer this guide distinguishes itself from), [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) (the general discipline of designing for continuity), [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) (impact tolerances and the evidence model - condensed here, owned there), [../banking/core_banking_systems_guide.md](../banking/core_banking_systems_guide.md) (the tier-0 cores that clusterware usually sits under).

**How to read this guide.** Section 1 is the decoder, the thesis and the layering boundary. Section 2 is the four jobs every clusterware product performs, and what breaks when one is missing. Section 3 is split-brain - the analytical core, and the longest section in the guide. Section 4 compares the five product families structurally, without ranking them. Section 5 is Oracle in depth, because banking runs it. Section 6 covers the others accurately and briefly. Section 7 explains why a bank still runs this. Section 8 decomposes failover time into its determinants and prints no total. Section 9 is the operational reality. Section 10 is what containers did and did not replace. Section 11 is the evidence question - how an institution proves the clustering works. Section 12 is the regulated angle. Section 13 is six failure modes with guardrails. Section 14 is the Cymbal Bank worked example. Section 15 is the claims audit, with a separate "What Could Not Be Verified" subsection. Section 16 is the glossary and the closing summary.

**Integrity convention.** Verified tick = read this pass at the vendor's own documentation, with the version gate and date recorded in section 15. Warning sign = partially verified, project-documentation sourced rather than vendor-documentation sourced, version-gated and not re-confirmed at the newest release, or class-of-behaviour only. Cross mark = rejected, with the reason. Product claims without a mark are architectural reasoning, not vendor facts. **No failover-time figure, throughput figure, benchmark or product ranking appears anywhere in this guide** - section 8 explains why the number is a property of the configuration and not of the product.

---

**Contents**

1. [The Overview: The Decoder, the Thesis, and the Boundary](#1-the-overview-the-decoder-the-thesis-and-the-boundary)
2. [What Clusterware Actually Does: The Four Jobs](#2-what-clusterware-actually-does-the-four-jobs)
3. [Split-Brain: Why the Subject Exists](#3-split-brain-why-the-subject-exists)
4. [The Product Families, Structurally](#4-the-product-families-structurally)
5. [Oracle Clusterware in Depth](#5-oracle-clusterware-in-depth)
6. [The Other Families, Accurately and Briefly](#6-the-other-families-accurately-and-briefly)
7. [Why a Bank Runs This](#7-why-a-bank-runs-this)
8. [The Determinants of Failover Time](#8-the-determinants-of-failover-time)
9. [The Operational Reality](#9-the-operational-reality)
10. [What Containers Did and Did Not Replace](#10-what-containers-did-and-did-not-replace)
11. [The Evidence Question](#11-the-evidence-question)
12. [The Regulated and Banking Angle](#12-the-regulated-and-banking-angle)
13. [The Failure Modes](#13-the-failure-modes)
14. [The Worked Example: Cymbal Bank's Tier-0 Database Cluster](#14-the-worked-example-cymbal-banks-tier-0-database-cluster)
15. [The Claims Audit](#15-the-claims-audit)
16. [Glossary and Closing Summary](#16-glossary-and-closing-summary)

---

## 1. The Overview: The Decoder, the Thesis, and the Boundary

### 1.1 The Thesis

**Clusterware does not prevent failure; it converts a node failure into a decision about who survives - and in a bank that decision is a control, not a feature.**

Everything else in this guide is an elaboration of that sentence, so it is worth unpacking before the vocabulary arrives.

- **It does not prevent failure.** Clusterware has no opinion about why a node stopped. It does not make disks more reliable, kernels less buggy, or networks less lossy. It assumes failure is normal and recurring, and it is designed for the moment after the failure has already happened.
- **It converts a failure into a decision.** The interesting part of a cluster is not the restart. It is the judgement, made under time pressure, with partial information, about which surviving part of the system is entitled to continue. That judgement is what quorum is for. A cluster is, structurally, a machine for making that judgement without a human in the loop.
- **The decision is about who survives.** In a two-node cluster, one node's survival is decided by the other node's evidence about it. A node that cannot be seen is presumed guilty, and the presumption is enforced by removing its ability to write - not by arguing with it.
- **In a bank, the decision is a control.** A control is something the institution must be able to demonstrate was designed, operated and evidenced. "The cluster failed over" is an operational event; "the cluster made the correct membership decision, the fence fired, and we can show the record" is a control assertion. Section 12 develops why that distinction is the one auditors and resilience supervisors actually care about, and section 11 describes what evidence satisfies it.

Two corollaries follow immediately, and the guide tests them throughout:

1. **A cluster that has never fenced is not a proven cluster.** It is an untested one. The fence is the part of the system that converts belief into fact, and it is the part most often disabled, most often untested, and most often the reason a "successful" failover was actually a lucky one.
2. **The most dangerous cluster is the one that works.** A cluster that fails over cleanly for years accumulates no evidence about the case that matters - the case where the network partitions and both halves believe they are the cluster.

### 1.2 The Decoder

Nine terms carry the whole subject. They are defined here once, at the level of meaning rather than product syntax, and then used consistently.

| Term | What it means | Why it matters |
|---|---|---|
| **Clusterware** | The software layer that makes a set of independent servers behave as one system for the purpose of availability: it decides membership, arbitrates quorum, fences the losers, and starts and stops managed resources in the right order. Oracle's own definition is that it "enables servers to communicate with each other, so that they appear to function as a collective unit" (Oracle Clusterware Administration and Deployment Guide, 19c). | It is the layer *below* the application and *beside* the operating system. It is not a database feature and not a scheduler. |
| **Cluster** | The set of nodes governed by one membership view and one configuration. A cluster is defined by what it agrees on, not by what it contains. | Two servers with the same software but no shared membership view are two clusters that happen to be near each other. |
| **Node** | One server (physical or virtual) that is a member of the cluster and runs the clusterware daemons. | Node granularity is the defining limit of the whole technology: clusterware moves a node's workloads, not a process's. |
| **Quorum** | The rule that determines whether a surviving subset of nodes is entitled to act as the cluster. Typically a majority of votes, where votes come from nodes and sometimes from a tie-breaking device. | Quorum is what makes "I am the cluster" a claim requiring witnesses rather than an assertion a node can make about itself. |
| **Fencing** | The act of removing a node's ability to do harm - to write to shared storage, to serve clients - usually by rebooting or powering it off. In Pacemaker's vocabulary, STONITH ("Shoot The Other Node In The Head"). | Fencing is the enforcement mechanism behind the membership decision. Without it, the decision is an opinion. |
| **Split-brain** | The state in which two subsets of a cluster each believe they are the cluster and both proceed, typically both writing to the same data. | It is the failure that clusterware exists to prevent, and the only one that is not recoverable by a restart. Section 3. |
| **VIP** | Virtual IP address - an address that is not bound to a physical NIC but is moved between nodes as a resource, so clients reconnect to the same address after a failover. Oracle calls the per-node instance a node VIP and the client-facing set SCAN VIPs. | The VIP is where the cluster's decision becomes visible to the application estate, and where it is most often incompletely implemented. Section 13(e). |
| **Resource group** | A named set of resources that must be managed together - started, stopped and moved as a unit, in a defined order. Veritas calls them service groups, IBM PowerHA calls them resource groups, Oracle models them as resources with dependencies, Pacemaker as resources with ordering and colocation constraints. | Grouping is how the cluster preserves the application's start order, which is a property of the application and not of the cluster. |
| **Failover** | The transfer of a resource group from one node to another as a consequence of a failure (as opposed to a *switchover*, which is the same transfer performed deliberately). | The distinction is the difference between evidence and rehearsal. Section 9 and section 11 both turn on it. |

### 1.3 The Layering Boundary, Stated Once and Then Assumed

**Clusterware sits below the application. It moves a whole node's workloads. It does not know what a transaction is, what a queue is, or what a pod is.** Almost every confused conversation about high availability in a bank comes from collapsing this boundary, so it is worth drawing three lines explicitly.

**Line 1 - against IBM MQ's own HA features.** The MQ guide in this repository ([ibm_mq_disaster_recovery_guide.md](ibm_mq_disaster_recovery_guide.md), section 4) owns that family and this guide does not re-derive it. The distinction that matters here:

- **Multi-instance queue managers (MIQM)** are an MQ-level shared-queue mechanism. The standby takes over by acquiring read/write locks on the queue manager's data on shared storage - a *file lock*, not a cluster vote. There is no membership protocol, no quorum, no fence. Two instances, one active, and the storage is a single point of failure.
- **RDQM** is genuinely built on clusterware - IBM documents that "the group configuration of the three nodes is handled by Pacemaker" and "the replication between the three nodes is handled by DRBD". But it *presents* as an MQ feature: it protects a queue manager, not a node's whole workload set, and its quorum rules belong to MQ's own object model.
- **Neither is a substitute for clusterware, and clusterware is not a substitute for MQ-level recovery of in-doubt work.** Clusterware will move a queue manager's resource group to another node; it will not resolve an in-doubt channel, and it will not tell you whether the messages that were in flight are where you think they are. The two layers answer different questions, and a bank needs both answers.

**Line 2 - against Kubernetes and OpenShift pod-level availability.** [openshift_workload_availability_guide.md](openshift_workload_availability_guide.md) owns scheduling, PodDisruptionBudgets, topology spread, etcd quorum and stateful workload patterns. The boundary is a matter of *granularity and of the write path*, and section 10 develops it. In one sentence: a scheduler decides *where a pod runs*; clusterware decides *which node is allowed to touch the data*, and those are not the same decision even when they run on the same hardware.

**Line 3 - against Oracle the database.** [oracle_database_guide.md](oracle_database_guide.md) owns the database: instance architecture, storage, backup, recovery, tuning. This guide owns the substrate beneath it. Oracle RAC and ASM appear here only as cluster-substrate components - RAC as the multi-instance database whose membership is granted by Clusterware, ASM as the disk-management layer that also hosts the cluster's own voting files and registry. Where the database guide says "RAC gives you instance-level redundancy", this guide asks the prior question: *on what basis did the surviving instances conclude that the failed ones were gone?*

### 1.4 What This Guide Is Not

It is not a product selection guide, and it deliberately contains no ranking. Section 4 explains each family's design centre - the problem it was built to solve and the platform it grew up on - because that is what determines fit, and because "which is best" is not a question that has an answer independent of the estate, the skills, the storage, and the change-control regime. It is also not an installation guide: no command syntax appears here that was not read at a vendor source, and where syntax would be version-fragile the guide describes the operation instead.
---

## 2. What Clusterware Actually Does: The Four Jobs

Every product in this space, from a two-node Pacemaker pair to a 64-hub-node Oracle Flex Cluster, performs the same four jobs. The names differ; the jobs do not. This section defines each one and then asks the question that makes the definition useful: **what happens when this job is absent?** A cluster missing one of the four is not a weaker cluster; it is a different, and usually worse, thing wearing the same word.

### 2.1 Job One: Membership

**Membership is the maintained, agreed answer to "who is in?"** It is not a configuration list. A configuration list says which nodes are *supposed* to be in the cluster; membership says which nodes *are* in it right now, and every member holds the same view.

Oracle's Cluster Synchronization Services is defined in exactly these terms: it "manages the cluster configuration by controlling which nodes are members of the cluster and by notifying members when a node joins or leaves the cluster" (Oracle Clusterware Administration and Deployment Guide, 19c, section 1). Pacemaker does not implement membership itself at all - it consumes it from Corosync, which Pacemaker's own documentation describes as the component that "provides reliable messaging, membership, and quorum information about the cluster" (Pacemaker Explained, 3.0, section 1.2.1). Microsoft's model is looser in vocabulary but the same in substance: nodes "collaborate to maintain what is known as a quorum", and the failure of a node is detected by the absence of heartbeat.

Membership is a *distributed agreement*, and that is the source of all its difficulty. There is no privileged observer inside the cluster. Each node's knowledge of the others arrives as messages, and messages are exactly what a failure disrupts.

**With membership absent**, there is no shared view of who is in, so there is no basis for any of the other three jobs. Every node's picture of the cluster is private. Resources get started twice because two nodes each believe the other is gone; or resources are never started because each node believes the other is still running them. This is the failure mode of a cluster whose membership layer is broken but whose resource manager is still running, and it is more dangerous than a total outage because everything looks healthy from the inside.

### 2.2 Job Two: Quorum

**Quorum is the rule that turns a subset of nodes into "the cluster".** Membership tells you who is present; quorum tells you whether the present set is entitled to act. Almost always it is a majority rule: a subset may act if it holds more than half of the votes. Votes normally come from nodes, and sometimes additionally from a tie-breaking device - a quorum disk, a file-share witness, a cloud witness, a quorum device service - precisely so that an even number of nodes can still produce a majority.

The reason quorum is a *majority* rule and not a *first-come* rule is the reason the whole subject exists: under a partition, both halves are present, both are healthy, and both would otherwise be entitled to act. A majority rule guarantees that **at most one** subset can satisfy it. That guarantee is the entire value proposition, and it is bought at a price that section 3 makes explicit: the minority half must stop, even though it may be the half with the more important workload on it.

Quorum is also where the products diverge most visibly, because the tie-break device is a physical and organisational choice:

- **Oracle** uses voting files, described in the 19c guide as the means by which Clusterware "determines which nodes are members of a cluster" and, in the same chapter, as the mechanism used "to provide fencing and cluster node membership determination". A minimum of three voting files on physically separate storage is recommended when they are not on ASM; the documented maximum is 15 and Oracle recommends not exceeding five.
- **Pacemaker** receives quorum from Corosync and then decides what to *do* about its absence via the `no-quorum-policy` cluster option, with documented values `ignore`, `freeze`, `stop` (the default), `demote` (since 2.0.5) and `fence` (since 2.1.9) (Pacemaker Explained, 3.0, section 3.4).
- **Microsoft** offers the richest menu of quorum *models*: Node Majority, Node and Disk Majority, Node and File Share Majority, and No Majority (Disk Only), with a cloud witness documented for Windows Server 2016 and later, and dynamic quorum managing node votes automatically in later releases.
- **Veritas InfoScale** and **IBM PowerHA** both use a shared-storage arbitration idiom, described at class level in section 6 because their documentation could not be reached this pass.

**With quorum absent** - or worse, with quorum *policy set to ignore* - you get two clusters. Each half has membership, each half has a resource manager, and each half starts the resources it believes it should own. If the workload is read-only, you get two answers to the same question. If it is write-oriented and shares storage, you get corruption. Section 3.

### 2.3 Job Three: Fencing

**Fencing is the removal of the loser's ability to write.** It is not a notification, not a request to shut down cleanly, and not a vote. It is an action taken by the survivors against a node that may still be running, using a mechanism the target cannot refuse: a power controller that cuts power, a management interface that forces a reset, a storage-layer action that revokes the target's access to the LUN, or - in the self-fencing variant - a watchdog timer on the target itself that fires when it can no longer prove it is healthy.

Pacemaker's documentation states the definition and the necessity in two sentences that are worth quoting in full, because they are the clearest vendor statement of the principle in this space:

> "Fencing, also known as STONITH (an acronym for Shoot The Other Node In The Head), is the ability to ensure that it is not possible for a node to be running a service. This is accomplished via fence devices such as intelligent power switches that cut power to the target, or intelligent network switches that cut the target's access to the local network."

> "A cluster cannot safely recover from certain failure conditions, such as an unresponsive node, without fencing."

(Pacemaker Explained, 3.0, section 1.2.)

Oracle's equivalent is node eviction, and its documentation is equally direct about the consequence: the `cssdagent` process "monitors the cluster and provides I/O fencing", and "a `cssdagent` failure may result in Oracle Clusterware restarting the node" (Oracle Clusterware Administration and Deployment Guide, 19c, section 1.3.1). Note the shape of the sentence: the *failure of the fencing component* causes a node restart. The mechanism is designed so that the inability to fence is itself treated as a reason to stop.

The distinction that people consistently get wrong: **fencing removes the loser's ability to write, not its belief that it won.** A fenced node does not concede. It is killed mid-thought, which is exactly the point - because a node that is allowed to finish its thought is a node that writes to shared storage after the survivors have begun using it. Everything after that is recovery, not failover.

**With fencing absent**, you have a cluster whose membership decisions are unenforceable. The survivors decide the loser is gone, start the resources, and then discover that the loser is still writing. Pacemaker's own documentation says this plainly in the description of its `fencing-enabled` option: if fencing is disabled, "unresponsive nodes are immediately assumed to be running no resources, and resource recovery on online nodes starts without any further protection (which can mean *data loss* if the unresponsive node still accesses shared storage, for example)" (Pacemaker Explained, 3.0, section 3.4). That sentence is a vendor telling you what turning the fence off actually means.

### 2.4 Job Four: Resource Management

**Resource management is the ordered start, stop, monitor and relocation of the things the cluster exists to protect.** It is the job everyone *thinks* is the whole job, because it is the visible one: the database comes up on the other node, the VIP moves, the listener follows.

The substance of the job is *order and dependency*. A database instance needs its filesystem mounted, its ASM disk group mounted, its VIP present, its listener started - in that order, and stopped in roughly the reverse. Oracle models this with resource dependencies and start/stop profiles; Pacemaker with ordering and colocation constraints; PowerHA with resource groups and their processing order; Veritas with service group dependencies. The names differ, the graph does not.

The second substance of the job is *monitoring*. A resource that has started is not a resource that is working. Clusterware therefore runs recurring monitor operations against resources, and the failure of a monitor operation is what triggers most relocations - which is why the depth of the monitor matters more than the speed of the failover. A monitor that only checks that a process exists will pass while the process serves nothing.

**With resource management absent**, you have a node that is up, a cluster that agrees it is a member, and nothing running in the right order - or something running in the wrong order, which is worse. The classic symptom is an application that starts before its database is accepting connections and then latches into a permanently broken state that a restart of the application does not fix.

### 2.5 The Four Jobs as One System

The jobs are not independent, and the dependency runs in one direction:

```
membership -> quorum -> fencing -> resource management
```

Resource management is only safe if the fence has completed. The fence is only justified if quorum said the survivors may act. Quorum is only meaningful if membership is accurate. Remove any link and the chain above it becomes a liability rather than an asset: a resource manager without a fence is an automated way to cause corruption, and a fence without quorum is an automated way to cause an outage.

That is the structure the thesis describes. The four jobs are the machinery of the decision; the decision is the product.

---

## 3. Split-Brain: Why the Subject Exists

**This section is the guide.** Everything else - the product families, the operational discipline, the audit evidence - is downstream of the problem described here. If a reader takes one section away, it should be this one.

### 3.1 What Split-Brain Is

Split-brain is the state in which two subsets of a cluster each believe they are the cluster, and both proceed. It is not a hypothetical. It is the *normal* consequence of a network failure in a system that has no arbitration, and it is the reason every serious clusterware product carries a quorum mechanism whether or not its users understand why.

Concretely: four nodes, two in rack A and two in rack B, one shared storage array reachable from both. The inter-rack link fails. Nothing else fails. Every node is healthy, every operating system is responsive, every database process is running, and every node can still reach the LUN. From each node's point of view, the other half of the cluster has vanished. With no arbitration, both halves do the obvious thing: each promotes its local instance, each mounts the data, each starts writing.

The result is not an outage. It is worse than an outage. Two writers have been interleaving updates against the same data structures, and the on-disk state is now a mixture of two histories that never existed. **A restart does not fix this.** Restarting a database recovers to a consistent point in *one* history; here there is no single history to recover to. The recovery path is restore-from-backup plus a reconciliation exercise conducted by humans with domain knowledge, and the exercises in section 11 exist because that outcome is unacceptable in a bank.

Three properties of split-brain deserve naming, because they shape every design decision that follows:

1. **It is caused by a partition, not by a crash.** The nodes did not fail. The *evidence* failed. Any mechanism that treats "I cannot see my peers" as equivalent to "my peers are dead" is a split-brain generator.
2. **It is silent at the moment it happens.** Both halves are healthy by every local measure. Monitoring inside each half reports green. The only observers who can see the problem are the ones who can see both halves, and they are usually the humans arriving twenty minutes later.
3. **It is asymmetric in cost.** The cost of preventing it is an outage that may not have been necessary. The cost of failing to prevent it is unrecoverable data. Section 3.5 is about living with that asymmetry honestly rather than pretending it away.

### 3.2 Why a Heartbeat Is Not Proof of Liveness

The instinctive design for failure detection is a heartbeat: node A sends a message to node B every interval; if B does not answer within a threshold, A declares B dead. It is simple, it is universal - every product in section 4 uses it in some form - and it is **not a proof of anything**.

The logic problem is that a missed heartbeat is evidence about *the path*, not about *the endpoint*. "A cannot hear B" is consistent with at least four very different world states:

| World state | What A observes | What is actually true |
|---|---|---|
| B is dead | No heartbeat | B is dead. Fencing is safe and the decision is correct. |
| B is alive, the network is broken | No heartbeat | B is running, serving clients, and writing to shared storage. Fencing is *necessary*. |
| B is alive, A's network stack is broken | No heartbeat | A is the broken node. Fencing B is the *wrong* decision, made confidently. |
| B is alive but cannot reach shared storage | Heartbeat fine, I/O fails | Two nodes each believe the other has the storage problem. Neither heartbeat nor its absence distinguishes this case. |

The fourth row is the one that catches experienced operators, because it produces a cluster that is *membership-healthy and storage-broken*. Heartbeats flow, quorum is satisfied, and the cluster's view of the world is coherent - while the data path underneath is split. This is why mature deployments do not rely on a single network path as their only evidence, and why storage-level fencing exists as a separate mechanism rather than as a detail of the network layer.

Two further properties of heartbeats matter, and both are configuration rather than product:

- **The interval and threshold are a detection-latency choice with a false-positive cost.** A short interval and a low threshold detect real failures faster and fence healthy nodes during transient congestion more often. A long interval and a high threshold do the reverse. There is no correct value in the abstract; there is only a value that matches the network's measured behaviour under load, which is why section 8 treats detection as the first link in a chain rather than as a product property.
- **The heartbeat path is itself a component that can fail asymmetrically.** One-way failures - A's traffic reaches B, B's replies are dropped - are common in practice (a failing SFP, a misconfigured ACL applied on one side, an asymmetric route) and are invisible to any monitoring that only asks "is the link up?"

The conclusion is not that heartbeats are bad. It is that **a heartbeat is a hint, and clusterware is the machinery that refuses to act on a hint alone.** Quorum is how the hint gets corroborated. Fencing is how the corroborated decision gets enforced.

### 3.3 What Quorum Does

**Quorum converts "I am the cluster" from a claim a node can make about itself into a claim that requires witnesses.**

The mechanism is deliberately blunt: votes are counted, and a subset may act only if it holds a majority. In a four-node cluster, no partition of two can hold a majority, so *neither* half may act - the cluster stops, and a human decides. In a five-node cluster, a partition of three may act and a partition of two may not. The asymmetry is the point: majority rule guarantees uniqueness, not justice.

This is where the tie-break devices earn their keep. An even number of nodes cannot produce a majority under partition, so a witness is added to make the arithmetic work:

- **A quorum disk or arbitration LUN** - a small shared device that one partition can claim, contributing a vote. Microsoft's Node and Disk Majority model is the canonical example; Microsoft's documentation explicitly frames the purpose as preventing "split-brain scenarios, where split portions of a cluster might try to operate independently, potentially causing inconsistencies".
- **A file-share or cloud witness** - an external, low-cost vote over SMB or HTTPS. Microsoft documents the cloud witness as requiring an Azure storage account and outbound HTTPS on port 443, with the witness's blob stored in a container the witness creates; it is documented as applying to Windows Server 2016 and later in the current failover-clustering documentation set.
- **A quorum device service** - a small daemon outside the cluster that answers the arbitration question for both halves and can only answer one of them affirmatively. In the Pacemaker world this role is played by the Corosync quorum device (`corosync-qdevice`, with its server counterpart `corosync-qnetd`); the design intent is documented in the Corosync project, but the vendor-documentation confirmation of its behaviour was not obtained this pass and it is therefore marked as class-of-behaviour in section 15.
- **Voting files or a cluster registry, replicated** - Oracle's approach, where multiple voting files on separate storage provide the arbitration, and the recommendation of at least three on physically separate storage exists precisely so that one storage failure cannot remove the arbitration capability.

Two properties of quorum are worth stating plainly because they are routinely misunderstood:

**Quorum is a property of the group, not of the node.** A node does not "have quorum"; a subset does. A node that is a member but whose partition lacks quorum is in the most awkward state in the whole system: it is running, it is connected to some peers, and it is forbidden to act.

**Quorum answers "may we act", never "should we act".** It is a safety property, not a liveness property. A cluster that satisfies quorum and then fails to start the resource is quorate and useless, which is why section 8's chain does not end at the membership decision.

### 3.4 What Fencing Actually Does

If quorum is the decision, fencing is the enforcement - and the gap between them is where clusters break.

**The decision is made by the survivors. The enforcement is applied to the loser.** The survivors cannot ask the loser to stop, because the loser may be unreachable, may be wedged, may be running a kernel that will not schedule the shutdown, or may simply disagree. So the survivors do something the loser cannot refuse: they cut its power, reset it, or revoke its access to storage. In Pacemaker's vocabulary this is a fence agent invoked through the fencer daemon (`pacemaker-fenced`), against a device that may be a power distribution unit, a management controller, a storage array, or a watchdog; in Oracle's vocabulary it is node eviction, performed by the clusterware's own fencing path, with a documented consequence that a fencing-component failure "may result in Oracle Clusterware restarting the node".

Three properties make fencing the hardest part of the discipline to operate:

**1. Fencing is destructive by design, and that is not a defect.** The whole value of a fence is that it does not depend on the target's cooperation. The corollary is that a fence cannot be tested non-destructively on a production node, which is why organisations test it on the node they can afford to lose - and why, all too often, they do not test it at all.

**2. Fencing has a completion time, and the survivors must wait for it.** A fence is an out-of-band action against physical or virtual hardware, and it can fail, time out, or return ambiguous results. Pacemaker exposes this honestly: `fencing-timeout` defaults to 60 seconds for the fence action to complete, and `fencing-max-attempts` defaults to 10 before the cluster stops immediately re-attempting. Those are *configuration defaults for the fence operation*, not failover times - section 8 is emphatic about the difference - but they show that the fence is a phase with a duration and a failure mode of its own.

**3. Fencing can be defeated by a device that lies.** A power controller that reports success without cutting power, a management interface whose credential has expired, a virtualisation layer that acknowledges a reset and then restores the VM from a snapshot, a storage-level fence applied to the wrong initiator group: each of these produces a cluster that believes it has fenced and has not. The failure is silent, and it converts a correctly-decided failover into a split-brain with a clean audit trail. This is the single most under-tested failure mode in the whole subject.

### 3.5 The Uncomfortable Truth: The Tie-Break Is a Policy Decision With a Data-Loss Consequence Either Way

Here is the part that vendor diagrams leave out. **There is no setting of a cluster that makes both halves safe.** The choice is not between a safe configuration and an unsafe one; it is between two unsafe configurations with different victims.

**Aggressive fencing.** The cluster fences any node it cannot see, quickly and without hesitation. Under a real node failure this is exactly right: the node is down, the fence is a formality, and the resource group restarts promptly. Under a transient network event, the same policy kills a healthy node that was serving transactions, and the "high availability" cluster has manufactured an outage. Do this often enough and the operations team learns to distrust the cluster, which is the beginning of the end - see section 13(b).

**Conservative fencing.** The cluster hesitates: longer detection windows, retries before declaring a node lost, policies that let resources continue without quorum, fences disabled during "sensitive" periods. Under a real partition this is exactly wrong: both halves keep running, both keep writing, and the institution gets corruption. Unlike an outage, corruption is **not recoverable by a restart**. It is recoverable by restore, reconciliation, and a customer-facing incident whose duration is measured in days.

The asymmetry is real and it is not symmetric in the other direction:

| | Aggressive fence | Conservative fence |
|---|---|---|
| Under a real node failure | Correct, fast, unremarkable | Slow, or a stall requiring a human |
| Under a network partition | An unnecessary outage, recoverable in minutes | **Corruption, not recoverable by restart** |
| Worst case | Availability loss | Integrity loss |
| Who notices first | Customers, immediately | Nobody, for hours or days |
| Audit position | Explainable: the fence fired as designed | Indefensible: the fence was off |

The defensible position - and it is a position, not a default - is that **integrity dominates availability, and the aggressive fence is the correct policy, provided the detection parameters are tuned to the measured network and the fence itself is proven to work.** A bank that fences aggressively and has tested its fence has an availability problem it can explain. A bank that fences conservatively has an integrity problem it does not yet know about.

This is where the guide's thesis stops being a rhetorical flourish. The tie-break is a **policy decision**, made by named humans, recorded in a design authority, with a data-loss consequence attached to one branch and an outage consequence to the other. Once that is true, the cluster configuration is not an infrastructure detail; it is a control, and the institution should be able to produce the decision, the rationale, and the evidence that the decision is the one actually running in production.

### 3.6 What the Products Actually Offer, and What They Cannot Offer

The mechanisms differ; the trade-off does not.

- **Oracle** makes voting files the arbitration device and ties fencing to cluster membership in the same breath: the 19c guide states that Clusterware "uses voting files to provide fencing and cluster node membership determination". Eviction is the enforcement, and the documentation is candid that the fencing component's failure "may result in Oracle Clusterware restarting the node".
- **Pacemaker** separates the concerns cleanly and makes the conservative end of the spectrum a first-class, documented option, which is useful and dangerous in equal measure: `no-quorum-policy` can be `ignore` (continue all resource management), `freeze` (continue, but do not recover resources from nodes not in the affected partition), `stop` (the default: stop everything in the affected partition), `demote` (since 2.0.5), or `fence` (since 2.1.9). Each is a defensible choice in some context and a data-loss decision in another. A cluster set to `ignore` is a cluster that has decided, on the record, that it prefers availability to uniqueness.
- **Pacemaker also exposes the 2-node problem as an explicit knob**, which is unusual in this family: `priority-fencing-delay` (since 2.0.4) applies a delay to fencing of the lost node with the highest total resource priority "so that the more significant nodes potentially win any fencing match (especially meaningful in a split-brain of a 2-node cluster)". In other words: in a two-node split brain, someone must die, and the product lets you say who - a policy decision with a data-loss consequence, expressed as a configuration attribute.
- **Microsoft** addresses the same problem through quorum *models* rather than through fence policy, and adds dynamic quorum so that the vote arithmetic changes as nodes leave. Its documentation frames the purpose exactly: quorum "prevents split-brain scenarios".
- **IBM PowerHA** and **Veritas InfoScale** both place arbitration in shared storage and fencing in a combination of storage-level and node-level actions. Their specific idioms are described at class level in section 6 and flagged in section 15, because their documentation hosts could not be reached this pass.

**What no product can offer** is a tie-break with no loser. Every mechanism in this section is a way of choosing which half stops. The choice can be made well or badly, tested or untested, documented or implicit - but it cannot be avoided, and a cluster whose configuration implies it was never made is a cluster that made it by accident.
---

## 4. The Product Families, Structurally

Five families dominate financial services. This section describes each one's **design centre** - the problem it was built to solve, the platform it grew up on, the idiom it uses for quorum and for fencing - and explicitly declines to rank them. Ranking would be a claim about fitness that cannot be made without the estate, the storage, the skills and the change regime, and it is precisely the kind of claim that ages badly.

### 4.1 The Five, Side by Side

| Family | Design centre | Platform gravity | Quorum idiom | Fence idiom |
|---|---|---|---|---|
| **Oracle Clusterware / Grid Infrastructure** | Being the substrate for one vendor's own multi-instance database, then generalising to manage any application's resources | Oracle's own platforms, with a strong Linux/UNIX centre; heterogeneous only in the sense that it runs on several OSs, not that it mixes them in one cluster | Voting files, ideally several on separate storage, plus a cluster registry holding configuration | Node eviction: the cluster restarts or removes a node that cannot prove membership, with the fencing component's own failure treated as a reason to restart |
| **Pacemaker + Corosync** | Being a general-purpose, scriptable cluster resource manager for whatever can be packaged as a resource agent | Linux distributions; the RHEL High Availability Add-On is the most common commercial packaging, and IBM MQ's RDQM ships it underneath | Corosync membership and quorum, optionally extended by an external quorum device service | STONITH through fence agents - power, management, storage or watchdog devices - plus SBD for shared-storage and watchdog-based self-fencing |
| **IBM PowerHA SystemMirror** | Availability for AIX and IBM i estates, with deep integration into the platform's own clustering services | IBM Power servers running AIX or IBM i; the successor name to HACMP | A shared repository disk plus the platform's own cluster services | Storage and node-level actions driven by the resource group's failure policy |
| **Veritas InfoScale (formerly Veritas Cluster Server)** | Cross-platform application availability with a strong storage-integration story, in heterogeneous estates | Multi-vendor UNIX and Linux, with Windows support; often found where the storage vendor's multipathing is also Veritas | Cluster membership over a dedicated low-level transport, with coordination disks for arbitration | I/O fencing against coordinator disks, plus node-level fencing |
| **Microsoft Failover Clustering (WSFC)** | Availability for Windows Server roles and for SQL Server failover cluster instances, tightly integrated with Active Directory and shared storage | Windows Server; the default answer for Windows-native tier-1 workloads | Quorum models - node majority, node and disk majority, node and file share majority, disk only - plus a cloud witness and dynamic quorum | Node removal from the cluster with resource-group restart; storage-level control through Cluster Shared Volumes ownership |

The table is a map, not a scorecard. Two observations follow from it that are worth more than a ranking would be.

**First, each family's quorum idiom is a fossil of the platform it grew up on.** Oracle's voting files are a database vendor's answer to arbitration: small, replicated, and managed by the same storage layer that manages the data. Microsoft's quorum models are an operating-system vendor's answer: a menu of topologies, because Windows clusters are deployed in every shape from a two-node branch office to a stretched multi-site cluster. Pacemaker's answer is to *not have* an opinion and to delegate to Corosync, with a pluggable external arbitrator - which is what a general-purpose tool looks like when it refuses to assume a topology. None of these is better; each encodes an assumption about how the cluster will be deployed.

**Second, the fence idiom tracks the failure domain the vendor cares most about.** Oracle fences the node, because in a shared-everything database the node is the thing that must not write. Veritas fences the I/O path against coordinator disks, because in a heterogeneous storage estate the dangerous thing is a host that can still reach the LUN. Pacemaker offers both and lets you choose, which is the honest position and the one that requires the most operator judgement. Section 3.5's trade-off reappears here in product form: the fence idiom is where each vendor encoded its answer to "what is the worst thing that can happen?"

### 4.2 What the Families Share

Despite the vocabulary, all five share a structure that is worth stating because it is what makes cross-product reasoning possible at all:

1. **A membership layer that is separate from the resource layer.** Oracle splits CSS from CRS. Pacemaker delegates membership to Corosync and keeps the resource manager to itself. PowerHA sits on the platform's cluster services. This separation exists because membership must survive the failure of the resource manager, and a single process that did both would take the cluster down when it crashed.
2. **A small, replicated, authoritative store of cluster state.** OCR for Oracle; the CIB for Pacemaker; the cluster database for WSFC; a repository for PowerHA; the cluster configuration for InfoScale. In every case the store is small, it must be readable by every node, and its corruption is a cluster-level event rather than a node-level one.
3. **A resource model with dependencies and ordered start.** Resources exist, they have start/stop/monitor operations, and they have order.
4. **A fence that is invoked by the cluster, not by the failing node.** This is the invariant that separates clusterware from a supervisor process. A supervisor can restart a dead process; only clusterware can act against a node that is still alive.
5. **A log that records the decisions.** Every family produces a cluster log with a transition history. Section 11 argues that this log is the actual deliverable of the whole technology, because it is the only artefact that can prove the decision was made correctly.

### 4.3 What the Families Do Not Share, and Why It Matters Operationally

- **Upgrade models.** Some families support rolling upgrades of the clusterware itself while resources keep running; others require the cluster to be stopped. This determines whether a clusterware patch is a change-window event or a multi-week programme, and it is a bigger operational differentiator than any performance characteristic.
- **Vendor support boundary.** Oracle's documentation is explicit that, from 19c, "the integration of vendor clusterware with Oracle Clusterware is deprecated, and can be desupported in a future release", and recommends aligning the next upgrade to transition off vendor cluster solutions. That is a lifecycle statement about a whole class of composite clusters, and it matters to any estate that still runs one.
- **Observability.** The depth of the transition history, the presence of a resource activity log, and whether the fence history is retained and queryable all vary. Section 11 treats this as a first-class selection criterion, because an unevidenced cluster is an unprovable control regardless of how well it works.

---

## 5. Oracle Clusterware in Depth

Oracle gets its own section because it is the family most banks run under their most consequential workloads, and because its component model is the one most often described from memory and therefore most often described wrongly. **Every factual claim in this section was read at the Oracle 19c documentation this pass** (Oracle Clusterware Administration and Deployment Guide, 19c, E95727-14, July 2026) unless explicitly marked otherwise.

### 5.1 What It Is, and Where It Came From

Oracle Clusterware is described in its own documentation as "portable cluster software that provides comprehensive multi-tiered high availability and resource management for consolidated environments", supporting "clustering of independent servers so that they cooperate as a single system". It "was first released with Oracle Database 10g release 1 (10.1) as the required cluster technology for the Oracle multi-instance database, Oracle RAC".

Two consequences of that origin are worth stating because they explain most of the product's shape:

- **It is not an optional add-on to RAC; it is the thing that makes RAC possible.** The documentation is unambiguous: "Oracle Clusterware is required for using Oracle RAC; it is the only clusterware that you need for platforms on which Oracle RAC operates." RAC's instance membership is granted by Clusterware, not negotiated by the database.
- **It has been generalised beyond the database.** The same software manages non-database resources, and Oracle positions it as "the high-availability and resource management framework for all applications on any major platform". In a bank, this is why a Clusterware cluster often runs more than RAC: listeners, VIPs, application processes and agent-managed services all become resources.

### 5.2 The Two Stored Components

This is the part most often got wrong, and the documentation is precise about it. Besides the binaries, Clusterware has **two stored components**:

> "Oracle Clusterware has two stored components, besides the binaries: The voting files, which record node membership information, and the Oracle Cluster Registry (OCR), which records cluster configuration information. Voting files and OCRs must reside on shared storage available to all cluster member nodes."

The two are collectively called **Oracle Clusterware files**, and the division of labour is clean:

| Component | What it holds | Why it is separate |
|---|---|---|
| **Voting files** | Node membership. "Oracle Clusterware uses voting files to determine which nodes are members of a cluster." The same chapter states that Clusterware "uses voting files to provide fencing and cluster node membership determination". | Membership must be arbitrable by a majority, so it lives in a small, replicated, shared artefact that can be read by every node and counted. |
| **OCR (Oracle Cluster Registry)** | Cluster configuration: "information about the components that Oracle Clusterware controls, such as Oracle RAC databases, listeners, virtual IP addresses (VIPs), and services and any applications", stored "in a series of key-value pairs in a tree structure". | Configuration must be *consistent*, not counted. It is updated transactionally through supported utilities and replicated across up to five locations. |
| **OLR (Oracle Local Registry)** | Node-specific Clusterware configuration and dependency information, on local storage; default path `Grid_base/crsdata/host_name/olr/`. | Some state is per-node by nature, and must be readable before shared storage is available - which is precisely the state a booting node is in. |

The OLR's existence is a design tell. A node coming up must be able to start its own stack *before* it can read the cluster's shared state, so it needs a local copy of what it is supposed to do. That is a general pattern in clusterware, not an Oracle peculiarity, and it is why a corrupt local registry can prevent a node from joining a perfectly healthy cluster.

### 5.3 Voting Files: Rules the Documentation Actually States

The 19c guide gives specific, checkable rules. These are worth listing because they are frequently misremembered:

- **Placement.** Voting files can be configured on Oracle ASM or on shared storage. If they are on ASM, "you do not need to manually configure the voting files. Depending on the redundancy of your disk group, an appropriate number of voting files are created."
- **Count when not on ASM.** "For high availability, Oracle recommends that you have a minimum of three voting files on physically separate storage. This avoids having a single point of failure. If you configure a single voting file, then you must use external mirroring to provide redundancy."
- **Upper bound.** "Oracle recommends that you do not use more than five voting files, even though Oracle supports a maximum number of 15 voting files." The gap between the recommendation and the limit is the interesting part: more voting files means more arbitration paths but also more I/O to a latency-sensitive artefact.
- **Dynamic change.** "If necessary, you can dynamically add or replace voting files after you complete the Oracle Clusterware installation process without stopping the cluster."
- **Backup.** "Oracle Clusterware automatically backs up voting file data in OCR as part of any configuration change and automatically restores the data to any voting file you add."
- **If all voting files are corrupted**, the documented procedure is to restore them - which is the cluster-level failure the multiple-copies rule exists to make unlikely.

Note the shape of the guarantee: **the arbitration device is itself replicated, and its contents are backed up inside the configuration store.** That interlock is what makes a total loss of arbitration a recoverable event rather than a rebuild.

### 5.4 OCR: Placement, Migration and the Version Traps

OCR's history is a sequence of placement changes, and getting the version wrong here is exactly the kind of error the brief warns about. The 19c guide records them explicitly:

- **12c: no raw or block devices.** "Oracle Clusterware 12c does not support the use of raw or block devices. To upgrade to Oracle Clusterware 12c from a previous Oracle Clusterware release on which you were using raw or block devices, you must migrate OCR and voting files to Oracle Automatic Storage Management (Oracle ASM) or a shared file system before you upgrade."
- **12.2: shared file system placement desupported.** "In Oracle Database 12c Release 2 (12.2), the placement of OCR and voting disk files directly on a shared file system was desupported."
- **19.3: the desupport is rescinded, but only for Standalone Clusters.** "Starting with Oracle Database 19c (19.3), that desupport is rescinded for Standalone Clusters. For Oracle Domain Services Clusters, you must continue to place OCR and voting disk files in disk groups managed by Oracle ASM."

So the answer to "can I put OCR on a shared file system?" is: at 12.2, no; at 19.3, yes if it is a Standalone Cluster; in a Domain Services Cluster, never. Three different answers for three different version gates, which is why the version column in section 15 is not decoration.

Additional documented constraints:

- **Up to five OCR locations**, each on shared storage accessible to all nodes, and "you can replace a failed OCR location online if it is not the only OCR location".
- **Updates only through supported utilities** - Oracle Enterprise Manager, CRSCTL, SRVCTL, OCRCONFIG, or DBCA. Direct editing is not a supported operation, which is the sort of constraint that exists because someone tried it.
- **Migrating OCR to ASM requires the ASM compatibility attribute set to `11.2.0.2` or later** when upgrading from a previous Clusterware release.
- **The ASM dependency is real and asymmetric.** The documentation notes that "if an Oracle ASM instance fails on any node, then OCR becomes unavailable on that particular node", and that if the CRSD process on that node happens to be the OCR writer and the majority of OCR locations are in ASM, "CRSD stops and becomes inoperable. Cluster management is now affected on this particular node." The very next sentence is the reassurance, and it is worth quoting because it is a precise statement of a blast-radius boundary: "Under no circumstances will the failure of one Oracle ASM instance on one node affect the whole cluster."
- **Do not mix storage types for OCR** simultaneously (ASM and shared file system together), except during a migration. Oracle explicitly does not support it.

### 5.5 ASM as Cluster Substrate

Oracle Automatic Storage Management appears in the Clusterware documentation as "disk management for Oracle Clusterware and Oracle Database", which is the right way to read it for the purposes of this guide: **ASM is not just the database's storage; it is where the cluster's own arbitration and configuration live** in the default modern deployment.

Three facts shape the cluster-level reasoning:

- **ASM is an instance, not a process.** The 19c guide notes that "Oracle ASM is not only one process, but an instance", and that under Oracle Flex ASM "Oracle ASM does not necessarily run on every cluster node but only some of them". This changes the failure model: losing ASM on a node is not the same as losing the node, and losing the node that hosts the ASM instance serving others is a bigger event than losing a node that does not.
- **Redundancy levels drive the arbitration count.** Because voting files placed in ASM are created according to the disk group's redundancy, a normal-redundancy disk group produces a different number of voting files than a high-redundancy one. The cluster's arbitration strength is therefore a consequence of a storage design decision made for other reasons - which is exactly the kind of hidden coupling section 13(a) is about.
- **The database guide owns the rest.** [oracle_database_guide.md](oracle_database_guide.md) covers ASM as a storage technology; this guide uses it only as the substrate on which Clusterware's own files sit.

### 5.6 The Daemons, and What Each Is For

The 19c documentation describes Clusterware as "two separate technology stacks: an upper technology stack anchored by the Cluster Ready Services (CRS) daemon (CRSD) and a lower technology stack anchored by the Oracle High Availability Services daemon (OHASD)". The split matters operationally: the lower stack is what starts the upper stack, so a problem in the lower stack presents as a node that never gets far enough to be managed.

| Component | Process (Linux/UNIX) | Role, as documented |
|---|---|---|
| **CRS** (Cluster Ready Services) | `crsd.bin` (root) | "The primary program for managing high availability operations in a cluster." Manages resources based on configuration in OCR, including "start, stop, monitor, and failover operations", and generates events on status change. |
| **CSS** (Cluster Synchronization Services) | `ocssd.bin`, `cssdmonitor`, `cssdagent` | "Manages the cluster configuration by controlling which nodes are members of the cluster and by notifying members when a node joins or leaves the cluster." The `cssdagent` process "monitors the cluster and provides I/O fencing", a role formerly provided by `oprocd` ("OraFenceService" on Windows). |
| **EVM** (Event Management) | `evmd.bin`, `evmlogger.bin` | "A background process that publishes events that Oracle Clusterware creates." |
| **OHASD** (Oracle High Availability Services) | `ohasd.bin` (root) | The anchor of the lower stack - the daemon that brings up the rest. |
| **CTSS** (Cluster Time Synchronization Service) | `octssd.bin` (root) | Cluster time management. **Deprecated in 19c**; the documentation directs installations to OS-level NTP or chrony instead, verified with `cluvfy comp clocksync`. |
| **GIPC** | `gipcd.bin` | Support daemon enabling Redundant Interconnect Usage. |
| **ONS** | `ons` | Publish/subscribe service for Fast Application Notification events - the path by which the database tells clients that something moved. |
| **GNS** | `gnsd` (root) | Name resolution for cluster-defined names; the reason a cluster can hand out VIPs and SCAN addresses dynamically. |
| **Oracle agent / root agent** | `oraagent.bin`, `orarootagent` | Extend Clusterware to Oracle-specific and complex resources; the root agent manages resources owned by root, such as the network and the Grid VIP. |
| **SYSMON / LOGGER** | `osysmond.bin`, `ologgerd.bin` | Metric collection and persistence into the Grid Infrastructure Management Repository; the logger runs on only two nodes in a cluster. |

Two details in that table carry more weight than their size suggests.

**`cssdagent` is where the theory becomes an outage.** The documented sentence - "a `cssdagent` failure may result in Oracle Clusterware restarting the node" - is the fence in its Oracle form. The component that provides I/O fencing is also the component whose failure causes the node to restart, because a node that cannot fence must not be trusted to keep running. That is section 3.4's principle implemented as a process supervisor's policy.

**ONS is the boundary-crossing mechanism.** Everything in this guide so far has been about the cluster deciding and enforcing. ONS is how the *application* learns about the decision quickly enough to react - which is why section 8's chain ends at application reconnect rather than at resource start, and why section 13(f) exists.

### 5.7 The Interconnect

Clusterware requires a second network:

> "A node that is part of a cluster requires a second network. This second network is referred to as the interconnect. For this reason, cluster member nodes require at least two network interface cards: one for a public network and one for a private network. The interconnect network is a private network using a switch (or multiple switches) that only the nodes in the cluster can access."

Two operational notes from the same page:

- **Redundancy is a requirement, not a suggestion, for HA clusters**: "at least two network interfaces for the public network, bonded to provide one address" and "at least two network interfaces for the private interconnect network".
- **Crossover cables are not supported** as Clusterware interconnects. A direct cable between two nodes is a single point of failure and an unsupported one, which rules out a shortcut that looks attractive in a two-node design.

The interconnect is also the physical basis of the split-brain problem in this product family. It carries the membership traffic, and in a RAC deployment it carries the cache-fusion traffic as well - so a degraded interconnect is simultaneously a quorum risk and a performance event, and the two symptoms appear together.

### 5.8 Why This Family Dominates Tier-0 Banking

Four reasons, none of them "because it is better":

1. **The database it exists to serve is the one the bank already runs.** A tier-0 core banking or card-management system on Oracle is not going to be re-platformed to obtain a different clusterware. The clusterware comes with the database, and the database is the reason the cluster exists.
2. **The single-writer-to-many-readers model is a consistency asset.** RAC gives one database with several instances over one set of data files. There is no application-level reconciliation, no conflict resolution, no eventual consistency window. Section 7 argues that this property is the real reason the family persists.
3. **The component model is old enough to be understood by an auditor.** Voting files, OCR, ASM and eviction are documented in the same book for fifteen years, and the names have not moved. That stability is worth a great deal in a change-controlled institution.
4. **The exit is expensive and everyone knows it.** That is not a technical argument, but it is a real one, and pretending otherwise would make this guide useless. Section 7 treats it as a legitimate architectural constraint rather than a failure of imagination.

### 5.9 Where the Version Traps Are

A short list, because the guide's credibility depends on it:

- **Flex Clusters by default from 12.2.** "In Oracle Clusterware 12c release 2 (12.2), all clusters are configured as Oracle Flex Clusters", with one or more Hub Nodes; the documented range for Hub Nodes is "at least one and can be as many as 64". Pre-12.2 clusters are "converted in place as part of the upgrade process, including the activation of Oracle Flex ASM (which is a requirement for Oracle Flex Clusters)". So a 12.1 design document's node topology may not describe a 19c cluster.
- **Vendor clusterware integration deprecated in 19c**, with an explicit recommendation to transition off. Any estate still running a composite vendor-plus-Oracle cluster should treat this as a roadmap item.
- **CTSS deprecated in 19c.** A design that relies on Clusterware managing time is a design on a deprecation path.
- **Node count.** The documentation's footnote states support for "up to 100 nodes in a cluster on configurations running Oracle Database 10g release 2 (10.2) and later releases", with Flex Clusters adding a large number of hub-supported nodes. Confirm against the release's own limits before designing to a number.
- **OCR on shared file system: 12.2 no, 19.3 yes for Standalone Clusters, never for Domain Services Clusters.** See section 5.4.

---

## 6. The Other Families, Accurately and Briefly

Enough that a reader recognises each one, verified where verification was possible, and not padded. **Where the vendor's documentation host could not be reached this pass, the section says so and describes the class of behaviour instead of inventing specifics** - the entries concerned are carried forward into section 15.2.

### 6.1 Pacemaker and Corosync

**What it is.** "Pacemaker is a high-availability cluster resource manager - software that runs on a set of hosts (a cluster of nodes) in order to preserve integrity and minimize downtime of desired services (resources)", maintained by the ClusterLabs community (Pacemaker Explained, 3.0, section 1.2). It is the reference open-source implementation of the discipline, and it is embedded in commercial products - IBM MQ's RDQM ships it, and the RHEL High Availability Add-On packages it.

**Structure, as documented.** The cluster stack has five parts: resources, resource agents ("scripts or operating system components that start, stop, and monitor resources"), fence agents, "cluster membership layer" (Corosync, which "provides reliable messaging, membership, and quorum information about the cluster"), the resource manager, and the tools. Pacemaker itself is seven daemons: `pacemakerd`, `pacemaker-attrd`, `pacemaker-based`, `pacemaker-controld`, `pacemaker-execd`, `pacemaker-fenced`, `pacemaker-schedulerd`.

The decision-making structure is worth understanding because it states the pattern explicitly: the cluster elects a **Designated Controller**, and "the DC responds to cluster events by taking a current snapshot of the CIB, feeding it to the scheduler, then asking the executors ... and the fencer to execute any necessary actions". Configuration lives in the **CIB**, an XML representation of "the cluster's configuration and the state of all nodes and resources", kept synchronised by `pacemaker-based`.

**Fencing.** Defined in section 1.2 as quoted in section 2.3 above, with the flat statement that "a cluster cannot safely recover from certain failure conditions, such as an unresponsive node, without fencing". Fence devices are modelled as "a special class of resource". The `pacemaker-fenced` daemon "handles requests to fence nodes. Given a target node, the fencer decides which cluster node(s) should execute which fencing device(s)".

**Quorum policy.** Documented exhaustively in section 3.4 of Pacemaker Explained, with the values and version gates given in section 3.6 above. The default is `stop`.

**SBD.** The ClusterLabs SBD project describes itself as "a highly reliable fencing or Shoot-the-other-node-in-the-head (STONITH) mechanism that works by utilizing shared storage", working with Pacemaker clusters and "currently known to compile and function on Pacemaker 1.1.7+ or 2.0.0+ and corosync 1.4.x, 2.3.x or 3.0.x". Pacemaker's own options expose it: `have-watchdog` is "set automatically by the cluster according to whether SBD is detected to be in use", and `fencing-watchdog-timeout` governs "watchdog-based self-fencing ... via SBD". The documented warning attached to that option is the kind of detail that separates a real design from a template: when used, the timeout "must be larger than `SBD_WATCHDOG_TIMEOUT` on all nodes that use watchdog-based SBD, and Pacemaker will refuse to start on any of those nodes where this is not true", and a misconfigured negative value "could" cause "data corruption or loss".

**What a reader should remember.** Pacemaker is the family that makes the trade-off *explicit and configurable*. That is a strength for a competent operator and a hazard for an organisation that wants a default. A cluster with `no-quorum-policy: ignore` and `fencing-enabled: false` is a documented, supported, and completely unprotected configuration - and nothing in the software will stop you.

**Not verified this pass:** the RHEL High Availability Add-On's exact packaging matrix (which Pacemaker and Corosync majors ship with which RHEL release). The `docs.redhat.com` host returned a fetch error on every attempt; see section 15.2. **Confirm the packaged versions against the RHEL release you deploy rather than from this guide.** The same applies to the `corosync-qdevice` / `corosync-qnetd` quorum device, which is described here at class level only.

### 6.2 IBM PowerHA SystemMirror

**What it is.** IBM's availability product for AIX and IBM i, and the successor name to HACMP - the rename is old enough that documentation and practitioners use both, and a design review that does not know they are the same lineage will misread an estate inventory.

**What can be said at class level, and is standard for the product line:** PowerHA clusters use a **repository disk** as the shared arbitration and configuration artefact; they group resources into **resource groups** with startup, fallback and failure policies that determine what happens to the group when a node leaves; and they build on the platform's own clustering services - **RSCT** (Reliable Scalable Cluster Technology) for group services and **CAA** (Cluster Aware AIX) for the AIX-level cluster communications and repository. Fencing is expressed through the resource group's failure policy plus storage and node-level actions, rather than through a separate STONITH-style command set.

**Verification status.** This entry is class-of-behaviour only. `www.ibm.com/docs` (including `/docs/en/powerha-aix/7.3.0`) returned a fetch error on every attempt this pass, and a web search returned no results. **Do not take command names, repository-disk sizing rules, or policy option names from this guide.** The entry exists so that a reader recognises the family and knows what to verify; it is flagged in section 15.

### 6.3 Veritas InfoScale (formerly Veritas Cluster Server)

**What it is.** A cross-platform availability product with a strong storage-integration heritage, found in heterogeneous UNIX and Linux estates where the storage layer is also Veritas. Its vocabulary is distinctive and worth knowing: **service groups** rather than resource groups; **GAB** (Group Atomic Broadcast) as the cluster's membership and messaging transport; **LLT** (Low Latency Transport) as the dedicated low-level network transport underneath it; and **I/O fencing** using **coordinator disks** to arbitrate which node may write.

**Why the I/O-fencing idiom matters conceptually.** Veritas's approach fences the *write path* rather than only the node - a host that has lost the arbitration is prevented from writing to shared storage even if it is still running. This is a direct, product-level answer to the fourth row of section 3.2's table (the node that is alive but cannot see its peers, or can see them but cannot see storage): revoking the I/O path is a defence that does not depend on the node's cooperation or on its network reachability.

**Verification status.** Class-of-behaviour only. `sort.veritas.com/documents` resolves to a document-search interface rather than to document text, `sort.veritas.com/public/documents/...` returned a 404 for the path attempted, and `docs.veritas.com` was blocked by the fetch layer. **The GAB/LLT/coordinator-disk description above is the class of behaviour, not a verified quotation**, and it is flagged in section 15.

### 6.4 Microsoft Failover Clustering

**What it is.** Windows Server's built-in clustering, documented as applying to Windows Server 2016, 2019, 2022 and 2025 (and Azure Local). "Failover clustering is a powerful strategy to ensure high availability and uninterrupted operations in critical environments. It involves a configuration of independent computers, known as nodes, which work together to enhance the availability and scalability of applications and services, now referred to as clustered roles."

**Quorum, as documented.** "Cluster nodes collaborate to maintain what is known as a quorum, which is essentially the minimum number of votes from cluster members required for the cluster to function correctly. This mechanism prevents split-brain scenarios, where split portions of a cluster might try to operate independently, potentially causing inconsistencies." The documented models are **Node Majority**, **Node and Disk Majority**, **Node and File Share Majority**, and **No Majority (Disk Only)**. Witness types documented in the quorum-witness article are disk witness, file share witness and cloud witness; the cloud witness requires an Azure storage account, creates an `msft-cloud-witness` container, and needs outbound HTTPS on port 443. Dynamic quorum is documented for later releases: clusters "automatically manage the witness vote and their nodes vote with Dynamic Quorum".

**Storage, as documented.** Cluster Shared Volumes give "multiple nodes in a Windows Server failover cluster ... simultaneous ... read-write access to the same LUN (disk) that is provisioned as an NTFS volume", with ReFS supported but the CSV placed in *redirected mode* ("write access is sent to the coordinator node"). The documented benefit is directly relevant to this guide's failover chain: "clustered roles can fail over quickly from one node to another node without requiring a change in drive ownership, or dismounting and remounting a volume."

**Networking and health.** Clusters "often employ dedicated private networks for internal functions such as heartbeat signals and cluster management, while separate public networks handle client access and application data", and "the cluster's health is continuously monitored through heartbeat signals".

**What a reader should remember.** WSFC is the family that treats quorum as a *menu of topologies* rather than as a fixed mechanism, because Windows clusters are deployed in every shape. Its CSV design also blurs the layer boundary this guide draws elsewhere: with CSV, multiple nodes have concurrent write access to the same volume, with coordination rather than exclusive ownership - a different concurrency model from the single-writer assumption that Oracle's and Veritas's fencing idioms assume.

**Not fully verified this pass:** the internal resource-hosting process architecture (the Resource Hosting Subsystem that loads resource DLLs) and the current default values for heartbeat and threshold parameters. Neither appears in this guide as a specific number or name.
---

## 7. Why a Bank Runs This

The two lazy answers are "banks are conservative" and "the cloud replaced this". Neither survives contact with an actual estate. Banks run clusterware because three specific properties are hard to obtain otherwise, and they keep running it because the change to something else carries a data-consistency risk the institution will not take.

### 7.1 Tier-0 Systems Predate the Container Estate

The core ledger, the card-management platform, the payments hub and the general ledger are not new systems. They were built when the deployment unit was a server, the failover unit was a node, and the network was a private interconnect. Their availability design is therefore node-granular because *that was the only granularity available* when the design was fixed. Nothing about that is a failure of imagination; it is what it means to run a system that predates the technology now being proposed to replace its availability model.

The corollary matters for planning: **you cannot retrofit pod-level availability onto a system whose transactional model assumes a single writer and a shared cache**, because the assumption is in the database engine, the schema, the locking model and often the application code. What you can do is wrap it - run it on a scheduler, run its database on clusterware, and accept that the availability properties come from below and the deployment properties come from above. That hybrid is the actual state of most large bank estates, and section 10 argues it is a coherent destination rather than a failure to modernise.

### 7.2 A Shared-Everything Database Cluster Has a Consistency Story a Distributed System Cannot Yet Match for the Same Workload

This is the strongest argument and it is usually stated badly, so here it is precisely.

A clustered shared-everything database - RAC being the canonical banking example - presents **one database, one set of data files, one writer set coordinated through a cache-fusion layer, and one truth**. A transaction that commits is visible to every other session immediately, in the same order, with the same isolation semantics. There is no replication lag, no conflict resolution, no read-your-writes problem across nodes, and no application-level reconciliation.

A distributed or active-active database achieves availability by **replicating and reconciling**. That is a legitimate and increasingly attractive design, but it moves a set of hard problems into the application and the operational model:

| Property | Shared-everything cluster | Replicated/active-active distributed database |
|---|---|---|
| Number of truths | One | One logical, several physical, converged by consensus or reconciliation |
| Conflict handling | Not required: one writer set arbitrates | Required, or avoided by partitioning the write set |
| Read consistency | Immediate | Immediate, stale, or session-consistent depending on configuration |
| Failure consequence | Instances fail; data stays single | A partition can require a choice between availability and consistency |
| Application burden | None added | Idempotency, conflict semantics, retry and reconciliation are now the application's problem |
| Where the guarantee lives | In the database engine and the cluster | In the consensus protocol and the topology |

For a workload whose invariant is "the ledger must have exactly one current state, and every reader must see it", the left column is not nostalgia. It is a smaller number of places for an inconsistency to live. **A bank that moves from left to right has not removed the hard problem; it has relocated it into the application, where it is harder to test and much harder to evidence to an auditor.**

### 7.3 The Change-Control Burden Is Accepted Because the Alternative Is Worse

A cluster is expensive to operate: patching is a project, the fence has to be tested, the design has to be documented, and a handful of people have to understand it. Institutions accept that cost because the alternative on the table is not "no cost" - it is a different cost, paid in a currency they weight more heavily:

- **A data-consistency risk they cannot bound.** Corruption's blast radius is not a service; it is a book of record.
- **A migration risk on a system they cannot easily test.** Tier-0 migrations are rehearsed for years precisely because the failure modes are unknown.
- **A skills risk in the other direction.** The cluster skills are scarce, but the distributed-database-with-conflict-resolution skills are scarcer in a bank that has never operated one.

### 7.4 Where Banks Have Actually Moved, and What Was Traded

Honesty requires naming the exits, because they are real:

- **To a replicated or active-active database.** The property traded is single-writer simplicity. What is gained is that the availability decision moves from a node-level fence to a consensus protocol, which is in some respects a *better* decision procedure - it is quorum-based by construction and does not depend on an out-of-band power device. What is paid is conflict handling, a new operational model, and the loss of the "one truth, no reconciliation" property for the workloads that genuinely needed it.
- **To a managed cloud platform service.** The property traded is control of the fence. The institution no longer owns the mechanism that decides which node survives; it owns an SLA and a shared-fate boundary with a provider. That can be a good trade - a provider with a real quorum-backed control plane may operate the decision better than the bank did - but the *evidence* obligation does not transfer. The bank still has to demonstrate to its supervisors that the decision is made correctly; it now has to demonstrate it about a system it cannot inspect. Section 12 treats that as the central unresolved problem of the regulated-cloud era.
- **To a container platform with a stateful operator.** The property traded is node-granular simplicity for declarative placement. Section 10.

In each case the guide's thesis holds: the failure was converted into a decision, and the decision moved. What changed is *who owns the decision* and *whether the institution can still produce evidence of it*. A migration that loses the evidence has traded a control for a service, and should be recorded as such.

---

## 8. The Determinants of Failover Time

**No total appears in this section, and none should appear anywhere.** A failover time is a property of a specific configuration on specific hardware under specific load, and any figure quoted without those qualifiers is either a lab measurement or a marketing claim. What can be said precisely is *what the total is made of*, in order, and which parts the institution controls.

### 8.1 The Chain

**Link 1 - failure detection.** The interval between the failure and the moment the cluster believes it happened. Determined by the heartbeat or health-check interval multiplied by the number of missed intervals tolerated, or by a database-level health check's own frequency. Every product exposes these as configurable parameters; none of them has a universal correct value. This is the first place where the institution's own choices set the floor on the total.

**Link 2 - the membership and quorum decision.** Once a node is missing, the survivors must establish that they are entitled to act. This involves the membership protocol's own timeout for declaring a node gone, and a quorum computation that may require contacting a witness device or reading an arbitration LUN. In Pacemaker terms this is the controller and Corosync layer, with documented configuration such as `dc-deadtime` (default 20s) governing how long the cluster waits for responses during a controller election. That is a configuration default, not a failover time.

**Link 3 - fencing execution and its wait.** The survivors must remove the loser's ability to write *before* starting anything. This is a round trip to an out-of-band device plus the device's own action time. Pacemaker exposes `fencing-timeout` with a documented default of 60 seconds for the fence action to complete, and `fencing-max-attempts` with a documented default of 10 before the cluster stops immediate re-attempts. Those are defaults for a phase, not a total - but they illustrate why the fence is often the largest single term in the sum, and why "we cannot afford the fence time" is usually an argument for a faster fence device rather than for no fence.

**Link 4 - resource stop, then resource start.** The old owner's resources are stopped (with a stop timeout that must be long enough to be safe and short enough to be tolerable), and the new owner's resources are started in dependency order. Database startup is usually the dominant term here, and it is dominated by recovery work rather than by process launch.

**Link 5 - VIP and address movement.** The address is bound on the new node, and the network must learn about it: gratuitous ARP or equivalent for IPv4, neighbour advertisement for IPv6, and whatever the load balancers and firewalls in the path do with a changed MAC-to-IP binding. This link is invisible in cluster logs and is a classic cause of "the cluster failed over but nobody could connect".

**Link 6 - storage or cluster filesystem remount.** If the design requires unmounting and remounting, or if a cluster filesystem must re-establish its locks, that work lands here. Designs that avoid the remount - Cluster Shared Volumes' documented property that roles "fail over ... without requiring a change in drive ownership, or dismounting and remounting a volume" is the clearest example - remove this link from the chain entirely.

**Link 7 - database instance recovery or crash recovery.** The database must recover to a consistent state before it can accept work: rolling back uncommitted transactions, rolling forward redo, resolving in-doubt distributed transactions. This term scales with the work in flight at the moment of failure, which is why a failover on a quiet Sunday is not evidence about a failover at month-end.

**Link 8 - application reconnect, and connection-pool drain.** The application tier must notice that its connections are dead, discard them, and establish new ones - or, better, be told by the cluster that they are dead. Oracle's Fast Application Notification through ONS exists precisely to shorten this link. Connection pools that hold dead connections, retry storms, and applications that require a manual restart are all failures of this link, and they are failures the cluster cannot fix.

### 8.2 What the Chain Implies

- **The total is additive across eight links, and the institution controls six of them directly.** Only links 1 and 3 are substantially product-determined, and even those are parameterised.
- **Optimising one link in isolation is usually counterproductive.** A faster detection with an unchanged fence produces more false fences, not faster recovery. A faster fence with an application that cannot reconnect produces a clean cluster log and an outage that lasts until an operator intervenes.
- **The link that gets measured is rarely the link that dominates.** Institutions measure "time to resource start" because the cluster logs it, and then discover that customers experienced a much longer outage caused by link 5 or link 8 - neither of which appears in the cluster log at all. Section 11 argues that the exercise must therefore be measured end-to-end at the application, not inside the cluster.

### 8.3 What Not to Do With This Section

Do not add the documented configuration defaults together and present the result as a failover time. They are defaults for individual phases in a reference configuration, they are frequently overridden, and the phases interact - the fence wait ends when the device responds, not when the timeout expires. **The only defensible number for a given estate is the one that estate has measured, end to end, on the workload it actually runs** - and even then, only for the failure mode that was exercised.

---

## 9. The Operational Reality

A cluster is a system that is only ever touched twice: when it is built and when something goes wrong. Everything between those two events is where the interesting failures live.

### 9.1 Patching a Live Cluster

Patching clusterware is not patching a server. The cluster is a distributed agreement, so a member running a different version may be unable to form a membership with its peers, or may form one and behave subtly differently. Two consequences:

- **The vendor's rolling-upgrade capability determines the shape of the change.** Some products support upgrading clusterware while resources keep running; others require the cluster to be stopped. Where a rolling path exists, it is usually constrained - one node at a time, a specific order, a compatibility window during which mixed versions are supported. Those constraints are the actual content of the change plan, and they are release-specific. **Read them at the vendor for the release you are moving to**; this guide deliberately states no version's rolling-upgrade procedure.
- **The fence is the thing that makes rolling work.** Taking a node out of the cluster deliberately and bringing it back is only safe because the cluster can fence it if it misbehaves on the way back. A rolling upgrade in a cluster with fencing disabled is a procedure with no safety net, which is why the two changes should never be combined in one window.

### 9.2 The Change Board

Cluster changes are hard to approve because their failure mode is not proportional to their apparent size. Changing a heartbeat interval looks like a one-line configuration edit and is actually a change to the institution's false-fence rate, its detection latency, and its split-brain exposure. A change board that reviews cluster changes as "configuration" rather than as "resilience policy" is the mechanism by which bad parameters reach production.

The defensible practice is to make the cluster's decision parameters **explicit, documented, reviewed and owned** - the quorum policy, the fence policy, the detection parameters, the timeout values, and the rationale for each - so that a change to any of them is visibly a change to a control. Section 12 calls this the control register; section 15's audit table is what one looks like when it is written down.

### 9.3 The "Never Touch It" Problem

There is a well-known pattern in bank infrastructure: a cluster kept at an old release because the change risk of patching it exceeds the risk of leaving it. The reasoning is locally rational - it works, it has been stable for years, and every previous patch window produced an incident. The aggregate consequence is an estate of clusters at versions the vendor no longer patches, running on operating systems the vendor no longer certifies, understood by people who are leaving.

Three things make this worse than it looks:

- **The cluster's own evidence decays with it.** A cluster at an unsupported level cannot raise a vendor case when the fence misbehaves, so the failure mode that section 3 says matters most becomes undiagnosable.
- **The compatibility surface grows.** Storage firmware, multipathing, the database, the OS and the clusterware are mutually certified in a matrix. Freezing one member of the matrix while the others move is what produces the "the vendor will not support this combination" outcome in section 13(d).
- **The knowledge leaves with the person.** Section 9.5.

The honest counter-argument is that a well-run frozen cluster is safer than a badly-run current one, and it is true. The problem is that "well-run" and "frozen" are in tension: a frozen cluster accumulates no operational evidence, because nothing about it is ever exercised.

### 9.4 The DR Test That Proves Nothing

Three test patterns recur, and none of them proves what the institution thinks it proves:

1. **The switchover dressed as a failover.** The operator stops the resource group cleanly, the group moves, everything comes up in order, the test passes. This proves that the *planned* path works. It says nothing about the fence, because a clean stop requires no fence; nothing about quorum, because no partition occurred; and nothing about detection, because nothing was undetected. It is a rehearsal of the runbook, which is valuable, and it is not a failover test.
2. **The failover with the fence disabled.** Frequently performed because the fence is slow, or because the test window is short, or because someone is afraid the fence will take down a node that the test needs. This test proves that the resource manager can start resources elsewhere. It proves nothing about whether the old owner stopped writing, which is the only question that matters. It is a test of the half of the mechanism that was never in doubt.
3. **The test that was announced.** Everyone knows the window, the application team has drained the connections, the batch has been rescheduled, and the traffic is a synthetic script. This is a legitimate way to find runbook defects and an illegitimate way to demonstrate resilience, because the real event will not come with a calendar invitation.

### 9.5 The Skills Question

**The person who can read a cluster log is retiring, and there is no obvious successor.**

Cluster logs are the least popular artefact in infrastructure. They are verbose, they are timestamped across nodes with imperfect clock agreement, and the interesting events are interleaved with noise from recurring monitor operations. Reading one requires knowing what the cluster *should* have done, which requires the design intent, which is usually in a document nobody has opened since the build.

The consequence is specific and predictable: when the cluster behaves unexpectedly, the institution cannot distinguish "the cluster made a wrong decision" from "the cluster made a right decision about a system that was already broken". That distinction is the entire content of the post-incident review, and without the skill it is resolved by whichever narrative is most comfortable - usually "the cluster failed", which is the one explanation that leads to no corrective action.

Mitigations that actually work are unglamorous: writing the design intent down next to the configuration, keeping the decision parameters in a register, rehearsing log reading against a known-bad case rather than a known-good one, and treating the fence test as a knowledge-transfer exercise rather than a compliance box. Section 11 formalises the last of these.
---

## 10. What Containers Did and Did Not Replace

The relationship between clusterware and a Kubernetes-based platform is not a contest. They operate at different layers, they make different decisions, and the failure of most comparisons is that they answer different questions and then argue about the answers.

### 10.1 What the Orchestration Layer Genuinely Does

Be precise about the achievement, because it is substantial:

- **Declarative desired state.** The operator declares how many replicas should exist and where they should be allowed to run; the control plane continuously reconciles reality to that declaration. This is a genuine advance over imperative resource scripts.
- **Per-pod placement and rolling replacement.** The scheduler places individual pods, respects taints, tolerations, affinity and topology spread, and can replace a whole fleet incrementally without the operator touching a node. Clusterware has no equivalent at this granularity - it moves a node's entire workload set.
- **Self-healing of workload objects.** A failed pod is replaced, a failed node's pods are rescheduled subject to disruption budgets.
- **A control plane with its own quorum.** The API server's backing store (etcd) is a quorum-replicated log, and the control plane's availability depends on that quorum in exactly the way section 3 describes. This is worth noting because it means the platform *contains* a clusterware problem even as it presents itself as an orchestrator: etcd's own membership and quorum is a majority rule with a fencing-adjacent story, and [openshift_workload_availability_guide.md](openshift_workload_availability_guide.md) owns that discussion.
- **Node draining as a first-class operation.** Graceful eviction of workloads before maintenance is a genuinely better operational primitive than "stop the resource group".

### 10.2 What It Does Not Do

Four gaps, stated in the terms this guide has been using:

**1. It does not fence a node out of shared storage.** A scheduler that declares a node unhealthy and reschedules its pods has done nothing to prevent that node from continuing to write to a LUN it can still reach. Pod eviction is not a fence: it removes a *workload object*, not a node's *ability to write*. Products that manage stateful workloads on a container platform address this with their own fencing and with storage-level controls, which is an admission that the problem is below the scheduler.

**2. It does not arbitrate a single-writer database cluster.** A shared-everything database with one data file set and one writer set needs a decision about which *instance set* may open the data. That decision is made by the database and its clusterware, not by the scheduler. Running the database in a pod does not change this: the pod is a packaging unit, and the arbitration still happens below or beside it.

**3. It does not provide the same consistency guarantee for one logical database.** Rescheduling replicas preserves the *number* of replicas, not the *consistency* of the data they hold. If the application's invariant is "one current state, visible to all readers", the orchestrator has no opinion about it.

**4. It does not give a node-granular membership view outside the kernel's control.** Clusterware's membership is deliberately outside the operating system's own view of itself - that is the point of an out-of-band fence and a separate membership layer. A container platform's view of a node is mediated by an agent on that node, which is exactly the dependency that a partitioned or wedged node breaks.

### 10.3 The Honest Synthesis

The defensible statement is this: **the scheduler replaced the deployment model; it did not replace the data-safety model.** For stateless tiers, the orchestrator is a clear improvement and clusterware has nothing to add. For stateful tiers with a single writer set, the orchestrator provides packaging, placement and lifecycle, while the availability decision remains with the database and its clusterware - or moves to a consensus protocol *inside* the database, which is a different answer to the same question rather than an elimination of it.

The hybrid that results is not a transitional mess. It is a coherent architecture:

| Layer | Owns | Decides |
|---|---|---|
| Orchestrator | Pod placement, replica count, rolling replacement, node drain | Where the workload runs |
| Clusterware / database cluster | Membership, quorum, fence, instance start order | Which node may write |
| Application | Reconnect, retry, idempotency, reconciliation | What to do when the answer changes |

A bank that can describe its estate in those three rows has understood the boundary. A bank that says "we moved it to Kubernetes so the clustering problem went away" has moved the problem and stopped looking at it.

---

## 11. The Evidence Question

**The audit question is not "is it clustered". It is "can you show the decision was made correctly".** This section is about what that evidence looks like, and why most institutions cannot produce it.

### 11.1 What Is Logged

Every family produces records, and the records differ in name and depth:

- **The cluster log / transition history.** The sequence of membership changes, resource operations and their outcomes. Pacemaker records transitions in its own log and keeps scheduler inputs; Oracle keeps a **cluster resource activity log** in the Grid Infrastructure Management Repository, which its documentation describes as providing "precise and specific information about a resource failure, separate from diagnostic logs", with writes "tagged with an activity ID and any related data gets the same parent activity ID ... creating a hierarchy of activity IDs" so that a failure can be traced across nodes. That hierarchical tagging is precisely what a post-incident review needs and rarely gets.
- **Eviction and fence records.** Which node was fenced, when, by which device, and with what result. This is the single most important record in the whole system, because it is the only proof that the decision was enforced rather than assumed.
- **Resource transition history with timings.** When each resource stopped and started, on which node, and how the operation terminated.
- **The configuration itself, versioned.** A cluster's decision parameters change over time, and an audit of a past event against today's configuration is meaningless.

### 11.2 What an Exercise Must Demonstrate

An exercise that produces evidence has four properties, and dropping any one of them reduces it to a rehearsal:

1. **Unplanned.** Nobody in the application team knows the moment. The failure is injected without a drain, without a batch hold, and without a traffic ramp-down.
2. **Unannounced in mechanism, if not in window.** The team may know the window; they must not know which node, which method, or which moment. A test that names its victim tests the runbook, not the cluster.
3. **The fence actually fired.** The evidence must show a fence record, not merely a resource start elsewhere. If the fence did not fire, the test proved that the old owner was not writing - which was true only because the test arranged it.
4. **The application reconnected without manual intervention.** Measured at the application, end to end, including the connection pool, the VIP propagation and the retry behaviour. A cluster that recovers and an application that does not is a failed exercise with a green cluster dashboard.

### 11.3 Why a Successful Failover Proves Less Than an Unsuccessful One Teaches

This is the counter-intuitive core of the section, and it follows directly from section 3.

**A successful failover is consistent with at least three different world states:** the cluster made the right decision and enforced it; the cluster made the right decision and got lucky because the old node happened not to write; or the cluster made a wrong decision and nothing bad followed because the workload was idle. The log looks the same in all three cases.

**A failed failover is diagnostic.** It tells you which link in section 8's chain broke, and it does so under conditions you can inspect. The most valuable exercises in a mature institution are the ones that found something - a fence that timed out, a VIP that did not propagate, a pool that held dead connections - because those are the findings that would otherwise have been discovered during a real event with customers watching.

The institutional implication is uncomfortable: **a test programme measured by pass rate is measuring the wrong thing.** A programme with a hundred passes and no findings has either an unusually mature estate or, far more likely, tests that cannot fail.

### 11.4 The Evidence Pack

What a bank should be able to hand a reviewer, per cluster, without a project:

| Artefact | What it proves |
|---|---|
| The design decision record: quorum policy, fence policy, detection parameters, and the rationale | That the tie-break was a decision, not an accident |
| The configuration as deployed, versioned and dated | That the decision in the record is the decision in production |
| Fence and eviction history for the period | That the enforcement mechanism operates |
| Resource transition history for the period | That failovers happen and complete |
| The last unplanned exercise report, end to end, with timings and findings | That the estate has been tested against the failure it is designed for |
| The application reconnect evidence from that exercise | That the boundary between cluster and application is intact |
| The skills and ownership record | That the evidence can be interpreted by someone still employed |

Five of those seven artefacts are things a cluster generates. Two - the decision record and the skills record - are things an institution has to write, and they are the two most often missing.

---

## 12. The Regulated and Banking Angle

**No penalty figures and no clause numbers appear here.** What follows is the class of obligation a cluster serves, described in the vocabulary of operational resilience rather than of any particular rulebook. The rulebook detail is owned by [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md).

### 12.1 The Cluster as a Control

In an operational-resilience frame, a control is a measure that is designed, operated, monitored and evidenced. Test a cluster against that definition and the gaps appear immediately:

| Control attribute | What it requires of a cluster | Where clusters typically fail |
|---|---|---|
| Designed | A documented decision about quorum, fence and detection, with rationale | The parameters exist but the rationale does not; they were set at build and never reviewed |
| Operated | The design is the running configuration, and changes go through change control | Drift: a fence disabled during an incident and never re-enabled (section 13(b)) |
| Monitored | The cluster's own health and the *fence* health are watched | Cluster health is monitored; fence capability is assumed |
| Evidenced | History retained and retrievable for the review period | Logs rotate faster than the review cycle, or are not centralised |

The important shift is that none of these attributes is about whether the cluster works. They are about whether the institution can *show* that it works, which is a different and harder property - and the one the thesis names when it says the decision is a control.

### 12.2 The Obligation Classes a Cluster Serves

Four classes recur across jurisdictions and are safe to describe generically:

- **Operational resilience.** The obligation to identify important business services, set an impact tolerance for each, and be able to stay within it under severe but plausible scenarios. Clusterware is the mechanism by which a node-level scenario stays inside the tolerance - and the scenario testing obligation is what makes section 11's unplanned exercise a regulatory artefact rather than good practice.
- **Change management.** The obligation to manage changes to production systems with proportionate risk assessment, testing, approval and rollback. A cluster parameter change is a change to a resilience control, and should be assessed as such rather than as routine configuration.
- **Recoverability evidence.** The obligation to demonstrate, not assert, that recovery works. This is the class that most directly indicts the switchover-dressed-as-failover pattern in section 9.4: an institution that can only produce clean-switchover evidence has evidence about a scenario that is not the risk.
- **Third-party dependency management.** The obligation to understand and manage reliance on external providers, including cloud platforms. For a cluster in a managed service, this is where the control becomes hard: the decision is made by a provider whose internal mechanism the bank cannot inspect, so the evidence must be contractual and attestational rather than technical.

### 12.3 The Audit Artefact

The artefact an auditor or supervisor actually wants is small and specific. It is not the architecture diagram. It is:

1. **The decision, in writing** - which quorum policy, which fence policy, which parameters, and why those and not the alternatives in section 3.5.
2. **The evidence it is running** - dated configuration, and the change history showing it has not drifted.
3. **The evidence it has been tested** - an unplanned exercise, with a fence record and an application-level timing.
4. **The evidence it is understood** - a named owner and a named deputy who can read the log.

An institution that can produce those four for its tier-0 clusters has answered the audit question. An institution that can produce a diagram and a build document has answered a different question.

### 12.4 The Uncomfortable Regulatory Consequence of Section 3.5

If the tie-break is a policy decision with a data-loss consequence either way, then the institution has made a risk acceptance - and risk acceptances must be approved at the right level, recorded, and revisited. A cluster configured conservatively (no-quorum-policy set to continue, fence disabled for a "critical" window) is an accepted risk of data corruption. Most institutions have made that acceptance without knowing they made it, because the setting was a default, a workaround, or a legacy choice.

**Naming the acceptance is the single most valuable thing a reviewer can do to a bank's cluster estate.** It converts an invisible configuration into a governed decision, which is what the thesis claims clusterware is for.
---

## 13. The Failure Modes

Six patterns account for most real cluster incidents. Each is presented as symptom, cause and guardrail, because the symptom is what the incident review sees and the guardrail is what prevents the next one.

### 13.1 (a) A Quorum Device on the Same Storage as the Data

**Symptom.** A storage event removes the cluster's ability to arbitrate and the data at the same moment. The survivors cannot establish quorum, so nothing restarts; or, worse, the arbitration succeeds on one path while the data path is degraded, and the cluster makes a confident decision on partial evidence.

**Cause.** The arbitration artefact was placed for convenience. A quorum disk on the same array as the data; a voting file in the same disk group as the database; a witness on a file share hosted by the same infrastructure that hosts the workload. The rule that was violated is the one Oracle states explicitly for voting files - "a minimum of three voting files on physically separate storage. This avoids having a single point of failure" - and the same principle applies to every family's witness.

**Guardrail.** Place the arbitration device in a failure domain that does not contain the data, and prove it: a documented failure-domain map showing that no single storage, array, switch, hypervisor host or site can remove both the data and the arbitration. Then test it by degrading the data path and confirming the cluster can still arbitrate.

### 13.2 (b) A Fence That Was Disabled "Temporarily"

**Symptom.** A failover completes; later, the data is found to be inconsistent, and the cluster log shows the old node was never fenced. Or: the cluster runs for months with no fence configured at all, and nobody notices until a real partition.

**Cause.** The fence was disabled during an incident to stop it taking down a node the team needed, and the change was never reversed. This is the most common and most consequential drift in the whole discipline, because it is a silent change of the institution's risk position - from "we prefer integrity" to "we prefer availability" - made by whoever was on shift at 03:00.

**Guardrail.** Three layers. First, monitor fence *capability* as a first-class health check, not just fence events: alert when fencing is disabled, when no fence device is reachable, and when the fence device's own credentials or connectivity have expired. Second, require a change record to disable a fence, with an expiry, so that "temporary" has a date attached. Third, test the reverse: a periodic exercise that verifies a fence would fire, on a node that can be lost. Pacemaker's documentation is explicit about the stakes - with fencing disabled, recovery "starts without any further protection (which can mean data loss if the unresponsive node still accesses shared storage)".

### 13.3 (c) A Failover Nobody Timed

**Symptom.** The cluster fails over correctly. The post-incident review reports "failover successful". Nobody knows how long the service was unavailable, because the measurement was the cluster's resource-start event, and the customers' experience was set by the VIP propagation and the connection pool.

**Cause.** The institution measured the layer that logs, not the layer that matters. Link 5 and link 8 of section 8's chain are invisible to the cluster.

**Guardrail.** Measure end to end, from the application's perspective, in every exercise. Synthetic transaction monitoring that runs against the VIP, timestamped against the injected failure, is the only measurement that answers the question the business asked. Record the result as a distribution over several exercises rather than a single number, because the dominant term changes with the load at the moment of failure.

### 13.4 (d) A Cluster So Customised the Vendor Will Not Support It

**Symptom.** An incident escalates to the vendor, and the first response is a request for the configuration; the second is a statement that the configuration is not supported. The institution is now debugging a tier-0 outage without support, at the worst possible moment.

**Cause.** Years of local customisation: hand-edited cluster state, unsupported storage combinations, third-party multipathing outside the certification matrix, clusterware versions outside the vendor's compatibility list, resources started by cron alongside the cluster's own management. Oracle's documentation names one instance of this class directly, in a statement that should be read as a warning to every estate that composites clusters: as of 19c, "the integration of vendor clusterware with Oracle Clusterware is deprecated, and can be desupported in a future release", with an explicit recommendation to transition off. Any such estate is one release away from an unsupportable configuration.

**Guardrail.** Maintain a supportability matrix per cluster - clusterware version, OS, storage firmware, multipathing, database, and the vendor's certification status for the combination - and review it at every upgrade cycle rather than at every incident. Treat any local change that the vendor's documentation does not describe as a supportability risk with a named owner.

### 13.5 (e) A DR Plan That Assumed a VIP Nobody Configured

**Symptom.** The DR test succeeds at the cluster layer and fails at the network layer. The database is up at the recovery site; the application is pointing at an address that does not exist there; the runbook's step 12 says "verify the VIP" and nobody has ever verified which VIP.

**Cause.** The VIP is the least-owned object in a clustered estate. It is a cluster resource, a DNS entry, a firewall rule, a load-balancer target and an application configuration item, and those five things are owned by four teams and documented in two places. Nothing in the cluster's own validation checks that the DNS record at the recovery site resolves to the address the cluster will actually bind.

**Guardrail.** Inventory every VIP as a first-class configuration item with all five of its representations, and make the DR exercise include a real client connection through the real name resolution path - not a connection to an IP address typed into a test script. A VIP that has never been resolved by a client at the recovery site has not been tested.

### 13.6 (f) A Cluster That Fails Over Perfectly Into an Application That Cannot Reconnect

**Symptom.** The cluster log is exemplary: fence fired, resources started in order, timings within expectation. The service is down for two hours because the application tier held dead connections, or retried in a tight loop that saturated the new node, or required a manual restart that nobody had authority to perform at 02:00.

**Cause.** The boundary. The cluster's job ends at the resource; the application's job begins there, and the two jobs are owned by different teams with different success criteria. The cluster team's dashboard is green; the service is red.

**Guardrail.** Make application reconnect an explicit, tested requirement with the same status as the fence: connection pools configured to detect and discard dead connections, retry with backoff rather than a storm, fast-failure notification consumed where the platform provides it (Oracle's FAN events through ONS being the canonical example), and an exercise that fails the node without touching the application tier. Section 11's fourth property is this guardrail stated as a test criterion.

---

## 14. The Worked Example: Cymbal Bank's Tier-0 Database Cluster

**Cymbal Bank is fictional and this example is illustrative.** It contains no data from any real institution, and no institution other than Cymbal Bank appears in it. Numbers are present only to make the reasoning concrete; they are not benchmarks, and none of them is a failover time.

### 14.1 The Estate

Cymbal Bank runs its core deposit ledger on a two-site, four-node Oracle RAC cluster: two nodes in the primary data centre, two in the secondary, one shared storage platform replicated between sites, and a Clusterware deployment whose voting files sit in a normal-redundancy ASM disk group. The application tier connects through a SCAN name; the batch tier connects directly to node VIPs for a small number of jobs that need instance affinity.

The design was built eight years ago and has been patched on a roughly annual cycle. It has failed over four times in production: twice for node hardware, once for a kernel panic, once for a storage-path failure that took one node's I/O away while its heartbeats remained healthy.

### 14.2 The Decision the Thesis Forced Into the Open

During a resilience review, an external reviewer asked a question nobody at Cymbal had asked in eight years: **what is the cluster configured to do if the inter-site link fails but both sites can still reach storage?**

The answer, discovered by reading the configuration rather than the design document, was that the cluster's quorum arithmetic made a site partition survivable by exactly one half - and that the surviving half was determined by which voting files each side could reach, not by which site held the more important workload. In the specific failure being discussed, the *secondary* site would have retained quorum and the primary would have been evicted, because two of the three voting-file copies were reachable from the secondary side of the storage replication.

That is a correct cluster decision. It is also a business decision that had never been made by the business: the bank had, in effect, decided that in a particular partition the disaster recovery site would win and the primary would be rebooted, and nobody had ever put that in front of anyone authorised to accept it.

### 14.3 What Cymbal Did About It

Three things, none of them a product change:

1. **Wrote the decision down.** The review produced a two-page decision record: the quorum arithmetic, the failure scenarios it covered, the scenarios where the outcome was counter-intuitive, and the rationale for keeping the configuration rather than changing it. The rationale was defensible - the alternative placements introduced a different, worse asymmetry - but the point was that the rationale now existed and was signed.
2. **Tested the fence rather than the failover.** The next exercise killed one primary node without warning, with the application tier live, and the acceptance criterion was not "the database came up elsewhere" but "a fence record exists for the killed node, and a synthetic transaction through the SCAN name succeeded afterwards, with no manual action". It took two attempts: the first exercise produced a clean resource start and no fence record, because a well-meaning operator had disabled fencing during the previous quarter's storage incident and the change had never been reversed. That finding - section 13(b) in its natural habitat - was worth more than the previous three years of successful switchover tests.
3. **Fixed the boundary.** The application tier's connection pool was configured to hold connections indefinitely on the assumption that the database would never move underneath it. It was changed to detect dead connections and retry with backoff, and the retry storm behaviour was tested by failing the node with the pool at full occupancy.

### 14.4 What It Cost

Being honest about cost is the point of a worked example:

- **Two change windows** for the pool configuration and its test, in a system where change windows are the scarcest resource in the institution.
- **One difficult conversation** about who owned the decision that the DR site would win a partition, which is a conversation about accountability rather than about technology.
- **A permanent obligation**: the decision record must be re-reviewed whenever the storage topology, the voting-file placement or the node count changes, because all three change the arithmetic.
- **And an uncomfortable finding**: the cluster had been, for an unknown period, in the "conservative fence" configuration described in section 3.5 - not by policy, but by an un-reversed workaround. The institution had accepted a corruption risk without knowing it.

### 14.5 What the Example Demonstrates

The cluster worked. It had worked for eight years. Nothing in the example is a product defect, a design error or an operational failure in the ordinary sense. What was missing was the thing the thesis names: **the decision existed, but it was not owned, not documented, and not evidenced** - and therefore it was not a control. It became one when three artefacts existed: a signed decision record, a fence record from an unplanned exercise, and an application-level measurement of the same exercise.

The last line of this guide is the sentence the example exists to illustrate.

---

## 15. The Claims Audit

**Legend.** Verified tick = read this pass at the vendor's own documentation (or, where noted, at the project's own documentation for open-source components), with the version gate and date recorded. Warning sign = partially verified, class-of-behaviour only, project-documentation sourced rather than vendor-documentation sourced, or version-gated and not re-confirmed at the newest release. Cross mark = rejected, with the reason. **No claim in this guide was rejected outright**; the ❌ entry in section 15.3 records a claim that was downgraded rather than asserted.

**Source-quality note.** `docs.oracle.com`, `learn.microsoft.com`, `clusterlabs.org` and `github.com/ClusterLabs/*` were read directly this pass. **`docs.redhat.com` (every URL attempted, including RHEL 8 and RHEL 9 variants), `www.ibm.com/docs` (including `/docs/en/powerha-aix/7.3.0`) and `docs.veritas.com` returned fetch errors or were blocked, and `sort.veritas.com` resolves to a document-search interface rather than to document text.** Web search returned zero results for the PowerHA query. Where a claim rests on one of those hosts it is marked warning and the guide states the class of behaviour rather than a specific. This is a tool limitation, not evidence that the material does not exist.

### 15.1 Verified at Vendor or Project Documentation

| # | Claim | Verdict | Version gate | Source | Date |
|---|---|---|---|---|---|
| **(a)** | **Oracle Clusterware definition:** "portable cluster software that provides comprehensive multi-tiered high availability and resource management for consolidated environments"; "supports clustering of independent servers so that they cooperate as a single system". | ✅ | Oracle Clusterware 19c | Oracle Clusterware Administration and Deployment Guide, 19c, ch. 1 (E95727-14) | Read Nov 2026 |
| **(a)** | **Origin:** "Oracle Clusterware was first released with Oracle Database 10g release 1 (10.1) as the required cluster technology for the Oracle multi-instance database, Oracle RAC." | ✅ | 19c documentation, historical statement | Same, ch. 1 | Read Nov 2026 |
| **(a)** | **Required for RAC:** "Oracle Clusterware is required for using Oracle RAC; it is the only clusterware that you need for platforms on which Oracle RAC operates." | ✅ | 19c | Same, ch. 1 | Read Nov 2026 |
| **(a)** | **Vendor clusterware deprecation:** "Starting with Oracle Database 19c, the integration of vendor clusterware with Oracle Clusterware is deprecated, and can be desupported in a future release. For this reason, Oracle recommends that you align your next software or hardware upgrade to transition off of vendor cluster solutions." | ✅ | From 19c | Same, ch. 1 | Read Nov 2026 |
| **(a)** | **Node limit:** "Oracle Clusterware supports up to 100 nodes in a cluster on configurations running Oracle Database 10g release 2 (10.2) and later releases" (chapter footnote). | ✅ (with the Flex Cluster caveat in (a) below) | 10.2 and later | Same, ch. 1, footnote 1 | Read Nov 2026 |
| **(a)** | **Flex Clusters:** "In Oracle Clusterware 12c release 2 (12.2), all clusters are configured as Oracle Flex Clusters"; Hub Node count "at least one and can be as many as 64"; pre-12.2 clusters "are converted in place as part of the upgrade process, including the activation of Oracle Flex ASM (which is a requirement for Oracle Flex Clusters)". | ✅ | From 12.2 | Same, ch. 1 | Read Nov 2026 |
| **(b)** | **Two stored components:** "Oracle Clusterware has two stored components, besides the binaries: The voting files, which record node membership information, and the Oracle Cluster Registry (OCR), which records cluster configuration information. Voting files and OCRs must reside on shared storage available to all cluster member nodes." | ✅ | 19c | Same, ch. 1 | Read Nov 2026 |
| **(b)** | **Voting files and fencing:** "Oracle Clusterware uses voting files to provide fencing and cluster node membership determination." | ✅ | 19c | Same, ch. 1.2.3 (Software Concepts) | Read Nov 2026 |
| **(b)** | **Voting file count:** minimum of three on "physically separate storage" recommended when not on ASM; single voting file "must use external mirroring"; "Oracle recommends that you do not use more than five voting files, even though Oracle supports a maximum number of 15 voting files". | ✅ | 19c | Same, ch. 1.2.3 | Read Nov 2026 |
| **(b)** | **Voting files in ASM are auto-created** according to disk-group redundancy; voting files can be added or replaced "without stopping the cluster"; Clusterware "automatically backs up voting file data in OCR as part of any configuration change and automatically restores the data to any voting file you add". | ✅ | 19c | Same, ch. 5 | Read Nov 2026 |
| **(b)** | **OCR contents and limits:** stores "information about the components that Oracle Clusterware controls, such as Oracle RAC databases, listeners, virtual IP addresses (VIPs), and services and any applications", in "a series of key-value pairs in a tree structure"; up to five locations; each on shared storage; a failed location replaceable online "if it is not the only OCR location"; updates only through supported utilities. | ✅ | 19c | Same, ch. 1.2.3, ch. 5 | Read Nov 2026 |
| **(b)** | **OLR:** per-node registry on local storage, default path `Grid_base/crsdata/host_name/olr/`, containing node-specific Clusterware manageability information used by Oracle High Availability Services. | ✅ | 19c | Same, ch. 5 | Read Nov 2026 |
| **(b)** | **OCR/voting placement history:** 12c does not support raw or block devices (migration to ASM or shared file system required before upgrade); 12.2 desupported direct placement on a shared file system; **19.3 rescinded that desupport for Standalone Clusters**, while Domain Services Clusters "must continue to place OCR and voting disk files in disk groups managed by Oracle ASM". | ✅ | 12c, 12.2, 19.3 | Same, ch. 5, "Notes" | Read Nov 2026 |
| **(b)** | **OCR on ASM dependency:** ASM compatibility must be `11.2.0.2` or later; "if an Oracle ASM instance fails on any node, then OCR becomes unavailable on that particular node"; if CRSD on that node is the OCR writer and most OCR locations are in ASM, "CRSD stops and becomes inoperable"; "Under no circumstances will the failure of one Oracle ASM instance on one node affect the whole cluster." | ✅ | 19c | Same, ch. 5.1 | Read Nov 2026 |
| **(c)** | **Two technology stacks:** upper "anchored by the Cluster Ready Services (CRS) daemon (CRSD)", lower "anchored by the Oracle High Availability Services daemon (OHASD)". | ✅ | 19c | Same, ch. 1.3 | Read Nov 2026 |
| **(c)** | **CRS role:** "The primary program for managing high availability operations in a cluster"; manages resources "based on the configuration information that is stored in OCR", including "start, stop, monitor, and failover operations"; generates events on status change. | ✅ | 19c | Same, ch. 1.3.1 | Read Nov 2026 |
| **(c)** | **CSS role:** "Manages the cluster configuration by controlling which nodes are members of the cluster and by notifying members when a node joins or leaves the cluster." | ✅ | 19c | Same, ch. 1.3.1 | Read Nov 2026 |
| **(c)** | **cssdagent / I/O fencing / eviction:** "The `cssdagent` process monitors the cluster and provides I/O fencing. This service formerly was provided by Oracle Process Monitor Daemon (`oprocd`), also known as `OraFenceService` on Windows. A `cssdagent` failure may result in Oracle Clusterware restarting the node." | ✅ | 19c | Same, ch. 1.3.1 | Read Nov 2026 |
| **(c)** | **EVM role:** "A background process that publishes events that Oracle Clusterware creates." Process names `evmd.bin`, `evmlogger.bin`. | ✅ | 19c | Same, ch. 1.3.1 | Read Nov 2026 |
| **(c)** | **Daemon name table:** `crsd.bin`, `ocssd.bin`, `cssdmonitor`, `cssdagent`, `octssd.bin`, `evmd.bin`, `gipcd.bin`, `gnsd`, `gpnpd.bin`, `ologgerd.bin`, `diskmon.bin`, `mdnsd.bin`, `oraagent.bin`, `ohasd.bin`, `ons`, `orarootagent`, `osysmond.bin`. | ✅ | 19c | Same, Table 1-1 | Read Nov 2026 |
| **(c)** | **CTSS deprecated in 19c**, with OS-level NTP/chrony directed instead and `cluvfy comp clocksync` for verification. | ✅ | 19c | Same, ch. 1 | Read Nov 2026 |
| **(d)** | **ASM as substrate:** "Oracle ASM: Provides disk management for Oracle Clusterware and Oracle Database"; "Oracle ASM is not only one process, but an instance"; under Flex ASM, ASM "does not necessarily run on every cluster node but only some of them". | ✅ | 19c | Same, ch. 1.3.1 and note | Read Nov 2026 |
| **(e)** | **Interconnect:** "A node that is part of a cluster requires a second network. This second network is referred to as the interconnect"; at least two NICs (public and private); private network "using a switch (or multiple switches) that only the nodes in the cluster can access"; for HA, "at least two network interfaces for the private interconnect network"; "Oracle does not support using crossover cables as Oracle Clusterware interconnects." | ✅ | 19c | Same, ch. 1.2.1 | Read Nov 2026 |
| **(e)** | **VIP and SCAN:** node VIPs optional by default included; SCAN is "a domain name registered to at least one and up to three IP addresses"; three SCAN addresses recommended for availability and scalability. | ✅ | 19c | Same, ch. 1.2.4 | Read Nov 2026 |
| **(f)** | **Cluster resource activity log:** located in the Grid Infrastructure Management Repository, gives "precise and specific information about a resource failure, separate from diagnostic logs"; writes "tagged with an activity ID and any related data gets the same parent activity ID", creating "a hierarchy of activity IDs". | ✅ | 19c | Oracle Clusterware Administration and Deployment Guide, 19c, App. J | Read Nov 2026 |
| **(g)** | **Pacemaker definition:** "a high-availability cluster resource manager - software that runs on a set of hosts (a cluster of nodes) in order to preserve integrity and minimize downtime of desired services (resources)"; maintained by the ClusterLabs community. | ✅ | Pacemaker 3.0 documentation | Pacemaker Explained, 3.0, section 1.2 | Read Nov 2026 |
| **(g)** | **STONITH definition:** "the ability to ensure that it is not possible for a node to be running a service ... accomplished via fence devices such as intelligent power switches that cut power to the target, or intelligent network switches that cut the target's access to the local network"; "A cluster cannot safely recover from certain failure conditions, such as an unresponsive node, without fencing." | ✅ | Pacemaker 3.0 | Same, section 1.2 (Note: Fencing) | Read Nov 2026 |
| **(g)** | **Stack components:** resources, resource agents, fence agents, cluster membership layer (Corosync, providing "reliable messaging, membership, and quorum information"), resource manager, tools. | ✅ | Pacemaker 3.0 | Same, section 1.2.1 | Read Nov 2026 |
| **(g)** | **Pacemaker daemons:** `pacemakerd`, `pacemaker-attrd`, `pacemaker-based`, `pacemaker-controld`, `pacemaker-execd`, `pacemaker-fenced`, `pacemaker-schedulerd`; Designated Controller election; CIB as an XML representation of configuration and state. | ✅ | Pacemaker 3.0 | Same, section 1.2.2 | Read Nov 2026 |
| **(g)** | **no-quorum-policy values and defaults:** default `stop`; allowed `ignore`, `freeze`, `stop`, `demote` (since 2.0.5), `fence` (since 2.1.9); `suicide` deprecated since 2.1.9. | ✅ | Pacemaker 3.0 | Same, section 3.4 | Read Nov 2026 |
| **(g)** | **fencing-enabled:** if false, "unresponsive nodes are immediately assumed to be running no resources, and resource recovery on online nodes starts without any further protection (which can mean data loss if the unresponsive node still accesses shared storage, for example)". | ✅ | Pacemaker 3.0 | Same, section 3.4 | Read Nov 2026 |
| **(g)** | **Fence operation parameters:** `fencing-timeout` default 60s; `fencing-action` `reboot` (default) or `off`; `fencing-max-attempts` default 10; `fencing-reaction` default `stop`, with `panic` available (since 2.0.3). These are configuration defaults for the fence operation, **not failover times**. | ✅ | Pacemaker 3.0 | Same, section 3.4 | Read Nov 2026 |
| **(g)** | **Two-node split-brain policy:** `priority-fencing-delay` (since 2.0.4) applies a delay to fencing of lost nodes with the highest total resource priority "so that the more significant nodes potentially win any fencing match (especially meaningful in a split-brain of a 2-node cluster)". | ✅ | Pacemaker 2.0.4+ | Same, section 3.4 | Read Nov 2026 |
| **(g)** | **Other documented timing defaults:** `dc-deadtime` 20s; `startup-fencing` default true (fencing unseen nodes at start-up is the safe setting); `node-pending-timeout` (since 2.1.7). | ✅ | Pacemaker 3.0 | Same, section 3.4 | Read Nov 2026 |
| **(h)** | **SBD:** "A highly reliable fencing or Shoot-the-other-node-in-the-head (STONITH) mechanism that works by utilizing shared storage"; works with Pacemaker clusters, "currently known to compile and function on Pacemaker 1.1.7+ or 2.0.0+ and corosync 1.4.x, 2.3.x or 3.0.x". | ✅ (project documentation) | SBD, current README | github.com/ClusterLabs/sbd README | Read Nov 2026 |
| **(h)** | **SBD integration in Pacemaker:** `have-watchdog` "set automatically by the cluster according to whether SBD is detected to be in use"; `fencing-watchdog-timeout` governs watchdog-based self-fencing via SBD; the timeout "must be larger than `SBD_WATCHDOG_TIMEOUT`", and Pacemaker "will refuse to start" where that is untrue; misconfiguration "could" cause "data corruption or loss". | ✅ | Pacemaker 3.0 | Pacemaker Explained, 3.0, section 3.4 | Read Nov 2026 |
| **(i)** | **WSFC definition and scope:** "a configuration of independent computers, known as nodes, which work together to enhance the availability and scalability of applications and services, now referred to as clustered roles"; documented as applying to Windows Server 2016, 2019, 2022, 2025 and Azure Local. | ✅ | Windows Server 2016-2025 | Microsoft Learn, *Failover Clustering* overview | Read Nov 2026 (page updated 25 Jun 2025) |
| **(i)** | **WSFC quorum and split-brain:** "Cluster nodes collaborate to maintain what is known as a quorum ... the minimum number of votes from cluster members required for the cluster to function correctly. This mechanism prevents split-brain scenarios, where split portions of a cluster might try to operate independently, potentially causing inconsistencies." | ✅ | Windows Server 2016-2025 | Same | Read Nov 2026 |
| **(i)** | **Quorum models:** Node Majority, Node and Disk Majority, Node and File Share Majority, No Majority (Disk Only). | ✅ | Windows Server 2016-2025 | Same | Read Nov 2026 |
| **(i)** | **Witness types and cloud witness requirements:** disk witness, file share witness, cloud witness; cloud witness requires an Azure Standard general purpose v2 storage account, creates the `msft-cloud-witness` container, and requires outbound HTTPS on port 443; "We recommend you always configure a witness if you're using Windows Server 2012 R2 and later. Clusters in later versions of Windows Server automatically manage the witness vote and their nodes vote with Dynamic Quorum." | ✅ | Windows Server 2016-2025 | Microsoft Learn, *Deploy a quorum witness* | Read Nov 2026 (updated 16 Jun 2025) |
| **(i)** | **CSV:** "enables multiple nodes in a Windows Server failover cluster ... to simultaneously have read-write access to the same LUN (disk) that is provisioned as an NTFS volume"; ReFS supported but the CSV is placed in "redirected mode", "meaning write access is sent to the coordinator node"; roles "can fail over quickly from one node to another node without requiring a change in drive ownership, or dismounting and remounting a volume". | ✅ | Windows Server 2016-2025 | Microsoft Learn, *Cluster Shared Volumes overview* | Read Nov 2026 (updated 9 Jul 2025) |
| **(i)** | **Networks and heartbeat:** clusters "often employ dedicated private networks for internal functions such as heartbeat signals and cluster management, while separate public networks handle client access and application data"; health "continuously monitored through heartbeat signals". | ✅ | Windows Server 2016-2025 | Microsoft Learn, *Failover Clustering* overview | Read Nov 2026 |

### 15.2 Class-of-Behaviour Only, or Not Verified This Pass

| # | Claim | Verdict | Version gate | Source attempted | Date |
|---|---|---|---|---|---|
| **(j)** | **PowerHA SystemMirror: repository disk, resource groups with startup/fallback/failure policies, RSCT group services, CAA cluster communications.** Described in the guide at class level only. | ⚠ | PowerHA SystemMirror 7.3 (AIX) intended | `www.ibm.com/docs/en/powerha-aix/7.3.0` and topic URLs - fetch error on every attempt; `ibm.com` docs host unreachable this pass | Attempted Nov 2026 |
| **(j)** | **PowerHA's lineage from HACMP** (the rename, and that documentation and practitioners use both names). | ⚠ | n/a | Not verified at a vendor source this pass | Nov 2026 |
| **(k)** | **InfoScale: service groups, GAB as membership/messaging transport, LLT as the low-level transport, I/O fencing with coordinator disks.** Described in the guide at class level only. | ⚠ | InfoScale Availability 7.4.x intended | `sort.veritas.com/documents` (search UI, no document text); `sort.veritas.com/public/documents/...` returned 404; `docs.veritas.com` blocked by the fetch layer | Attempted Nov 2026 |
| **(k)** | **InfoScale's former name, Veritas Cluster Server.** | ⚠ | n/a | Not verified at a vendor source this pass | Nov 2026 |
| **(l)** | **RHEL High Availability Add-On packaging: which Pacemaker and Corosync major versions ship with which RHEL release.** The guide states no mapping. | ⚠ | RHEL 8 / RHEL 9 intended | `docs.redhat.com` RHEL 9 and RHEL 8 chapter and single-page URLs - "All scraping engines failed"; `access.redhat.com` not reached | Attempted Nov 2026 |
| **(l)** | **`corosync-qdevice` / `corosync-qnetd` quorum device behaviour** (that the device answers the arbitration question for one partition only). The guide describes the design intent at class level. | ⚠ | Corosync 3.x / qdevice | `github.com/ClusterLabs/corosync-qdevice` returned 404 (repository path not found); no vendor page reached | Attempted Nov 2026 |
| **(m)** | **WSFC internal architecture** - the Cluster Service process and the **RHS (Resource Hosting Subsystem)** that loads resource DLLs - plus **current default heartbeat/threshold values**. Non-product-specific background only; no process name or timing default is asserted anywhere in this guide. | ⚠ | Windows Server 2016-2025 | Not located in the pages read; deliberately omitted rather than asserted | Nov 2026 |
| **(m)** | **Oracle: the exact release in which OCR and voting files became storable in ASM.** The guide states the 12c raw-device desupport, the 12.2 shared-file-system desupport, the 19.3 rescission for Standalone Clusters, and the 19c ASM-compatibility requirement - all verified - but does **not** assert a single "first release" for ASM placement. | ⚠ | 12c / 12.2 / 19.3 | Oracle 19c guide (verified statements only) | Read Nov 2026 |
| **(m)** | **Oracle Flex Cluster maximum node counts beyond the Hub Node range** (documented as 1-64 Hub Nodes, with "many more" supported nodes). The guide does not state a total. | ⚠ | 12.2+ | Oracle 19c guide ch. 1 | Read Nov 2026 |

### 15.3 Downgraded, and Why

- **"Oracle Clusterware supports 100 nodes."** ❌ as a flat current statement, ⚠ as the documented footnote. The 19c guide's footnote says support for "up to 100 nodes in a cluster on configurations running Oracle Database 10g release 2 (10.2) and later releases", while the same chapter describes Flex Clusters with up to 64 Hub Nodes and "many more" supported nodes. The guide therefore reports the footnote as a historical statement and does **not** present 100 as a current architectural limit. Confirm against the release's own limits before designing to a number.
- **"Quorum prevents split-brain."** ✅ as a statement about the *mechanism's purpose* (this is Microsoft's own wording), ⚠ as a statement about outcomes. Quorum prevents two partitions from both being *entitled* to act; whether split-brain actually occurs depends on whether the entitlement is *enforced*, which is the fence. The guide keeps the two separate throughout section 3 for this reason.
- **"A cluster cannot safely recover from an unresponsive node without fencing."** ✅ as a Pacemaker documentation quotation, and scoped as such. It is not presented as a claim about every product's implementation, since implementations differ in whether the fence is external, storage-level or self-fencing.

### 15.4 What Could Not Be Verified

Listed explicitly, because an unlisted gap becomes an assumed fact.

1. **The IBM PowerHA SystemMirror documentation could not be reached at all.** `www.ibm.com/docs`, including the PowerHA 7.3.0 product path, returned errors on every attempt, and a web search for the PowerHA CAA/RSCT/repository-disk material returned zero results. **The PowerHA entries in section 6.2 are the class of behaviour, not verified quotations.** Do not take component names, policy option names, repository-disk sizing or command syntax from this guide. This mirrors the same host limitation recorded in [ibm_mq_disaster_recovery_guide.md](ibm_mq_disaster_recovery_guide.md), where the IBM documentation host was read through a public mirror.
2. **The Veritas InfoScale documentation could not be reached.** `sort.veritas.com` presents a document-search interface with no document text, the direct document path attempted returned a 404, and `docs.veritas.com` was blocked by the fetch layer as a private/internal address. **The GAB, LLT and coordinator-disk description in section 6.3 is the class of behaviour.** No InfoScale command, option name or sizing rule in this guide is vendor-verified.
3. **The Red Hat Enterprise Linux High Availability Add-On documentation could not be reached.** Every `docs.redhat.com` URL attempted (RHEL 9 chapter and single-page, RHEL 8 chapter) failed at the scraping layer. **The guide therefore states no mapping between RHEL releases and Pacemaker/Corosync majors**, which is a fact a reader will need and must obtain from Red Hat directly.
4. **The Corosync quorum device** (`corosync-qdevice` / `corosync-qnetd`) is described at design-intent level only. The ClusterLabs repository path attempted returned 404 and no vendor page was reached, so the specific behaviour - that the device can grant quorum to only one partition - is stated as intent, not as verified product behaviour.
5. **Windows Server internals below the documented surface.** The resource-hosting process architecture and the current default heartbeat and threshold values were not located in the pages read, and are deliberately absent from the guide rather than approximated.
6. **Oracle's node-eviction decision logic in detail.** The guide quotes the documented consequence of a `cssdagent` failure ("may result in Oracle Clusterware restarting the node") and the documented role of voting files in fencing, but does **not** describe the eviction decision sequence, the reboot-time parameter, or the disk-monitor interaction, none of which was read this pass. **Do not take an eviction-troubleshooting procedure from this guide.**
7. **The exact Oracle release that first allowed OCR and voting files in ASM.** The verified statements are the 12c raw-device desupport, the 12.2 shared-file-system desupport, the 19.3 rescission for Standalone Clusters, and the 19c ASM-compatibility requirement. No single "introduced in" release is asserted.
8. **Per-release confirmation that the described behaviours are unchanged at Oracle 23ai, at Pacemaker 3.x point releases beyond the 3.0 documentation read, or at Windows Server 2025.** Most claims were read at one documentation level. **Version-gated behaviour must be re-checked at the release you deploy.**
9. **Any failover time, RTO, throughput or benchmark figure.** None appears in this guide by design, and none was sought. Where a vendor's configurable default is quoted (a fence timeout, a deadtime, a heartbeat parameter) it is labelled as a configuration default for a phase, never as a failover time.
10. **Any ranking of the five product families.** Deliberately absent. Section 4 describes design centres; it does not evaluate them against each other, because that judgement depends on the estate, the storage, the skills and the change regime, and cannot be made in the abstract.
---

## 16. Glossary and Closing Summary

### 16.1 Glossary

- **ASM (Oracle Automatic Storage Management)** - Oracle's disk-management layer, documented as providing "disk management for Oracle Clusterware and Oracle Database". An ASM *instance*, not merely a process; under Flex ASM it need not run on every cluster node. Where Oracle Clusterware's own files live in ASM, an ASM instance failure makes OCR unavailable on that node but does not affect the whole cluster. Section 5.5.
- **CAA (Cluster Aware AIX)** - the AIX-level clustering service that IBM PowerHA builds on, providing cluster communications and the shared repository. Class-of-behaviour only in this guide; not vendor-verified this pass. Section 6.2.
- **CIB (Cluster Information Base)** - Pacemaker's XML representation of the cluster's configuration and of the state of all nodes and resources, kept synchronised across the cluster by `pacemaker-based`. Section 6.1.
- **Cluster** - the set of nodes governed by one membership view and one configuration. Defined by what it agrees on, not by what it contains. Section 1.2.
- **Clusterware** - the software layer that makes independent servers behave as one system for availability: membership, quorum, fencing, and ordered resource management. Sits below the application and beside the operating system. Section 1.2, section 2.
- **Corosync** - the membership, messaging and quorum layer that Pacemaker consumes; Pacemaker does not implement membership itself. Section 6.1.
- **CSV (Cluster Shared Volume)** - the Windows Server mechanism giving multiple cluster nodes simultaneous read-write access to the same NTFS LUN, with ReFS supported in redirected mode; roles can fail over without a change of drive ownership or a remount. Section 6.4.
- **Designated Controller (DC)** - the Pacemaker controller instance elected to centralise cluster decision-making; if it fails, a new one is established. Section 6.1.
- **Dynamic quorum** - the Windows Server behaviour in which the cluster manages the witness vote and node votes automatically as nodes leave and rejoin. Section 6.4.
- **EVM (Event Management)** - the Oracle Clusterware background process that publishes events created by Clusterware. Section 5.6.
- **Failover** - transfer of a resource group to another node as a consequence of a failure, as distinct from a switchover, which is the same transfer performed deliberately. The distinction is the difference between evidence and rehearsal. Section 1.2, section 9.4.
- **Fencing** - the removal of a node's ability to do harm, usually by rebooting or powering it off, or by revoking its storage access. It removes the loser's *ability to write*, not its belief that it won. Section 2.3, section 3.4.
- **Hub Node** - in an Oracle Flex Cluster, a node with direct access to shared storage, documented as numbering between 1 and 64; other nodes access data through the Hub Nodes. Section 5.9.
- **Interconnect** - the private second network a cluster node requires, on a switch or switches only cluster nodes can access; crossover cables are not supported as Oracle Clusterware interconnects. Section 5.7.
- **Membership** - the maintained, agreed answer to "who is in?", held identically by every member. Not a configuration list. Section 2.1.
- **no-quorum-policy** - the Pacemaker cluster option governing behaviour without quorum: `ignore`, `freeze`, `stop` (default), `demote` (since 2.0.5), `fence` (since 2.1.9). Each is a policy decision with a data-loss or availability consequence. Section 3.6.
- **OCR (Oracle Cluster Registry)** - the Clusterware component storing cluster configuration - resources, listeners, VIPs, services, applications - as key-value pairs in a tree structure, replicated across up to five shared-storage locations and updatable only through supported utilities. Section 5.2, section 5.4.
- **OHASD (Oracle High Availability Services daemon)** - the anchor of Clusterware's lower technology stack, the daemon that brings up the rest. Section 5.6.
- **OLR (Oracle Local Registry)** - the per-node registry on local storage holding node-specific Clusterware configuration, readable before shared storage is available. Section 5.2.
- **ONS (Oracle Notification Service)** - the publish/subscribe service carrying Fast Application Notification events, the path by which applications learn quickly that a resource moved. Section 5.6, section 8.1.
- **Quorum** - the rule determining whether a surviving subset of nodes is entitled to act as the cluster, normally a majority of votes, sometimes including a witness device. A property of the group, never of a node; answers "may we act", never "should we act". Section 2.2, section 3.3.
- **Quorum device** - an external arbiter that can grant quorum to only one partition: a quorum disk, a file-share witness, a cloud witness, or a quorum device service. Its placement determines whether one storage failure can remove both the data and the arbitration. Section 3.3, section 13.1.
- **Resource agent** - the script or operating-system component that starts, stops and monitors a resource, providing a uniform interface between the cluster manager and the managed service. Section 6.1.
- **Resource group** - a named set of resources managed together, started and stopped in a defined order. Veritas calls them service groups; PowerHA calls them resource groups; Pacemaker expresses the same idea through ordering and colocation constraints. Section 1.2, section 2.4.
- **RSCT (Reliable Scalable Cluster Technology)** - the IBM group-services infrastructure underlying PowerHA. Class-of-behaviour only in this guide; not vendor-verified this pass. Section 6.2.
- **SBD (Storage-Based Death)** - a fencing mechanism using shared storage, described by its project as "a highly reliable fencing or Shoot-the-other-node-in-the-head (STONITH) mechanism that works by utilizing shared storage"; integrates with Pacemaker, including watchdog-based self-fencing. Section 6.1.
- **SCAN (Single Client Access Name)** - an Oracle cluster name resolving to between one and three IP addresses, giving clients a stable connection target across instance changes. Section 5.7.
- **Split-brain** - the state in which two subsets of a cluster each believe they are the cluster and both proceed, typically both writing to the same data. Caused by a partition, not a crash; silent when it happens; not recoverable by a restart. Section 3.
- **STONITH** - "Shoot The Other Node In The Head": Pacemaker's term for fencing, defined as "the ability to ensure that it is not possible for a node to be running a service". Section 2.3.
- **Switchover** - the deliberate transfer of a resource group, as distinct from a failover. A switchover test proves the runbook; it does not prove the fence. Section 9.4.
- **VIP (virtual IP address)** - an address moved between nodes as a resource so clients can reconnect to a stable target. Its propagation after a failover is invisible in cluster logs and is a common cause of "the cluster failed over but nobody could connect". Section 1.2, section 8.1, section 13.5.
- **Voting file (voting disk)** - the Oracle Clusterware artefact recording node membership, used "to provide fencing and cluster node membership determination"; three on physically separate storage recommended when not in ASM, five recommended as a maximum even though 15 are supported. Section 5.3.

### 16.2 The Closing Summary

Six propositions, in the order the guide argued them:

1. **Clusterware performs four jobs - membership, quorum, fencing, resource management - and each is unsafe without the one before it.** A resource manager without a fence is an automated way to cause corruption; a fence without quorum is an automated way to cause an outage. Section 2.
2. **A heartbeat is a hint, not a proof.** "I cannot see my peers" is consistent with a dead peer, a broken path, a broken local stack, or a peer that can see the cluster but not the storage. Quorum is how the hint is corroborated; fencing is how the corroborated decision is enforced. Section 3.2.
3. **The tie-break is a policy decision with a data-loss consequence either way.** Aggressive fencing converts transient network events into outages; conservative fencing converts partitions into corruption, and corruption is not recoverable by a restart. Integrity dominates availability, and the decision belongs on the record. Section 3.5.
4. **Failover time is a property of the configuration, not of the product.** It is eight linked phases from detection to application reconnect, six of which the institution controls directly, and the two most commonly measured are rarely the ones that dominate. Section 8.
5. **The scheduler replaced the deployment model, not the data-safety model.** Per-pod placement and declarative state are genuine advances; neither fences a node out of a LUN nor arbitrates a single-writer database cluster. Section 10.
6. **The audit question is not "is it clustered" but "can you show the decision was made correctly".** That requires a signed decision record, dated configuration, fence and eviction history, an unplanned exercise that fired the fence, and an application that reconnected without help - plus a named person who can still read the log. Section 11, section 12.

The evidence from the estates is uncomfortable but consistent. Most clusters work. Most clusters have never been tested against the failure they were built for. The fence is the component most often disabled and least often verified, and the configuration that decides who survives under partition is frequently a default, a workaround, or a legacy choice that no named person has ever approved. None of that is a product failure. It is the ordinary consequence of a control that nobody wrote down.

Which returns the argument to where it began. A cluster is not valuable because it restarts a service. It is valuable because it makes a decision under conditions where no human could make it in time, and because the institution can show, afterwards, that the decision was the one it chose. That is what distinguishes a control from a feature, and it is why the sentence below is the thesis rather than a summary.

Clusterware does not prevent failure; it converts a node failure into a decision about who survives - and in a bank that decision is a control, not a feature.
