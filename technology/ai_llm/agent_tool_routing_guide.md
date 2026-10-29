# Agent Tool Routing - the model selects a name

> **A 2026 deep-dive on the step between "the tool exists" and "the tool ran".** A tool set is not a menu of capabilities; it is a list of labels and descriptions, and the model emits one of those labels. Everything that makes a label ambiguous, a description untrue, or the choice premature is the routing problem. This guide covers the selection decision, the argument the model builds for it, and the failure handling around both. It does not cover where tools come from, how they are loaded, how they are exposed, or how drift is measured - four sibling guides own those.

> **Series context and boundary (read this first).** This guide develops a layer the repository has named but never developed. The harness taxonomy names a Tools layer and describes the tension between capability coverage and decision quality; it leaves routing as a scope note. This is that scope note, expanded. Four neighbours own the adjacent ground, and none of them is repeated here:
>
> - **mcp_progressive_disclosure_guide.md** (900 lines) - owns *how much of the tool surface enters the context window*: deferred loading, tool search, code-mode surfaces, instruction-level disclosure, gateway aggregation.
> - **mcp_discovery_guide.md** (640 lines) - owns *where tools come from*: the discovery problem, the official registry, directories and marketplaces, registry mechanics, remote and dynamic discovery, enterprise discovery.
> - **mcp_framework_tools_guide.md** (697 lines) - owns *how tools are exposed*: MCP concepts, architecture patterns, the tools ecosystem, server implementation, protocol-level security.
> - **ai_agent_drift_guide.md** (1,624 lines) - owns *when selection has changed*: tool selection drift as a named drift type, with detection, measurement and mitigation methodology.
>
> The mechanical test used to police this boundary, applied to every section below: **if the sentence is still true when the tool set contains exactly one tool, it is probably not about routing.** A single-tool agent cannot choose wrongly between names. It can still choose *whether* to call, and it can still build a wrong argument - those two are in scope. Everything else that survives the one-tool test belongs to a neighbour.

> **Verification policy.** Facts marked ✅ were checked against a primary source - a provider's own documentation or engineering post, a standard, or a paper's abstract - during writing (October 2026 baseline). ⚠ marks a vendor claim published about the vendor's own product, a figure without a published method, or a claim that could not be traced in this pass. ❌ marks a claim considered and rejected. **No leaderboard position, score, percentage, ranking or token-count delta is printed anywhere in this guide.** Benchmarks appear only as evidence that a failure mode exists and is measured by someone with a name and a date. Where a number would have been the finding, the mechanism is written instead.

> Byline: Jack Liu Shurui, Solution Architect
>
> Repo: personal research library - Series: LLM/AI Engineering & Operations - Domain: AI Engineering - Agent Platform
>
> Date: October 2026

**Contents**

1. Overview and the Boundary Declared
2. What Routing Actually Comprises
3. Why the Model Picks Wrong
4. The Description Is the Interface
5. Routing Versus Retrieval
6. The Disambiguation Patterns
7. Failure Handling
8. The Harness Responsibilities
9. The Evidence
10. The Banking and Regulated Angle
11. The Failure Modes
12. The Cymbal Bank Worked Example
13. The Anti-Patterns
14. The State of Practice
15. The Claims Audit and What Could Not Be Verified
16. Glossary, Cross-References and Closing Summary

---

## 1. Overview and the Boundary Declared

### 1.1 The claim in one paragraph

The model does not select a tool. It selects a **name** - one label out of the labels it was offered - and it does so on the evidence of a short natural-language description attached to that label. The function behind the name is never inspected. Its implementation, its authorization model, its side effects and its preconditions are invisible at selection time except to the extent that the description mentions them. This is not a limitation to be engineered away; it is the interface. **Everything that makes the name ambiguous is the routing problem**, and the routing problem is therefore a writing problem, a tool-set-shape problem and a control problem - not a search problem.

Two corollaries follow, and both are tested below rather than assumed:

- **(a) The model routes on the description, not the capability.** If two tools differ in behaviour but not in description, the model cannot reliably prefer the right one. If a description omits a precondition, the model will call the tool without meeting it.
- **(b) Routing is a decision problem, not a retrieval problem.** Retrieval ranks candidates by resemblance to the request. Routing has to answer a different question - *is any offered tool the right action here, under the policy in force, with these preconditions satisfied* - and resemblance is only one input to that answer.

### 1.2 Where the honest complications are

A guide that overstated the thesis would be easier to write and less useful. Three complications are stated up front and carried through:

- **The choice is constrained.** With schema-constrained or grammar-constrained decoding, and with a `tool_choice` setting of `required` or a named function, the emitted name must be one of the offered names. So "selection" is a **choice among labels**, not free generation. That strengthens the thesis rather than weakening it: if the output space is a closed set of labels, then label design *is* the whole interface, and there is nowhere else for the decision to come from.
- **Argument construction is separately constrained and semantically free.** The input schema constrains shape - types, enums, required fields - but not meaning. A schema-valid `amount: 100` can be the wrong amount. Schema validity is not correctness, and the harness that treats a schema pass as a semantic pass will ship wrong actions.
- **Retrieval-style pre-filtering does help.** When the catalogue is large and descriptions are discriminative, embedding or keyword shortlisting genuinely reduces prompt cost and can improve the choice. The argument in section 5 is about **what retrieval cannot do**, not that it never helps.

### 1.3 The decoder

The words below are used loosely in the field, often for the same thing. They are not the same thing.

| Term | What it means here | Who owns it | The question it answers |
|---|---|---|---|
| **Routing** | The whole act of getting from "a request arrived" to "this tool ran with these arguments", including the decision not to call anything | Harness + model, jointly | "What should happen next, and through which name?" |
| **Selection** | The narrow step of emitting one tool name from the offered set | Model, constrained by the offered set | "Which of these labels?" |
| **Argument construction** | Filling the input schema with values drawn from the request, the context and prior results | Model, constrained by schema | "With what inputs?" |
| **Dispatch** | The harness side: validate, authorize, execute, capture, return | Harness | "Is this call permitted and what came back?" |
| **Tool schema** | The machine-readable input contract: names, types, enums, required fields, sometimes output shape | Tool author | "What shape of arguments is legal?" |
| **Tool description** | The prose the model reads: what the tool does, when to use it, when not to, what it returns | Tool author | "Why would I pick this one?" |
| **Result** | What the tool returns, and how it is rendered back into the conversation | Tool author + harness | "What did we learn, and what does it license next?" |

The distinction that matters most for practice: **schema is for the machine, description is for the model, and only one of them is read at selection time.** A team can spend a quarter on schema rigour and still ship a tool set the model cannot route, because schema rigour does not disambiguate.

### 1.4 The four neighbours, and what each owns

- **mcp_progressive_disclosure_guide.md** - owns the inference-time surface: which definitions are in the prompt this turn, deferral, tool search, code mode. Its subject is *token placement*, not choice quality.
- **mcp_discovery_guide.md** - owns finding and adopting servers: registries, directories, marketplace mechanics, enterprise approval. Its subject is *does this exist and may I attach it*.
- **mcp_framework_tools_guide.md** - owns the protocol and the server: primitives, transports, implementation, connect-time security. Its subject is *how the capability is exposed*.
- **ai_agent_drift_guide.md** - owns change over time: that selection degrades, how to detect it, how to measure it, how to mitigate it. Its subject is *the delta*, not the mechanism.

The crisp contrast with the first neighbour, since it is the closest: **disclosure decides what the model can see; routing decides what the model does with what it sees.** A team can defer perfectly and still route badly. A team can offer three tools and route badly. The failure survives every loading optimisation, which is why it needs its own treatment.

### 1.5 How to read this guide

Sections 2 to 4 are the mechanism: what the decision actually consists of, why it fails, and why the description carries the weight. Sections 5 and 6 are the two design arguments that follow - retrieval is not routing, and disambiguation is a writing discipline. Sections 7 and 8 are the harness contract: what to do when it goes wrong, and what belongs in code rather than in a prompt. Section 9 is the evidence, with no scores. Sections 10 to 12 take it into a regulated institution and then into a worked example. Sections 13 to 16 are the residue: anti-patterns, state of practice, claims audit, glossary.

If you read one section, read 4. If you build one thing, build the audit trail in 8.

---

## 2. What Routing Actually Comprises

### 2.1 Four steps that get called one word

Practitioners say "the agent picked the wrong tool" about at least four different failures. They have different causes, different observability and different fixes. Conflating them is why the fix often misses.

**Step 1 - the act decision.** Should any tool be called at all? Some requests are answerable from context, some need a clarifying question, some need a tool, and some must be refused. The failure here is binary and expensive in both directions: calling when nothing should be called (a write tool fired on an ambiguous request), or answering from memory when a tool was required (a stale balance quoted as current).

**Step 2 - the name decision.** Given that a tool should be called, which label? This is the step the thesis is about. It is a classification over a closed set, performed on descriptions.

**Step 3 - the argument decision.** Given a name, what values? The schema constrains shape; the request and prior results supply content; the model supplies interpretation. Most production incidents that get filed as "wrong tool" are actually this step - the right name with a plausible but wrong identifier.

**Step 4 - the result decision.** Given a result, what now? Is the result sufficient, is it an error to be retried, is it a partial answer to be completed with a second call, or is it evidence that the first choice was wrong? This is where silent fallbacks live.

### 2.2 The four steps, their failure modes and their owners

| Step | Question | Typical failure | What observes it | Who fixes it |
|---|---|---|---|---|
| **1. Act** | Call anything? | Tool fired on a request that needed clarification; or no call when data was stale | Absence or presence of a call against request class | Prompt policy + harness precondition |
| **2. Name** | Which label? | Adjacent tool chosen; correct tool never chosen | Emitted name against expected name, per request class | Tool-set shape + descriptions |
| **3. Arguments** | Which values? | Schema-valid, semantically wrong identifier, amount, date or scope | Post-condition check, not schema validation | Schema tightening + examples + confirmation |
| **4. Result** | What now? | Error swallowed; partial result presented as complete; retry of a write | Result-to-answer diff; retry counts | Harness error contract |

### 2.3 Why the distinction pays for itself

Because each step has a different repair. If step 2 is failing, rewriting the schema will not help and adding examples to the prompt may actively hurt - it lengthens the prompt without separating the labels. If step 3 is failing, no amount of description work will help, because the model already chose correctly; the fix is a stricter type, an enum, a resolved identifier instead of a free string, or a confirmation step. If step 1 is failing, the fix is a precondition in code - a check that the request contains the fields the tool needs before the tool is offered at all.

The practical diagnostic question, asked of any routing incident: **was the name wrong, or was the name right and everything after it wrong?** Teams that cannot answer this from their logs are not measuring routing; they are measuring task success and guessing at the cause.

### 2.4 The act decision deserves its own owner

Most tool-set design assumes the model will call a tool when one is needed. That assumption fails in a specific and under-instrumented way: models **under-call**. A tool that is correct, present and well described can simply not be used, because the model judged the request answerable from what it already had. This is the failure that no wrong-name metric catches, since no name was emitted.

The harness lever is not a stronger instruction ("you must use tools"). It is making the *absence* of a call observable: log the request class, log that no tool was called, and compare against the classes where a tool was required. Section 8 treats this as an audit obligation rather than a prompt problem, because it is one.

### 2.5 The one-tool test applied

Under the one-tool test, step 2 disappears and steps 1, 3 and 4 remain. That is the correct division of labour for this guide: it owns the act decision, the name decision, the argument decision and the result decision, and it owns none of them for a single-tool agent except the name one, which is vacuous there. The name decision is the reason the guide exists; the other three are in scope because they are routinely misattributed to it.

---

## 3. Why the Model Picks Wrong

### 3.1 Five causes, one mechanism

Every routing error reduces to the same mechanism: **the model chose the label whose description best matched its reading of the need, and that label was not the one whose behaviour satisfied the need.** The five causes below are the ways that gap opens.

### 3.2 Name ambiguity

Two labels that a human would distinguish from context are, to a model, two strings with high mutual similarity. `get_account` and `get_account_details`. `search_records` and `find_records`. `update_status` and `set_status`.

The model has no access to the implementation, no access to the OpenAPI operation behind the tool, and often no access to the team that named them. It has the name and the description. If the name carries no discriminating information, the decision falls through to the description; if the description also carries none, the decision falls to position, phrasing or chance - and BiasBusters documents exactly that: models disproportionately favour tools that appear earlier in context, and small perturbations to descriptions shift choices (Blankenstein et al., arXiv 2510.00307, ICLR 2026 camera-ready).

Name ambiguity is cheap to fix and rarely fixed, because names are usually generated from the underlying function or endpoint name and nobody owns the rename.

### 3.3 Description overlap

Overlap is subtler than ambiguity. Two descriptions can each be clear, accurate and well written, and still fail to separate their scopes:

- "Retrieves customer information for a given customer." (`get_customer`)
- "Fetches details about a customer record." (`lookup_customer`)

Each is fine alone. Together they are the same sentence twice. The model now chooses on token-level texture. Whoever wrote the second one probably meant "by fuzzy match on name, for when you do not have an ID" - which is the discriminating fact, and it is not in the text.

The general shape: **overlap happens when the discriminator is a precondition (do I have an ID?), a scope (one record or many?), or a side effect (read or write?), and none of those three is written down.**

### 3.4 Description drift from behaviour

Descriptions are written once and behaviour changes. The tool that "returns the settled balance" now returns the posted balance plus pending items after a ledger migration. The description still says settled. The model routes correctly to a tool that no longer does what its label says, and the failure surfaces downstream as a wrong number rather than a wrong call.

This is not tool-selection drift in the drift guide's sense - the model's behaviour has not changed, the tool's meaning has. It is the routing-specific version of configuration drift, and it is invisible to any evaluation that only checks whether the right name was called. Section 8 puts description-to-behaviour verification in the harness for this reason.

### 3.5 The unstated precondition

A precondition the description does not mention is a precondition the model will violate. Common forms:

- **Identity**: the tool needs a resolved customer ID, and the request contains a name.
- **State**: the tool only works on accounts in a given status, and the request concerns one that is not.
- **Sequence**: the tool needs a prior lookup to have populated a session, and the model calls it cold.
- **Authority**: the tool performs an action above the caller's approval limit, and nothing in the text says so.

The last one is where routing becomes a control problem rather than a quality problem (section 10). A model cannot respect a limit it was never told about, and telling it in the description is weaker than enforcing it in dispatch.

### 3.6 The correct tool that is never chosen

The inverse failure, and the one that evaluation suites systematically miss: the tool exists, is described, is available, and is simply not used. The agent answers from parametric knowledge instead, or takes a longer path through three other tools that happens to arrive at a similar place.

This shows up in metrics as "acceptable answer, higher cost, longer latency" and is therefore invisible to answer-level grading. It is a routing failure with no wrong name to point at - a step-1 failure in the taxonomy of section 2.

Anthropic's engineering guidance names the four failure classes together - agents "might call the wrong tools, call the right tools with the wrong parameters, call too few tools, or process tool responses incorrectly" (11 September 2025, vendor engineering post, ✅ as published). "Call too few tools" is this cause, and it is the one most often left out of routing work.

### 3.7 What the five causes have in common

None of them is fixed by a better model, a longer prompt, or a bigger tool budget. All five are fixed by making the offered set **smaller and more honestly described**, or by moving the check out of the model and into the harness. That is the thesis restated as an engineering agenda.

---

## 4. The Description Is the Interface

### 4.1 The argument

If the model selects on description, then the description is not documentation of the interface - **it is the interface**. The two common counter-arguments do not survive contact with the mechanism:

- *"The model also sees the schema."* It sees parameter names and types, which help with step 3 and almost nothing with step 2. A schema that distinguishes two tools by an optional parameter nobody reads at selection time does not disambiguate them.
- *"The model knows what the tool does from its name."* Only if the name encodes the distinction, which is the naming discipline in section 6, and which most tool sets do not practise.

Provider documentation converges on the same point from three directions. Anthropic's tool-definition guidance calls detailed descriptions "by far the most important factor in tool performance", asks for what the tool does, when it should and should not be used, and what each parameter means, and recommends at least three to four sentences per description (Claude Platform docs, read October 2026, ✅). Its engineering post on writing tools for agents says the same in the register of practice: prompt-engineering tool descriptions and specs is "one of the most effective methods for improving tools", and small refinements can produce large behavioural changes (11 September 2025, ⚠ vendor claim with a worked anecdote, no published method).

None of this is measured independently. That is the honest status: **the description-first position is well supported by mechanism and by practitioner report, and under-supported by controlled study.** Section 15 records that gap rather than hiding it.

### 4.2 Three worked pairs

The pairs below are invented and deliberately small. Each is a real shape of failure, not an MCP-specific one.

**Pair 1 - the generic verb.**

```
BAD
  name: get_data
  description: Gets data for the request.

GOOD
  name: get_invoice_status
  description: Returns the payment status of one invoice, identified by
    invoice_id. Use when the caller already has an invoice identifier and
    needs to know whether it is paid, part-paid or overdue. Does not return
    invoice line items, and does not accept a customer name or account
    number. For a customer's full invoice list, use list_invoices.
```

The bad version is not wrong; it is empty. `get_data` selects on nothing, so it wins whenever nothing else matches - which is to say, unpredictably. The good version names the object, the identifier it requires, the fact it returns, and the two adjacent tools it is not.

**Pair 2 - overlapping scope.**

```
BAD (a)
  name: search_customers
  description: Searches for customers.

BAD (b)
  name: find_customer
  description: Finds a customer.

GOOD (a)
  name: search_customers
  description: Returns up to 20 customer records matching a partial name,
    trading name or registration number. Use when the caller does not have
    a customer_id and needs to discover one. Ambiguous matches are returned
    as a list for the caller to choose from; never guess between them.

GOOD (b)
  name: get_customer
  description: Returns the single customer record for a known customer_id.
    Use only when the identifier is already known and unambiguous. Does not
    search by name; if the caller has a name and no identifier, use
    search_customers first.
```

The discriminator here is a precondition - *do I have an identifier* - plus a cardinality difference (list versus single). Both are stated, and each description points at the other. That cross-reference is the cheapest disambiguation technique available and the least used.

**Pair 3 - the missing negative constraint.**

```
BAD
  name: issue_refund
  description: Issues a refund to a customer.

GOOD
  name: issue_refund
  description: Issues a card refund against a settled transaction, up to the
    original transaction amount, for transactions settled in the last 120
    days. Requires transaction_id and a reason code. Use only after the
    customer has confirmed the amount and the destination card.
    Do NOT use for: chargebacks (use open_dispute), fee reversals (use
    reverse_fee), or transactions older than 120 days (escalate to a manual
    adjustment). Does not move money immediately; settlement follows the
    card scheme cycle.
```

The bad description is true and useless. The good one carries the amount ceiling, the age limit, the required identifier, the confirmation precondition, the three adjacent cases with their correct tools, and the fact that the effect is not immediate. Every one of those clauses exists because a model would otherwise assume the opposite.

### 4.3 What a routing-grade description contains

Working from the pairs, the clauses that earn their tokens:

| Clause | Why it exists | Failure if absent |
|---|---|---|
| **Verb + object** in the name | The name is read before the description | Adjacent names collapse |
| **One sentence of scope** | Establishes the boundary | Model infers a broader scope |
| **Required identifiers** | Step 3 depends on it | Plausible-but-wrong arguments |
| **"Use when"** | Positive trigger | Undertriggering |
| **"Do not use when"** | Negative constraint | Confident misuse of the near neighbour |
| **Adjacent tool by name** | Turns a dead end into a redirect | Model retries the same wrong tool |
| **Preconditions** (state, authority, sequence) | Precondition violations are silent | Action taken that should have been refused |
| **Return shape and latency** | Sets the model's expectation of what comes next | Partial result treated as complete |
| **Side-effect statement** | Read versus write is the highest-stakes distinction | Write tool called to answer a read question |

### 4.4 What does not belong in a description

- **Marketing language.** "Powerful", "comprehensive", "seamless" carry no discriminative information and consume the attention that the boundary sentence needed.
- **Implementation detail.** Retry counts, internal service names, database table names. The model cannot act on them and they crowd the clauses it can act on.
- **Instructions to the model** ("always call this first"). The description is a property of the tool, not of the agent; the same tool is offered to agents with different policies. Put the policy in the harness.
- **Ambiguous pronouns.** "It", "this", "the record". Descriptions are read out of context, in a list, by something with no memory of the paragraph that introduced the noun.

### 4.5 The description is code, and should be reviewed like code

If the description is the interface, then a description change is an interface change. It should be reviewed by someone who knows the tool's behaviour, versioned with the tool, and tested by a routing evaluation that includes the near neighbours. The practical failure is that descriptions live in a schema file owned by whoever wrote the wrapper, are edited during incidents, and never appear in a change record. Section 8 makes this an audit item.

---

## 5. Routing Versus Retrieval

### 5.1 The distinction, sharply

Retrieval answers: *which of these documents resembles this query?* Its objective is ranking quality against a query, and its success metric is recall of the relevant item.

Routing answers: *what should be done about this request, and through which name, under the policy in force?* Its objective is the correctness of an action, and its success metric is whether the resulting action satisfied the need without violating a constraint.

The two are related the way "find the relevant paragraph" is related to "decide what to do". A shortlist of plausible tools is an input to a routing decision. It is not the decision. The decision additionally requires: whether to act at all, whether preconditions hold, whether the caller is authorized, whether several tools must be used in sequence, and whether the nearest-matching description is describing the right thing.

### 5.2 What retrieval genuinely solves

Stated plainly, because the argument is weaker without it:

- **Catalogue size.** A catalogue of hundreds or thousands of tools cannot be offered whole. Retrieval makes the problem tractable at all. ToolLLM built exactly this - a retriever over a very large API corpus, recommending candidate APIs per instruction (Qin et al., arXiv 2307.16789, 2023, ✅ as published).
- **Prompt cost.** Shortlisting descriptions instead of offering all of them reduces prompt tokens, which is the mechanism RAG-MCP reports: semantic retrieval before the model sees anything, then only the selected descriptions are passed (Gan and Sun, arXiv 2505.03275, 6 May 2025, ✅ as published).
- **Noise reduction** when descriptions are discriminative and the request vocabulary matches the tool vocabulary. In that regime, a shortlist is a real improvement in the choice, not merely in the cost.

The limits of that evidence matter: RAG-MCP is measured on selection alone, against a deliberately bloated baseline, in a simulated catalogue. It shows that retrieval beats *offering everything*. It does not show that retrieval beats *a well-shaped small tool set*, and it does not measure task success.

### 5.3 Where retrieval fails as a routing substitute

**Complementary tools that must be used together.** A task may require `search_customers` and then `get_customer`, or `list_transactions` and then `issue_refund`. Similarity to the request ranks these against each other, because both resemble the request. The decision is not "which one" but "both, in this order". A ranker has no representation for that.

**Sequencing.** The second call's arguments come from the first call's result, not from the query. Retrieval is stateless with respect to the trajectory: it sees the original request, not the intermediate state. An agent that retrieves per turn re-ranks on a query that no longer describes the need.

**Negative constraints.** A "do not use this for chargebacks" clause is not a similarity signal. An embedding of the description may even place the forbidden tool *closer* to a chargeback request, because the description mentions chargebacks. Negative constraints have to be read as constraints, which means the decision has to be made by something that reads clauses rather than distances.

**Tools whose name does not resemble the need.** The user says "reconcile the ledger". The tool is `get_postings`. Thematically adjacent, lexically distant. Retrieval misses it; a description written with the user's vocabulary ("use when reconciling the ledger") catches it. This is the strongest argument for treating description writing as the primary lever: it is the only mechanism that can bridge the vocabulary gap deliberately rather than statistically.

**The zero-candidate case.** Sometimes no tool is right. Retrieval always returns something, because ranking a list returns a first element. Routing has to be able to return "nothing" - BFCL builds a category for precisely this, where none of the provided functions are relevant and the expected output is no call at all (Patil et al., UC Berkeley, ✅ as published in the benchmark's own description).

### 5.4 The correct composition

Retrieval in front of routing, not instead of it:

1. **Retrieve** a shortlist by resemblance, sized generously - recall matters more than precision at this stage.
2. **Route** over the shortlist with full descriptions, negative constraints and preconditions intact.
3. **Enforce** in dispatch: authorization, precondition checks, post-condition verification.

The failure pattern is collapsing steps 1 and 2 - taking the top match as the decision. That configuration has no representation for "none of these", no representation for "both of these", and no way to weigh a constraint against a resemblance score.

---

## 6. The Disambiguation Patterns

### 6.1 Boundary statements

A boundary statement is one sentence in each of two adjacent descriptions that states the difference in terms the model can act on. It is not documentation; it is a discriminator.

```
get_account_balance
  Returns the posted ledger balance. Excludes pending card authorizations.
  For the balance a customer can spend right now, use get_available_balance.

get_available_balance
  Returns the spendable balance: posted balance less holds and pending
  authorizations. Use when the customer asks what they can spend. For
  reconciliation or statement figures, use get_account_balance.
```

Two descriptions, one distinction, stated twice from both sides. The duplication is deliberate: only one of the two will be read closely, and either reading should be sufficient.

### 6.2 Negative examples

A negative example is a short clause naming a case the tool is not for, ideally with the correct alternative. "Do not use for X; use Y." This is the highest-yield clause in the entire description vocabulary, because the model's default error is over-application of a plausible tool, and over-application is exactly what a negative clause blocks.

Three rules for writing them:

- **Name the alternative.** "Do not use for chargebacks" leaves the model to guess; "use open_dispute" ends the search.
- **Use the user's vocabulary, not the system's.** The clause triggers on the words the request will contain.
- **Keep them to the two or three nearest neighbours.** A description that excludes everything becomes unreadable and the exclusion that mattered gets lost.

### 6.3 The "do not use this when" clause and its cost

The clause is not free. It lengthens every description, and length is paid on every turn the tool is offered. It also creates a maintenance obligation: a negative clause that names a tool which has since been renamed is worse than no clause, because it redirects to a label that no longer exists.

The discipline that keeps it affordable: negative clauses only between tools that are genuinely confusable, and each clause reviewed when either tool changes. On a well-shaped tool set of a dozen tools, this is two or three clauses. On a set of two hundred, it is unmanageable - which is the argument for shaping the set (section 8) rather than writing around it.

### 6.4 Naming discipline

Names are read before descriptions and are cheaper to compare. Four rules that survive contact with real tool sets:

- **Verb plus object, always.** `issue_refund`, not `refund`; `get_customer`, not `customer`.
- **Namespace by the resource or service, consistently.** Anthropic's guidance recommends prefixing by service (`github_list_prs`, `slack_send_message`) and reports that choosing between prefix and suffix naming had non-trivial effects on its own tool-use evaluations, varying by model (Claude Platform docs, ✅; engineering post, 11 September 2025, ⚠ vendor claim as to effect size).
- **One concept, one word, across the whole set.** If reads are `get` in one family and `fetch` in another, the model learns two vocabularies for one operation and the discriminator is lost.
- **Do not encode version or environment in the name** (`get_balance_v2`, `get_balance_prod`). It creates pairs that are near-identical by construction and invites the model to choose on the suffix.

### 6.5 The deprecated-tool problem

Retiring a tool is a routing change, and the three available options trade off differently:

| Option | Effect on routing | Cost |
|---|---|---|
| **Remove it outright** | Cleanest decision surface; the wrong choice becomes impossible | Callers that still reference it fail; removing tools mid-conversation also invalidates prompt caching (documented by Anthropic, ✅) |
| **Keep it, mark it deprecated in the description** | Preserves compatibility; leaves a plausible label in the choice set | The model may still choose it; the description now has to argue against itself |
| **Keep it as an alias that redirects** | Preserves compatibility, no wrong behaviour | The choice set stays ambiguous; the redirect hides which name was chosen, which is an audit problem (section 10) |

The rule that follows: **deprecation is a routing decision and belongs in the change process, not in a comment.** The worst configuration is a deprecated tool whose description says "deprecated, use X" without saying what happens if it is called - the model reads it as available, and the harness, having kept it, honours it.

### 6.6 Disambiguation patterns at a glance

| Pattern | Instrument | Fails when |
|---|---|---|
| Boundary statement | One discriminator sentence in each of two descriptions | Only one side is updated |
| Negative example | "Do not use for X; use Y" | The named alternative is stale |
| Precondition clause | "Requires customer_id"; "only for status ACTIVE" | The precondition is not enforced in dispatch |
| Cardinality clause | "Returns one record" versus "returns up to 20" | Descriptions drift from the implementation's actual limit |
| Namespacing | Consistent prefix or suffix by resource | Inconsistent between families |
| Consolidation | One tool with a mode parameter instead of three tools | The mode enum becomes its own routing problem |
| Retraction | Removal or alias | Applied silently, with no eval or audit trace |

Consolidation deserves the warning in the last row. Merging three tools into one with an `action` parameter reduces name ambiguity and is recommended by provider guidance for exactly that reason (Claude Platform docs, ✅). It converts a name choice into an argument choice - which means the routing error moves from step 2 to step 3, where it is harder to observe and where the schema has to carry the discrimination instead. Fewer tools is not automatically better; fewer *ambiguous* tools is.

---

## 7. Failure Handling

### 7.1 Four failures, four different responses

A harness that responds to every routing problem with "retry the turn" converts one wrong action into two. The four cases below need distinct responses because they are observable in different places and carry different risk.

| Case | What the harness sees | Risk | Response |
|---|---|---|---|
| **Wrong tool, valid call** | A name was emitted; the result does not satisfy the post-condition | Medium - depends on whether the tool was a read or a write | Re-prompt with the discriminating clause and the shortlist; never blindly retry |
| **Right tool, wrong arguments** | The call passed schema validation; the outcome is wrong or the tool returned a not-found or mismatch error | High if the tool writes | Stop, do not auto-correct identifiers, ask for confirmation or re-resolve the identifier |
| **No tool when one was needed** | No call emitted on a request class where a call was required | High - a wrong answer delivered confidently | Detect by request class, then re-prompt with an explicit requirement or fail closed |
| **A tool that must not be called** | A name was emitted that the policy forbids for this caller, this state or this amount | Highest | Refuse in dispatch, before execution; log as a control event, not a quality event |

### 7.2 Wrong tool: re-prompt, do not retry

The critical distinction: a retry repeats the same decision with the same evidence and gets the same answer, possibly with more confidence. A **re-prompt** changes the evidence. Practically it means one of three moves, in ascending cost:

1. **Add the discriminator to the conversation**: "The tool `get_account_balance` excludes pending authorizations; `get_available_balance` includes them net of holds. The customer asked what they can spend." This works because the model was missing exactly one clause.
2. **Narrow the offered set** for the retry: offer only the two plausible tools plus the one correct one. This is a harness-side change and is more reliable than a conversational hint, because it changes the choice space rather than the argument for a choice.
3. **Hand off** to a deterministic sub-flow or a human, if the second attempt also fails.

Bounded: at most one re-prompt per routing decision. An unbounded loop over a misdescribed tool set produces a long transcript, a high token bill, and the same wrong answer.

### 7.3 Right tool, wrong arguments: the schema-valid trap

`schema`-valid is not `correct`. The failure modes that pass validation:

- **Wrong identifier of the right type.** `customer_id` from the wrong customer, because the model picked the first match from an earlier search result.
- **Wrong amount**, off by a unit, a currency, or a sign.
- **Wrong date range**, where the model inferred "last month" instead of using the statement period in context.
- **Wrong scope**, where an `all: true` flag was set on a request that concerned one record.

The harness response is **verification, not retry**. Two mechanisms earn their cost: a post-condition check that is specific to the tool (did the amount match the transaction we were discussing; does the returned record belong to the customer in the session) and, for anything that writes, a human confirmation showing the resolved values rather than the model's summary of them.

The rule that prevents most of this class: **resolve identifiers in code before the model sees them**. If `customer_id` arrives from a prior tool result rather than being composed by the model, the class largely disappears.

### 7.4 No tool chosen when one was needed

This is the hardest to observe, because a plausible answer is emitted. Detection is a harness obligation:

- **Request classification.** Tag incoming requests by class (balance enquiry, dispute, payment, general) and record, per class, whether a tool call occurred.
- **Staleness assertion.** If the answer contains a figure that must come from a system of record, require that a tool call produced it. An answer with no provenance for a number is a routing failure whether or not the number happens to be right.
- **Fail closed on required classes.** For classes where an answer without a tool call is unacceptable, the harness withholds the answer path - it does not let the model answer from memory.

Note what this is not: it is not a stronger system-prompt instruction. Instructions raise the rate; they do not make the absence observable, and an unobservable failure cannot be audited.

### 7.5 A tool that must not be called

Here the harness is the only defence that matters. Three layers, in order:

- **Existence**: the tool is not offered to this caller at all. Nothing to select, nothing to refuse. This is the strongest control and the cheapest to reason about.
- **Authorization in dispatch**: the tool is offered, the call is intercepted, the policy check runs against the *resolved* arguments, and the call is refused with a typed error the model can read.
- **Confirmation**: for a narrow band of actions, a human approves the resolved arguments. Four-eyes, in the banking sense (section 10).

A description clause ("do not call this above the approval limit") is not a control. It is a hint that a model may honour and that an attacker, a prompt injection, or a confused context may override. Descriptions shape the decision; dispatch enforces it.

### 7.6 The error contract the model needs

A failure the model cannot interpret is a failure it will retry blindly. The result returned on refusal or error should carry: what went wrong in one sentence, whether retrying with different arguments could succeed, and what the correct next tool is if there is one. This is the result-side of routing and it is usually the least designed part of a tool set - an exception stringified into a tool result, which the model then reads as a transient failure and retries.

---

## 8. The Harness Responsibilities

### 8.1 What belongs in code rather than in the prompt

Four things, and all four are routinely put in the prompt instead, where they are advisory:

| Responsibility | In the harness | Why not the prompt |
|---|---|---|
| **Tool-set shape** | Which tools exist for this agent, this role, this step | Prompts cannot add or remove labels |
| **Existence guardrail** | The allowlist per role and per task stage | A model cannot be trusted to exclude a tool it can see |
| **Retry policy** | Bounded re-prompts, no blind retry of writes, escalation path | Policy in prose is a suggestion |
| **Audit** | The record of what was chosen, on what evidence, with what arguments | The model cannot attest to its own decision |

The harness guide's layer taxonomy puts this work in the Tools (T) layer with an observability obligation in O; the routing view adds that the T layer's shape is a *control* surface, not only a capability surface. See agent_harness_engineering_guide.md sections 3.4 and 3.5 for the taxonomy, and section 6 there for why observability has to be designed alongside the tool interface rather than after it.

### 8.2 Tool-set shape

Shape is the highest-leverage decision and the least discussed. Three constraints:

- **Count.** Google's Gemini function-calling guidance states it directly: keep the active tool set to 10-20 tools maximum (ai.google.dev, page footer dated 2026-09-23, ✅). This is prescriptive guidance, not a measured threshold - no published study establishes where the cliff is, and the number will move with model capability. It is still the only concrete count a major provider publishes, and it is a defensible default.
- **Coherence.** A set whose members are all in one domain can be large; a set spanning payments, HR and infrastructure cannot be small enough. Coherence is what makes the count affordable.
- **Stability.** The set offered on turn one should be the set offered on turn five unless the task genuinely changes stage. Churn in the offered set is a routing hazard: the model's earlier reasoning about which tools exist no longer holds.

### 8.3 The existence guardrail

Decide, in code, which tools a given agent may ever see. Three levels that compose:

- **Role**: a servicing agent does not see administration tools. Not because it would misuse them, but because their presence is one more label competing for attention.
- **Stage**: during discovery, only read tools are offered. Write tools appear after the target is resolved. This converts a sequencing requirement from a prompt instruction into a structural fact.
- **Authority**: amount and approval limits resolve to a different tool set, or to the same tool with dispatch-level enforcement.

The design principle: **the safest routing decision is the one that was never available to make.**

### 8.4 Retry policy

Bounded and typed:

- One re-prompt per decision, with new evidence (section 7.2).
- No automatic retry of a write, ever. A retry of a read is cheap; a retry of a transfer is a duplicate.
- Escalation after the second failure, to a deterministic flow or a human, not to a third attempt.
- Every retry logged with the reason, so that the routing failure rate is measurable separately from the tool error rate.

### 8.5 The audit of what was chosen

The record, per decision: the request class, the set of tools offered (with a hash of the descriptions, so a later review knows what the model actually read), the name emitted, the arguments, the post-condition result, whether a human confirmed, and the outcome. This is the minimum for answering "why did it do that" six weeks later, and it is the artefact a regulated institution will be asked for (section 10).

Two properties make the difference between an audit trail and a log: **the offered set must be recorded, not just the chosen name** (a wrong choice from a well-shaped set is a different finding from a wrong choice from a set that should never have contained the tool), and **description versions must be resolvable** (otherwise the decision cannot be reconstructed after a description edit).

### 8.6 What the harness should not do

It should not attempt to *decide* the routing in code and use the model as a formatter. That configuration - a classifier in the harness selecting the tool, the model only filling arguments - is legitimate and sometimes correct, but it is not agent routing; it is a workflow with a language model at the edge. It belongs in a different discussion, and it loses the property that makes agents useful: the ability to handle a request nobody anticipated.

The harness shapes the decision space and enforces the consequences. The decision stays with the model. That division is the whole design.

---

## 9. The Evidence

### 9.1 What the benchmarks establish

The claim "models route tools badly" is not an anecdote. It is measured, repeatedly, by named groups, with the measurement instrument published. What follows is mechanism and attribution only - no positions, no scores, no comparisons, by design.

| Instrument | Source, date | What it establishes | What it does not establish |
|---|---|---|---|
| **MetaTool** | Huang et al., arXiv 2310.03128, 4 Oct 2023, revision Dec 2024 | Tool-use awareness and tool selection are separable problems; four selection subtasks including selection among similar tools; most of the models tested struggled, and the authors recommend rewriting descriptions per downstream model | Nothing about multi-step task success; description rewriting is recommended, not proven to fix it |
| **BiasBusters** | Blankenstein et al., arXiv 2510.00307, ICLR 2026 camera-ready | Selection is sensitive to presentation: earlier tools are favoured, small description perturbations shift choices, and semantic alignment between query and metadata is the strongest driver | Does not separate "description is the interface" from "description is a correlate"; the benchmark uses functionally equivalent tools, so correctness is not the variable |
| **RAG-MCP** | Gan and Sun, arXiv 2505.03275, 6 May 2025 | Retrieval over descriptions before the model sees them reduces prompt cost and improves a selection-only metric against a bloated baseline | Selection-only metric, simulated catalogue, no task success, no argument correctness |
| **tau-bench** | Yao et al., June 2024 | Tool-agent-user interaction with a domain policy; reliability measured over repeated trials rather than a single pass; agents fail on rule-following as well as on tool choice | Domain APIs are curated and few; the failure taxonomy does not separate wrong name from wrong argument |
| **BFCL** | Patil et al., UC Berkeley, 2024 onward | Executable and AST-based checks on calls; a dedicated relevance-detection category where no offered function is correct and the expected output is no call | Single-turn emphasis in the earlier versions; tool sets are supplied by the benchmark, not grown by an institution |
| **ToolBench / ToolLLM** | Qin et al., arXiv 2307.16789, 2023 | Very large real-API corpora with single- and multi-tool instructions; a retriever over API descriptions as part of the pipeline | API descriptions are harvested, not written for routing; the retriever is part of the method, so retrieval and routing are not separated |

### 9.2 What this body of evidence is actually evidence of

Three findings survive the differences between instruments:

- **Selection is a distinct capability from execution.** A model can construct a valid call and choose the wrong function. MetaTool separates awareness from selection; BFCL separates relevance detection from call correctness.
- **Selection is unstable under presentation changes.** BiasBusters isolates metadata effects; MetaTool's recommendation to rewrite descriptions per model points the same way from a different direction.
- **Selection degrades as the offered set grows or becomes redundant.** This is the consistent practitioner claim across provider guidance, and the direction the retrieval work assumes.

### 9.3 What none of them measure

The gaps are specific enough to name:

- **The sequencing case.** No instrument in this list isolates "chose correctly but in the wrong order" from "chose wrongly". A multi-tool task that fails after a correct first choice is scored as a task failure, and the routing question - was the *second* choice wrong, or was the first choice's argument wrong - is not answered.
- **The real tool set.** Benchmark tool sets are curated, small, and written by the benchmark's authors. An institution's tool set is large, overlapping, historically accreted, and described by people with other jobs. Nothing here measures that.
- **The cost of a wrong call.** Every instrument scores correctness. None scores a wrong call that was schema-valid, authorized, and expensive - the duplicate payment, the wrong customer's data disclosed, the unnecessary hold. The economically important failures are the ones with a cost dimension, and the cost dimension is absent.
- **Argument semantics.** AST and execution checks verify that a call is well-formed and runs. They do not verify that the amount was the amount the customer meant.

### 9.4 How to use the evidence honestly

Cite it for the existence and shape of the problem. Do not cite it for a threshold ("switch to retrieval above N tools"), a technique ranking, or a claim about any particular model. The published work establishes that the failure is real and measurable; the design decisions in sections 4 to 8 are engineering judgement informed by mechanism, not by a measured comparison.

---

## 10. The Banking and Regulated Angle

### 10.1 Routing as a control point

In an unregulated setting a wrong tool is a quality defect. In a bank it is a control event, because the tool may have moved money, disclosed data, or created a record that will be relied upon. The question an auditor asks is not "was the model accurate" but **"which tool was chosen, by what, on whose authority, and what evidence exists that the choice was reviewed"**.

That question has four parts, and each maps to something this guide has already argued:

| Audit question | What must exist | Where it comes from |
|---|---|---|
| **Which tool was chosen** | The emitted name, recorded per decision | Section 8.5 |
| **By what** | The offered set and description versions at the time | Section 8.5 |
| **On whose authority** | The caller identity, the entitlement that permitted the tool, the approval if required | Section 8.3, dispatch-level authorization |
| **Reviewed by whom** | Four-eyes evidence for write actions above a threshold | Section 7.5 |

A system that records only the final answer cannot answer any of the four. This is the practical reason routing deserves a named owner in a regulated estate: not because the model is unreliable, but because the *decision* is the thing that has to be evidenced.

### 10.2 Least privilege, applied to labels

Least privilege is usually discussed as a permission model. Under a routing view it has a second meaning: **the tool set offered to an agent is itself an entitlement**. Offering a write tool to an agent that only needs to answer questions is a privilege grant, even if the model never calls it, because the offered set is what the model reasons over and what a prompt injection can reach for.

The control follows: derive the offered set from the role and the task stage, log it, and treat a change to it as a change to an entitlement - which means it goes through review, not through a configuration edit during an incident.

### 10.3 Four-eyes on write tools

The pattern, without citing any regime's clause numbers (none read for this guide):

- **Classify tools by effect**: read, reversible write, irreversible write. The classification lives with the tool definition and is not inferred by the model.
- **Two-key for irreversible writes**: the model proposes, a human approves the resolved arguments, dispatch executes. Approval shows the values, not the model's summary.
- **Maker-checker within the agent's own flow** for reversible writes above a threshold: a second, independent check - deterministic rule or a second model call with a different prompt - before dispatch.
- **No self-approval path**: the approval mechanism must not be a tool the same agent can call.

### 10.4 The audit-trail question when the wrong tool had write access

The scenario that makes the case: the agent chose a fee-reversal tool where a refund was intended. Both are writes, both are authorized for the role, both are plausible from the request. The money moved. The reconciliation team finds it a week later.

What an investigation needs, and what most agent stacks cannot produce:

- The **offered set** at the time, to establish whether the correct tool was even available.
- The **description versions**, to establish whether the distinction was written down.
- The **arguments as resolved**, to establish whether the amount was right even under the wrong tool.
- The **approval record**, to establish whether a human saw the resolved values.
- The **post-condition result**, to establish what the system believed had happened.

Five artefacts, all cheap to record at decision time, none reconstructable afterwards. That asymmetry - trivial to capture, impossible to rebuild - is the argument for treating routing as an audit subject rather than a model-quality subject.

### 10.5 What the institution should not claim

It should not claim that a well-described tool set makes the agent's choices safe. Descriptions improve the rate; authorization, thresholds and four-eyes bound the damage. A control framework that rests on description quality alone is a control framework with no enforcement layer, and it will not survive its first incident review.

---

## 11. The Failure Modes

Six patterns that recur across tool sets. Each is written as symptom, cause and guardrail, because the three are usually known by different people.

### 11.1 Two tools with overlapping descriptions

- **Symptom**: the wrong one is chosen intermittently, and the same request routes differently on different days. Retries with no prompt change produce different names.
- **Cause**: the discriminating fact - a precondition, a cardinality, a side effect - exists in the implementation and nowhere in the text. The model chooses on texture, and texture is sensitive to ordering and phrasing.
- **Guardrail**: a boundary statement in both descriptions, plus a routing test case that contains both tools and a request that must resolve to one. The test is the guardrail; the sentence is the fix.

### 11.2 A tool that is selected but should never have existed

- **Symptom**: an action was taken that nobody intended to authorize, by a tool that everyone agrees should not have been in the set. Often it is a debug tool, an admin tool, or a legacy path kept "for a while".
- **Cause**: availability was decided by convenience rather than by role or stage. The tool was reachable, so it was offered; once offered, it is a candidate.
- **Guardrail**: the existence guardrail of section 8.3 - the tool is not offered, so there is nothing to select. If it must exist, it exists behind a separate principal that the agent does not hold.

### 11.3 A silent fallback that hides a routing failure

- **Symptom**: task success looks stable while the routing path degrades. The agent stops using the specialized tool and starts using a generic one, or answers from context. Answers are usually still acceptable, occasionally wrong.
- **Cause**: the harness has a fallback that produces a plausible answer when a call fails or is not made, and nothing records that the fallback fired. The failure is absorbed rather than surfaced.
- **Guardrail**: make every fallback an event with a reason, count fallback rate per request class as a first-order metric, and alert on it. A fallback that is silent is not a safety net; it is a blindfold.

### 11.4 An argument constructed confidently from a wrong reading

- **Symptom**: the right tool, a schema-valid call, a wrong outcome. Often the identifier is from the wrong record, or the amount is from the wrong field of a prior result.
- **Cause**: the model composed a value that the schema accepts and the request plausibly implies, and no post-condition check existed to catch the semantic mismatch. Confident phrasing in the transcript is not evidence of correctness.
- **Guardrail**: resolve identifiers in code, echo the resolved values back for confirmation on writes, and assert a tool-specific post-condition. The schema is a shape check, not a meaning check.

### 11.5 A tool set grown past what any description can disambiguate

- **Symptom**: description quality is high, review is diligent, and selection still degrades. Adding a tool now visibly disturbs the routing of unrelated tools.
- **Cause**: descriptive disambiguation does not scale linearly. Each new near neighbour requires clauses in every neighbour's description, and the clause set grows faster than the tool count. At some density, the descriptions become unreadable and the distinctions collapse.
- **Guardrail**: shape the set rather than describe around it - split by role and stage, consolidate genuine families, and treat the tool count as a budget with an owner. Google's published 10-20 guidance is the only provider figure available and is a reasonable default to argue against (✅ as documented, prescriptive not measured).

### 11.6 A routing change nobody noticed because the eval only checked the final answer

- **Symptom**: an evaluation suite is green; the routing path has changed. A model upgrade, a description edit, a reordering of the tool list, or a consolidation moved traffic from one tool to another, and the answers stayed good enough.
- **Cause**: the eval grades outcomes, and outcomes are robust to which tool was used right up until they are not. No assertion exists on the chosen name, so a change in choice is invisible.
- **Guardrail**: assert on the name, per request class, in addition to the outcome - and treat a shift in the name distribution as a signal worth investigating even when outcomes hold. This is the boundary with the drift guide: *that* the distribution changed is measured there; *why the text made it change* is answered here.

---

## 12. The Cymbal Bank Worked Example

### 12.1 The agent

Cymbal Bank runs a card and deposit servicing assistant. It answers customer questions in a chat channel, and it can act on a narrow set of servicing operations within limits. Its tool set, before the work described here, was eleven tools:

```
get_customer            search_customers
get_account_balance     get_available_balance
list_transactions       search_transactions
get_transaction         place_hold
reverse_fee             issue_refund
escalate_to_human
```

Every tool was correctly implemented, correctly authorized for the servicing role, and documented in the schema repository. The team's view was that routing was fine, because the schema was fine.

### 12.2 The incident that started the review

A customer asked what they could spend on their card while a large hotel authorization was pending. The assistant called `get_account_balance` and quoted the posted ledger balance, which did not yet reflect the hold. The customer spent against a number that was too high. The overspend was small; the finding was not, because the same pair of tools was reachable by any request that mentioned "balance", and nothing in either description said which one answered "what can I spend".

The post-incident review found three things, in this order:

1. The chosen name was correct by the schema and wrong by the question. No test asserted on the name, so nothing had caught it.
2. The two descriptions were, in full, "Returns the account balance" and "Returns the available account balance". The distinction existed only in the tool names, and only for a reader who already knew it.
3. The model had no way to know a hold was pending, because neither description mentioned holds.

### 12.3 The decision the routing view changed

The team's first proposal was a prompt instruction: "when the customer asks what they can spend, use the available balance tool". That is a routing fix in the wrong layer - it works for the phrasing in the incident and fails for "how much is left on my card", "can I cover this", "what is my limit today".

The routing view produced a different decision: **collapse the pair into one tool and make the distinction an argument**.

```
BEFORE                            AFTER
get_account_balance               get_balance
get_available_balance               basis: "posted" | "available"
                                    (required, no default)
```

with a single description that states what each basis means, that "available" is the default answer to a spendability question, and that "posted" is for statement and reconciliation contexts.

Three consequences, each of which the team had to accept deliberately:

- **The decision moved from step 2 to step 3.** There is no longer a wrong name to choose. There is now a required argument that must be set correctly, which is a *smaller* error surface (one enum, no plausible alternative label) but a *less visible* one - a wrong `basis` looks like a valid call. Mitigation: the enum is required with no default, so the model must state a basis, and the harness logs it as a routing-relevant field.
- **The count went down by one, and the coherence went up.** Eleven tools became ten, and more importantly the balance family stopped competing with itself.
- **The prompt instruction was deleted.** The policy moved into the interface: the description says which basis answers which question, and the harness asserts that a spendability-class request carries `basis: available`.

### 12.4 What changed in the audit trail

The audit record gained three fields it did not have: the `basis` argument as a first-class routing attribute, the description hash of the consolidated tool, and a request-class tag on every balance enquiry. The first made the decision reviewable, the second made it reconstructable, the third made the absence of a call observable - the failure mode of section 7.4, which had been invisible before.

A second, smaller change followed from the same review: `reverse_fee` and `issue_refund` were both offered during discovery, before the transaction had been identified. The stage guardrail moved the two write tools out of the discovery tool set entirely. The routing error rate did not change; the *blast radius* of a routing error did, because the tools were no longer reachable at the moment the model was least informed.

### 12.5 What Cymbal did not establish

- The consolidation was evaluated on Cymbal's own replay set, not on a public benchmark, and the result is not comparable to anything published.
- The claim that a required enum is safer than a second label is a design argument, not a measured finding. It reduced the error surface by construction; it did not eliminate semantic errors in the argument.
- Nothing here shows the tool set is now correctly routed. It shows that one class of ambiguity was removed and that the remaining decisions became observable, which is the most a routing review can honestly claim.

The lesson the team wrote down, in their own words, was the thesis of this guide: they had spent a quarter improving schemas, and the failure had been in the names.

Which is the general form of it: at the moment of choice there was no tool to inspect, only two labels and two sentences - and the sentences did not separate them. The model did what it always does. It read the names.

---

## 13. The Anti-Patterns

1. **Treating the description as documentation.** Writing it for a human reviewer after the tool works, then never reading it again. *Correction*: write it for the selector, review it as an interface, and version it with the tool.
2. **Fixing routing with prompt instructions.** "Always use X for Y" works for the phrasing in the incident and fails for the next paraphrase. *Correction*: put the distinction in the description, or remove the ambiguity from the tool set.
3. **Adding an example instead of separating the labels.** A longer prompt with more examples raises the rate on the examples and does nothing for the labels that remain indistinguishable. *Correction*: reduce the choice set or write the discriminator.
4. **Measuring only the final answer.** Outcome grading cannot see a routing change that happened to produce an acceptable answer. *Correction*: assert on the chosen name per request class, alongside the outcome.
5. **Assuming schema validity means correctness.** Strict mode guarantees shape. *Correction*: post-conditions on writes, resolved identifiers in code, confirmation on irreversible actions.
6. **Offering every tool to every agent.** The default framework behaviour, and the reason tool sets accrete. *Correction*: derive the offered set from role and stage; treat set membership as an entitlement.
7. **Blind retry on failure.** Repeating the same decision with the same evidence. *Correction*: one bounded re-prompt with new evidence, then escalate; never auto-retry a write.
8. **Silent fallback.** A generic path that hides a routing failure and keeps the success metric green. *Correction*: every fallback is an event with a reason, counted per request class.
9. **Deprecating by comment.** Keeping a tool whose description says "deprecated" without changing what happens when it is called. *Correction*: removal or alias, decided in the change process, with an eval run before and after.
10. **Consolidating without moving the check.** Merging tools into a mode parameter removes a name choice and creates an argument choice. *Correction*: make the mode required, log it as a routing attribute, and assert on it.
11. **Trusting the model to respect a limit it was told about.** A description clause is a hint. *Correction*: enforce limits in dispatch against resolved arguments.
12. **Letting retrieval stand in for the decision.** Taking the top match as the answer, with no representation for "none", "both" or "not yet". *Correction*: retrieve a shortlist, then route over it with constraints intact.

---

## 14. The State of Practice

### 14.1 What providers document

Documented, in provider documentation read during this pass:

- **Descriptions carry the selection signal.** Anthropic's tool-definition guidance calls detailed descriptions the most important factor in tool performance, asks for when-to-use and when-not-to-use clauses, and recommends at least three to four sentences (✅). Its engineering post makes description prompt-engineering a first-class technique (⚠ as to effect size).
- **Fewer, consolidated tools reduce selection ambiguity.** Claude's guidance recommends consolidating related operations behind one tool with an action parameter, and namespacing by service (✅ as guidance).
- **A count ceiling exists as guidance.** Google publishes 10-20 active tools maximum (✅ as documented; prescriptive, not measured).
- **The name is a constrained emission.** `tool_choice` settings and strict mode mean the emitted name must be one of the offered names and the arguments must satisfy the schema (OpenAI and Claude docs, ✅; Gemini's function-calling modes are the same idea with different names).

### 14.2 What measurement establishes

- Selection is a capability distinct from call construction, and models fail it in ways that are reproducible across instruments (MetaTool, BFCL).
- Selection is sensitive to presentation: ordering, and small description perturbations (BiasBusters).
- Retrieval in front of the model reduces prompt cost and can improve a selection-only metric against a bloated baseline (RAG-MCP) - and does not measure task success.
- Reliability, not just capability, is the operative property: the same request can route differently on repeat (tau-bench's repeated-trial framing).

### 14.3 What is conjecture

Stated as conjecture, because it is what it is:

- That a description-first discipline produces measurably better routing in a production estate. The mechanism is sound and the practitioner reports are consistent, but no controlled study isolates description quality from tool-set size, model capability and request distribution.
- That there is a tool-count cliff, and where it is. The published guidance is a prescription with no method.
- That consolidating tools is net safer. It removes name ambiguity and adds argument ambiguity; the net effect is unmeasured.
- That routing failures and argument failures have a stable ratio across domains. No instrument separates them cleanly enough to say.

### 14.4 The practical position

Design as if the description is the interface, because the mechanism says it is and the cost of being wrong is low. Enforce as if the model will get it wrong, because sometimes it will and the cost of being wrong there is high. Measure the name, not only the answer. Keep the tool set small enough that a human can hold the distinctions in mind - which is, not coincidentally, about the size at which the descriptions can still be read.

---

## 15. The Claims Audit and What Could Not Be Verified

| # | Claim | Verdict | Source, quality, date |
|---|---|---|---|
| 1 | The model emits a tool name drawn from the offered set; the function behind the name is not inspected at selection time | ✅ Verified | Provider tool-use documentation (OpenAI, Anthropic, Google), primary, read October 2026 |
| 2 | Anthropic guidance: descriptions are the most important factor in tool performance; include when to use and when not to; at least three to four sentences | ✅ Verified | Claude Platform docs, "Define tools", primary, read October 2026 |
| 3 | Anthropic guidance: consolidate related operations into fewer tools to reduce selection ambiguity; namespace by service | ✅ Verified as guidance | Same page, primary |
| 4 | Anthropic engineering: small refinements to tool descriptions can produce large behavioural improvements | ⚠ Vendor claim | "Writing effective tools for agents", 11 September 2025; anecdotal, no published method |
| 5 | Anthropic reports prefix-versus-suffix naming has non-trivial effects on its own tool-use evaluations, varying by model | ⚠ Vendor claim | Same post, 11 September 2025; no figures or eval description |
| 6 | Google guidance: keep the active tool set to 10-20 tools maximum | ✅ Verified as documented guidance | ai.google.dev function-calling best practices, page dated 2026-09-23; prescriptive, no method |
| 7 | OpenAI strict mode makes calls adhere to the schema rather than being best effort | ✅ Verified | platform.openai.com function-calling guide, primary, read October 2026 |
| 8 | Tool choice can be constrained to auto, required, a named function, a subset, or none | ✅ Verified | Same page, primary |
| 9 | MetaTool separates tool-use awareness from tool selection, includes selection among similar tools, and reports that most tested models struggled | ✅ Verified from the abstract | Huang et al., arXiv 2310.03128, 4 Oct 2023 (v6 Dec 2024) |
| 10 | MetaTool recommends rewriting tool descriptions with a model appropriate to the downstream LLM | ✅ Verified from the abstract | Same |
| 11 | BiasBusters: models favour earlier tools; small description perturbations shift choices; query-metadata semantic alignment is the strongest driver | ✅ Verified from the abstract; peer-reviewed | Blankenstein et al., arXiv 2510.00307, ICLR 2026 camera-ready |
| 12 | RAG-MCP retrieves over descriptions before the model sees them and reduces prompt tokens against a bloated baseline | ✅ Verified from the abstract | Gan and Sun, arXiv 2505.03275, 6 May 2025 |
| 13 | RAG-MCP is measured on selection only, in a simulated catalogue, with no task-success metric | ✅ Verified from the abstract and framing | Same |
| 14 | tau-bench evaluates tool-agent-user interaction under a domain policy and measures reliability over repeated trials | ✅ Verified from the abstract | Yao et al., June 2024 |
| 15 | BFCL includes a function-relevance-detection category in which none of the offered functions is correct and the expected output is no call | ✅ Verified | BFCL blog and site, UC Berkeley, read October 2026 |
| 16 | ToolBench/ToolLLM builds a very large real-API corpus with single- and multi-tool instructions and uses a retriever over API descriptions | ✅ Verified from the abstract | Qin et al., arXiv 2307.16789, 2023 |
| 17 | No leaderboard score, percentage or model ranking is printed in this guide; benchmarks are cited only as evidence of mechanism | ✅ Verified by construction | This file, held to the standard during writing and checked in the final verification pass |
| 18 | Retrieval cannot represent "use both, in this order" or "call nothing" | ⚠ Design argument | Stated as reasoning, not measured; no study isolates the sequencing case |
| 19 | Consolidating tools moves the routing error from name selection to argument selection | ⚠ Design argument | Mechanism follows from the taxonomy; no measurement found |
| 20 | Provider description-writing guidance is prescriptive and not derived from a published measurement | ✅ Verified as a negative finding | Read the guidance pages; none publishes a method or a study |
| 21 | A wrong call that is schema-valid, authorized and expensive is not scored by any instrument reviewed here | ✅ Verified as a negative finding | Read the six instrument descriptions; correctness is scored, cost is not |
| 22 | "The model routes on the description, not the capability" is fully established | ❌ Rejected as an overstatement | The mechanism supports it and BiasBusters is consistent with it, but no study isolates description from capability, ordering and model prior |

### What Could Not Be Verified

- **No public study isolates the sequencing failure from the selection failure.** Instruments score a multi-tool task as a whole. Whether a failed task had a correct first choice, a correct second choice with wrong arguments, or a wrong second name is not reported anywhere this pass could find. The claim in section 9.3 is therefore a statement about the instruments' design, not a measured quantity.
- **No measurement of description quality as an isolated variable.** Every instrument that varies descriptions varies something else as well (RAG-MCP varies retrieval, BiasBusters varies metadata holistically). The description-first position in section 4 rests on mechanism plus practitioner report, and is labelled as such.
- **The tool-count threshold is unpublished.** Google's 10-20 guidance carries no method, no study and no model list. It is quoted as guidance, not as a finding, and the guide says so wherever the number appears.
- **No cost model for a wrong tool call.** The economically important failures - duplicate payment, wrong-customer disclosure, unnecessary hold - have no published measurement, and this guide does not invent one. Section 10's argument is about auditability, not about loss quantification.
- **No verification of the effect size of consolidation.** The claim that merging tools reduces routing error while increasing argument error is a design argument from the taxonomy in section 2. It was not measured here and no study was found that measures both sides.
- **Web search was unreliable on this host during writing.** Several queries returned empty; primary pages were fetched directly instead. An empty search is recorded as a tool limitation, not as evidence that the material does not exist.

---

## 16. Glossary, Cross-References and Closing Summary

### 16.1 Glossary

**Act decision** - The first step of routing: whether to call any tool at all, answer from context, ask a clarifying question, or refuse. Failure in either direction is a routing failure, and the "no call when one was needed" direction is the harder one to observe.

**Argument construction** - Filling the tool's input schema with values. Constrained in shape by the schema and free in meaning; a schema-valid argument can be semantically wrong, which is why post-conditions exist.

**Disambiguation** - Any technique that increases the distinguishability of two or more labels: boundary statements, negative clauses, naming discipline, consolidation, retraction.

**Dispatch** - The harness side of a call: validate, authorize against resolved arguments, execute, capture the result, return a typed error or success. The enforcement layer, as distinct from the decision layer.

**Fallback** - A path that produces an answer when a call fails or is not made. Legitimate as a safety net, dangerous as a silent one: an unlogged fallback converts a routing failure into a quality metric that still looks healthy.

**Post-condition** - A tool-specific check that the result is consistent with the intent, not merely well-formed. The only defence against a schema-valid but semantically wrong argument.

**Result interpretation** - The fourth step of routing: deciding whether the returned result is sufficient, an error to retry, a partial answer needing a second call, or evidence that the choice was wrong.

**Routing** - The whole act of getting from an incoming request to an executed call, including the decision not to call anything. Owned jointly by the harness (shape, authority, enforcement) and the model (the choice itself).

**Selection** - The narrow step of emitting one tool name from the offered set. A classification over a closed label set, performed on descriptions, and the step this guide exists to develop.

**Tool description** - The prose attached to a tool definition. Under this guide's thesis it is the interface, not the documentation: it is the only evidence the model has about when the tool applies.

**Tool schema** - The machine-readable input contract: property names, types, enums, required fields, sometimes output shape. Constrains shape; says nothing about when the tool applies.

**Tool set** - The set of tools offered to a given agent at a given moment. Its size, coherence and stability are routing decisions made by the harness, not properties of the catalogue.

### 16.2 Cross-references

Same directory:

- [mcp_progressive_disclosure_guide.md](mcp_progressive_disclosure_guide.md) - deferred loading, tool search, code mode, instruction-level disclosure. Read it for *what enters the context window*; this guide assumes the tool is already there.
- [mcp_discovery_guide.md](mcp_discovery_guide.md) - registries, directories, marketplaces, enterprise discovery. Read it for *where the tool came from*; this guide starts after adoption.
- [mcp_framework_tools_guide.md](mcp_framework_tools_guide.md) - MCP concepts, architecture, server implementation, protocol security. Read it for *how the tool is exposed*.
- [ai_agent_drift_guide.md](ai_agent_drift_guide.md) - tool selection drift as a named drift type, with detection and measurement. Read it for *whether selection has changed*; this guide explains *why the text made it change*.
- [agent_harness_engineering_guide.md](agent_harness_engineering_guide.md) - the ETCLOVG layer taxonomy; sections 3.4 and 3.5 name the Tools layer and the coverage-versus-decision-quality tension this guide develops.
- [agent_experience_guide.md](agent_experience_guide.md) - the tool-as-interface argument from the designer's side, including the provider guidance on writing tools for agents.
- [llm_agents_failures_production_guide.md](llm_agents_failures_production_guide.md) - failure taxonomy at the system level, of which routing failure is one class.
- [prompt_injection_guide.md](prompt_injection_guide.md) - why a description clause is not a control: injected content can override instructions the model was given.

Banking:

- [../../banking/agentic_grc_guide.md](../../banking/agentic_grc_guide.md) - governance, risk and control framing for agentic systems; the audit-trail argument in section 10 lands here.
- [../../banking/ai_risk_register_guide.md](../../banking/ai_risk_register_guide.md) - how a routing failure is recorded as a risk with an owner, rather than as a model-quality note.

### 16.3 Closing summary

Routing is the step the rest of the tooling conversation assumes. Discovery establishes that a capability exists; disclosure decides whether its definition is in the prompt; the framework decides how it is exposed; drift detection notices when the pattern of choices changes. None of them decides, on this request, whether to act, through which name, with which arguments, and what to do when the answer is wrong. That is routing, and it is a writing problem and a control problem before it is a model problem.

Three things follow, and they are the actionable residue of the guide. Write descriptions as interfaces, with the discriminator, the negative clause and the adjacent tool named. Shape the tool set in code, small and stage-appropriate, so that the wrong choice is often not available. Enforce in dispatch, against resolved arguments, because a model that was told about a limit is not a model that respects it.

And keep the distinction that the whole argument rests on: retrieval finds candidates, routing makes a decision, and only one of those can say no.

The model does not select a tool; it selects a name - and everything that makes the name ambiguous is the routing problem.
