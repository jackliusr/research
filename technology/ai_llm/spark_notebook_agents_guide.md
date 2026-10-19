# Spark in the Notebook — an agent writing and executing Apache Spark in a stateful Jupyter session

> **Author:** Jack Liu Shurui · **Role:** Solution Architect
> **Repo:** [jackliusr/research](https://github.com/jackliusr/research) · **Series:** Technology / AI & LLM · **Domain:** Data engineering, agent execution, reproducibility
> **Date:** 2026-10-19

---

**Contents**

1. [Overview: the join this library does not own](#1-overview-the-join-this-library-does-not-own)
2. [Why a notebook is a hostile environment for an agent](#2-why-a-notebook-is-a-hostile-environment-for-an-agent)
3. [The kernel and session model](#3-the-kernel-and-session-model)
4. [Spark's four specific hazards](#4-sparks-four-specific-hazards)
5. [The feedback loop problem](#5-the-feedback-loop-problem)
6. [What an agent may and may not do in a notebook session](#6-what-an-agent-may-and-may-not-do-in-a-notebook-session)
7. [The reproducibility discipline](#7-the-reproducibility-discipline)
8. [The session boundary: agent, human, and both at once](#8-the-session-boundary-agent-human-and-both-at-once)
9. [The tooling landscape, structurally](#9-the-tooling-landscape-structurally)
10. [The regulated angle: reproducibility as an audit question](#10-the-regulated-angle-reproducibility-as-an-audit-question)
11. [The failure modes](#11-the-failure-modes)
12. [The Cymbal Bank worked example](#12-the-cymbal-bank-worked-example)
13. [The anti-patterns](#13-the-anti-patterns)
14. [The state of practice](#14-the-state-of-practice)
15. [The claims audit](#15-the-claims-audit)
16. [What Could Not Be Verified, glossary and cross-references](#16-what-could-not-be-verified-glossary-and-cross-references)

---

## 1. Overview: the join this library does not own

### 1.1 What this guide is

This is the guide to a specific, narrow and unusually badly-served arrangement: **an AI agent that writes Apache Spark code and executes it inside a stateful Jupyter or Python notebook session**. Not Spark on a cluster submitted as a job, not an agent calling a SQL warehouse through a tool, but the arrangement where the agent's output is a *cell*, the cell runs in a *kernel*, the kernel holds *state*, and the state is the only thing that makes the agent's next cell meaningful.

The library already owns both halves of this arrangement and owns the join nowhere:

- [`technology/spark_tuning_guide.md`](../spark_tuning_guide.md) owns how Spark works and how to make it fast — architecture, memory model, AQE, shuffle, joins, partitioning, the UI as a tuning instrument.
- [`technology/ai_llm/databricks_agents_guide.md`](databricks_agents_guide.md) owns the agent platform surface — governed agents, tools as catalog securables, evaluation and tracing, deployment, and the notebook product as Databricks ships it.

Neither answers the question this guide answers: **when an agent types Spark into a notebook, what does the agent actually know about what happened, and what can a second person reconstruct afterwards?** A keyword check across the library before writing found `agent` near `spark` in a single passing mention, `spark` near `notebook` in four passing mentions, and `agent` near `notebook` in five files, three of which are personal profile guides rather than technical ones. The subject is real, the treatment is absent, and the absence is what this guide fills.

Five things are in scope, and nothing else:

1. The **execution model** the agent is actually operating — kernel, namespace, request queue, lazy plans, sampled observation.
2. The **four technical hazards** that follow from that model, taught mechanically rather than gestured at.
3. The **evidential gap** between what the agent can see and what determines the answer, and the reason the agent's confidence runs ahead of its evidence.
4. The **reproducibility verdict** — what a notebook records by default, what a defensible result requires, and why the agent-driven case is worse than the human-driven one.
5. The **rulings** — what an agent may do, must not do, and who remains accountable for each.

### 1.2 The decoder

Ten terms carry the whole argument. They are defined here once; later sections use them without re-definition.

| Term | Meaning in this guide |
| --- | --- |
| **Agent** | A model-driven process that emits executable code, observes the result through a text channel, and decides what to emit next. The observation channel is the subject of §5. |
| **Notebook** | A document of cells plus, optionally, the outputs those cells produced on some occasion. The document is not the session and does not contain the session. |
| **Kernel** | The process that executes cells. It holds one mutable namespace and consumes execution requests in the order they arrive (Jupyter messaging protocol 5.5, retrieved 2026-10-19). |
| **Session** | The live attachment between a frontend and a kernel. Session state is process state: it ends when the kernel ends, and two sessions do not share it. |
| **Cell** | A unit of source text submitted as one execution request. It has no declared inputs and no declared outputs. Its position in the file implies an order that the kernel does not enforce. |
| **Namespace** | The single mapping from names to objects inside the kernel. Every cell reads and writes it. Rebinding a name changes what every later cell means. |
| **SparkSession** | The driver-side entry point that owns the `SparkContext` — the object the Cluster Mode Overview defines as coordinating the application — together with the catalog and the configuration in force (Spark 4.2.0 docs, Cluster Mode Overview, retrieved 2026-10-19). |
| **Action** | An operation that requires a result to be returned to the driver — `count`, `collect`, `show`, `write`, and similar. Only an action causes computation (Spark 4.2.0 docs, RDD Programming Guide, retrieved 2026-10-19). |
| **Physical plan** | The plan Spark will execute, after analysis and optimisation. It is inspectable before execution with `explain()`, and it can be *re-planned at runtime* by adaptive query execution (Spark 4.2.0 docs, SQL Performance Tuning, retrieved 2026-10-19). |
| **Cluster** | The driver, executors, tasks, stages and jobs that do the work outside the notebook process, reachable for inspection at the Spark Web UI on port 4040 while the application lives (Spark 4.2.0 docs, Web UI, retrieved 2026-10-19). |

### 1.3 A note on the name "AgentSpark"

The phrase that prompted this guide reads like a product name, so it was checked as one. A web identity check found five unrelated companies using the string: a real-estate content-tools site, an AI RPA vendor, an app builder, an automotive-retail product, and a consultancy. None of the five is an Apache Spark product, a Jupyter product, or a notebook product; none is canonical; the string is not a category. This guide therefore treats the phrase as **three technical terms in a row** — an agent, Apache Spark, a Jupyter notebook — and describes the arrangement between them. The five sites are recorded once in the claims audit as a dispatcher identity check (secondary source, names only, 2026-10-19) and are not researched, linked or speculated about further, because the subject is the arrangement and not the name.

### 1.4 The thesis

**An agent that writes Spark is easy; an agent that knows what its Spark did is the hard part.**

Writing Spark is a text-generation problem against a well-documented API, and it is the part that demonstrably works. Knowing what the Spark did is an *evidence* problem, and the notebook makes it hard in three independent ways that compound:

1. **The session is stateful and unordered in practice.** The kernel executes a queue of requests against one namespace. There is no dependency graph, only an arrival order. A cell that ran third may have been written first, and a name may have been rebound by a cell the agent never read.
2. **Execution is lazy.** A transformation builds a plan and returns a DataFrame handle. Nothing has run. An agent that reports "done" after a transformation has reported on the construction of an object, not on data.
3. **Observation is sampled and truncating.** The default inspection call shows twenty rows with long strings elided, which is a prefix with unknown distribution, not a sample from which anything about cardinality, nulls, skew or tails can be inferred.

The thesis is testable and this guide tests it, including where the evidence complicates it. Four instruments genuinely narrow the gap and are conceded in place: `explain()` inspects the plan without executing the data path, which is cheap and honest about the plan as planned; the Spark Web UI and the event-log-backed History Server hold the per-stage truth about spill, skew and task distribution; a pinned table version converts "the data" from a moving target into a nameable object; and a client that records its own execution requests recovers the order the file loses. What none of them does is close the gap automatically for an agent reading stdout, and §5 shows exactly where each one stops.

### 1.5 The boundary, declared by name

A section that explains Spark internals or Databricks product surface has left this guide. The ownership split is:

| Ground | Owner | This guide's relationship |
| --- | --- | --- |
| How Spark works, and how to tune it: driver/executor anatomy, unified memory model, AQE mechanics, shuffle, join strategies, partition tuning, UI-driven tuning workflow | [`technology/spark_tuning_guide.md`](../spark_tuning_guide.md) | Cross-referenced, never re-derived. §4.4 names AQE as a reason the plan may not be the plan that ran; the mechanics of *which* re-optimisation and how to tune it are that guide's §5. |
| The Databricks platform's agent products, the notebook UI, the governed execution model, cluster types, Unity Catalog permissions | [`technology/ai_llm/databricks_agents_guide.md`](databricks_agents_guide.md) | Cross-referenced. This guide quotes the Databricks notebook page only for the *state* facts it states about REPLs and `_sqldf`, which are evidence about session semantics, not product description. |
| Storage layout, table formats, snapshot and version semantics, on-prem versus object-storage lakehouse architecture | [`technology/data/on_prem_lakehouse_guide.md`](../data/on_prem_lakehouse_guide.md), [`technology/cloud_object_storage_lakehouse_guide.md`](../cloud_object_storage_lakehouse_guide.md) | Cross-referenced where §7 requires a data snapshot and §12 pins one. How a table version is stored is not this guide's subject; that it must be named is. |
| Isolation, sandboxing, and the runtime that hosts agent tool execution | [`technology/ai_llm/agent_sandboxing_strategies_guide.md`](agent_sandboxing_strategies_guide.md), [`technology/ai_llm/agent_runtime_cache_design_guide.md`](agent_runtime_cache_design_guide.md) | Cross-referenced for the isolation spectrum and for runtime-level caching. This guide assumes the agent's code execution is already isolated and asks what it can *know*, not what it can *reach*. |

The rule of thumb: **this guide answers "what does the agent know, and what can be reconstructed?"** — not "how does Spark execute a shuffle?" and not "what does the platform ship?".

---

## 2. Why a notebook is a hostile environment for an agent

### 2.1 The cell is not a statement about the program

In a program, a statement's meaning is fixed by its position in a text that is compiled or interpreted top to bottom. In a notebook, a cell's meaning is fixed by the state of the namespace **at the moment it is submitted**, which is a function of every cell that ran before it in that kernel, in the order they ran, including cells that are no longer in the file, cells that ran twice, and cells that ran after the cell below them.

That is not a defect in Jupyter's design; it is what a REPL is for. It becomes a defect the moment the producer of the cells is a process that (a) did not observe the earlier history, and (b) reports the result of its own cell as if the cell's text determined the outcome. Both conditions hold for an agent operating in a notebook it did not create. The agent usually receives some cells, not the session; it receives a file, not the queue.

```
# cell 4, as written by the agent
rate = failed.count() / total.count()
print(f"failed payment rate = {rate:.4f}")
```

Read as a program, this computes a ratio from two named DataFrames. Read as a notebook cell, it computes a ratio from whatever `failed` and `total` were bound to when the cell ran — which may be the agent's definitions from cells 2 and 3, or a previous analyst's definitions from a session three hours earlier that the agent's `failed = ...` never replaced because that cell was skipped. The text is identical in both cases. The number is not.

### 2.2 Re-execution mutates state the agent may not have written

A cell is not idempotent by construction. `df.cache()`, `spark.sql("CREATE OR REPLACE TEMP VIEW ...")`, `df.createOrReplaceTempView(...)`, an append to a list held in a module-level variable, a `%run` of another notebook, a monkey-patched function — all of these have effects that persist in the kernel and are not undone by running the cell again. Running the cell twice does not restore the prior state; it applies the effect twice.

The common agent behaviour that triggers this is not malice, it is debugging: the agent's cell errored, so the agent fixes the text and re-runs. If the first attempt got as far as the side effect and then failed, the re-run operates on state the first attempt already changed. The notebook shows one output, from the successful attempt, and no record of the partial one. There is no transaction boundary around a cell, and no rollback.

### 2.3 Out-of-order execution produces a result the notebook cannot explain

Notebook frontends let a user run cell 9, then cell 3, then cell 9 again. The file order remains 1..9; the execution order does not. The only artefact of that order is the per-cell execution counter, and even that is optional in the file format (§7). A saved notebook with counters reading `1, 2, 3, 7, 5, 9` is a document whose provenance a reader must reverse-engineer, and if the counters were stripped by the client or the file was re-saved from a different kernel, even that is gone.

For a human this is a known sin with a known remedy — "restart and run all" before believing anything. For an agent, the remedy is unavailable in the general case, because the agent did not necessarily start the kernel, cannot necessarily restart it (restarting destroys state the human's other work depends on), and has no way to know whether the run it is looking at was a run-all or a sequence of opportunistic re-runs.

### 2.4 The reproducibility deficit is a design output, not a bug

Put the three together and the property falls out: the notebook, as a format, records a *document* and, optionally, the *outputs of the last execution of each cell*. It does not record a *run*. There is no field for the order cells executed in, no field for which cells were skipped, no field for the Spark version or configuration, no field for the table version read, no field for the physical plan that produced a timing claim. §7 quotes the format's own optional `execution` object to show precisely how little it carries.

This matters more for an agent than for a human for one structural reason: **a human's notebook session has a witness, and an agent's does not.** The human remembers running cells out of order; the person sitting next to them saw it; the standup the next morning heard about it. The agent's ordering decisions are in a transcript that lives with the agent's runtime, not with the notebook, and the notebook is what gets attached to the ticket.

| What actually happened | What the saved notebook shows |
| --- | --- |
| Cell 3 was re-run after cell 7, rebinding `total` | Cell 3's output, cell 7's output, counters out of order if the client wrote them |
| A transformation was submitted and never actioned | A cell with no output; indistinguishable from a cell whose output was cleared |
| `count()` succeeded on a cached subset, not the full frame | A number, formatted normally |
| The cluster spilled 40 GB during the aggregation | Nothing, unless the agent called `explain()` or read the UI |
| The table was read at version 41, then rewritten to 42 by another job | Nothing; the table name is the same string |
| The agent restarted the kernel to clear an error | Counters reset to 1; the pre-restart history is gone |

The rest of this guide is the mechanism behind each row, and the discipline that recovers what can be recovered.

---

## 3. The kernel and session model

### 3.1 One kernel, one namespace, one queue

A Jupyter kernel is a process that owns a namespace and consumes execution requests. The messaging protocol that governs it is explicit that the shell channel is a **single ROUTER socket** carrying "a sequence of request/reply actions from each frontend and the kernel", and that requests for code execution are made there (Jupyter messaging protocol 5.5, retrieved 2026-10-19). The consequences that matter for this guide:

- **One namespace per kernel.** Two cells do not have separate scopes, imports, or SparkSessions. A name defined in cell 1 is visible to cell 900 and can be replaced by it.
- **Ordered consumption, not ordered authorship.** The kernel processes requests in arrival order. Arrival order is whatever the frontend sent, which is whatever the human or agent clicked, which need not be file order.
- **No dependency information.** The protocol has no message that says "this cell depends on that cell". There is no cell-level DAG, no invalidation, no re-computation of dependents. Dependencies exist only as name reads and writes inside Python, at runtime, invisibly to the protocol.

### 3.2 The five sockets, and what each one does and does not tell you

The protocol defines five channels. An agent that speaks to a kernel through a client library sees all five, and it is worth being precise about which one carries evidence.

| Socket | Purpose (per the protocol) | What it gives the agent | What it does not give |
| --- | --- | --- | --- |
| **Shell** | "this single ROUTER socket allows multiple incoming connections from frontends, and this is the socket where requests for code execution, object information, prompts, etc. are made" | The request/reply path: `execute_request` in, `execute_reply` out, including the reply status and `execution_count` | Any statement about what ran *inside* the request beyond success or an error |
| **IOPub** | "the 'broadcast channel' where the kernel publishes all side effects (stdout, stderr, debugging events etc.)" | Stream output, rich display data, `status: busy` / `status: idle` | Structure: stdout is text, not a result set |
| **stdin** | Input requests to the frontend (`input()`) | Nothing an agent should ever rely on; an unattended kernel blocks here | — |
| **Control** | "identical to Shell, but operates on a separate socket to avoid queueing behind execution requests"; used for shutdown and restart, and for debugging | The ability to interrupt or restart a kernel that is stuck | Any execution result; control messages do not run user code |
| **heartbeat** | A liveness channel between client and kernel | Whether the kernel process is alive | Whether it is making progress; a kernel can be alive and busy for a long time |

The IOPub broadcast is the reason "the agent reads stdout" is a defensible description of its whole feedback loop: everything the kernel wants to say, it says on a channel designed for display, in the shape a notebook displays it. §5 builds the evidential table from this fact.

### 3.3 Busy and idle: a bracket, not a progress bar

The protocol's execution lifecycle is precise and worth quoting because agents routinely misread it:

> "The client sends an `<action>_request` message (such as `execute_request`). The kernel receives that request and immediately publishes a `status: busy` message on IOPub. ... `<action>_reply` message, such as `execute_reply`. ... the kernel publishes a `status: idle` message." (Jupyter messaging protocol 5.5, retrieved 2026-10-19)

So the observable lifecycle of one cell is: request sent → `busy` broadcast → zero or more output messages → `execute_reply` on shell → `idle` broadcast. Two things follow.

First, **busy/idle is a bracket around one request, not a measurement of work.** A cell that defines a DataFrame and returns in 40 ms and a cell that runs a six-hour aggregation both produce exactly one busy and one idle. The duration between them is wall-clock time in the kernel, which includes queueing behind other requests, Python-level plan construction, and driver-side waiting on the cluster. It is not a measure of cluster work and should never be reported as one.

Second, **`idle` means the kernel is free, not that the work is done.** For a lazy transformation, `idle` arrives while nothing has been computed. This is the single most consequential misreading available to an agent, and §4.2 is devoted to it.

### 3.4 `execution_count` is a counter, not a dependency graph

Every `execute_reply` and `execute_input` message carries `execution_count`, populated from a counter the kernel maintains. The semantics that matter:

- It counts **executions of this kernel**, in order, starting from 1. It is monotonic within a kernel's life.
- It **resets on restart**. Counters reading 1..12 in a notebook may be the twelfth kernel of the day.
- It advances only for requests that store history: `silent=True` "forces store_history to be False", and `store_history` defaults to True when `silent` is False (Jupyter messaging protocol 5.5, retrieved 2026-10-19). An agent that runs cells silently leaves no counter trace at all.
- It is **not a dependency graph and not a version of the data**. Two cells with counters 4 and 5 tell you only that 4 arrived before 5. They do not tell you that 5 read what 4 wrote; 4 may have written a name that 5 never used, and 5 may have rebound a name 4 depends on.

A notebook whose counters read `1,2,3,9,4,5,10` is telling the truth about arrival order and nothing else. It is not telling you that cell 9 ran before cell 4 in any causal sense — only that the kernel saw it first.

### 3.5 Restart, and exactly what it destroys

Restarting the kernel ends the process and starts a new one. What is destroyed, precisely:

| Destroyed | Survives |
| --- | --- |
| Every Python name, import, function, class and module-level object | The notebook file and its cell text |
| The `SparkSession` and its `SparkContext`, and with them all cached data, all temp views, all registered UDFs, all session-level configuration set by `spark.conf.set` | Data written to external storage by a completed action |
| Broadcast variables, accumulators, checkpoint metadata held in the driver | The cluster's own existence, if it is an external cluster the new session can re-attach to |
| All `execution_count` history; counters restart at 1 | Outputs already saved into the `.ipynb` (they are text in the file, not state in the kernel) |

The asymmetry in that table is the trap. **The file keeps the outputs; the kernel loses the state that produced them.** After a restart, a reader looking at the notebook sees numbers in cells 1–12 and a kernel that can no longer compute any of them until every cell is re-run — in order, against the same data, with the same configuration. If any of those three has changed, the visible numbers are now unreproducible *and* still displayed. A saved notebook with stale outputs is a document, not a log.

### 3.6 What the protocol does not have

Listing the absences is more useful than listing the features, because each absence maps to a failure mode in §11:

- no message carrying a **data version** or table snapshot;
- no message carrying the **Spark configuration** in force;
- no message carrying a **physical plan**;
- no message carrying **cell-to-cell dependency**;
- no record of **skipped** cells — an unexecuted cell is simply silent, and silence is indistinguishable from "ran and printed nothing";
- no record of **partial execution** within a cell — a cell that fails halfway is one error message, not a list of the side effects it already applied.

An agent designing its own observation strategy has to supply all six from outside the protocol. §6 and §7 say what that looks like.

---

## 4. Spark's four specific hazards

The kernel model is generic to notebooks. Spark adds four hazards that are specific to it, and each one converts a generic notebook weakness into a wrong number rather than an untidy document. They are taught here in the order an agent encounters them: state, then laziness, then observation, then the cluster.

### 4.1 Trap one: the stateful kernel and the name nobody admits binding

**Mechanism.** Spark state lives in the kernel process and is per-REPL, not per-notebook. Databricks states this from the vendor side with unusual directness: "When you invoke a language magic command, the command is dispatched to the REPL in the execution context for the notebook. Variables defined in one language (and hence in the REPL for that language) are not available in the REPL of another language. REPLs can share state only through external resources such as files in DBFS or objects in object storage." And: "Variables and state are isolated between different language REPLs. For example, Python variables are not accessible in Scala cells." (Databricks docs, last updated 2026-10-13)

The general lesson is that **state is attached to an execution context, and the boundaries between contexts are not the boundaries a reader assumes**. The file looks like one document; the runtime is a set of REPLs with different lifetimes and different contents.

The concrete proof that a later cell can silently rebind a name an earlier cell believes it owns is `_sqldf`. On Databricks, "results from a SQL language cell are automatically made available as an implicit DataFrame assigned to the variable `_sqldf`", usable "in any Python and SQL cells you run afterward, regardless of their position in the notebook" — and then the sentence that matters: "**The variable `_sqldf` is reassigned each time a SQL cell is run.**" (Databricks docs, last updated 2026-10-13)

Read that as a mechanism rather than a vendor quirk. There exists a name in the namespace that **no cell in the file binds explicitly**, that is bound by the act of running a SQL cell, and that changes identity every time any SQL cell runs. An agent that reads `_sqldf`, transforms it, and then runs one more SQL cell before its action has, without writing a single assignment, changed the input to its own computation. Nothing in the file shows this. The cell text that reads `_sqldf` is identical before and after; the object behind the name is different, and the second read may be a completely different query result.

Three implications for an agent:

1. **Reading a name is not evidence about its provenance.** `df` in the namespace has whatever history the kernel gave it. The agent can inspect its schema and its plan, and neither tells it which cell bound the name or when.
2. **Writing a name is a mutation of shared state.** `total = ...` in a cell does not shadow a previous `total`; it replaces it, for every subsequent cell, including cells the agent did not write.
3. **A restart is the only clean slate, and it is not the agent's to take.** Restarting destroys the human's work too (§3.5, §8). The agent's correct move is to bind every name it depends on inside the cell that uses it, or to a uniquely namespaced name, and to verify the binding rather than assume it.

**What an agent should do instead of trusting a name.** Rebind from a source it can name, in the same cell, immediately before use:

```python
# Bind from a pinned source, in the cell that uses it.
snapshot = "payments@v41"          # the version is part of the name
src = spark.read.option("versionAsOf", 41).table("payments")
src.createOrReplaceTempView("payments_v41")   # explicit, owned, versioned name
total = src.count()                            # action, in the same cell
```

The point is not the specific API. The point is that the cell's correctness does not depend on a name some other cell may have rebound.

### 4.2 Trap two: laziness — the transformation that never ran

**Mechanism.** This is the most load-bearing sentence in the guide, quoted verbatim from the Spark 4.2.0 documentation:

> "All transformations in Spark are _lazy_, in that they do not compute their results right away. Instead, they just remember the transformations applied to some base dataset (e.g. a file). The transformations are only computed when an action requires a result to be returned to the driver program. This design enables Spark to run more efficiently." (Spark 4.2.0 docs, RDD Programming Guide, retrieved 2026-10-19)

A DataFrame transformation — `filter`, `select`, `withColumn`, `join`, `groupBy().agg()`, `repartition`, `dropDuplicates` — returns a **new DataFrame object describing a plan**. It does not read a row. It does not contact the cluster beyond metadata. It returns in milliseconds regardless of whether the input is twenty rows or twenty terabytes. The kernel goes `busy` then `idle` and the agent sees a success reply.

That laziness is semantic, not stylistic, is confirmed by the documentation's own note on accumulators: "Accumulators do not change the lazy evaluation model of Spark. If they are being updated within an operation on an RDD, their value is only updated once that RDD is computed as part of an action. Consequently, accumulator updates are not guaranteed to be executed when made within a lazy transformation like `map()`." (Spark 4.2.0 docs, RDD Programming Guide, retrieved 2026-10-19). Even a side-effect mechanism designed for counting does not fire until an action. There is no back door.

**The consequences, stated as failure modes rather than warnings:**

- **Success is reported on work that never ran.** The agent writes `clean = raw.filter(...).withColumn(...)`, sees `execute_reply: ok`, and reports "cleaned the data". No data was cleaned. A plan was built. If the agent's next step is to tell the user the job is done, the report is false and nothing in the feedback channel contradicted it.
- **Failures surface cells later and are attributed to the wrong cell.** A schema mismatch, a missing column, an unresolved function, a permission error on the underlying table — many of these are only detected when the plan is executed, which is when an action runs, which may be three cells after the defective transformation. The traceback points at the action's cell. The agent, reading a traceback with a line number, fixes the wrong cell. This is the most common way an agent "fixes" a bug by adding a cast in a place that had nothing to do with it.
- **Cost is deferred and then concentrated.** Planning is free; execution is not. An agent that iterates cheaply on transformations, checking each one for errors, is checking nothing. The first real check is the first action, and by then the plan may be expensive.

**The single discipline that fixes this.** Every claim about data must be attached to an action, and the action must be named:

| Claim the agent wants to make | The action that would support it |
| --- | --- |
| "The filter removed the bad rows" | `clean.count()` compared against `raw.count()`, both reported |
| "The join matched" | `joined.count()` and a null check on the join key, as actions |
| "The dedup worked" | `clean.select(key).distinct().count() == clean.count()`, as an action |
| "The write succeeded" | The write action returned, and a read-back of the target |
| "The query is fast" | Not an action at all — a timing claim, which requires a plan and a measured run (§5, §7) |

The distinction to hold onto: **an action proves that something ran; it does not prove that what ran is what the agent thinks it wrote.** That is the subject of the next two traps.

### 4.3 Trap three: sampled observation — twenty rows that are not the answer

**Mechanism.** The default inspection call is not neutral. Its signature is a statement about what the agent is allowed to see:

`DataFrame.show(n=20, truncate=True, vertical=False)` — "Prints the first `n` rows of the DataFrame to the console", with `n` defaulting to **20** and `truncate` defaulting to **True**, which truncates "strings longer than 20 chars" (PySpark 4.2.0 API docs, retrieved 2026-10-19). The documentation's own example output ends with the line "only showing top 2 rows".

Three separate limitations are encoded in that one signature:

1. **`n=20` is a prefix, not a sample.** `show()` returns the first rows the plan produced. Which rows those are depends on partition order and on the plan. There is no randomisation, no stratification, and no statement about what fraction of the data was seen. A prefix of a sorted or clustered dataset is systematically unrepresentative; a prefix of a shuffled dataset is arbitrary but still not a sample with a known distribution.
2. **`truncate=True` means the agent may be reading elided values.** A 40-character status string is displayed as its first 20 characters plus an ellipsis. Two distinct categories that share a prefix become indistinguishable in the output. An agent that infers "there are three status values" from a truncated column is inferring from a lossy projection of the data.
3. **`show()` is an action with a small result, so it proves execution but not scale.** It does run the plan — this is the one place where `show()` is genuinely better than a transformation — but it runs it far enough to produce twenty rows on the driver. It says nothing about the other partitions beyond the fact that the plan did not fail before producing them.

**The statistical point, made properly.** A twenty-row prefix supports **no inference** about:

- **cardinality** — twenty rows of a high-cardinality key look exactly like twenty rows of a low-cardinality one;
- **null rate** — if nulls are rare, a prefix of twenty may contain none while the column is 80% null in the tail;
- **skew** — key distribution in the prefix reflects the first partition's ordering, not the cluster's shuffle;
- **tail behaviour** — the outliers that break an aggregation are, by construction, not in the first twenty rows;
- **truncated categories** — elided strings hide distinctions the agent is about to reason over.

The correct mental model is: **`show()` is a smoke test for "did the plan produce rows of roughly this shape", and nothing else.** It is a good smoke test. An agent that treats it as a data profile is not sampling; it is guessing with a screenshot.

**The two escapes, and their costs.** `collect()` "Returns all the records in the DataFrame as a list of `Row`" and its own documentation warns: "This method should only be used if the resulting list is expected to be small, **as all the data is loaded into the driver's memory**" (PySpark 4.2.0 API docs, retrieved 2026-10-19). On a large frame this is not an answer; it is a driver OOM, and the failure takes down the session that held the rest of the work. `head()` / `take(n)` return "the first n rows", which is the same prefix problem with a different name.

| Call | What it returns | What it costs | What it can support |
| --- | --- | --- | --- |
| `show()` | First 20 rows, strings truncated at 20 chars | One small action | Shape, obvious type errors, "did anything come out" |
| `show(n=200, truncate=False)` | First 200 rows, full values | One slightly larger action | Column contents, category spellings, still a prefix |
| `head()` / `take(n)` | First n rows as Python objects | Driver memory for n rows | Same prefix caveat; useful for feeding a few values to code |
| `collect()` | All rows | Driver memory for the whole frame | Only when the frame is known small — and "known" needs a `count()` first |
| `count()` / `agg()` | Scalars computed on the cluster | One full pass | Cardinality, null counts, sums, min/max, quantiles — real inference |
| `describe()` / `summary()` | Column statistics computed on the cluster | One full (or sampled, per implementation) pass | Distributional facts at a stated confidence |

The pattern in the bottom three rows is the answer to the trap: **push the question to the cluster as an aggregate, and let a scalar come back.** An aggregate is not a sample. It is computed over all rows and returns a number whose provenance is the plan. An agent that wants to know the null rate should ask for the null rate, not look at twenty rows and count nulls by eye — and it should say which query produced the number.

### 4.4 Trap four: the cluster the agent cannot see

**Mechanism.** The notebook process is the driver. The work happens elsewhere:

> "Spark applications run as independent sets of processes on a cluster, coordinated by the `SparkContext` object in your main program (called the _driver program_)." ... "Once connected, Spark acquires _executors_ on nodes in the cluster, which are processes that run computations and store data for your application. Next, it sends your application code ... to the executors. Finally, SparkContext sends _tasks_ to the executors to run." (Spark 4.2.0 docs, Cluster Mode Overview, retrieved 2026-10-19)

The same page's glossary fixes the vocabulary this guide uses: an **executor** is "A process launched for an application on a worker node, that runs tasks and keeps data in memory or disk storage across them"; a **task** is "A unit of work that will be sent to one executor"; a **job** is "A parallel computation consisting of multiple tasks that gets spawned in response to a Spark action (e.g. `save`, `collect`)"; and "Each job gets divided into smaller sets of tasks called _stages_ that depend on each other" (Spark 4.2.0 docs, Cluster Mode Overview, retrieved 2026-10-19).

Now the evidential problem in one sentence: **every one of those objects is outside the notebook's text interface.** Partition counts, per-task duration distribution, shuffle read and write sizes, spill to disk, GC time, executor memory pressure and the physical plan are properties of the job, the stages and the tasks — not of the cell — and the cell's output is whatever the driver chose to print.

Where the truth actually lives, and its lifetime:

> "The Web UI is built into every Spark application: while the application is running, it serves a set of web pages that let you inspect what is happening inside it." ... "By default the Web UI is available at `http://<driver-host>:4040`." ... "The Web UI is tied to the lifetime of the application: once it exits, the UI is no longer reachable. To inspect an application after it has finished, enable event logging and run the Spark History Server, which reconstructs an equivalent UI from the persisted event log" (Spark 4.2.0 docs, Web UI, retrieved 2026-10-19).

Two consequences. First, **the evidence has a lifetime shorter than the notebook's.** The `.ipynb` survives; the UI does not. An agent that finishes a job and then gets asked "why was that slow" has, by default, nothing to look at — the application that would have answered is gone unless event logging was on before it started. Second, **the SQL tab holds a different class of object from the one the agent read**: "Detail of the execution plan with parsed logical plan, analyzed logical plan, optimized logical plan and physical plan" (Spark 4.2.0 docs, Web UI, retrieved 2026-10-19). The agent typically read the output of `explain()`, which is the plan at the moment of the call — not the plan after runtime re-optimisation.

That last gap is the fourth trap's sharp edge. Adaptive query execution re-optimises the plan **at runtime**, using statistics that only exist once stages have run — coalescing post-shuffle partitions, splitting skewed partitions, converting a sort-merge join to a broadcast join or a shuffled hash join (Spark 4.2.0 docs, SQL Performance Tuning, retrieved 2026-10-19; the tuning mechanics belong to [`../spark_tuning_guide.md`](../spark_tuning_guide.md) §5 and are not re-derived here). So a plan read before execution can differ from the plan that ran, in exactly the dimension — partition count, join strategy, skew handling — that determines whether the job was fast, slow, or correct at scale. The `explain()` output is truthful about what Spark intended. It is not a record of what happened.

### 4.5 The four traps in one table

| Trap | What the agent observes | What is actually true | The lie the observation supports |
| --- | --- | --- | --- |
| Stateful kernel | A name resolves; a cell returns ok | The name may be bound by another cell, another REPL, or an implicit binding like `_sqldf` that was reassigned | "This is the data I defined" |
| Laziness | `execute_reply: ok`, fast, no output | A plan was constructed; nothing was read or computed | "The transformation ran" |
| Sampled observation | 20 rows, truncated strings, plausible values | A prefix of unknown distribution, with elided cells | "The data looks like this" |
| Invisible cluster | A scalar, a duration, an `explain()` plan | Partitions, skew, spill, executor memory and possibly a re-planned physical plan determined the outcome | "I know why this took that long" |

Each row is a different epistemic failure, and they compose: a lazy transformation observed through a twenty-row prefix, over a name rebound by an implicit binding, timed on a cluster whose plan was re-optimised after the plan was read, is a result that is wrong in four independent ways and looks entirely normal in the notebook.

---

## 5. The feedback loop problem

### 5.1 The asymmetry, stated as a table

This is the analytical core of the guide. An agent's competence at writing Spark is bounded by its model; its competence at *knowing what its Spark did* is bounded by its observation channel. The channel is narrow, and the narrowing is systematic rather than random.

| What the agent can observe | What actually determines the outcome |
| --- | --- |
| **stdout / stream text** — prints, `show()` tables, warnings | Nothing about the data path. stdout is a display encoding chosen by the driver code. |
| **An exception string** — a Python traceback, a JVM stack trace, an error class | The failing task's *first* failure. Retries, speculative execution and partial writes are not in it. |
| **A sampled frame** — 20 rows, truncated at 20 chars by default (PySpark 4.2.0 API docs, retrieved 2026-10-19) | The full distribution: cardinality, null rate, key skew, tail values, category spellings beyond the truncation. |
| **A schema** — column names and types | The data's conformance to that schema. A schema is a claim by the reader, not a fact about the rows; a permissive read produces a schema for data that does not match it. |
| **An `explain()` plan** — logical and physical, without executing the data path (PySpark 4.2.0 API docs, retrieved 2026-10-19) | The plan *after* runtime re-optimisation. AQE re-plans on statistics that only exist once stages have run (Spark 4.2.0 docs, SQL Performance Tuning, retrieved 2026-10-19). |
| **A wall-clock duration** — the gap between busy and idle on IOPub | Where the time went: queueing, driver planning, shuffle, spill, GC, a single straggler task. |
| **A returned scalar** — the number from `count()`, `sum()`, an aggregate | Whether the plan that produced it read the table version the agent believes it read, and whether a later cell rebound the input. |

Read the two columns as a claim about *inferential distance*. The left column is text produced by the driver for a human to look at. The right column is the behaviour of a distributed system on a specific dataset at a specific version. There is no mapping from one to the other that does not pass through assumptions the agent cannot check from inside the notebook.

### 5.2 Why the agent's confidence runs ahead of its evidence

Four mechanisms, and they are worth naming because each one has a different remedy.

**1. The success signal carries no truth content.** `execute_reply: ok` means the kernel accepted the request and the Python call returned without raising. For a transformation, returning without raising is the *only* thing that happened. The protocol has no vocabulary for "the work this describes has been done" — because for laziness, it has not (Jupyter messaging protocol 5.5, retrieved 2026-10-19; Spark 4.2.0 docs, RDD Programming Guide, retrieved 2026-10-19). An agent trained to treat a non-error as progress will treat plan construction as completion, and the signal it is reading is genuinely a success signal. It is just a success signal about the wrong thing.

**2. Fluent output looks like verified output.** A DataFrame printed as a table with aligned columns and plausible values is the same visual artefact whether it came from twenty rows of real data or twenty rows the agent itself fabricated in a `createDataFrame` three cells earlier. The display layer normalises both. An agent with no independent check has no way to distinguish "the pipeline produced this" from "my test fixture produced this", and the second is a common and reasonable thing for it to have created earlier in the session.

**3. Only the visible cells vote.** The notebook shows the outputs of cells that ran. A cell that never ran shows nothing, and nothing looks like "no output" rather than "not executed". A cell that ran and printed nothing looks identical. A cell whose output was cleared looks identical again. The agent's evidence set is therefore the union of *things that produced output*, which is a biased sample of *things that happened* — and the bias runs toward the cells that produced something reassuring.

**4. The agent cannot see its own ordering decisions as a whole.** An agent working cell by cell has, at each step, a local view: what it just wrote, what it just saw. It does not have a global view of which cells it re-ran, which it skipped because they looked fine, and which it edited after running. The human reviewer has that view if they watched, and loses it the moment they look away. The session transcript may have it; the notebook does not.

### 5.3 Where the gap genuinely narrows — and where each tool stops

The thesis is not that the gap is absolute. Four instruments narrow it materially, and honesty requires conceding each one before naming its limit.

| Instrument | What it genuinely gives | Where it stops |
| --- | --- | --- |
| **`explain()`** | A plan without executing the data path. `extended=False` prints only the physical plan; `mode="formatted"` splits it into outline and node details; `mode="cost"` prints "a logical plan and statistics if they are available" (PySpark 4.2.0 API docs, retrieved 2026-10-19) | It is the plan *as planned*. Runtime re-optimisation can change the join strategy and the partition count (Spark 4.2.0 docs, SQL Performance Tuning, retrieved 2026-10-19). And `cost` mode's "if they are available" is doing real work: with no statistics, the estimate is a guess the agent will read as a measurement. |
| **The Spark Web UI and History Server** | The per-stage truth: task duration distribution, shuffle sizes, spill, GC, executor memory, and the SQL tab's parsed/analyzed/optimized/physical plans (Spark 4.2.0 docs, Web UI, retrieved 2026-10-19) | Tied to application lifetime. The UI dies with the application; only event logging plus the History Server survives it (Spark 4.2.0 docs, Web UI, retrieved 2026-10-19). The agent must have enabled logging *before* the run, and must be able to read a UI or an event log at all — most agent feedback loops cannot. |
| **A pinned table version** | Converts "the data" into a nameable object. A read against version *N* is a read against version *N*, and a later write to the table does not change it. | It pins the input, not the computation. The same snapshot read with a different configuration, a different Spark version, or a different plan can produce a different number — and the pin says nothing about the plan. Storage-level mechanics belong to [`../data/on_prem_lakehouse_guide.md`](../data/on_prem_lakehouse_guide.md) and [`../cloud_object_storage_lakehouse_guide.md`](../cloud_object_storage_lakehouse_guide.md). |
| **A recorded session** (execution order captured by the client, not by the file) | The one thing nbformat does not carry: the order cells actually ran in, including re-runs and skips, and the code as submitted rather than as saved | It records what was *submitted*, not what *resulted*. A recorded re-run of a cell that failed halfway still needs the side-effect analysis from §2.2, and the recording does not know which names were rebound. |

The honest formulation: **each instrument closes one axis of the gap and leaves the others open.** `explain()` closes *what was planned* but not *what ran*. The UI closes *what ran* but not *what it meant*. A version pin closes *which data* but not *which computation*. A recorded session closes *what order* but not *what state*. A defensible result needs all four axes, and the default notebook session supplies none of them.

### 5.4 The practical consequence for agent design

An agent that is built to report on Spark should be built to report **the evidence it has, labelled as such**, not a conclusion that outruns it. The difference in practice:

```
Weak:   "The failed-payment rate is 3.8%."
Honest: "The failed-payment rate is 3.8%, computed by cell 14
         (count of status='FAILED' / count of all rows) over
         payments at version 41, on Spark 4.2.0, with plan captured
         below. The figure has not been re-run from a clean kernel."
```

The second sentence is longer and less satisfying, and it is the only one a reviewer can act on. §6 turns this into rulings; §7 turns it into an artefact.

---

## 6. What an agent may and may not do in a notebook session

### 6.1 The rulings

The table below is the guide's operative content. The accountability column is not decorative: for every allowed action, a named human role remains answerable, and the role is different from "the agent" by construction because an agent cannot be accountable. An agent is a tool; the person who reviewed and merged the cell, or who published the number, is the person a supervisor will ask.

| Action | Ruling | Why | Who remains accountable |
| --- | --- | --- | --- |
| Write cells | **May** | Text generation is the low-risk half. Cells are reviewable before they run. | The human who reviewed and merged the cell into the notebook of record. |
| Run cells it wrote, in a session it can identify | **May** | Execution is how the agent learns anything at all. | The notebook owner, who granted the agent a session against shared state. |
| Inspect a schema (`printSchema`, `dtypes`) | **May** | Cheap, non-executing, and narrows one axis (§5.3). | No separate accountability; covered by the cell reviewer. |
| Sample rows (`show`, `head`, `take`) | **May**, within a stated row count | Cheap and useful as a smoke test, provided no conclusion is drawn beyond shape (§4.3). | The analyst who relied on the sample for a decision. |
| `explain()` before a performance claim | **Must**, if any performance claim is made | The plan is the minimum evidence that the claim is about the query the agent thinks it is. | The reviewer who accepted the performance claim. |
| Pin a table version for any published figure | **Must** | A figure over a moving table is a figure over an unnamed dataset. | The data owner who certified the snapshot; the publisher of the figure. |
| Overwrite a name it did not bind | **Must not** | Rebinding mutates shared state for every later cell, including the human's (§4.1). | The agent operator, for not constraining the agent's namespace. |
| Drop, alter or truncate a table or view | **Must not** | Destructive, irreversible without a snapshot, and outside the agent's mandate in an exploration session. Propose the statement; let a human run it. | The human who executes the DDL, and the data owner. |
| Kill or restart a session with unsaved state | **Must not**, without explicit human consent | Restart destroys cached data, temp views and the human's namespace (§3.5). | The human who consented, and the session owner. |
| Run an unbounded `collect()` | **Must not** | It loads all records into driver memory (PySpark 4.2.0 API docs, retrieved 2026-10-19) and fails as an OOM, taking the session with it. | The reviewer who accepted code containing an unguarded `collect()`. |
| Re-run a cell whose effect is not idempotent | **Must not**, without checking the effect | Second application of a side effect is not a retry; it is a second effect (§2.2). | The agent operator, for not requiring idempotence in the harness. |
| Claim a performance result without a plan | **Must not** | A duration without a plan is a number about the cluster's mood, not about the query (§5.1). | The person who repeated the claim to a stakeholder. |
| Report a number without its snapshot, version and cell | **Must not** | The number cannot be reconstructed by anyone else (§7). | The publisher of the number, by name. |
| Modify cluster configuration mid-session | **May not**, without recording it | `spark.conf.set` persists in the session and silently changes every later computation's behaviour. | The notebook owner, who owns the session's configuration. |

### 6.2 What "accountable" means concretely

Four roles appear in the table. Naming them matters because "the team is accountable" is how accountability disappears.

- **The cell reviewer.** The person who read the diff and merged the cell. Their signature is on the code, and a supervisor's first question — "who wrote this line?" — resolves to them, not to the model. This is the same control as code review, extended to notebook cells, and it is the reason agent-authored cells should arrive as reviewable text rather than as keystrokes in a live session.
- **The notebook owner.** The person who owns the session and the cluster attachment. They granted the agent access to a namespace that also holds their own work; they are answerable for the blast radius of a restart, a rebind, or a runaway `collect()`.
- **The data owner.** The person who certified the table and the snapshot. When the figure is challenged, the question "which version of the data?" is answered by them, and a pinned version is their signature.
- **The publisher.** The person whose name is on the number in the report, the deck or the regulatory return. This is the only role that cannot be delegated to a reviewer, because publishing is the act that creates reliance.

An agent has no place in that list. It can be *audited* — the transcript exists, the cells exist — but it cannot be *answerable*, and any governance design that says "the agent is responsible for its outputs" has moved the accountability to a party that cannot hold it.

### 6.3 The rule that generates the rest

If only one ruling can be enforced, enforce this one: **the agent may not be the only party who knows what the cell did.** Every "must not" in §6.1 is a case where the agent's knowledge of its own action would be the sole record — the rebind, the restart, the `collect()`, the re-run, the unpublished number. Where a human or an artefact holds the same knowledge, the action is tolerable; where the agent is the only witness, it is not.

---

## 7. The reproducibility discipline

### 7.1 The verdict

**A notebook is an exploration artefact, not a production record. An agent-driven notebook is less reproducible than a human-driven one, because the agent may have executed cells in an order nobody recorded, and the record it leaves is the same file a human would leave.**

The second clause is the part people resist. The intuition is that an agent is *more* deterministic than a human — it does not get distracted, it does not forget which cell it ran. That intuition is wrong in the dimension that matters. A human's out-of-order execution is an anomaly they can be asked about; an agent's is a routine behaviour with no witness, because the agent's own record of its actions lives in the agent's runtime and the notebook is what gets kept. The agent also re-runs more often (it iterates on error messages), rebinds more often (it redefines names rather than editing them), and restarts more readily (it treats a kernel as a resource to reset). Each of those is a normal engineering behaviour and each one widens the gap between the file and the run.

### 7.2 What must be captured

Four things, and a result missing any of them is not defensible:

| # | What must be captured | Why it is load-bearing | What it answers |
| --- | --- | --- | --- |
| **a** | **The executed cell sequence** — the order cells actually ran, including re-runs and skipped cells, with the code as submitted | The file's order is authorship order, not execution order (§2.3). Without the sequence, "which version of cell 14 produced this number" has no answer. | "In what order was it run?" |
| **b** | **The Spark version and the full effective configuration** — not the defaults you assume, the values in force, including session-level `spark.conf` settings | Computation depends on the version and on settings that change partitioning, join strategy, shuffling and AQE behaviour. A number is a number *for a configuration*. | "On what engine, with what settings?" |
| **c** | **The data snapshot / table version** | A table name is a mutable pointer. A figure over "payments" is a figure over whatever "payments" meant at the time — and another job may have rewritten it since. | "Over which data?" |
| **d** | **The physical plan, whenever any performance claim is made** | Without the plan, a duration is unattributable; with it, a reviewer can see the join strategy, the partition count, and whether AQE changed them (§4.4). | "With what plan?" |

### 7.3 What the notebook does not capture by default

The format's own schema settles this, and it is worth being exact because "the notebook records everything" is a common and false belief.

nbformat v4.5 defines an **optional** per-cell `execution` object whose purpose is "Execution time for the code in the cell. This tracks time at which messages are received from iopub or shell channels", with four ISO-8601 timestamps — `iopub.execute_input`, `iopub.status.busy`, `shell.execute_reply`, `iopub.status.idle` (nbformat v4.5 schema, retrieved 2026-10-19). Beside it sits the integer `execution_count`.

That is the entire per-cell execution record the format can carry, and it is *optional* — a client may write it, and many do not. It contains:

- four timestamps, if written, which give you a duration and a rough order **within one kernel's life**;
- an integer counter that resets on restart (§3.4).

It contains **no** Spark version, **no** configuration, **no** table version or data snapshot, **no** physical plan, **no** record of which cells were skipped, and **no** record of what happened across a restart — because the counter that would have shown the discontinuity restarts at 1, and the pre-restart cells keep their old counters in the file. A reader cannot distinguish "cell 3 ran before cell 9" from "cell 3 ran in the previous kernel and cell 9 in this one" using the file alone.

The four timestamps are also weaker than they look. They record when *messages* were received, not when work happened. For a lazy transformation, the window between `execute_input` and `status.idle` is the time to build a plan — 40 ms of Python — and it is recorded with the same fidelity as a six-hour aggregation's window. The format is measuring the kernel's responsiveness, not the computation.

### 7.4 The minimum defensible artefact

The fix is a discipline, not a product, and it has a name: **the run record**. One artefact, produced per published result, containing the four captures in §7.2 and stored with the notebook rather than inside it. It is deliberately boring.

```
RUN RECORD — payments failed-rate, 2026-10-19
  result        : 3.79% failed-payment rate (illustrative; the unrecomputed
                  figure was 3.81% — see §12)
  cell sequence : 1,2,3,4,6,7,9,10,11   (5 and 8 skipped: scratch)
                  re-runs: 9 (twice; first attempt failed on schema), 11 (once)
  engine        : Spark 4.2.0 (released 2026-07-14)
  config        : spark.sql.adaptive.enabled=true
                  spark.sql.shuffle.partitions=200   # recorded as found, not as advice
                  spark.sql.adaptive.coalescePartitions.enabled=true
                  [full effective config dumped, not just overrides]
  data          : catalog.schema.payments @ version 41 (pinned, read with
                  versionAsOf=41; certified by data owner on 2026-10-19)
  plan          : captured from explain(mode="formatted") before execution,
                  plus the SQL tab plan after the run (event log enabled)
  kernel        : fresh kernel at start of run; restarted once at 09:14
                  (pre-restart cells 1-6 re-run in order afterwards)
  produced by   : agent session <id>, reviewed and merged by <human>, published
                  by <human>
```

Three properties make this artefact work where the notebook does not:

1. **It is written against a clean kernel.** "Restart and run all, then record" is the only procedure that makes the cell sequence equal the file order, and it is the one procedure an agent should be required to trigger before a number is published — with the human's consent, because it destroys the session (§6.1).
2. **It names versions rather than assuming them.** Spark version, configuration, table version. Each is a string a reviewer can check; none is inferable from the notebook.
3. **It separates the plan read from the plan run.** Both are captured when a performance claim is made, which is the only way to see whether AQE changed the answer between them.

What this artefact does *not* do is make the number reproducible by itself. It makes the number **reconstructible**: a second person can re-run the sequence against the named snapshot on the named engine and compare. Reconstruction with a stated procedure is the standard a regulated institution can actually hold; bit-identical reproduction of a distributed computation is not, and promising it would be worse than the notebook's silence because it would be believed.

---

## 8. The session boundary: agent, human, and both at once

### 8.1 One kernel, several frontends

The protocol permits a fan-in that most users never notice:

> "A single kernel can be simultaneously connected to one or more frontends." (Jupyter messaging protocol 5.5, retrieved 2026-10-19)

The shell socket is a **ROUTER**, which is what makes this work: multiple frontends connect, and the kernel routes replies back to the requesting frontend. Execution requests from all of them enter the **same ordered queue** against the **same namespace**. There is no per-frontend isolation, no per-frontend view of history, and no notification to frontend A that frontend B just rebound a name.

This is the arrangement whenever an agent is attached to a notebook a human is also using, and it is the arrangement in most "agent in your notebook" designs. The consequences:

| Property | Shared between frontends? | Evidence |
| --- | --- | --- |
| The namespace (variables, imports, functions) | **Yes** — one namespace per kernel | Jupyter messaging protocol 5.5: one kernel, multiple frontends, one execution queue (retrieved 2026-10-19) |
| Execution order | **Yes** — one queue, arrival order | Same |
| Outputs | **Yes, on IOPub** — a broadcast, so every attached frontend sees every side effect | Same: IOPub is "the 'broadcast channel'" |
| `execution_count` | **Yes** — one counter per kernel, not per frontend | Same |
| Cell text in the file | **No** — the file is whichever frontend saves it, and it may not contain the other's cells at all | §2.1, §7.3 |
| Cached DataFrames and temp views | **Yes** — they are kernel state | §3.5 |
| Understanding of what the other frontend did | **No** — there is no message for it | §3.6 |

The last row is the whole problem. The protocol shares *state* perfectly and shares *intent* not at all.

### 8.2 What a forked or handed-over session does and does not carry

Agent designs often assume a session can be forked, checkpointed, or handed to a human for review. The evidence says to be careful about which of those is real.

- **A second session against the same cluster does not see the first session's temporary state.** Databricks documents the parallel case precisely: "While a command is running and your notebook is attached to an interactive cluster, you can run a SQL cell simultaneously with the current command. The SQL cell is executed in a new, parallel session." And then the limits: "Because the cell is run in a new session, temporary views, UDFs, and the implicit Python DataFrame (`_sqldf`) are not supported for cells that are executed in parallel. In addition, the default catalog and database names are used during parallel execution." (Databricks docs, last updated 2026-10-13)
- **Two `SparkContext`s do not share data at all.** "data cannot be shared across different Spark applications (instances of SparkContext) without writing it to an external storage system" (Spark 4.2.0 docs, Cluster Mode Overview, retrieved 2026-10-19). A forked session that gets a new context is not a fork of the computation; it is a second computation that must re-read from storage.
- **A handover carries the file, and the file carries neither the order nor the state.** §7.3. The human receiving a notebook from an agent receives cell text and, if the client wrote them, outputs and counters. They do not receive the namespace, the cached frames, the temp views, the session configuration, or the execution order across a restart.

So "fork the session" is, in practice, one of three things, and only one of them is a fork:

| What people call a fork | What it actually is | What it preserves |
| --- | --- | --- |
| A second frontend attached to the same kernel | **Sharing**, not forking — same namespace, same queue | Everything, including the other party's mistakes |
| A new kernel over the same notebook | **Replay** — every cell must re-run, against whatever the data is now | The text; not the state, not the snapshot, not the order |
| A new session against the same cluster | **A second computation** — no temp views, no UDFs, no shared cache | The cluster; nothing else |

### 8.3 The handover protocol that actually works

Given the above, a handover from agent to human (or agent to agent) should transfer four things explicitly, because none of them is in the notebook:

1. **The kernel's identity and age** — a fresh kernel started for this task, or a shared one with prior history. If shared, the human needs to know that names may be stale before they trust a single output.
2. **The names the agent bound, listed** — so the human can see what was replaced. This is cheap to produce (the agent knows what it assigned) and it is the only defence against the `_sqldf`-class problem in §4.1.
3. **The data versions read** — as strings, per table. Not "the payments table".
4. **The stopping point** — which cells ran, which did not, and whether the last action completed. A handover that says "I got as far as cell 11, cell 12 is untested" is worth more than a notebook with twelve green outputs, because the twelfth output may be from an earlier run.

The alternative — hand over the `.ipynb` and let the human infer — is the failure mode in §11 row six, and it is the default behaviour of every notebook-based agent integration that has not thought about this.

---

## 9. The tooling landscape, structurally

### 9.1 Why categories rather than products

The temptation in a section like this is a comparison table of named tools, which would immediately become a ranking and would be stale within a quarter. The useful structure is **where the code executes and what the executing layer can therefore observe** — because that, and not the feature list, determines whether the agent can answer the questions in §5.1. Four categories cover the space. None is recommended; each is described by what it can and cannot see.

### 9.2 Category one: the code-interpreter runtime

A process — usually a container or a microVM — that accepts a snippet, runs it, and returns stdout, stderr and a status. Sessions are usually scoped to a conversation; state may persist within the session and is discarded when it ends.

| | |
| --- | --- |
| **What it can observe** | Exit status, stdout/stderr, returned values, files written into its own workspace. If it hosts a Python process with PySpark installed, it can hold a SparkSession and a namespace exactly as a notebook kernel does. |
| **What it cannot observe** | Anything outside its process: the cluster's UI, the executor logs, the event log, the table's version history. Its view of Spark is the driver's view, narrowed to what the snippet printed. |
| **What it is structurally good at** | Isolation and cleanup. A discarded runtime cannot leave a stale binding behind for the next conversation. Cross-reference [`agent_sandboxing_strategies_guide.md`](agent_sandboxing_strategies_guide.md) for the isolation spectrum this sits on. |
| **Where it fails this guide's subject** | It reproduces the notebook's core weakness without the notebook's one mitigation: there is no file at all, so there is not even a document to review afterwards. The transcript is the artefact, and the transcript is not a run record. |

The important structural point: **a code interpreter is a kernel with a different lifetime policy.** It has the same namespace semantics, the same laziness exposure, the same sampled observation. What changes is that the session is disposable, which helps reproducibility (clean state) and hurts auditability (no durable artefact).

### 9.3 Category two: the kernel-protocol client

A client that speaks the Jupyter messaging protocol directly to a real kernel — the five sockets, `execute_request`, `execute_reply`, IOPub broadcasts. This is what an agent integration looks like when it drives a notebook rather than a sandbox.

| | |
| --- | --- |
| **What it can observe** | Everything the protocol carries: the reply status per request, the `execution_count` per execution, stream and display output on IOPub, `busy`/`idle` bracketing, and its own send order — which is the closest thing to an execution log that exists anywhere in the stack. |
| **What it cannot observe** | Anything the protocol does not carry: data versions, configuration, plans, cell dependencies, skipped cells (§3.6). It also cannot see what another attached frontend did, beyond the state it left behind (§8.1). |
| **What it is structurally good at** | Recording order. A client that logs every request it sends, with the code and the reply, has the executed cell sequence that nbformat lacks — for its own requests. This is the one instrument in §5.3 that closes the *order* axis, and it closes it only for the agent's own traffic. |
| **Where it fails this guide's subject** | The record is the client's, not the notebook's. If the client is not the system of record — if the notebook is what gets attached to the ticket — the order is lost at exactly the moment it is needed. The client also inherits the kernel's statefulness in full: it is driving a shared namespace, and it has no more insight into prior bindings than a human would. |

### 9.4 Category three: notebook-native execution

A runner that takes a notebook file and executes it as a unit — all cells, in file order, in a fresh kernel, writing an output notebook. Sometimes with parameters injected at a declared cell.

| | |
| --- | --- |
| **What it can observe** | The file order *is* the execution order by construction, which makes the cell sequence (§7.2a) trivially recoverable and eliminates the re-run and out-of-order failure modes entirely. Failures are attributed to the cell that raised. |
| **What it cannot observe** | Anything about the cluster beyond what the cells print; the runner does not add Spark awareness. Nor does it capture versions or snapshots — it fixes the *order* axis and leaves the other three as they were. |
| **What it is structurally good at** | Converting an exploration artefact into a repeatable one. Run-all-in-a-fresh-kernel is the procedure §7.4 requires, made into a mechanism rather than a discipline. This is the single largest reproducibility gain available without changing anything about Spark. |
| **Where it fails this guide's subject** | It is a batch execution model wearing a notebook's clothes. The interactivity that made the notebook useful to the agent — write a cell, look, write the next — is gone; an agent driving a run-all runner is really submitting jobs and reading logs, and should be designed as such. It also does not stop an agent from having explored interactively first and then written a tidy notebook that reproduces a *different* computation from the one that produced the number. |

### 9.5 Category four: platform-native execution

The notebook as a first-class object on a managed platform: attached to a managed cluster or a serverless compute plane, schedulable, permissioned through a catalog, with job-level run history and cluster logs held by the platform. Some platforms also offer a *declarative* path — SQL or a pipeline definition — that never exposes a kernel at all.

| | |
| --- | --- |
| **What it can observe** | Run-level metadata (who, when, which notebook version, success/failure), cluster logs, and — if event logging is enabled — a History Server view of the Spark application after it exits (Spark 4.2.0 docs, Web UI, retrieved 2026-10-19). Permissions are enforced at the catalog, so the agent's reach is bounded by a grant rather than by prompt instructions. |
| **What it cannot observe** | Cell-level intent. Platform run history says the notebook ran; it does not say the agent re-ran cell 9 twice and skipped cell 5. The kernel semantics are unchanged by being managed — the platform documents per-REPL isolation and implicit bindings like `_sqldf` precisely because they still apply (Databricks docs, last updated 2026-10-13). |
| **What it is structurally good at** | Durability of evidence. The run record, the cluster log and the event log outlive the session, which the Web UI does not (Spark 4.2.0 docs, Web UI, retrieved 2026-10-19). For a regulated institution this is the difference between an investigation and a guess. |
| **Where it fails this guide's subject** | The platform holds evidence the agent cannot read. Run history is a UI and an API, not a message on the agent's feedback channel; an agent that could query it would be doing something no documented agent integration does by default. The governed surface is the platform guide's subject — [`databricks_agents_guide.md`](databricks_agents_guide.md) — and is not re-described here. |

### 9.6 The structural summary

| Category | Order captured? | Cluster visible? | Durable artefact? | State isolated? |
| --- | --- | --- | --- | --- |
| Code-interpreter runtime | Only as transcript | No | Weak (transcript) | Yes, by disposal |
| Kernel-protocol client | Yes, for its own requests | No | No (unless the client persists it) | No — shared kernel |
| Notebook-native execution | Yes, by construction | No | Yes (output notebook) | Yes, fresh kernel |
| Platform-native execution | Partly (run level, not cell level) | Yes, via logs and History Server | Yes (run history + event log) | No — session semantics unchanged |

No row is all four. That is the honest state of the tooling: the properties are distributed across categories, and an institution that needs all four is assembling them, not buying them.

---

## 10. The regulated angle: reproducibility as an audit question

### 10.1 The question underneath the question

A supervisor, an internal auditor or an external reviewer does not ask "was the agent reliable?". That question has no test. They ask a question that does have a test:

> **Can the number be reconstructed by someone who was not there?**

Everything in §7 exists to make that question answerable. The framing matters because it converts an argument about AI trust — which cannot be settled — into an argument about evidence, which can.

### 10.2 The five questions a reviewer will actually ask

They arrive in this order, and each one has a specific artefact that answers it or does not.

| # | Question | Answered by | Answered by a default notebook? |
| --- | --- | --- | --- |
| 1 | **Who ran it, and what did they run?** | The cell sequence plus the reviewer/publisher names in the run record | No. The notebook has no author field per cell that is trustworthy, and no executor field at all. |
| 2 | **In what order?** | The executed cell sequence, including re-runs and skips | No. `execution_count` is optional and resets on restart (nbformat v4.5 schema, retrieved 2026-10-19); timestamps are optional and record message arrival, not work. |
| 3 | **On which version of the engine, with which settings?** | Spark version and the full effective configuration | No. The format has no field for either. |
| 4 | **Over which data?** | A pinned table version or snapshot identifier, with the data owner's certification | No. A table name is a mutable pointer; the notebook records the name, not the version. |
| 5 | **With what plan — and did the plan change?** | The pre-execution `explain()` output plus the post-run SQL tab plan from the event log | No. Neither is captured by the format, and the UI that held the second one is gone when the application exits (Spark 4.2.0 docs, Web UI, retrieved 2026-10-19). |

A "no" on any of the five is not a finding of wrongdoing. It is a finding that the number is **attested** rather than **evidenced** — it rests on the operator's say-so. Institutions accept attestation for low-stakes figures all the time, and that is a legitimate choice. The failure is accepting attestation *while believing you hold evidence*, which is what a notebook with saved outputs looks like from a distance.

### 10.3 The reconstruction test

The practical control is a test any team can run in an afternoon:

1. Take the published number and the notebook that produced it.
2. Hand both to an analyst who was not involved, with no access to the original session.
3. Ask them to reproduce the number, and to write down every assumption they had to make.

The output of the test is not the number — it is **the list of assumptions**. If the list contains "which version of the table", "which cells actually ran", or "which Spark version", the artefact is insufficient, and the list is precisely the content the run record in §7.4 should have carried. A team that runs this test once will design its next notebook differently.

### 10.4 Requirements, derived rather than copied

These are requirements this guide's argument generates. They are not a transcription of any framework, and they are deliberately expressed as properties rather than as controls to tick.

- **R1 — Every published figure carries its snapshot.** A figure without a version identifier is not published; it is circulated. The version is part of the number, not metadata attached later.
- **R2 — Every agent-authored cell is reviewed as text before it runs against shared state.** The review is the control; a live agent typing into a session is unreviewable by construction.
- **R3 — Restart-and-run-all precedes any figure that leaves the team.** This is the only procedure that makes file order equal execution order, and it is the cheapest available fix (§7.4).
- **R4 — A performance claim carries a plan.** Not a duration; a plan and a duration together. A duration alone is unattributable (§5.1).
- **R5 — The evidence outlives the session.** Event logging on before the run, run record stored with the notebook, transcript retained for agent sessions. The UI's lifetime is the application's (Spark 4.2.0 docs, Web UI, retrieved 2026-10-19); the evidence's lifetime must be the record's.
- **R6 — Accountability resolves to a person, per figure.** The publisher, by name, in the run record. §6.2.

### 10.5 What this is not

It is not a claim that agents should not touch regulated data. The controls above are the same controls a human analyst's notebook needs, and an agent that operates inside them is no less defensible than a junior analyst who does. What is different is the *rate*: an agent produces cells faster than a human can review them, and it produces them in a session where re-runs are cheap, so the volume of unreviewed state grows faster than the review process does. The requirement is not a ban; it is a gate that scales with the agent's throughput — which, in practice, means the agent must produce its work as reviewable cells with a run record, not as keystrokes in a shared kernel.

---

## 11. The failure modes

### 11.1 The table

Eight modes, each observed in the mechanism rather than the abstract. The guardrail column is what would have caught it, and every guardrail is implementable with the material in §6 and §7.

| # | Symptom | Cause | Guardrail |
| --- | --- | --- | --- |
| 1 | The agent reports "the data is cleaned / filtered / joined" and the next cell fails on a column that should exist | **A lazy transform the agent believed ran.** The transformation returned a DataFrame handle; no action executed it (Spark 4.2.0 docs, RDD Programming Guide, retrieved 2026-10-19) | Require an action per claim, named in the report: `clean.count()` against `raw.count()`. A claim without an action is rejected by the harness, not by the reviewer. |
| 2 | A number changes between two runs of the "same" notebook, and nothing in the code changed | **A stale DataFrame after an upstream change.** The table was rewritten by another job; the session held a cached frame or a temp view bound before the rewrite | Pin the table version at read time (§7.2c) and re-read rather than reusing a cached frame for a published figure. Record the version in the run record. |
| 3 | The published figure differs from the exploratory figure, and both "came from the same notebook" | **A cell re-run in a different order.** The second run followed a different path through the namespace, so a name held a different object at the moment of the action | Restart-and-run-all before publishing (R3). The run record's cell sequence makes divergence visible instead of silent. |
| 4 | The kernel dies during inspection, taking the session's cached work with it | **An unbounded `collect()`.** "all the data is loaded into the driver's memory" (PySpark 4.2.0 API docs, retrieved 2026-10-19) | Ban unguarded `collect()`. Use `count()`/`agg()` for questions and `take(n)`/`show()` for inspection; permit `collect()` only after a `count()` proves the frame small. |
| 5 | A column that was a string is now a double, or a new enum value appears, and downstream arithmetic shifts | **Silent schema drift.** The read is permissive; the schema is inferred from the data at read time, so a changed source changes the frame without erroring | Assert the schema explicitly (`StructType`) at the boundary; fail the cell on mismatch rather than letting inference absorb the change. Log the schema in the run record for published figures. |
| 6 | The number cannot be found again; the notebook has an output but no cell that produces it | **A result that exists only in a cell nobody saved.** The cell was edited after running, or ran in a frontend that did not save, or was deleted from the file while its output persisted in another cell's text | Save before publishing, and store the run record beside the notebook. Treat an output whose producing cell is absent as no result at all. |
| 7 | A cell that worked yesterday raises `AnalysisException: cannot resolve` today, with no code change | **A rebind by another cell or another frontend.** A later cell replaced the name, or another attached frontend did (§4.1, §8.1) | Bind from source in the cell that uses it; namespace agent-owned names; list bound names at handover (§8.3). |
| 8 | A timing claim cannot be reproduced and nobody can say why | **A performance claim without a plan.** The plan read before execution differed from the plan that ran after runtime re-optimisation (Spark 4.2.0 docs, SQL Performance Tuning, retrieved 2026-10-19) | Capture both plans when a performance claim is made; enable event logging before the run so the SQL tab survives the application (Spark 4.2.0 docs, Web UI, retrieved 2026-10-19). |

### 11.2 The pattern across the eight

Five of the eight (1, 3, 6, 7, 8) are failures of *record*, not failures of *computation*. The Spark job did exactly what it was told; what failed was the connection between the number and its provenance. Two (2, 5) are failures of *input identification* — the computation was right about data that was not the data. One (4) is a resource failure with an obvious guardrail.

That distribution is the argument for §7. An institution that invests only in making the Spark correct — better tests, better code review of the SQL — addresses one of the eight. The other seven are addressed by capturing what happened.

---

## 12. The Cymbal Bank worked example

> **This section is explicitly fictional and illustrative.** Cymbal Bank is an invented institution; the analyst, the agent and the dataset are invented; the figure in it is arithmetic chosen to make a point, not a measurement of anything. No real bank, no real dataset and no real system is described. The only claim the section makes is about *evidence*, and it is a claim about the example.

### 12.1 The setting

A payments analyst at Cymbal Bank is asked a simple question before a steering meeting: **what share of card payments failed last month, and is it moving?** The data is a `payments` table in the lakehouse, written daily by an ingestion job, at roughly a hundred million rows a month. The analyst has an agent attached to their notebook — the arrangement this guide describes — and asks it to work out the rate.

The agent does what a competent analyst would do. It reads the table, filters to last month, classifies each row by a status column, and computes a ratio. Eleven cells, four of which it re-runs after errors. The number it prints is **3.81%**. The steering deck is due in an hour.

### 12.2 What happened, cell by cell

The trace below is the kind of record the notebook does *not* contain, reconstructed here so the reader can see where the number came from.

| Cell | What the agent wrote | What actually happened |
| --- | --- | --- |
| 1 | `import` block, `spark` handle | Fine. The `SparkSession` already existed; the agent attached to it. |
| 2 | `payments = spark.read.table("payments")` | A lazy read. No data touched — and not even the version was fixed: `payments` was at version 41 when the line was written, but which version it would read was decided later, when the first action ran. |
| 3 | `payments.printSchema()` | Schema printed — one column that should be `decimal(18,2)` came back as `string`. The agent did not comment on it. |
| 4 | `last_month = payments.filter(col("txn_date") >= "2026-09-01")` | Another lazy transform. Returns in 40 ms. |
| 5 | *(skipped — the agent changed approach)* | Nothing. The cell still sits in the file with no output. |
| 6 | `last_month.createOrReplaceTempView("payments_lm")` | Registers a temp view over the lazy frame. State mutation, no execution. |
| 7 | `spark.sql("SELECT status, COUNT(*) c FROM payments_lm GROUP BY status").show()` | **First real action.** Output: six status values, counts in the low millions, one value `FAILED_PAYME…` truncated by `truncate=True`. |
| 8 | *(skipped)* | Nothing. |
| 9 | A corrected SQL cell joining a `status_map` lookup table | First attempt failed on a type mismatch: the string column in cell 3 versus the decimal in the map. Re-run after a `cast`. |
| 10 | `spark.sql("SELECT COUNT(*) ... WHERE status='FAILED'")` → `.collect()[0][0]` | Returns 1,412,908. An action, and a `collect()` on a one-row aggregate — safe, because the aggregate is one row. |
| 11 | `rate = failed / total`, printed as `3.81%` | Arithmetic in Python over two scalars. |

The number is not obviously wrong. It is a ratio of two counts, computed by the cluster, over a filtered frame. The failure is not arithmetic; it is that **nothing in the notebook says which data, which engine, or which cell order produced it** — and the agent's own report says none of those either.

### 12.3 What the number is worth

Ask the reconstruction question from §10.1: *can a second person reproduce 3.81%?*

| Requirement | Status in this notebook | Consequence |
| --- | --- | --- |
| Executed cell sequence | Counters exist, out of order, and cells 5 and 8 are silent | A reproducer cannot tell whether 5 ran before 7 and was abandoned, or never ran. |
| Spark version and configuration | Not recorded | A different Spark version, or AQE on versus off, changes partition counts and possibly the plan — and the reproducer has no baseline to match. |
| Data snapshot | "payments", no version | The ingestion job rewrites the table nightly. By the time of the meeting, `payments` is at version 42; 3.81% may be a figure about data that no longer exists. |
| Physical plan | None captured — and no performance claim was made, so this one is *not* a defect here | The figure is a count, not a timing. The plan matters for the *cost* of recomputation, not for the number's truth. |

Two of the three applicable captures are missing and the third is degraded. The honest valuation is: **the number is a well-formed arithmetic result over an unidentified dataset on an unidentified engine, produced by an order nobody recorded.** It might be correct. It cannot be shown to be correct, and the difference between those two sentences is the entire subject of this guide.

There is also a quieter problem the trace reveals. Cell 7's output shows a status value truncated at twenty characters — `FAILED_PAYME…`. Cells 9 to 11 filter on `status='FAILED'`. If the taxonomy contains both `FAILED_PAYMENT` and `FAILED_PAYMENT_REVERSED`, the filter in cell 10 counts one of them, and the agent's evidence — a truncated prefix — could not have distinguished them. Whether it did is unknowable from the notebook, because the cell that would show the full values was never written.

### 12.4 The decision the notebook-agent nature changed

The analyst has two options at 09:40, with the deck due at 11:00.

**Option A — publish 3.81% with a footnote.** Fast, plausible, and unreconstructible. If the number is challenged in the meeting — and a failed-payment rate that moved is exactly the kind of number that gets challenged — the analyst has no answer to "over which data?", because the table has been rewritten since.

**Option B — recompute from a pinned snapshot with the sequence captured.** Read `payments` at version 41 explicitly, assert the schema at the boundary, collapse the computation into a single reviewed cell that binds its own inputs, run it in a fresh kernel, capture the counts and the configuration in a run record, and publish the run record with the figure. It takes forty minutes of the hour.

The analyst takes Option B, and the reason is specific to the *agent-driven* nature of the session rather than to the stakes: the analyst cannot personally attest to the order the cells ran in, because they did not type them. A human analyst who had built the notebook by hand would have a memory of the sequence and could vouch for it. This analyst watched an agent work, approved cells as they appeared, and genuinely does not know whether cell 5 ran. That is not a criticism of the analyst or the agent; it is the structural consequence of §8.1 — the human was a second frontend on a kernel they did not solely control.

So the decision changes in a way that is small and consequential: **the analyst stops treating the notebook as the record and starts treating the run record as the record.** The notebook is attached as supporting material; the run record is what the figure rests on. The figure that reaches the deck is 3.79%, not 3.81%, and the reason is trap two rather than arithmetic: the exploratory run's read was lazy, so `payments` resolved to whatever version existed when cell 7 finally executed — by then version 42, with the nightly ingestion half-applied — whereas the run record pins version 41. The difference is two basis points and the difference in defensibility is total: one number can be reconstructed by a reviewer with the run record, and the other cannot be reconstructed by anyone, because the snapshot it was computed over no longer exists under that name.

### 12.5 The thesis, applied

Everything the agent did in this example was competent. It wrote valid Spark, it used an aggregate rather than a `collect()` of the frame, it checked a schema (and missed what it saw), it fixed a type error correctly. The failure was not in the writing. It was in the knowing: the agent could not tell that cell 7's truncated output hid a taxonomy question, could not tell that its cached view had outlived the data it described, and could not tell that its own report omitted the three facts a reviewer would need.

That is the shape of the problem in every institution that will deploy this arrangement. The writing improves with every model generation. The knowing does not improve with the model, because it is not a model problem — it is a problem of what the session records, what the agent can observe, and what the human is willing to sign.

**An agent that writes Spark is easy; an agent that knows what its Spark did is the hard part.**

---

## 13. The anti-patterns

Each of these has a name because a name is what lets a reviewer say "that is the victory lap" in a design review instead of arguing about intent. Each is listed with why it is tempting — none of them is tempting for a stupid reason — and what it costs.

### 13.1 The victory lap

**What it is.** The agent runs a transformation, sees `execute_reply: ok`, and reports success. The cell printed nothing, which the agent reads as "no errors".

**Why it is tempting.** The signal really is a success signal. There is no error, the call returned, and the DataFrame has the right schema. Everything the agent can see agrees with the report.

**What it costs.** A false completion is reported to a human who now believes data was processed. The failure surfaces later, at the next action, attributed to a different cell — and by then the human has repeated the claim. This is the most common failure in the guide and the cheapest to fix: an action per claim (§4.2).

### 13.2 The twenty-row census

**What it is.** Inferring data properties from `show()` — "there are about six status values", "the null rate looks low", "the amounts are all positive".

**Why it is tempting.** It is one call, it is fast, and the output is legible. It is also the only inspection primitive many agent harnesses expose.

**What it costs.** The inference is unsupported by construction: a prefix of unknown distribution with strings truncated at 20 characters (PySpark 4.2.0 API docs, retrieved 2026-10-19). Cardinality, null rate, skew and tails are exactly the properties a prefix cannot show, and they are exactly the properties that change a number. §12's truncated `FAILED_PAYME…` is this anti-pattern costing two basis points and a taxonomy error.

### 13.3 The collect of last resort

**What it is.** The agent cannot get the shape it wants from SQL, so it calls `collect()` and does the rest in Python.

**Why it is tempting.** It works in development, on the sample. Python is more expressive than SQL for the specific transformation the agent is improvising, and the agent is very good at Python.

**What it costs.** "all the data is loaded into the driver's memory" (PySpark 4.2.0 API docs, retrieved 2026-10-19). On a large frame this is an OOM that kills the kernel and takes every cached frame, temp view and unsaved result with it. The blast radius is the whole session, not the cell.

### 13.4 Cell roulette

**What it is.** The cell errored; the agent edits it and re-runs. It errored again; edit and re-run. Eventually it returns ok, and the agent moves on.

**Why it is tempting.** Iterating against an error message is a legitimate and effective strategy for writing code. It is how the model was trained to debug.

**What it costs.** Every iteration may have applied a side effect before failing (§2.2). The successful run operates on state the failed runs modified, and the notebook records only the success. The final cell is not the cell that was designed; it is the survivor of a sequence of partial applications, and nobody — including the agent — can enumerate them.

### 13.5 The ghost binding

**What it is.** The agent uses a name it did not bind in the current session — `df`, `total`, `_sqldf`, a temp view — because it appeared in the notebook's earlier cells or in the agent's context.

**Why it is tempting.** The name is right there in the file. Reading the notebook top to bottom, the binding looks established.

**What it costs.** The name may have been rebound by a cell that ran later, by another frontend, or by an implicit binding that reassigns on every SQL cell (Databricks docs, last updated 2026-10-13). The agent's computation then reads an object it did not choose, and the cell text gives no indication. This is the anti-pattern with the worst ratio of likelihood to detectability: nothing fails, and the number is about something else.

### 13.6 The notebook as evidence

**What it is.** The `.ipynb` is attached to the ticket, the change request, or the model documentation as the record of how the number was produced.

**Why it is tempting.** It is the artefact that exists. It contains the code, and it contains outputs that look like results. Producing anything else feels like duplicate work.

**What it costs.** The file carries no Spark version, no configuration, no table version, no plan, no record of skipped cells, and at most four optional timestamps per cell (nbformat v4.5 schema, retrieved 2026-10-19). It is a document that looks like a log. A reviewer who treats it as a log will accept a number that cannot be reconstructed, and will not know that is what they did.

### 13.7 The config by vibe

**What it is.** The agent sets `spark.conf.set(...)` mid-session to make something work — a shuffle partition count, a broadcast threshold, a timezone — and does not record it.

**Why it is tempting.** Session configuration is mutable and the change is invisible in the cell's output. It is the shortest path past an error or a slow stage.

**What it costs.** The setting persists for every subsequent computation in the session, including computations the agent is not thinking about, and it is not in the notebook's record. A reproducer running the same cells on defaults gets a different plan and possibly a different number, with no clue why. §7.2b exists for this.

### 13.8 The silent schema

**What it is.** The read is permissive, the schema is inferred, and the agent proceeds. A column arrives as `string` that should be `decimal`, or a new enum value appears.

**Why it is tempting.** Inference almost always works, and an explicit schema is boilerplate the agent can skip. The frame is usable either way.

**What it costs.** Drift is absorbed rather than surfaced: comparisons become string comparisons, `sum()` on a string column fails late or casts silently, and a new category either joins the denominator or does not depending on how the filter was written. The failure is not an exception; it is a slightly different number (§11 row 5).

### 13.9 The pattern

Six of the eight anti-patterns are ways of **treating an observation as a conclusion**. The victory lap treats a reply as completion; the census treats a prefix as a distribution; the ghost binding treats a name as a value; the notebook-as-evidence treats a document as a log; cell roulette treats a successful re-run as a clean one; the silent schema treats inference as fact. Only the collect and the config are different in kind — resource and configuration hazards rather than epistemic ones.

That ratio is the guide's argument in miniature. The interesting failures of agent-written Spark are not crashes. They are confident reports about things the agent had no way to see.

---

## 14. The state of practice

### 14.1 What is demonstrably supported

Claims in this column are supported by a document that says so, and the document is cited.

| Capability | Status | Evidence |
| --- | --- | --- |
| A kernel executes requests in arrival order against one namespace, with busy/idle bracketing | **Documented protocol semantics** | Jupyter messaging protocol 5.5 (retrieved 2026-10-19) |
| A single kernel can serve multiple frontends simultaneously | **Documented** | Same |
| `execution_count` is a per-kernel counter that resets on restart | **Documented** | Same |
| Notebook state is per-REPL, not per-notebook, and implicit bindings like `_sqldf` are reassigned on every SQL cell | **Documented by the vendor** of the notebook product in question | Databricks docs, last updated 2026-10-13 |
| A parallel SQL cell runs in a new session without temp views, UDFs or `_sqldf` | **Documented** | Same |
| Spark transformations are lazy and compute only on an action | **Documented, and stated as a design property** | Spark 4.2.0 docs, RDD Programming Guide, retrieved 2026-10-19 |
| `show()` defaults to 20 rows with 20-character truncation | **Documented in the API signature** | PySpark 4.2.0 API docs, retrieved 2026-10-19 |
| `collect()` loads all records into driver memory | **Documented, with a warning** | Same |
| `explain()` prints logical and physical plans without executing the data path, with `cost` statistics "if they are available" | **Documented** | Same |
| AQE re-optimises at runtime | **Documented** | Spark 4.2.0 docs, SQL Performance Tuning, retrieved 2026-10-19 |
| The Web UI is at port 4040, dies with the application, and the History Server reconstructs it from an event log | **Documented** | Spark 4.2.0 docs, Web UI, retrieved 2026-10-19 |
| nbformat v4.5 carries an optional per-cell `execution` object with four ISO timestamps and an `execution_count` | **Documented in the schema** | nbformat v4.5 schema, retrieved 2026-10-19 |
| No data is shared across `SparkContext`s without external storage | **Documented** | Spark 4.2.0 docs, Cluster Mode Overview, retrieved 2026-10-19 |

That is a substantial list, and it is the reason this guide can make precise claims instead of gesturing. **Every mechanism in §3, §4 and §8 is documented behaviour, not inference.** The argument does not rest on speculation about what agents do; it rests on what the execution model is.

### 14.2 What is promise

Claims in this column are things the arrangement would need and no document in this pass promises.

| Capability | Status | Why it is not established |
| --- | --- | --- |
| An agent that can tell whether its own transformation executed | **Not demonstrated** | Requires an action, which the agent must choose to run. No protocol message reports "this plan has been executed" (§3.6). |
| A notebook format that records the executed cell sequence | **Not in the format** | The schema carries an optional counter and timestamps, not a sequence with re-runs and skips (§7.3). A client can record it; the file does not. |
| A notebook format that records data version, engine version or plan | **Not in the format** | No such fields exist in nbformat v4.5. |
| An agent that reads the Spark UI or event log as part of its feedback loop | **Not documented as a capability of any integration reached** | The UI is a web surface for humans; the event log is a file for the History Server. Neither is a message on the agent's channel. |
| Automatic detection of stale state after an upstream table change | **Not documented** | Version pinning is a discipline the reader applies, not a signal the session emits (§7.2c). |
| A reproducibility guarantee for an agent-driven notebook | **Not claimed anywhere, correctly** | The format has no fields for it and the execution model has no transactions (§2.2). |

The gap between the two tables is the guide's practical finding: **the mechanics are documented and the knowing is not.** Every fact needed to explain *why* an agent cannot see what its Spark did is in a vendor or project document. No document describes a system that closes the gap.

### 14.3 What would count as evidence of progress

Three observable developments would move a capability from the second table to the first, and none of them requires a new model:

1. **A run-record format adopted by a notebook client** — the executed sequence, the engine version, the effective configuration, the table versions, and the plans, written as structured data alongside the notebook. The mechanics exist (§7.4); the adoption does not.
2. **An agent-facing read of the event log** — so that the feedback loop includes the per-stage facts that currently live only in a UI. The event log already contains them (Spark 4.2.0 docs, Web UI, retrieved 2026-10-19); nothing exposes them as a tool result.
3. **A convention for binding at point of use** — the discipline in §4.1, enforced by the harness rather than left to the prompt. It costs nothing and removes the ghost-binding class entirely.

None of the three is a product announcement. All three are engineering decisions available today, which is why the guide's recommendations are disciplines rather than purchases.

---

## 15. The claims audit

Every claim the guide relies on, with its source type and date. Dates are the retrieval date (2026-10-19) except where the source carries its own date: the Spark 4.2.0 release (2026-07-14) and the Databricks page's stated last update (2026-10-13).

| # | Claim | Source | Source type | Date |
| --- | --- | --- | --- | --- |
| 1 | Spark transformations are lazy and "only computed when an action requires a result to be returned to the driver program" | Apache Spark 4.2.0 docs, RDD Programming Guide | Project primary | 2026-10-19 |
| 2 | Accumulator updates are not guaranteed within a lazy transformation, confirming laziness is semantic | Same page | Project primary | 2026-10-19 |
| 3 | Spark 4.2.0 was released 2026-07-14 | spark.apache.org news index | Project primary | 2026-07-14 (release) / 2026-10-19 (retrieval) |
| 4 | A Spark application is coordinated by the `SparkContext` in the driver, which acquires executors and sends tasks | Apache Spark 4.2.0 docs, Cluster Mode Overview | Project primary | 2026-10-19 |
| 5 | Executor / task / job / stage definitions as quoted in §4.4 | Same page, glossary | Project primary | 2026-10-19 |
| 6 | Data cannot be shared across `SparkContext`s without external storage | Same page | Project primary | 2026-10-19 |
| 7 | The Web UI is built into every application, defaults to port 4040, and is tied to the application's lifetime | Apache Spark 4.2.0 docs, Web UI | Project primary | 2026-10-19 |
| 8 | The History Server reconstructs an equivalent UI from a persisted event log | Same page | Project primary | 2026-10-19 |
| 9 | The SQL tab's execution detail carries parsed logical, analyzed logical, optimized logical and physical plans | Same page | Project primary | 2026-10-19 |
| 10 | AQE re-optimises at runtime (coalescing, skew splitting, join-strategy conversion) | Apache Spark 4.2.0 docs, SQL Performance Tuning | Project primary | 2026-10-19 |
| 11 | `show()` defaults to `n=20` and `truncate=True` (strings over 20 chars truncated) | PySpark 4.2.0 API, `DataFrame.show` | Project primary (API reference) | 2026-10-19 |
| 12 | `collect()` returns all records as a list and loads all data into the driver's memory | PySpark 4.2.0 API, `DataFrame.collect` | Project primary (API reference) | 2026-10-19 |
| 13 | `head()`/`take()` return the first n rows | Same page, "See also" | Project primary (API reference) | 2026-10-19 |
| 14 | `explain()` prints logical and physical plans; `extended=False` prints only physical; `mode="cost"` prints statistics "if they are available" | PySpark 4.2.0 API, `DataFrame.explain` | Project primary (API reference) | 2026-10-19 |
| 15 | A single kernel can be simultaneously connected to one or more frontends | Jupyter messaging protocol 5.5 | Project primary (protocol specification) | 2026-10-19 |
| 16 | Shell is a single ROUTER socket carrying execution requests and replies; IOPub is the broadcast channel for side effects and status; Control avoids queueing behind execution and carries shutdown/restart | Same document | Project primary (protocol specification) | 2026-10-19 |
| 17 | The execution lifecycle is `_request` → `status: busy` → reply → `status: idle` | Same document | Project primary (protocol specification) | 2026-10-19 |
| 18 | `silent=True` forces `store_history=False`; `store_history` defaults to True when `silent` is False; the counter populates `execution_count` on replies and inputs | Same document, `execute_request` fields | Project primary (protocol specification) | 2026-10-19 |
| 19 | nbformat v4.5 defines an optional per-cell `execution` object with four ISO-8601 timestamps (`iopub.execute_input`, `iopub.status.busy`, `shell.execute_reply`, `iopub.status.idle`) and no fields for engine version, configuration, data version, plan or skipped cells | nbformat v4.5 JSON schema | Project primary (format specification) | 2026-10-19 |
| 20 | Notebook state is per-REPL; variables are isolated between language REPLs; REPLs share state only through external resources | Databricks docs, "Develop code in Databricks notebooks" | Vendor primary | Last updated 2026-10-13 |
| 21 | `_sqldf` is an implicit DataFrame from a SQL cell, available to later Python and SQL cells, and "is reassigned each time a SQL cell is run" | Same page | Vendor primary | Last updated 2026-10-13 |
| 22 | A parallel SQL cell runs in a new session, where temp views, UDFs and `_sqldf` are not supported, and default catalog/database names are used | Same page | Vendor primary | Last updated 2026-10-13 |
| 23 | The string "AgentSpark" is used by five unrelated companies, none of them an Apache Spark, Jupyter or notebook product; treated as three technical terms, not a product | Dispatcher identity check (`web_search` on the phrase) | Secondary, names only | 2026-10-19 |
| 24 | **No benchmark, no vendor ranking, no adoption statistic and no cost or latency figure is printed anywhere in this guide**, because the brief forbids them and none would have been defensible from the sources reached; the only numbers in the text are the documented API defaults (20 rows, 20-character truncation, port 4040) and the explicitly fictional figures in §12 | This guide, self-audit | Editorial ruling | 2026-10-19 |

### 15.1 Notes on the audit

- **The load-bearing quote is row 1.** If laziness were an optimisation rather than a semantic property, §4.2 and §11 row 1 would collapse. The documentation states it as a design property and the accumulator note confirms it is not bypassable.
- **Row 23 is deliberately minimal.** The identity check is recorded because it explains why the guide has no product section, and it names no product. The five sites were not re-opened in this pass (§16).
- **Row 24 is the constraint made visible.** The brief bans figures, and the temptation with a subject like this is to reach for one — a speed-up, a percentage of teams, a cost per thousand jobs. None would have been supportable, and a guide that argues about evidence has no business printing unevidenced numbers of its own.

---

## 16. What Could Not Be Verified, glossary and cross-references

### 16.1 What could not be verified

Recorded as explicit negatives, because the temptation to fill them with inference is exactly what this guide warns against. Nothing here is a negative about Spark, Jupyter or any platform; it is a statement about the evidence reached in this pass.

1. **The five "AgentSpark" sites were not independently re-opened.** The identity finding in §1.3 is a dispatcher check — names and one-line descriptions, secondary, 2026-10-19 — and the brief instructed the guide to state the finding and move on. It is not a claim about any of the five companies beyond the fact that the string is not a Spark or notebook product.
2. **The Jupyter and nbformat documentation hosts failed to scrape.** `jupyter-client.readthedocs.io` (both `/en/stable/` and `/en/latest/`) and `nbformat.readthedocs.io` returned "Internal Server Error: All scraping engines failed" on this host. The raw GitHub sources were used instead: the `jupyter_client` `docs/messaging.rst` (protocol 5.5) and the `nbformat` v4.5 JSON schema. Both are authoritative — the messaging document describes itself as "the authoritative description of the Jupyter messaging protocol" — but they are the repository text, not the rendered documentation site, and a reader checking a quote will land on a different page than the one cited.
3. **No independent measurement of how often agents produce stale-state errors exists in the sources reached.** The guide's claims about agent behaviour (re-running on error, rebinding rather than editing, restarting readily) are structural arguments from the execution model, not measurements. No study, telemetry or incident dataset was located, and the brief forbids adoption statistics in any case. The argument does not depend on frequency: it depends on the failure being undetectable from inside the notebook, which is a property of the mechanism.
4. **The exact behaviour of every notebook client with respect to the `execution` object is not established.** nbformat v4.5 makes it optional; which clients write it, which strip it, and what they write on re-save was not surveyed. The guide therefore says "at most four timestamps, if the client writes them" rather than asserting a default.
5. **Whether any agent integration reads the Spark event log or the Web UI as a tool result was not established.** No documentation for such a capability was located in this pass. The guide says "not documented as a capability of any integration reached" and does not say "no such integration exists".
6. **Table-version semantics were not verified in this pass.** The guide treats a pinned version as an available discipline and cites no format-level specification for it, because the storage and table-format mechanics belong to [`../data/on_prem_lakehouse_guide.md`](../data/on_prem_lakehouse_guide.md) and [`../cloud_object_storage_lakehouse_guide.md`](../cloud_object_storage_lakehouse_guide.md). The claim made here is narrow: that a figure should name a version. How the version is stored is out of scope and unverified here.
7. **No measurement of the reconstruction test in §10.3 was performed.** The test is proposed as a control, not reported as an experiment. No team was observed running it, and no result from such a run is claimed.

### 16.2 Glossary

| Term | Meaning in this guide |
| --- | --- |
| **Action** | An operation requiring a result to be returned to the driver (`count`, `collect`, `show`, `write`). Only an action causes computation (Spark 4.2.0 docs, RDD Programming Guide, retrieved 2026-10-19). |
| **AQE (adaptive query execution)** | Runtime re-optimisation of the physical plan using statistics available only after stages run. The reason a plan read before execution may differ from the plan that ran. |
| **Busy / idle** | The IOPub status bracket around a single execution request. Not a progress indicator and not a measure of cluster work. |
| **Cell** | One unit of source text submitted as one execution request. No declared inputs, no declared outputs, no enforced position in an execution order. |
| **Code-interpreter runtime** | A process that accepts a snippet, executes it, and returns output and status; usually disposed at the end of a session (§9.2). |
| **`collect()`** | Returns all records as a list, loading all data into driver memory (PySpark 4.2.0 API docs, retrieved 2026-10-19). |
| **Driver** | The process running the `SparkContext`, coordinating the application (Spark 4.2.0 docs, Cluster Mode Overview, retrieved 2026-10-19). In a notebook, the kernel process. |
| **Event log** | The persisted record from which the History Server reconstructs a UI after the application exits (Spark 4.2.0 docs, Web UI, retrieved 2026-10-19). |
| **`execution_count`** | A monotonic per-kernel counter of executions, reset on restart; not a dependency graph and not a data version. |
| **Executor** | "A process launched for an application on a worker node, that runs tasks and keeps data in memory or disk storage across them" (Spark 4.2.0 docs, retrieved 2026-10-19). |
| **`explain()`** | Prints logical and physical plans without executing the data path; `mode="cost"` includes statistics "if they are available" (PySpark 4.2.0 API docs, retrieved 2026-10-19). |
| **History Server** | A service that serves an equivalent UI from a persisted event log after the application has exited. |
| **Job** | "A parallel computation consisting of multiple tasks that gets spawned in response to a Spark action" (Spark 4.2.0 docs, retrieved 2026-10-19). |
| **Kernel** | The process that executes cells: one namespace, one ordered request queue, five sockets. |
| **Laziness** | The property that transformations build a plan and compute nothing until an action (Spark 4.2.0 docs, RDD Programming Guide, retrieved 2026-10-19). |
| **Namespace** | The single mapping from names to objects in a kernel. Shared by every cell and every attached frontend. |
| **nbformat** | The notebook file format. v4.5 carries an optional per-cell `execution` object with four timestamps (retrieved 2026-10-19). |
| **Physical plan** | The plan Spark will execute, after analysis and optimisation. Inspectable via `explain()`; may be re-planned at runtime. |
| **Prefix vs sample** | A prefix is the first n rows a plan produced, with no known distribution. A sample has a stated selection mechanism. `show()` gives the former; the guide never treats it as the latter. |
| **REPL** | Read-eval-print loop; the execution context a language runs in. State is per-REPL, not per-notebook (Databricks docs, last updated 2026-10-13). |
| **Run record** | The guide's name for the minimum defensible artefact: cell sequence, engine version and effective configuration, data snapshot, and plan where a performance claim is made (§7.4). |
| **Session** | A live attachment between a frontend and a kernel. Ends when the kernel ends; not shared between kernels. |
| **`SparkSession`** | The driver-side entry point owning the `SparkContext`, catalog and configuration. |
| **Stage** | "Each job gets divided into smaller sets of tasks called stages that depend on each other" (Spark 4.2.0 docs, retrieved 2026-10-19). |
| **Task** | "A unit of work that will be sent to one executor" (Spark 4.2.0 docs, retrieved 2026-10-19). |
| **Temp view** | A session-scoped named query over a DataFrame. Destroyed by restart; not visible to a parallel session (Databricks docs, last updated 2026-10-13). |
| **`truncate`** | `show()`'s string-elision flag, default `True`, truncating at 20 characters (PySpark 4.2.0 API docs, retrieved 2026-10-19). |
| **`_sqldf`** | An implicit DataFrame bound by running a SQL cell in a Databricks notebook, reassigned on every SQL cell run (Databricks docs, last updated 2026-10-13). Used here as the concrete case of an implicit rebind. |
| **Web UI** | The Spark application UI at port 4040, tied to the application's lifetime. Where stages, spill, skew and the SQL plan are visible while the application lives. |

### 16.3 Cross-references

Within this repository:

- [`technology/spark_tuning_guide.md`](../spark_tuning_guide.md) — **owns Spark internals and tuning.** §2 for driver/executor anatomy, §4 for the memory model, §5 for AQE mechanics, §6–§7 for concurrency, partitions and shuffle, §10–§11 for joins and partitioning, §12 for the UI as a tuning instrument. This guide cites AQE only as a reason the plan may not be the plan that ran, and cites the UI only as an evidential surface. If a question is "how do I make this faster", it belongs there.
- [`technology/ai_llm/databricks_agents_guide.md`](databricks_agents_guide.md) — **owns the platform's agent surface.** Agent products, tools as governed catalog objects, evaluation and tracing, deployment. This guide quotes its notebook page only for the state facts (per-REPL isolation, `_sqldf` reassignment, parallel-session limits) that are evidence about *session semantics*.
- [`technology/data/on_prem_lakehouse_guide.md`](../data/on_prem_lakehouse_guide.md) and [`technology/cloud_object_storage_lakehouse_guide.md`](../cloud_object_storage_lakehouse_guide.md) — **own storage, table formats and snapshot semantics.** §7.2c requires a data snapshot; these explain what a snapshot is and how a table version is held.
- [`technology/ai_llm/agent_sandboxing_strategies_guide.md`](agent_sandboxing_strategies_guide.md) — **owns isolation.** The spectrum from in-process to microVM, and what each boundary can contain (§9.2 assumes the agent's execution is already isolated).
- [`technology/ai_llm/agent_runtime_cache_design_guide.md`](agent_runtime_cache_design_guide.md) — **owns runtime caching.** Relevant to §8: what a session-scoped cache is, and why a second session does not inherit it.
- [`technology/ai_llm/agents_work_fall_apart_guide.md`](agents_work_fall_apart_guide.md) and [`technology/ai_llm/llm_agents_failures_production_guide.md`](llm_agents_failures_production_guide.md) — the general failure literature. This guide's contribution is the Spark-and-notebook-specific mechanism, not a general taxonomy of agent failure.

### 16.4 Closing summary

The arrangement this guide describes is not exotic. An agent writes a cell, a kernel runs it, the agent reads the output, and the cycle repeats — and it is already happening in notebooks wherever a coding agent has been pointed at a data platform. What this guide adds is a precise account of what the agent can and cannot know while doing it.

Four mechanisms carry the account. The **kernel** is one namespace and one ordered queue, with no dependency graph and an execution counter that resets on restart (§3). **Laziness** means a transformation is a plan and a success reply is a statement about Python, not about data (§4.2). **Observation** is a twenty-row prefix with elided strings, which supports no inference about the properties that change a number (§4.3). And the **cluster** — partitions, skew, spill, executor memory, the re-planned physical plan — lives outside the text interface, in a UI that dies with the application (§4.4). Compose them and the result is a session that produces confident, well-formatted, unreconstructible numbers.

The verdict is unsoftened because the evidence supports no softer version. A notebook is an exploration artefact, not a production record, and an agent-driven notebook is *less* reproducible than a human-driven one — the agent re-runs more, rebinds more, restarts more, and its ordering decisions have no witness, while the file it leaves records at most four optional timestamps per cell and nothing about the engine, the configuration, the data version or the plan (§7). What a defensible result needs is four captures and one artefact: the executed cell sequence, the engine and effective configuration, the data snapshot, the plan where performance is claimed — written into a run record stored beside the notebook, not inferred from it.

The concessions matter as much as the verdict. `explain()` is cheap and honest about the plan as planned. The event log and History Server hold per-stage truth after the application is gone. A pinned table version turns a moving table into a nameable object. A recorded session recovers the order the file loses. Each closes one axis; none closes all four, and none of them is read by an agent by default (§5.3).

The rulings in §6 are the operative part: bind at point of use, action every claim, never overwrite a name you did not bind, never drop or restart without a human, never publish a figure without its snapshot — and resolve accountability, per figure, to a person (§6.2). None of that is a product. All of it is available today, which is the point: the gap is not waiting on a vendor.

**An agent that writes Spark is easy; an agent that knows what its Spark did is the hard part.**
