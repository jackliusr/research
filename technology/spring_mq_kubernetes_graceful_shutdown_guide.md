# Graceful Shutdown for Spring, IBM MQ and Kubernetes: Three Clocks and One Message That Cannot Wait

**Jack Liu Shurui, Solution Architect**

> **Author:** Jack Liu Shurui - Solution Architect at Cymbal Bank, Singapore
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Series:** Integration / Middleware / Platform (technology/)
> **Audience:** Solution architects, Java integration engineers, platform and SRE engineers, MQ administrators who own a Spring application consuming from IBM MQ on Kubernetes
> **Last Updated:** November 2026

*A deep-dive on the least glamorous and most frequently botched operation in a messaging estate: the planned termination of a JVM that is consuming from IBM MQ inside a Kubernetes pod. It covers the boundary between planned termination and inherited failure, the three independent clocks that run during every rolling deployment (the pod's termination grace period, the Spring application context's per-phase lifecycle timeout, and the JMS session's in-flight unit of work), what the message caught between them can and cannot do, the endpoint race that protects HTTP traffic and does nothing at all for a queue consumer, the complete documented configuration surface with every default traced to a primary source, the BackoutCount problem that the IBM documentation quietly hands back to the application, a fictional Cymbal Bank worked example, an ordered runbook, a failure catalogue, the claims audit, an honest statement of what could not be verified, and a glossary.*

**Cross-references:** [ibm_mq_disaster_recovery_guide.md](ibm_mq_disaster_recovery_guide.md) (the **unplanned** recovery guide - HA versus DR, media-image recovery, multi-instance, RDQM, Native HA, site loss, and its section 6 on failover and client-side recovery: queue manager identity, automatic client reconnect, in-doubt units of work, the CCDT, backout and backout-requeue as safety valves, browse cursors. This guide assumes that material and does not re-derive it), [dockerfile_best_practices_guide.md](dockerfile_best_practices_guide.md) (section 9 owns PID 1 and signal delivery in a container **generically** - the `pid_namespaces(7)` rule, shell form versus exec form, `STOPSIGNAL`, the `docker stop` grace period, `HEALTHCHECK` versus probes; cited here in one sentence and not re-derived), [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) (the cross-platform reliability discipline - delivery semantics, idempotency, dedupe keys, the transactional outbox; this guide points there rather than repeating it), [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) (the general rolling-deployment and drain problem for request-serving systems), [openshift_workload_availability_guide.md](openshift_workload_availability_guide.md) (pod disruption budgets and workload availability on the OpenShift side of the same platform), [enterprise_middleware_integration_platform_guide.md](enterprise_middleware_integration_platform_guide.md) (the integration platform landscape and the IBM MQ object model), [../banking/payment_rails_guide.md](../banking/payment_rails_guide.md) (the rails whose message flow this shutdown discipline protects), [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) (the resilience framework - change-related outage as a category of impact).

**How to read this guide.** Section 1 is the framing - the one-paragraph answer, the three clocks, and the key-facts table. Section 2 is the boundary - what this guide owns, what the two adjacent guides own, and why the distinction between inheriting a failure and controlling a termination is the whole point. Sections 3, 4, and 5 take one clock each: the pod's grace period, the Spring context's lifecycle timeout, and the JMS session's in-flight unit of work, in that order, because that is the order in which they are decided. Section 6 is the message in the middle and the honest test of the thesis. Section 7 is the endpoint race. Section 8 is the configuration surface, every line traced. Section 9 is the BackoutCount problem. Section 10 is the fictional Cymbal Bank worked example. Section 11 is the runbook. Section 12 is the failure catalogue and the evidence you cannot collect. Section 13 is the claims audit, including what could not be verified as 13.7. Section 14 is the glossary. Section 15 is the idempotency argument, the evidence that proves no message was lost, and the banking audit question. Section 16 is cross-references and the closing summary. Cross-reference convention: sibling guides in `technology/` are plain filenames, guides in `banking/` are prefixed `../banking/`, repo-root files are prefixed `../`. **Integrity convention:** ✅ = verified this pass against a primary source (vendor documentation or specification text, with the version and date recorded in section 13); ⚠ = flagged: partially verified, inferred rather than documented, version-gated and not re-confirmed, or sourced from a search-result extract rather than a full page read; ❌ = rejected. **Version note:** every default value, property name, and behavioural statement in this guide carries the release it was read at, and every one of them was read on 12 November 2026. Nothing here was assembled from memory, and no property key appears that is not quoted from a documentation page listed in section 13.

---

**Table of Contents**

1. [The Overview: Three Clocks and One Message That Cannot Wait](#1-the-overview-three-clocks-and-one-message-that-cannot-wait)
2. [The Boundary: Planned Termination Is Not Inherited Failure](#2-the-boundary-planned-termination-is-not-inherited-failure)
3. [Clock One - The Pod's Grace Period](#3-clock-one---the-pods-grace-period)
4. [Clock Two - The Spring Context's Lifecycle Timeout](#4-clock-two---the-spring-contexts-lifecycle-timeout)
5. [Clock Three - The JMS Session's In-Flight Unit of Work](#5-clock-three---the-jms-sessions-in-flight-unit-of-work)
6. [The Message in the Middle](#6-the-message-in-the-middle)
7. [The Endpoint Race](#7-the-endpoint-race)
8. [The Configuration Surface, Line by Line](#8-the-configuration-surface-line-by-line)
9. [The BackoutCount Problem](#9-the-backoutcount-problem)
10. [The Worked Example: Cymbal Bank's Payment Instruction Consumer](#10-the-worked-example-cymbal-banks-payment-instruction-consumer)
11. [The Runbook: Ordering a Termination You Control](#11-the-runbook-ordering-a-termination-you-control)
12. [The Failure Catalogue and the Evidence You Cannot Collect](#12-the-failure-catalogue-and-the-evidence-you-cannot-collect)
13. [The Claims Audit](#13-the-claims-audit) (with 13.7, What Could Not Be Verified)
14. [Glossary](#14-glossary)
15. [Idempotency, the Audit Question, and the Evidence](#15-idempotency-the-audit-question-and-the-evidence)
16. [Cross-References and the Closing Summary](#16-cross-references-and-the-closing-summary)

---

## 1. The Overview: Three Clocks and One Message That Cannot Wait

### 1.1 The One-Paragraph Answer

**Graceful shutdown of a Spring application consuming from IBM MQ on Kubernetes is not a flag you set. It is an ordering problem, and the ordering is decided by three clocks that no single component owns.** When a pod is deleted, the Kubernetes control plane starts a countdown (`terminationGracePeriodSeconds`, documented default 30 seconds), the kubelet runs any `preStop` hook, and then the container runtime sends `SIGTERM` to process 1 in each container. At approximately the same moment the Spring application context begins closing, and Spring Boot's graceful shutdown stops the embedded web server in the earliest phase of stopping `SmartLifecycle` beans, under a per-phase timeout (`spring.lifecycle.timeout-per-shutdown-phase`, documented default `30s`). Somewhere inside that, a `DefaultMessageListenerContainer` has one or more invoker tasks sitting inside `MessageConsumer.receive()`, possibly inside an open unit of work on a transacted JMS session, with a message that has already been handed to the application. Those three clocks were designed by three different teams for three different purposes, none of them reads the others, and there is no negotiation between them. The pod's clock ends in `SIGKILL`, which cannot be caught, blocked, or deferred. The Spring clock ends in a log line and a context that closes whether or not the listener finished. The JMS session's clock ends when the connection closes, and what happens to the in-flight message at that instant depends on an acknowledgment mode and a transaction boundary that most teams have never looked at. **The only actor in the system that cannot ask for more time is the message that has already been read.**

### 1.2 The Three Clocks, Stated Plainly

| Clock | Who owns it | Where it is set | Documented default | How it ends |
|---|---|---|---|---|
| **The pod's grace period** | Kubernetes (the pod spec, enforced by the kubelet and the container runtime) | `terminationGracePeriodSeconds` in the pod spec | `30` seconds ✅ | `SIGKILL` to every process still running in the container. Not catchable, not deferrable. |
| **The Spring context's lifecycle timeout** | Spring Boot (the application context close path) | `spring.lifecycle.timeout-per-shutdown-phase` | `30s` ✅ | The phase's stop processing gives up and the context continues closing; the listener container is stopped either way. |
| **The JMS session's in-flight unit of work** | The application, the JMS provider (IBM MQ classes for JMS), and the transaction manager if there is one | The session's acknowledgment mode or transaction demarcation; the application's own processing time | No default exists; it is whatever your listener does | Commit, rollback, or an abrupt disconnect whose outcome the provider decides. |

The table is the whole guide in miniature. Two of the three clocks have documented defaults of 30 seconds, which produces the single most common false comfort in this problem space: the numbers match, so people assume the clocks are aligned. They are not. The pod's 30 seconds starts at deletion and includes the `preStop` hook. The Spring 30 seconds starts when the context begins closing, which is *after* the container has received `SIGTERM`, and it applies **per phase**, not to the whole shutdown. The third clock has no default at all and is not a timeout - it is a duration that your code determines and nobody has measured.

### 1.3 What This Guide Argues, and How It Tests the Argument

The thesis is stated once here and then tested section by section rather than asserted:

> **Graceful shutdown is not a flag; it is an ordering problem - and the only component in the system with no way to ask for more time is the message already read.**

The test is whether the evidence actually supports it, and in three places it complicates it. **The Jakarta Messaging specification does define an orderly session close** - section 6.2.15 says session close "should not return until its message processing has been shut down in an orderly fashion", meaning no listener is running and a pending receive has returned ✅ - so at the *session* layer the specification is more helpful than the thesis implies, and the gap is not in the specification but in what the Spring container's `stop()` is documented to do between the session and the connection (section 5.6). **A `preStop` hook can buy time** - it runs before `SIGTERM` and the kubelet will request "a small, one-off grace period extension of 2 seconds" if the hook overruns ✅ - which is a real lever, and section 3.4 explains how small it is and why it usually buys the wrong thing. **The acknowledgment mode decides whether the message is at risk at all** - in a transacted session a termination mid-unit-of-work produces a rollback rather than a loss, and section 5.5 keeps those two cases apart, because conflating "we might lose a message" with "we might process one twice" produces the wrong remediation. Where the guide cannot settle a question from documentation, it says so and stops, rather than describing what the code probably does.

### 1.4 Key Facts

| # | Fact | Verdict |
|---|---|---|
| 1 | `terminationGracePeriodSeconds` defaults to 30 seconds; the `preStop` hook runs inside that budget, and if it overruns the kubelet requests a one-off 2-second extension. | ✅ Kubernetes, *Pod Lifecycle*, page revised 28 Oct 2026 |
| 2 | When the grace period expires, the runtime sends `SIGKILL` to processes still running - which no application can handle. | ✅ Same |
| 3 | `preStop` does **not** stop a message listener. It delays `SIGTERM`, so a consumer keeps taking new messages throughout the hook. | ⚠ Inference from the documented ordering; see section 3.4 |
| 4 | Spring Boot's graceful shutdown "occurs as part of closing the application context and is performed in the earliest phase of stopping `SmartLifecycle` beans", with a per-phase timeout. | ✅ Spring Boot 4.1.x reference, *Graceful Shutdown* |
| 6 | Graceful shutdown **arrived** in Spring Boot 2.3, described in the release notes as opt-in via `server.shutdown=graceful`. The current property appendix lists the `server.shutdown` default as `graceful`. The release in which the default flipped is **not established** by the pages read. | ✅ arrival / ⚠ default flip |
| 7 | `DefaultMessageListenerContainer` is "a loop of `MessageConsumer.receive()` calls", and in the default `AUTO_ACKNOWLEDGE` mode "this container applies automatic message acknowledgment before listener execution, with no redelivery in case of an exception". | ✅ Spring Framework 7.0.10 API javadoc |
| 8 | `spring.jms.listener.receive-timeout` defaults to `1s`, and the documentation says a value of `0` "is generally discouraged since it prevents clean shutdown". | ✅ Spring Boot property appendix, 4.1.x line |
| 9 | IBM MQ's `BackoutThreshold` diversion is documented as a function of WebSphere Application Server and the Application Server Facilities. "For all other applications, apart from allowing its value to be queried, the queue manager takes no action based on the value of the attribute." | ✅ IBM Documentation, BackoutThreshold queue attribute, 9.2.x |
| 10 | Therefore a plain JMS `Session` plus `MessageConsumer` - which is what `DefaultMessageListenerContainer` uses - is not documented to get automatic parking on `BackoutThreshold`. | ⚠ Inference, argued in section 9 |
| 11 | `JMSXDeliveryCount` is "not guaranteed to be exactly correct", and the provider "is not expected to persist this value". IBM's `BackoutCount`, by contrast, "always survives restarts of the queue manager". | ✅ Jakarta Messaging 3.1 section 3.5.11; IBM Documentation 9.3.x |

### 1.5 What This Guide Is Not

It is not a guide to IBM MQ high availability, disaster recovery, failover, delivery guarantees, PID 1, signal disposition, or image construction - each of those is owned by another guide (section 16.1). It does not re-derive the general idempotency discipline, which [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) owns; what section 15 does is state what idempotency has to mean for the specific failure this guide describes, and what evidence would prove the failure did not occur. It is not a broker comparison, and it contains no ranking, no benchmark number, no measured latency, and no error-rate statistic, because none were produced by this pass and inventing them would be the exact failure mode the guide is about. The only numbers in it are documented defaults, quoted as defaults, or explicitly-marked illustrative arithmetic.

---

## 2. The Boundary: Planned Termination Is Not Inherited Failure

### 2.1 What the Two Adjacent Guides Own

Two sibling guides cover the ground on either side of this one, and the boundaries are sharp enough to state as a rule.

**[ibm_mq_disaster_recovery_guide.md](ibm_mq_disaster_recovery_guide.md) owns unplanned recovery.** Its section 6, "Failover and Client-Side Recovery: Identity, Reconnection, In-Doubt Work", covers queue manager identity across failover, automatic client reconnection (including the documented limitation that IBM MQ classes for Java do not support it), in-doubt units of work and channel resynchronisation, the CCDT and connection name lists, backout and backout-requeue queues as safety valves, and the invalidity of a browse cursor held across a failover. **None of that is re-derived here.** Where this guide needs in-doubt work or reconnection, it points there. Where this guide needs the backout-requeue mechanism as a safety valve, it points there too - and then adds something that guide does not cover, because the interesting question is what happens to `BackoutCount` when the *cause* is a deployment rather than a poison message (section 9).

**[dockerfile_best_practices_guide.md](dockerfile_best_practices_guide.md) owns PID 1 and signals generically.** Its section 9 establishes, from `pid_namespaces(7)`, that PID 1 in a namespace only receives signals for which it has installed a handler, that `SIGKILL` and `SIGSTOP` from an ancestor namespace are forcibly delivered regardless, and that shell-form `ENTRYPOINT` puts `/bin/sh -c` in the PID 1 seat so the signal never reaches the application. That rule is a precondition for everything in this guide and it is stated here once: **if your JVM is not PID 1, or if PID 1 has no `SIGTERM` handler, the Kubernetes termination sequence degrades to "wait the full grace period, then die hard", with no drain at all.** ✅ For the mechanism, read section 9 of that guide. It is not repeated here.

### 2.2 The Distinction That Matters

> **In an unplanned failover you inherit whatever the broker decides. In a planned termination you control the order - and therefore a message read but not committed is your mistake rather than an accident.**

That sentence is the reason this guide exists separately from the DR guide. The two situations look similar from a distance - a consumer stops mid-unit-of-work, a message comes back or does not - and they are governed by completely different things.

| Dimension | Unplanned failover (DR guide) | Planned termination (this guide) |
|---|---|---|
| **Who decides the instant** | The failure. A node, a queue manager, a network link, a site. | You. A `kubectl rollout`, a scale-down, a node drain, a Helm upgrade. |
| **How many consumers are affected at once** | However many were on the failed node - potentially all of them. | One pod at a time, in a rolling fashion, if the rollout strategy is sane. |
| **Is the outcome knowable in advance** | No. The queue manager recovers to its last committed state; in-doubt channels resolve later. | Yes, in principle: you can compute what happens from the acknowledgment mode, the transaction boundary, and the three clocks. |
| **Is the window repeatable** | No - which is why failover is hard to test honestly. | Yes - which is why there is no excuse for not measuring it. |
| **What the message can do about it** | Nothing, and neither can you; you inherit the broker's decision. | Nothing either - but *you* can change the order, and that is the whole lever. |
| **The typical defect** | An untested runbook and a stale CCDT. | A termination sequence nobody ever read end to end. |
| **Where the duplicate or loss originates** | At the break, between two systems that disagree about an outcome. | At the `SIGKILL`, between a listener that was still working and an acknowledgment that had already happened or had not yet happened. |

### 2.3 Why "Planned" Makes It Worse, Not Better

There is a counter-intuitive point here that is worth stating early, because it inverts the usual intuition about risk.

An unplanned failure is rare, alarming, and investigated. A planned termination happens on every deployment, every scale event, every node drain, every cluster upgrade, and every configuration change - and it is *supposed* to happen, so it produces no incident, no page, and no postmortem. If a termination sequence loses one message in a hundred deployments, the estate is losing messages continuously and the loss is invisible because each individual event looks like a successful rollout. **The planned path is the high-frequency path, and it is the one nobody measures**, because measuring it would require treating a successful deployment as a test case rather than as an absence of failure.

That is also why the ordering problem is worth this much attention. A failure you cannot control is a risk you accept and provision against. A termination you *can* control, that still drops messages, is a defect in your own change process - repeated indefinitely, at the cadence of your release train.

### 2.4 What Follows From the Boundary

Three consequences shape the rest of the guide. **The DR guide's answers do not transfer** - "turn on automatic client reconnect" is a reconnection answer and does nothing for a message acknowledged before the listener ran, and "configure a backout queue" is a poison-message answer and does nothing for a message that was never rolled back. **The Dockerfile guide's answers are necessary but not sufficient** - a JVM that receives `SIGTERM` promptly and then closes its JMS connection with work in flight has simply failed faster. And **the only thing this guide can actually change is order**: not the grace period alone, not the timeout alone, not the acknowledgment mode alone, but the sequence in which intake stops, work drains, and the connection closes. Sections 3 to 5 establish what each clock allows; section 11 assembles them into an order.

---
## 3. Clock One - The Pod's Grace Period

### 3.1 The Documented Sequence

The Kubernetes *Pod Lifecycle* page sets out the termination flow as a numbered sequence. The ordering is not incidental detail; it is the specification of the first clock, so it is worth reading in the vendor's own words rather than in summary.

The pod is updated in the API server and shows as `Terminating`. The kubelet begins local shutdown. Then, for each container:

> "(2.1) If one of the Pod's containers has defined a preStop hook and the terminationGracePeriodSeconds in the Pod spec is not set to 0, the kubelet runs that hook inside of the container. The default terminationGracePeriodSeconds setting is 30 seconds." ✅

> "(2.2) The kubelet triggers the container runtime to send a TERM signal to process 1 inside each container." ✅

And at the end of the budget:

> "(4.1) ... sends SIGKILL to any processes still running in any container in the Pod." ✅

Four properties of this clock follow directly, and each one is a design constraint rather than an implementation detail:

1. **It is a countdown, not a request.** Nothing in the flow asks the application how much time it needs. The number is in the pod spec, written by whoever wrote the manifest, usually months before anyone measured a unit of work.
2. **It starts at deletion, not at `SIGTERM`.** The `preStop` hook is inside the budget. Whatever the hook spends is time the application does not get.
3. **The signal goes to process 1**, which is why the adjacent Dockerfile guide's PID 1 material is a precondition here rather than a tangent. ✅
4. **The clock ends in an uncatchable signal.** `SIGKILL` cannot be handled, so there is no "last resort" code path in the JVM. There is no `finally` block, no shutdown hook, no transaction rollback performed by the application. Whatever is in flight at that instant is decided by the broker.

### 3.2 What the Default of 30 Seconds Implies

The default is 30 seconds ✅, and it is the same numeral as Spring Boot's default per-phase lifecycle timeout ✅. That coincidence is the single most misleading fact in this problem space, and it misleads in a specific direction: it suggests the two budgets are the same budget.

They are not, for two reasons that compound.

- **The pod's budget is consumed before Spring's budget starts.** Spring's graceful shutdown "occurs as part of closing the application context" ✅, and the context begins closing when the JVM receives `SIGTERM` - which is step 2.2, after `preStop` has finished. So if the hook runs for any length of time at all, the application's usable budget is already smaller than the pod's.
- **The pod's budget is a hard ceiling on everything, while Spring's is per phase.** The Spring timeout applies to "the shutdown of any phase (group of `SmartLifecycle` beans with the same phase value)" ✅ - each phase gets its own timeout. Two phases that each take close to the timeout can therefore exceed a pod budget that matches a single timeout.

The practical reading: **the pod's grace period is the outer bound, and every other clock must fit inside it with room to spare.** A design in which the pod budget equals the Spring phase timeout has no room to spare by construction.

### 3.3 The Two-Second Extension, and What It Is For

Kubernetes does provide one documented soft edge:

> "If the preStop hook is still running after the grace period expires, the kubelet requests a small, one-off grace period extension of 2 seconds." ✅

This is worth understanding precisely, because it is frequently cited as if it were headroom for the application. It is not. It is headroom for **the hook**, granted when the hook overruns, so that the hook can be observed to finish rather than being cut off mid-call. It is described as "small" and "one-off" by the documentation itself, and its documented purpose is the `preStop` hook - not the main container's application threads.

The honest statement is: **2 seconds is not a drain allowance. It is a rounding allowance.** Any design that depends on it is a design that has already lost.

### 3.4 preStop Buys the Wrong Time

Here is where the first genuine complication to the thesis appears, and it cuts against the comfortable version of the story.

A `preStop` hook *is* a lever on the pod's clock. It runs before `SIGTERM`, so it delays the moment the application is asked to stop. Many teams use it exactly as the platform documentation's own tutorial does: as a sleep, to give the control plane time to stop routing traffic to a pod that is about to die. That is a legitimate and well-documented pattern for a request-serving workload.

For a **message-consuming** workload it does something different, and worse. The hook runs *inside the container*, before `SIGTERM`. During the hook, the JVM has not been asked to stop. Its listener container has not been told to stop. Its consumers are still attached to the queue and still receiving messages. So a `preStop` sleep does not drain a queue consumer - **it extends the window in which the consumer keeps taking new work while the deployment is already committed to killing it.** The consumer is not stopping; it is accelerating.

That is an inference from the documented ordering rather than a documented statement, so it is marked ⚠, but the inference is short: the hook precedes `SIGTERM` (step 2.1 before 2.2), and nothing in the hook's definition stops a JMS consumer. A hook that calls out to a management endpoint to flip a flag *could* stop intake, but that is the application's own mechanism wrapped in a hook, not the hook doing anything.

The corollary matters for the runbook in section 11: **`preStop` is useful in this architecture only for the HTTP half of the pod, and for delaying `SIGTERM` long enough that endpoint removal has propagated.** It is not a drain, it cannot be a drain, and using it as one converts a short window of risk into a longer one.

### 3.5 Multi-Container Pods, Sidecars, and the Documentation's Own Overrun

The ordering guarantee that people assume exists does not exist for the general multi-container case:

> "Otherwise, the containers in the Pod receive the TERM signal at different times and in an arbitrary order." ✅

For sidecar containers specifically - the ones declared with `restartPolicy: Always` in `initContainers`, in the current API - the behaviour is better, and the kubelet delays their termination "until the last main container has fully terminated", in reverse definition order. ✅ That is the correct shape: the helper outlives the workload it helps. But the same documentation names the failure mode:

> "if the Pod has a preStop hook that exceeds the termination grace period, emergency termination may occur" ✅

which is the sidecar case arriving at the same wall as everything else. If a sidecar's `preStop` overruns the budget, the pod does not get its orderly reverse-order teardown; it gets an emergency. A service mesh proxy with a slow drain in its own hook can therefore take the whole pod down hard, including the JVM in the main container that was behaving perfectly.

The Kubernetes tutorial *Explore Termination Behavior for Pods And Their Endpoints* (last modified 18 November 2024) makes the relationship concrete: its pod sets `terminationGracePeriodSeconds: 120` and gives the `preStop` hook a `sleep` of 180 - a hook deliberately longer than the entire budget - and shows the terminating endpoint as `ready=false`, `serving=true`, `terminating=true` while the server keeps processing requests throughout. ✅ Two lessons follow. **A `preStop` hook cannot exceed the budget it is given:** the hook is not a way to extend the pod's life, it is a way to spend the pod's life differently, and 180-versus-120 is a demonstration of the overrun rather than a recommended configuration. And **`preStop` can only buy time it is actually given** - which is why the last thing this clock does is kill the work.

The design consequence: **in a multi-container pod, the application's drain budget is not the pod's grace period minus its own hook. It is the pod's grace period minus every hook in the pod that runs before or concurrently with it.** That is a number nobody computes, and it changes whenever a sidecar is added. **What this clock can do:** set an outer bound, delay `SIGTERM` with a hook, grant a one-off two-second extension to an overrunning hook, deliver the signal to process 1, and give the control plane time to remove the pod from endpoints. **What it cannot do:** ask the application how much time it needs, stop a JMS consumer from receiving during a hook, grant the application any extension at all, or remove work that is already in the queue or already in the consumer's hands - which is what section 7 is about.

---

## 4. Clock Two - The Spring Context's Lifecycle Timeout

### 4.1 What the Current Documentation Says

Spring Boot's *Graceful Shutdown* reference page (4.1.x line, read 12 November 2026) is precise about both the mechanism and the scope:

> "Graceful shutdown is enabled by default with all three embedded web servers (Jetty, Reactor Netty, and Tomcat) and with both reactive and servlet-based web applications. It occurs as part of closing the application context and is performed in the earliest phase of stopping SmartLifecycle beans. This stop processing uses a timeout which provides a grace period during which existing requests will be allowed to complete but no new requests will be permitted." ✅

Three things in that paragraph deserve to be lifted out, because each one is a boundary of the second clock:

- **"part of closing the application context"** - the clock is the context's, not the pod's. It starts when the JVM's shutdown hook runs the context close, which is after `SIGTERM`, which is after `preStop`.
- **"the earliest phase of stopping SmartLifecycle beans"** - the web server is stopped early relative to other lifecycle beans. That is deliberate: stop accepting work first, then tear down what serves it.
- **"during which existing requests will be allowed to complete but no new requests will be permitted"** - the unit of work this clock was designed around is an **HTTP request**. That is the sentence that tells you what the feature is for, and section 4.6 is about what it therefore does not cover.

The page also documents the two properties that configure it: `spring.lifecycle.timeout-per-shutdown-phase` for the timeout, and `server.shutdown` for the mode - "To disable graceful shutdown, configure the `server.shutdown` property", with `server.shutdown=immediate` as the disabling value. ✅

### 4.2 The Same Page, One Minor Version Back, Says Something Different About Undertow

The 3.5.x version of the same reference page describes four embedded web servers rather than three, and distinguishes Undertow's behaviour: "all four embedded web servers (Jetty, Reactor Netty, Tomcat, and Undertow)" ✅, and "Jetty, Reactor Netty, and Tomcat will stop accepting new requests at the network layer. Undertow will accept new connections but respond immediately with a service unavailable (503) response." ✅ It changes what "no new requests" means at the socket level - three servers close the door, one keeps it open and answers with a refusal - which matters for a client with retry logic and for a load balancer's health accounting.

**What this guide does not do is speculate about why Undertow left the list**, because neither page says. Whether it reflects a support-matrix change or a documentation scope decision is **not established by the pages read**, and it is recorded as ⚠ in section 13 rather than guessed at.

### 4.3 The Feature Arrived in 2.3, Described as Opt-In

The Spring Boot 2.3 Release Notes (wiki page, edited 14 November 2024) record the feature's arrival and its then-default:

> "Graceful shutdown is supported with all four embedded web servers (Jetty, Reactor Netty, Tomcat, and Undertow) and with both reactive and Servlet-based web applications. When enabled using `server.shutdown=graceful`, upon shutdown, the web server will no longer permit new requests and will wait for a grace period for active requests to complete. The grace period can be configured using `spring.lifecycle.timeout-per-shutdown-phase`." ✅

"When enabled using `server.shutdown=graceful`" is the language of an opt-in feature: in 2.3, graceful shutdown was something you turned on. The `LifecycleProperties` javadoc in the 3.5.16 API confirms the introduction release independently - `Since: 2.3.0`. ✅

### 4.4 The Default Flipped, and Nobody Can Say When

Here the evidence produces a genuine, unresolved tension, and it is worth stating exactly rather than smoothing over.

- The 2.3 release notes describe graceful shutdown as enabled by a property you set. ✅
- The 3.5 and 4.1 reference pages say it "is enabled by default with all three embedded web servers". ✅
- The current property appendix lists `server.shutdown` with a documented default of `graceful`. ✅

So the default is `graceful` now, and it was opt-in at introduction. **The release in which the default changed from `immediate` to `graceful` is not established by any page read in this pass.** It is plausible that the change is recorded in some intermediate release-notes page; that page was not retrieved, and inventing a version number for it would be exactly the kind of claim this guide is written against.

The operational consequence of the uncertainty is small but real, and it points in an awkward direction: **a team that reads only the property appendix will believe graceful shutdown is on, and a team that reads only the 2.3 notes will believe it is off, and both will be reading accurate documentation for a different release.** In an estate with applications pinned across several Spring Boot lines, the same configuration file can mean two different things. The safe discipline is to set `server.shutdown` explicitly rather than rely on whichever default the deployed line happens to carry - and to treat "we did not configure it" as an unverified assumption, not a default.

### 4.5 "Phase" Is the Unit, and the Unit Is Not the Application

`spring.lifecycle.timeout-per-shutdown-phase` is documented as the "Timeout for the shutdown of any phase (group of `SmartLifecycle` beans with the same phase value)", default `30s`. ✅

That definition has a consequence that is easy to miss: **the timeout is not a budget for the application's shutdown. It is a budget for one phase's stop processing.** If several phases each take up to their timeout, total shutdown time is the sum, and the pod's grace period has to cover the sum, not one term of it.

Two further honesty notes on this clock, both of which constrain what can be claimed:

- **The phase values themselves are not something this guide will quote.** The documentation describes the mechanism in terms of phase values and names the web server's stop as happening in "the earliest phase", but the numeric phase of a given bean type is not a value read from the pages consulted in this pass. Do not take phase integers from this guide.
- **The listener container's participation in this ordering is documented only at the level of its own lifecycle methods.** Its javadoc documents `stop()` as "Stop this container", `doStop()` as "Notify all invoker tasks and stop the shared Connection, if any", `shutdown()` as "Stop the shared Connection, call `doShutdown()`, and close this container", and `doShutdown()` as "Close the registered invokers. ... A shared JMS Connection, if any, will automatically be closed afterwards." ✅ Those are the documented operations. **The relationship between those operations and the context's phase ordering is not spelled out in the pages read**, and section 5.6 treats that as the documentation gap it is.

### 4.6 The Scope Limit That Matters Most

Read the sentence again: "existing **requests** will be allowed to complete but no new **requests** will be permitted". ✅

The clock was designed for the request/response case. Its purpose is to let an HTTP handler finish writing its response before the server socket closes, so a rolling deployment does not produce a burst of connection resets. That is a real and valuable thing, and it is *not* the problem this guide is about.

A JMS listener has no request. It has a `receive()` call that blocks until a message arrives or the receive timeout elapses, and then a listener invocation that runs for however long the business logic takes. There is no socket facing a client, no response to finish, and no notion of "existing request" for the web server's graceful shutdown to track. **`server.shutdown=graceful` is therefore necessary for the HTTP half of a Spring application and does nothing whatsoever for the MQ-consuming half** - unless the listener container's own stop path happens to be orderly, which is precisely the question section 5 examines.

This is the point at which the three-clocks framing stops being a metaphor and becomes the actual architecture: the two clocks that have documented defaults were built for two different workloads, and the workload in the middle - a message already delivered, already being processed, with an acknowledgment pending - is not served by either.

It does not drain a JMS listener by itself, because its documented subject is requests - which section 5 tests.

---
## 5. Clock Three - The JMS Session's In-Flight Unit of Work

### 5.1 What the Container Actually Is

The third clock has no documentation page of its own, no default value, and no name in any manifest. It is the duration of one listener invocation plus the acknowledgment that follows it, and to reason about it at all you have to know what the container is doing while it runs. The class javadoc for `org.springframework.jms.listener.DefaultMessageListenerContainer` (Spring Framework 7.0.10 API, read 12 November 2026) is unusually direct about the mechanism:

> "Message listener container variant that uses plain JMS client APIs, specifically a loop of `MessageConsumer.receive()` calls that also allow for transactional receipt of messages (registering them with XA transactions)." ✅

> "On startup, it obtains a fixed number of JMS Sessions to invoke the listener, and optionally allows for dynamic adaptation at runtime (up to a maximum number)." ✅

> "Actual `MessageListener` execution happens in asynchronous work units which are created through Spring's `TaskExecutor` abstraction. By default, the specified number of invoker tasks will be created on startup, according to the `concurrentConsumers` setting." ✅

Three architectural facts follow, and every one of them matters at shutdown:

1. **The container is a loop of blocking `receive()` calls.** A JVM thread is parked inside `receive()` waiting for a message, or it is inside your `onMessage`. There is no event loop to quiesce, no queue of pending work to inspect, and no point at which the container "knows" how many messages are in flight except by asking its own threads.
2. **The sessions are obtained at startup and the invoker threads are created at startup.** Concurrency is a fixed pool of threads, each with its own session, each sitting in `receive()`. That is what "consumers" means here: threads, not subscriptions.
3. **"Dynamic adaptation at runtime" is a documented capability, not a default.** It happens "up to a maximum number", which is why the configuration surface exposes a minimum and a maximum rather than a single concurrency value (section 8.3).

The `receive()` loop is what makes the third clock invisible. There is no request to complete and no connection to drain from the outside; there is a set of threads whose state - blocked in `receive()`, or executing the listener, or committing - is the only thing that determines whether a message is at risk at the instant the connection closes.

### 5.2 The Acknowledgment Mode Decides Whether Anything Is at Risk

Before the clocks can be compared, the outcome has to be pinned down, and the outcome is set by one configuration value. The container's javadoc is explicit about the default and about its consequence:

> "It is strongly recommended to either set `sessionTransacted` to `true` or specify an external `transactionManager`." ✅

> "Note that for the default `AUTO_ACKNOWLEDGE` mode, this container applies automatic message acknowledgment **before** listener execution, with no redelivery in case of an exception." ✅

Read that second sentence slowly, because it is the single most consequential sentence in this guide. In the default mode, the acknowledgment happens **before** your code runs. The message is acknowledged to the broker, and only then is the listener invoked. If the listener throws, there is no redelivery. If the JVM is killed mid-listener, there is no redelivery either - the broker has already been told the message was consumed.

This is not a Spring quirk and it is not a defect; it is a documented design decision with a documented recommendation attached ("strongly recommended to ... set `sessionTransacted` to `true`"). But it does mean that **the most common configuration in the world - the default - has the message at risk for the entire duration of the listener invocation, with no recovery mechanism on the broker side.** The window is not a narrow race at the end of the work; it is the whole of the work.

### 5.3 The Specification Says Something Slightly Different, and the Difference Is Real

The Jakarta Messaging 3.1 specification (Final Release, 25 February 2022) defines `AUTO_ACKNOWLEDGE` in section 6.2.10:

> "`AUTO_ACKNOWLEDGE` - With this option, the session automatically acknowledges a client's receipt of a message when it has either successfully returned from a call to `receive` or the message listener it has called to process the message successfully returns." ✅

And section 6.2.11 explains what that means for redelivery:

> "When a client uses the `AUTO_ACKNOWLEDGE` mode, it is not in direct control of message acknowledgment. Since such clients cannot know for certain if a particular message has been acknowledged, they must be prepared for redelivery of the last consumed message. This can be caused by the client completing its work just prior to a failure that prevents the message acknowledgment from occurring. **Only a session's last consumed message is subject to this ambiguity.**" ✅

So the specification's `AUTO_ACKNOWLEDGE` acknowledges **after** the listener "successfully returns" - which means a kill mid-listener would leave the message unacknowledged and therefore redeliverable, and the specification's promise of ambiguity confined to "only a session's last consumed message" would hold.

The Spring javadoc says the container acknowledges **before** listener execution.

**These two statements describe different acknowledgement moments, and this guide will not pretend they are the same.** The most likely reconciliation is that they are describing different layers - the specification describes what a provider does when a session auto-acknowledges, while the container is described as applying acknowledgement itself, explicitly, ahead of the callback - but that reconciliation is an interpretation, and it was not confirmed from source in this pass. What can be said with confidence is the operational consequence: **if the container's javadoc is accurate for your version, the specification's "only your last consumed message is ambiguous" comfort does not apply, because the message was acknowledged before your code had a chance to fail.**

This is the first place where the evidence complicates the brief's framing. The thesis says the message cannot negotiate. That is true. But the reason it cannot negotiate is not simply that the clocks are misaligned - it is that **in the default configuration the message has already been spent before the work begins.** The clock that would have protected it was disabled by a default.

### 5.4 The Transacted Configuration, and What It Changes

Set `sessionTransacted` to `true` (or supply an external transaction manager, which is the other half of the javadoc's recommendation ✅) and the picture changes completely. From the specification, section 6.2.7:

> "A session may be optionally specified as _transacted_. Each transacted session supports a single series of transactions. Each transaction groups a set of produced messages and a set of consumed messages into an atomic unit of work. ... When a transaction commits, its atomic unit of input is acknowledged and its associated atomic unit of output is sent. If a transaction rollback is done, its produced messages are destroyed and its consumed messages are automatically recovered." ✅

And section 6.2.15, on closing:

> "Closing a transacted session must rollback its transaction in progress." ✅

That single sentence is the difference between loss and duplication. If the connection closes while a transacted unit of work is open, the work rolls back. The message returns to the queue. The broker's state and the application's state agree that the work did not happen.

But "returns to the queue" is not the same as "nothing happened", and the guide would be dishonest if it stopped there. Three consequences follow from a rollback, and all three are the operator's problem:

- **Any side effect already performed outside the transaction is still there.** If the listener wrote to a downstream system that is not enlisted in the same transaction, the rollback undoes the message consumption and not the write. That is a duplicate, and only idempotency resolves it - which is the territory of [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md), not this guide.
- **`BackoutCount` increments.** IBM MQ documents that the counter "is incremented each time that a message within a unit of work is backed out" ✅ - and section 9 is about what that does to a message that was perfectly valid and simply unlucky enough to be in flight during a deployment.
- **The redelivery is immediate and unbounded by the container.** The message goes back on the queue, and whichever consumer picks it up next - possibly in a pod that is about to be terminated too - processes it again from the beginning.

So: `AUTO_ACKNOWLEDGE` converts a shutdown into a possible **loss with no redelivery**. A transacted session converts it into a **guaranteed redelivery, with the duplicate risk that implies**. Neither converts it into "nothing happened". The only configuration that produces "nothing happened" is one where the drain completes before the connection closes, and that is an ordering problem, not a configuration value.

### 5.5 The Receive Timeout: One Second, and Why It Is Not a Detail

`spring.jms.listener.receive-timeout` is documented in the property appendix (4.1.x line, read 12 November 2026) as:

> "Timeout to use for receive calls. Use -1 for a no-wait receive or 0 for no timeout at all. The latter is only feasible if not running within a transaction manager and is generally discouraged since it prevents clean shutdown." ✅

Default: `1s`. ✅

This is the most under-appreciated property in the whole surface, and the documentation says why in six words: **"it prevents clean shutdown"**. A `receive()` with no timeout blocks indefinitely. A thread blocked indefinitely in `receive()` cannot observe a stop request, cannot return, and cannot be drained - so the container's stop must either interrupt it or wait for a message that may never arrive.

With the default `1s`, a blocked invoker returns from `receive()` with `null` at least once per second, which is what gives the container's stop path a chance to be observed. The timeout is therefore not a performance tuning knob. **It is the granularity at which the third clock can be interrupted**, and setting it to `0` for a marginal throughput gain removes the only regular opportunity the container has to notice that it is supposed to be stopping.

There is a related documented behaviour on the idle side. `setIdleReceivesPerTaskLimit`:

> "Marks the consumer as idle after the specified number of idle receives have been reached. An idle receive is counted from the moment a null message is returned by the receiver after the potential `setReceiveTimeout(long)` elapsed" ✅

with an effective default of 10 when the default task executor is in use, and a documented refinement: as of 6.2, "this idle limit does not apply to core consumers within `setConcurrentConsumers` but rather just to surplus consumers up until `setMaxConcurrentConsumers`". ✅ That matters at shutdown because it tells you which consumers are *surplus* - the ones that can be retired without reducing the container's core concurrency - and surplus consumers are the ones a scale-down should be allowed to release first.

`setMaxMessagesPerTask` completes the picture: "Default is unlimited (-1) in case of a standard `TaskExecutor`, reusing the original invoker threads until shutdown". ✅ Unlimited means the invoker task lives for the container's lifetime, so there is no natural per-batch boundary at which a thread is between messages. The thread is either in `receive()` or in your listener, and the transition is the only place a shutdown can be observed cleanly.

### 5.6 What "Stopped" Means, and Where the Documentation Stops

This is the crux of the whole guide, so it is worth being pedantic about what is documented and what is not.

The superclass javadoc, `org.springframework.jms.listener.AbstractJmsListeningContainer`, documents four operations:

| Method | Documented meaning | Verdict |
|---|---|---|
| `stop()` | "Stop this container." | ✅ |
| `doStop()` | "Notify all invoker tasks and stop the shared Connection, if any." | ✅ |
| `shutdown()` | "Stop the shared Connection, call `doShutdown()`, and close this container." | ✅ |
| `doShutdown()` | "Close the registered invokers. ... A shared JMS Connection, if any, will automatically be closed afterwards." | ✅ |

Now read what those four sentences actually promise. `doStop()` **notifies** invoker tasks and **stops the shared connection**. `doShutdown()` **closes** the invokers and then closes the connection. Neither sentence says "waits for an in-flight listener invocation to return before the connection closes". Neither says "blocks until the current unit of work completes". Neither describes a barrier, a latch, a count of in-flight messages, or a maximum drain duration.

**The javadoc documents that the container notifies its invoker tasks and stops the shared connection. It does not document a per-message drain barrier that waits for an in-flight listener invocation to finish before the shared connection closes.** That is the documentation gap, stated as a gap.

And this guide will not fill the gap by describing what the code probably does. Reading the implementation would settle it in an afternoon, and it is the correct next step for anyone making a production decision on the strength of this section - but this pass did not read the source, and a plausible reconstruction of the code's behaviour is exactly the kind of claim that produces confident, wrong runbooks. What can be said is what the documentation supports:

- The operations are named `stop` and `shutdown`, and both are described in terms of notifying tasks and closing resources. ✅
- The session-level specification *does* require orderly closure - "session close ... should not return until its message processing has been shut down in an orderly fashion. This means that none of its message listeners are running, and that if there is a pending receive, it has returned with either null or a message." ✅ - so an orderly close at the session layer is a specified behaviour, not an aspiration.
- Whether the container's stop path reaches that session-level orderly close for *every* invoker, under *every* acknowledgment mode, within the configured timeout, is **not documented in the pages read**.

That third bullet is the honest limit of this section, and it is what makes the ordering problem a real engineering problem rather than a reading exercise.

### 5.7 The Connection Cache Warning

One more documented behaviour belongs with the third clock, because it changes what "close the connection" means:

> "stopping and restarting a listener container will only work with an independent, locally cached Connection, not with an externally cached one" ✅

and, for the externally cached case, "dynamic scaling ... can result in JMS messages delivered to cached consumers even when they are no longer attached to the listener container". ✅

### 5.8 What the Third Clock Is

It is configured nowhere directly: the duration of your listener invocation plus the acknowledgment or commit that follows it, with no default, because it is a property of your code and your downstream dependencies. Nothing in the application bounds it; the first clock that expires kills it. What determines the outcome when it is cut short is the acknowledgment mode - acknowledged before execution means no redelivery, transacted means rollback - and whether there is a documented drain barrier at the container level is section 5.6's open question. **It is the one clock the operator can actually change, and the only one nobody writes down** - which is why deployments lose messages.

---

## 6. The Message in the Middle

### 6.1 What the Message Carries

**`JMSRedelivered`**, from specification section 3.4.7:

> "If a client receives a message with the `JMSRedelivered` indicator set, it is likely, but not guaranteed, that this message was delivered but not acknowledged in the past." ✅

It is set "whenever it is redelivering a message" ✅ - so it is a hint, not a ledger, and it carries no count.

**`JMSXDeliveryCount`**, section 3.5.11:

> "When a client receives a message the mandatory Jakarta Messaging-defined message property `JMSXDeliveryCount` will be set to the number of times the message has been delivered. The first time a message is received it will be set to 1, so a value of 2 or more means the message has been redelivered." ✅

> "The value of the `JMSXDeliveryCount` property is not guaranteed to be exactly correct. The Jakarta Messaging provider is not expected to persist this value to ensure that its value is not lost in the event of a failure." ✅

So the counter an application can read is **explicitly not durable**. It is provider-maintained in memory, a restart may reset it, and its documented purpose is "to allow consuming applications to identify whether a particular message is being repeatedly redelivered and take appropriate action" ✅ - a real purpose, but not a ledger.

**`BackoutCount`**, IBM MQ's own counter, is the durable one: "The backout count always survives restarts of the queue manager." ✅

The pairing is worth pausing on, because it shapes section 9. **The counter the specification gives the application is not durable, and the counter that is durable is not one the application is told to act on.** The application reads a number that may reset; the queue manager maintains a number that survives everything; and the documented mechanism that would connect the durable number to an action is scoped to a class of applications this one does not belong to.

### 6.2 What the Message Cannot Do

The message cannot do any of the following, and the list is short because the list is the point:

- **It cannot ask for more time.** There is no protocol element by which a consumer says "hold the deadline, I am nearly done". The transaction will commit or it will not.
- **It cannot retry itself.** Retry is a property of the container, the provider, or your code. A message is passive.
- **It cannot be protected after the fact.** Once acknowledged, it is gone. Once rolled back, it is a duplicate risk. There is no third outcome available at the moment of termination.

That last point is the operational heart of the thesis. Every option available to the operator is exercised **before** the message is read, or not at all.

### 6.3 The Complication: The Specification Does Specify an Orderly Close

Here the evidence pushes back on the thesis, and it is worth accepting the push rather than deflecting it. The claim "the message has no way to ask for more time" could be read as "the messaging layer has no concept of an orderly shutdown", and that reading would be wrong. Section 6.2.15 of the specification is explicit:

> "When session close is invoked it should not return until its message processing has been shut down in an orderly fashion. This means that none of its message listeners are running, and that if there is a pending receive, it has returned with either null or a message." ✅

That is a specification-level requirement that a session close **waits for the listener to finish** - precisely the drain barrier whose absence section 5.6 identified at the container level. The specification provides the guarantee; the question is whether the path from a Kubernetes `SIGTERM` to a session `close()` goes through it. The same section adds a detail that sharpens the problem rather than softening it:

> "If close succeeds and the acknowledge mode of the session is set to `AUTO_ACKNOWLEDGE`, the current message will still be acknowledged automatically when the `onMessage` call completes." ✅

So even in the middle of an orderly close, in the default acknowledgment mode, the in-flight message is **acknowledged on completion** - correct behaviour for a session close, and simultaneously the reason the message cannot be recovered if the process dies before the `onMessage` call completes. The orderly close protects the message during the close; it cannot protect it during the kill.

Revised thesis, therefore - and this is the version the guide will defend:

> **The message cannot ask for more time. The specification gives it an orderly close, and the default configuration spends it before the listener runs. What remains is order: whether intake stops before the connection closes, and whether anything waits for the work in between.**

That is a narrower claim than "graceful shutdown is not a flag", and it is a better one, because it identifies the exact mechanism: two of the three clocks are documented, one is measured by nobody, and the acknowledgment mode determines whether the third clock's expiry is a loss or a duplicate.

### 6.4 The Three Outcomes, Named

At the instant a termination lands on an in-flight listener, there are exactly three possible outcomes. Naming them makes the rest of the guide - the worked example, the runbook, the failure catalogue - mechanical rather than rhetorical.

| Outcome | Configuration that produces it | What the operator observes | Is it a defect? |
|---|---|---|---|
| **Silent loss** | `AUTO_ACKNOWLEDGE` (the documented default), killed mid-listener | Nothing. No error, no redelivery, no dead-letter message. The business process that expected the message never receives it. | Yes - and it is the worst kind, because it is unobservable |
| **Redelivery with duplicate risk** | `sessionTransacted=true` or an external transaction manager, rolled back | The message comes back, `JMSRedelivered` is set, `JMSXDeliveryCount` increments, `BackoutCount` increments | Not necessarily - it is the designed behaviour, and it becomes a defect only if the side effects are not idempotent |
| **Clean completion** | Work finishes, commits, and the connection closes after it | The message is gone from the queue because it was processed | No - this is the only outcome the operator should accept as the normal case |

The entire discipline in this guide is an attempt to move every termination from the first row to the third, and to make the second row survivable when the move fails. There is no fourth option, and no configuration makes the first row disappear except by moving into the second.

---
## 7. The Endpoint Race

### 7.1 The Race Is Documented, and It Is Not a Bug

Kubernetes is unusually candid about the fact that pod termination and endpoint removal are concurrent rather than ordered. From the *Pod Lifecycle* page, in the same numbered flow that section 3 quoted:

> "At the same time as the kubelet is starting graceful shutdown of the Pod, the control plane evaluates whether to remove that shutting-down Pod from EndpointSlice objects..." ✅

> "Any endpoints that represent the terminating Pods are not immediately removed from EndpointSlices, and a status indicating terminating state is exposed from the EndpointSlice API. Terminating endpoints always have their ready status as false (for backward compatibility with versions before 1.26), so load balancers will not use it for regular traffic." ✅

> "If traffic draining on terminating Pod is needed, the actual readiness can be checked as a condition `serving`." ✅

And the mechanism by which readiness removes a pod from service, from the probes documentation:

> "If the readiness probe returns a failed state, the EndpointSlice controller removes the Pod's IP address from the EndpointSlices of all Services that match the Pod." ✅

This is the race that `preStop` sleeps exist to paper over, and for an HTTP service it is a real problem with a real fix: the pod is dying, the endpoint still exists for a moment, requests arrive at a socket that is closing, and the client sees a connection reset. A `preStop` sleep of a few seconds covers the propagation window. That is the pattern, and it is sound **for its intended workload**.

### 7.2 Why None of This Helps a Queue Consumer

Here is the point that most Kubernetes guidance about graceful shutdown misses entirely, and it is the reason this guide exists as a separate document.

The endpoint race exists because a request-serving pod receives its work **through the endpoint**. Remove the endpoint, and new work stops arriving. The client that would have sent the next request looks up the service, does not find the pod, and picks another. The drain is real because the intake path is the same path the endpoint controls.

A JMS consumer receives its work **through a connection to the queue manager**. Its name is on no Service. No load balancer routes messages to it. No client resolves its IP. The messages arrive because a `MessageConsumer` has an open session on a queue and the queue manager delivers to it - and that relationship has nothing whatsoever to do with EndpointSlices.

Which produces the following table, and the contrast is the whole argument:

| Event | HTTP service | JMS consumer |
|---|---|---|
| Pod marked `Terminating` | New requests stop arriving once the endpoint is removed | **Nothing changes.** Messages keep being delivered. |
| Readiness probe fails | Endpoint removed from the Service; load balancer stops routing | **Nothing changes.** Messages keep being delivered. |
| `preStop` sleep runs | Covers endpoint propagation; the drain is genuine | **Messages keep being delivered, for longer.** |
| `SIGTERM` reaches the JVM | Embedded server stops accepting; in-flight requests finish | Container stop begins; whether the listener drains is section 5.6's open question |
| Grace period expires | `SIGKILL` | `SIGKILL` |

The second column is the documented, well-understood behaviour. The third column is the same mechanism applied to a workload it does not serve, and the result is not a weaker drain - it is **no drain at all**. The only event that reduces a queue consumer's intake is the consumer itself stopping, and the only thing that stops it is `SIGTERM`.

### 7.3 The Readiness Trap

There is a specific and common misconfiguration that follows from the confusion, and it is worth naming because it makes things worse rather than merely failing to help.

A team wanting "graceful shutdown for our MQ consumer" adds a readiness probe that fails when the application receives `SIGTERM`, believing this will "stop traffic". What actually happens is:

1. `SIGTERM` reaches the JVM. The probe starts failing. The endpoint is removed.
2. The container's stop path begins, and may or may not drain (section 5.6).
3. The pod is now **out of the Service and still consuming from the queue**, until the container's stop path reaches the connection, or until `SIGKILL`.
4. If the application's shutdown is slow for the HTTP side - a long request, a slow context close - the JMS side is still taking messages throughout, because readiness delayed nothing.

The probe did not stop intake. It changed which pods the *HTTP* traffic reaches, which may be a legitimate goal for the HTTP half of the same application, and it has no effect on the queue. **A readiness probe is a statement about whether a pod should receive routed traffic. It is not a statement about whether a pod should receive queue messages**, and there is no readiness mechanism that makes the second statement.

---

## 8. The Configuration Surface, Line by Line

### 8.1 The Rule This Section Follows

Every configuration key printed in this section was read from a documentation page during this pass, with the version and date recorded. **No key was assembled from memory, and no key appears here that is not in the list below.** Where a key that people commonly expect is absent from the documentation, the section says so explicitly rather than printing it and hoping.

Keys available to this section:

- Spring Boot application properties, from the property appendix (docs.spring.io, 4.1.x line, read 12 November 2026): `server.shutdown`, `spring.lifecycle.timeout-per-shutdown-phase`, `spring.jms.listener.receive-timeout`, `spring.jms.listener.session.transacted`, `spring.jms.listener.session.acknowledge-mode`, `spring.jms.listener.auto-startup`, `spring.jms.listener.min-concurrency`, `spring.jms.listener.max-concurrency`, `spring.jms.listener.max-messages-per-task`, `spring.jms.cache.enabled`, `spring.jms.cache.consumers`.
- Kubernetes pod spec fields, from the *Pod Lifecycle* page (revised 28 October 2026): `terminationGracePeriodSeconds`, `lifecycle.preStop`.

That is the entire surface this guide will print. Anything else is out of scope by construction.

### 8.2 The Table

| Key | Documented description (verbatim where in quotation marks) | Documented default | What it does to a clock |
|---|---|---|---|
| `server.shutdown` | "Type of shutdown that the server will support." | `graceful` ✅ | Clock two, HTTP half only. `immediate` disables the drain. |
| `spring.lifecycle.timeout-per-shutdown-phase` | "Timeout for the shutdown of any phase (group of `SmartLifecycle` beans with the same phase value)." | `30s` ✅ | Clock two, per phase. |
| `spring.jms.listener.receive-timeout` | "Timeout to use for receive calls. Use -1 for a no-wait receive or 0 for no timeout at all. The latter is only feasible if not running within a transaction manager and is generally discouraged since it prevents clean shutdown." | `1s` ✅ | Clock three's interruptibility. |
| `spring.jms.listener.session.transacted` | "Whether the listener container should use transacted JMS sessions. Defaults to false in the presence of a `JtaTransactionManager` and true otherwise." | No static default; **conditional** ✅ | Decides loss versus duplicate. |
| `spring.jms.listener.session.acknowledge-mode` | "Acknowledge mode of the listener container." | `auto` ✅ | Decides when the message is spent. |
| `spring.jms.listener.auto-startup` | Whether the listener container starts with the context. | `true` ✅ | Whether the consumer is attached at all. |
| `spring.jms.listener.min-concurrency` | The minimum number of concurrent consumers. | See 8.3 | Baseline threads, each in `receive()`. |
| `spring.jms.listener.max-concurrency` | The maximum number of concurrent consumers. | See 8.3 | The ceiling for runtime adaptation. |
| `spring.jms.listener.max-messages-per-task` | The maximum number of messages per task. | See 8.3 | Whether invoker threads are reused indefinitely. |
| `spring.jms.cache.enabled` | Whether to cache sessions/producers. | `true` ✅ | Whether a connection is locally cached. |
| `spring.jms.cache.consumers` | Whether to cache consumers. | `false` ✅ | Section 5.7's warning territory. |
| `terminationGracePeriodSeconds` | The pod's grace period; the `preStop` hook runs inside it. | `30` seconds ✅ | Clock one. |
| `lifecycle.preStop` | A hook run inside the container before `SIGTERM`. | None | Delays clock one's `SIGTERM`; does not drain. |

Two rows in that table are worth more than the other eleven, and they are the two that are almost never set deliberately.

**`spring.jms.listener.session.transacted`** is the conditional one. Its documented description says it "Defaults to false in the presence of a `JtaTransactionManager` and true otherwise" ✅ - which means the *effective* default depends on what else is on the classpath and configured in the context, not on a constant. An application with a JTA transaction manager gets `false`; an application without one gets `true`. Two applications with identical `spring.jms` configuration can therefore behave differently at shutdown because of a bean that a different team added. **This is the one property in the surface whose default cannot be read off the page, and it decides whether a termination loses a message or duplicates it.** Set it explicitly.

**`spring.jms.listener.receive-timeout`** is the one whose documented rationale is a shutdown rationale. Every other property in the table is about throughput or durability; this one carries the vendor's own warning that a particular value "prevents clean shutdown". ✅ A surface in which the shutdown-critical property is the one nobody tunes is a surface that misleads by omission.

### 8.3 Concurrency Is a Pair, Not a Scalar

There is a specific absence worth recording, because it is a common source of configuration that does not do what its author believes.

**The property appendix consulted for this guide contains no `spring.jms.listener.concurrency` key.** Concurrency is expressed through the `min-concurrency` / `max-concurrency` pair, which matches the container javadoc's own description of the design: it "obtains a fixed number of JMS Sessions to invoke the listener, and optionally allows for dynamic adaptation at runtime (up to a maximum number)". ✅ A minimum that is the steady-state consumer count, and a maximum that the container may grow to when there is work to do.

The documented semantics of the pair, and the reason they matter at shutdown, are visible in the idle-receive behaviour (section 5.5): the idle limit "does not apply to core consumers within `setConcurrentConsumers` but rather just to surplus consumers up until `setMaxConcurrentConsumers`". ✅ So there is a documented distinction between *core* consumers - which are always there and always in `receive()` - and *surplus* consumers, which can be retired when idle. **Which of the two a given thread is determines whether stopping it loses work or merely releases capacity**, and the container's own configuration is what decides.

**If a configuration file contains a key named `spring.jms.listener.concurrency`, this guide cannot tell you what it does**, because that key is not in the documentation read for this pass. It should be treated as unverified rather than as a shorthand.

Similarly, `max-messages-per-task` and the container javadoc's `setMaxMessagesPerTask` note - "Default is unlimited (-1) in case of a standard `TaskExecutor`, reusing the original invoker threads until shutdown" ✅ - describe a container in which invoker threads never rotate. There is no per-batch boundary at which a thread is reliably between messages, which is one more reason the cleanup point is the `receive()` timeout rather than task completion.

### 8.4 A Manifest and a Properties File, Traced

The following is not a recommendation for values. It is a demonstration that every line traces to a documented key, with the default left in place where the default is the right answer.

```yaml
apiVersion: apps/v1
kind: Deployment
spec:
  template:
    spec:
      terminationGracePeriodSeconds: 60
      containers:
        - name: payment-consumer
          lifecycle:
            preStop:
              exec:
                command: ["/bin/sh", "-c", "sleep 5"]
```

- `terminationGracePeriodSeconds` and `lifecycle.preStop` are the two pod-spec fields named on the *Pod Lifecycle* page. ✅
- The value `60` is a deployment decision, not a documented recommendation, and the number is not derived from anything in this guide. **It must be derived from the measured third clock** (section 11.2).
- The `preStop` `sleep 5` is illustrative and is deliberately *not* presented as a recommendation. Section 3.4 explains why a sleep buys the HTTP half of a pod and nothing at all for the consumer.

```properties
# The HTTP half. Explicit, because the default differs across Boot lines.
server.shutdown=graceful
# Clock two, per phase: sized so every phase fits inside the pod budget.
spring.lifecycle.timeout-per-shutdown-phase=25s
# Clock three's interruptibility: keep a finite receive timeout.
spring.jms.listener.receive-timeout=1s
# The default here is conditional; set it rather than inheriting it.
spring.jms.listener.session.transacted=true
# Consumer caching off: section 5.7's warning is about the other case.
spring.jms.cache.consumers=false
```

Every key in that block appears in section 8.2 with a documented description and default. The *values* on the right-hand side of the non-default lines are engineering decisions that this guide argues for but does not present as documented recommendations, and the `25s` is chosen only to illustrate the arithmetic of section 3.2 - not as a figure with any authority behind it.

---

## 9. The BackoutCount Problem

### 9.1 What IBM Documents

IBM MQ maintains a per-message counter that increments when a unit of work is rolled back. From *Committing and backing out units of work* (IBM MQ 9.3.x):

> "To help handle units of work, IBM MQ provides the `BackoutCount` attribute. This is incremented each time that a message within a unit of work is backed out. If the message repeatedly causes the unit of work to abnormally end, the value of the `BackoutCount` finally exceeds that of the `BackoutThreshold`. This value is set when the queue is defined." ✅

From *Messages that are backed out* (IBM MQ 9.3.x):

> "The backout count always survives restarts of the queue manager." ✅

The intent is clear and it is good intent: a message that keeps failing should eventually stop failing and start being visible. The mechanism is a threshold on the queue, and the action when the threshold is crossed is to move the message to a backout requeue queue.

### 9.2 The Sentence That Changes Everything

Here is the complication, quoted exactly, from the `BackoutThreshold` queue attribute documentation (IBM MQ 9.2.x):

> "Applications running inside WebSphere Application Server and those that use the IBM MQ Application Server Facilities use this attribute to determine if a message should be backed out. **For all other applications, apart from allowing its value to be queried, the queue manager takes no action based on the value of the attribute.**" ✅

That is the whole problem in one sentence. The queue manager maintains the counter. The queue manager does not act on the threshold. **The diversion is performed by a class of applications, not by the queue manager** - specifically applications inside WebSphere Application Server and those using the IBM MQ Application Server Facilities.

A second piece of evidence confirms the pattern from a different angle. `MQRC_BACKOUT_THRESHOLD_REACHED` (reason code 2362, `093A`), from the 10.0.x documentation:

> "On queue managers that do not support the `BackoutThreshold` and `BackoutRequeueQName` queue attributes, JMS ConnectionConsumer uses a value of 20 for the backout threshold. When the `BackoutCount` of a message reaches this threshold, the message is processed as specified by the disposition options." ✅

Note the subject of that sentence: **JMS `ConnectionConsumer`**. That is a specific Jakarta Messaging interface from the application-server facilities chapter of the specification, not a plain `Session` with a `MessageConsumer`. The diversion behaviour is documented for the application-server delivery model.

And the legacy manual, *Messages that are backed out* (WebSphere MQ for z/OS, (c) IBM 1993, 2002 - a legacy edition, not current-release documentation), names the failure mode that the threshold exists to prevent:

> "If a backout occurs, the messages which were retrieved from the queue are reinstated on the queue, and they can be processed again in another unit of work. If the processing of a particular message is causing the problem, the unit of work is backed out again. This could cause a processing loop." ✅

> "An application can detect messages that are caught up in such a loop by testing the `BackoutCount` field of `MQMD`." ✅

"An application can detect" - again, the application, not the queue manager.

### 9.3 What This Means for a Spring Listener

Now connect the two halves of the guide, and mark precisely where the documentation ends.

**Documented:** `DefaultMessageListenerContainer` is "a loop of `MessageConsumer.receive()` calls" using "plain JMS client APIs". ✅ It is not a JMS `ConnectionConsumer`. It is not running inside WebSphere Application Server. It does not use the IBM MQ Application Server Facilities.

**Documented:** For applications that are none of those things, "the queue manager takes no action based on the value of the attribute". ✅

**Inference, and marked as inference:** a `DefaultMessageListenerContainer` consumer that is repeatedly killed mid-unit-of-work - or repeatedly fails and rolls back - will drive `BackoutCount` up, and **nothing documented will park the message**. The counter climbs; the message stays on the queue; the next consumer picks it up; the cycle repeats. Whether anything eventually intervenes depends on whether the application, the container, or some other reader implements the check.

This guide does not assert that MQ automatically moves the message for a `DefaultMessageListenerContainer`. The evidence says the opposite: the automatic behaviour is documented as belonging to the application-server facilities, and the queue manager is documented as taking no action.

### 9.4 The Two Consequences the Operation Actually Faces

The inference above has two operational consequences, and they are different problems that arrive from the same mechanism.

**(i) A deploy storm inflates the counter on perfectly valid messages.** Every rolling deployment terminates pods. Every termination that lands mid-unit-of-work on a transacted session rolls back. Every rollback increments `BackoutCount`. The message did nothing wrong - it was simply in the wrong session at the wrong second, during a release that happens on a schedule. Run enough deployments and a valid message accumulates a backout history that looks, to any operator or tool that reads the counter, exactly like a poison message. Then, on the day someone does implement a threshold check, the deploy storm's victims are the first messages parked - and the parking queue fills with good work that needs manual release.

**(ii) Whether that becomes a parked message depends entirely on who is reading `BOTHRESH`.** If nothing reads it, the counter is a diagnostic curiosity that grows without effect. If the application reads it, parking happens on the application's terms and the application must own the release process. If some intermediary reads it - a monitoring tool, a bespoke housekeeping job, an inherited script that predates the container deployment - then the behaviour changes when that component's configuration changes, invisibly to the team that owns the consumer.

Neither consequence is a product defect. Both are the predictable result of a threshold whose documented enforcement lives somewhere other than where most people assume it lives.

### 9.5 The Counters Disagree, and That Is the Real Finding

Bring the evidence together and a genuinely awkward shape appears:

| Counter | Who maintains it | Durable? | Who acts on it |
|---|---|---|---|
| `JMSXDeliveryCount` | The JMS provider, in memory | **No** - "not expected to persist this value" ✅ | The application, per the specification's stated purpose ✅ |
| `BackoutCount` | IBM MQ, in the message | **Yes** - "always survives restarts of the queue manager" ✅ | WebSphere Application Server and Application Server Facilities applications; "the queue manager takes no action" ✅ |
| The application's own dedupe or attempt store | The application | Whatever the application built | The application |

The counter the specification tells the application to read is the one that may reset. The counter that survives everything is the one whose automatic action is scoped to a delivery model this application does not use. And the third row - the only durable, actionable record - is the one nobody builds, because the first two look like they already exist.

That is the sharpest form of the complication this guide has to report: **the message carries a durable record of how many times it has been mistreated, and the documented machinery for acting on that record does not reach the component that is mistreating it.** For a `DefaultMessageListenerContainer` on IBM MQ, the threshold check is application code, and if nobody wrote it, the threshold does not exist.

---

## 10. The Worked Example: Cymbal Bank's Payment Instruction Consumer

### 10.1 The System

Cymbal Bank runs a payment instruction service. The shape is common enough to be worth stating in full, because the failure depends on the shape:

- **Intake.** An upstream payment gateway puts instruction messages on an IBM MQ queue. A transmission queue and a channel carry them from the gateway's queue manager to the payment service's queue manager. The messages are persistent.
- **Processing.** A Spring Boot application consumes from the queue with a `DefaultMessageListenerContainer`. For each message it validates the instruction, writes a payment record to a database, and calls a downstream clearing adapter.
- **Scale.** Three replicas, in one Deployment, on a Kubernetes cluster. Concurrency is a fixed number of consumers per pod, expressed through the minimum and maximum concurrency pair (section 8.3), with the minimum equal to the maximum so there is no runtime adaptation.
- **The rest of the pod.** The same application serves an HTTP endpoint for operational queries, which is what made the team reach for `server.shutdown=graceful` in the first place.

The estate's configuration, before the review that this section describes, was the default in every dimension that mattered:

| Setting | Value before review | Where it came from |
|---|---|---|
| `terminationGracePeriodSeconds` | Not set, so `30` ✅ | The platform default, never considered |
| `preStop` | A `sleep` of a few seconds, added for the HTTP endpoint | Copied from a template for a web service |
| `server.shutdown` | Not set | Default, believed to be `immediate` by the team and `graceful` by the documentation (section 4.4) |
| `spring.lifecycle.timeout-per-shutdown-phase` | Not set, so `30s` ✅ | Default |
| `spring.jms.listener.receive-timeout` | Not set, so `1s` ✅ | Default |
| `spring.jms.listener.session.transacted` | Not set | Default, which is conditional (section 8.2) - nobody knew which side of the condition applied |
| `spring.jms.cache.consumers` | Not set, so `false` ✅ | Default |

Every value in that table is a default. That is the first finding, and it is the finding in most real estates: **the shutdown behaviour of a payment-path consumer was determined entirely by defaults chosen by four different projects for four different reasons.**

### 10.2 The Arithmetic That Fails

The team's belief was that the two 30-second defaults were "aligned". Set out the arithmetic with the documented values and the belief collapses immediately.

The pod's budget starts at deletion. The `preStop` hook runs inside it. `SIGTERM` arrives when the hook finishes. So the application's usable budget is:

```
application budget = terminationGracePeriodSeconds - preStop duration
```

With the documented default of 30 seconds and any non-zero hook, the application budget is **strictly less than 30 seconds**. ✅ for the 30; the subtraction is arithmetic.

The Spring timeout is 30 seconds **per phase**, starting when the context begins closing - which is after `SIGTERM`. ✅

So, with both defaults and any `preStop` at all, the following is true by arithmetic: **the pod's clock expires before the lifecycle timeout can.** The `SIGKILL` arrives while the context is still inside its phase stop processing, and the timeout that the team believed was protecting them never fires. The timeout is not a guard rail in that configuration; it is a number that is never reached.

The same is true with no `preStop` hook at all, for a smaller reason: the grace period starts at deletion and `SIGTERM` arrives after the API server update, the kubelet's local shutdown, and the runtime's signal delivery, so the two 30-second clocks start at slightly different instants and the pod's started first.

Two conclusions follow, and the second is the important one:

1. **Matching defaults are not aligned clocks.** They are two numbers that happen to be equal, started at different times, measuring different things, one of them per phase.
2. **The pod's clock is always the one that binds**, unless the operator deliberately makes the pod's budget larger than the sum of everything inside it. That is the only configuration in which the Spring timeout has any protective function at all.

### 10.3 What They Changed

The changes are listed in the order the team made them, because the order is the argument.

1. **They set the acknowledgment mode explicitly** - `spring.jms.listener.session.transacted=true` rather than an inherited conditional default. This did not make shutdown graceful; it changed the failure from silent loss to redelivery, which is a failure the operation can see and reconcile. That is a move from the first row of section 6.4's table to the second, not to the third.
2. **They measured the third clock** - timestamps at listener entry and exit, and a timeout on the downstream call, because an unbounded call makes the third clock unbounded and therefore unmatchable to any budget.
3. **They sized the pod's budget from the measurement.** The rule is section 11.4's inequality: the pod's budget must exceed the `preStop` duration plus the longest unit of work plus the commit, with margin. Because the work was now bounded, this was arithmetic rather than hope.
4. **They set the lifecycle timeout below the pod budget** so the Spring clock expires first and the context close reports a timeout - an observable event - instead of being cut off by a `SIGKILL` that reports nothing. A clock that fires and logs is worth more than one that never gets to fire.
5. **They removed the consumer's dependence on `preStop`.** The hook stayed for the HTTP endpoint; the runbook stopped treating it as a drain.
6. **They added an application-level check on the durable counter.** Since nothing documented parks a message on `BackoutThreshold` for this container type (section 9.3), the application reads the durable count on redelivery and routes a repeatedly-backed-out message to a queue a human watches. Application code the team wrote, because the alternative was a policy that existed only in a design document.

What did not change: the `SIGKILL` still exists and still cannot be handled, so the changes moved the probability of being killed mid-work and the consequence of it, not the hard edge itself. The duplicate risk is now real where the loss used to be, which means the downstream clearing adapter must tolerate being called twice for the same instruction - work owned by [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md), and the team's honest position was that the shutdown fix made an existing idempotency gap *visible* rather than creating it. And the counter inflation continued, because every deployment still rolls back units of work on messages that were perfectly valid (section 9.4), which forced the application check to distinguish rollback-because-terminated from rollback-because-bad - a distinction not present in the message, and one the team had to add.

The worked example's real lesson is not the list of settings. It is that **the review started with "what happens to the message at the instant of the kill" and worked backwards to the configuration** - the opposite direction from reading a properties file and asking what each line does. And it ended where the thesis says it must: not at a flag that was turned on, but at an order that was chosen, a duration that was measured, and an idempotency gap that the new order made survivable. The message that had no way to ask for more time was protected by the only means available - the work around it was arranged so that being cut short was survivable, and the message could no longer be spent before the listener ran.

---

## 11. The Runbook: Ordering a Termination You Control

### 11.1 Ordering the Decisions, Then Ordering the Shutdown

A runbook for this problem has two parts that are frequently confused: the **decisions** you make once, in design, and the **sequence** you follow at deploy time. The decisions come first because they determine whether the sequence has anything to work with. A perfect deploy-time sequence applied to an `AUTO_ACKNOWLEDGE` listener with a ten-minute unit of work and a 30-second grace period is a sequence of no-ops.

The decisions, in the order they must be taken:

1. What is the acknowledgment mode, explicitly?
2. What is the measured maximum unit of work, including the downstream calls?
3. Do the two clocks have room for that measurement, with the Spring clock expiring first?
4. What happens to the message when the answer to question 2 is exceeded despite everything?

Only then is there a sequence worth writing.

### 11.2 Decision One: Set the Acknowledgment Mode Explicitly

The reason is section 8.2: the default is conditional on the presence of a `JtaTransactionManager`, so the effective value is a property of the assembled context rather than of the configuration file. Two applications in the same estate, with the same `spring.jms` block, can land on different sides of the condition.

Choose between two honest positions:

- **Transacted, with idempotency downstream.** A termination mid-unit-of-work rolls back and the message returns. The cost is duplicates, which is a solvable problem with a dedupe key. This is the position most payment paths should take, because a duplicate can be reconciled and a loss cannot.
- **`AUTO_ACKNOWLEDGE`, with the loss accepted in writing.** The message is acknowledged before the listener runs ✅, so a kill mid-work is a silent loss. This is a legitimate position only where the message is genuinely disposable or where a reconciliation process will re-drive it from a source of truth. It must be a decision, not an inheritance.

There is no third position. The guide is not neutral between them, but it is neutral about which one you choose - what it is not neutral about is choosing by accident.

### 11.3 Decision Two: Measure the Third Clock

The third clock is the only one of the three whose value you control, and it is the only one that nobody measures. Measuring it requires two timestamps and a bound:

- **Timestamp at listener entry and at listener exit.** Without both, there is no distribution, only an anecdote.
- **A bound on every call the listener makes.** An untimed downstream call makes the third clock unbounded, and an unbounded clock cannot be compared to any budget. This is the single most common reason a "we set the grace period generously" claim turns out to be false: the generosity was sized against the fast path.

The number to carry forward is the maximum, not the mean. A budget sized to the mean is a budget that fails on the tail, and the tail is where a deployment does its damage.

### 11.4 Decision Three: The Inequality

The clocks relate by an inequality, and it is worth writing down because it makes the failure modes obvious:

```
preStop duration
  + maximum unit of work (including commit)
  + margin
  <= terminationGracePeriodSeconds

spring.lifecycle.timeout-per-shutdown-phase
  <  terminationGracePeriodSeconds - preStop duration
```

The first line says the pod's budget must contain the work. The second says the Spring clock must expire **before** the pod's, so that a shutdown that is going to fail fails *observably* - with a timeout logged by the context close - rather than being cut off by a `SIGKILL` that produces nothing.

That second line is counter-intuitive and it is the one teams get wrong. The instinct is to make the lifecycle timeout large, so the application "has time". But a lifecycle timeout larger than the pod budget can never fire: the pod's clock ends first, every time (section 10.2's arithmetic). **A timeout larger than the pod budget is not a generous timeout; it is a disabled one.** The correct configuration makes the Spring clock the inner, visible bound, and the pod's clock the outer, fatal one.

With the documented defaults the inequality fails at the second line for any non-zero `preStop` (section 10.2), which is the concrete reason the defaults are not a safe configuration for a message consumer even though they look like one.

### 11.5 The Deploy-Time Sequence, as a Timeline

Six steps, each with its position on the pod's clock, the clock it serves, and the race it loses if it is misread. This is the sequence for a single pod; the deployment's rolling strategy is what makes it one pod at a time, and the platform's own controls for that are outside this guide's scope. **The `T+` notation is relative ordering, not measurement** - no timing figure in this table is a measured value, and the only numbers available are the documented defaults of sections 3 and 4.

| # | Position on the pod's clock | Step | Clock served | The race at this step |
|---|---|---|---|---|
| 1 | `T+0` | Pod marked `Terminating`; the control plane begins endpoint removal | Clock one starts | The endpoint removal is concurrent, not ordered (section 7.1): routed traffic can still arrive at a pod that is already dying |
| 2 | `T+0` to `T+preStop` | The `preStop` hook runs inside the container | Clock one, spent | The consumer is still attached and still receiving queue messages for the whole hook - `preStop` drains HTTP, not a queue (section 3.4) |
| 3 | `T+preStop` | `SIGTERM` reaches process 1; the context begins closing | Clock two starts | Endpoint propagation may still be incomplete on some nodes, and the queue manager has no equivalent mechanism to propagate at all |
| 4 | `T+preStop` to `T+preStop+receive-timeout` | Listener container stops; blocked invokers return from `receive()` | Clock three's interruptibility | A thread parked in an indefinite `receive()` never observes the stop (section 5.5) |
| 5 | Inside the phase timeout | In-flight units of work complete and commit, or roll back if the budget runs out | Clock three | `SIGKILL` arriving mid-commit, and a commit whose outcome the application never learns |
| 6 | Before grace expiry | The connection closes; the pod exits | Clocks one and two, jointly | The grace period expiring before step 5 completes - the only race that is fatal by construction |

Step 4 is the one that depends on a documented-but-not-obvious property: the receive timeout. A finite `receive()` timeout is what gives a blocked invoker a regular opportunity to notice the stop (section 5.5), and the documentation's own warning about `0` - "it prevents clean shutdown" ✅ - is the same point from the other direction.

Step 5 is the one the operator cannot fully control, because whether the container waits for it before closing the connection is the documentation gap of section 5.6. The mitigation is not a better sequence; it is step 5's *consequence* being made survivable by step one of the decisions: a transacted session means an incomplete step 5 is a rollback rather than a loss. **What the runbook cannot do:** make the `SIGKILL` avoidable, make an unbounded unit of work bounded, or make a duplicate unnecessary - only reconcilable. And it cannot substitute for the measurement: if nobody knows how long the work takes, every number in the sequence is a guess with a decimal point.

---

## 12. The Failure Catalogue and the Evidence You Cannot Collect

### 12.1 Five Ways a Planned Termination Loses a Message

Each of these is a distinct mechanism with a distinct remedy, and conflating them is why "we fixed graceful shutdown" sometimes fixes nothing.

| Mechanism | Configuration that produces it | What it leaves behind | Remedy |
|---|---|---|---|
| **1. Acknowledged but unprocessed (silent loss)** | `AUTO_ACKNOWLEDGE`, the documented default, which applies "automatic message acknowledgment before listener execution, with no redelivery in case of an exception" ✅ | **Nothing.** No error, no redelivery, no dead-letter message | A transacted session, or an idempotent re-drive from a source of truth. Nothing at the pod level |
| **2. Rolled back and reprocessed (visible duplicate)** | `sessionTransacted=true` or an external transaction manager | `JMSRedelivered` set, `JMSXDeliveryCount` incremented within its documented limits ✅, `BackoutCount` incremented and durable ✅ | Idempotency and a dedupe key at every non-transactional boundary ([message_queue_data_loss_guide.md](message_queue_data_loss_guide.md)) |
| **3. preStop overrun (hard kill with a hook in flight)** | A hook longer than the grace period, or a sidecar whose hook overruns | A hook that never logged completion; a pod that terminated later than expected | Nothing in any hook may outlast the budget, and the budget must cover every hook in the pod (section 3.5) |
| **4. Sidecar ordering surprise** | A multi-container pod; shutdown order assumed rather than checked | Interleaved log timestamps from two containers, and nothing that says this was the problem | Know which kind of container you have; do not put a slow drain in a sidecar hook without budgeting for it |
| **5. The counter that climbs** | Any transacted consumer, deployed often | A counter that rises across deployments and looks to anyone reading it later exactly like a poison message | Distinguish rollback-due-to-deployment from rollback-due-to-bad-message at the point of rollback; own the threshold check in application code (section 9.4) |

Mechanisms 3 and 4 are Kubernetes-side and have cheap fixes. Mechanisms 2 and 5 are acknowledgment-side and are fixed by configuration plus application code. **Mechanism 1 is the one with no evidence trail at all**, which is why it is the only row that cannot be found after the fact - and why the whole guide is arranged to move work out of it.

### 12.2 Why the Evidence Is Missing

The reason these five are hard to find is structural: **a successful shutdown is not an event.** A `SIGKILL` logs nothing, and a successful drain logs nothing either, because nobody logs "I stopped correctly". The platform records a pod that terminated, and it records the same thing whether the pod drained perfectly or lost four messages on the way out. A failover produces alarms, a change of active node, channel events and often a postmortem; a rolling deployment produces a green pipeline. **The high-frequency path has the lowest observability**, and the loss it causes is distributed in units too small to notice individually. Hence the operational corollary of the thesis: an ordering problem that fails silently must be verified by construction, because it cannot be verified by observation - which is why section 15.4's instruments are not optional.

**What to record instead.** Record the configuration as deployed with its **effective** values, including the ones whose defaults are conditional - a deployment that cannot state whether `session.transacted` was true cannot say whether it lost or duplicated. Record the measured distribution of the third clock, with its maximum. Record the termination as an event: a log line at the start of the context close, one at the end of the drain, one when the container stops. And record the redelivery evidence on the other side - `JMSRedelivered`, `JMSXDeliveryCount` and the durable `BackoutCount` - joined to the deployment timeline: a message redelivered within a minute of a rollout is a message a termination touched, and that join is the closest thing to a shutdown audit trail the architecture offers. Section 15.4 turns this list into named instruments.

---
## 13. The Claims Audit

**Legend.** ✅ = verified this pass against a primary source (vendor documentation or specification text), with the version gate and the retrieval date recorded. ⚠ = flagged: partially verified, inferred rather than documented, version-gated and not re-confirmed, or read via a search-result extract rather than a fully retrieved page. ❌ = rejected. Retrieval date for every item below: **12 November 2026**.

**Source-quality note on IBM Documentation.** `www.ibm.com/docs/*` was unreachable to this run's fetcher on every attempt during this pass - a scraper error, not a 404. The IBM MQ claims in section 9 therefore rest on **search-result extracts** that reproduce the IBM topic text, plus one legacy manual read in full from `publibfp.boulder.ibm.com`. Each such row carries that provenance explicitly. This is a source-quality caveat, not a content caveat: the text quoted is the text the search index returned for the IBM topic, but a reader relying on any of these for a production decision should confirm it against `www.ibm.com/docs` at their own release level.

### 13.1 Kubernetes (kubernetes.io, *Pod Lifecycle*, page last modified 28 October 2026)

| # | Claim | Verdict | Version gate | Source | Date |
|---|---|---|---|---|---|
| (a) | The `preStop` hook runs inside the container before `SIGTERM`, and "The default `terminationGracePeriodSeconds` setting is 30 seconds." | ✅ | Documented as of the page revision dated 28 Oct 2026; API reference links on that page are to the v1.37 API | *Pod Lifecycle*, section "Pod Termination Flow", step 2.1 | 12 Nov 2026 |
| (a) | "The kubelet triggers the container runtime to send a TERM signal to process 1 inside each container." | ✅ | Same | *Pod Lifecycle*, step 2.2 | 12 Nov 2026 |
| (a) | At grace expiry the runtime "sends SIGKILL to any processes still running in any container in the Pod". | ✅ | Same | *Pod Lifecycle*, step 4.1 | 12 Nov 2026 |
| (a) | "If the preStop hook is still running after the grace period expires, the kubelet requests a small, one-off grace period extension of 2 seconds." | ✅ | Same | *Pod Lifecycle* | 12 Nov 2026 |
| (a) | "At the same time as the kubelet is starting graceful shutdown of the Pod, the control plane evaluates whether to remove that shutting-down Pod from EndpointSlice objects..." | ✅ | Same | *Pod Lifecycle* | 12 Nov 2026 |
| (a) | Terminating endpoints "always have their ready status as false (for backward compatibility with versions before 1.26), so load balancers will not use it for regular traffic." | ✅ | Note the documented backward-compatibility carve-out for versions before **1.26** | *Pod Lifecycle* | 12 Nov 2026 |
| (a) | "If traffic draining on terminating Pod is needed, the actual readiness can be checked as a condition `serving`." | ✅ | Same | *Pod Lifecycle* | 12 Nov 2026 |
| (a) | "Otherwise, the containers in the Pod receive the TERM signal at different times and in an arbitrary order." | ✅ | Same | *Pod Lifecycle* | 12 Nov 2026 |
| (a) | With sidecar containers, the kubelet delays TERM to sidecars "until the last main container has fully terminated", in reverse definition order. | ✅ | Sidecar containers as documented at the page revision cited | *Pod Lifecycle* | 12 Nov 2026 |
| (a) | "if the Pod has a preStop hook that exceeds the termination grace period, emergency termination may occur" | ✅ | Same | *Pod Lifecycle* | 12 Nov 2026 |
| (a) | "If the readiness probe returns a failed state, the EndpointSlice controller removes the Pod's IP address from the EndpointSlices of all Services that match the Pod." | ✅ | Same | kubernetes.io, probes documentation | 12 Nov 2026 |
| (a) | The tutorial sets `terminationGracePeriodSeconds: 120` with a `preStop` sleep of 180, shows a terminating endpoint with `ready=false`, `serving=true`, `terminating=true`, and notes the server keeps processing requests throughout. | ✅ | Tutorial last modified **18 Nov 2024** | *Explore Termination Behavior for Pods And Their Endpoints* | 12 Nov 2026 |

### 13.2 Spring Boot (docs.spring.io)

| # | Claim | Verdict | Version gate | Source | Date |
|---|---|---|---|---|---|
| (b) | "Graceful shutdown is enabled by default with all three embedded web servers (Jetty, Reactor Netty, and Tomcat)... It occurs as part of closing the application context and is performed in the earliest phase of stopping SmartLifecycle beans. This stop processing uses a timeout which provides a grace period during which existing requests will be allowed to complete but no new requests will be permitted." | ✅ | **4.1.x** reference line | *Graceful Shutdown* reference page | 12 Nov 2026 |
| (b) | "To disable graceful shutdown, configure the `server.shutdown` property", with `server.shutdown=immediate` as the disabling value. | ✅ | 4.1.x | Same | 12 Nov 2026 |
| (b) | The 3.5.x page says "all four embedded web servers (Jetty, Reactor Netty, Tomcat, and Undertow)". | ✅ | **3.5.x** reference line | *Graceful Shutdown*, 3.5.x | 12 Nov 2026 |
| (b) | "Jetty, Reactor Netty, and Tomcat will stop accepting new requests at the network layer. Undertow will accept new connections but respond immediately with a service unavailable (503) response." | ✅ | 3.5.x | Same | 12 Nov 2026 |
| (b) | "Graceful shutdown is supported with all four embedded web servers... When enabled using `server.shutdown=graceful`, upon shutdown, the web server will no longer permit new requests and will wait for a grace period for active requests to complete. The grace period can be configured using `spring.lifecycle.timeout-per-shutdown-phase`." | ✅ | Spring Boot **2.3** release notes, wiki page edited 14 Nov 2024 | Spring Boot 2.3 Release Notes | 12 Nov 2026 |
| (b) | Graceful shutdown arrived in 2.3. | ✅ | "Since: **2.3.0**" | `org.springframework.boot.autoconfigure.context.LifecycleProperties` javadoc, 3.5.16 API | 12 Nov 2026 |
| (b) | **The release in which the `server.shutdown` default changed from `immediate` to `graceful` is NOT established.** | ⚠ **flagged** | Unknown | Not found in the pages read; no version named | 12 Nov 2026 |
| (b) | `server.shutdown` - "Type of shutdown that the server will support." Default `graceful`. | ✅ | 4.1.x line | Property appendix | 12 Nov 2026 |
| (b) | `spring.lifecycle.timeout-per-shutdown-phase` - "Timeout for the shutdown of any phase (group of SmartLifecycle beans with the same phase value)." Default `30s`. | ✅ | 4.1.x line | Property appendix | 12 Nov 2026 |
| (b) | `spring.jms.listener.receive-timeout` - "Timeout to use for receive calls. Use -1 for a no-wait receive or 0 for no timeout at all. The latter is only feasible if not running within a transaction manager and is generally discouraged since it prevents clean shutdown." Default `1s`. | ✅ | 4.1.x line | Property appendix | 12 Nov 2026 |
| (b) | `spring.jms.listener.session.transacted` - "Whether the listener container should use transacted JMS sessions. Defaults to false in the presence of a `JtaTransactionManager` and true otherwise." **No static default.** | ✅ | 4.1.x line | Property appendix | 12 Nov 2026 |
| (b) | `spring.jms.listener.session.acknowledge-mode` - "Acknowledge mode of the listener container." Default `auto`. | ✅ | 4.1.x line | Property appendix | 12 Nov 2026 |
| (b) | `spring.jms.listener.auto-startup` default `true`; `spring.jms.cache.enabled` default `true`; `spring.jms.cache.consumers` default `false`. | ✅ | 4.1.x line | Property appendix | 12 Nov 2026 |
| (b) | There is **no** `spring.jms.listener.concurrency` key in the property appendix consulted; concurrency is expressed through the min/max-concurrency pair. | ✅ (absence confirmed in the appendix as read) | 4.1.x line | Property appendix | 12 Nov 2026 |

### 13.3 Spring Framework JMS (javadoc; current = Spring Framework 7.0.10 API)

| # | Claim | Verdict | Version gate | Source | Date |
|---|---|---|---|---|---|
| (c) | `DefaultMessageListenerContainer` is "a loop of `MessageConsumer.receive()` calls that also allow for transactional receipt of messages (registering them with XA transactions)". | ✅ | 7.0.10 API | `DefaultMessageListenerContainer` javadoc | 12 Nov 2026 |
| (c) | "On startup, it obtains a fixed number of JMS Sessions to invoke the listener, and optionally allows for dynamic adaptation at runtime (up to a maximum number)." | ✅ | 7.0.10 API | Same | 12 Nov 2026 |
| (c) | "Actual MessageListener execution happens in asynchronous work units which are created through Spring's TaskExecutor abstraction. By default, the specified number of invoker tasks will be created on startup, according to the concurrentConsumers setting." | ✅ | 7.0.10 API | Same | 12 Nov 2026 |
| (c) | "It is strongly recommended to either set sessionTransacted to true or specify an external transactionManager." | ✅ | 7.0.10 API | Same | 12 Nov 2026 |
| (c) | "Note that for the default AUTO_ACKNOWLEDGE mode, this container applies automatic message acknowledgment before listener execution, with no redelivery in case of an exception." | ✅ | 7.0.10 API | Same | 12 Nov 2026 |
| (c) | "stopping and restarting a listener container will only work with an independent, locally cached Connection, not with an externally cached one" | ✅ | 7.0.10 API | `CachingConnectionFactory` javadoc | 12 Nov 2026 |
| (c) | With an externally cached connection, "dynamic scaling ... can result in JMS messages delivered to cached consumers even when they are no longer attached to the listener container." | ✅ | 7.0.10 API | Same | 12 Nov 2026 |
| (c) | `setIdleReceivesPerTaskLimit`: "Marks the consumer as idle after the specified number of idle receives have been reached. An idle receive is counted from the moment a null message is returned by the receiver after the potential setReceiveTimeout(long) elapsed"; effective default 10 with the default task executor; as of 6.2 the limit applies to surplus consumers up to max-concurrency rather than to core consumers. | ✅ | 7.0.10 API (the 6.2 note is the javadoc's own version note) | Same | 12 Nov 2026 |
| (c) | `setMaxMessagesPerTask`: "Default is unlimited (-1) in case of a standard TaskExecutor, reusing the original invoker threads until shutdown". | ✅ | 7.0.10 API | Same | 12 Nov 2026 |
| (c) | `stop()` - "Stop this container."; `doStop()` - "Notify all invoker tasks and stop the shared Connection, if any."; `shutdown()` - "Stop the shared Connection, call doShutdown(), and close this container."; `doShutdown()` - "Close the registered invokers. ... A shared JMS Connection, if any, will automatically be closed afterwards." | ✅ | 7.0.10 API | `AbstractJmsListeningContainer` javadoc | 12 Nov 2026 |
| (c) | **The javadoc does not document a per-message drain barrier that waits for an in-flight listener invocation to finish before the shared Connection closes.** | ✅ (as a documentation gap; the absence was checked in the four method descriptions above) | 7.0.10 API | Same | 12 Nov 2026 |

### 13.4 IBM MQ

| # | Claim | Verdict | Version gate | Source | Date |
|---|---|---|---|---|---|
| (d) | "To help handle units of work, IBM MQ provides the BackoutCount attribute. This is incremented each time that a message within a unit of work is backed out. If the message repeatedly causes the unit of work to abnormally end, the value of the BackoutCount finally exceeds that of the BackoutThreshold. This value is set when the queue is defined." | ✅ (extract) | IBM MQ **9.3.x** | IBM Documentation, *Committing and backing out units of work*, via search-result extract | 12 Nov 2026 |
| (d) | "The backout count always survives restarts of the queue manager." | ✅ (extract) | IBM MQ **9.3.x** | IBM Documentation, *Messages that are backed out*, via search-result extract | 12 Nov 2026 |
| (d) | "Applications running inside WebSphere Application Server and those that use the IBM MQ Application Server Facilities use this attribute to determine if a message should be backed out. For all other applications, apart from allowing its value to be queried, the queue manager takes no action based on the value of the attribute." | ✅ (extract) - **the load-bearing claim of section 9** | IBM MQ **9.2.x** | IBM Documentation, `BackoutThreshold` queue attribute, via search-result extract | 12 Nov 2026 |
| (d) | "On queue managers that do not support the BackoutThreshold and BackoutRequeueQName queue attributes, JMS ConnectionConsumer uses a value of 20 for the backout threshold. When the BackoutCount of a message reaches this threshold, the message is processed as specified by the disposition options." | ✅ (extract) | IBM MQ **10.0.x** | IBM Documentation, `MQRC_BACKOUT_THRESHOLD_REACHED` (2362, 093A), via search-result extract | 12 Nov 2026 |
| (d) | "If a backout occurs, the messages which were retrieved from the queue are reinstated on the queue, and they can be processed again in another unit of work. If the processing of a particular message is causing the problem, the unit of work is backed out again. This could cause a processing loop." / "An application can detect messages that are caught up in such a loop by testing the BackoutCount field of MQMD." | ✅ (full read) but **legacy edition** | WebSphere MQ for z/OS manual, (c) IBM 1993, 2002 - **not current-release documentation** | *Messages that are backed out*, publibfp.boulder.ibm.com | 12 Nov 2026 |
| (d) | **Therefore a DMLC consumer is not documented to get automatic diversion on `BackoutThreshold`.** | ⚠ **inference** - each premise is documented; the conclusion is not stated by IBM | - | Reasoning from the two rows above; stated as inference in section 9.3 | 12 Nov 2026 |
| (d) | That MQ **automatically moves** the message for a DMLC. | ❌ **rejected** | - | Nothing in the sources supports it; the documentation scopes the behaviour to ASF/ConnectionConsumer-style delivery | 12 Nov 2026 |
| (d) | Any IBM MQ topic read in full from `www.ibm.com/docs`. | ❌ **not achieved** | - | `www.ibm.com/docs/*` unreachable to this run's fetcher on every attempt (scraper error) | 12 Nov 2026 |

### 13.5 Jakarta Messaging 3.1

| # | Claim | Verdict | Version gate | Source | Date |
|---|---|---|---|---|---|
| (e) | `AUTO_ACKNOWLEDGE`: "the session automatically acknowledges a client's receipt of a message when it has either successfully returned from a call to receive or the message listener it has called to process the message successfully returns." | ✅ | Jakarta Messaging **3.1**, Final Release 25 Feb 2022 | Section 6.2.10, *Message acknowledgment* | 12 Nov 2026 |
| (e) | `CLIENT_ACKNOWLEDGE`: "a client acknowledges a message by calling the message's acknowledge method. Acknowledging a consumed message automatically acknowledges the receipt of all messages that have been delivered by its session." | ✅ | 3.1 | Section 6.2.10 | 12 Nov 2026 |
| (e) | "If a client uses automatic acknowledgment, calls to acknowledge are ignored." | ✅ | 3.1 | Section 3.x, `Message.acknowledge` | 12 Nov 2026 |
| (e) | "When a client uses the AUTO_ACKNOWLEDGE mode, it is not in direct control of message acknowledgment... Only a session's last consumed message is subject to this ambiguity." | ✅ | 3.1 | Section 6.2.11, *Duplicate delivery of messages* | 12 Nov 2026 |
| (e) | Transacted session: "When a transaction commits, its atomic unit of input is acknowledged and its associated atomic unit of output is sent. If a transaction rollback is done, its produced messages are destroyed and its consumed messages are automatically recovered." | ✅ | 3.1 | Section 6.2.7, *Transactions* | 12 Nov 2026 |
| (e) | "Closing a transacted session must rollback its transaction in progress. Closing a client-acknowledged session does NOT force an acknowledge." | ✅ | 3.1 | Section 6.2.15, *Closing a session* | 12 Nov 2026 |
| (e) | "When session close is invoked it should not return until its message processing has been shut down in an orderly fashion. This means that none of its message listeners are running, and that if there is a pending receive, it has returned with either null or a message." | ✅ | 3.1 | Section 6.2.15 | 12 Nov 2026 |
| (e) | "If close succeeds and the acknowledge mode of the session is set to AUTO_ACKNOWLEDGE, the current message will still be acknowledged automatically when the onMessage call completes." | ✅ | 3.1 | Section 6.2.15 (and the parallel passage in section 6.1.8 for connection close) | 12 Nov 2026 |
| (e) | `JMSRedelivered`: "it is likely, but not guaranteed, that this message was delivered but not acknowledged in the past." | ✅ | 3.1 | Section 3.4.7 | 12 Nov 2026 |
| (e) | `JMSXDeliveryCount`: "set to the number of times a message has been delivered. The first time a message is received it will be set to 1"; "The value of the JMSXDeliveryCount property is not guaranteed to be exactly correct. The Jakarta Messaging provider is not expected to persist this value". | ✅ | 3.1 | Section 3.5.11 | 12 Nov 2026 |
| (e) | Footnote on transacted-session terminology: "The term 'transacted session' refers to the case where a session's commit and rollback methods are used to demarcate a transaction local to the session. In the case where a session's work is coordinated by an external transaction manager, a session's commit and rollback methods are not used and the result of a closed session's work is determined later by the transaction manager." | ✅ | 3.1 | Document footnote 4, referenced from section 6.1.8 | 12 Nov 2026 |

### 13.6 Rejected

Two claims were rejected during this pass, and both were candidates that a plausible guide would have asserted:

- **"IBM MQ automatically diverts a message to the backout requeue queue when `BackoutCount` exceeds `BackoutThreshold`, for any consumer."** ❌ as stated. The `BackoutThreshold` attribute documentation scopes the threshold's *effect* to "Applications running inside WebSphere Application Server and those that use the IBM MQ Application Server Facilities", and states that for all other applications the queue manager "takes no action based on the value of the attribute". The automatic diversion is a documented behaviour of a delivery model that `DefaultMessageListenerContainer` does not use.
- **"The `spring.lifecycle.timeout-per-shutdown-phase` default of 30 seconds means a Spring application has 30 seconds to drain before Kubernetes kills it."** ❌ as stated. The timeout is per phase, it starts after `SIGTERM`, and with the documented pod default of 30 seconds and any `preStop` hook at all, the pod's clock expires first - so the timeout never fires (section 10.2). The two defaults being equal does not make the two budgets equal.

A third candidate was **downgraded rather than rejected**, and is recorded so that a downgrade is not mistaken for a verification:

- **"The specification's `AUTO_ACKNOWLEDGE` redelivery guarantee applies to `DefaultMessageListenerContainer`."** ⚠ as stated. The specification defines the acknowledgment moment as the successful return of the listener, while the container's javadoc describes acknowledgment "before listener execution". Section 5.3 states the tension and declines to resolve it, because the resolution was not verifiable from documentation in this pass.

---

### 13.7 What Could Not Be Verified

Listed explicitly, because an unlisted gap becomes an assumed fact.

1. **The exact Spring Boot release in which the `server.shutdown` default changed to `graceful`.** The 2.3 release notes describe the feature as opt-in; the 3.5 and 4.1 pages and the current property appendix describe it as the default. The intermediate release note that records the flip was not retrieved. **Do not cite a version for this change on the strength of this guide.** Section 4.4.
2. **Why Undertow is absent from the 4.1 graceful-shutdown sentence.** The 3.5 page names four servers and describes Undertow's 503 behaviour; the 4.1 page names three. Neither page explains the difference. Section 4.2 deliberately does not speculate.
3. **Whether `DefaultMessageListenerContainer` waits for an in-flight listener invocation before closing the shared `Connection`.** The javadoc documents `stop()` as "Stop this container" and `doStop()` as notifying invoker tasks and stopping the shared connection, but documents no drain barrier. **The implementation was not read in this pass**, so the actual behaviour is unknown to this guide, and section 5.6 states the gap rather than filling it. This is the single most consequential unverified item in the document: everything in section 11's step 5 depends on it.
4. **How the specification's `AUTO_ACKNOWLEDGE` moment and the container's "acknowledgment before listener execution" reconcile.** Both are quoted; the reconciliation is an interpretation. Section 5.3.
5. **The numeric `SmartLifecycle` phase values** for the web server and for the listener container, and therefore the precise interleaving of their stop processing. Section 4.5 declines to state them.
6. **Any IBM Documentation page read in full from `www.ibm.com/docs`.** Every attempt returned a scraper error. The four IBM claims in section 9 rest on search-result extracts carrying the IBM topic text, plus one legacy z/OS manual read in full from `publibfp.boulder.ibm.com` that is explicitly a 1993/2002 edition. **A reader making a production decision on the `BackoutThreshold` scoping should confirm the sentence against `www.ibm.com/docs` at their release level** - it is the claim the whole of section 9 turns on.
7. **Whether IBM MQ emits any event, log record, or statistic that identifies a consumer disconnect during an open unit of work in a way that can be joined to a deployment.** Nothing was read on this, and section 12.2 therefore describes the missing evidence as an architectural property rather than asserting that MQ records nothing.
8. **The exact behaviour of `receive()` on a stopping container when the receive timeout is `-1` (no-wait).** The property documentation mentions `-1` as a value ("Use -1 for a no-wait receive") but the interaction with the stop path was not documented in the pages read, and section 5.5 therefore discusses only the finite-timeout case and the documented warning about `0`.
9. **IBM MQ classes for JMS specifics** - connection factory properties, reconnect behaviour, and any provider-specific shutdown behaviour. This guide treats the provider as a black box behind the Jakarta Messaging API, deliberately, because the DR guide's section 6 already owns reconnection and the Java client's documented limitation, and this guide does not want to contradict or duplicate it.
10. **The behaviour of any service mesh sidecar's own drain configuration**, and how a mesh's `preStop` interacts with a main container's termination. Section 3.5 states the general rule from the Kubernetes documentation and stops there.
11. **Kubernetes patch-level behaviour below the page revision cited.** The *Pod Lifecycle* page was read at its 28 October 2026 revision, and its API reference links point at the v1.37 API. Where a behaviour might be gated to a specific minor version, this guide says "documented as of the page revision dated 28 Oct 2026" rather than naming a version it did not verify.
12. **Anything about the `ContainerStopSignals` feature gate.** A per-container `stopSignal` override exists in the platform (alpha, gated, disabled by default) and is discussed generically in [dockerfile_best_practices_guide.md](dockerfile_best_practices_guide.md) section 9.4; it was not re-verified in this pass and is not relied on anywhere in this guide.

---
## 14. Glossary

- **`AUTO_ACKNOWLEDGE`** - The Jakarta Messaging acknowledgment mode, and the documented default for `spring.jms.listener.session.acknowledge-mode` (`auto`). The specification defines acknowledgment as occurring when `receive` returns or when the listener successfully returns; the container's javadoc describes it as applying acknowledgment **before** listener execution. The tension is real and is stated in section 5.3.
- **Backout** - The rollback of a unit of work, returning its messages to the queue. Increments `BackoutCount`. The mechanism by which a transacted shutdown becomes a duplicate rather than a loss. Section 5.4.
- **`BackoutCount`** - IBM MQ's per-message rollback counter, which "always survives restarts of the queue manager". Durable, unlike `JMSXDeliveryCount`. Sections 6.1, 9.1.
- **`BackoutRequeueQName` / backout requeue queue** - The queue a message is moved to when it exceeds the backout threshold. The destination exists as a queue attribute; **the diversion is not performed by the queue manager for a plain JMS consumer**. Sections 9.2, 9.3.
- **`BackoutThreshold`** - The queue attribute holding the backout count at which a message should be diverted. Documented as acted on by "Applications running inside WebSphere Application Server and those that use the IBM MQ Application Server Facilities"; for "all other applications ... the queue manager takes no action based on the value of the attribute". The load-bearing quote of section 9.
- **Clock one** - The pod's termination grace period, `terminationGracePeriodSeconds`, documented default 30 seconds, containing any `preStop` hook and ending in `SIGKILL`. Section 3.
- **Clock two** - The Spring application context's per-phase shutdown timeout, `spring.lifecycle.timeout-per-shutdown-phase`, documented default `30s`, applying to one phase of `SmartLifecycle` beans at a time. Section 4.
- **Clock three** - The duration of one listener invocation plus its acknowledgment or commit. No default, no configuration key, measured by nobody. Section 5.
- **`ConnectionConsumer`** - A Jakarta Messaging interface from the application-server facilities chapter, used by application servers to deliver messages to session pools. The delivery model for which IBM documents automatic backout-threshold diversion. A `DefaultMessageListenerContainer` is not one. Sections 9.2, 9.3.
- **`DefaultMessageListenerContainer` (DMLC)** - Spring's message listener container built on "a loop of `MessageConsumer.receive()` calls"; obtains a fixed number of sessions at startup and runs listener invocations as task-executor work units. Sections 5.1, 8.3.
- **Drain** - The period during which intake has stopped but in-flight work is still completing. **The word is used loosely in most guidance and precisely nowhere**; for a queue consumer there is no platform mechanism that produces one (section 7.2).
- **Effectively once** - The honest name for at-least-once delivery plus an idempotent consumer. It is a property of the pair, not of the broker, and it is not "exactly once". Section 15.1.
- **Endpoint race** - The documented concurrency between a pod's termination and its removal from EndpointSlice objects. It protects routed traffic; it does not protect a queue consumer, because queue messages do not arrive through an endpoint. Section 7.
- **`graceful` / `immediate`** - The two documented values of `server.shutdown`. `graceful` is the documented default on the 3.5 and 4.1 lines; `immediate` disables the drain. Section 4.1.
- **Grace period extension (2 seconds)** - The "small, one-off grace period extension" the kubelet requests when a `preStop` hook is still running at grace expiry. Headroom for the hook, not for the application. Section 3.3.
- **Idempotency** - The property that processing the same message twice has the same effect as processing it once. Not a shutdown feature: it is what makes the redelivery a transacted shutdown produces harmless rather than merely visible. Requires a business key, a uniqueness constraint committed with the business write, and the same key carried to every external effect. Section 15.2.
- **Idle receive** - A `receive()` that returns `null` after the receive timeout elapses, counted against `setIdleReceivesPerTaskLimit` (effective default 10) to mark a surplus consumer idle. Section 5.5.
- **In-flight unit of work** - A message that has been delivered to a consumer and whose acknowledgment or commit has not yet completed. The subject of the whole guide.
- **`JMSRedelivered`** - A message header set when a message is being redelivered. Documented as "likely, but not guaranteed" evidence that the message was delivered but not acknowledged before. Section 6.1.
- **`JMSXDeliveryCount`** - A mandatory message property counting deliveries, starting at 1. Documented as "not guaranteed to be exactly correct" and **not persisted** by the provider. The counter applications are told to read, and the one that may reset. Sections 6.1, 9.5.
- **Phase (SmartLifecycle)** - A group of lifecycle beans sharing a phase value, stopped together under one timeout. The unit `spring.lifecycle.timeout-per-shutdown-phase` actually bounds. Section 4.5.
- **preStop (`lifecycle.preStop`)** - The pod-spec hook run inside the container before `SIGTERM`. Delays clock one's `SIGTERM`; **does not stop a message listener**. Section 3.4.
- **Readiness probe** - A probe whose failure removes the pod's IP from the EndpointSlices of matching Services. A statement about routed traffic, **not** about queue delivery. Section 7.3.
- **`receive()`** - The blocking JMS call at the centre of the container's loop. Its timeout (`spring.jms.listener.receive-timeout`, default `1s`) is the granularity at which the third clock can be interrupted, and the documentation warns that `0` "prevents clean shutdown". Section 5.5.
- **Redelivery** - A message delivered again after a rollback, a recovery, or a connection failure. Visible, and therefore preferable to a silent loss - but only survivable if downstream is idempotent. Section 6.4.
- **`serving` condition** - The EndpointSlice condition the Kubernetes documentation points at when "traffic draining on terminating Pod is needed", as distinct from `ready`. Section 7.4.
- **`sessionTransacted`** - The container setting that makes acknowledgment transactional. Documented default is **conditional**: false in the presence of a `JtaTransactionManager`, true otherwise. Section 8.2.
- **`SIGKILL`** - The uncatchable signal sent to processes still running at grace expiry. The end of clock one, and the reason no application-level "last resort" exists. Section 3.1.
- **`SIGTERM`** - The signal the runtime sends to process 1 at step 2.2 of the termination flow, after any `preStop` hook. The start of clock two.
- **Sidecar container** - A container declared with `restartPolicy: Always` in `initContainers`, whose termination the kubelet delays until the last main container has terminated, in reverse definition order. Section 3.5.
- **Silent loss** - The outcome of a `SIGKILL` during a listener invocation in `AUTO_ACKNOWLEDGE` mode: acknowledged before execution, no redelivery, no error, no evidence. Section 6.4, item 1 of section 12.1.
- **`spring.jms.listener.auto-startup`** - Whether the listener container starts with the context. Documented default `true`. Section 8.2.
- **`spring.jms.listener.max-concurrency` / `min-concurrency`** - The pair through which concurrency is expressed. There is no `spring.jms.listener.concurrency` key in the appendix consulted. Section 8.3.
- **`spring.jms.cache.consumers`** - Whether consumers are cached. Documented default `false`, which is the safe side of the documented warning about messages delivered to cached consumers no longer attached to the container. Section 5.7.
- **Termination grace period** - See **Clock one**.
- **Three clocks** - The organising frame of this guide: the pod's countdown, the context's per-phase timeout, and the in-flight unit of work. Independent, unsynchronised, and owned by three different teams.

---

## 15. Idempotency, the Audit Question, and the Evidence

### 15.1 Why No Ordering Is Sufficient

Everything in sections 3 to 12 is an attempt to make the termination orderly. It is worth being blunt about what that buys: **ordering changes the frequency of the bad outcome, not its possibility.** There are four independent reasons, and they do not share a fix.

1. **The last instant is not yours.** `SIGKILL` cannot be caught, blocked or deferred ✅. However carefully the sequence is arranged, there is a final moment that the application does not control and cannot observe.
2. **The default mode spends the message before the work begins.** In `AUTO_ACKNOWLEDGE` the container "applies automatic message acknowledgment before listener execution, with no redelivery in case of an exception" ✅. The window of exposure is therefore the whole listener invocation, not a narrow race at its end.
3. **The container's drain barrier is undocumented** (section 5.6). A sequence that assumes the connection closes only after the in-flight work completes is assuming something the javadoc does not promise.
4. **The side effect that survives a rollback is the one that was never in the transaction.** A rollback returns the message; it does not un-call the clearing adapter, un-send the notification, or un-write the row in a second database. The duplicate is created at a boundary the messaging transaction does not cover.

A fifth reason is organisational rather than technical, and it is the one that decides whether any of the other four matter: **an orderly shutdown can only be verified by construction, and construction drifts.** A manifest is edited, a sidecar is added, a downstream call loses its timeout, a Spring Boot line is upgraded across the default flip of section 4.4. The sequence that was proven correct in March is a different sequence by November, and nothing announces the change.

The conclusion is not that ordering is pointless. Ordering is what moves the failure from the silent-loss row of section 6.4's table into the visible-duplicate row, and that is a large improvement - a duplicate can be reconciled, a loss cannot. **But the discipline only closes when the consumer is idempotent, because idempotency is the only property that makes the duplicate harmless rather than merely visible.** At-least-once delivery plus an idempotent consumer is the strongest guarantee available in this architecture, and it is usually written as "effectively once" - a phrase worth distrusting if it is used to mean "exactly once", because nothing in the three clocks delivers that.

### 15.2 What "Idempotent" Has to Mean for a Message That May Arrive Twice

The word is used loosely enough that it is worth stating as a specification an implementation can be checked against. Six requirements, in the order that matters:

**1. The key is a business identifier carried in the message, not the transport identity.** A payment instruction already has one - the instruction reference the originator assigned - and that is the key to dedupe on. The broker's own message identity is assigned by the provider, and whether it is preserved across a redelivery, a rollback or a reconnect is provider behaviour this guide did not verify (⚠). A design that keys its dedupe store on a value the provider owns is a design with an unverified dependency in its most important field.

**2. The dedupe decision and the business write commit together.** The pattern that fails is check-then-act: query the dedupe store, find nothing, do the work, write the record. Two consumers can pass the check concurrently - and after a rolling deployment there are, briefly, two consumers by construction. The pattern that holds is a uniqueness constraint in the same local transaction as the business write: the second attempt fails at commit, and the failure is caught and treated as "already done" rather than as an error to be retried.

**3. The constraint is the record.** A dedupe table without a unique index is a cache with a race. The index is what makes the guarantee enforced by the database rather than by the care of the code path that happened to run.

**4. Every external effect carries the key through.** The clearing adapter, the notification service and the ledger posting cannot join your database transaction, so each of them must receive the same business key and be idempotent on it in its own right. A consumer that is idempotent locally and calls a downstream that is not has moved the duplicate, not removed it.

**5. The retention window outlives the redelivery horizon.** The dedupe record must survive at least as long as a message can come back: the broker's retry behaviour, the redelivery after a rollback, the replay after a failover (the DR guide's territory), and the operational reality of a manual re-drive days later. A dedupe record with a one-hour TTL is a guarantee with a one-hour scope, and the shutdown case it is meant to cover is the one where the retry arrives after the deploy window.

**6. Idempotency is per effect, not per message.** One instruction may cause three effects - a database row, a downstream call, an audit entry - and each needs its own guarantee. The common failure is to make the database write idempotent, declare the consumer idempotent, and leave the downstream call exposed.

None of these is specific to shutdown. That is the point: **the shutdown problem is where a pre-existing idempotency gap becomes visible, not where it is created.** A team that discovers, during a graceful-shutdown review, that its consumer was never idempotent has found something more valuable than the review they set out to do.

### 15.3 Which Fix Comes First

There is a sequencing question here that teams get wrong in a specific direction, and it is worth stating with its counter-argument rather than as a rule.

**The argument for idempotency first:** setting `session.transacted=true` converts a silent loss into a guaranteed duplicate (section 5.4). If idempotency is not in place, that change converts an invisible defect into a visible one the operation cannot yet absorb - a duplicate payment instruction is a worse incident than a lost one, because it moves money twice and someone has to explain it. Ordering the work as *idempotency, then transacted, then measurement, then budget* means each step strictly improves the estate.

**The counter-argument, which is also honest:** a team that cannot yet make the consumer idempotent may still be better off transacted than non-transacted, because a duplicate is detectable and a loss is not - and detection is the precondition for every reconciliation control the audit question in section 15.5 depends on. The two positions differ on what to do while the dedupe work is unfinished, not on where to end up.

What is not defensible is the third position: leaving the acknowledgment mode at an inherited conditional default, having never decided, and describing the result as "we have not had a problem". Section 8.2 is why that position is not a position - the effective value depends on whether a `JtaTransactionManager` happens to be in the context, which is a property of the assembled application rather than of the configuration file.

### 15.4 The Evidence: Proving That Nothing Was Lost

The claims audit in section 13 is about what this guide could verify. This subsection is about what an operator has to verify about their own estate, which is a different and harder problem, because **a successful shutdown is not an event** (section 12.2). Four claims have to be evidenced, and each one has a different instrument.

| Claim | How it is observed | What a failure looks like | Why a log search cannot establish it |
|---|---|---|---|
| **No message was lost** | Reconciliation by business reference between the producer's record of what it sent and the consumer's record of what it processed, over the change window | A reference present in the producer's record with no corresponding processing record, and no dedupe or backout trace | The loss mode logs nothing at all; the deployment is green either way |
| **No effect happened twice** | Dedupe-store hit counts per reference; unique-constraint violations on the business key; downstream reconciliation on the same key | A reference with two processing records, or a downstream posting without a matching local record | A duplicate handled correctly is silent by design - the second attempt is rejected and logged at most at debug level |
| **No unexpected backout** | The distribution of `BackoutCount` on the queue before and after the deploy window, and the backout-requeue queue depth | A cluster of messages whose count rose during the window without any corresponding processing failure | The counter is durable and visible to tooling, but nothing correlates it with a deployment unless you do |
| **The shutdown actually happened in order** | Three application log lines per pod: context close started, drain finished, container stopping - plus the container's exit code and termination timestamp | A pod that stops logging after "context close started", or an exit that arrives at the grace-period boundary | Order is invisible in aggregate metrics; only the sequence of lines per pod shows it |

Two disciplines make the first row real rather than aspirational. **Run the reconciliation on a controlled population, not on production as a whole.** A deploy performed against a known set of instruction references - a canary stream, or a test value date - gives a bounded population in which a single missing reference is findable. Reconciliation over an unbounded production population answers "roughly the right number of messages", which is not an answer an auditor can use. **Reconcile by reference, never by depth.** A queue depth that returns to zero proves that messages were consumed; it proves nothing about whether they were processed, and a message acknowledged before the listener ran (section 5.2) leaves the depth at zero with the work undone.

The uncomfortable arithmetic of this section is that **the instrument has to exist before the deployment you want to evidence.** A team that decides after an incident to reconcile by business reference discovers that the producer's record of what it sent was never retained in a joinable form. That is not a shutdown defect; it is an observability defect that a shutdown review exposes.

### 15.5 The Banking and Regulated Angle: The Question Is "Prove It", Not "Did It Happen"

In a regulated institution the interesting property of this failure is not its severity but its **evidentiary shape**, and it is worth stating as the distinction that makes this a banking subject rather than a middleware subject.

**An unplanned outage produces a bounded population.** It has a start, an end, and a scope: these services, these hours, these customers. Reconciliation is therefore a finite exercise with a defensible answer, and the operational-resilience framework - impact tolerances, the evidence model, the change-related outage category - owns how that answer is produced. None of that is re-derived here; [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) is the reference.

**A silent shutdown loss produces an unbounded one.** There is no start time, because each deployment looks like a success. There is no end time, because the condition persists until someone changes the sequence. There is no scope, because the messages affected are the ones that happened to be in flight at an instant nobody recorded. The question an auditor asks is not "did an incident occur" - the answer to that is genuinely "no incident was recorded" - but **"prove that no instruction was lost or duplicated during the change window"**, and the change window is every deployment in the period under review.

That reframing has four consequences that land on the engineer rather than the auditor:

- **Change is the highest-frequency risk in the estate, and it is the one with the weakest evidence.** A deployment is a change event that should be evidence-producing by design. Most are not, because the evidence of a good deployment is indistinguishable from the evidence of a quiet failure.
- **Undetectable loss is worse than a visible outage, and not only rhetorically.** An outage consumes the error budget and produces a reconciliation scope. A silent loss consumes nothing, triggers nothing, and leaves a population that cannot be enumerated - so the control that would catch it has to be built before there is any reason to build it.
- **"No incident" is not a control.** A control produces evidence on a schedule, whether or not anything went wrong. If the only artifact of a graceful shutdown is a green pipeline, the control does not exist and the absence of incidents is an observation about luck.
- **The acknowledgment mode is an audit-relevant fact, not a tuning choice.** A team that cannot state its effective `spring.jms.listener.session.transacted` value cannot answer the audit question at all, because it cannot say whether a bad shutdown would have lost or duplicated. Section 8.2's conditional default is therefore not a configuration curiosity; it is an unanswerable question sitting in the middle of a control.

The honest framing for a solution architect is this: **the payment instruction is the unit of audit, and the shutdown is the moment at which the estate stops being able to say what happened to it.** Everything else in this guide is machinery in service of being able to answer, for one reference, whether it was processed once.

---

## 16. Cross-References and the Closing Summary

### 16.1 Where to Read Next

**The two adjacent guides, and why.** [ibm_mq_disaster_recovery_guide.md](ibm_mq_disaster_recovery_guide.md) is the **unplanned** recovery guide and the owner of everything this guide deliberately does not cover: HA versus DR, media-image recovery and the logging decision, the local resilience family, the site-loss architectures, and section 6's treatment of queue manager identity, automatic client reconnection (including the documented limitation for IBM MQ classes for Java), in-doubt units of work, the CCDT, and backout and backout-requeue as safety valves. **Read its section 6 before designing anything that involves a failover**; this guide's section 9 builds on its backout material rather than replacing it. [dockerfile_best_practices_guide.md](dockerfile_best_practices_guide.md) section 9 owns PID 1 and signal delivery generically - the `pid_namespaces(7)` rule, shell form versus exec form, `STOPSIGNAL`, the `docker stop` grace period, `HEALTHCHECK` versus probes - and everything here assumes that material.

**The wider set.** [message_queue_data_loss_guide.md](message_queue_data_loss_guide.md) owns the reliability discipline whose subject this guide's duplicate risk becomes (delivery semantics, idempotency, dedupe keys, the transactional outbox) - read the two as a pair, because a transacted shutdown converts a loss into a duplicate and hands the problem sideways. [zero_downtime_system_design_guide.md](zero_downtime_system_design_guide.md) covers the rolling-deployment and drain problem for request-serving systems, which is the case the platform's graceful-shutdown machinery was designed for. [openshift_workload_availability_guide.md](openshift_workload_availability_guide.md) covers pod disruption budgets and workload availability on the OpenShift side. [enterprise_middleware_integration_platform_guide.md](enterprise_middleware_integration_platform_guide.md) covers the integration platform landscape and the MQ object model. For the banking context, [../banking/payment_rails_guide.md](../banking/payment_rails_guide.md) and [../banking/operational_resilience_framework_guide.md](../banking/operational_resilience_framework_guide.md) cover the rails this protects and the framework in which change-related outage is a category of impact rather than an absence of incident.


### 16.2 The Closing Summary

**The single sentence.** Three clocks run during every deployment, two of them have documented defaults that look identical and mean different things, and the message caught between them cannot ask any of them for more time.

**What this guide established.**

- **Graceful shutdown is an ordering problem, and the ordering is in no single component.** The pod's countdown, the context's per-phase timeout, and the in-flight unit of work are owned by three different pieces of software, configured in three places, and read by nobody as a set. Two defaults of 30 seconds start at different instants, one of them applies per phase, and the third clock has no default at all.
- **The two equal defaults are actively misleading.** With the documented pod default of 30 seconds and any `preStop` hook at all, the pod's clock expires before the Spring lifecycle timeout can fire - so the timeout never fires and `SIGKILL` arrives while the context is still closing. **A timeout larger than the pod budget is not a generous timeout; it is a disabled one.** This is arithmetic on documented values (section 10.2), and it is the most useful single thing in the guide.
- **`server.shutdown=graceful` is necessary and does not solve this.** Its documented subject is requests: "existing requests will be allowed to complete but no new requests will be permitted". A JMS listener has no request, so the property serves the HTTP half and the MQ half needs a different answer.
- **The acknowledgment mode decides whether a bad shutdown is a loss or a duplicate, and its default is conditional.** `spring.jms.listener.session.transacted` "defaults to false in the presence of a `JtaTransactionManager` and true otherwise", so two applications with the same configuration file can behave differently because of a bean another team added. The default `AUTO_ACKNOWLEDGE` mode acknowledges "before listener execution, with no redelivery in case of an exception" - the message is spent before the work begins.
- **The endpoint race does nothing for a queue consumer, and the container's stop path is the documentation gap.** Readiness removes a pod from routed traffic, but messages arrive through a queue connection; a `preStop` sleep extends the window in which the consumer keeps taking work the deployment is about to kill. `doStop()` "notifies all invoker tasks and stops the shared Connection" and `doShutdown()` closes the invokers - neither sentence promises a per-message drain barrier, and the specification *does* require an orderly session close (6.2.15), so whether the container's path reaches it is not documented in the pages read.
- **The BackoutCount problem is the one IBM hands back to the application.** The counter increments on any rollback and survives restarts, but automatic diversion on threshold is documented as a function of WebSphere Application Server and the Application Server Facilities; "for all other applications ... the queue manager takes no action based on the value of the attribute". A listener that keeps getting killed drives the counter up with nothing documented parking the message, so a deploy storm inflates the count on valid messages and whether that ever becomes a parked message depends on who reads the threshold (section 9.4).
- **The evidence is missing by construction, and ordering never removes the need for idempotency.** A `SIGKILL` logs nothing and a successful drain logs nothing either, so an ordering problem that fails silently has to be verified by construction - configuration, measurement, reconciliation by business reference. And because the best sequence still ends in an uncatchable signal, only an idempotent consumer makes the surviving duplicate harmless rather than merely visible (sections 12.2, 15.1).
- **The runbook is six steps and four decisions, and the decisions come first.** Acknowledgment mode, measured maximum unit of work, the inequality that puts the Spring clock inside the pod's clock, and a plan for when the work outlives the budget anyway. A perfect sequence applied to an `AUTO_ACKNOWLEDGE` listener with an unbounded unit of work and a 30-second budget is a sequence of no-ops (section 11.1).
- **In the fictional Cymbal Bank case, every setting that mattered was a default.** The payment path's shutdown behaviour was determined by four projects' defaults meeting for the first time. Changing it required measurement, one explicit acknowledgment decision, and an application-level threshold check that product documentation did not provide - not a flag.

**The last thing worth saying.** The uncomfortable property of this problem is that it is entirely within the operator's control and almost entirely invisible. There is no incident to investigate, no vendor to escalate to, no failure that announces itself - there is a deployment that went green, and a payment instruction that was read, acknowledged, and never processed, somewhere in the middle of it. The platform gives you a deadline you can lengthen. Spring gives you a timeout you can size. The specification gives you a session close that waits for the listener. None of those three will arrange themselves in the right order, and the component with the strongest interest in that order is the one with no voice in it at all.

**Graceful shutdown is not a flag; it is an ordering problem - and the only component in the system with no way to ask for more time is the message already read.**

*End of guide.*
