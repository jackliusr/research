# Static Stability - Configured Not to Care

**Author:** Jack Liu Shurui, Solution Architect
**Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research) - **Series:** Technology Research Guides - **Version:** 1.0 - **Date:** October 2026
**Context:** A first-principles examination of AWS static stability - what AWS actually documents, what the term means precisely, why a recovery path that runs through an impaired control plane is worse than no recovery path at all, what the patterns cost in idle capacity and complexity, how to test the property rather than assert it, and where the whole idea is the wrong choice.

**Companion guides (cross-reference, do not re-derive):** [resilience_engineering_guide.md](../management/resilience_engineering_guide.md) for Hollnagel's safety science, the four abilities and Safety-II; [operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) for MAS, PRA/FCA, DORA, impact tolerances and the BCP lifecycle; [ibm_mq_disaster_recovery_guide.md](ibm_mq_disaster_recovery_guide.md) for messaging-tier recovery mechanics; [cloud_providers_guide.md](cloud_providers_guide.md) for the provider comparison this guide deliberately avoids; [chaos_engineering_guide.md](chaos_engineering_guide.md) for failure injection as a discipline.

**Table of Contents**

1. [The Overview, the Decoder and the Provenance of the Term](#1-the-overview-the-decoder-and-the-provenance-of-the-term)
2. [What Static Means, Precisely](#2-what-static-means-precisely)
3. [The Control Plane and the Data Plane, Concretely](#3-the-control-plane-and-the-data-plane-concretely)
4. [The Canonical Contrast: Two Designs, One Zone Failure](#4-the-canonical-contrast-two-designs-one-zone-failure)
5. [Why Reaction Through an Impaired Control Plane Is Worse Than No Reaction](#5-why-reaction-through-an-impaired-control-plane-is-worse-than-no-reaction)
6. [The Patterns That Achieve Static Stability, and What Each Costs](#6-the-patterns-that-achieve-static-stability-and-what-each-costs)
7. [The Cost of Static Stability, Stated Plainly](#7-the-cost-of-static-stability-stated-plainly)
8. [Testing It Rather Than Asserting It](#8-testing-it-rather-than-asserting-it)
9. [The Banking and Regulated Angle](#9-the-banking-and-regulated-angle)
10. [The Failure Modes](#10-the-failure-modes)
11. [The Cymbal Bank Worked Example](#11-the-cymbal-bank-worked-example)
12. [The Anti-Patterns of Claiming Static Stability](#12-the-anti-patterns-of-claiming-static-stability)
13. [The State of Practice: Documented, Conventional, Undocumented](#13-the-state-of-practice-documented-conventional-undocumented)
14. [The Claims Audit](#14-the-claims-audit)
15. [What Could Not Be Verified, the Glossary and the Cross-References](#15-what-could-not-be-verified-the-glossary-and-the-cross-references)
16. [The Closing Summary](#16-the-closing-summary)

---

## 1. The Overview, the Decoder and the Provenance of the Term

The thesis of this guide is one sentence long: **a statically stable system does not fight the failure; it was already configured not to care.** Everything that follows is an attempt to make that sentence survive contact with an actual estimator, an actual incident, and an actual examiner. The sentence is easy to say and unusually hard to build, because the thing it asks for is not a capability but an absence - the absence of a reaction.

Most resilience writing describes recovery as an action. Something breaks, something detects it, something decides, something moves. Static stability describes recovery as a configuration that was already true before anything broke. The distinction sounds like semantics until you notice that the action-based version has a dependency the configuration-based version does not: the actor. And in AWS, the actor is very often the part that is already having a bad day.

### The decoder

Before the argument, the vocabulary. Six terms carry the guide, and several of them are used loosely in the industry, so they are pinned here.

| Term | Working meaning in this guide |
|---|---|
| **Static stability** | A property of a system whose behaviour under a named failure is identical to its behaviour without it, because the capacity and routing required were already in place, so the failure requires no decision and no control-plane call. |
| **Control plane** | The administrative machinery that creates, modifies, deletes and describes resources, and propagates those changes to where they take effect. |
| **Data plane** | The daily business of provisioned resources - the hot path that actually serves traffic or moves bytes. |
| **Availability Zone** | A logically isolated, physically separated section of a Region, with independent power and infrastructure, connected to its siblings by fast private fibre. |
| **Region** | A geographic grouping of Availability Zones under one administrative and API boundary. |
| **Cell** | An independently failing partition of a workload, sized so that its failure consumes a bounded share of total capacity. Not an AWS-defined term; see Section 6. |
| **Pre-arranged state** | The capacity, routing, credentials and configuration that were provisioned before the failure and therefore require no call during it. |

Two of these definitions are the guide's own construction and two are paraphrases of AWS wording; the distinction matters and Section 14 keeps the accounts. The decoder is not neutral - the phrase "no decision and no control-plane call" is doing work that AWS's own definition does not do, and Section 14 is where that divergence is confessed in full.

### The provenance of the term

The term has a documented lineage, and it is shorter than most people assume. It is not an old idea with a new name; it is a named AWS position that appears in public material from late 2019 and gets progressively more formal.

**Source A - AWS Fault Isolation Boundaries whitepaper, page "Static stability", publication date November 16, 2022** (retrieved 2026-10-24). This is the canonical definition, and it is printed here rather than paraphrased because the paraphrase is where most of the confusion enters:

> "One of the most important resilience characteristics of AWS services is what AWS calls static stability. What this term means is that systems operate in a static state and continue to operate as normal without the need to make changes during the failure or unavailability of dependencies."

Note what that definition is actually about. It is about *changes*, and it is about *dependencies* in the plural and generally. It does not say "capacity", it does not say "Availability Zone", and it does not say "no human decision". Most practitioners read the term as "we have spare capacity in the other zones". That reading is a special case of the definition, not the definition.

Two further sentences from the same page carry most of the operational weight. The first is the design principle behind the whole idea:

> "The most reliable recovery and mitigation mechanisms are the ones that require the fewest changes to achieve recovery."

**Fewest changes**, not fastest. That is a claim about failure surfaces rather than about speed, and it is the reason Section 2 argues that static stability deletes steps rather than accelerating them.

The second sentence establishes that this is not an internal AWS practice being described from the outside:

> "it is also a pattern that can be used by customers. In fact, a majority of the best practice guidance for using the different types of AWS services in a resilient way is to implement static stability for production environments."

That is a strong claim - *a majority* of the best-practice guidance - and it is the reason the term deserves to be treated as a design constraint rather than an interesting property of AWS's own infrastructure.

**Source B - Amazon Builders' Library, "Static stability using Availability Zones", Mike Furr, published December 5, 2019, last modified December 5, 2019** (retrieved 2026-10-24). This is the earliest dated source retrieved, the most detailed, and the one that supplies the mechanics:

> "In a statically stable design, the overall system keeps working even when a dependency becomes impaired. Perhaps the system doesn't see any updated information (such as new things, deleted things, or modified things) that its dependency was supposed to have delivered. However, everything it was doing before the dependency became impaired continues to work despite the impaired dependency."

That second sentence is the most under-read sentence in the entire literature. It says that a statically stable system is allowed to be *stale*. It is allowed to not know about the new thing. The requirement is only that it keeps doing what it was already doing. Static stability, properly understood, is a claim about tolerating missing updates, and capacity is one instance of that.

**Source C - AWS Well-Architected Reliability Pillar, "REL11-BP05 Use static stability to prevent bimodal behavior", pillar publication date November 6, 2024** (retrieved 2026-10-24). This is where the concept becomes a normative best practice with a risk rating:

> "Workloads should be statically stable and only operate in a single normal mode. Bimodal behavior is when your workload exhibits different behavior under normal and failure modes."

It also states the risk explicitly, and the number is worth noting because it is *not* the top of the scale: "Level of risk exposed if this best practice is not established: Medium." The framework does not treat this as an emergency. It treats it as a trade that most workloads should make and some should not.

**Source D - AWS Well-Architected Reliability Pillar, "REL11-BP04 Rely on the data plane and not the control plane during recovery", same pillar, November 6, 2024** (retrieved 2026-10-24). This is the strongest single piece of evidence in the guide, and Section 5 is built on it.

**Source E - AWS Fault Isolation Boundaries whitepaper, page "Control planes and data planes", November 16, 2022** (retrieved 2026-10-24). The taxonomy source.

**Source F - AWS Architecture Blog, "Minimizing Dependencies in a Disaster Recovery Plan"** (retrieved 2026-10-24). No publication date is printed on the page as retrieved; its image asset paths are dated 2022-01-25. It is cited here for its testing method and its Route 53 ARC description, not for dating the concept.

So the record is: a 2019 article, a 2022 whitepaper, and a 2024 best-practice pair. The Builders' Library article references a re:Invent 2019 session (DOP328, "Introducing the Amazon Builders' Library"), which makes 2019 a firm floor for the public term. The concept is older than the term in engineering practice - N+1 provisioning is not new - but the vocabulary is recent, and that matters: a term this young is still being defined, and the definitions are still diverging.

### The boundary, declared by name

Two companion guides live close enough to this one to be confused with it, and neither owns this territory.

[resilience_engineering_guide.md](../management/resilience_engineering_guide.md) is 1071 lines of Hollnagel's safety science - the four abilities (respond, monitor, anticipate, learn), Safety-I versus Safety-II, work-as-imagined versus work-as-done, the ETTO principle, FRAM, and the New View of human error. It is about how organisations understand and adapt to risk. It contains **zero** mentions of control plane, static stability, zone failure or cell-based design. It is not a cloud guide and does not pretend to be.

[operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) is 880 lines of regulatory framework - MAS, PRA/FCA, DORA, BCBS, business impact analysis, important business services, impact tolerances, RTO/RPO tiers, the BCP lifecycle, third-party resilience. It is about what a regulator requires you to be able to demonstrate. It does not own the engineering mechanism either.

One is a safety science. The other is a regulatory regime. **This guide is the AWS Reliability Pillar engineering concept, and the thing it owns is the mechanism**: which plane is called, whether a call happens at all, what the idle capacity costs, and how you prove the property instead of asserting it. Section 9 is the only place the regulatory frame appears, and there it appears as a consumer of this pattern, not as a restatement of it.

### Why the term is not already covered

The repository is not short of resilience material, which makes the gap worth measuring rather than asserting. Counts below were taken in this pass on 2026-10-24, by case-insensitive grep across all 676 markdown files in the repository, counting *files containing at least one match* rather than match occurrences:

| Pattern | Files matched |
|---|---|
| `static stability` | 0 |
| `disaster recovery` | 49 |
| `failure` \| `runbook` \| `playbook` | 527 |
| `failover` | 103 |
| `control plane` \| `data plane` | 90 |
| `cell-based` \| `bulkhead` \| `blast radius` | 96 |
| `availability zone` \| `multi-AZ` | 38 |
| `chaos engineering` | 18 |

Two of these numbers are worth dwelling on, because they are the argument. **`static stability` matches zero of 676 files.** The exact term this guide is about appears nowhere in a repository that discusses failover in 103 files and control/data planes in 90. The vocabulary is genuinely absent while the adjacent vocabulary is abundant - which is the profile of a concept that practitioners approximate without naming, and therefore without checking.

The `availability zone | multi-AZ` count of 38 is the second number worth dwelling on. It is *lower* than the failover count by a wide margin, which means most of this repository's failover discussion is not zonal failover at all - it is tier failover, instance failover, or region failover. Zonal design, the substrate of the entire static stability argument, is a minority topic here.

An honesty note on method: an earlier pass over this repository reported `disaster recovery` at 187 files and `failure|runbook|playbook` at 254. Neither figure reproduced under the file-union method used above, which returned 49 and 527 respectively. The `failover` figure came out at 103 on re-measurement and is stable across case-sensitive and case-insensitive matching. The numbers printed in the table are the ones actually measured in this pass; where an earlier figure disagrees, the earlier figure is not used. This is a small thing, but a guide arguing for evidence-based claims does not get to smuggle in an unreproduced number.

---

## 2. What Static Means, Precisely

"Static" is a misleading word. Nothing about a statically stable system is motionless: it serves traffic, it autoscales within its provisioned envelope, it retries, it sheds load. The word refers to one narrow thing - the *configuration* does not change in response to the failure. The system's shape is fixed at the moment of failure, and the failure is absorbed by that shape.

So the precise claim has three parts, and a system that satisfies two of them is not statically stable.

### No action

Nothing is created, launched, promoted, failed over, re-pointed or scaled *because* the failure happened. This is the part everyone remembers.

### No decision

There is no branch point that a human or a system must take during the event. This is the part everyone forgets, and it is the load-bearing one. A runbook step that says "if the zone is impaired, promote the replica in zone B" is not static stability even if the promotion is a single click, because it contains a conditional. The conditional will be evaluated with bad information - a partial signal, a stale dashboard, an alarm that fired for the wrong reason - and it can be evaluated wrongly. The decision is the fragile component, not the click.

### No control-plane call

The mechanism of recovery, whatever it is, runs on the data plane. Traffic shifts because a load balancer stopped health-checking a zone, not because a record changed. Capacity is present because it was provisioned, not because a scaling group was asked for more. This is the part that Section 5 argues is not merely convenient but necessary.

A system with action-but-no-decision (fully automated failover) is better than a system with a decision, and worse than a system with neither. A system with a decision but no control-plane call (a human flips a data-plane routing control that was pre-provisioned) is a defensible compromise and appears in Section 6 as such. The full property requires all three.

### The dynamic pattern, as a sequence

The contrast is cleanest as a sequence. The conventional dynamic recovery path has four steps:

1. **Detect** - an alarm, a health check, a synthetic probe, a human noticing.
2. **Decide** - classify the failure, choose a response, select a target.
3. **React** - call an API to create, promote, re-point or scale something.
4. **Serve** - the new resource warms, registers, passes health checks, and takes traffic.

Static stability does not make steps 1 through 4 faster. It **deletes steps 2 and 3**. There is nothing to decide because the response is unconditional, and there is nothing to call because the response was pre-arranged. What remains is step 1 as telemetry (useful for humans, not on the critical path) and step 4 reduced to its trivial case: the surviving capacity was already warm and already registered, so it absorbs load the instant the health check withdraws.

This is why "we have automated failover, so we are basically static" is wrong. Automation compresses steps 2 and 3 to milliseconds. Static stability removes them. The distinction is not latency, it is the existence of a failure surface - an automated decision can be automated and wrong, and an automated API call can be automated and throttled.

### Constant work

Source C gives the concept its name, in a passage about a failure mode that is not about capacity at all:

> "An example of bimodal behavior would be a network timeout that could cause a system to attempt to refresh the configuration state of the entire system. This would add an unexpected load to another component and might cause it to fail... Instead, you can build systems that are statically stable and operate in only one mode. A statically stable design would do constant work and always refresh the configuration state on a fixed cadence. When a call fails, the workload would use the previously cached value and initiate an alarm."

**Constant work** is the generalisation of static stability beyond capacity. The system does the same amount of work per unit time regardless of whether its dependencies are healthy. It refreshes on a fixed cadence - not on failure, not on demand spikes, not on cache misses. When the refresh fails, it keeps the old value and raises an alarm. The alarm is the only thing that changes behaviour, and the alarm does not serve traffic.

This is a much stronger and more useful idea than "over-provision across zones", and it is the one that applies to systems with no compute capacity to over-provision at all. A configuration cache that stampedes on expiry, a service-discovery client that hammers the registry when the registry is slow, a token cache that re-authenticates on 401 during an identity-provider wobble - all three are dynamic in exactly the way that hurts, and all three are fixed by the same move: fixed cadence, cached value, alarm on failure.

### Bimodal behaviour as the observable symptom

The reason Source C frames this as "bimodal behavior" rather than "insufficient capacity" is that bimodality is *observable from outside*, which makes it testable. You cannot see static stability by reading an architecture diagram. You can see bimodality in a graph: the system has one shape at 10:00 on a Tuesday and a different shape during an incident.

> "For example, you might try and recover from an Availability Zone failure by launching new instances in a different Availability Zone. This can result in a bimodal response during a failure mode. You should instead build workloads that are statically stable and operate within only one mode. In this example, those instances should have been provisioned in the second Availability Zone before the failure. This static stability design verifies that the workload only operates in a single mode."

So the working test for the property is: **does the system have one mode?** If the answer is "it has a normal mode and a degraded mode", the question becomes whether the transition between them requires a call, and if it does, the system is not statically stable regardless of how well the transition is rehearsed.

### The cache-bypass anti-pattern

Source C names the second-most-common violation, and it is worth quoting because it is usually done with good intentions:

> "Another example of bimodal behavior is allowing clients to bypass your workload cache when failures occur. This might seem to be a solution that accommodates client needs but it can significantly change the demands on your workload and is likely to result in failures."

A cache that clients can bypass on error is a system that converts a cache problem into a database problem, on the theory that the database is more reliable. Sometimes it is. But the bypass is a mode change, it is triggered by failure, and it multiplies load on the tier least able to absorb a surprise during an incident. The statically stable alternative is the constant-work pattern: serve the stale value, alarm, and let a human decide whether the staleness is acceptable - after the event, not inside it.

The general shape of every anti-pattern in this section is the same: **a failure in one component is converted into a behaviour change in another.** Static stability is the discipline of refusing that conversion.
---

## 3. The Control Plane and the Data Plane, Concretely

The distinction is old, and AWS says so. Source E traces it to its actual origin rather than presenting it as a cloud invention:

> "AWS separates most services into the concepts of control plane and data plane. These terms come from the world of networking, specifically routers. The router's data plane, which is its main functionality, is moving packets around based on rules. But the routing policies have to be created and distributed from somewhere, and that's where the control plane comes in."

That etymology is not decoration. The router's forwarding path was separated from its routing protocol for the same reason static stability exists: forwarding must not stop when the protocol converges, and the protocol is where the complexity lives. Every cloud control plane is a routing protocol with a database attached.

### The two definitions

Source B gives the operational pair, and these are the definitions used throughout this guide:

> "A control plane is the machinery involved in making changes to a system - adding resources, deleting resources, modifying resources - and getting those changes propagated to wherever they need to go to take effect. A data plane, in contrast, is the daily business of those resources, that is, what it takes for them to function."

Two things follow immediately. First, the control plane includes *propagation*, not just the API call - a change that was accepted but not yet propagated is a change that has not happened. Second, the data plane is defined by what resources do, not by what they are, which is why the same service can be a control plane in one interaction and a data plane in another. Creating a DynamoDB table is control plane. Reading an item from it is data plane. The line is drawn per-operation, not per-service.

### What each plane actually does

Abstractions hide the asymmetry, so Source B makes it concrete with EC2. Launching one instance is not one operation:

- finds a physical server respecting placement group and VPC tenancy requirements;
- allocates a network interface out of the VPC subnet;
- prepares an EBS volume;
- generates IAM role credentials;
- installs security group rules;
- stores results in data stores of downstream services;
- propagates configurations to the server and the network edge.

Seven classes of work, across at least five downstream systems, for one instance. The data plane, by contrast, does two things:

- routes packets per the VPC route tables;
- reads and writes EBS volumes.

That is the whole comparison, and it is the argument. A packet arriving at a running instance touches a route table lookup and a block device. A packet does not consult the placement service, the IAM credential generator, or the network edge configuration store. **The data plane's correctness was decided at launch time and frozen into the running resource**, which is precisely why the running resource survives things the launch path does not.

Source A states the same point with an analogy that is worth keeping because it is pre-cloud:

> "An example of static stability can be found in Amazon EC2. Once an EC2 instance has been launched, it is just as available as the physical server in a data center. It does not depend on any control plane APIs in order to stay running, or to start running again after a reboot."

A reboot is the interesting case. Rebooting an instance does not re-run the launch path - it does not re-run placement, or re-allocate the network interface, or re-generate credentials. The instance restarts from its own state, on its own host, using its own configuration. That is what "no dependency on the control plane" means operationally, and it is why a fleet of running instances is a different kind of asset from a launch template that could produce one.

### The availability asymmetry, quoted

This is the property that makes the concept necessary rather than merely tidy. Source E:

> "Data planes are intentionally less complicated, with fewer moving parts compared to control planes, which usually implement a complex system of workflows, business logic, and databases. This makes failure events statistically less likely to occur in the data plane versus the control plane. While both the data and control plane contribute to the overall operation and success of the service, AWS considers them to be distinct components. This separation has both performance and availability benefits."

And Source B, more specifically for EC2:

> "the Amazon EC2 data plane is far simpler than its control plane. As a result of its relative simplicity, the Amazon EC2 data plane's design targets a higher availability than that of the Amazon EC2 control plane."

Read those two sentences together and the design constraint falls out. The data plane is not merely *usually* healthier; it is *engineered* to a higher availability target, and it is engineered that way because it is simpler. The simplicity is not an aesthetic property, it is the causal mechanism. Fewer moving parts means fewer independent failure modes, fewer cross-service dependencies, and a smaller configuration space in which to be wrong.

### Why, structurally

Four mechanisms produce the asymmetry, and all four are consequences of the control plane doing more.

**More moving parts.** Source E's "complex system of workflows, business logic, and databases" is a description of a distributed transaction. Creating a resource is a saga: allocate, reserve, persist, propagate, confirm, and compensate on partial failure. Each step can fail, and each failure leaves state that must be reconciled. The data plane has no saga because it has nothing to create.

**Cross-service coupling.** The EC2 launch path in Source B touches placement, networking, block storage, identity, security groups, and downstream data stores. A control plane is therefore only as available as the union of its dependencies, and its blast radius is the union of theirs. A data plane's dependencies are its own hardware and the route table - both local by construction.

**Orchestration is where correctness lives.** Source E is blunt: "Control planes tend to be complicated orchestration and aggregation systems." Source D repeats it as a design warning: "Control planes tend to be complicated orchestration." Two AWS documents independently choosing the word *complicated* for the same component is not stylistic. Complexity is where bugs live, and the control plane is where the complexity was deliberately concentrated so that the data plane could be simple.

**Change velocity.** The data plane changes rarely and carefully, because changing it breaks everything at once. The control plane ships features constantly, because that is what customers ask for. A component that ships weekly has a higher defect rate than a component that ships yearly, independent of design quality. This is an inference rather than a quoted AWS claim, and it is marked as such in Section 14.

### The three mechanisms that make it worse

The asymmetry is bad on its own. Three properties of how AWS builds control planes make it worse in exactly the situation that matters - a zonal or regional failure.

**Control planes are regionally scoped.** In most AWS services the control plane for a Region lives in that Region, and some of its state lives in a single Region globally. Source D gives the sharpest documented example: the Route 53 management APIs "run on control planes that are designed to prioritize the strong consistency and durability that you need when managing DNS. To achieve this, the control planes are located in a single Region: US East (N. Virginia)." A regional event therefore does not merely degrade the control plane - for some services it takes the control plane out entirely, and for others it means the control plane for the affected Region is reachable only through infrastructure in the affected Region.

**Throttling and retries bite hardest at the worst moment.** Control planes are rate-limited by design, because a control plane without limits is a control plane that can be taken down by a retry storm. The limits are calibrated for normal operations. During a regional event, every workload in the Region simultaneously retries, every automation fires, and every human with a console open refreshes. The control plane is then asked to do several times its normal work at the moment it is least able to, and its response is to throttle - which triggers more retries. This is not a hypothetical dynamic; it is the standard shape of a control-plane brownout.

**Demand is correlated.** The other callers are not random. They are every other customer in the Region running the same recovery playbook against the same API, which is the same API your recovery path depends on. On the data plane, your traffic is your traffic. On the control plane during an incident, your traffic competes with everybody else's emergency.

### The Availability Zone, defined

The failure domain the whole argument turns on, in Source B's wording:

> "Availability Zones are logically isolated sections of an AWS Region: Each AWS Region has multiple Availability Zones that are designed to operate independently. Availability Zones are physically separated by a meaningful distance to protect from correlated impact from potential issues such as lightning strikes, tornadoes, and earthquakes. They don't share power or other infrastructure, but they are connected to each other with fast, encrypted private fiber-optic networking to enable applications to quickly fail over without interruption."

The design intent is a *statistical* independence claim: separate power, separate flood plain, separate fault lines. What it is not is an independence claim about the control plane. Zones are isolated from each other; the control plane that manages resources in all of them is not zonal. That gap is the entire subject of Section 5.

### The nuance that keeps the argument honest

Source B supplies a caveat that prevents the naive reading "stateless means we do not need to worry":

> "Although each of the zonal architectures described earlier shows the entire stack contained within a single Availability Zone, we replicate any hard state across multiple Availability Zones for disaster recovery purposes. For example, we typically store periodic database backups in Amazon S3 and maintain read replicas of our data stores across Availability Zone boundaries. These replicas aren't necessary for the primary Availability Zone to function."

Statelessness in the compute tier is not statelessness of the data. The replicas exist, they are cross-zone, and they are explicitly *not* on the primary's serving path. That is the pattern in miniature: the durable copy is pre-arranged and cold, and the serving path never waits for it. A design that promotes a replica in order to serve is a different design, and Section 6 prices the difference.

---

## 4. The Canonical Contrast: Two Designs, One Zone Failure

Two designs, one failure, mechanical treatment. Both are drawn from Source B's own framing, because the contrast it draws is the clearest thing written on the subject.

The failure: a single Availability Zone stops accepting traffic, at 02:14 on a Tuesday, with no warning and no partial degradation beforehand. The control plane's state during this event is *unknown* - possibly healthy, possibly throttled, possibly impaired. That unknown is the whole point, and Design A will be evaluated under all three.

### Design A - the dynamic design

An Auto Scaling group with instances spread across three zones, minimum capacity sized for steady-state load, an elastic load balancer with cross-zone balancing and per-zone health checks, and a scaling policy that adds capacity when healthy capacity drops. On paper this is a textbook multi-AZ design and it passes most architecture reviews.

Source B names it as the anti-pattern:

> "A less effective approach might be to deploy to multiple Availability Zones with the expectation that, should there be an impairment within one Availability Zone, the service will scale up (perhaps using AWS Auto Scaling) in other Availability Zones and be restored to full health. This approach is less effective because it relies on reacting to impairments as they happen, rather than being prepared for those impairments before they happen. In other words, it lacks static stability."

Minute by minute, what has to go right:

**T+0 to T+1 - traffic shifts.** The load balancer's health check for zone A fails, its nodes are removed from rotation, and traffic redistributes to zones B and C. This is a **data plane** operation, it is local, and it works. It is the one link in the chain that is genuinely reliable, and it is the reason the design feels safe.

**T+1 to T+2 - the remaining zones are now serving 100% of load.** Zones B and C were each sized for roughly a third of steady-state demand, because the ASG's minimum was sized for the total across three zones with even spread. They are now each asked for half of the total. Whether they can serve it depends on headroom per instance, connection limits, thread pools, downstream connection pooling, and whether the service degrades gracefully or collapses. **This link fails silently and often**, because per-zone headroom is rarely modelled and load balancers will happily send traffic to a zone that is drowning.

**T+2 to T+3 - the scaling policy fires.** An alarm on healthy capacity, or a target-tracking policy, decides to add capacity. This is step 2 of the dynamic sequence: a **decision**. It is made from an alarm that fired on a partial signal, and it can be wrong in both directions - too slow, or a runaway that adds capacity the downstream tiers cannot absorb.

**T+3 - the launch call goes to the EC2 control plane.** This is the link the design's authors never examined. The call is regionally scoped. If the event that removed zone A also degraded the regional control plane - which is the case the design was supposed to cover - the call fails or hangs. If the control plane is healthy but saturated by every other workload in the Region executing the same logic, the call is throttled. Retries make it worse, and a `RunInstances` that succeeds on the fourth attempt has cost three timeouts' worth of wall clock.

**T+3 to T+8 - capacity must exist.** Even a successful API call only reserves capacity. The instances must be placed on physical hosts that exist in zones B and C, with the required instance type available, in an account with sufficient vCPU quota in that family. During a large event, the surviving zones are exactly where everyone else is trying to launch. Source D's warning is not theoretical: "avoid provisioning new instances manually or instructing Auto Scaling Groups to add instances in response."

**T+8 to T+12 - instances must boot and warm.** AMI resolution, user data, package or container image pull, JIT warmup, cache population, connection pool establishment, and any startup dependency the service has - including, recursively, calls to *other* control planes for credentials or configuration. A cold JVM is not serving at full rate for a while after it passes its health check.

**T+12 onward - registration and health check pass.** The instance registers with the load balancer, passes its health check threshold, and enters rotation. Every one of those steps is another chance to fail.

**The aggregate.** Design A's recovery is a chain of seven links, of which exactly one (traffic withdrawal) is a data plane operation. The other six are control-plane calls, capacity assumptions, warmup times, or health-check thresholds. The chain's reliability is the product of its links. If each link is 95% likely to work under stress - a generous estimate for a throttled API and a cold start - the chain succeeds about 70% of the time, and the 30% case is an outage that the architecture review believed was handled.

### Design B - the static design

Same three zones, same load balancer, different capacity decision: each zone is sized for **N-1**, meaning each zone alone can carry the load that the whole estate would carry with one zone missing. Nothing is launched during the event. Nothing is decided. The only actor is the load balancer withdrawing a zone, which is a data plane operation and was already configured to happen.

Source B's positive form:

> "a more effective, statically stable service would overprovision its infrastructure to the point where it would continue operating correctly without having to launch any new EC2 instances, even if an Availability Zone were to become impaired."

The steps that **disappear** rather than accelerate:

| Step in Design A | What happens in Design B |
|---|---|
| Detect reduced healthy capacity | No alarm is on the serving path; detection is telemetry only. |
| Decide to scale | Deleted. There is no scaling policy for zone loss. |
| Call `RunInstances` / scaling API | Deleted. No control-plane call exists. |
| Wait for capacity to be available | Deleted. Capacity is already allocated and paid for. |
| Boot and warm | Deleted. The surviving instances have been warm for weeks. |
| Register and pass health checks | Deleted. They were already registered and already passing. |
| Surviving zones absorb 100% | **Retained**, but pre-solved: they were sized to absorb it. |
| Traffic shifts off the failed zone | **Retained**, unchanged, data plane. |

Six of eight steps are removed outright. The two that remain are the two that were always reliable. That is what static stability buys, and it is not speed - it is the elimination of a failure surface.

### The honest counter-case

Design A is cheaper, and for a large class of failures it is genuinely adequate. The guide is worthless if it pretends otherwise, so here is the list of failures where Design A is the correct engineering answer:

- **A single instance fails.** The ASG replaces it in a healthy zone with a healthy control plane. Nothing about the event impairs the launch path. Design B would have paid for capacity it did not need.
- **A single process wedges** (memory leak, deadlock, thread starvation). Same reasoning. This is the most common real failure by a wide margin, and Design A handles it well.
- **A bad deployment** rolls out. The fix is a rollback, which is a control-plane operation by definition and is not made worse by Design A - the control plane is healthy, and the deploy pipeline is the thing being used regardless of design.
- **Load grows gradually** over weeks. Autoscaling is the correct response, and static provisioning would either over-pay or be wrong at both ends of the curve.
- **A dependency is slow but up.** Retries, circuit breakers and timeouts do the work; neither design is decisive.

What all five have in common: **the control plane is healthy when the response runs.** Design A's assumption is not "failures never happen"; it is "when a failure happens, the API I need will be available". That assumption holds for instance-level and deploy-level failures and fails precisely for the events that justify the multi-AZ architecture in the first place. Design A is not wrong. It is optimised for a different failure set than the one its diagram advertises.

The correct conclusion is not "Design B everywhere". It is: **the design must match the failure it claims to survive**, and a design whose recovery path runs through a control plane cannot claim to survive an event that impairs control planes.
---

## 5. Why Reaction Through an Impaired Control Plane Is Worse Than No Reaction

This is the section the guide exists for. Everything before it establishes that the two planes are different and unequally reliable. This section argues the consequence: **a design that must ask the control plane to react has made its recovery path depend on a service that is itself impaired.** And a recovery path that depends on an impaired service is worse than no recovery path, because it consumes the time and attention that a truthful "we cannot recover" would have freed.

### The strongest documented evidence

Source D is an AWS Well-Architected best practice titled "Rely on the data plane and not the control plane during recovery". It opens with the taxonomy and then states the operational rule:

> "Control planes provide the administrative APIs used to create, read and describe, update, delete, and list (CRUDL) resources, while data planes handle day-to-day service traffic. When implementing recovery or mitigation responses to potentially resiliency-impacting events, focus on using a minimal number of control plane operations to recover, rescale, restore, heal, or failover the service. Data plane action should supersede any activity during these degradation events."

"Minimal number" and "should supersede" are not decorative. This is a best practice inside the Reliability Pillar telling architects to route recovery around the control plane. Its risk statement is equally direct:

> "Medium: For certain types of service degradations, control planes are affected. Dependencies on extensive use of the control plane for remediation may increase recovery time (RTO) and mean time to recovery (MTTR)."

But the passage that converts this from guidance into evidence is the Route 53 case, which is the only place in the retrieved corpus where AWS describes, concretely, a production system in which the data plane and the control plane have materially different availability characteristics - and says so in its own words:

> "Route 53 routing policies use the control plane, so do not rely on it for recovery. The Route 53 data planes answer DNS queries and perform and evaluate health checks. They are globally distributed and designed for a 100% availability service level agreement (SLA)."

> "The Route 53 management APIs and consoles where you create, update, and delete Route 53 resources run on control planes that are designed to prioritize the strong consistency and durability that you need when managing DNS. To achieve this, the control planes are located in a single Region: US East (N. Virginia). While both systems are built to be very reliable, the control planes are not included in the SLA. There could be rare events in which the data plane's resilient design allows it to maintain availability while the control planes do not. For disaster recovery and failover mechanisms, use data plane functions to provide the best possible reliability."

Read that last paragraph slowly, because it contains four distinct admissions in six sentences. The control plane is **single-Region**. It is **outside the SLA**. Its design goal is **consistency and durability**, not availability - a different optimisation target, deliberately chosen. And there exist events in which **the data plane is up and the control plane is not**.

That is the entire thesis, published by AWS, about AWS's own DNS service. The control plane is where you would naturally implement "fail over to the other Region". It is also the component that AWS says not to rely on for exactly that. A workload whose regional failover is a Route 53 record change has built its last line of defence on the one system AWS explicitly excludes from the availability guarantee while including the query path in it.

The practical consequence is not "Route 53 is unreliable". The data plane is genuinely excellent, and the SLA is real. The consequence is narrower and more useful: **the correction you would make during a failure is a call to the component that is least available during a failure.** Source D supplies the data-plane substitute in its conversion table (Section 6): a Route 53 ARC routing control rather than a record change.

### The argument, in five parts

**(a) It converts a bounded failure into an unbounded one.** A zone loss is bounded by definition: zones are independent, so the failure has a known shape and a known scope. A recovery path through an impaired control plane has no such bound, because AWS publishes no recovery-time objective for control planes. You are waiting on a system whose recovery time is undocumented, using a mechanism whose failure mode is "still waiting". The estate's outage duration becomes a function of a variable nobody has measured. That is strictly worse than a design that says "we have capacity for one zone, and beyond that we are down" - the second design has a known capacity bound, and known bounds can be planned around.

**(b) It introduces a decision at the moment information is worst.** Step 2 of the dynamic sequence is a branch, and branches need information. During a regional event the information is degraded in a specific way: telemetry lags, alarms are noisy, dashboards show the surviving zone's success rate rather than the failed zone's error cause, and the operator is being told by three different systems that three different things are broken. Source D names the failure directly: "Dependence on changing DNS records to re-route traffic." The decision can also be *wrong in a durable way* - fail over to a zone that is also impaired, or promote a replica whose lag was never checked, and now you have two problems and less capacity.

**(c) It cannot be tested in the normal case.** A path that only runs during events is exercised a handful of times a year at most, and never under the conditions that matter. Source F's honest warning about the automated version applies to the manual version with more force:

> "Relying on an automated health check to control Regional failover can be tricky. A health check might not be perfectly reliable if a Region is experiencing some type of degradation."

The partial remedy is the SCP-based simulation described in Section 8 - deny the API and see whether the estate notices. That is genuinely useful, and it is a simulation: it proves the estate survives API denial, not that it survives a regional event in which the API denial is one of several simultaneous problems.

**(d) It creates correlated demand.** Every workload in the Region that has a reaction path executes it. The control plane's capacity was provisioned for steady-state API traffic, not for a simultaneous retry storm from every tenant. The recovery script that works beautifully in a game day with one participant fails when ten thousand participants run it at once. This is not a fault in anyone's design; it is a property of shared infrastructure under correlated stress, and it is why the data plane - where your traffic is only your traffic - is the safer place to put the response.

**(e) Failure of the recovery action is silent by default.** `RunInstances` returning `InsufficientInstanceCapacity`, a scaling activity that never completes, a health check that never goes healthy - none of these raise an alarm in a default configuration. They surface as a service that is still degraded. The operator learns that the recovery did not work by noticing that things are still broken, which may be many minutes after the recovery was attempted, and by then the incident clock has burned the time the recovery was supposed to save. A static design has no such step to silently fail, which is not the same as being invulnerable - it is being invulnerable to *that* class of silent failure.

### The prescription AWS actually gives

Having established the asymmetry, Source D does not leave the reader to infer the design. It states it:

> "Design your compute infrastructure to be statically stable to avoid using the control plane during an incident. For example, if you are using Amazon EC2 instances, avoid provisioning new instances manually or instructing Auto Scaling Groups to add instances in response. For the highest levels of resilience, provision sufficient capacity in the cluster used for failover. If this capacity threshold must be limited, set throttles on the overall end-to-end system to safely limit the total traffic reaching the limited set of resources."

Every clause in that paragraph is doing work. "Avoid provisioning new instances manually" rules out the operator-with-a-console pattern. "Or instructing Auto Scaling Groups to add instances in response" rules out the automated version, which is the one most teams believe is exempt because it is not a human. "Provision sufficient capacity in the cluster used for failover" is the static requirement. And the final sentence is the graceful degradation clause: if you cannot afford the capacity, bound the demand rather than promising a capacity you do not have.

Source F makes the same point from the disaster-recovery direction, and is unusually candid about the trap:

> "The Availability and Beyond whitepaper discusses the concept of static stability for improving resilience. What does static stability mean with regard to a multi-Region disaster recovery (DR) plan? What if the very tools that we rely on for failover are themselves impacted by a DR event?"

> "Both dependencies might violate static stability, because we are relying on resources in our DR plan that might be affected by the outage we're seeing. Ideally, we don't want to depend on other services running so we can failover and continue to serve our own traffic."

"What if the very tools that we rely on for failover are themselves impacted by a DR event" is the question this section exists to ask. Most DR plans have an answer that is a procedure, and a procedure is a set of calls.

### The documented/undocumented ledger

This is the single most important honesty requirement in the guide, because the strong version of the thesis is very tempting and is not what AWS says.

**What AWS documents, in its own words:**

1. Control planes are "statistically more likely to fail than data planes" (Source A).
2. Data planes "design targets a higher availability" than control planes, because they are simpler (Source B, Source E).
3. "For certain types of service degradations, control planes are affected" (Source D's risk statement).
4. For Route 53 specifically, the control plane is single-Region, outside the SLA, and there are events in which the data plane is available and the control plane is not (Source D).

**What AWS does not document:**

- That the control plane is *usually* or *frequently* impaired when a data plane fails. No retrieved source says this, in any wording.
- Any quantified probability of control-plane impairment, conditional on any class of failure. There is no published number, and the word "statistically" in Source A is not accompanied by a statistic.
- Any per-service statement of which control planes are regional, which are global, and which have zonal components. The Route 53 case is documented; the general pattern is not tabulated.
- Any recovery-time objective for control planes. This absence is itself the point of argument (a), and it is an absence in the documentation, not a documented absence of capability.

So the defensible claim, and the one this guide defends, is: **AWS states that control planes are less available than data planes, engineers data planes to a higher target, and confirms for its own DNS service that events exist in which the data plane outlives the control plane.** From that, the engineering conclusion follows without embellishment: a recovery path through the control plane is a path through the less available component.

The stronger claim - "when an AZ fails, the regional control plane is frequently impaired too" - is a **practitioner inference**, widely believed, consistent with the mechanisms in Section 3 (regional scope, throttling under correlated demand), and not publicly documented at that strength. It is marked as inference here and again in Sections 13 and 14. A guide that inflates this into a documented fact would be doing exactly what Section 12 accuses others of.

The reason the inference is worth stating at all, even flagged, is that it changes what evidence you would accept. If you believe the strong version, a zone-failure drill that leaves the control plane healthy is a *misleading* test, not a passing one. If you accept only the documented version, the same drill is a legitimate test of capacity and a silent non-test of the thing you care about. Either way the conclusion for design is the same - do not put the recovery on the control plane - which is precisely why the concept survives the uncertainty.

---

## 6. The Patterns That Achieve Static Stability, and What Each Costs

There is no single pattern. There is a family, and each member buys something specific at a specific price. The table is the summary; the prose is where the prices are honest.

| Pattern | What it buys | What it costs |
|---|---|---|
| **Pre-provisioned capacity across zones (N-1)** | Removes the launch step entirely. The surviving zones already carry the load. | Idle capacity, continuously billed. The N-1 sizing must be right or the pattern is decorative. |
| **Active-active rather than active-passive** | No promotion decision, no standby that might fail to start. | Both sides carry full load; data consistency work; the discipline of keeping the standby not-cold. |
| **Cell-based / partition isolation** | Blast radius bounded; a cell failure consumes a bounded share of capacity. | Cell-aware routing, uneven cell load, cross-cell data, and a term AWS does not define. |
| **Deliberate refusal to automate the failure path** | No automation running blind during a regional event. | A human in the loop, acceptable only if the human action is itself data-plane and pre-rehearsed. |
| **Routing that was already correct** | Removes the record-change step. | You accept static weights or a data-plane routing control, and lose fine-grained traffic steering. |

### Pre-provisioned capacity, and what N-1 actually means

The core pattern, and the one Source B states as the positive form: capacity sized so that the estate continues operating "without having to launch any new EC2 instances, even if an Availability Zone were to become impaired."

The sizing rule matters more than the slogan. **N-1 is not 2N.** For a three-zone estate where each zone must survive the loss of one peer, each zone needs capacity for the load that three zones would carry divided by two - one and a half times its steady-state share, not twice it. For a two-zone estate, N-1 means each zone carries 100% of load at rest, which is 2N and is why two-zone static designs are expensive. This is the arithmetic behind Source C's most useful economic observation:

> "It is also possible to implement a more statically stable design by using more resources in each zone. By adding more zones, you reduce the amount of additional compute you need for static stability."

Four zones sized for N-1 need each zone to carry four thirds of its normal share - a third more, not a half more. Six zones need a fifth more. The idle-capacity premium falls as the zone count rises, which means the cheapest static design is often a wider one. That is a genuinely counter-intuitive result and it is the single best cost lever available.

The failure mode of this pattern is sizing that was never verified: capacity is described as N-1 in a document, and no one has ever loaded a single zone to the level it would see during a peer's loss. Section 8's first test exists for exactly this.

### Active-active, and the standby that is not cold

Active-passive looks static because the standby "already exists". It usually is not, for a reason that has nothing to do with capacity: the promotion is a decision and a control-plane call. Source D lists it as an anti-pattern twice over - "Dependence on control-plane scaling operations to replace impaired components due to insufficiently provisioned resources" and "Relying on extensive, multi service, multi-API control plane actions to remediate any category of impairment."

Active-active removes the promotion by making both sides serve. Its cost is not capacity - both sides were going to be paid for either way - it is the data tier. Two active writers need conflict resolution, or a single-writer region with cross-region reads, or a partition-tolerant store. That is real engineering, and it is where most active-active programmes stall: the compute tier goes active-active in a quarter, and the database stays active-passive for two years, which means the estate is active-passive with extra steps and an unexercised promotion path.

The related discipline is that a standby must not be *cold*. A passive region that has not served traffic in six months has untested configuration, expired certificates, drifted security groups, and an autoscaling history that says nothing about its real capacity. "We have a DR region" and "we have a DR region that would work" are different claims, and only the second one is a resilience property.

### Cell-based design, and the term AWS does not define

**Cell-based architecture** partitions a workload into independently failing cells, each sized so its failure consumes a bounded fraction of total capacity, with routing that keeps a request inside one cell. It buys the thing zone-based design only approximates: a blast radius that is bounded by construction rather than by luck, and a failure that is contained to one cell's share of traffic instead of a third of the estate.

Its costs are real. Routing must be cell-aware - usually a shard key at the edge, which is a data-plane lookup but a new piece of infrastructure. Load distributes unevenly across cells and unevenly over time. Data that must be shared across cells reintroduces the coupling the cells were meant to remove. And cell size becomes a capacity decision that must be revisited as the estate grows, because a cell that was 5% of traffic at design time is 20% of traffic two years later if cells are added slowly.

One honesty note that matters for citation discipline: **AWS does not define "cell" as a formal isolation boundary.** The Fault Isolation Boundaries whitepaper's named boundaries are Availability Zone, Region, control plane and data plane. Cell-based design is engineering convention, widely used and well understood, and it is not an AWS term. It is treated here as convention, and the deeper treatment lives in the chaos and resilience engineering literature rather than in a provider document - see [chaos_engineering_guide.md](chaos_engineering_guide.md) for injection practice and the blast-radius framing.

### The refusal to automate

The cheapest pattern and the most counter-cultural: **do nothing automatically on the failure path.** No Lambda that reacts to an alarm by creating something. No autoscaler whose target is defined in terms of zone loss. No script that promotes a replica. Source B's anti-pattern is precisely the automation, not the incompetence.

What it buys is the removal of an entire class of failure - automation that runs blind, at scale, against a degraded API, with no one watching. What it costs is a human, and a human is only acceptable if the human's action is itself a data-plane operation that was rehearsed. "A human decides to change a DNS record" is not this pattern. "A human presses a pre-provisioned ARC routing control, having done it in a game day last quarter" is.

There is a further honesty point: refusing to automate is not the same as refusing to act. A human deciding to *shed load* or *disable a feature* is often the correct response, and both are data-plane operations on your own estate. The refusal is specific to automation that creates or reconfigures infrastructure during the event.

### Routing that was already correct

The last pattern removes the record-change step by never needing it. Two forms.

**Static weights**: traffic is split across zones or Regions at fixed proportions, chosen so that the loss of one target leaves the remainder able to serve. Nothing changes during the failure; the load balancer or DNS simply stops returning the failed target. This is elegant and it gives up fine-grained steering - you cannot shift 5% to a canary by editing weights if the weights are load-bearing.

**A data-plane routing control**: Source D names the substitute explicitly, "Route 53 Record change (control plane) or Amazon Application Recovery Controller (data plane)". Source F describes what that means mechanically:

> "It provisions a Route 53 health check that we can manually control with a Route 53 ARC routing control, and is a data plane operation. The Route 53 ARC data plane is highly resilient, using a cluster of five Regional endpoints. You can revise the health check if three of the five Regions are available."

Five Regional endpoints, three of five required. That is a data plane designed for the job the control plane cannot do, and it is the clearest concrete instance in the retrieved corpus of AWS building a separate path specifically so that failover does not depend on the management API.

### The conversion table

Source D's table is the practical core of this section, and it is reproduced verbatim because paraphrasing it loses the specificity:

| Control-plane action | Data-plane equivalent |
|---|---|
| Auto Scaling (control plane) to pre-scaled Amazon EC2 resources (data plane) | Pre-scale the fleet so that no scaling call is needed during the event. |
| Amazon EC2 instance scaling (control plane) to AWS Lambda scaling (data plane) | Let the serverless platform scale within provisioned concurrency rather than adding instances. |
| "Assess any designs using Kubernetes and the nature of the control plane actions. Adding pods is a data plane action in Kubernetes. Actions should be limited to adding pods and not adding nodes. Using over-provisioned nodes is the preferred method to limit control plane actions." | Over-provision nodes; scale pods, never nodes, during an incident. |
| Route 53 Record change (control plane) or Amazon Application Recovery Controller (data plane) | Pre-provision an ARC routing control and flip it. |

The Kubernetes row is the one to reread, because it makes the distinction concrete in a system most estates run. Adding a pod is a data-plane operation: the scheduler places it on a node that already exists and is already configured. Adding a node is a control-plane operation: it provisions infrastructure, joins it to the cluster, and pulls images. A cluster with no headroom in its nodes cannot add pods, so the "pods not nodes" rule only holds if nodes were over-provisioned - which is the same N-1 argument, one layer down.

Source D also applies the rule to managed services, with a warning that catches people out:

> "For services like Amazon DynamoDB, Amazon API Gateway, load balancers, and AWS Lambda serverless, using those services leverages the data plane. However, creating new functions, load balancers, API gateways, or DynamoDB tables is a control plane action and should be completed before the degradation as preparation for an event and rehearsal of failover actions."

Using a Lambda function is data plane. Creating one is control plane. An estate that spins up a new function per tenant, or a new API Gateway per release, has put control-plane calls on a path it may need during an incident - and has probably done so for reasons that had nothing to do with resilience.

### Regional-calls-regional versus zonal

Source B distinguishes two architectural patterns, and the distinction is about what kind of service you are.

> "The simpler pattern: regional-calls-regional. This is often the best choice for external-facing services, and appropriate for most internal services as well."

> "The more complex pattern: regional-calls-zonal or zonal-calls-zonal. When designing internal, and in some cases external, data plane components within Amazon EC2 (for example, network appliances or other infrastructure that sits directly in the critical data path) we follow the pattern of Availability Zone independence and use instances that are siloed in Availability Zones, so that network traffic remains in its same Availability Zone. This pattern not only helps keep impairments isolated to an Availability Zone but also has favorable network traffic cost characteristics in AWS."

The reasoning is worth stating plainly, because it is not the reasoning most teams use. Zonal siloing is chosen for **data plane components on the critical path** - network appliances, gateways, things whose every packet crosses an AZ boundary if they are not siloed. For those, cross-AZ calls add latency, add cost (cross-AZ data transfer is billed), and add a dependency on the inter-AZ network. For an external-facing API, the cross-AZ hop is amortised over a request's whole lifetime and the simplicity of regional routing is worth more.

And AWS names the price of the complex pattern rather than hiding it:

> "This choice comes with the cost of additional complexity. For us in Amazon EC2, the additional complexity comes in the form of managing zonal, rather than regional, service environments. For customers of NAT gateway, the additional complexity comes in the form of having multiple NAT gateways and route tables..."

Multiple NAT gateways and multiple route tables, per zone, permanently, with the configuration drift that implies. That is the honest trade: zonal independence is bought with a multiplied configuration surface, and every multiplier is something that can be wrong in one zone and right in another.

---

## 7. The Cost of Static Stability, Stated Plainly

Static stability is bought with idle capacity, and the idle capacity is a real line item that appears on a real invoice every month, while the benefit is probabilistic and appears at most a few times a year. Any guide that does not lead with that is selling something. This section is the price list.

### (i) The capacity you pay for and do not use

The bill is continuous; the benefit is not. That asymmetry is the whole political problem of static stability, and it is why the conversation always ends up in a budget meeting rather than an architecture review.

The sizing rule from Section 6 is the first lever: **N-1 per zone, not 2N.** For a three-zone estate, each zone carries 1.5x its steady-state share, so the estate runs at roughly two-thirds utilisation at rest instead of one-third. That is a 50% capacity premium, not a 100% premium, and it is the difference between a proposal that survives finance and one that does not.

The second lever is Source C's zone-count observation: "By adding more zones, you reduce the amount of additional compute you need for static stability." A four-zone N-1 design needs each zone at 4/3 of its share - a 33% premium. Six zones need 6/5 - a 20% premium. Wider is cheaper per unit of resilience, which inverts the usual instinct that more zones means more spend.

The third lever is the one nobody likes: **the premium is not a one-time decision.** It is a standing commitment that must be maintained as traffic grows, as instance families change, and as zone weights drift. A static design that was correctly sized in year one and never revisited is not a static design in year three - it is an under-provisioned dynamic design with extra steps.

And the honest note on the arithmetic: the premium computed above is a capacity premium, not a cost premium. Spare capacity in the form of a smaller number of larger instances, or of committed-use discounts on baseline capacity, costs less than the naive multiplication suggests. Reserved and savings-plan pricing exists precisely because baseline capacity is predictable, and a statically stable estate is the most predictable capacity profile there is. That is a genuine offset and it is not large enough to make the premium disappear.

### (ii) The rigidity

Capacity baked into a zone cannot follow demand that has moved. If a statically stable estate was weighted 34/33/33 and traffic shifts so that zone A now receives 50% of requests, the design is quietly broken: zone A is over-loaded relative to its peers, its N-1 neighbours are sized for the old distribution, and the estate's actual resilience is lower than its documentation claims.

This produces two costs. The first is maintenance: zone weights must be revisited whenever traffic distribution shifts, which means someone must own the distribution as a monitored quantity rather than a diagram annotation. The second is subtler and worse - **over-provisioning hides a capacity model that has stopped being right.** An estate with generous headroom will keep serving after its model has drifted, which removes the signal that would have prompted a review. The estate is resilient and wrong at the same time, and nobody finds out until the failure domain is larger than the headroom.

### (iii) The complexity AWS itself names

This is not the guide's complaint; it is AWS's, in Source B: "managing zonal, rather than regional, service environments", and for customers, "having multiple NAT gateways and route tables". Every zone dimension you add is a configuration surface that exists three or four times, with three or four opportunities to drift.

The complexity is not just configuration volume. It is that zonal designs make *state* harder to reason about: which zone holds the authoritative copy, what happens to in-flight sessions when a zone withdraws, whether a per-zone cache is warm, whether per-zone connection pools to a shared database are sized for the doubled load. Regional designs collapse all of those questions into one instance of each. Zonal designs multiply them by the zone count and then require that the multiplication be correct.

### (iv) The operational cost

More things to patch, because there are more hosts and they are idle. More idle hosts to keep current, because an idle host still needs its AMI refreshed, its agent updated, its certificate rotated. An estate where nothing ever gets exercised by load, which means latent configuration errors accumulate without ever surfacing - the idle zone that has never served a full share of traffic has never had its connection pool tested, and it will discover the limit during the event.

There is also a human cost that is rarely counted: a statically stable estate has no *practice*. The dynamic estate exercises its recovery path every time an instance is replaced, which is often, and gets good at it. The static estate's recovery path runs twice a year, which means the humans who would operate it have never operated it. This is an argument for the game day, and a warning that the game day is not the same as practice.

### (v) The legitimate cases for dynamic reaction

Stated without hedging, because a guide that cannot say when its subject is wrong is not a guide:

- **Batch and analytics workloads with no RTO worth paying for.** A nightly aggregation that finishes by 06:00 has hours of slack. Paying a standing capacity premium to protect a two-hour window that has six hours of headroom is bad engineering. Dynamic is correct.
- **Development and test.** The availability target is "when the engineer needs it", and the cost of an interruption is a slack message. Dynamic, always.
- **Failures that do not correlate with control-plane health.** A single bad instance, a bad deploy, a memory leak, gradual growth - Section 4's counter-case. For these, dynamic recovery is correct, cheaper, and *more* responsive than a static design, because the control plane is healthy and the launch path is the fastest path to capacity.
- **Genuinely elastic, unpredictable demand** where the idle capacity would exceed the business value of the availability. AWS says this in its own words, in Source C: "This must be weighed against the cost for this model and the business value of maintaining the workload under all resilience cases. It's less expensive to provision less compute capacity and rely on launching new instances in the case of a failure, but for large-scale failures (such as an Availability Zone or Regional impairment), this approach is less effective because it relies on both an operational plane, and sufficient resources being available in the unaffected zones or Regions." Note the structure of that sentence: AWS does not call the dynamic approach wrong, it calls it *less expensive* and *less effective*, and leaves the trade to the reader.
- **Constrained budgets** where the correct answer is a lower availability target plus throttles, not a promise the estate cannot keep. Source D's guidance is explicit: "If this capacity threshold must be limited, set throttles on the overall end-to-end system to safely limit the total traffic reaching the limited set of resources." Throttling is a data-plane mechanism, it bounds the damage, and it is honest about the capacity you actually have.

**Static stability is the wrong choice** when the failure you are defending against does not impair the control plane, when demand is too spiky for the idle capacity to be rational, or when the business has not approved the bill. In that last case the honest move is to **record the accepted risk** - name the failure, name the consequence, name who accepted it - and not to claim static stability in a deck while running a dynamic design. The failure mode of unapproved static stability is covered in Section 10, and it is the most common way the pattern dies.
---

## 8. Testing It Rather Than Asserting It

Every property in this guide is invisible in a diagram. That is the practical difficulty: static stability is an absence, and absences do not appear in architecture reviews. The only way to know whether an estate has it is to deny something and watch.

### (a) A zone-failure simulation that leaves the control plane healthy proves nothing

This is the first principle of testing this property, and it invalidates most of what passes for multi-AZ testing. Killing instances in one zone tests the capacity path and the routing path. It does not test the dependency path, because the API that would have launched replacements was healthy the whole time - so the test cannot distinguish a static design from a dynamic one that happened to have a working control plane.

Killing instances is also, quietly, the *easiest* thing to test. It requires no permissions changes, no scoping decisions, and no risk of breaking the environment. That is exactly why it is what everyone tests, and exactly why it proves the least.

The documented technique for the harder test comes from Source F, and it is worth quoting at length because it is specific about both mechanism and danger:

- **Service control policies (SCPs)** "allows you to simulate a Route 53 or IAM control plane outage by restricting access to the service".
- **Network ACLs (NACLs)** can be used to simulate a disaster by blocking traffic to a dependency.
- The warning, in Source F's words: "create a dedicated AWS testing environment with an AWS Organizations setup. Make sure that you don't attach SCPs to your organization's root but instead create a dedicated organization unit (OU)".

That warning is not boilerplate. An SCP attached at the root denies the API to every account in the organization, including the accounts running production and including, potentially, the accounts you would need to detach it. Attach it to a dedicated OU, containing the test accounts, and only after verifying that the detach path itself does not depend on the denied API.

Source F also points at a worked implementation: the sample repository `github.com/aws-samples/r53-arc-iad`, which "evaluates Route 53 ARC capabilities if both Route 53 and IAM control planes aren't accessible." That is a precise and unusually aggressive test target - two control planes denied simultaneously - and it is the right shape of test for a failover mechanism.

One further dependency Source F flags, which catches regulated estates: federated authentication adds an identity-provider dependency to the failover path. If the operator cannot log in because the IdP is in the affected Region, the runbook is unreachable by the person who needs it. The IdP's cross-Region dependence must be tested as part of the same exercise, not assumed.

### (b) The three-part test

The property decomposes into three claims, and each needs its own test. Passing one is not passing the others.

**Test 1 - the capacity test: does the estate serve full load with a zone gone?**
Remove a zone from service and drive production-level load at the remainder. Pass condition: latency and error rate within the service's normal envelope, sustained for longer than the failure is expected to last. What this catches: zones sized for their own share rather than N-1; connection pools sized for per-zone traffic; downstream tiers that cannot absorb the redistribution. What it does not catch: anything to do with the control plane, because nothing was denied.

**Test 2 - the dependency test: does it do so without any control-plane call?**
Deny the relevant APIs using an SCP in a dedicated OU, then remove the zone. Pass condition: the estate behaves identically to Test 1. The instrumentation matters here - you are looking for *absence of calls*, so the assertion is on API call metrics and on the absence of new resource IDs, not just on availability. An estate that passes Test 1 and fails Test 2 was dynamic all along and Test 1 hid it.

**Test 3 - the procedural test: does the runbook contain any decision?**
Read the runbook and count the conditionals. Pass condition: zero branches on the serving path. This test requires no infrastructure and is the most frequently skipped, which is a poor trade because it is nearly free. A runbook that reads "verify the zone is impaired, then promote the replica in zone B" fails, regardless of whether the promotion is a click or a script. A runbook that reads "confirm the alarm is the zone-loss alarm; no action required" passes.

The three tests are cumulative and ordered by cost. Run Test 3 first - it is free and it catches most designs. Run Test 1 second. Run Test 2 only when the first two pass and the estate's criticality justifies the organisational work of an SCP in a dedicated OU.

### (c) What each test can and cannot prove

Honesty about coverage is the difference between a test programme and a ritual.

| Test | Proves | Does not prove |
|---|---|---|
| Zone removal under load (Test 1) | Capacity is N-1; routing withdraws a zone cleanly; downstream tiers absorb redistribution. | Anything about control-plane dependence. It is a capacity test wearing a resilience label. |
| SCP API denial (Test 2) | The estate survives the API being unavailable - the dependency is genuinely absent from the serving path. | Real regional degradation, where API denial is one of several simultaneous symptoms, and where the network path itself may be impaired. |
| Runbook review (Test 3) | No decision exists on the serving path, and the documented response is unconditional. | That the runbook is accurate, or that anyone can execute it under stress. |
| Full-region failover drill | The end-to-end path works when exercised deliberately, with warning, and with the control planes healthy. | That it works unannounced, at 02:14, with an impaired control plane and correlated demand from every other tenant. |

The pattern in the right-hand column is consistent: **every test that is practical to run is a proxy for a condition it cannot create.** A drill in a healthy region cannot create a degraded region. An SCP cannot create a degraded network. The mitigation is not a better test; it is combining tests whose blind spots do not overlap, and being explicit in the test record about which condition was simulated and which was assumed.

### (d) A successful test is weak evidence; a failed test is strong evidence

This asymmetry is worth stating plainly because test programmes usually treat pass and fail as symmetric.

A pass means the estate survived *this* failure, created *this* way, at *this* load, with *these* dependencies healthy. It is one sample from a large space, and the space includes conditions the test could not create. A fail, by contrast, is a concrete existence proof: the estate does not have the property, at least under the simulated condition. One failure is worth many passes, and the correct response to a failure is not to fix the test.

The corollary for reporting: a test record should say what was simulated and what was not, not simply "passed". "Zone removed under production load; control plane healthy; no API denial performed" is an honest record. "Multi-AZ failover test: passed" is a claim that will be read as much stronger than it is - by an auditor, by a new architect, and by whoever writes the post-incident review.

### (e) On injection tooling

AWS operates a managed fault-injection service (AWS Fault Injection Service, historically Fault Injection Simulator) that can run experiment templates against a target estate, including actions that stop instances, disrupt network paths, and inject API faults. It is the natural vehicle for Tests 1 and 2 at scale, and its API-level fault actions are precisely the capability that ordinary instance-killing tests lack.

It is mentioned here with a caveat that belongs in the record: **it was not retrieved or verified in this pass**, so no capability claim is made beyond its existence and general purpose. It appears in Section 15's unverified list. The mechanism it enables - deny the API and observe - is documented in Source F and is what matters; the tool is an implementation choice. For the discipline of designing injections that find real faults rather than confirming known ones, see [chaos_engineering_guide.md](chaos_engineering_guide.md).

---

## 9. The Banking and Regulated Angle

Nothing in this section re-derives the regulatory framework. [operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) owns MAS, PRA/FCA, DORA, BCBS, business impact analysis, impact tolerances and the BCP lifecycle, and it is the correct reference for all of it. What this section does is narrower: it maps one engineering pattern onto obligations that guide already describes, and it is honest about where the mapping is awkward.

### Impact tolerances become a property of the estate

An impact tolerance is a statement about how much disruption an important business service can absorb. In most programmes it is met by a combination of documentation, a recovery plan, and an assumption that the plan works.

Static stability changes the nature of the claim. Instead of "we can recover within the tolerance", the claim becomes "the tolerance is met by capacity that was already in place, so there is no recovery to perform". That is a materially stronger statement, and it is stronger for a specific reason: it does not depend on anything happening correctly during the disruption. The tolerance is satisfied by the estate's shape, which was fixed before the event and can be measured after it.

That reframing is worth real money in an impact-tolerance assessment, because it converts a claim about *process* into a claim about *capacity*, and capacity is auditable in a way that process is not.

### Severe-but-plausible scenarios

Regulators push for scenarios that are severe but plausible, and the phrase is doing work: a scenario that is severe but implausible is theatre, and a scenario that is plausible but mild is not a test. The scenario that matters here is a **full Availability Zone loss combined with control-plane impairment** - the correlated case. It is severe (a third of capacity, plus the mechanism you would use to replace it), and it is plausible for exactly the reasons in Section 3: zones are independent, control planes are not zonal, and control planes are the component AWS documents as statistically more likely to fail.

That scenario is also precisely the one dynamic designs fail, and they fail it in a way that is invisible on paper: the architecture diagram shows three zones and an autoscaling group, which reads as resilient, and the failure is in the arrow between them that nobody drew. A severe-but-plausible scenario exercise is the only routine forum in which that arrow gets examined.

### Important business services, mapped

The mapping runs from the business service down to the plane, and it has to be done per service rather than per estate.

A payments service depends on an authorization path. The authorization path's recovery - whatever it is - must not depend on an API resident in an impaired Region. Concretely, for each important business service, the question is: **if the control plane for every service in this path were unavailable for two hours, would the service still authorise transactions?** If the answer is yes, the path is statically stable and the impact tolerance is a property of the estate. If the answer is no, the tolerance depends on an API, and the dependency should be recorded as a single point of failure in the service's own risk assessment rather than left implicit in a diagram.

The output of that exercise is short and useful: a list of services, each with a yes/no and, for the "no" cases, the specific control-plane call that is load-bearing. That list is a better resilience artefact than most business impact analyses, because it is falsifiable.

### The evidence an examiner would want

Four artefacts, and they are all things a statically stable estate produces naturally as a by-product of being one:

1. **The runbook with the decisions removed.** Not a runbook that says "escalate and decide", but one whose serving-path steps are unconditional. An examiner comparing two runbooks can see the difference immediately, and the version with no branches is a stronger artefact than the version with well-documented branches.
2. **The capacity plan showing N-1 sizing per zone.** Arithmetic, per service, showing that each zone can carry the load of a peer's loss. This is checkable against the fleet's actual configuration, which is what makes it evidence rather than assertion.
3. **The test record with the control plane denied.** Specifically Test 2 from Section 8, with the simulated condition stated. A record that says "zone removed, control plane healthy" is weaker than one that says "SCP denied `ec2:RunInstances` and `route53:ChangeResourceRecordSets`; zone removed; no new resource IDs created; error rate unchanged".
4. **The change record showing zone weights were maintained.** This is the one nobody produces, and it is the one that distinguishes a maintained static design from an abandoned one. If zone weights are load-bearing, their history should show reviews after traffic shifts.

### The honest tension

Here is the awkward part, stated plainly because the guide would be dishonest to skip it.

Regulators tend to want a documented, tested, repeatable recovery procedure with named owners, defined triggers, and evidence of execution. Static stability's answer is **"there is no procedure"**. That is a harder thing to evidence. An examiner asking "show me your failover procedure" is not asking a hostile question; they are asking the question their framework is built around, and "we do not have one because we do not need one" is an answer that requires more supporting material than a procedure would.

So the evidence has to be constructed as an **absence plus a capacity proof**: here is the runbook with no decisions in it, here is why there are no decisions (the capacity is already deployed), here is the arithmetic proving the capacity suffices, and here is the test where the API was denied and nothing happened. That is a coherent evidentiary package, and it is a different shape from the package a procedure-based programme produces. Teams that assume the procedure-shaped package is the only acceptable one end up writing a failover procedure for an estate that does not need one - which reintroduces the decision the design was built to remove.

There is a second tension worth naming. Static stability raises the standing cost of an important business service, and cost is itself a regulated concern in most jurisdictions - not because regulators object to spending, but because an estate that is over-provisioned without a documented rationale looks like poor governance. The capacity plan in artefact 2 is therefore doing double duty: it is the resilience evidence and the spending justification. An N-1 plan with the accepted-risk register beside it answers both questions at once.

The relationship between the two guides is simply stated: **the regulatory guide owns the framework and the obligations; this guide owns one engineering pattern that helps satisfy them.** Neither substitutes for the other, and a programme that has the pattern without the framework has no evidence, while a programme with the framework and no pattern has evidence of a plan that may not work.

---

## 10. The Failure Modes

Five ways a static stability programme fails in practice. They are distinct from Section 12's anti-patterns, which are about *claiming* the property; these are about *building* it and getting it wrong.

| # | Failure mode | Symptom | Cause | Guardrail |
|---|---|---|---|---|
| 1 | **Static in the happy path, dynamic in the failure path** | The estate looks over-provisioned, but the scaling group's maximum is still the actual recovery mechanism. | Capacity was raised without removing the scaling policy, so the reaction path still exists and is still what runs. | Diff the serving path against the scaling configuration. If a scaling policy can still fire on zone loss, the estate is dynamic. Delete the policy, do not merely raise the maximum. |
| 2 | **A control-plane dependency nobody drew** | A design that reviews as static fails during an API denial test. | The dependency is a runbook step, a Lambda that creates something, a DNS record change, a certificate fetch at boot, or an autoscaler nobody counted. | Enumerate every API call made by the serving path and by the runbook. Any `Create*`, `Put*`, `Modify*`, `Update*`, `Associate*` on that list is a finding. Include certificate and secret fetches, which are the ones most often missed. |
| 3 | **Over-provisioned but still not statically stable** | Capacity exists and the estate still degrades. | Three variants: the routing still needs a change; the extra capacity is in the wrong zone; or the capacity is in the right zone but the load balancer was never configured to balance across zones. | Verify three things independently: (a) each zone's capacity alone exceeds the estate's requirement with one zone missing; (b) traffic actually reaches all zones under normal conditions - check per-zone request distribution, not just target health; (c) no routing change is required on the failure path. |
| 4 | **A test that faked the failure** | The test passes and the estate fails in production. | Instances were killed instead of the API being denied. The test proved capacity and was reported as proving resilience. | For any test claimed as evidence of static stability, the record must name the denied API. If no API was denied, relabel the test as a capacity test. |
| 5 | **Bought at a cost the business never approved** | The bill arrives at the next budget cycle; the estate is quietly returned to dynamic, undocumented. | The premium was approved verbally or by implication, not in a budget line with an owner. | Put the premium in the budget as its own line with a named owner and a stated failure it buys. Record the accepted risk if it is declined. An unapproved premium is a scheduled rollback. |

The five share a structure. Each one is a **claim that outran its evidence** - a design that says static while a reaction path survives, a test that says resilience while proving capacity, a cost that says approved while sitting outside the budget. Static stability has no runtime signature, so the only defence against all five is the same: make the absence explicit and check it mechanically rather than by reading the diagram.

Failure mode 3 deserves one more sentence, because the cross-zone load balancing variant is the most common and the least visible. An estate with three zones of correctly sized instances and a load balancer that only distributes within the zone it receives traffic in is, functionally, three independent estates - and the loss of one takes its traffic with it rather than redistributing it. The capacity was right, the routing was not, and nothing in the capacity plan would have caught it.

---

## 11. The Cymbal Bank Worked Example

**Everything in this section is illustrative.** Cymbal Bank is a fictional institution used across this repository's guides as a persona. The figures below are invented to make the arithmetic visible. They are not benchmarks, they are not measured from any real estate, and they must not be cited as data. The only real content in the section is the decision structure, which is representative of the decision every estate faces.

### The estate

Cymbal Bank runs a card authorization path - the service that decides, in the request path, whether a transaction is approved. It is an important business service with a tight latency budget and a strict availability expectation. The estate is three Availability Zones in one Region: an autoscaling fleet of authorization services behind a load balancer, a shared cache tier, and a database with a cross-zone read replica.

For the purposes of the arithmetic, say the authorization path runs 30 application instances, spread ten per zone, each sized to handle 120 requests per second comfortably. Steady-state peak is 1,800 requests per second, so the fleet is running at 50% utilisation at its busiest hour - a headroom position that the operations team considers conservative.

### The design that was there

The fleet was multi-AZ and autoscaled. Minimum ten instances per zone, maximum sixty across the estate, a target-tracking policy on CPU, and a zone-loss alarm that triggered a scaling action. In an architecture review this reads as a well-built, three-zone, horizontally scalable service. It had passed a multi-AZ test: instances killed in one zone, replacements launched in the other two, service restored.

That test was Test 1 only, and the replacements launched because the control plane was healthy. The design was dynamic and nobody had said so.

### The failure it would have met

A zone withdrawal at peak. Traffic shifts - data plane, works. Zones 2 and 3 are now carrying 1,800 requests per second between them, at 900 each against a combined comfortable capacity of 2,400. So far so good, and this is why the design felt safe: the *first* minute works.

Then the scaling policy fires and calls the control plane for twenty additional instances. Three outcomes, all plausible, none of which the design distinguishes:

- The call succeeds, instances launch over several minutes, warm up, register, and the estate returns to its 50% utilisation position. The recovery took eight to twelve minutes, during which the two surviving zones ran at 75% with cold-start gaps. Acceptable for most services, marginal for an authorization path with a latency budget.
- The call is throttled because every other Cymbal workload in the Region is doing the same thing. Retries extend the window; the estate stays at 75% utilisation and the latency budget starts to bite.
- The control plane is impaired by the same event. No capacity is added. The estate runs indefinitely at 75% of its comfortable capacity, with no bound on how long, and the team's mental model - "it autoscales" - is now actively misleading.

### The decision

The team faced the choice Section 4 lays out, and it is worth recording how the arithmetic actually looked, because the arithmetic is what decided it.

**Dynamic (status quo).** Thirty instances. Cheap. Recovery depends on a control-plane call that may or may not work, and on capacity being available in the surviving zones at the moment everyone else wants it.

**Static.** Each zone must be able to carry the load the estate sees with one zone gone. Peak is 1,800 requests per second; with one of three zones out, the surviving pair must cover all of it, so each surviving zone needs 900 requests per second of capacity. At 120 requests per second per instance, that is eight instances per zone.

The arithmetic here is instructive rather than embarrassing: **the estate already ran ten instances per zone.** Ten instances provide 1,200 requests per second of capacity, against a requirement of 900. The fleet was already N-1 for a three-zone design, because running at 50% utilisation at peak means the estate carries 2x headroom, and N-1 for three zones needs only 1.5x.

So the honest finding was that the capacity premium for static stability was close to zero, and the real gap was **not capacity at all**. The real gap was that the scaling policy existed, the zone-loss alarm existed, and the runbook had a decision in it. The estate was one configuration change away from being static and nobody had noticed, because the conversation had been about buying more instances - which is the wrong conversation, and the one the term tends to provoke.

What the team did: pinned each zone to a floor of ten instances (above the eight required), removed the zone-loss scaling action entirely, kept CPU-based scaling for *gradual growth* only, rewrote the runbook to remove the branch, and re-ran the test with `ec2:RunInstances` denied by an SCP in a dedicated OU.

### The cost that decision carried

Two costs, both real, both approved in a budget line with an owner.

**The idle capacity.** Ten instances per zone at rest instead of a fleet that could shrink to twenty in quiet hours. That is roughly a fifth of the fleet idle during off-peak, continuously, forever. It was approved because the alternative - an authorization path that depends on an API during a regional event - was not acceptable for an important business service, and because the number was small enough to state in a budget meeting without an argument.

**The rigidity.** Pinning each zone to ten instances removed the estate's ability to follow demand across zones. When a later marketing change shifted traffic toward one zone, the floor no longer matched the distribution, and the zone weights had to be revisited - which is exactly the maintenance burden Section 7 describes. The team added a per-zone request distribution metric to the monthly capacity review so the drift would surface before it mattered.

### The decision static stability changed

The decision that changed was not "buy more instances". It was **"stop treating autoscaling as the recovery mechanism for zone loss"**. Once that was said out loud, the capacity conversation became small, the runbook lost its branch, and the test became able to fail.

That is the thesis, in the form a real estate meets it. The system did not get better at fighting the failure. It was configured so that the failure required no fight - the capacity was already there, the routing was already correct, and the only thing that changed at 02:14 was which zones the load balancer was willing to send traffic to.
---

## 12. The Anti-Patterns of Claiming Static Stability

Section 10 was about building the property and getting it wrong. This section is about a different failure: **asserting the property without having it.** These are claims, and each one is checkable - which is the point of listing them, because a claim that can be checked by asking one question is a claim that should be checked.

**Calling an active-passive DR pair "static" because the standby exists.** The standby existing is a capacity fact. The failover is a decision and a control-plane call, and the standby is almost certainly cold. The question that settles it: what happens on the serving path when the primary fails? If the answer includes the word "promote", the design is not static. Active-passive can be made statically stable, but only when promotion is data-plane and pre-rehearsed - which is a much narrower claim than "we have a standby".

**Calling over-provisioning "static" when the runbook still scales.** Capacity was raised and the reaction was left in place, so the estate is over-provisioned *and* dynamic, which is the worst of both: you are paying the premium and keeping the dependency. The question: does any scaling policy or runbook step fire because a zone was lost? If yes, it is not static, and the extra capacity is buying you time rather than removing the step.

**Quoting the term in an architecture deck without the capacity proof.** The term is becoming a badge. A deck that says "statically stable" without the N-1 arithmetic per zone, the per-zone distribution under normal conditions, and the absence of control-plane calls on the serving path is making a claim with no evidence behind it. The question: where is the arithmetic? If the answer is "it is in the design", there is no arithmetic.

**Claiming it for compute only while the data tier promotes a replica.** This is Source C's fourth stated anti-pattern, in its own words: "Considering static stable designs for compute resources only." It is the most common instance of the whole family, because compute is where autoscaling is obvious and data is where promotion is hidden. A statically stable fleet in front of a database that promotes a cross-zone replica via a control-plane call is a static design with a dynamic dependency, and the dependency is on the critical path. The question: what happens in the data tier when the zone goes? "The replica is promoted" is the answer that fails.

**Claiming it because the last incident went well.** A single event in which the estate survived is one sample. It may have survived because the control plane was healthy, because the failure was partial, because it happened at low load, or because a human improvised well. None of those are the property. The question: was the API denied during the event, or was it merely not needed?

**Treating "we have multi-AZ" as a synonym.** Multi-AZ is a topology. Static stability is a behavioural property. Every design in Section 4 was multi-AZ, and one of them was dynamic. The question: with one zone gone and every control plane denied, does the estate serve full load? Multi-AZ is necessary for the answer to be yes and is nowhere near sufficient.

There is a pattern across all six: each claim substitutes a **topological or inventory fact** for a **behavioural one**. The estate has three zones, it has a standby, it has spare capacity, it survived last time. None of those are statements about what happens during the failure. Static stability is a claim about behaviour under denial, and only a denial test can support it.

---

## 13. The State of Practice: Documented, Conventional, Undocumented

Three buckets, no statistics. The purpose of separating them is that mixing them is how a convention acquires the authority of a specification.

### Documented by the provider

Four documents carry the concept, and each establishes something specific.

**The Builders' Library article (December 5, 2019)** establishes the mechanics and the contrast. It is the earliest dated source retrieved, the most detailed, and the source of the canonical anti-pattern, the positive form, the control/data plane definitions, the EC2 task lists, the availability asymmetry claim for EC2, the AZ definition, and the two architectural patterns with their cost. If one source had to be kept, this is it.

**The Fault Isolation Boundaries whitepaper (November 16, 2022)** establishes the canonical definition, the claim that control planes are "statistically more likely to fail", the statement that data planes are "statically-stable to an impairment in the control plane", the claim that "the most reliable recovery and mitigation mechanisms are the ones that require the fewest changes to achieve recovery", and the naming of isolation boundaries as AZ, Region, control plane and data plane.

**REL11-BP04 (pillar published November 6, 2024)** establishes the normative rule - rely on the data plane during recovery - the Route 53 documented asymmetry, the conversion table, and the three anti-patterns about DNS changes, control-plane scaling, and multi-service control-plane remediation.

**REL11-BP05 (same pillar)** establishes the single-mode framing, bimodal behaviour as the observable symptom, the four anti-patterns, the risk level (Medium), the cost trade in AWS's own words, the zone-count economic observation, and the concept of constant work.

That is the documented set. Notice what it is not: it is not a specification of how to build a statically stable system. It is a definition, a rationale, a set of anti-patterns, and a table of substitutions. The engineering - the sizing, the maintenance, the testing - is left to the reader.

### Practitioner convention

The ideas that do the work in real estates, and that predate or run parallel to the AWS vocabulary:

- **N+1 and N-1 provisioning** - capacity planning convention, decades old, from data-centre design.
- **Active-active over active-passive** - a general distributed-systems preference, motivated by latency and by the unreliability of promotion.
- **Bulkhead isolation** - from ship design, via the resilience literature; the direct ancestor of cell-based design.
- **Cell-based architecture** - the modern form, with bounded blast radius and shard-key routing.
- **"No automation on the failure path"** - operational folklore that turns out to be exactly Source D's advice.
- **Pre-warmed pools** - the mechanism by which serverless and container platforms avoid cold starts, and the same move as pre-provisioning one layer down.

These correspond to the same engineering idea under other names, and other platforms and vendors implement equivalents without using AWS's term. **That correspondence should not be overstated.** A Kubernetes cluster with over-provisioned nodes and pod-level scaling is doing the same thing as an N-1 EC2 fleet, and describing it as "statically stable" imports a term from a document that is specifically about AWS Availability Zones, Auto Scaling, regional control planes and Route 53. The mechanics are AWS-shaped; the idea is not. Sections 6 and 10 use the Kubernetes row of Source D's table for exactly this reason - it is AWS's own bridge from its vocabulary to a portable one.

### Not publicly documented

Collected here so the three buckets stay distinct, and expanded in Section 15:

- Any statement that control planes are *usually* impaired during a data-plane failure.
- Any quantified probability of control-plane impairment, conditional on any failure class.
- A per-service tabulation of which AWS control planes are regional, global, or partly zonal. Route 53 is documented; the general pattern is not.
- Any recovery-time objective for control planes.
- A formal definition of "cell" as an AWS isolation boundary.

The line between bucket one and bucket three is where this guide's honesty lives. A claim that crosses it without being marked is the failure mode of every guide written about a young term, and Section 14 is where the crossings are counted.

---

## 14. The Claims Audit

Every substantive claim in the guide, its source, its date, its kind, and its status. **Status legend:** ✅ established from a primary source read in this pass; ❌ rejected - a circulated claim the sources do not support; ⚠ flagged - plausible or partly supported but not verified. These are the repository's standard audit marks; all prose punctuation in this guide is ASCII, and source punctuation inside quotations has been normalised to ASCII (em dashes rendered as ` - `, curly quotes as straight quotes).

| Claim | Source | Date | Kind | Status |
|---|---|---|---|---|
| AWS definition of static stability: systems "operate in a static state and continue to operate as normal without the need to make changes during the failure or unavailability of dependencies" | Fault Isolation Boundaries, "Static stability" | 2022-11-16, retrieved 2026-10-24 | Primary, AWS whitepaper | ✅ |
| The Builders' Library article on static stability using AZs was published | Builders' Library, Mike Furr | 2019-12-05, retrieved 2026-10-24 | Primary, AWS article | ✅ |
| The whitepaper carries a printed publication date of November 16, 2022 | Fault Isolation Boundaries, front page | 2022-11-16, retrieved 2026-10-24 | Primary, AWS whitepaper | ✅ |
| Control planes are "statistically more likely to fail than data planes" | Fault Isolation Boundaries, "Static stability" | 2022-11-16, retrieved 2026-10-24 | Primary, AWS whitepaper | ✅ |
| The EC2 data plane "design targets a higher availability than that of the Amazon EC2 control plane" | Builders' Library | 2019-12-05, retrieved 2026-10-24 | Primary, AWS article | ✅ |
| Data planes are simpler, so failures are "statistically less likely to occur in the data plane versus the control plane" | Fault Isolation Boundaries, "Control planes and data planes" | 2022-11-16, retrieved 2026-10-24 | Primary, AWS whitepaper | ✅ |
| Route 53 control planes sit in a single Region, are outside the SLA, and events exist where the data plane outlives the control plane | REL11-BP04 | 2024-11-06, retrieved 2026-10-24 | Primary, AWS best practice | ✅ |
| "For certain types of service degradations, control planes are affected" | REL11-BP04, risk statement | 2024-11-06, retrieved 2026-10-24 | Primary, AWS best practice | ✅ |
| Bimodal behaviour is the observable symptom; workloads should operate in one mode | REL11-BP05 | 2024-11-06, retrieved 2026-10-24 | Primary, AWS best practice | ✅ |
| The four anti-patterns listed in Section 2 and Section 12, including "Considering static stable designs for compute resources only" | REL11-BP05 | 2024-11-06, retrieved 2026-10-24 | Primary, AWS best practice | ✅ |
| Adding zones reduces the extra compute needed for static stability | REL11-BP05 | 2024-11-06, retrieved 2026-10-24 | Primary, AWS best practice | ✅ |
| The control-plane-to-data-plane conversion table, including the Kubernetes pods-not-nodes rule | REL11-BP04 | 2024-11-06, retrieved 2026-10-24 | Primary, AWS best practice | ✅ |
| Route 53 ARC routing controls are a data plane over five Regional endpoints, usable if three of five Regions are available | Architecture Blog, "Minimizing Dependencies in a Disaster Recovery Plan" | date not printed as retrieved; assets dated 2022-01-25; retrieved 2026-10-24 | Primary, AWS blog | ✅ |
| SCPs can simulate a control-plane outage, and must be attached to a dedicated OU rather than the organization root | Same | Same | Primary, AWS blog | ✅ |
| Static stability entered the Reliability Pillar's best-practice list on 5 May 2022 | Pillar revision history, revision note (unnamed) | 2022-05-05 note; pillar 2024-11-06 | Inference from a revision note | ⚠ |
| The control plane is frequently or usually impaired when a zone or Region fails | None retrieved | - | Practitioner inference, not documented | ❌ as stated; ⚠ only as a hedge |
| "Static stability" appears in 0 of 676 repository markdown files | This repository, measured in this pass | 2026-10-24 | Measurement | ✅ |
| Failover appears in 103 files; control plane or data plane in 90; cell-based, bulkhead or blast radius in 96 | This repository, measured in this pass | 2026-10-24 | Measurement | ✅ |
| Disaster recovery appears in 187 files and failure/runbook/playbook in 254 | Earlier pass, not reproduced | - | Unreproduced measurement | ❌ |
| AWS defines "cell" as a formal isolation boundary | Not found in retrieved sources | - | Absence of evidence | ⚠ |
| The Availability and Beyond whitepaper is the origin of the concept and predates the 2019 article | Referenced by AWS's blog only; not retrieved | Unknown | Circulated, unverified | ⚠ |

### Rejected

Two claims are rejected outright rather than flagged, and both because they are stated with more confidence than any retrieved source supports.

**"When an Availability Zone fails, the regional control plane is frequently impaired too."** This is the strong version of the thesis and it is not in any retrieved AWS document. What is documented is weaker and sufficient: control planes are statistically more likely to fail, data planes target higher availability, certain degradations do affect control planes, and Route 53's control plane is single-Region and outside its SLA. The guide's conclusion survives on the documented version alone, which is why the strong version is not needed and is marked ❌ as stated. It is defensible as a *hedge* (⚠) - a mechanism-based expectation, consistent with regional scoping and with throttling under correlated demand - and that is the strongest form used here.

**"Disaster recovery appears in 187 files; failure, runbook or playbook in 254."** Neither figure reproduced on re-measurement with the file-union method, which returned 49 and 527. The other six counts reproduced exactly. Two figures that do not reproduce are not evidence of anything except a method that changed between passes, and the measured numbers are used throughout in their place.

### Flagged

**"Static stability entered the Reliability Pillar on 5 May 2022."** The revision history contains a note dated 5 May 2022 reading "Added two new best practices to Reliability Pillar in sections Use Fault Isolation to Protect Your Workload and Design your Workload to Withstand Component Failures." REL11-BP04 and REL11-BP05 both sit in the "Design your workload to withstand component failures" section, and they are the only two best practices there matching that subject matter. But **the note does not name them**, so the exact entry date of REL11-BP05 is an inference from a section name and a count, not a documented fact.

**"The Availability and Beyond whitepaper is where the concept lives."** AWS's own blog refers to it as discussing static stability. Its text and publication date were not retrieved in this pass, so no wording from it is quoted and no precedence claim is made. See Section 15.

**"AWS has no formal definition of a cell."** This is an absence of evidence, not evidence of absence. The retrieved whitepaper names four isolation boundaries and does not include cells; no retrieved source defines the term. That is what is claimed, and nothing more.

### The one thing this audit cannot do

It cannot certify that no AWS document outside the four retrieved sources says something different. Four documents were read; the AWS corpus is vastly larger. The audit certifies what these four establish, and marks everything else accordingly.

---

## 15. What Could Not Be Verified, the Glossary and the Cross-References

### What could not be verified

**(a) The Availability and Beyond whitepaper's own publication date and text.** AWS's architecture blog refers to "The Availability and Beyond whitepaper" as the place where static stability is discussed, and that reference is the reason the whitepaper is widely cited as the origin of the concept. It was not retrieved in this pass. Consequently: no wording from it is quoted anywhere in this guide, and **no claim is made that it predates the December 2019 Builders' Library article**. That precedence claim is extremely common and is not supported by anything read here. A reader who needs it should retrieve the whitepaper directly and check both its date and its wording.

**(b) The exact date static stability entered the Well-Architected Reliability Pillar's best-practice list.** The 5 May 2022 revision note records two new best practices in a named section but does not name them. REL11-BP04 and REL11-BP05 are the two items that match, which makes the inference strong and still an inference. The pillar's own current publication date (November 6, 2024) is printed and is not in doubt.

**(c) Whether AWS defines "cell" as a formal term.** Not found. The Fault Isolation Boundaries whitepaper's named isolation boundaries are Availability Zone, Region, control plane and data plane. Cell-based architecture is treated in this guide as engineering convention, described in its own terms, and explicitly not attributed to AWS. A definitive answer would require reading the whitepaper's full text rather than the two pages retrieved.

**(d) Any AWS statement of the probability of control-plane impairment during a data-plane failure.** Not published. The word "statistically" in Source A is not accompanied by a statistic, and no retrieved document gives a conditional probability, a rate, or even a qualitative frequency beyond "more likely". This absence is load-bearing for Section 5's argument (a) - an undocumented recovery time is the reason a control-plane-dependent recovery path is unbounded - and it is also the reason the strong version of the thesis cannot be asserted.

**(e) A tool limitation, recorded as such.** `web_search` returned empty or unreliable results on this host during this pass. The primary URLs were therefore fetched directly, which worked. This is recorded as a **tool limitation and not as an absence of material** - the difference matters, because "search found nothing" would be a false claim about the world while "search did not work" is a true claim about the tooling.

**(f) AWS Fault Injection Service capabilities.** The service is referenced in Section 8 as an injection mechanism, with no capability claim beyond its existence and general purpose. It was not retrieved or verified in this pass. The documented technique it would implement - deny the API with an SCP in a dedicated OU - is verified via Source F and stands on its own.

### Glossary

**Static stability** - a property of a system whose behaviour under a named failure is identical to its behaviour without it, because the capacity and routing required were already in place, so the failure requires no decision and no control-plane call.

**Control plane** - the administrative machinery that creates, modifies, deletes and describes resources, and propagates those changes to where they take effect. Includes propagation, not just the API call.

**Data plane** - the daily business of provisioned resources: the hot path that serves traffic or moves bytes. Defined per-operation, not per-service.

**Availability Zone** - a logically isolated, physically separated section of a Region with independent power and infrastructure, connected to its siblings by fast private fibre, designed to operate independently.

**Region** - a geographic grouping of Availability Zones under one administrative and API boundary. Control planes are usually regionally scoped, which is why a regional event is also a control-plane event.

**Cell** - an independently failing partition of a workload, sized so that its failure consumes a bounded share of total capacity, with routing that keeps a request inside one cell. Engineering convention; not an AWS term.

**Active-active** - a topology in which every site serves traffic, so no promotion decision exists. Distinct from active-passive, which requires one.

**Pre-provisioned capacity** - capacity allocated and paid for before the failure, so that no launch call is required during it. The mechanism behind N-1.

**Blast radius** - the share of a system's function lost when a given component fails. Bounded by partitioning; unbounded by shared dependencies.

**Bimodal behaviour** - the property of a workload that behaves differently under normal and failure modes. The observable symptom of missing static stability.

**Constant work** - doing the same amount of work per unit time regardless of dependency health: fixed refresh cadence, cached value on failure, alarm rather than behaviour change.

**N-1** - sizing each failure domain so that the loss of one leaves the remainder able to carry full load. For three zones, each carries 1.5x its steady-state share; for six, 1.2x.

### Cross-references

- [resilience_engineering_guide.md](../management/resilience_engineering_guide.md) - Hollnagel's safety science, the four abilities, Safety-I and Safety-II, work-as-imagined versus work-as-done, ETTO, FRAM. The organisational frame this guide's engineering sits inside; it does not discuss control planes.
- [operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) - MAS, PRA/FCA, DORA, BCBS, impact tolerances, important business services, the BCP lifecycle. Section 9 is the only bridge, and it does not re-derive any of it.
- [ibm_mq_disaster_recovery_guide.md](ibm_mq_disaster_recovery_guide.md) - messaging-tier recovery mechanics, including the queue-manager and channel state that a statically stable messaging design has to pre-arrange.
- [cloud_providers_guide.md](cloud_providers_guide.md) - provider comparison. This guide deliberately contains no vendor ranking and no cross-provider feature claims.
- [chaos_engineering_guide.md](chaos_engineering_guide.md) - failure injection as a discipline, blast-radius framing, and the practice of designing experiments that can actually fail.

---

## 16. The Closing Summary

The argument, in order.

Static stability means the configuration does not change in response to the failure, so the failure requires no action, no decision, and no control-plane call. AWS's own definition is broader than the popular reading: it is about not needing to make changes, and it explicitly tolerates staleness, which makes the constant-work pattern as much a part of the concept as over-provisioned capacity.

The reason the concept is necessary is structural, not stylistic. Control planes do orchestration across many subsystems with databases and workflows; data planes move bytes along a narrow path. AWS documents that control planes are statistically more likely to fail, that data planes are engineered to a higher availability target, that certain degradations affect control planes, and that for Route 53 the control plane is single-Region and outside the SLA. The strong version of the claim - that control planes are frequently impaired whenever zones fail - is not documented and is marked as inference here, and the engineering conclusion does not need it.

The canonical contrast is mechanical. A dynamic design's recovery is a chain of seven links, six of which are control-plane calls, capacity assumptions, warmup times or health-check thresholds. A static design deletes six of those steps rather than accelerating them, and the two that remain were the reliable ones. The dynamic design is cheaper and genuinely adequate for instance failures, bad deploys, memory leaks and gradual growth - a list that shares one property: the control plane is healthy when the response runs.

The price is real and continuous: idle capacity, rigidity, the complexity AWS itself names in multiple NAT gateways and route tables, and an estate that nothing ever exercises. The legitimate cases for dynamic reaction are equally real, and the guide states them - batch, dev/test, uncorrelated failures, genuinely elastic demand, constrained budgets. Where the bill is not approved, the honest move is to record the accepted risk, not to claim the property.

Testing is where the claim becomes falsifiable, and the order matters: read the runbook and count the decisions, then remove a zone under load, then deny the API and watch. A test that kills instances and leaves the control plane healthy proves capacity and is routinely reported as proving resilience.

In regulated settings the pattern is worth having because it converts an impact tolerance from a claim about process into a claim about capacity, which is auditable. The awkwardness is that regulators expect a procedure and static stability's answer is that there is no procedure - so the evidence has to be constructed as an absence plus a capacity proof, and that is a different evidentiary shape.

The failure modes and the anti-patterns are all one failure: a claim that outran its evidence. Static stability has no runtime signature, so the only defence is to make the absence explicit and check it mechanically.

**a statically stable system does not fight the failure; it was already configured not to care.**
