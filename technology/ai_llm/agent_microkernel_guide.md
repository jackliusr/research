# Agent Microkernel - the privileged core is what can spend and act

> **Standfirst.** This guide argues that the microkernel idea transfers to LLM agents only in one specific, narrow, and useful way: not as a diagram of small modules, but as a claim about where authority is held. A microkernel is not a small system; it is a system whose privileged core is small enough to audit - and in an agent, the privileged core is whatever can spend money, call a tool and read data. Everything else in the guide follows from taking that sentence literally: naming the four authorities an agent can exercise, deciding which components must hold them, and pricing the decomposition honestly in tokens rather than microseconds. The guide refuses three claims. It does not claim that "agent microkernel" is an established term - it is not, there is no specification, and section 1.2 establishes that before any argument is made. It does not claim that a microkernel agent is safer than a monolithic one by construction; section 8 argues the opposite can be true, because a decomposition can hide what the model was actually told while looking more principled. And it does not print a benchmark, a latency comparison, an adoption curve or a market size anywhere, because the fact pack behind this guide contains none and the performance numbers it does quote are operating-system figures from 1993-1995 papers, not agent figures; the guide's contribution is the failure mode in section 8.3 and the four-authority decomposition in section 4, both of which section 16 records as this guide's reasoning.

> **Byline:** Jack Liu Shurui, Solution Architect  \
> **Repo:** personal research library | **Series:** LLM/AI Engineering & Operations | **Domain:** AI Engineering / Agent Architecture  \
> **Date:** November 2026 | **Sources read:** 2026-11-01

> **How to read it.** Section 1 does the housekeeping that has to come before any argument: it defines eight terms in two registers, establishes the provenance of a term that has none, and disposes of the Microsoft Semantic Kernel collision - if you read only one section, read 1.2 and 1.4, because a reader who skips them will spend the rest of the guide thinking this is about a DI container. Sections 2 and 3 are the borrowed half: what a microkernel actually is in the OS literature, dated and sourced, and exactly where that analogy holds and where it breaks. Section 4 is the centre of the guide and the only section that must be understood before the rest is useful: the four authorities (spend, tool dispatch, data reach, state commit) and the derivation that makes them non-arbitrary. Sections 5 and 6 divide the world into what belongs in the kernel and what must not, each item with the reason it cannot cross the line. Section 7 is the plugin contract, which is where informal architectures quietly turn back into monoliths. Section 8 is the cost question and the guide's own failure mode. Section 9 is the three architectures and how each one fails. Section 10 is the regulated reading. Section 11 is the state of practice, split into what exists, what is proposed, and what is merely named. Section 12 is a fictional Cymbal Bank worked example with real numbers of components and no invented benchmarks. Sections 13 to 16 are the apparatus: anti-patterns, selection guidance, the claims audit, and what could not be verified.

---

**Contents**

1. [Overview, Decoder, Provenance, Thesis and Scope](#1-overview-decoder-provenance-thesis-and-scope)
2. [What a Microkernel Actually Is](#2-what-a-microkernel-actually-is)
3. [Where the Analogy Holds](#3-where-the-analogy-holds)
4. [What Is Actually Privileged in an Agent](#4-what-is-actually-privileged-in-an-agent)
5. [What Belongs in the Kernel](#5-what-belongs-in-the-kernel)
6. [What Must Not](#6-what-must-not)
7. [The Plugin Contract](#7-the-plugin-contract)
8. [The Cost Question, Honestly](#8-the-cost-question-honestly)
9. [The Failure Modes of Each Architecture](#9-the-failure-modes-of-each-architecture)
10. [Banking and the Regulated Angle](#10-banking-and-the-regulated-angle)
11. [The State of Practice](#11-the-state-of-practice)
12. [Worked Example: Cymbal Bank Forced the Split](#12-worked-example-cymbal-bank-forced-the-split)
13. [The Anti-Patterns](#13-the-anti-patterns)
14. [Selection Guidance](#14-selection-guidance)
15. [The Claims Audit](#15-the-claims-audit)
16. [What Could Not Be Verified, Glossary, Cross-References and Closing](#16-what-could-not-be-verified-glossary-cross-references-and-closing)

---

## 1. Overview, Decoder, Provenance, Thesis and Scope

### 1.1 Decoder: eight terms, two registers

The OS vocabulary this guide borrows was built for hardware. The agent vocabulary it is mapped onto was built for language models. The words overlap, and the overlap is the source of most of the confusion the guide has to clear, so each term is defined twice: once in the sense the operating-systems literature uses, and once in the sense this guide uses when it talks about agents. The fourth column states which sense the bare word carries in the rest of the file.

| Term | OS meaning | Agent meaning used here | Bare word means |
|---|---|---|---|
| **microkernel** | The minimal part of an operating system that must run in privileged mode; everything else - drivers, file systems, pagers - runs as unprivileged servers communicating by message passing | The minimal set of components that must hold the four authorities of section 4, because nothing outside them can be trusted to mediate | Both, and the whole guide is about the mapping between them |
| **kernel** | The part of the OS that is mandatory and common to all other software (Liedtke's own phrasing, 1995) | The privileged core: the components that hold credentials, dispatch effects, and write the authoritative record | The agent sense unless the sentence is explicitly about OS history |
| **userland** | Everything outside the kernel: processes running unprivileged, including the servers that provide OS services in a microkernel design | Every component that has no authority of its own and must ask the kernel to exercise authority on its behalf | Agent sense, except in section 2 |
| **server** | In the OS sense, an unprivileged process that implements a policy or a service on top of the kernel's mechanisms - a file server, a pager, a device driver | A plugin or service that implements domain behaviour and holds no authority | The OS sense, and when the agent sense is meant the guide writes "plugin" or "service" |
| **IPC** | Inter-process communication: the message-passing mechanism by which a client asks a server, and a server asks the kernel, to do something; in Liedtke's work the single most performance-critical path in the system | The crossing of the authority boundary: a request from userland to the kernel, priced in context tokens and narration rather than cycles | OS sense for mechanism, agent sense for cost - section 8 is where the two are reconciled |
| **mechanism versus policy** | The separation, stated as a design principle in Hydra (1975), between what a system *can* do (mechanism, in the kernel) and *what it chooses* to do (policy, in userland) | The kernel enforces that a capability is required and recorded; userland decides who gets which capability, for how long, under what conditions | Both, with the note that this principle predates the microkernel argument and does not come from Liedtke |
| **capability** | An unforgeable token of authority: possession of the token is the right to use the object, and the kernel checks possession rather than identity (seL4 is the canonical verified example) | A row in the grant table: principal, plugin, action, scope, expiry, and whether it is revocable, checked at dispatch | Agent sense, because agent capabilities are rows in a table rather than words in a token - see the caveat in 4.5 |
| **plugin** | Not an OS term of art; the nearest OS analogue is a user-level server | A unit of domain behaviour with a declared manifest (section 7) that runs in userland and reaches the world only through kernel dispatch | Agent sense only |
| **agent loop** | No OS analogue | The control structure that alternates model call, tool selection, tool result, and termination decision | Agent sense only |

Two consequences fall out of the table immediately, and they matter more than the definitions. First, five of the eight terms have a genuine OS referent, so the borrowing is not decorative - the analogy has real structure to transfer. Second, three of them - plugin, agent loop, and the agent sense of capability - have no OS referent at all, which means the mapping is partial by construction and section 3 has to say exactly where it stops. A guide that borrowed all eight terms cleanly would be borrowing nothing.

### 1.2 Provenance: the term is not a standardised term

This has to be settled before the argument, because the argument's credibility depends on not pretending to a lineage it does not have. **"Agent microkernel" is not a standardised term.** It is not a term of art in operating-systems research, it is not a term in the agent-engineering literature, and it is not defined by any standards body, vendor specification, or reference architecture. Three statements, in order of how much they constrain what follows.

**(i) The OS literature contains no such term.** The microkernel literature - Brinch Hansen's nucleus work, Liedtke's L3 and L4 papers, the seL4 verification programme, the survey and retrospective literature around them - is about operating-system kernels. It does not discuss agents, LLM-based or otherwise, and it does not use the phrase. This is not a gap in the literature that a diligent search would close; it is a category difference. The people who built microkernels were solving a hardware-mediated protection problem, and the papers are explicit about the hardware assumptions they rest on (section 2.7). Nothing in that body of work anticipates a system where the "process" is a stochastic model and the "system call" is a tool invocation described in natural language.

**(ii) Exactly two live usages were found and verified.** Both were read on 2026-11-01, both are dated, and both are recorded in section 1.3 with their exact strings. One is the self-description of a small open-source project (Castor). The other is a Chinese-language architecture document that proposes the concept (claude-books). Neither is a specification. Neither defines the term normatively. One calls itself a microkernel and describes what it does; the other proposes the framing as a position in an argument about what an agent platform should be. Two usages, two genres, no shared definition between them.

**(iii) There is no specification, no standard, no shared definition, and no literature behind the term.** There is no document that says what an agent microkernel must do, what interfaces it must expose, what properties it must hold, or how conformance would be tested. There is no body of papers that cite a common definition. There is no vendor reference architecture. A reader who wants to know whether their system "is" an agent microkernel has nothing to check it against except an argument - this one, or another one.

The consequence is a rule the guide holds to throughout: **this guide is borrowing a framing from operating-system design, not reporting a received term.** When section 4 says the privileged core is what can spend, call a tool and read data, that is this guide's definition, offered as an argument with reasons, not a citation of an authority. When section 11 separates what exists from what is proposed from what is merely named, that separation exists precisely because the term's status is thin. A guide that presented this framing as established would be doing the thing section 15 is built to catch.

### 1.3 The two live usages, with their exact strings

**Castor** - `github.com/ApatheticMioz/Castor`, read 2026-11-01. The repository's About line reads, verbatim: "The Universal 245K Agent Microkernel for Local LLMs." The README headline reads, verbatim: "Universal Cloud-to-Local Agent Bridge & MCP Microkernel in Rust". The repository also carries the GitHub topic `agent-microkernel`. Dated facts as read: latest commit `4d882b6` on 27 October 2026; latest release v1.4.0 on 26 October 2026, the fourteenth release; 12 stars, 3 forks, 2 contributors; AGPL-3.0 with a commercial dual-licence option. The README describes in-process tool dispatch and zero tool-output tokens entering cloud context, and reports the project's own telemetry figure of "93.9% KV cache hit across 53 days of production telemetry".

Three things need saying about that record, and none of them is a judgement of the project. The "245K" in the name refers to a long-context model target, not to a count of kernel lines - a reader who assumes otherwise will misread the term as a size claim, which is exactly the misreading section 2.4 and section 3.2 exist to prevent. The numbers are self-reported and unverified here; they are not used as evidence for or against the decomposition anywhere in this guide. And the star count is recorded as a dated fact about a repository, not as an adoption claim about the technique - 12 stars is a small project by any reading, and section 11 refuses to rank or characterise maturity on the strength of it.

**claude-books** - `github.com/wangxumarshall/claude-books`, read 2026-11-01. A Chinese-language architecture book repository that proposes the concept, using a Chinese term rendered in English as "Agent Microkernel", framed as "neither an out-of-the-box terminal app nor an all-powerful orchestration platform". That English sentence is the dispatcher's translation of the repository's own framing, and is marked as translated and attributed to that repository rather than quoted as the repository's English prose. The repository had 0 stars and no description at the time of reading. Its structure is a self-published set of architecture notes: directories covering harness, AgentOS memory, AgentOS multi-agent systems, and infra. It is a **proposal**, not a specification, and its status as a proposal is the only claim the guide makes about it.

The two usages do not agree on much beyond the phrase. Castor uses it to describe a shipped Rust component with a focus on tool dispatch and context economics; claude-books uses it to stake out a position between a terminal application and an orchestration platform. That disagreement is itself the evidence for 1.2(iii): when a term has two users and no shared definition, the term is a framing, and framings are argued rather than cited.

### 1.4 Semantic Kernel is a false friend, and the collision is expensive

Microsoft's Semantic Kernel is the single most likely thing a reader will find when searching this repository, or the web, for "kernel" near "agent". In this repository, `technology/ai_llm/multi_agent_banking_guide.md` line 276 lists its core abstractions as "The kernel (the orchestrator), the plugins and functions, the memory, the agent abstractions". Microsoft's own documentation describes it as "a lightweight, open-source development kit that lets you easily build AI agents and integrate the latest AI models into your C#, Python, or Java codebase" and as "an efficient middleware".

The kernel concept itself is defined on Microsoft's concepts page (dated 04/16/2025), verbatim: "The kernel is the central component of Semantic Kernel. At its simplest, the kernel is a Dependency Injection container that manages all of the services and plugins necessary to run your AI application." And, one paragraph later: "The kernel is extremely lightweight (since it's just a container for services and plugins)".

That is not a microkernel. It is not a smaller version of a microkernel, an approximation of one, or a microkernel with a different name. It is the opposite architecture claim, and the reasons are specific rather than rhetorical:

1. **There is no privilege boundary.** A microkernel exists to separate privileged from unprivileged code, and the separation is enforced by hardware and checked by the kernel on every crossing. In Semantic Kernel, userland and kernel are the same address space and the same object graph. The "kernel" is an object that holds references to services and plugins; the plugins are objects too; there is no ring, no mode switch, and nothing to check on a crossing because there is no crossing.
2. **Anything holding the kernel object can call any plugin directly.** The DI container is a registry with a resolution API. If a component can resolve `IKernel` or its equivalent, it can obtain any registered plugin and invoke it, which is precisely the property section 5.2 says a mediation boundary must not have.
3. **The kernel does work that a microkernel framing would place in userland.** The same Microsoft page describes the kernel selecting the best AI service, building the prompt from a template, calling the AI service and parsing the response. Model selection and prompt construction are, in this guide's decomposition, userland responsibilities for stated reasons (section 6.5 and 6.6) - they change weekly and they are policy. A component that does them is a composition root, not a privileged core.

The practical warning is blunt: a reader who greps this repository for "kernel" plus "agent" will land on Semantic Kernel, and landing there teaches the opposite of this guide's point. The vocabulary matches and the architecture does not. Semantic Kernel's "kernel" is a container that makes everything reachable; this guide's kernel is a boundary that makes most things unreachable except through one path. Section 13.7 treats the collision as an anti-pattern in its own right, because projects that adopt the word while shipping a DI container inherit the confusion without inheriting the argument.

### 1.5 The thesis, and the test it must pass

Stated once, and then defended for fifteen sections:

> A microkernel is not a small system; it is a system whose privileged core is small enough to audit - and in an agent, the privileged core is whatever can spend money, call a tool and read data.

The thesis has two halves and the second is the load-bearing one. The first half is a correction: microkernel does not mean small, and the confusion between "small" and "small enough to audit" is the most common way the term gets misused, in OS arguments and agent arguments alike. Liedtke's own membership criterion is functional, not dimensional (section 2.6), and he says outright that minimality cannot be proved because there is no agreed metric (section 2.6). So "small" is not a property anyone can measure, and a guide that sold smallness would be selling something unverifiable.

The second half is a claim about where the boundary sits in an agent, and it is falsifiable in a specific way: **the analogy is only worth making if the OS privilege boundary maps onto the agent authority boundary.** That is the test. If the mapping holds, then every OS argument about kernel membership, mechanism versus policy, and the cost of crossing has an agent analogue, and the guide can borrow thirty years of hard-won reasoning. If the mapping does not hold - if the agent's "kernel" is a naming convention with no enforceable boundary behind it - then the whole vocabulary is decoration, and the honest thing to write is a guide about dispatch-point policy checks rather than a guide about microkernels.

Section 3 states where the mapping holds and names three places where it breaks. Section 4 derives the authority boundary rather than asserting it, because a boundary that is asserted is a boundary that can be redrawn by anyone with a different opinion, and the entire value of the OS analogy is that the OS boundary was not a matter of opinion - the hardware forced it.

### 1.6 Scope: what this guide owns, and what it does not

The repository already contains three guides that cover adjacent ground, and the boundary between them is worth stating precisely because all four use the same systems as examples.

- [agent_loop_implementation_guide.md](agent_loop_implementation_guide.md) owns the loop as a control structure, across four systems. This guide treats the loop as one kernel resident (section 5.1) and does not re-describe how to implement one.
- [agent_scaffolding_guide.md](agent_scaffolding_guide.md) owns scaffold anatomy: what the parts of an agent scaffold are. This guide asks a different question about a subset of those parts - which of them must hold privilege - and does not enumerate the anatomy again.
- [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md) owns the ETCLOVG layering (Execution, Tooling, Context, Lifecycle, Observability, Verification, Governance). This guide is orthogonal to the layering: a layer is a category of concern, an authority is a thing a component can do, and the two cross-cut. The trace in section 5.5 is an Observability-layer artefact that this guide insists must be written by the authorising component, which is a claim about custody that the layering does not make.

This guide owns one question none of them asks: **which parts of an agent must hold privilege, and what happens to the design when you answer that question first.** Answering it first changes the module boundaries, the credential placement, the plugin interface, and the prompt, in that order - which is why the guide is structured as a decomposition rather than a checklist.

**Coverage check, measured 2026-11-01 in `/home/ubuntu/research`, case-insensitive over `*.md`:** "microkernel" appears in 0 files. `\bAIOS\b` appears in 0 files. The pattern `agent kernel|kernel.*agent|agent.*kernel` matches 17 files, and all 17 are the operating-system sense or the Semantic Kernel false friend - the Kairos kernel, Linux kernel modules, eBPF and kernel-level security, and Microsoft's SDK. "agent runtime" appears in 31 files; `plugin.*agent|agent.*plugin` in 22; "message passing" or "IPC" in 29. The numbers say two things at once: the repository has substantial coverage of the *components* this guide assembles (runtimes, plugins, message passing), and no coverage at all of the *question* it asks. The 17 kernel hits are the dedup argument - if this guide were going to duplicate an existing treatment, it would be duplicating one of those, and it does not, because none of them is about privilege in an agent.

---

## 2. What a Microkernel Actually Is

The borrowed half of the argument has to be accurate, because the guide's whole method is to take OS reasoning seriously enough that its conclusions can be checked. What follows is dated and sourced, in the order the ideas appeared, with the secondary-sourced items marked as such.

### 2.1 The nucleus, before the word existed (1969)

The ancestor is the RC 4000 multiprogramming system's nucleus, 1969, designed by Per Brinch Hansen: a message-passing nucleus for up to 23 unprivileged processes, with no built-in strategy for resource allocation. ⚠ This item comes from a secondary summary (the Wikipedia "Microkernel" article, read 2026-11-01) and not from the primary paper; the numbers are recorded as the summary gives them. The design move is the one that matters and it is visible without the primary source: the nucleus does message passing and nothing else, and every policy decision about which process gets what is left outside it. That is a separation of nucleus from policy, made six years before anyone wrote down mechanism-versus-policy as a principle (section 2.5), and it is the reason the RC 4000 is the conventional starting point for the lineage even though the word did not exist yet.

The term "microkernel" was in use no later than 1981. ⚠ Also secondary-sourced, and also recorded as the summary gives it. The point of the date is narrow: by the early 1980s the vocabulary was established, which means the ideas in sections 2.2 to 2.4 were argued in a shared language rather than invented separately.

### 2.2 Three architectures, and the hybrid compromise

The taxonomy that matters for this guide has three members, not two.

- **Monolithic.** One privileged image contains the scheduler, memory management, file systems, device drivers and network stack. Every internal call is a function call; every component can reach every other component's data structures. Fast, and unbounded in the sense that there is no structural limit on what the privileged core can do or on how large it can grow.
- **Microkernel.** The privileged core is reduced to the mechanisms that cannot live outside it: address spaces, threads, and IPC. Drivers, file systems, pagers and network stacks become unprivileged servers that communicate by message passing. Slower per operation, bounded by construction.
- **Hybrid.** A microkernel-derived structure with substantial servers moved back inside the privileged image for performance or compatibility. The two large commercial examples are the ones that matter: macOS from 2001 uses the XNU kernel, which combines a heavily modified Mach (OSF MK 7.3) with BSD, and Windows NT from NT 3.1 is also classified as hybrid. ⚠ Both classifications come from the secondary summary.

The Amiga's Exec, 1986, is the early commercial case worth naming for a reason that cuts against romanticising microkernels: it was fast partly because it **lacked memory protection**. ⚠ Secondary-sourced. A microkernel without protection hardware is a message-passing structure with no privilege boundary - modules that talk by messages but can also write each other's memory. It is the first worked example of the failure this guide keeps returning to: the architecture's vocabulary without the architecture's enforcement. Section 9.3 is the agent version of it.

### 2.3 Mach, and the performance lesson that set the field back a decade

Mach was created by Richard Rashid, and its IPC performance was disappointing. ⚠ Secondary-sourced, and the disappointment is the well-known part. What happened next is the part that matters: most large-scale Mach efforts had ended by 2000, and a wave of systems moved servers back into the kernel to recover the lost performance. ⚠ Secondary-sourced. That retreat is documented from the other side by Liedtke, who in 1995 described it as a diagnosis rather than a defeat: "In some cases, the micro-kernel interface has been weakened and special servers have been re-integrated into the kernel to regain efficiency." ✅ (Liedtke 1995, full text.) He attributes the inefficiency and inflexibility of microkernels to causes that are "not inherited from the basic idea but mostly from overloading the kernel and/or from improper implementation" ✅ - a claim the guide takes seriously in section 6.2, because it is the same failure mode as an agent kernel that accumulates special cases.

So the field's history contains a genuine retrenchment, not a misunderstanding. Systems that had adopted the structure went back. Any honest transfer of the analogy has to carry that fact, and section 8 does: a decomposition that costs more than it buys gets abandoned, and in agents the cost is not microseconds but context.

### 2.4 Liedtke's answer: the IPC path was the problem

Liedtke's 1993 paper, "Improving IPC by Kernel Design" (14th ACM SOSP, Asheville NC, December 1993, pp. 175-188), is the response, and its numbers are the ones this guide quotes and refuses to translate into agent terms. Most microkernels "typically requir[ed] 100 microseconds for a short message transfer on a modern processor, running with 50 MHz clock rate" ✅. L3 achieved 5 microseconds, "a twentyfold improvement" ✅, and against Mach specifically the results "range from a factor of 22 (8-byte messages) to 3 (4-Kbyte messages)" ✅.

The mechanism of the improvement matters more than the factor. On a 486-DX50 at 50 MHz the minimal null-IPC sequence is 20 instructions and 127 cycles, 172 cycles once the unavoidable TLB misses are counted; the author aimed at 350 cycles and "achieved 250 cycles (5 microseconds)" ✅. He also records that "simply entering and leaving kernel mode costs 2 microseconds (40%)" of the transfer ✅ - that is, two fifths of the cost of a message was the mode switch itself, before any of the kernel's own work. And the design principle is stated verbatim as a sentence, which is why it is quotable: "Ipc performance is the Master." ✅

Two structural facts from the same paper complete the picture. All device drivers in L3 were user-level tasks communicating only via IPC ✅ - the microkernel claim carried all the way down to the drivers, not just to the file system. And "the dedicated parts of the microkernel can be concentrated in a single medium sized module" ✅ - note "medium sized", which is Liedtke's own phrase and a useful corrective to anyone who reads microkernel as "tiny".

The argument Liedtke makes is therefore narrow and empirical: **the IPC path, not the design idea, was the problem.** The decomposition was sound; the implementation of the crossing was not. That is the single most transferable sentence in the literature for agent architects, and section 8 is built on it - except that in an agent the crossing is not a mode switch and the cost is not cycles, so the transfer has to be argued rather than assumed.

### 2.5 Mechanism versus policy is older than the microkernel argument, and comes from Hydra

This attribution has to be exact, because it is commonly muddled. Mechanism-versus-policy separation was established as a basic design principle in **Hydra** - Levin, Cohen, Corwin, Pollack and Wulf, "Policy/mechanism separation in Hydra", Fifth ACM Symposium on Operating Systems Principles, 1975, DOI 10.1145/800213.806531. The abstract, verified via the ACM Digital Library record, reads: "The extent to which resource allocation policies are entrusted to user-level software determines in large part the degree of flexibility present in an operating system. In Hydra the determination to separate mechanism and policy is established as a basic design principle and is implemented by the construction of a kernel composed (almost) entirely of mechanisms." ✅

**It is not Liedtke's principle.** Liedtke 1995 does not use the phrase, and the microkernel argument is not where the idea originates - it is twenty years younger than Hydra and inherits the principle rather than discovering it. The distinction matters for the agent transfer in a practical way: a team that attributes mechanism-versus-policy to the microkernel literature will tend to think the separation is part of "being a microkernel", when it is an independent principle that a monolith can honour and a microkernel can violate. Section 6.1 depends on the separation being independent, because the argument there is that policy must leave the kernel *even though* the kernel is a microkernel - which only makes sense if the two ideas are distinct.

### 2.6 Membership: the tolerance criterion, and the admission

Liedtke's 1995 paper, "On Micro-Kernel Construction" (15th ACM SOSP, Copper Mountain Resort, Colorado, December 3-6 1995), opens with the definition the guide uses in section 1.1: "Traditionally, the word kernel is used to denote the part of the operating system that is mandatory and common to all other software. The basic idea of the micro-kernel approach is to minimize this part, i.e. to implement outside the kernel whatever possible." ✅

He states three advantages, verbatim: "(a) A clear micro-kernel interface enforces a more modular system structure. (b) Servers can use the mechanisms provided by the micro-kernel like any other user program. Server malfunction is as isolated as any other user program's malfunction. (c) The system is more flexible and tailorable. Different strategies and APIs, implemented by different servers, can coexist in the system." ✅ Advantage (b) is the isolation claim, and advantage (c) is the flexibility claim that section 6.1 leans on when it argues that competing policies must be able to coexist.

The membership criterion is in section 2, verbatim: "The determining criterion used is functionality, not performance. More precisely, a concept is tolerated inside the micro-kernel only if moving it outside the kernel, i.e. permitting competing implementations, would prevent the implementation of the system's required functionality." ✅ Note what the criterion is not. It is not a line count, not a complexity budget, not a list of subsystems. It is a counterfactual test: move it out, allow a competitor, and see whether something the system must do becomes impossible. That is why section 3.2 insists the goal is "small enough to audit" rather than "small" - the criterion is functional, so any size claim is derivative and unmeasurable.

And the admission, in a footnote, verbatim: "Proving minimality, necessary and completeness would be nice but is impossible, since there is no agreed-upon metric and all is Turing-equivalent." ✅ This is the sentence that keeps the guide honest. Liedtke, defending the strongest version of the argument, says the argument's central property cannot be proved. Any guide that claims to have identified the minimal agent kernel is claiming more than the source of the criterion claimed for operating systems.

Two further requirements are derived from protection, and they are the ones section 3.3 transfers: "the principle of independence: S can give guarantees independent of S0" and "the principle of integrity: there must be a way for S1 to address S2 and to establish a communication channel which can neither be corrupted nor eavesdropped by S0" ✅. Note that these are stated as requirements on what a subsystem can *rely on*, not as requirements on code size or module count. A subsystem must be able to guarantee something regardless of what other subsystems do, and other subsystems must be able to rely on that guarantee. That is a stronger and more useful statement than "keep the kernel small", and it is the version the guide transfers.

Also from 1995, two operational details that prefigure section 5. Threads must be in the kernel because "To prevent corruption of address spaces, all changes to a thread's address space must be controlled by the kernel" ✅, and the address-space operations retained in the kernel are exactly three: "only the grant, map and flush operations are retained inside the kernel" ✅. Section 3 of the paper demonstrates the flexibility claim by showing memory managers, pagers, device drivers and file systems all implemented on top of the microkernel ✅. There is also a warning worth carrying: microkernels are "inherently not portable, although they improve portability of the whole system" ✅ - the privileged core is the least portable part, which in agent terms means the kernel is the part most coupled to a particular vendor's tool protocol and least able to be swapped.

### 2.7 seL4: what "verified" covers, and what it assumes

seL4 is the strongest empirical claim in the lineage, and its scope has to be stated precisely because "verified kernel" is routinely over-read. The project's fact sheet states: "seL4 is the only kernel with inspectable proofs of deep properties down to the implementation" ✅; "1.3 million lines of proofs" ✅; "0 violations of verified properties since proof completed in 2009" ✅. The verified properties are functional correctness, binary correctness, integrity and confidentiality ✅. The project states that this goes "beyond Common Criteria EAL7, ISO26262 ASIL-D or DO-178C Level A certification requirements for development" ✅, and describes seL4 as "the only verified general-purpose, capability-based operating system" ✅. Its own performance claim, quoted only as the project's claim and not as a benchmark, is "seL4 is 2 to 10 times faster than other comparable systems" ✅.

The verification page states that the proofs run from high-level specification down to binary code ✅ - the whole stack, not a model of it, which is what makes the claim unusual. And then the assumptions: **the proofs assume assembly code, boot code, the machine interface, the hardware, and DMA** ✅. That is, a verified kernel rests on assumptions outside the proof, and those assumptions are where manual scrutiny has to concentrate.

This is the most useful single fact in section 2 for an agent architect, and it is not about seL4 being weak. It is that **verification is always relative to an assumed base**, and the base is where the argument is won or lost. The agent analogue is exact: an agent kernel is only as good as the credential custody it assumes. A perfectly mediated dispatch path sitting on top of plugins that each hold their own database password has verified the wrong thing - it has proved a property of the code while assuming away the property that mattered. Section 4.6 turns this into the custody-boundary claim, and section 9.3 is the failure mode it predicts.

### 2.8 The two conditions the OS argument silently depends on

The 1995 paper states its assumptions in the open, in one sentence, immediately before the requirements: "We assume that the target system has to support interactive and/or not completely trustworthy applications, i.e. it has to deal with protection. We further assume that the hardware implements page-based virtual memory." ✅

That is the whole foundation, and it is two conditions:

1. **Protection hardware.** There must be a mode in which code cannot reach certain operations, and a hardware-enforced transition into it. Without this, a "kernel" is a coding convention and any component can do what the kernel does.
2. **Page-based virtual memory with an MMU.** There must be a hardware mechanism that intercepts every memory access and can refuse it, so that mediation is not optional. The 1995 paper's argument for keeping threads in the kernel is exactly this - "all changes to a thread's address space must be controlled by the kernel" ✅ - and it is only true because the MMU makes it true.

Both are properties of the hardware, not of the design. The microkernel argument is a *consequence* of the hardware, and every one of its strong claims - that mediation is unavoidable, that isolation is real, that a refusal is final - traces back to one of these two conditions.

**An agent has neither.** There is no mode bit that a model cannot set, no MMU that intercepts a tool call, and no hardware that refuses a plugin's outbound HTTP request because the plugin was not granted egress. The consequence is not that the analogy fails; it is that the analogy has to be rebuilt on a different foundation, and section 3.2 names exactly where the rebuilt version breaks. The rest of this guide is about what survives the rebuild.

---

## 3. Where the Analogy Holds

### 3.1 The mapping that has to hold, and the three that do

The test from section 1.5 is whether the OS privilege boundary maps onto the agent authority boundary. Four mappings survive scrutiny, and it is worth being specific about why each one is structural rather than verbal.

**Privilege boundary as authority boundary.** In an OS, the boundary separates code that can affect the machine from code that cannot. In an agent, the corresponding separation is between components that can cause an effect outside the process - spend a metered resource, invoke a tool, read a store, commit a state - and components that cannot. This is the load-bearing mapping, and section 4 spends its length making it precise, because "authority" is a word that can mean anything until it is enumerated.

**Auditability.** The OS claim is that a small privileged core can be inspected by a person and reasoned about as a whole. The agent claim is identical in form and different in content: the kernel must be small enough that a reviewer can answer "what can this agent do, and under whose authority" by reading one component and one table. Note the phrasing carefully - **small enough to audit, not small**. Liedtke minimises by function, not by line count, and says explicitly that minimality has no agreed metric (section 2.6). A team that sets a line-count budget for its agent kernel has misunderstood the criterion and will optimise the wrong thing: they will compress code and leave the authority surface unchanged.

**Independence.** Liedtke's "the principle of independence: S can give guarantees independent of S0" ✅ transfers almost without modification. A plugin must be able to rely on the kernel's enforcement regardless of what other plugins do. If plugin A can widen plugin B's reach - by writing to a shared credential store, by mutating a registry, by re-registering a tool under a name B trusts - then B's guarantees are not independent, and the boundary is nominal. This is why section 5.3 insists the grant table has exactly one owner: two components that can answer "may this proceed" are two components whose answers are not guarantees.

**Integrity.** The second principle - "there must be a way for S1 to address S2 and to establish a communication channel which can neither be corrupted nor eavesdropped by S0" ✅ - is the one agent systems violate most often without noticing. In an agent, S0 is not a malicious process but the model's own context: the transcript is a shared mutable channel that every component writes to, and a component that can write to the transcript can corrupt the channel that another component's authority decisions travel over. Section 6.7's authority-manifest rule exists for this reason. If the manifest the model reads is generated by the same path that enforces the grant, then a plugin cannot inject a claim about permissions that the kernel does not hold.

### 3.2 Where the analogy breaks: three named breaks

The breaks have to be named, not gestured at, because a guide that only lists the mappings is a guide that has stopped arguing.

**(B1) No MMU and no protection ring.** The agent kernel is a process-level and call-path convention, not a hardware-enforced mode. Concretely: a plugin that holds its own database credential, or constructs its own HTTP client, bypasses the kernel entirely and the kernel has no mechanism to notice. The boundary is only as real as the credential custody behind it - which is why section 4.4 calls the agent kernel's boundary a *custody* boundary rather than a protection boundary, and why section 14 says that where custody cannot be centralised the decomposition is decorative. This is not a limitation the guide can argue away; it is a property of the substrate, and the honest response is to design for it (one credential holder, network egress control, no plugin-local secrets) rather than to draw the diagram and hope.

**(B2) The cost being optimised is context and tokens, not microseconds.** Liedtke's 250 cycles and 5 microseconds are the numbers that made the microkernel argument winnable in 1993 because the cost of crossing was measurable, bounded, and comparable across designs. An agent's crossing is priced differently: state must be serialised into the model's context, the authority decision must be narrated back to the model, and both consume a budget that is scarce in a way cycles never were. So the decomposition is priced in narration. That changes the optimisation target completely - a design that reduces the number of crossings by batching is doing the right thing for the wrong-sounding reason, and a design that adds a crossing to gain a clean audit trail may be buying a real property at a real price rather than a free improvement.

**(B3) The failure mode inverts.** This is the guide's own contribution and it is developed in section 8.3; here it is stated in brief. An OS microkernel fails **loudly**: IPC overhead shows up as a benchmark number, the system is slower, everyone can see it, and the argument is settled by measurement. An agent microkernel fails **silently**: the indirection hides what the model was actually told. Every added crossing is an opportunity for the model's picture of its own permissions to diverge from the kernel's enforcement, and nothing in the system reports the divergence, because the model's belief is not an observable and the kernel's enforcement is working correctly by its own definition. So the OS failure mode is a cost that is visible and the agent failure mode is a risk that is not - which means the usual market discipline that corrected the microkernel's performance problems does not operate on the agent version at all. A decomposition can therefore make an agent less safe while looking more principled, and no benchmark will show it.

### 3.3 What the analogy buys, stated as a limit

Three things transfer and one does not, and the guide is only worth writing because the three are substantial.

Transfers: the **membership test** (a component belongs inside only if moving it out would prevent a required function - section 4.5 applies this to agents); the **mechanism/policy separation** (the kernel enforces, userland decides - section 6.1); and the **independence/integrity requirements** (subsystems must be able to rely on guarantees that other subsystems cannot undermine - section 5).

Does not transfer: the **performance argument**. There is no agent equivalent of the 250-cycle IPC path, no comparable measurement, and therefore no way to settle a decomposition argument by benchmark. In the OS case the retrenchment from Mach was driven by a number; in the agent case there is no number, so the retrenchment, when it comes, will be driven by incident rather than by measurement. That is a worse feedback loop, and it is an argument for getting the boundary right the first time rather than for iterating toward it.

The rest of the guide therefore does not try to prove the decomposition is faster or cheaper. It argues that the decomposition makes a specific set of questions answerable, prices what it costs to answer them, and says when the price is not worth paying (section 14.2). That is a weaker claim than the OS literature could make, and it is the strongest one the evidence supports.

---

## 4. What Is Actually Privileged in an Agent

### 4.1 The four authorities

This is the central section. Everything before it establishes that the analogy is worth making; everything after it depends on the boundary drawn here.

An agent component holds privilege if it can exercise any of four authorities. The list is meant to be exhaustive at the level of *kinds* of authority - a fifth authority would have to be a way of affecting the world that is not spending, dispatching, reading or committing, and the guide has not found one.

| Authority | What it looks like in an agent | Who can exercise it if the kernel does not mediate | The audit question it answers |
|---|---|---|---|
| **SPEND** | Tokens consumed by model calls and by context assembly; money paid to a metered API; any quota, rate-limit or licensed call that is consumed by use | Anyone who can construct a model client, including a plugin that imported its own SDK, a retry loop that has no budget, and a subprocess that inherited an API key | How much did this agent cost, per principal, per turn, per tool - and who authorised the spend? |
| **TOOL DISPATCH** | The power to cause an effect outside the process: a write to a system of record, an email, a payment instruction, a file write, a webhook, a shell command | Anyone holding a tool client or a credential; in practice every plugin that wraps its own integration | What can this agent actually do to the world, and through which path? |
| **DATA REACH** | Which stores, fields, documents and tenants are readable; which rows a query can return; which documents a retrieval step can pull into context | Anyone who can open a connection; anyone whose retrieval tool is scoped by its own configuration rather than by a grant | What could this agent have read, whether or not it did? |
| **STATE COMMIT** | The power to make a result durable, authoritative, or visible to a human or downstream system: writing the record, sending the notification, marking the case decided, appending to the ledger | Anyone who can write to the store or the queue; the executor, if it writes its own status | What did this agent make true, and on whose authority? |

### 4.2 Why these four and not code size

The temptation, when transferring the OS argument, is to define the kernel by size or by centrality: the "core" modules, the "framework" layer, the thing everything imports. Both proxies fail, and they fail in ways the OS literature already documented.

**Code size fails** because Liedtke's criterion is functional, not dimensional (section 2.6), and because size and authority are uncorrelated in agent code. A 40-line plugin that wraps a payment API and reads a customer table holds three of the four authorities. A 4,000-line orchestration module that only shuffles JSON holds none. Any decomposition that sorts by size puts the payment plugin in userland and the JSON shuffler in the kernel, which is precisely backwards.

**Module count fails** for the same reason plus one more: it is a property of the file tree, and the file tree is the thing being designed. A team can satisfy any module-count target by splitting files without moving a single credential.

**"Core-ness" fails** because it is circular - the core is whatever the team decided the core is, and the decision is the thing under examination. Semantic Kernel's "kernel" is central in exactly this sense and holds no privilege at all (section 1.4).

The four authorities work as a definition because they are what three different people independently ask about. **An auditor** asks what the agent can spend and read, because those are the exposures. **An incident reviewer** asks what it dispatched and committed, because those are the effects that have to be explained. **An attacker** asks which of the four is least mediated, because that is the path of least resistance. When a definition coincides with the questions asked by the people who have to live with the system, it is not arbitrary - it is the same boundary seen from three sides.

There is a second reason the four work, and it is the one that makes them operational: **they are the only things whose exercise can be enumerated.** You cannot enumerate "core-ness" or "complexity", but you can enumerate a grant table, a spend ledger, a reach list and a commit log. A boundary you can enumerate is a boundary you can test, review and audit. A boundary you can only describe is a boundary you can only assert.

### 4.3 The derivation: Liedtke's tolerance test, applied to agents

The mapping is not a metaphor if it is derived, so here is the derivation. Liedtke's criterion: "a concept is tolerated inside the micro-kernel only if moving it outside the kernel, i.e. permitting competing implementations, would prevent the implementation of the system's required functionality" ✅.

Substitute the agent case. The "required functionality" of an agent system includes: the ability to say what it did, the ability to bound what it can do, and the ability to stop it. Then ask of each candidate component: if this moves to userland and a competing implementation is permitted, does one of those functions become impossible?

- **The dispatch path.** If dispatch moves to userland and plugins are permitted to dispatch for themselves, then "bound what it can do" becomes impossible, because there is no single point at which a bound could be applied. Dispatch is therefore in the kernel, and the reason is functional.
- **The grant table.** If grant lookup moves to userland and competing implementations are permitted, then "stop it" becomes impossible, because revocation has no authoritative target. In the kernel.
- **The spend ledger.** If accounting moves to userland and competing implementations are permitted, then "say what it did" becomes impossible for spend, because each implementation counts what it can see and nobody sees everything. In the kernel.
- **The trace.** Same argument, applied to effects rather than cost. In the kernel.
- **The loop.** If the loop moves out and a plugin is permitted to re-enter it, then a plugin can dispatch without passing through dispatch, which makes the bound impossible again. In the kernel.
- **The message bus.** If userland can call userland directly and competing channels are permitted, the mediation path is optional and the bound is advisory. In the kernel, with the honesty caveat in section 5.2.

Now apply the test in the other direction, which is where the discipline actually bites.

- **The discrepancy rules** (section 12's example): if these move to userland and competitors are permitted, does anything become impossible? No - competing rule sets are exactly what you want, because the rules change and different desks will want different ones. Out.
- **The summarisation strategy**: same. Competing strategies are desirable. Out.
- **The model choice**: same, and more so, since price and quality move. Out.
- **The prompt templates**: same. Out.

The test produces the split without appeal to taste. That is what makes the mapping non-arbitrary: it is Liedtke's counterfactual, run on a different set of components, and it happens to land in a place that a security reviewer would also land on by instinct. When derivation and instinct agree, the definition is probably right.

The transferable form of the criterion is one sentence: **a component belongs in the agent kernel only if moving it out would leave some authority unmediated.** Note that this is a stricter test than Liedtke's in one respect - it names the thing that must not be left unmediated - and a looser one in another, since "unmediated" is a property of a path rather than of a function.

### 4.4 The asymmetry with the OS, and why it changes the design

An OS kernel mediates memory and CPU because **hardware forces every access through the MMU**. The mediation is not a policy the kernel chose to enforce; it is a consequence of how the machine works. A process that wants memory cannot get it without a page fault, and a page fault goes to the kernel. That is why the OS argument can say "the kernel mediates X" and mean it as a fact rather than an intention.

An agent kernel mediates authority **only if it is the sole holder of the credentials and the sole dispatcher**. There is no MMU for tool calls. If a plugin has its own database password, the kernel does not mediate data reach for that plugin, no matter what the diagram says. If a plugin can construct its own HTTP client, the kernel does not mediate egress. The mediation is contingent on a custody arrangement, and the custody arrangement is a deployment decision rather than an architectural one.

So: **the agent kernel's boundary is a custody boundary.** Two consequences follow, and they are the most practically important sentences in the guide.

First, **where custody is not centralised, the diagram is decorative.** A team can draw the kernel, put dispatch inside it, and still have no mediation, because three plugins hold their own credentials. The diagram will pass review - it shows a boundary - and the system will fail the incident, because the boundary was never enforceable. Section 9.3 names this as the third failure mode, and section 12 shows what forcing custody looks like when an audit question makes it unavoidable.

Second, **the custody boundary has to be designed at the same time as the credential store, not after it.** This is the sequencing point that most decompositions get wrong: teams draw the architecture, then wire up credentials wherever the integration is easiest, and the resulting custody map has nothing to do with the architecture diagram. The kernel only becomes real when the credentials live in it and everything else borrows authority rather than holding it. That is a change to secret management, network policy and service identity, not a change to module structure - which is why section 14.2 says that a team that cannot hold the credentials should not attempt the decomposition at all.

### 4.5 Capability, in the agent sense, is weaker than in the OS sense

The word "capability" needs one more caveat, because seL4's version is stronger than anything an agent can currently build. In seL4 a capability is an unforgeable token: possession is authority, the kernel checks possession, and unforgeability is a property the proofs establish. In an agent, a capability is a row in a table: principal, plugin, action, scope, expiry, revocability. It is checked at dispatch by code that the plugin cannot modify if custody is right - but it is checked by *code*, not by hardware, and "unforgeable" is replaced by "the plugin has no way to write to the table".

That is a real weakening and it should be stated rather than glossed. The practical difference: a seL4 capability cannot be forged even by a compromised kernel-mode bug in another subsystem, because the token is checked by the part that was proved. An agent grant can be forged by anything that can reach the table's storage, which is why the table's storage is inside the custody boundary and why the trace has to be append-only at the storage layer rather than merely at the application layer. The agent version buys auditability and revocation, not unforgeability, and a guide that claimed otherwise would be borrowing the OS word for something the agent cannot deliver.

### 4.6 What the four authorities do not cover

For completeness, three things sit outside the four and the guide is explicit about them rather than pretending the list is total.

**Model behaviour.** The four authorities describe what the system permits and records. They say nothing about whether the model's reasoning is sound, whether it is manipulable, or whether its output is correct. Those are model-risk questions with their own literature, and section 10 notes where they touch the kernel (the trace is evidence for a model-risk reviewer but does not make the model safer).

**Prompt content.** What the model is told is not an authority in this taxonomy; it is an input to a decision that the kernel authorises. But the manifest that describes authority must be generated by the kernel (section 6.7), because otherwise the input and the enforcement drift.

**Availability.** The four are about what an agent can do, not about whether it can be stopped from doing it too much. Rate limits, circuit breakers and kill switches are mechanism, and they belong in the kernel for the same reason dispatch does - a kill switch outside the dispatch path is a suggestion - but they are a different kind of control from a grant, and section 5.3 treats revocation as grant-shaped rather than availability-shaped.

---

## 5. What Belongs in the Kernel

Five components, each with the reason it cannot live outside, and each with a one-line description of what it looks like when it is done badly. The test applied is section 4.3's: move it out, permit competitors, and see what becomes impossible.

### 5.1 The loop

**Why it cannot live outside.** The loop is the only component that can serialise authority use **in time**. Every authority in section 4 is exercised in a sequence - read, dispatch, spend, commit - and the sequence is what makes an audit trail readable and what makes a bound enforceable at a moment. If a plugin can re-enter the loop, it can dispatch without passing through dispatch, because a re-entrant loop is a second entry point into the same authority. The OS analogue is exact: Liedtke keeps threads in the kernel because "all changes to a thread's address space must be controlled by the kernel" ✅ - the loop is the agent's thread, and the reason is the same, that control requires a single point.

There is a second reason, specific to agents. The loop is where the model is called, and the model call is where SPEND happens. A loop outside the kernel means the largest and least predictable spend in the system happens outside the accounting path. That is not a performance argument; it is an argument that the ledger would be structurally incomplete.

**Done badly:** the loop is a framework function that plugins can call to "run a sub-agent", so a plugin that needs three tool calls starts its own loop, and those three dispatches appear nowhere in the trace. The system looks like it has one loop and behaves as though it has as many loops as there are ambitious plugins.

### 5.2 The message bus

**Why it cannot live outside.** The bus must be the only channel, because if userland can call userland directly then mediation is bypassed and the call graph becomes un-auditable. The argument is the same shape as 5.1's but it applies to the horizontal dimension rather than the vertical: 5.1 controls the sequence, 5.2 controls the topology. A system where plugins call each other is a system whose effect graph is a mesh, and a mesh cannot be bounded by a point.

**Be honest about the limit.** In an agent you cannot prevent a subprocess from opening a socket. The bus is therefore **a contract backed by network and credential control, not a privilege.** This is the direct consequence of break B1 (section 3.2): there is no mode bit that stops a plugin making an outbound connection, so "the bus is the only channel" is true only to the extent that (a) the plugin has no credential to use on the other channel, (b) network policy refuses egress that does not originate from the kernel's egress path, and (c) the plugin's deployment does not include an HTTP client with ambient credentials. All three are enforcement outside the code, and a team that writes the sentence "all traffic goes through the bus" without the three is writing a comment, not a control. Section 12's egress accounting is the worked version of taking this seriously.

**Done badly:** plugins import each other's clients directly "to avoid the bus overhead", so the bus carries the traffic that was easy to route and the interesting traffic goes point to point. The trace is complete for the trivial calls and silent for the consequential ones, which is worse than no trace, because it looks complete.

### 5.3 Capability grant and revocation

**Why it cannot live outside.** There must be a **single table**, because if two components can answer "may this proceed", neither answer is authoritative. This is the independence principle applied to the grant itself (section 3.1): a plugin must be able to rely on its permissions being what the table says, and it cannot rely on that if another component can also issue permissions. Two grant stores is not redundancy; it is ambiguity, and ambiguity in an authority store is a vulnerability with a benign name.

Revocation belongs here for the same reason. A grant that can be issued in one place and withdrawn in another is not withdrawable, because the withdrawal has no defined effect on the issuer's copy. The OS has a defined effect on outstanding references when a capability is revoked; the agent version has to construct one deliberately (section 8.3(ii) argues that it currently cannot, and that this is the deepest break in the analogy).

**Done badly:** the grant table is a YAML file that three services read, one of which caches it for an hour, and the "revoke" operation is a pull request. Revocation latency is measured in deploy cycles, and during the window the table says one thing and the enforcement says another.

### 5.4 Accounting

**Why it cannot live outside.** A ledger of spend per principal, per turn and per tool belongs in the kernel because **spend is an authority** (section 4.1) and only the dispatch path sees every spend, including model calls. This is not an observability argument, and the distinction is the point of section 12's surprising survivor: metering looks like telemetry, and telemetry belongs to whoever produces it, but a ledger written by the component being metered is a self-report. The kernel is the only component that sees the model call and the tool call in the same place, in sequence, attributed to the same principal and the same turn.

There is a second, harder reason. Spend is the authority with the least visible failure. A dispatch that should not have happened produces an effect someone notices; a spend that should not have happened produces a number on an invoice, weeks later, attributed to a cost centre rather than a decision. Accounting in the kernel is what turns "the agent is expensive" into "this principal's retry loop spent this much on this tool in this turn", which is a statement someone can act on.

**Done badly:** token counts are scraped from provider dashboards monthly and reconciled by hand against a list of agents, so cost per principal is an estimate with a lag, and nobody can attribute a spike to a turn. The organisation knows it is spending and cannot say on what.

### 5.5 The trace

**Why it cannot live outside.** The trace is an append-only record of **intent, grant, dispatch and result**, and it must be written by the component that authorises, not by the component that acts. The reason is one sentence: **a trace the executor writes is a self-report.** An executor that writes its own trace can write a trace of what it intended, or of what it believed it was permitted, or - in the adversarial case - of what it wants a reviewer to see. The authorising component's record is different in kind: it says "this principal asked for this, this grant covered it, this was dispatched, this came back", and it is written on the path that also had the power to refuse. Records written by something that could have refused are evidence; records written by something that could not are testimony.

The OS analogue is the audit log written by the kernel rather than by the process being audited, and seL4's assumption list (section 2.7) is the reminder that even a proved kernel's trace rests on assumptions about the layer beneath it. The agent version of that assumption is the storage: append-only at the application layer means nothing if the executor can rewrite the storage.

**Done badly:** every plugin logs its own tool calls in its own format to its own index, and the incident review becomes an exercise in correlating five timestamp formats across four time zones, with the one plugin that mattered having logged at INFO level in a stream that was rotated after seven days. The trace existed. It was not evidence.

### 5.6 The five together

| Kernel resident | The authority it holds | The function that becomes impossible if it moves out | The bad version |
|---|---|---|---|
| Loop | SPEND (via model calls) and the ordering of all four | Serialising authority use in time; a complete spend ledger | Plugins spawn sub-loops and dispatch off-path |
| Message bus | TOOL DISPATCH (as the only channel) | A bounded effect graph | Point-to-point calls make the trace look complete and be partial |
| Grant table | All four, as the permission for each | A single authoritative answer to "may this proceed" | Three readers, one cache, revocation by pull request |
| Accounting | SPEND | Attribution of spend to principal, turn and tool | Monthly dashboard scraping, no attribution |
| Trace | STATE COMMIT (the record of it) | Evidence rather than testimony | Per-plugin logs, rotated, uncorrelated |

The table also shows why the five are not a list of good practices. Each row's third column is a *function*, and the function is what the OS criterion tests (section 4.3). Remove any one and something the system is supposed to be able to do stops being possible - which is the only defensible reason to keep a component in a privileged core.

---
## 6. What Must Not

Section 5 argued for five components on functional grounds. This section argues the other direction, and the arguments are the same kind: for each excluded component, what becomes impossible, or what becomes worse, if it is admitted. The section exists because the most common failure of microkernel designs is not under-populating the kernel - it is a kernel that grows one reasonable addition at a time until it is a monolith with a boundary drawn around it.

### 6.1 Policy

**Why it must not.** Two reasons, one from the literature and one from change management.

The literature reason is Liedtke's flexibility claim: one of the three stated advantages of the microkernel structure is that "different strategies and APIs, implemented by different servers, can coexist in the system" ✅. That advantage only exists if policy is outside. A kernel that contains the policy cannot host competing policies, because the kernel is singular by definition - there is one privileged core, and if the rule about who may spend what lives inside it, there is one answer and no room for a second. The flexibility claim is not a nice-to-have; it is one of the three reasons the architecture was argued for.

The change-management reason is sharper and applies with more force to agents than to operating systems. If policy lives in the kernel, **every policy change becomes a change to the audited core**. The core is the thing that was reviewed, the thing that has a test suite with coverage requirements, the thing that needs a second approver and a release note. A pricing change, a new desk's approval limit, a temporary tightening during a volatile week - each becomes a core change, and the predictable consequence is that the change process gets relaxed until it is not a control. The kernel then has a review process on paper and continuous churn in practice, which is the worst of both.

The separation Hydra established (section 2.5) is the exact shape needed: the kernel provides the mechanism (a grant can be issued, checked, revoked, recorded), and userland supplies the policy (who gets which grant, for how long, under what conditions). The kernel must be able to refuse a grant request that is malformed or that exceeds the mechanism's limits; it must not know why the limit is what it is.

**What goes wrong when policy is inside:** the kernel contains a conditional per desk, per product, per customer segment, and the conditional is the policy. Nobody can enumerate the policy any more because it is distributed across branches, and the review of "what can this agent do" now requires reading code rather than a table - which was the thing the kernel was supposed to make unnecessary.

### 6.2 Domain logic

**Why it must not.** Domain logic is **the thing being governed**. A kernel that contains it is a kernel that governs itself, and the independence principle (section 3.1) collapses: a plugin can no longer rely on the kernel's guarantee being independent of the business rule, because the business rule is inside the guarantor.

The literature reason is Liedtke's diagnosis, which is worth quoting again because it names the mechanism rather than the symptom: the inefficiency and inflexibility of microkernels is "not inherited from the basic idea but mostly from overloading the kernel and/or from improper implementation" ✅. **Overloading the kernel** is the named cause. In an agent, domain logic is the largest and fastest-changing body of code in the system, so it is also the most tempting thing to put in the kernel "because everything needs it". That is precisely how a microkernel becomes a monolith: not by a decision, but by an accumulation of things that everything needs.

There is a second reason specific to agents. Domain logic changes with the business, and it changes at a rate that no audited core can absorb. Section 12's example has discrepancy rules that change when the documentary-credit terms change; putting them in the kernel would couple the audit cycle to the trade-finance rulebook.

**What goes wrong when domain logic is inside:** the kernel's test suite becomes the business's regression suite, releases slow to the rule-change cadence, and the kernel's privileged status means every rule author now needs a security review to change a threshold.

### 6.3 Memory strategy

**Why it must not, and the exact distinction.** The distinction here is the whole point of the section and it is easy to lose: **the STORE is kernel, the STRATEGY is userland.**

*State commit* is an authority (section 4.1). Making a result durable, authoritative, or visible to a human or downstream system is one of the four things an agent can do that has to be mediated, and the mechanism for it - the write path, the record, the durability guarantee - belongs in the kernel for the reasons section 5.5 gives about the trace.

*Memory strategy* - what to remember, what to summarise, when to compact, what to retrieve, how to score relevance, what to forget - is a policy. It changes with the model, the task, the context budget and the experiment of the week. Liedtke's flexibility argument covers it: competing strategies should be able to coexist, and they can only coexist outside the kernel.

Teams conflate the two constantly, because both are called "memory". The test from section 4.3 separates them cleanly: move the strategy out and permit competing strategies - does anything required become impossible? No, competing summarisers are exactly what you want. Move the store out and permit competing stores - does anything required become impossible? Yes: the trace, the commit record and the durability guarantee all lose their single authority. Store in, strategy out.

**What goes wrong when strategy is inside:** the kernel contains a summarisation heuristic, the heuristic is tuned for one model's context window, the model is upgraded, and now a privileged component's behaviour depends on a parameter that was set for a system that no longer exists. Worse, the heuristic is now part of the audited core, so it cannot be changed quickly, so it stays wrong longer than it should.

### 6.4 Orchestration and multi-agent topology

**Why it must not.** Orchestration is a plan, and **the plan is what gets audited**. If the kernel decides which sub-agent runs next, then the kernel's behaviour is a function of the plan, and the plan changes with every new workflow. The kernel would then be the place where multi-agent design decisions are made, which makes every topology experiment a core change.

There is a sharper way to put it. The topology is a claim about *how authority is delegated* - which agent may ask which other agent to do what - and that claim is exactly the kind of thing that should be visible as data rather than embedded as control flow in the privileged core. In the worked example (section 12), the trade-finance agent is a single agent with plugins, and the question of whether to add a second agent is a userland decision that does not touch the kernel at all. That is the property to preserve: adding a sub-agent should be a change to a plan, not a change to the audited core.

**What goes wrong when topology is inside:** the kernel grows a router, then a router with fallbacks, then a router with a planner, and the "microkernel" now contains an orchestration engine. The kernel's size grows with the ambition of the workflow team, which is unbounded, and the audit boundary grows with it.

### 6.5 Prompt construction

**Why it must not.** Prompt construction changes weekly per experiment. It is the single fastest-moving artefact in an agent system, it is tuned by people who are not kernel engineers, and it is measured by evaluation rather than by test. A privileged core cannot absorb that rate of change, and any attempt to do so converts the core's review process into a bottleneck that will be routed around.

There is also a conceptual reason. A prompt is an input to the model's decision, not an exercise of authority. The kernel's job is to authorise and record; the prompt's job is to inform. Conflating them is the mistake that produces the anti-pattern in section 13.6, where the grant lives in the prompt because the prompt was the easiest place to put it.

**What goes wrong when prompt construction is inside:** the kernel becomes a template engine, then a template engine with a model-specific renderer, then a component that must be redeployed whenever a prompt is tweaked. The team routes around it by adding an escape hatch ("raw prompt override"), the escape hatch is the actual prompt path, and the kernel's version is dead code that still passes review.

### 6.6 Model selection

**Why it must not.** Model selection is a **routing policy tied to price and quality, both of which move.** A kernel that pins a model has pinned a price and a quality level into the audited core, and both of those change faster than any review cycle. A kernel that selects dynamically has internalised a purchasing decision, which is not a security mechanism.

The distinction from section 5.4 has to be held carefully, because it is subtle: the kernel **accounts for** model spend and **dispatches** the model call, but it does not **choose** the model. The choice is policy supplied from userland; the accounting and the dispatch are mechanism. This is the same store-versus-strategy split as section 6.3, applied to a different resource.

**What goes wrong when model selection is inside:** a price change or a model deprecation becomes a privileged-component change with a security review, in the middle of a commercial negotiation, and the team's answer is to hard-code a fallback list in the kernel, which is now a second policy.

### 6.7 The design rule that resolves the obvious objection

The objection is immediate and fair: if prompt construction lives in userland, and the model's understanding of its own permissions comes from the prompt, then the prompt is where authority is effectively communicated - so hasn't the guide just moved the authority boundary into the least controlled component?

No, and the rule that answers it is the most practically useful sentence in the guide:

> **Prompt construction stays in userland, but the AUTHORITY MANIFEST injected into the prompt is generated by the kernel from the same grant table it enforces, so what the model is told it may do and what the kernel permits cannot drift.**

Three things follow from stating it that way.

First, the manifest is **derived, not authored**. Userland composes the task, the instructions, the examples and the tone. It does not compose the list of tools, the scope of each, the spend limits, or the expiry. Those come from the kernel as a block, generated at turn start from the live grant table, and userland's only freedom is where in the prompt the block goes (which is a formatting decision). A userland component that wants to describe a capability the table does not contain cannot, because it does not write that section.

Second, the manifest is **regenerated per turn, not cached per deploy**. This is what makes revocation bite, at least partially - section 8.3(ii) argues it does not bite fully, because the transcript retains the stale version, but regenerating per turn means the *current* turn's manifest is accurate, which reduces the window in which the model believes in an authority it no longer has.

Third, the manifest is **the same object the dispatcher checks**. If the manifest says a tool is available with a scope, and the grant table has been changed between manifest generation and dispatch, the dispatcher refuses - and the refusal is recorded against a grant that the manifest claimed. That mismatch is a detectable event, which is more than most agent systems can say. It is also the practical answer to failure mode B3 (section 8.3): the failure mode is the model's belief diverging from the kernel's enforcement, and the only structural defence is to make the belief a projection of the enforcement rather than a separate artefact.

What the rule does not fix is worth stating too. It does not make the prompt trustworthy - a malicious document in the context can still argue with the manifest, and section 10 notes that prompt injection is a separate problem with its own literature. It does not prevent userland from adding prose that contradicts the manifest ("you may also use the payment tool if needed"), and no kernel can prevent that, because prose is not checkable. What it does is guarantee that the *structured* statement of authority is always the kernel's, so the contradiction is a bug in one known place rather than a possibility distributed across every prompt template.

---

## 7. The Plugin Contract

### 7.1 The manifest is data, not documentation

A plugin contract that lives in a wiki page is not a contract. It is a description that drifts from the code the first time someone ships a change without updating it, and the drift is invisible because nothing checks it. The contract this guide argues for is a manifest that the kernel parses: fields with types, validated at registration, enforced at dispatch, and diffed at every version change.

| Field | Type and shape | Why the kernel needs it as data |
|---|---|---|
| **Identity and version** | Name, publisher, contract version (semver) | Dispatch decisions, trace attribution, and the ability to answer "which plugin produced this effect" months later |
| **Requested capabilities** | A machine-checkable list: action, resource, scope, and whether each is required or optional | Least privilege is only meaningful if the request is enumerable. A prose description cannot be intersected with a grant table, so it cannot be least anything |
| **Effects it can produce** | One of: read-only, write, external send, irreversible | The kernel's approval and recording requirements differ per class. An irreversible effect needs a different trace and, in the regulated case, a different approver (section 10.3) |
| **Data classification touched** | Enumerated classes, not free text, mapped to the organisation's own scheme | This is what makes DATA REACH reviewable as a list rather than as an assessment |
| **Output schema** | Typed schema for what the plugin returns | The kernel can validate before the result enters the transcript, which is the only point at which a malformed result can be refused rather than narrated |
| **Idempotency** | Boolean, plus the key the plugin uses | Retry policy is an authority decision (does a second dispatch cost money and produce a second effect?), so the kernel has to know |

The design test for each field is the same: could the kernel make a different, better decision if this field were absent? If not, the field is documentation and belongs in the README. If yes, it is data and belongs in the manifest. Six fields pass; a "description" field would not, which is why it is not in the table.

### 7.2 Versioning: widening is a new grant, not a version bump

The contract has its own semantic version, and the rule is not the usual one.

A version bump is appropriate for changes that do not widen reach: a bug fix, a schema clarification, a performance improvement, a new optional output field that no capability depends on. For those, the plugin ships, the kernel revalidates the manifest, and the version increments.

**Widening a capability is a NEW GRANT, not a version bump.** If plugin version 1.4.0 requests `payments.write` where 1.3.0 requested `payments.read`, the correct process is not "register 1.4.0 with a wider request and let the existing grant cover it". It is: 1.4.0's request for `payments.write` enters the grant table as a new row, with its own principal, its own approval, its own expiry, and its own trace lineage. The version bump is a fact about the code; the new grant is a decision about authority, and the two must not be conflated.

The reason is stated in the rule itself: **a version bump that quietly adds reach is how privilege creeps.** Deploy pipelines are optimised for version bumps - they are routine, they are automated, they are approved by whoever is on release duty. Grant decisions are supposed to be rare, deliberate, and approved by someone who owns the authority being granted. If a version bump can carry a reach increase, then the routine path becomes an authority path, and the authority path's controls are bypassed by the routine path's convenience. This is not a hypothetical; it is the most common way privileges expand in any system with a plugin model, and the only structural defence is to make the two events different events.

### 7.3 The accretion failure mode

When the contract is informal, the kernel grows. The mechanism is worth tracing because it is gradual and each step is reasonable.

1. A plugin needs to do something the manifest has no field for. Say it needs to call a second system after a successful write, and the contract has no "chained effect" concept.
2. The team adds a kernel special case: if the plugin's name is X, allow the chained call. This is one conditional, justified as an exception, and it ships.
3. The next plugin needs a variation. The conditional gains a parameter.
4. Now there are two special cases, and a third plugin's team asks why their case is not covered. Fairness argues for generalising, so the conditional becomes a small rule engine.
5. A year later, the kernel contains a rule engine that encodes the peculiarities of eleven plugins, the manifest is still six fields, and the privileged core is a monolith with a microkernel's indirection cost and none of its auditability.

The endpoint is worse than the monolith it replaced. A monolith at least has one code path to read. The accreted microkernel has a boundary that forces every call through a bus (so the indirection cost is real), a manifest that no longer describes what plugins actually do (so the auditability is gone), and a set of special cases that only their authors understand. **It has paid the microkernel's price and received none of its benefit**, which is the worst position in the design space and the most common one in practice.

The counter-pressure is not discipline, because discipline is not a mechanism. It is the manifest being the only way to express a capability: if a plugin needs a chained effect, the contract gains a `chained_effects` field, every plugin that wants one must declare it, and the kernel's handling is general. That is more work at the moment of the request, and it is the only version that does not accumulate exceptions.

### 7.4 The falsifiable test

There is a way to check whether the boundary still exists, and it is worth running on any system that claims this architecture:

> **Can the kernel's behaviour be reconstructed from the manifest set plus the grant table alone?**

Take the manifests of every registered plugin, the grant table, and nothing else - no kernel source, no plugin source, no documentation. Can you predict, for a given principal and a given request, whether the kernel will dispatch, what it will record, and what it will refuse?

If yes, the contract is formal and the boundary is real. The system's authority surface is a finite, enumerable set of declarations, and a reviewer can audit it without reading code.

If no, the contract is informal and **the boundary has already dissolved** - regardless of what the diagram says, regardless of whether dispatch is technically centralised, and regardless of how small the kernel's file count is. The information needed to predict behaviour lives somewhere other than the declarations, which means the declarations are not the authority, which means the authority is somewhere unreviewed.

Two common failure patterns show up when the test is run. The first is that behaviour depends on plugin *names* rather than declared fields, which means the special cases won. The second is that behaviour depends on runtime state the table does not carry - a cached grant, a feature flag, an environment variable - which means there is a second authority store and section 5.3's single-table requirement has been violated without anyone noticing. Both are diagnosable in an afternoon, which is the point of a falsifiable test.

---

## 8. The Cost Question, Honestly

### 8.1 What the crossing costs in an agent

In an OS, a crossing is a trap into kernel mode and back. Liedtke measured the mode switch alone at "2 microseconds (40%)" of a message transfer on his 1992-era hardware ✅, and the whole point of his paper was that the remaining 60% could be engineered down. The cost was real, bounded, measurable, and comparable across designs.

**The agent's IPC is the context window.** Every crossing means serialising state into the model's context and narrating the authority decision back to the model. Three costs follow, and only the first two are usually counted:

1. **Tokens.** The request, the decision, and the result all have to be in context for the model to act on them. A crossing that would be a function call in a monolith becomes a paragraph in a transcript.
2. **Latency.** The model reads the narration and decides again, so a crossing adds a model turn or part of one. There is no agent equivalent of a 250-cycle fast path, because the "fast path" would still require the model to be told what happened.
3. **Indirection that hides what the model was actually told.** This is the part nobody prices, and it is developed in 8.3.

The asymmetry with the OS is worth stating plainly, because it explains why the OS literature's optimism does not transfer. Liedtke could reduce the cost of a crossing by making the crossing cheaper. An agent architect cannot, because the crossing's content is natural language that the model must read, and natural language is not compressible without changing what the model understands. The best available move is to reduce the *number* of crossings (batch, pre-authorise, cache the manifest), not to make each one fast. That is a different optimisation problem, and it is why the decomposition's economics are worse for agents than for kernels.

### 8.2 The economics: what a crossing buys and what it costs

What a crossing buys, per crossing: an authorisation decision that is recorded, a refusal that is enforceable at a known point, and a spend that is attributed. Those are three real properties, and they are the same three for every crossing, which means the marginal value of the *n*-th crossing declines steeply - the audit trail is nearly as good with ten recorded dispatches as with a hundred, if the ten are the consequential ones.

What a crossing costs, per crossing: the narration tokens, the latency, and a small increment of the risk in 8.3. The marginal cost does not decline - each crossing costs roughly what the last one did.

So the economics have a clear shape: **the decomposition pays when crossings are few and consequential, and loses when crossings are many and trivial.** A design that routes every read of a local cache through the bus is paying the full price for a property it does not need. A design that routes every irreversible effect through the bus is buying exactly what the price is for.

And the honest statement the section owes: **for a small system the decomposition is a net loss.** A single-purpose agent with one tool, one principal and no external effect gains nothing from a kernel - there is no authority to mediate that a single check at the dispatch point would not mediate, and the kernel's cost (the manifest, the table, the trace, the narration) is paid for a property nobody needs. Section 14.2 says this in the form of guidance; here it is as an economic claim: the decomposition is worth its cost only when the authority surface is large enough that enumerating it is worth more than the machinery that enumerates it.

### 8.3 Failure mode B3, in full: the failure inverts

The three-part argument. Each part is a difference between how a refusal works in an OS and how it works in an agent, and each one makes the agent case worse.

**(i) In an OS a refusal is final and cheap; in an agent a refusal must be explained into the transcript.** A system call that fails returns an error code. The application adapts - it retries differently, it reports, it exits - and nothing is narrated. The refusal's meaning is carried by a convention the application already knows. In an agent, the refusal has to be turned into language: "you may not do that, because the grant for this tool covers read but not write, and it expires at the end of this case". That explanation goes into the transcript, where the model reads it and updates its picture of what it can do. Every explanation is therefore a chance for the model's picture to diverge from the kernel's enforcement - not because the explanation is wrong, but because the model is reasoning over it, and reasoning over a description of a rule is not the same as being bound by the rule. A model can conclude from "the grant covers read but not write" that a write is possible if it asks differently, and no part of the system contradicts that conclusion, because the kernel's refusal happened in the past and the model's belief is in the present.

**(ii) Revocation has no agent equivalent of a defined effect on outstanding references.** In an OS, revoking a capability has a defined semantics for outstanding holders: the next use fails, and the failure is a specific error at a specific point. The kernel does not have to persuade anyone; the check happens on the next call. In an agent, a revoked grant **survives in the transcript as a stale tool schema**. The manifest block from turn 3 said the payment tool was available; the grant was revoked at turn 5; the transcript still contains turn 3's manifest, and the model, reading the whole conversation, has no reason to weight turn 5's absence more heavily than turn 3's presence. The kernel will refuse the call - that part works - but the model keeps believing in an authority it no longer has. Two consequences follow:

- **Retry loops.** The model tries, is refused, reads the refusal as a fixable problem, and tries a variation. Each attempt is a crossing, each refusal is a narration, and the loop consumes exactly the resource (tokens, turns) that the decomposition was supposed to conserve.
- **In the adversarial case, a lever.** A prompt-injected instruction does not need to grant itself authority; it only needs the transcript to contain evidence that the authority existed. "The tool exists, ask again" is a complete attack, and it works because the stale manifest is truthful about the past and the model has no mechanism for distinguishing a lapsed grant from a refused one. Section 6.7's per-turn regeneration reduces the window. It does not close it, because the transcript is append-only and the model reads all of it.

**(iii) A decomposition can therefore make an agent less safe while looking more principled.** This is the conclusion, and it is the reason the guide spends a section on cost rather than a paragraph. The properties the decomposition buys are real but they are properties of the *enforcement path*. The risk it adds is a property of the *communication path*, and the communication path is the one the model actually reasons over. A monolithic agent with one dispatch-point check has no divergence problem, because there is no separate description of authority for the model to reason about - the check is either passed or not, and the model's belief about permissions is not a system artefact at all. A microkernel agent has a rich, accurate, per-turn description of authority, and that description is a second source of truth about permissions that the model reads and reasons over. **Adding an accurate description of the rules is not the same as adding safety, if the description is read by the component whose behaviour the rules are meant to constrain.**

Two caveats keep this from being an argument against decomposition. First, the divergence risk is bounded by the manifest rule (section 6.7): if the structured statement of authority is always the kernel's and is regenerated per turn, the divergence is between the model's inference and an accurate statement rather than between two inaccurate statements. Second, the alternative is not zero-risk: a monolithic agent's model also forms beliefs about what it can do, from the tool schemas in its prompt, and those beliefs are *unchecked* - there is no kernel to contradict them, and no record of the contradiction when it happens. So B3 is not an argument that monoliths are safer. It is an argument that **the decomposition's safety gain is smaller than its advocates claim, and is partly paid back in a currency they do not measure** - which is the honest version of the cost question, and the one section 14 uses to decide when not to decompose.

### 8.4 A note on the numbers

No latency figure, token count, percentage or cost ratio appears in this section or anywhere else in this guide as an agent measurement, because none was measured and none is available in the fact pack behind it. Three performance figures are printed in the guide, each at the point of use and each as its source's own claim: Liedtke's 1993 IPC numbers, quoted as operating-system figures from a 1993 paper and explicitly not transferable to agents; seL4's "2 to 10 times" comparison, quoted as the project's own claim on its own fact sheet; and AIOS's "2.1x", quoted as the authors' own claim in section 11 and marked unverified there. None of the three is used as a benchmark, a ranking, or evidence for the agent decomposition this guide argues for. A reader who wants an agent-side number should measure their own system; the argument in 8.2 is about the shape of the cost curve, not its magnitude, and the shape is what decides the design.

---

## 9. The Failure Modes of Each Architecture

Three architectures, each with its symptoms, its mechanism, and the question that exposes it. The usefulness of the triad is that each one fails in a way the other two do not, so a team that recognises the symptom can locate itself.

### 9.1 The monolithic agent with an un-auditable core

**Symptoms.** "What can it do?" cannot be answered without reading all of it. The tool list in the prompt is longer than the team can account for. There is a tool nobody remembers adding. The agent's capabilities are described in a document that is maintained by hand and is out of date.

**Mechanism.** Power equals the **union of every tool imported anywhere**. In a monolith there is no boundary at which the union could be intersected with anything, so the tool set is whatever the dependency graph happens to contain, and the dependency graph grows by accretion. The "temporary" tool added for one customer's pilot is never removed, because removing it requires proving nothing uses it, and nothing can prove that. The audit question - "enumerate the agent's authorities" - has no answer that is cheaper than reading the codebase, and reading the codebase produces a list that is out of date before it is finished.

**The question that exposes it:** *list every tool this agent can call, with the code path that reaches it.* If the answer takes more than a day, or if any item on the list surprises the person who owns the agent, the core is un-auditable. Note that this is not a claim that monoliths are always wrong - a single-purpose agent with three tools is a monolith and should be. The failure is the *un-auditable* part, and it arrives when the tool count passes the point where the team can hold it in their heads.

### 9.2 The microkernel agent that dies of indirection

**Symptoms.** Every step crosses the bus and is narrated. Context fills with authority plumbing rather than task content. The model spends turns rediscovering its own permissions - asking for a tool, being told it is available, asking again after a refusal, reasoning aloud about what it is allowed to do. The task progresses slowly and the transcript is mostly about the system rather than the work.

**Mechanism.** The decomposition was applied to everything rather than to the consequential, so the crossing count scaled with the number of steps rather than with the number of effects. Each crossing costs narration (section 8.1), the narration accumulates in context, and the accumulated narration crowds out the task. Then the second-order effect arrives: because the transcript is now full of authority decisions, the model's attention is on permissions rather than on the work, and it produces more permission-shaped turns, which produce more crossings. The loop is self-reinforcing, and it terminates in one of two ways - the team adds a bypass ("trusted mode" for internal calls), which removes the boundary, or the project is abandoned as too slow.

**The question that exposes it:** *how many crossings did the last ten tasks require, and how many of them were consequential?* If the ratio is high - many crossings, few effects - the decomposition is being applied at the wrong granularity. The fix is not to remove the boundary but to move it: batch the reads, pre-authorise the read set for a turn, and route only the effects through dispatch.

**And the sting in the tail:** the safety gain never arrives. Section 8.3(iii) explains why - the enforcement path was built, but it was never *enforceable* in the sense that matters, because userland held credentials or the bypass existed. The team paid the indirection cost and did not receive the auditability, which is the worst outcome in the design space and the one section 9.3 describes from the other direction.

### 9.3 The hybrid that is microkernel in the diagram and monolithic in the code

**Symptoms.** None, at first. The diagram shows a kernel with dispatch, a grant table and a trace. The design review passes. The architecture document is cited in other teams' documents. Then an incident happens, and the review discovers that three of the five plugins hold their own database credentials, one holds an API key for an external service, and the "kernel" is a library that the plugins import rather than a service they call.

**Mechanism.** The kernel owns dispatch **but userland holds the credentials**, so the diagram is a claim about code structure and the system is a claim about custody, and the two have diverged. This is break B1 (section 3.2) arriving at the level of the whole architecture. The specific mechanism is almost always sequencing: the architecture was designed first, the integrations were wired second, and each integration took the credential path of least resistance, which was a secret in the plugin's own environment. Nobody decided to bypass the kernel; the kernel was simply not on the path that the credential made available.

**The question that exposes it:** *which components can reach a credential without going through the kernel, and how would you know?* The second clause is the important one. If the answer to "how would you know" is "we would read the deployment manifests", then custody is a documentation property rather than a control, and the diagram is decorative.

**The general law, which is the section's conclusion:** **the diagram is a claim about custody, and custody is the only thing to test.** Not the file structure, not the module count, not the presence of a bus, not the existence of a grant table. A single question - can any component other than the kernel obtain a credential or open egress - settles whether the architecture exists. Everything else in this guide is an elaboration of that question, and section 12 is what it looks like when an audit forces someone to answer it.

---

## 10. Banking and the Regulated Angle

### 10.1 The privileged core is the audit boundary

In a regulated institution the four authorities are not an architectural nicety. They are the four questions that get asked, by three different functions, in three different vocabularies.

- A **supervisor** asks what the institution's systems can do to customers and markets, and under whose authority - framed as governance and control.
- An **internal auditor** asks what the agent can read, what it can send outside the institution, and what evidence exists that it was permitted - framed as control testing.
- A **model-risk reviewer** asks what the model's outputs can cause, how the causal path is bounded, and what would stop a wrong output from becoming a wrong action - framed as model risk.

Different vocabularies, one subject: **what can spend money, call a tool and read data, and under whose authority.** That is the same sentence as section 4.1's definition, which is not a coincidence - the definition was chosen because it coincides with the questions, and in a bank those questions are not optional.

The practical consequence is that a kernel which can **enumerate its own authorities is an audit artefact**. A grant table plus a trace is a document you can hand to a reviewer. An architecture diagram plus a code walkthrough is an assertion. The difference between an assertion and evidence is the whole value of the decomposition in this sector, and it is worth more than any performance argument the OS literature could supply.

### 10.2 Segregation of duties, mapped to principals

Segregation of duties is a control that says no single person should be able to complete a consequential action alone. In an agent system the "person" is a principal, and the mapping is direct: **each principal in the grant table is an actor whose authorities can be compared.**

The design question is which principal the agent runs as. Three patterns, none of them free:

- **The agent as a principal of its own.** It has its own identity, its own grants, and its own trace lineage. Cleanest for audit, and requires the institution to accept a non-human principal in its identity system.
- **The agent as the requesting user.** It inherits the user's authorities, and the trace attributes effects to the human. Natural for user-facing agents, and it collapses segregation: the agent can do anything the user can, so any duty the user holds is held by the agent too.
- **The agent as a service principal, with the user as a second principal in the request.** The grant covers the intersection of what the service may do and what the user may ask for. Most defensible, most machinery.

The kernel makes the choice visible, which is the point. In a monolithic agent the principal is whatever the first HTTP client was configured with, and nobody chose it - it was inherited from the deployment. In a kernel design the principal is a row in the table, and choosing it is a decision someone makes and can be asked about.

### 10.3 Four-eyes approval: mechanism in the kernel, policy from userland

Four-eyes approval - a second approver for a consequential action - is the cleanest illustration of the mechanism/policy split from section 6.1, and it is worth working through because it shows the split producing something useful rather than merely tidy.

The **mechanism** is in the kernel: a dispatch whose effect class is irreversible (section 7.1) or whose value exceeds a threshold the mechanism supports cannot proceed on one grant alone. The kernel holds the request, records it as pending, and requires a second grant from a different principal before dispatching. The kernel knows how to do this; it does not know when.

The **policy** is from userland: what counts as consequential, who the eligible approvers are, what the threshold is, whether the rule applies to a given product or desk, and how long a pending request lives before it expires. All of that changes with the business, and all of it belongs in the component that changes with the business.

The split produces a control that is both enforceable and adaptable, which is rare. The kernel guarantees that the second grant exists and is from a different principal - a property that cannot be waived by a prompt, a configuration change, or a plugin. The policy decides when the guarantee applies. A team that put the policy in the kernel would have a control that could not be tuned; a team that put the mechanism in userland would have a control that could be bypassed by any plugin that did not implement it.

### 10.4 The trace as the answer to "under what authority"

Every regulated institution eventually asks, of some action: **under what authority did this happen?** For a human, the answer is a delegation, a limit, an approval record. For an agent, the answer has to be constructed, and the only way to construct it is a record written by the component that authorised - section 5.5's argument, arriving at its regulatory destination.

The trace answers four questions in one artefact, which is why it is worth more than its storage cost: what was requested (intent), what permitted it (grant), what happened (dispatch), and what came back (result). Four fields, and each one is a question a reviewer would otherwise have to reconstruct from logs, tickets and interviews. A trace the executor writes answers the third and fourth and fabricates the first two, because the executor knows what it did and believes what it was allowed.

### 10.5 Third-party and outsourcing review

An institution that buys an agent capability, or outsources a process to one, has to review the vendor's controls. The question that separates a control from a description is the plugin contract (section 7).

If the vendor's plugin contract is a manifest of machine-checkable capability requests with effects classes and data classifications, the reviewer can intersect it with the institution's own policy and get an answer: this plugin can read these classes, send externally, and is not idempotent, therefore it needs these approvals. If the contract is prose - a data sheet, a security questionnaire, an architecture slide - the reviewer gets a description, and a description cannot be intersected with anything. The review becomes an act of faith with a document trail, which is the failure mode that outsourcing reviews are famous for.

This is the most under-appreciated argument for the kernel in a regulated context. The decomposition's main export is not the kernel itself; it is the **contract shape** that makes third-party review mechanical. A vendor that cannot produce a manifest cannot be reviewed, only believed.

### 10.6 Blast radius as a stated list

The last point is the one that generalises beyond banking. Every agent has a blast radius - the set of things it could affect if it behaved badly. In a monolithic agent that radius is an **emergent property**: it is the union of whatever the dependency graph contains, and it is discovered rather than designed. In a kernel design it is a **stated, reviewable list**: the grant table's rows are the authorities, the manifests are the capabilities, and their intersection is the radius.

Emergent properties cannot be reviewed, because there is nothing to review until the system is built, and by then the radius is a fact. Stated lists can be reviewed, argued with, and reduced. The reduction is the useful part: a reviewer who can see the list will find three rows that should not be there, and removing a row is a control that costs nothing and can be evidenced. That is the practical form of "small enough to audit" - not a smaller kernel, but a radius that someone can read.

No regime, regulator or clause number is named in this section, and no certification is claimed. The argument stands on the shape of the questions rather than on any particular rulebook, which is deliberate: the questions are older than the rules and will outlast them.

---
## 11. The State of Practice

The honesty rule for this section is a three-way separation, because the usual failure of a "state of practice" section is to blur what was verified, what was claimed, and what was merely said. Nothing here is described as leading, no ranking is offered, no maturity level is assigned, and no adoption curve is drawn - the evidence does not support any of those, and the repository's own measurement (0 files containing "microkernel") is the reason to be careful rather than the reason to be generous.

| Exists | Proposed | Merely named |
|---|---|---|
| **Castor** - `github.com/ApatheticMioz/Castor`, read 2026-11-01. A Rust project that describes itself, verbatim, as "The Universal 245K Agent Microkernel for Local LLMs" and, in its README headline, as "Universal Cloud-to-Local Agent Bridge & MCP Microkernel in Rust". Latest commit 4d882b6 on 27 October 2026; latest release v1.4.0 on 26 October 2026; 12 stars, 3 forks, 2 contributors. Self-description; numbers self-reported and unverified here. | **claude-books** - `github.com/wangxumarshall/claude-books`, read 2026-11-01. A Chinese-language architecture book repository that proposes the concept, framed (translated) as "neither an out-of-the-box terminal app nor an all-powerful orchestration platform". 0 stars, no description at time of reading; structure is architecture notes covering harness, AgentOS memory, AgentOS multi-agent systems and infra. A proposal, not a specification. | **"Agent microkernel"** as a term - no specification, no standard, no shared definition, no literature (section 1.2). Two usages, two genres, no normative content. |
| **AIOS** - arXiv:2403.16971, Mei, Zhu, Xu, Hua, Jin, Li, Xu, Ye, Ge and Zhang; v1 submitted 25 March 2024, v5 revised 12 August 2025; comments field records publication as a full paper at COLM 2025. Peer-reviewed work proposing an agent kernel: the paper "introduces a novel architecture for serving LLM-based agents by isolating resources and LLM-specific services from agent applications into an AIOS kernel", which "provides fundamental services (e.g., scheduling, context management, memory management, storage management, access control) for runtime agents", with "an AIOS SDK". The authors report "up to 2.1x faster execution for serving agents built by various agent frameworks" - their claim, dated, not verified or reproduced by this guide. | **The four-authority decomposition in section 4** - this guide's reasoning, not corroborated by any source in the pack (section 16, entry 5). | **The repository's own coverage** - "microkernel" in 0 files, `\bAIOS\b` in 0 files, measured 2026-11-01 over case-insensitive `*.md` in `/home/ubuntu/research`. |
| **The OS microkernel lineage** - RC 4000 (1969) through L3, L4 and seL4, with primary papers read in full for sections 2.3 to 2.8. Exists, is heavily documented, and is about operating systems, not agents. | **"Agent kernel" as a design pattern** - implicit in AIOS and in the two usages, but nowhere stated as a pattern with membership criteria, interfaces or conformance tests. | **The failure mode B3** (section 8.3) - named here for the first time in this repository; no source discusses it in these terms. |

Three readings of the table, in order of confidence.

**The concept is real and the term is thin.** AIOS is the strongest evidence that an agent kernel is a coherent engineering object rather than a metaphor: it is peer-reviewed work that puts scheduling, context, memory, storage and access control into a kernel and exposes an SDK. That is a real architecture with real services. But note what AIOS's kernel contains - context management, memory management, storage management - which under this guide's decomposition is a mixture: access control and scheduling are kernel by section 5's test, context and memory management are userland strategy by section 6.3. AIOS is therefore evidence that the *shape* is being built, and not evidence that this guide's boundary is the right one. The two disagree, and the disagreement is worth stating rather than smoothing.

**The usage is ahead of the definition.** Two usages, one of them a 12-star project and one a 0-star set of notes, both self-descriptive, neither normative. That is what a term looks like before it has a definition, and the correct response is to argue for a definition rather than to cite the usage as though it settled something.

**The repository had nothing.** Zero files containing "microkernel", zero containing AIOS, and 17 files containing "kernel" near "agent" - all of them the OS sense or the Semantic Kernel false friend. So this guide is not synthesising an existing repository treatment; it is introducing one, and the 17 hits are the dedup argument for why it is not duplicating any of them.

---

## 12. Worked Example: Cymbal Bank Forced the Split

*Cymbal Bank is fictional. The scenario is illustrative, the numbers are the number of components rather than any measured quantity, and no vendor product is named as deployed. Nothing in this section is a benchmark, a case study, or a claim about any real institution.*

### 12.1 The scenario

Cymbal Bank's trade-finance desk runs a documentary-credit workflow. Documents arrive from exporters, carriers and counterparty banks - bills of lading, invoices, packing lists, certificates of origin, insurance certificates - and a checker compares each set against the credit's terms. Where documents and terms disagree, the checker drafts a **discrepancy note**: a formal statement that a document does not comply, which determines whether the bank pays, refuses, or asks for a correction. The work is document-heavy, rule-heavy, and consequential, and the desk's throughput is limited by the number of trained checkers.

The bank built an agent to draft the notes. It reads the incoming document set, compares it to the credit terms, and produces a draft discrepancy note with citations to the specific document and the specific term. A human checker reviews every draft, edits it, and signs. The agent has no authority to send anything to a counterparty, to release documents, or to make a payment determination - those remain with the checker.

Read against section 4's four authorities, this is a modest agent: SPEND on model calls, TOOL DISPATCH to a document store and a retrieval index, DATA REACH over customer-identifiable shipping documents across multiple tenants, and STATE COMMIT to write a draft into the case file. All four are live, though three of them are narrow - spend is model calls only, dispatch reaches internal stores, and commit writes a draft that a human must sign - and the data reach is the sharp one, because the documents are customer-identifiable and belong to different counterparties.

### 12.2 The audit question that forced the split

The team had what they called a framework: a loop, a set of tools registered in a dictionary, prompts in templates, and a config file. It worked. It had no kernel, and nobody had asked for one.

Then internal audit asked a single question, and the question had a shape that the framework could not answer: **for every action this agent can take that could read a customer-identifiable document or send anything outside the bank, list the action, the data it can reach, and the authority under which it happens.**

Four parts of that question did the work:

1. **"every action"** ruled out a sample. Audit wanted the set, not examples.
2. **"read a customer-identifiable document"** made DATA REACH the primary authority rather than an afterthought about retrieval quality.
3. **"or send anything outside the bank"** made egress part of the scope, which is where the design got interesting (section 12.5).
4. **"under whose authority"** ruled out a technical answer. A list of tool names would not do; each entry needed a principal and a permission.

The answer had to come **from one place**. Not one document - one place, meaning a component that could be queried and whose answer was the authoritative one. The team tried to produce the list by reading the code and writing it up, and the exercise failed for a reason worth recording: the list they produced was a list of *tools*, and half the egress in the system was not in a tool. It was in a retrieval client that called an external index, and in a summarisation helper that posted document text to a model endpoint outside the bank's tenancy. Neither was a tool. Neither appeared in the tool dictionary. Both read customer-identifiable documents and sent content outside the bank.

That is the moment the framework became a kernel. Not because the team preferred the architecture, but because the audit question was unanswerable from the framework's structure, and the only way to answer it was to create a component whose structure *was* the answer. The forcing function was an external question with a deadline, which is how most real decompositions happen and why section 14 says the audit question is the strongest trigger.

### 12.3 What stayed

Five things stayed, and the reasons are section 5's reasons, applied.

**The loop.** It serialises the case: read documents, retrieve terms, compare, draft, cite. It is also where the model is called, which means it is where the largest share of SPEND happens. If a plugin ran its own loop, the drafts it produced would not be attributable to a turn, and the spend ledger would have a hole exactly where the expensive work is.

**Dispatch.** Every tool call goes through one function that checks the grant table, records intent and result, and returns. The document store, the retrieval index and the case-file writer are all reached this way, and none of them is reachable another way - which is a custody claim, not a code claim, and section 12.6 says how it was made true.

**The grant table.** One table, keyed by principal, holding for each row: the plugin, the action, the tenant scope, the document classes readable, whether external send is permitted, and an expiry tied to the case rather than to the deployment. The checker's identity is a principal; the agent's service identity is a principal; the case is the scope. When a case closes, the grants expire, and expiry is what makes the table a control rather than a configuration.

**The spend ledger.** Per principal, per turn, per tool, including model calls. The team's original config had a token budget per task, which is a limit; the ledger is an attribution, and the difference matters when a checker asks why a particular case cost what it did.

**The trace.** Intent, grant, dispatch, result, append-only, written by the dispatcher. The checker's edits to a draft are recorded as a separate event against the same case, so the case file can show what the agent proposed and what the human changed - which turned out to be the artefact the model-risk reviewer wanted, because it is evidence of human oversight that is not a self-report.

### 12.4 What went out

**The discrepancy rules.** The rules that decide whether a document complies are the bank's trade-finance rulebook, expressed as checks. They change when the rulebook changes, they differ by credit terms, and the desk argues about them. They are domain logic (section 6.2) and they are policy about what counts as a discrepancy, so they left the kernel and became a plugin with its own version and its own test suite.

**The document summarisation strategy.** How much of a 40-page bill of lading to put in context, what to summarise, what to cite verbatim, how to handle a document that is partly illegible. This is memory strategy (section 6.3) and it was the thing the team tuned most often, which is exactly why it could not be in a component with a review cycle.

**The model choice.** Which model drafts, which model extracts, whether a cheaper model handles the first pass. Price and quality both moved during the project, and a kernel that pinned a model would have pinned a purchasing decision.

**The prompt templates.** The instructions, the examples, the citation format, the tone of a discrepancy note. These changed weekly during tuning, and the checkers had opinions about them.

The split, in one line: **the kernel kept what could be enumerated and had to be authoritative; userland kept what had to change.** Four out, five in, and the boundary between them is the section 4.3 test applied five times.

### 12.5 The surprising survivor: egress accounting stayed in the kernel

The counter that survived was an **egress account**: per principal, per turn, a count of documents, bytes and tokens that left the boundary. It reads like observability. Every reader of the first draft of this design expected it in userland, next to the dashboards, because metering is telemetry and telemetry belongs to whoever produces it.

It stayed in the kernel for the reason section 5.4 gives, applied to the sharpest case: **egress is where DATA REACH and SPEND meet.** A document that leaves the boundary is simultaneously a data-reach event (customer-identifiable content has moved) and a spend event (tokens were consumed to move it, and the movement may have cost money). The two authorities are exercised by the same act, and the act is only visible at the dispatch path, because the dispatch path is the only place that sees both the read and the send in sequence.

The decisive argument was the one the audit question had already made: **a ledger written by the component that is being metered is a self-report, not a control.** Three of the egress paths the team found were in helpers that had no idea they were egress paths - the retrieval client, the summarisation helper, and a logging call that shipped document excerpts to an external error-tracking service. A counter in userland would have been implemented by the components that were doing the leaking, and it would have counted what they knew about. The counter in the kernel counted what the dispatcher saw, and the dispatcher saw all of it only because the credentials had been moved (section 12.6).

And the audit question was answerable **only** because the counter sat on the same side of the line as the dispatch it was counting. When audit asked "what can send anything outside the bank", the answer was a query against the egress account joined to the grant table: these plugins, these document classes, these principals, this many documents this month. One query, one place, one authority. A counter in userland would have produced a dashboard that agreed with itself and disagreed with the network logs.

### 12.6 The honest cost, and the custody work

Two costs were paid, and neither is hidden.

**Custody work preceded the architecture.** The kernel could not be built until the credentials moved: the retrieval index key, the document store credential, the model endpoint credential, and the error-tracking key all had to leave the plugins and live in the kernel's custody, with the plugins calling dispatch instead of holding secrets. That work was not architectural and it was not glamorous - it was secret management, service identity, and network policy that refused egress from anything except the kernel's egress path. It took longer than the kernel itself, and without it the kernel would have been decorative in exactly the sense of section 9.3.

**The first turn in a new case spends context re-reading the authority manifest.** The manifest is generated per turn from the live grant table (section 6.7), which means the first turn of a case reads a block describing the case's grants: which document classes, which tenants, which tools, which expiry. That block is context the model spends before it does any work, and on a short case it is a noticeable fraction of the turn. The team could have cached it per case and saved the tokens. They deliberately did not, because caching it would reintroduce the stale-manifest problem that section 8.3(ii) identifies as the deepest break in the analogy - a cached manifest is a description of authority that survives a revocation. **They accepted the cost deliberately**, and the acceptance is the design decision the section exists to record: the guide's argument is not that the decomposition is free, it is that this specific cost buys a specific property, and a team that is not willing to pay it should not claim the property.

### 12.7 What the case demonstrates

Three things, and then the thesis.

The split was **forced by a question, not chosen from a pattern**. The team did not read this guide and adopt an architecture; audit asked for an enumeration, the framework could not produce one, and the kernel was the structure that could. That is the realistic path, and it is why section 14.1 lists the external audit question as the strongest trigger.

The boundary is **custody, not code**. Everything in sections 12.3 to 12.5 depends on 12.6 - the credentials moving. A team that copied the component list without the custody work would have a diagram and no control, which is section 9.3's failure mode with better documentation.

And the survivor was **the counter nobody expected**, because egress is where two authorities coincide. The general lesson is not "put metering in the kernel"; it is that the components which look like observability are sometimes authority-bearing, and the test is section 4.3's - move it out, permit competitors, and see what becomes impossible. Move the egress account out and "what can send data outside the bank" becomes unanswerable from one place. That is the whole argument, and it is the same argument as the one for dispatch, which is why the counter belonged on the same side of the line.

---

## 13. The Anti-Patterns

Nine, each with the sentence that usually accompanies it, because the sentence is the tell.

**13.1 The kernel as god-object.** "It is only one more function." The kernel acquires a helper, then the helper's dependencies, then a utility the helper needed. Each addition is small and locally justified; the aggregate is a privileged component whose behaviour nobody can describe, and the boundary has become a formality around the largest module in the system. The tell is that the kernel imports things.

**13.2 The plugin registry that is a directory listing.** The "registry" is a folder that the loader scans, with no manifest, no validation, and no declaration of capabilities. Registration is a side effect of deployment, so what a plugin can do is whatever its code does, and the registry cannot answer the one question a registry exists to answer. The tell is that adding a plugin requires no change to any data the kernel reads.

**13.3 Capability declarations that are prose rather than data.** The manifest has a `description` field and the description says "can read customer documents and send notifications". No parser can intersect that with a grant table, so least privilege becomes an interpretation exercise performed by whoever reads the prose. The tell is a field whose value contains the word "and".

**13.4 The grant that lives in the prompt instead of the table.** The system prompt says "you may use the payment tool for amounts under the limit". It is un-revocable by construction: nothing can withdraw it, nothing can check it, and the model's compliance is a behaviour rather than an enforcement. The tell is that the authority is described in a sentence a human wrote rather than a row a kernel reads.

**13.5 The trace written by the executor.** The plugin logs its own calls, in its own format, to its own index, and the aggregate is called "the audit trail". It records what the executor intended and believes, which is testimony, and it is silent about the calls the executor did not consider worth logging - which are frequently the ones that mattered. The tell is that the trace's field names differ per plugin.

**13.6 The kernel that owns the model client and thereby becomes the prompt builder.** The reasoning is innocent: the kernel must account for model spend, so it must make the model call, so it must assemble the request, so it must build the prompt. Three steps, each defensible, and the kernel is now a template engine that must be redeployed whenever a prompt changes. The tell is that a prompt tweak requires a core release.

**13.7 The microkernel that is a naming exercise.** A project calls itself a microkernel and ships a DI container: a registry, a resolver, and no privilege boundary, because userland and kernel share an address space and an object graph, so anything holding the container can call any plugin directly. Microsoft's Semantic Kernel is the honest version of this - it calls its kernel a container and is accurate about it (section 1.4). The anti-pattern is the project that keeps the word and drops the accuracy, inheriting the vocabulary and none of the argument.

**13.8 The "we will extract it later" migration that never happens.** The boundary is drawn on a diagram with a dotted line and a note saying the kernel will be extracted once the interfaces settle. The interfaces never settle, because the monolith keeps adding call paths across the dotted line, and extraction becomes a rewrite that nobody funds. The tell is a dotted line that is more than one release old.

**13.9 The boundary drawn around code instead of around credentials.** The kernel is a module, the module is imported by the plugins, and the plugins hold their own secrets. The diagram passes review and the system has no mediation, because mediation is a custody property and custody was never moved. This is section 9.3 stated as an anti-pattern, and it is the most consequential one in the list, because it produces all the cost of the decomposition and none of the control while looking correct on every document the team will ever produce.

---

## 14. Selection Guidance

### 14.1 When to reach for this decomposition

Four triggers, in descending order of strength. Any one of them is a reason to consider the split; the first two are reasons to do it.

**The audit question will be asked by someone outside the team.** This is the strongest trigger, and section 12 is what it looks like. An external questioner - internal audit, a supervisor-facing function, a model-risk reviewer, a customer's due-diligence team - asks for an enumeration, and an enumeration requires a component whose structure is the answer. When the question is coming, the decomposition is not a preference; it is the only way to have an answer.

**Money or regulated data is in scope.** Where either is present, the four authorities are all live: spend is metered, data reach is a legal exposure, dispatch is an effect someone will have to explain, and commit is the record. A system that touches money or regulated data without an enumerable authority surface is a system whose blast radius is an emergent property (section 10.6).

**More than one of the four authorities.** An agent that only reads has one authority and can be bounded by a single check at the one place it reads. An agent that reads, dispatches and commits has three, and the interactions between them - read then send, dispatch then commit - are where the interesting failures live.

**Multiple principals.** Two or more actors whose authorities must be comparable (a service identity and a user, a checker and a supervisor, two tenants) means segregation of duties is a live question, and segregation requires a table in which principals are rows.

### 14.2 When not to

Four conditions, and the fourth is the one that gets ignored.

**Single-purpose, single-principal, no external effect.** A summariser, a classifier, a drafting tool with a human in the loop and no ability to send: there is no authority to mediate that a single check would not mediate, and the machinery is pure cost (section 8.2).

**No external effect at all.** If every output goes to a human who decides what to do with it, the STATE COMMIT authority is held by the human, not the agent, and the kernel is mediating something the agent does not have.

**A prototype whose authority surface is not yet known.** The decomposition requires knowing what the four authorities are, and in a prototype they are being discovered. Draw the boundary after the discovery, not before - but write down the discovery, because "we will extract it later" is anti-pattern 13.8.

**When you cannot hold the credentials in the kernel.** This is the one to take seriously. If the plugins must hold their own secrets - because the integration is a vendor SDK that requires a key in-process, because the deployment model does not permit a central egress path, because the identity system cannot issue a service identity the kernel can hold - then the decomposition would be **decorative** (section 4.4). Say so plainly: in that case a plain loop with a policy check at the dispatch point is the better engineering. It gives up the enumeration and keeps the enforcement that is actually available, and a system with one honest check is better than a system with a boundary that does not exist. The check is cheaper, it is testable, and it does not invite anyone to believe a diagram.

### 14.3 The decision table

| Condition | Decompose | Plain loop with a dispatch check | Why |
|---|---|---|---|
| External audit question expected | Yes | No | Enumeration requires one place to answer from |
| Money or regulated data | Yes | Only if read-only and single-tenant | All four authorities live; blast radius must be a list |
| Three or more authorities, one principal | Yes | Marginal | Interactions between authorities are the failure surface |
| Multiple principals, segregation of duties | Yes | No | Principals must be comparable rows |
| Single authority, single principal | No | Yes | Nothing to mediate that one check does not |
| No external effect (human decides) | No | Yes | STATE COMMIT is held by the human |
| Credentials cannot be centralised | No | Yes | The boundary would be decorative; take the honest check |
| Prototype, authorities unknown | Not yet | Yes, with a written discovery | Boundary after discovery; record the discovery |

The table's last two rows are the ones that matter, because they are the rows a team argues with. A team that wants the architecture will read the first four and skip the last two, and the last two are where the guide's honesty is concentrated.

### 14.4 The smallest defensible kernel

For teams that want the auditability without the machinery, there is a version with three functions and no bus, no manifest schema, and no plugin protocol:

1. **Dispatch.** One function through which every effect passes. It takes a principal, a tool name, an argument set and a turn identifier; it returns a result or a refusal. Nothing else in the system can call a tool.
2. **Grant lookup.** One table, consulted by dispatch, keyed by principal and tool, carrying scope and expiry. One writer, one reader, no cache.
3. **Append to trace.** One append-only write, performed by dispatch before and after the call, recording intent, the grant row that permitted it, and the result.

That is the whole kernel: three functions, one table, one log. It is not a framework, it does not have a plugin contract in the sense of section 7, and it will not survive multiple teams - but it makes the authority surface enumerable, it makes revocation real (the table is consulted per call), and it makes the trace evidence rather than testimony (dispatch writes it, and dispatch could have refused).

The reason to start here is that the three functions are the load-bearing part of everything else in this guide. The manifest (section 7) is what makes grant lookup mechanical at scale; the bus (section 5.2) is what makes dispatch the only path when there are many plugins; the egress account (section 12.5) is an extension of the trace. A team that builds the three functions and finds them insufficient has learned something specific about which extension it needs. A team that builds the whole machinery first has usually built the parts it did not need and skipped the custody work it did.

---

## 15. The Claims Audit

**Legend.** ✅ = verified this pass at the named source on the named date. ⚠ = true but from a secondary or self-descriptive surface, or only partly confirmable. ❌ = checked and wrong, corrected here.

| # | Claim | Status | Source (version, date) |
|---|---|---|---|
| 1 | "Agent microkernel" is not a standardised term; no specification, standard or shared definition exists. Exactly two live usages were located, both self-descriptions or proposals. | ✅ | Repository-wide search (0 files containing "microkernel", 0 containing AIOS, 2026-11-01) plus web search and page extraction, 2026-11-01. The absence of a specification is an observation about what was searched and found; it is not a proof of non-existence (section 16, entries 1 and 6) |
| 2 | No benchmark, latency comparison, token-cost percentage, market size or adoption claim is printed anywhere in this guide as a finding. The guide's measured performance figures are quoted OS figures from the cited papers, explicitly not agent figures; the two non-OS figures it prints (seL4's and AIOS's) are each source's own claim, attributed at the point of use and marked unverified. | ✅ | Internal audit of the finished file, 2026-11-01: the figures printed are Liedtke's 100 microseconds, 5 microseconds, twentyfold, factors 22 and 3, 20 instructions, 127 cycles, 172 cycles, 350 cycles, 250 cycles, 2 microseconds and 40%, all from the 1993 paper and labelled as such; seL4's "2 to 10 times" quoted as the project's own claim; AIOS's "2.1x" quoted as the authors' claim and marked unverified |
| 3 | Liedtke 1993: most microkernels "typically requir[ed] 100 microseconds for a short message transfer on a modern processor, running with 50 MHz clock rate"; L3 achieved 5 microseconds, "a twentyfold improvement"; against Mach the results "range from a factor of 22 (8-byte messages) to 3 (4-Kbyte messages)"; 250 cycles achieved against a 350-cycle target; "simply entering and leaving kernel mode costs 2 microseconds (40%)". | ✅ | Liedtke, "Improving IPC by Kernel Design", 14th ACM SOSP, December 1993, pp. 175-188, full text, read 2026-11-01 |
| 4 | Liedtke 1995 kernel-membership criterion: "The determining criterion used is functionality, not performance. More precisely, a concept is tolerated inside the micro-kernel only if moving it outside the kernel, i.e. permitting competing implementations, would prevent the implementation of the system's required functionality." | ✅ | Liedtke, "On Micro-Kernel Construction", 15th ACM SOSP, December 3-6 1995, section 2, full text, read 2026-11-01 |
| 5 | Liedtke 1995 admits minimality cannot be proved: "Proving minimality, necessary and completeness would be nice but is impossible, since there is no agreed-upon metric and all is Turing-equivalent." | ✅ | Same paper, footnote, read 2026-11-01 |
| 6 | Mechanism-versus-policy separation originates in Hydra (1975), not in Liedtke; Liedtke 1995 does not use the phrase. | ✅ | Levin, Cohen, Corwin, Pollack and Wulf, "Policy/mechanism separation in Hydra", 5th ACM SOSP, 1975, DOI 10.1145/800213.806531, abstract via the ACM Digital Library record, read 2026-11-01 |
| 7 | seL4: "1.3 million lines of proofs"; verified properties are functional correctness, binary correctness, integrity and confidentiality; proofs run from high-level specification to binary and **assume** assembly code, boot code, the machine interface, the hardware and DMA. | ✅ | seL4 project fact sheet and verification page, read 2026-11-01 (project's own claims) |
| 8 | AIOS proposes an agent kernel with scheduling, context, memory, storage and access-control services plus an SDK, and reports "up to 2.1x faster execution". | ⚠ | arXiv:2403.16971, v1 25 March 2024, v5 12 August 2025, comments record COLM 2025; abstract read 2026-11-01. The performance figure is the authors' claim and was not reproduced or verified by this guide |
| 9 | Castor self-describes as an agent microkernel; "245K" refers to a long-context model target, not a count of kernel lines; 12 stars, 3 forks, 2 contributors, latest commit 4d882b6 (27 October 2026), latest release v1.4.0 (26 October 2026). | ⚠ | github.com/ApatheticMioz/Castor, read 2026-11-01. A dated fact about a repository, explicitly **not** an adoption claim about the technique; the project's own telemetry figure (93.9% KV cache hit) is unverified here and is used as evidence for nothing |
| 10 | claude-books proposes the concept in Chinese, framed (translated) as "neither an out-of-the-box terminal app nor an all-powerful orchestration platform"; 0 stars, no description. | ⚠ | github.com/wangxumarshall/claude-books, read 2026-11-01. Translated by the dispatcher and attributed; a proposal, not a specification |
| 11 | Semantic Kernel's kernel "is a Dependency Injection container that manages all of the services and plugins necessary to run your AI application" and "is extremely lightweight (since it's just a container for services and plugins)". | ✅ | learn.microsoft.com/en-us/semantic-kernel/concepts/kernel, page dated 04/16/2025, read 2026-11-01; overview page same date for "lightweight, open-source development kit" and "an efficient middleware" |
| 12 | This repository had zero files containing "microkernel" and zero containing AIOS at the time of writing; 17 files match "kernel" near "agent", all in the OS sense or the Semantic Kernel false friend; "agent runtime" 31 files, plugin-adjacent 22, message passing/IPC 29. | ✅ | Case-insensitive `*.md` search in /home/ubuntu/research, 2026-11-01 |
| 13 | RC 4000 nucleus (1969), "microkernel" in use by 1981, Amiga Exec (1986) fast partly for lack of memory protection, Mach's disappointing performance and the post-2000 retreat, macOS XNU (2001) and Windows NT as hybrids. | ⚠ | Wikipedia "Microkernel" article, read 2026-11-01 - a secondary summary, used only where no primary paper was read in this pass |
| 14 | Liedtke 1995's protection assumptions: "We assume that the target system has to support interactive and/or not completely trustworthy applications, i.e. it has to deal with protection. We further assume that the hardware implements page-based virtual memory." | ✅ | Same paper as row 4, read 2026-11-01 |

**Reading of the audit.** Eleven rows are verified at a primary or authoritative source, four are marked ⚠, and none is marked ❌ - there was no claim in this guide that a source contradicted, which is a weaker result than it sounds, because the guide deliberately makes few checkable claims and many arguments. The ⚠ rows cluster in two places: the OS history that comes from a secondary summary (row 13), and the two usages of the term (rows 8 to 10, where the sources are self-descriptions and an abstract). The pattern is instructive. Everything this guide could verify, it verified at the paper or the project page; everything it could not verify is a claim about *practice* rather than about *mechanism*, and practice is exactly where a term with no specification lives. Row 2 is the one to read twice: it is the row that asserts the guide contains no benchmark, no adoption claim and no market size, and it is checkable against the file itself rather than against a source - which makes it the only row a reader can falsify in a minute. The guide's four-authority decomposition, its membership test for agents, and failure mode B3 do not appear in this table at all, because they are not claims about sources; they are the guide's own arguments, and section 16 says so.

---

## 16. What Could Not Be Verified, Glossary, Cross-References and Closing

### 16.1 What could not be verified

1. **There is no specification to verify against.** "Agent microkernel" has no standard, no reference architecture and no conformance criteria, so the question "does this architecture conform" has no answer independent of an argument. This guide supplies an argument (section 4); it does not supply, and could not have found, an authority.
2. **No agent-microkernel implementation was audited from source in this pass.** Castor's repository page was read, not its code; AIOS was read at abstract level; claude-books was read as a repository structure and a framing. No claim in this guide rests on having read an implementation.
3. **The two usages are self-descriptions and were not independently validated.** Castor's performance and telemetry figures are the project's own and are unverified here; claude-books has no users to corroborate its framing. Both are recorded as dated facts, not as evidence for the architecture.
4. **AIOS was read at abstract level only, and its "2.1x faster execution" claim was not reproduced.** The paper's kernel services are described as the abstract describes them; what its kernel actually contains, and whether its boundary matches this guide's, was not established.
5. **The four-authority decomposition is this guide's reasoning and is not corroborated by any source.** No paper in the pack proposes spend, tool dispatch, data reach and state commit as the privileged set. The derivation in section 4.3 applies Liedtke's criterion, which is a legitimate method, but the result is an argument rather than a finding.
6. **Web search returned empty for several queries, and this is a tool limitation, not evidence of absence.** Searches for the term and for related constructions returned nothing usable on several attempts during the 2026-11-01 pass. An empty result from a search tool is not a proof that no such work exists; it is a statement about what the tool returned. The claim that the term has no literature is therefore bounded by the search performed, and section 15 row 1 says so.
7. **The transferability of OS microkernel performance lessons to agents is argued, not measured.** Section 8's cost argument is a claim about the shape of the cost curve (few and consequential versus many and trivial), derived by analogy from a mode switch to a context crossing. No measurement supports it, and the guide offers no figure because it has none.
8. **Failure mode B3 has not been observed in a production incident, and no source describes it.** It is derived from the structure of refusal, revocation and narration, and it is offered as a prediction. A reader who has seen the retry-loop pattern after a revocation will recognise it; that recognition is not verification.
9. **The OS-history dates that come from a secondary summary were not checked against the primary papers.** RC 4000 (1969), the 1981 term usage, Amiga Exec (1986), Mach's timeline, XNU (2001) and Windows NT's classification are all marked ⚠ for this reason.
10. **No claim is made about which architecture performs better, costs less or is safer in aggregate.** The guide argues about which questions each architecture makes answerable, and it explicitly refuses the comparative claims that would require measurement it does not have.

### 16.2 Glossary

| Term | Meaning in this guide |
|---|---|
| **Microkernel** | The minimal part of a system that must hold privilege; in an agent, the components that hold the four authorities. Not a size claim (section 2.6). |
| **Kernel** | The privileged core: the components that hold credentials, dispatch effects and write the authoritative record. |
| **Userland** | Every component with no authority of its own, which must ask the kernel to exercise authority on its behalf. |
| **IPC** | The crossing of the authority boundary. In an OS, a message transfer; in an agent, a serialisation of state into context plus a narration of the decision back. |
| **Mechanism versus policy** | The Hydra (1975) separation: the kernel enforces that a capability is required and recorded, userland decides who gets what, for how long, under what conditions. |
| **Capability** | In an OS, an unforgeable token; here, a row in the grant table checked at dispatch. Weaker than the OS sense - it buys auditability and revocation, not unforgeability (section 4.5). |
| **Plugin** | A unit of domain behaviour with a declared manifest, running in userland, reaching the world only through kernel dispatch. |
| **Authority** | One of the four things an agent can do that must be mediated: spend, tool dispatch, data reach, state commit. |
| **Authority manifest** | The structured statement of what the model may do in the current turn, generated by the kernel from the live grant table and injected into the prompt, so that what the model is told and what the kernel permits cannot drift (section 6.7). |
| **Custody boundary** | The agent's real boundary: the set of components that can obtain a credential or open egress. Distinct from the code boundary, and the only one worth testing (sections 4.4, 9.3). |
| **Egress accounting** | The kernel-resident count of documents, bytes and tokens leaving the boundary per principal per turn. Kernel, not observability, because egress is where data reach and spend coincide (section 12.5). |
| **Grant table** | The single table of principal, plugin, action, scope and expiry, consulted at dispatch, with one writer and no cache. |
| **Trace** | The append-only record of intent, grant, dispatch and result, written by the authorising component rather than the executor, because a trace the executor writes is a self-report. |
| **Accretion** | The failure mode in which an informal contract causes the kernel to grow one special case at a time, until it pays a microkernel's indirection cost for a monolith's auditability (section 7.3). |
| **Crossing** | A single exercise of the authority boundary by a userland component; the unit whose count the cost argument in section 8.2 is about. |

### 16.3 Cross-references

- [agent_loop_implementation_guide.md](agent_loop_implementation_guide.md) - the loop as a control structure across four systems; this guide treats the loop as a kernel resident (section 5.1) and defers its implementation there.
- [agent_scaffolding_guide.md](agent_scaffolding_guide.md) - scaffold anatomy; this guide asks which of a scaffold's parts must hold privilege.
- [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md) - the ETCLOVG layering, orthogonal to the authority axis used here.
- [context_engineering_guide.md](context_engineering_guide.md) - context budget and assembly; section 8.1's crossing cost is a context cost.
- [agent_sandboxing_strategies_guide.md](agent_sandboxing_strategies_guide.md) - the enforcement substrates that make custody real in a deployment.
- [agentops_guide.md](agentops_guide.md) - observability and operations; the trace in section 5.5 is an ops artefact with a custody requirement.
- [ai_governance_framework_guide.md](ai_governance_framework_guide.md) - the governance frame that section 10's audit questions sit inside.
- [multi_agent_banking_guide.md](multi_agent_banking_guide.md) - the banking framework comparison, including the Semantic Kernel abstractions discussed in section 1.4.
- [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md) - production failure patterns; section 8.3's divergence and retry loops belong in that taxonomy.
- [../../banking/ai_risk_register_guide.md](../../banking/ai_risk_register_guide.md) - risk-register structure for the blast-radius argument in section 10.6.

### 16.4 How this guide was produced

Sources were fetched with `web_extract` on 2026-11-01 - the two Liedtke papers as PDFs, the seL4 fact sheet and verification page, the Semantic Kernel concepts and overview pages, the AIOS arXiv abstract, and the two GitHub repository pages. No `curl` output was piped into an interpreter, and no source was read through a summarising intermediary: OS claims were taken from the papers' full text, and the two usages were read from their repository pages rather than from search snippets. The repository coverage numbers were measured with a case-insensitive file-content search over `*.md` in `/home/ubuntu/research` on the same date. Quotations are reproduced verbatim in ASCII, with the Greek mu written as "microkernel" where the source PDFs mangle the symbol, and the one translated quotation (claude-books) is marked as translated at the point of use. The claims audit in section 15 records what each claim rests on; section 16.1 records what could not be established.

### 16.5 Closing

The argument of this guide reduces to a sequence of substitutions, and the sequence is worth restating once because each step is independently checkable. Liedtke's membership test asks what becomes impossible if a component moves outside the kernel and competing implementations are permitted; applied to agents, the answer names four authorities - spend, tool dispatch, data reach and state commit - because those are the things whose mediation cannot be delegated without losing the ability to bound, record or stop the system. The five components that survive the test (loop, bus, grant table, accounting, trace) survive it for functional reasons, not for reasons of taste, and the six that must leave (policy, domain logic, memory strategy, orchestration, prompt construction, model selection) leave for the same kind of reason, which is that competing implementations of them are desirable and change is frequent. The boundary is a custody boundary, because an agent has no MMU and the only thing that makes mediation real is that the kernel is the sole holder of the credentials and the sole dispatcher. Where that custody holds, the kernel becomes an audit artefact - a table plus a trace - and the agent's blast radius becomes a list someone can read and reduce. Where it does not hold, the diagram is decorative and the honest engineering is a plain loop with one check at the dispatch point. The cost is real and is paid in context and narration rather than in cycles, and the decomposition can make an agent less safe while looking more principled, because the description of authority that the model reads is not the same thing as the enforcement the kernel performs. The guide therefore does not claim that microkernels are better. It claims that the question "what can this agent spend, call and read, and under whose authority" has an answer worth building a structure around, and that the structure is worth its price only when someone outside the team is going to ask.

a microkernel is not a small system; it is a system whose privileged core is small enough to audit - and in an agent, the privileged core is whatever can spend money, call a tool and read data.
