# Multi-Pass LLM Pipelines: Architectures, Patterns, and Best Practices

> A comprehensive guide to chaining multiple LLM passes for improved output quality, reliability, and complex task completion.
>
> Part of the LLM/AI Engineering series. Also see: [MLOps Lifecycle Guide](../mlops_lifecycle_guide.md), [LLM Compression Guide](../llm_compression_guide.md), [Guard Models Guide](../guard_models_guide.md), [ML Platform Guide](../ml_platform_guide.md).

---

## 1. What Are Multi-Pass LLM Pipelines?

A **multi-pass LLM pipeline** chains multiple LLM calls — sequential, parallel, or hybrid — to decompose complex tasks, verify outputs, refine results, or combine perspectives. Each "pass" has a narrow, focused responsibility.

### 1.1 Single-Pass Limitations

| Limitation | Description |
|---|---|
| **Hallucination** | Plausible but factually incorrect content with no verification |
| **Missing details** | Complex multi-step tasks compressed into one output cause omissions |
| **Shallow reasoning** | No intermediate deliberation or backtracking from wrong paths |
| **Format inconsistency** | Hard to satisfy content + format + style constraints in one shot |
| **No self-correction** | Errors in early reasoning propagate with no recovery |

### 1.2 Multi-Pass Pipeline Anatomy

```
Single-Pass:  Input ──► [LLM Call] ──► Output
Multi-Pass:   Input ──► [Pass 1: Decompose] ──► [Pass 2: Extract] ──► [Pass 3: Verify] ──► [Pass 4: Refine] ──► Output
```

Each pass has a specific purpose: decompose, extract, classify, verify, critique, refine, format, or aggregate. Intermediate state flows between passes via an orchestrator.

### 1.3 Key Concepts

- **Pass (Stage):** One LLM invocation with defined input, prompt, and output schema.
- **Pipeline topology:** Sequential, parallel, conditional, cyclic, or graph-based structure.
- **Intermediate state:** Data flowing between passes (raw outputs, structured records, scores).
- **Orchestrator:** Runtime managing pass execution, state propagation, error handling, and retries.
- **Termination condition:** Fixed iterations, quality thresholds, or timeout.

---

## 2. Why Multi-Pass Matters

### 2.1 Accuracy via Decomposition

Complex tasks contain sub-problems. Multi-pass lets each call focus exclusively on its sub-task — a classifier sees only classification instructions, a validator checks only correctness criteria. Accuracy improvements of **5–30%** are typical depending on task and base model.

### 2.2 Hallucination Reduction

A dedicated verification pass — often using a different prompt, model, or even a separate LLM — catches hallucinations from the generation pass. **Generate-then-verify** and **critic** patterns are especially effective.

### 2.3 Complex Reasoning

Multi-step reasoning (mathematical problem-solving, multi-hop QA, legal analysis) benefits from spreading reasoning across passes: CoT makes intermediate reasoning explicit; ToT explores branches; Self-Consistency aggregates over multiple reasoning chains.

### 2.4 Self-Correction and Refinement

The **generate → critique → refine** loop (Reflection pattern) lets a model: (1) produce an initial answer, (2) analyse its output for errors, (3) generate an improved version. Improves factual accuracy by 10–25% on QA and summarisation (Madaan et al., 2023; Shinn et al., 2023).

### 2.5 Ensemble Diversity

Multiple calls with different prompts, models, temperatures, or seeds produce diverse outputs. Aggregating these (voting, ranking, merging) yields more robust results than any single generation.

### 2.6 Constraint Satisfaction

Multi-pass enforces hard constraints incrementally: generate freely → check constraints → repair violations → re-check. Far more reliable than asking one pass to satisfy all constraints.

---

## 3. Core Pipeline Patterns

### 3.1 Sequential Chaining

Output of one LLM call feeds as input to the next. `Extract → Classify → Format`.

- **Use when:** Clear linear decomposition with step-wise dependencies.
- **Pros:** Simple to implement, clear data flow.
- **Cons:** Latency = sum of all passes; error in early pass propagates.
- **Example (LangChain LCEL):**
  ```python
  chain = (
      {"doc": extract_fields}
      | classify_document
      | validate_fields
      | format_output
  )
  result = chain.invoke({"raw_text": text})
  ```

### 3.2 Parallel Fan-Out

Multiple independent LLM calls on the same input, then aggregate (merge, vote, pick-best).

- **Use when:** Diversity of output is valuable — multiple perspectives, hallucination risk, subjective tasks.
- **Pros:** Parallel → latency ≈ max of individual calls; diversity improves robustness.
- **Cons:** N× cost; aggregation step is a potential bottleneck.
- **Example (asyncio):**
  ```python
  async def fan_out(text: str) -> str:
      summaries = await asyncio.gather(*[
          llm_call(prompt_bullet, text),
          llm_call(prompt_narrative, text),
          llm_call(prompt_tldr, text),
      ])
      return await llm_call(prompt_select_best, summaries)
  ```

### 3.3 Staged Refinement (Generate → Critique → Refine)

Iterative cycle: generate, critique, improve. Often called the **Reflection pattern**.

- **Use when:** Open-ended creative tasks, code generation, content writing.
- **Pros:** Progressive quality improvement, self-correcting.
- **Cons:** Unbounded runtime — needs max-iteration guard; diminishing returns after 2–3 iterations.
- **Implementation:**
  ```python
  def refine_cycle(prompt: str, max_iter: int = 3) -> str:
      output = llm_call(prompt_generate, prompt)
      for i in range(max_iter):
          c = llm_call(prompt_critique, {"output": output, "task": prompt})
          if c["score"] >= THRESHOLD: break
          output = llm_call(prompt_refine, {"output": output, "critique": c})
      return output
  ```

### 3.4 Verify-Then-Generate

First verify constraints/preconditions, then generate within verified bounds.

- **Use when:** High-stakes tasks where generating outside constraints is dangerous (medical, legal, financial).
- **Pros:** Safety-first; clear error handling path.
- **Cons:** Constraint verification can miss edge cases; can be overly conservative.

### 3.5 Generate-Then-Verify

Generate candidate, verify against criteria, regenerate with feedback if verification fails.

- **Use when:** Verifiable outputs — code (compile/run tests), structured data (schema validation), factual claims (search verification).
- **Pros:** Highly reliable for verifiable outputs; can use deterministic tools for verification.
- **Cons:** Regeneration may produce same errors; needs retry budget.
- **Example:** Code generation → compile/lint → run tests → feed errors back → fix.

### 3.6 Decomposition (Divide and Conquer)

Break complex task into sub-tasks, solve each with a focused LLM call, compose results.

- **Use when:** Complex, multi-faceted problems that partition cleanly.
- **Pros:** Each sub-problem is simpler → higher per-pass quality; sub-tasks parallelisable.
- **Cons:** Decomposition quality is critical; composition must handle cross-references.

### 3.7 Map-Reduce

Process large inputs in chunks (map), then combine results (reduce).

- **Use when:** Processing long documents, logs, or datasets exceeding context windows.
- **Pros:** Handles arbitrarily long inputs; map phase is embarrassingly parallel.
- **Cons:** Chunk boundaries can lose cross-chunk context; reduce quality depends on map quality.

### 3.8 Judge-LLM Pattern

Separate LLM evaluates output quality and triggers regeneration if below threshold.

- **Use when:** Subjective quality criteria (creativity, helpfulness, safety).
- **Pros:** Flexible quality gate; judge can be a different (cheaper) model.
- **Cons:** Judge LLM itself can be unreliable; adds latency and cost per evaluation.

---

## 4. Multi-Agent Patterns

### 4.1 Orchestrator-Worker

One orchestrator LLM delegates sub-tasks to specialised workers, then composes results.

- **Use when:** Complex projects requiring diverse expertise (reports, research, architecture).
- **Workers can use different models** — cheap for simple tasks, powerful for complex reasoning.
- **Requires** careful design of the delegation protocol and error recovery strategy.

### 4.2 Debate Pattern

Multiple LLMs debate a question (each arguing a position), then converge on an answer via a judge or iterative refinement. Based on "Improving Factuality and Reasoning through Multiagent Debate" (Du et al., 2023).

- **Use when:** Controversial or ambiguous questions benefiting from multiple viewpoints.

### 4.3 Round-Table Pattern

Multiple LLMs discuss a problem iteratively, each building on others' responses in a shared conversation context.

- **Use when:** Problems benefiting from brainstorming and collaborative refinement.

### 4.4 Reflection Pattern

A single LLM reflects on its own output, identifies issues, and suggests improvements.

- **Variants:** Self-Refine (Madaan et al., 2023), Reflexion (Shinn et al., 2023), CRITIC (Gou et al., 2023).
- **Use when:** The model can recognise its own weaknesses — code review, essay writing.

### 4.5 Critic Pattern

A separate LLM (the critic) evaluates the primary LLM's output with instructions to find flaws.

- **Best practice:** Use a different model family for critic (e.g., Claude critiques GPT-4); instruct the critic to be harsh; output structured scores + free-text feedback.
- **Use when:** High-stakes outputs needing independent verification.

### 4.6 RAG with Multi-Pass

`Retrieve → Generate → Verify Against Documents → Refine → Guard Output`

1. **Retrieve** documents from vector DB or search.
2. **Generate** answer using retrieved context.
3. **Verify** each claim against source documents; mark unsupported claims.
4. **Refine** — drop or flag unsupported claims; add citations.
5. **Guard** — final safety/format check.

### 4.7 Validator Pattern

After generation, run separate passes for safety, format compliance, and constraint satisfaction. Can use dedicated guard models (NeMo Guardrails, Guardrails AI).

### 4.8 Tool-Use Pattern

LLM generates tool calls, tool results feed back, LLM incorporates them. Fundamentally multi-pass: plan → format call → execute (non-LLM) → interpret result → decide next.

---

## 5. Reasoning Patterns

### 5.1 Chain of Thought (CoT)

Intermediate reasoning steps within a single pass. Append "Let's think step by step" or provide few-shot examples with explicit reasoning.

- **Variants:** Zero-shot CoT, Few-shot CoT, Least-to-Most Prompting.
- **Improves accuracy:** 10–20% on math and logic tasks.

### 5.2 Self-Consistency

Multiple independent CoT samples (temperature > 0), then majority vote on final answer. Improves accuracy 5–15% over single CoT (Wang et al., 2022).

- **Cost:** ~N× (parallelisable); **latency:** ~1× if parallelised.
- **Use when:** Single correct answer exists, multiple valid reasoning paths.

### 5.3 Tree of Thoughts (ToT)

Explore multiple reasoning paths, evaluate intermediate states, use BFS/DFS with pruning (Yao et al., 2023).

1. **Thought decomposition:** Break into intermediate steps.
2. **Generation:** k candidate next thoughts per state.
3. **Evaluation:** LLM scores each state's promise.
4. **Search:** BFS or DFS with pruning.

- **Cost:** ~10–100× single-pass; **quality improvement:** +20–40% on hard puzzles (Game of 24, crossword).

### 5.4 Plan-and-Solve

First pass produces a plan (high-level steps), second pass executes each step, optional third pass verifies.

---

## 6. Multi-Pass by Task Type

| Task | Pipeline | Typical Delta |
|---|---|---|
| **Document processing** | Classify type → Extract fields → Validate schema → Format | +10–20% F1 |
| **Code generation** | Gen → Static analysis → Run tests → Fix → Security scan | +20–30% pass@1 |
| **Content creation** | Outline → Draft → Critique → Refine → Fact-check → Polish | +0.5–1.0/5 human eval |
| **Data analysis** | Query → Interpret → Validate stats → Summarise | +15–25% accuracy |
| **Translation** | Translate → Back-translate → Compare → Refine | +3–8 BLEU points |
| **Summarisation** | Extract key points → Rewrite for style → Fact-check | +15–25% relevance |
| **Reasoning/Math** | CoT samples → Majority vote → Self-verify | +5–15% (CoT-SC) |
| **RAG** | Retrieve → Gen → Verify against docs → Refine → Guard | +10–25% citation accuracy |
| **Classification** | Multiple classifiers → Ensemble vote | +3–8% accuracy |
| **Conversation** | Plan → Respond → Reflect → Improve | +10–20% user satisfaction |

### Key Pipelines in Detail

**Document processing:**
```
Pass 1: Cheap model classifies document type
Pass 2: Extract fields based on type
Pass 3: Validate against type-specific schema (regenerate if invalid)
Pass 4: Format as JSON
```

**Code generation:**
```
Pass 1: Generate code from spec
Pass 2: Deterministic checks (lint, type-check)
Pass 3: Run unit tests
Pass 4: Feed errors to LLM, regenerate
Pass 5: Security scan
```
Deterministic passes provide ground-truth feedback, making the LLM fix cycle highly reliable.

**Translation with back-translation:**
```
Pass 1: Translate source → target
Pass 2: Back-translate target → source (could use different model)
Pass 3: Compare original vs back-translation; identify meaning drift
Pass 4: Refine to eliminate drift (+15–25% meaning preservation)
```

---

## 7. Implementation Approaches

### 7.1 LangChain (LCEL + LangGraph)

**LCEL** — simple sequential chaining with `RunnablePassthrough`:
```python
chain = (
    RunnablePassthrough.assign(extracted=lambda x: extract_llm.invoke(x["text"]))
    | RunnablePassthrough.assign(validated=lambda x: validate_llm.invoke(x["extracted"]))
    | RunnablePassthrough.assign(formatted=lambda x: format_llm.invoke(x["validated"]))
)
```

**LangGraph** — stateful, cyclic, conditional pipelines with state machine:
```python
from langgraph.graph import StateGraph, END
graph = StateGraph(PipelineState)
graph.add_node("generate", generate)
graph.add_node("critique", critique)
graph.add_node("refine", refine)
graph.add_conditional_edges("critique", should_continue, {"accept": END, "refine": "refine"})
graph.set_entry_point("generate")
```

| Feature | LCEL | LangGraph |
|---|---|---|
| Topology | Linear | Cyclic / DAG |
| State management | Simple data flow | Full state machine |
| Error recovery | Limited | Built-in retry / fallback |
| Human-in-loop | No | Supported |

### 7.2 LlamaIndex Workflows

Declarative multi-step pipelines via Python decorators:
```python
class DocWorkflow(Workflow):
    @step async def classify(self, ctx, ev: StartEvent) -> ClassifyEvent: ...
    @step async def extract(self, ctx, ev: ClassifyEvent) -> ExtractEvent: ...
    @step async def validate(self, ctx, ev: ExtractEvent) -> StopEvent: ...
```

### 7.3 DSPy

Declarative signatures + automatic prompt optimisation:
```python
class ExtractFields(dspy.Signature):
    """Extract fields from invoice text."""
    text = dspy.InputField()
    fields = dspy.OutputField(desc="JSON with invoice_number, date, total, vendor")

class InvoicePipeline(dspy.Module):
    def __init__(self):
        self.extract = dspy.ChainOfThought(ExtractFields)
        self.validate = dspy.ChainOfThought(ValidateFields)
    def forward(self, text):
        fields = self.extract(text=text).fields
        v = self.validate(fields=fields)
        return fields if v.is_valid else ...

# Compile optimises prompts automatically
optimizer = dspy.teleprompt.BootstrapFewShot()
optimized = optimizer.compile(InvoicePipeline(), trainset=training_data)
```

**Key advantage:** DSPy automatically optimises prompts and few-shot examples for each pipeline stage.

### 7.4 AutoGen

Multi-agent conversations with role-based agents:
```python
planner = autogen.AssistantAgent(name="Planner", llm_config={...})
researcher = autogen.AssistantAgent(name="Researcher", llm_config={...})
critic = autogen.AssistantAgent(name="Critic", llm_config={...})
groupchat = autogen.GroupChat(agents=[planner, researcher, critic], max_round=10)
manager = autogen.GroupChatManager(groupchat=groupchat)
```

### 7.5 CrewAI

Role-based agent teams with task delegation:
```python
planner = Agent(role="Planner", goal="Break down complex tasks")
writer = Agent(role="Writer", goal="Generate high-quality content")
crew = Crew(agents=[planner, writer], tasks=[plan_task, write_task], process=Process.sequential)
```

### 7.6 Semantic Kernel

Plugin-based chaining for the Microsoft ecosystem (.NET, Azure).

### 7.7 Custom Python (asyncio + State Machine)

Maximum flexibility using Python's asyncio and a state machine pattern:
```python
class PipelineState:
    stage: Stage = Stage.EXTRACT
    extracted: dict = field(default_factory=dict)
    retries: int = 0

class StateMachinePipeline:
    async def step(self, state: PipelineState) -> PipelineState:
        match state.stage:
            case Stage.EXTRACT:
                state.extracted = await self.llm.call("extract", state.input)
                state.stage = Stage.VALIDATE
            case Stage.VALIDATE:
                result = await self.llm.call("validate", state.extracted)
                state.stage = Stage.FORMAT if result["valid"] else Stage.EXTRACT
            case Stage.FORMAT:
                state.output = await self.llm.call("format", state.extracted)
                state.stage = Stage.DONE
        return state
```

### 7.8 Framework Selection Guide

| Framework | Best For | Strengths | Learning Curve |
|---|---|---|---|
| **LangChain LCEL** | Simple sequential chains | Easy syntax, rich ecosystem | Low |
| **LangGraph** | Stateful cyclic pipelines | Cycles, state, HITL, durable execution | Medium-High |
| **DSPy** | Programmatic optimised pipelines | Auto prompt optimisation, compiler | Medium |
| **LlamaIndex** | RAG multi-step workflows | Great document processing | Low-Medium |
| **AutoGen** | Multi-agent conversations | Role-based agents, flexible conversation | Medium |
| **CrewAI** | Role-based agent teams | Intuitive agent/task/crew model | Low |
| **Custom Python** | Full control | No dependencies, minimal overhead | High (maintenance) |

**Hybrid approaches** are common in production: LangGraph for orchestration + DSPy for optimising individual LLM calls; AutoGen for agent conversation + LangChain for tool integrations.

---

## 8. Caching and Optimization

### 8.1 Semantic Caching

Cache LLM responses by semantic similarity (embedding cosine similarity ≥ threshold, typically 0.90–0.95) rather than exact match. Multi-pass pipelines with similar intermediate states benefit significantly — 20–60% cost reduction on real-world workloads.

### 8.2 Speculative Execution

Start later passes with placeholder assumptions while earlier passes run. If assumptions hold, latency reduced; if not, recompute. Useful in RAG (start generating while retrieval runs) with a "draft context" assumption.

### 8.3 Parallel vs Sequential Trade-off

| Scenario | Approach | Rationale |
|---|---|---|
| Independent sub-tasks | Parallel | Minimise latency at same cost |
| Dependent sub-tasks | Sequential | Later passes need earlier results |
| Mixed | Hybrid DAG | Max parallelism within dependency constraints |
| Cost-sensitive | Sequential + early exit | Stop when acceptable quality reached |

### 8.4 Cost Management Strategies

| Strategy | How | Savings |
|---|---|---|
| **Cheaper critic** | Haiku/Mini for validation, Sonnet/4o for generation | 50–80% on critic passes |
| **Early exit** | Stop after gen if quality exceeds threshold | Variable |
| **Semantic cache** | Cache by similarity | 20–60% |
| **Model cascade** | Cheapest first, escalate if quality insufficient | 30–60% |
| **Prompt compression** | Reduce input token count per pass | 20–50% |

### 8.5 Model Selection Per Pass

| Pass Type | Recommended Model |
|---|---|
| Simple extraction, classification | GPT-4o-mini, Claude Haiku, Llama 3.1-8B |
| Formatting, style rewriting | Medium (Sonnet, 4o-mini) |
| Complex reasoning, generation | Frontier (GPT-4o, Claude Opus, DeepSeek-V3) |
| Critic, validation | Medium, different model family than generator |
| Code generation | Frontier or code-specialised (DeepSeek-Coder) |

**Rule:** Use cheapest model meeting the pass's quality target. Only the most demanding passes warrant expensive models.

### 8.6 Prompt Caching (KV Cache)

Reuse KV cache across passes by keeping shared prompt prefixes (system prompts, document context) at the beginning. Place varying parts at the end. Supported by Anthropic, OpenAI (variable), Gemini, DeepSeek.

---

## 9. Evaluation and Quality Metrics

| Metric | Definition | Target |
|---|---|---|
| **Pass-through rate** | % of runs completing without rejection | > 95% |
| **First-pass accuracy** | Quality of initial generation | Baseline |
| **Final accuracy** | Quality after all passes | Target |
| **Delta** | Final - First-pass | Positive (negative = pipeline hurts) |
| **Pass-specific quality** | Per-pass intermediate output metrics | Monitor per pass |
| **Cost per task** | Σ(pass tokens × price) | Budget-driven |
| **P50 / P95 latency** | End-to-end timing | SLA-driven |
| **Token efficiency** | Quality score / total tokens | Compare vs single-pass |

**Degradation rate:** % of runs where final < first-pass. If > 5%, refinement passes are harming quality and need investigation.

**Cost breakdown:** Track average cost (successful), average cost (rejected/wasted), and P95 cost for budgeting.

**Per-pass monitoring:** Extraction precision/recall, critique false-positive rate, refinement delta score.

---

## 10. Multi-Pass vs Single-Pass Trade-offs

### Quality Improvement

| Task | Single-Pass | Multi-Pass | Delta |
|---|---|---|---|
| Structured extraction | 70–80% F1 | 85–95% F1 | +10–20% |
| Long-doc summarisation | 60–75% | 80–90% | +15–25% |
| Code generation (pass@1) | 40–60% | 60–80% | +20–30% |
| Math reasoning (GSM8K) | 70–85% | 85–95% | +5–15% |
| Translation (BLEU) | 35–45 | 40–50 | +3–8 pts |
| Classification | 85–92% | 90–96% | +3–8% |

*With frontier models (GPT-4o, Opus, DeepSeek-V3), single-pass quality is higher and multi-pass delta is smaller.*

### Cost & Latency Multipliers

| Pattern | Cost × | Latency (P50) |
|---|---|---|
| Single-pass | 1× | 1–5s |
| Sequential chaining (3 passes) | ~3× (cheap) to ~9× (expensive) | 5–20s |
| Parallel fan-out (3+1) | ~4× | 3–10s |
| Staged refinement (3 iters) | ~3–5× | 10–30s |
| Generate-then-verify (1 retry) | ~2–3× | 5–15s |
| Self-Consistency (5 samples) | ~5× | 5–10s (parallel) |
| Tree of Thoughts | ~10–100× | 30s–5m |
| Multi-agent (3 agents, 5 rounds) | ~15× | 30s–2m |

### Reliability

**Multi-pass improves:** Catches errors via verification, retries recover transient failures, ensemble reduces variance, decomposition prevents reasoning single-point-of-failure.

**Multi-pass introduces new failure modes:** Error propagation (pass 1 error infects all downstream), refinement collapse (over-fitting), judge inconsistency, non-convergent loops, multi-agent coordination errors.

### When Single-Pass Is Enough

| Condition | Rationale |
|---|---|
| Task is simple | Single intent, straightforward generation |
| Frontier model is very capable | May handle complex tasks in one pass |
| Latency critical | Real-time apps (chat, autocomplete) |
| Cost constrained | 3×+ cost may be infeasible |
| Quality already sufficient | Multi-pass gives marginal benefit at significant cost |
| Output is low-stakes | Internal notes, rough drafts, brainstorming |

**Decision flow:**
1. Try single-pass first. Measure quality.
2. Identify specific failure modes.
3. Add one pass at a time, measuring delta.
4. Stop when marginal benefit < marginal cost.

---

## 11. Multi-Pass for Agentic Systems

### 11.1 Plan → Execute → Observe → Reflect Loop

| Step | LLM? | Description |
|---|---|---|
| Plan | Yes | Create action sequence |
| Execute | No | Tool call (API, code exec, search) |
| Observe | Yes | Interpret tool output |
| Reflect | Yes | Evaluate progress, identify errors |
| Decide | Yes | Next action or final answer |

### 11.2 Tool Call Retry

When a tool fails: `Analyse Error (LLM) → Fix Arguments (LLM) → Retry (Tool) → Incorporate (LLM)`

### 11.3 Multi-Step Tool Selection

`Analyse Task → List Tools → Select Best Tool → Format Arguments → Execute` — each an LLM pass.

### 11.4 Agent Planning Pass

Dedicated planning pass: decompose goal into sub-goals, identify required info and tools, order steps with dependencies, identify failure modes and fallbacks.

### 11.5 Error Recovery

| Error | Recovery | LLM Passes |
|---|---|---|
| Tool syntax error | Regenerate with error feedback | 1 |
| Empty result | Reformulate query, try different tool | 1–2 |
| Hallucinated intermediate | Verification catches, request regeneration | 2 |
| Agent loop | Timeout + replan | 1 |
| API unavailable | Fallback to alternative tool | 1 |

---

## 12. Best Practices

### 12.1 Start with a Single-Pass Baseline

Measure single-pass quality before adding passes. Identify specific failure modes. Design only the passes that address them. **Anti-pattern:** Adding passes preemptively without knowing what they'll fix.

### 12.2 Add Passes Incrementally

Add one pass at a time, measure quality impact. If a pass doesn't improve quality by at least your threshold (e.g., 5%), remove it.

### 12.3 Gate Before Accepting Output

Every pipeline needs a gate that checks factual accuracy, format compliance, and safety before returning output to the user. Without a gate, you can't trust the pipeline.

### 12.4 Parallelise Independent Passes

Use `asyncio.gather()` for independent passes (map-phase, multiple CoT samples, parallel validation checks). Can reduce multi-pass latency from N× to ~1×.

### 12.5 Choose Model Per Pass

Match model capability to pass difficulty. Use different model families for generator and critic (diverse biases). Cheap models for simple passes; frontier for complex reasoning.

### 12.6 Cache Aggressively

Semantic caching (similar inputs), exact caching (identical inputs), KV caching (shared prompt prefixes), result caching (expensive passes like retrieval).

### 12.7 Monitor Key Metrics

```
Pipeline Dashboard
├── Pass-through rate (hourly/daily)
├── Avg cost per task (P50, P95)
├── Latency (P50, P95, P99)
├── Pass-specific quality scores
├── Retry rates per pass
├── Top failure modes by pass
└── Single-pass baseline (for comparison)
```

### 12.8 Timeouts and Fallbacks

Every pass needs: **Timeout** (default 30s), **Fallback** (use cache / skip pass / cheaper model / route to human), **Circuit breaker** (disable after repeated failures, alert).

### 12.9 Design for Degraded Operation

| Scenario | Degraded Behaviour |
|---|---|
| Critique unavailable | Skip critique, return direct generation |
| Verification times out | Accept current output (best-effort) |
| Expensive model slow | Fall back to cheaper, faster model |
| Semantic cache down | Run passes normally (higher latency) |
| Orchestrator crashes | State recovery from checkpoint |

### 12.10 Log Intermediate Outputs

Log every pass output for debugging, audit, improvement analysis, and evaluation dataset building. Include: pipeline ID, pass name, model, latency, tokens, cost, success status, and input/output preview.

### 12.11 Test Edge Cases

Test: empty inputs, adversarial prompts, boundary length inputs, ambiguous requests, multi-modal content, failure cascades (what happens when each pass fails).

### 12.12 Document Pipeline Design

For each pipeline document: purpose, topology, per-pass details (model, purpose, fallback), expected quality improvement, known failure modes, cost budget.

---

## 13. Pattern Selection Guide

| Pattern | What It Does | When to Use | Cost × | Quality Δ | Example |
|---|---|---|---|---|---|
| **Sequential Chaining** | Linear pass-by-pass transform | Clear stepwise dependencies | ~N× | +5–15% | Extract → Classify → Format |
| **Parallel Fan-Out** | Multiple independent → aggregate | Subjectivity, risk reduction | ~N× (par) | +5–10% | 3 summaries, pick best |
| **Staged Refinement** | Gen → Critique → Refine cycle | Creative tasks, content | ~3–5× | +10–25% | Draft → Critique → Rewrite |
| **Verify-Then-Generate** | Verify constraints before gen | High-stakes constrained outputs | ~2× | Prevents OOB errors | Financial report with reg checks |
| **Generate-Then-Verify** | Gen → Verify → Regenerate if fail | Verifiable outputs | ~2–3× | +15–30% | Code: gen → compile → fix |
| **Decomposition** | Break task → solve parts → compose | Complex multi-faceted problems | ~N× + comp | +10–20% | Strategy report decomposition |
| **Map-Reduce** | Process chunks parallel → combine | Long docs, large datasets | ~N× (par) | +15–25% | 100-page document summary |
| **Judge-LLM** | Separate LLM evaluates quality | Subjective quality control | ~2–5× | +5–15% | Content quality gate |
| **Chain of Thought** | Intermediate reasoning steps | Math, logic, multi-hop QA | ~1× | +10–20% | "Let's think step by step" |
| **Self-Consistency** | Multiple CoT samples, majority vote | Single-correct-answer problems | ~5× (par) | +5–15% | GSM8K with CoT-SC |
| **Tree of Thoughts** | BFS/DFS over reasoning paths | Hard puzzles, planning | ~10–100× | +20–40% | Game of 24, crossword |
| **Orchestrator-Worker** | Delegate to specialist LLMs | Multi-expertise projects | ~N× + OH | +10–25% | Research report with specialists |
| **Debate** | Multiple LLMs argue & converge | Ambiguous questions | ~5–15× | +5–15% | Ethical dilemma analysis |
| **Reflection** | Self-critique and self-improve | Code review, essay revision | ~2–4× | +10–20% | Self-Refine loop |
| **Critic** | Separate LLM evaluates output | High-stakes verification | ~2–3× | +5–15% | Cross-model fact-checking |
| **RAG Multi-Pass** | Retrieve → Gen → Verify → Refine | Hallucination-critical RAG | ~3–5× | +10–25% | Support with citations |
| **Validator** | Guard model checks safety/format | Prod safety requirements | ~2× | Prevents invalid outputs | Toxicity + format gate |
| **Tool-Use** | LLM calls tools, incorporates results | External data, computation | ~2–5× | Enables new capabilities | Code exec, web search |

---

## 14. References and Further Reading

### Foundational Papers

| Paper | Authors | Year | Contribution |
|---|---|---|---|
| Chain-of-Thought Prompting Elicits Reasoning in LLMs | Wei et al. | 2022 | CoT prompting |
| Self-Consistency Improves CoT Reasoning | Wang et al. | 2022 | Self-Consistency |
| Tree of Thoughts: Deliberate Problem Solving | Yao et al. | 2023 | Tree-of-Thoughts |
| Self-Refine: Iterative Refinement with Self-Feedback | Madaan et al. | 2023 | Self-Refine |
| Reflexion: Language Agents with Verbal RL | Shinn et al. | 2023 | Reflexion |
| Multiagent Debate Improves Factuality | Du et al. | 2023 | Multi-agent debate |
| DSPy: Compiling Declarative LLM Calls into Self-Improving Pipelines | Khattab et al. | 2023 | DSPy |

### Frameworks

| Tool | URL | Description |
|---|---|---|
| LangChain / LangGraph | https://github.com/langchain-ai/langgraph | Stateful multi-pass orchestration |
| DSPy | https://github.com/stanfordnlp/dspy | Declarative optimisable pipelines |
| LlamaIndex | https://github.com/run-llama/llama_index | RAG multi-step workflows |
| AutoGen | https://github.com/microsoft/autogen | Multi-agent conversations |
| CrewAI | https://github.com/joaomdmoura/crewAI | Role-based agent teams |
| Semantic Kernel | https://github.com/microsoft/semantic-kernel | Plugin-based chaining |

### Further Reading

- "Building LLM Applications for Production" — Chip Huyen (O'Reilly, 2024)
- "Patterns for Building LLM-Based Systems & Products" — Eugene Yan et al., 2024
- "LLM Verification Loops: Best Practices and Patterns" — Tim Williams, 2026
- "Revision or Re-Solving? Decomposing Second-Pass Gains in Multi-LLM Pipelines" — arXiv 2604.01029, 2026

---

> **Document Information**
>
> Part of the LLM/AI Engineering series by Jack Liu Shurui.
> Repository: https://github.com/jackliusr/research
> Path: `technology/multi_pass_llm_pipelines_guide.md`
> Topics: LLM pipelines, multi-agent systems, prompt engineering, production AI
