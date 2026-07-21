# Spec-Driven Development for LLMs

*A comprehensive guide to using formal specifications, benchmarks, evaluations, and validation to drive LLM application development in a test-first, quality-gated manner.*

---

## 1. What Is Spec-Driven Development for LLMs?

Spec-Driven Development (SDD) for LLMs adapts principles from Test-Driven Development (TDD) and API-first design to the unique challenges of building applications with large language models. **Specifications come first; implementation follows.**

### 1.1 The Core Idea

In traditional software engineering, teams write an OpenAPI spec before writing server code. The spec defines what the API must accept and return. SDD for LLMs applies the same principle: **write an executable specification for desired LLM behavior before building any prompts, retrieval pipelines, or agent logic.**

The specification becomes the single source of truth for:

- **Quality** — what "good" looks like, expressed as measurable criteria
- **Regression testing** — a baseline that changes must not break
- **Acceptance criteria** — the conditions under which a prompt/agent is "done"
- **Audit trail** — documented, version-controlled proof of intended behavior

### 1.2 The Shift: From Prompt Engineering to Specification Engineering

Traditional LLM development: write a prompt → test a few examples manually → tweak based on feel → ship → discover regressions in production → repeat.

SDD replaces this with a disciplined cycle:

1. **Define** — Write an executable specification for desired behavior
2. **Benchmark** — Build an evaluation dataset that tests spec compliance
3. **Implement** — Develop the prompt, RAG pipeline, or agent against the spec
4. **Evaluate** — Run the benchmark, measure spec compliance score
5. **Iterate** — Improve until compliance meets the threshold
6. **Gate** — Deploy only when spec compliance passes
7. **Monitor** — Continuously evaluate production samples against the spec

This represents a shift from **prompt engineering** (the art of crafting the perfect incantation) to **specification engineering** (the discipline of defining success in rigorous, testable terms).

### 1.3 What Makes LLM SDD Unique

| Challenge | Why It Matters |
|---|---|
| Non-determinism | Same prompt → different outputs; specs provide deterministic ground truth |
| No compile-time checks | Errors manifest at inference time, not development time |
| Emergent behavior | Changing one aspect can unexpectedly break others |
| Continuous model drift | The underlying model changes without notice |

SDD addresses these by providing a deterministic evaluation framework that catches regressions, validates quality, and provides measurable assurance.

---

## 2. Why Spec-Driven Development Matters for LLMs

### 2.1 Non-Deterministic Outputs Need Deterministic Ground Truth

LLMs are stochastic. JSON schema validation either passes or fails. A factual accuracy check either passes or fails. By anchoring evaluation to spec compliance, teams measure quality objectively despite underlying stochasticity.

### 2.2 Prompt Changes Break Behavior Unexpectedly

A seemingly minor prompt change can cascade into behavioral regressions: format breakage, refusal degradation, style drift, capability regression, or safety erosion. SDD catches these systematically — every prompt change runs against the full spec benchmark before deployment.

### 2.3 Regulatory Requirements Demand Verifiable Quality

Financial services, healthcare, and legal sectors increasingly require:
- **Auditability** — proof that the LLM system meets documented quality standards
- **Fairness** — outputs don't discriminate against protected groups
- **Explainability** — rationale for key decisions
- **Compliance** — adherence to regulatory frameworks (MAS FEAT, BCBS 239, EU AI Act)

Specifications provide the audit trail. Every evaluation run produces a compliance report that can be archived, reviewed, and presented to regulators.

### 2.4 Measurable Quality Thresholds

Replace subjective "does it look good?" with concrete thresholds:
- Format compliance: ≥98% must pass schema validation
- Factual accuracy: ≥92% on golden dataset
- Safety compliance: 0% toxicity, 0% PII leakage
- Edge case handling: ≥85% pass on adversarial examples
- Performance: p99 latency < 500ms, cost < $0.01/call

### 2.5 Enabling CI/CD

Traditional LLM development is hard to integrate into CI/CD because evaluation requires human review. SDD unlocks automated pipelines:

| Stage | SDD Integration |
|---|---|
| Pull Request | Run spec benchmark, block merge if compliance drops |
| Staging | Full suite + baseline comparison |
| Release | Compliance must exceed threshold |
| Production | Sample traffic, evaluate, monitor drift |

### 2.6 Reducing Manual Evaluation

Manual evaluation doesn't scale. SDD reduces it to: initial benchmark curation, edge case identification, periodic judge calibration, and spot-checking flagged samples. The bulk becomes automated.

---

## 3. Spec Types for LLMs

### 3.1 Output Format Specs

Define the **structure** of LLM outputs.

| Tool | Example | Evaluation |
|---|---|---|
| JSON Schema | `{"type":"object","properties":{"risk_score":{"type":"number","minimum":0}}}` | Schema validators |
| Pydantic | `class CreditDecision(BaseModel): approved: bool; limit: confloat(ge=0)` | Pydantic validation |
| Regex | `^\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}Z$` | `re.match()` |

```python
from pydantic import BaseModel, Field
from typing import Literal

class CreditDecision(BaseModel):
    application_id: str = Field(..., pattern=r"^APP-\d{8}$")
    decision: Literal["approved", "declined", "escalated"]
    credit_limit: float | None = Field(None, ge=0, le=1_000_000)
    rationale: str = Field(..., min_length=20, max_length=2000)
    regulations_applied: list[str] = Field(..., min_length=1)
    confidence_score: float = Field(..., ge=0, le=1)
```

**Best practice:** Define specs as Pydantic models first, then derive JSON Schema. This gives IDE support, type checking, and serialization in one step.

### 3.2 Content Quality Specs

Define the **substance** of outputs.

| Dimension | Example Spec | Evaluation |
|---|---|---|
| Factual Accuracy | ≥90% verifiable claims | LLM-as-judge + grounding |
| Relevance | Score ≥ 0.85 | BERTScore / embedding similarity |
| Completeness | All required sections present | Structured validation + checklist |
| Conciseness | Length within ±20% of target | Token count |
| Tone/Style | Professional register | LLM-as-judge classifier |

### 3.3 Safety Specs

Define what the LLM **must never do**.

| Dimension | Spec | Method |
|---|---|---|
| Toxicity | Score ≤ 0.01 | Detoxify / Perspective API |
| Bias | No demographic stereotyping | LLM-as-judge bias audit |
| PII Absence | Zero leakage | Presidio + regex |
| Content Policy | No harmful instructions | Policy classifier |
| Hallucination | Zero unsupported claims | Faithfulness check |

### 3.4 Behavioral Specs

Define **how the LLM should respond** to edge cases.

- Refusal patterns: polite, with explanation, max 150 characters
- Ambiguity handling: ask clarifying question (max 3 rounds)
- Escalation triggers: high-value transactions, customer distress, regulatory queries
- Error recovery: retry with explanation, escalate after 3 failures

### 3.5 Retrieval Specs (RAG)

| Dimension | Spec | Method |
|---|---|---|
| Citation Accuracy | ≥95% recall, ≥90% precision | Citation precision/recall |
| Context Relevance | map@k ≥ 0.85 | Embedding similarity |
| Faithfulness | Zero invented details | NLI entailment check |
| Retrieval Latency | p99 < 200ms | Timing instrumentation |

### 3.6 Performance Specs

- Latency SLA: p95 < 1s, p99 < 3s
- Token budget: ≤ 4096 per call
- Cost per call: < $0.01
- Success rate: ≥99.5%

### 3.7 Domain-Specific Specs

| Domain | Specs |
|---|---|
| Banking (MAS FEAT) | Fairness, Ethics, Accountability, Transparency checks |
| Banking (BCBS 239) | Data lineage, risk aggregation, audit trail |
| EU AI Act | Risk category, transparency, human oversight |
| Medical | HIPAA compliance, clinical accuracy |
| Legal | Jurisdiction citation, disclaimer format |

---

## 4. The Spec-Driven Development Workflow

### 4.1 Seven-Step Cycle

```
1. Define Spec → 2. Create Benchmark → 3. Implement Prompt/Agent
       ↑                                            ↓
7. Monitor ← 6. Gate ← 5. Iterate ← 4. Evaluate
```

**Step 1: Define Spec.** Write an executable specification (Pydantic, JSON Schema, YAML) defining input/output format, behavioral constraints, and quality thresholds.

**Step 2: Create Benchmark.** Build an evaluation dataset: golden examples (50-200+), edge cases (20%+ of total), adversarial examples, regression tests from historical failures.

**Step 3: Implement.** Develop the prompt or pipeline against the spec. The spec is source of truth; the prompt is implementation detail.

**Step 4: Evaluate.** Run the benchmark, produce per-example pass/fail results and aggregate compliance scores. Compare against baseline.

**Step 5: Iterate.** Focus on the lowest-scoring dimensions. One change at a time. Add failing cases to regression suite.

**Step 6: Gate.** Deploy only when: overall compliance ≥ threshold, no critical failures, no regressions in previously passing dimensions, safety spec at 100%.

**Step 7: Monitor.** Continuously evaluate production samples. Track compliance trends. Detect drift. Identify spec gaps from new failure patterns.

### 4.2 Spec-First vs Test-First vs Prompt-First

| Dimension | Spec-First (SDD) | Test-First | Prompt-First |
|---|---|---|---|
| Starting point | Executable spec | Test cases | Prompt intuition |
| Quality defined by | Domain experts + engineers | Engineers | Prompt engineers |
| Documentation | Formal, version-controlled | Test suite | README + comments |
| Regression detection | Full spec suite | Test scope | Manual |
| Audit trail | Versioned spec + eval reports | Test results | None |
| Best for | Regulated, quality-critical | Strong testing culture | Experimentation, prototyping |

---

## 5. Benchmarking for LLMs

### 5.1 Benchmark Types

**Golden Dataset:** Curated inputs with human-approved expected outputs. 50-200+ examples per capability. Representative of real usage. Versioned with the spec.

```jsonl
{"input": "What is the DTI cutoff for a home loan?",
 "expected": {"answer": "Maximum DTI is 43%.",
              "citations": ["Fannie Mae Guide B3-6-02"]},
 "tags": ["policy", "mortgage"]}
```

**Adversarial Benchmarks:** Inputs designed to test boundaries and safety guardrails. Jailbreak attempts, out-of-scope queries, ambiguous inputs, conflicting information. 20%+ of total benchmark.

**Property-Based Benchmarks:** Auto-generated test cases from spec constraints. Uses Hypothesis-style frameworks to explore combinatorial input space more thoroughly than hand-crafted examples.

```python
@given(age=st.integers(18, 120), income=st.floats(0, 1_000_000),
       credit_score=st.integers(300, 850))
def test_output_format(age, income, credit_score):
    output = llm_call(f"Age: {age}, Income: {income}, Score: {credit_score}")
    CreditDecision.model_validate_json(output)
```

**Regression Benchmarks:** Historical failures that must not regress. Every production bug becomes a spec test. Grows over time.

**Diversity Benchmarks:** Ensure coverage across income levels, loan types, customer tenure, credit histories, employment status, query sentiment.

### 5.2 Design Principles

- Represent real usage distribution (sample from production logs)
- Edge cases: 20%+ of benchmark
- Version-controlled with spec
- Deterministic evaluation where possible
- Human-validated golden set (≥1 domain expert per example)
- One benchmark per capability (isolated failures = targeted fixes)

### 5.3 Benchmark Size Guidelines

| Maturity | Golden | Edge Cases | Regression | Total |
|---|---|---|---|---|
| Getting Started | 50 | 10 | 0 | 60 |
| Baseline | 100-200 | 30-50 | 20-50 | 150-300 |
| Mature | 300-500 | 100-200 | 100-300 | 500-1000+ |

**Rule of thumb:** A benchmark is too small if removing any single example meaningfully changes the aggregate compliance score.

### 5.4 Benchmark Maintenance

| Activity | Cadence | Owner |
|---|---|---|
| Add production failure | Within 1 week | Quality engineer |
| Review golden examples | Quarterly | Domain experts |
| Add edge cases | Monthly | Red team / security |
| Check diversity | Quarterly | Quality engineer |

---

## 6. Evaluation Methodologies

### 6.1 LLM-as-Judge

Use a stronger LLM to evaluate spec compliance. Best for semantic quality, factual accuracy, open-ended outputs.

```python
JUDGE_PROMPT = """Evaluate if the response satisfies the specification.

Spec: {spec}
Query: {input}
Response: {output}

For each criterion: PASS or FAIL with reason.
Return JSON: {"overall": "PASS"|"FAIL",
  "criteria_results": {"criterion": {"status": "...", "reason": "..."}},
  "compliance_score": 0.0-1.0}"""
```

**Considerations:** Cost (use cheaper models), position bias (randomize order), verbosity bias (judges prefer longer outputs), self-enhancement bias (avoid same model family as evaluee). Calibrate against human judgments regularly.

### 6.2 Structured Evaluation

Parse output and validate deterministically against schema/properties. Fast, cheap, zero calibration. For: format validation, value range checks, enum validation, required fields.

```python
from pydantic import ValidationError

def evaluate_format(output: str, model_class: type[BaseModel]) -> dict:
    try:
        parsed = model_class.model_validate_json(output)
        return {"status": "PASS", "parsed": parsed}
    except ValidationError as e:
        return {"status": "FAIL", "errors": e.errors()}
```

### 6.3 Hybrid Evaluation

| Dimension | Method | Rationale |
|---|---|---|
| Format compliance | Structured | Deterministic |
| Factual accuracy | LLM-as-judge | Requires semantics |
| Relevance | LLM-as-judge | Semantic comparison |
| Toxicity | Classifier (Detoxify) | Fast, well-calibrated |
| PII detection | Presidio + regex | Deterministic |
| Completeness | Structured (checklist) | Objective |
| Safety policy | LLM-as-judge + classifier | Layered defense |

### 6.4 Human Evaluation

Necessary for judge calibration and subjective quality. Use 2+ reviewers per example. Measure inter-rater reliability (Cohen's kappa). With IRR < 0.7, refine criteria. Track human vs LLM judge disagreement patterns.

### 6.5 Automated Metrics

BLEU, ROUGE (text similarity), BERTScore (embedding similarity), Faithfulness (NLI entailment), Perplexity, Answer Relevance. These correlate imperfectly with human judgment — use as signals, not replacements.

### 6.6 Continuous Evaluation

| Stage | Scope | Frequency |
|---|---|---|
| Development | Subset | Every change |
| Pull Request | Full suite | Every PR |
| Staging | Full + baseline | Every deploy |
| Production | Sampled traffic | Hourly |
| Monthly | Full audit + calibration | Monthly |

---

## 7. Validation Strategies

### 7.1 Schema Validation

First line of defense. Pydantic/JSON Schema on structured outputs. Type checking, required fields, value constraints, enums, nested structure.

### 7.2 Semantic Validation

LLM-based validation of meaning and correctness. Checks output grounded in provided context, no contradictions, logical consistency across turns.

### 7.3 Constraint Validation

Business rule and regulatory validation. Example: cannot approve if DTI > 43%; high-value loans (>$500k) require specific disclosures; self-employed applicants need income verification.

### 7.4 Cross-Validation

Multiple LLM calls for consensus. Call 3+ times, check decision consistency. For high-stakes decisions (credit assessment, medical advice). Detects output instability (high variance = prompt not robust).

```python
def cross_validate(query, llm_fn, n=3, threshold=0.8):
    outputs = [llm_fn(query) for _ in range(n)]
    decisions = [parse(o).decision for o in outputs]
    consensus = max(set(decisions), key=decisions.count)
    return {"consensus": consensus,
            "ratio": decisions.count(consensus)/n,
            "stable": decisions.count(consensus)/n >= threshold}
```

### 7.5 Human-in-the-Loop

Escalate based on confidence, risk level, or sampling rules. Tiers:

| Tier | Criteria | SLA |
|---|---|---|
| Auto-approve | High confidence, low risk | Instant |
| Auto + audit | Medium confidence | 24h review |
| Human review | Low confidence, high risk | 1h |
| Emergency | Safety/regulatory violation | 15min |

### 7.6 Acceptance Test-Driven Development

Define acceptance criteria as validation rules before any implementation. The strongest form of SDD — tests written before any prompt exists, validating the spec's own correctness.

---

## 8. SDD for Different LLM Architectures

### 8.1 Prompts (Single LLM Call)

Spec defines prompt behavior, output structure, quality, safety. Evaluation tests prompt compliance across golden, edge, and adversarial sets. Regression suite catches prompt drift.

### 8.2 RAG Pipelines

Spec defines retrieval quality, citation accuracy, context utilization, faithfulness. Evaluate: retriever (recall, latency), generator (citation accuracy, hallucination rate), end-to-end (accuracy, latency).

**RAG evaluation dimensions:**
- Citation precision: of provided citations, how many are correct?
- Citation recall: of facts in output, how many are cited?
- Context relevance: are retrieved chunks relevant to the query?
- Faithfulness: does output contradict retrieved context? (NLI: premise=context, hypothesis=output)
- Completeness: does output cover all needed information?

### 8.3 Agents

Spec defines tool selection correctness, plan coherence, error recovery, multi-step reasoning quality. Evaluate trajectories: tool call sequence validity, recovery from failures, escalation behavior.

```python
def evaluate_agent(trajectory, tool_spec):
    actual_tools = [s["tool"] for s in trajectory if s["type"] == "tool_call"]
    tool_accuracy = sum(1 for t in actual_tools if t in tool_spec.allowed_tools) / len(actual_tools)

    missing = [t for t in tool_spec.required_tools if t not in actual_tools]
    recoveries = [s for s in trajectory if s.get("error") and s.get("retry_count",0) <= tool_spec.max_retries]

    return {"tool_accuracy": tool_accuracy, "missing_tools": missing,
            "recovery_rate": len(recoveries)/len([s for s in trajectory if s.get("error")]) if any(s.get("error") for s in trajectory) else 1.0}
```

### 8.4 Multi-Pass Pipelines

Spec defines each stage's output quality, handoff correctness, end-to-end performance. Evaluate stage-level schema compliance, handoff validity, stage interaction failures, data loss between stages.

---

## 9. Tools and Frameworks

### 9.1 DeepEval

Pytest-style LLM testing with 20+ built-in metrics. Faithfulness, relevancy, hallucination, bias, toxicity, G-Eval, BERTScore. Native CI/CD integration (GitHub Actions, Jenkins, GitLab CI).

```python
from deepeval.test_case import LLMTestCase
from deepeval.metrics import FaithfulnessMetric

def test_spec_compliance():
    test = LLMTestCase(input="...", actual_output=llm_call("..."),
                       retrieval_context=["policy.pdf"])
    faithfulness = FaithfulnessMetric(threshold=0.9)
    faithfulness.measure(test)
    assert faithfulness.score >= 0.9
```

### 9.2 LangSmith

Dataset management, evaluation runs, comparison views, human annotation, production monitoring. Good for managing evaluation datasets across prompt versions.

### 9.3 MLflow LLM Evaluation

Open-source experiment tracking with built-in and custom metrics. Model comparison, registry, spec compliance tracking across experiments.

### 9.4 RAGAS

RAG-specific evaluation: context_precision, context_recall, faithfulness, answer_relevancy, aspect_critic. Purpose-built for retrieval pipelines.

### 9.5 Pydantic

Schema validation for structured outputs. Foundation of format specs. Derive JSON Schema from models. Field validators for business rules.

### 9.6 JSON Schema

Industry-standard cross-language schema format. Interoperable.

### 9.7 Great Expectations

Data quality validation framework adaptable for LLM outputs. Column-level expectations, validation suites.

### 9.8 Custom Evaluation Harness

Combine pytest + DeepEval + Pydantic for full control:

```
credit-llm/
├── specs/credit_decision_spec.py       # Pydantic spec
├── benchmarks/golden.jsonl             # Golden examples
├── tests/test_spec_compliance.py       # pytest evaluations
└── ci/evaluate_pr.py                   # CI integration
```

### Tool Selection Matrix

| Tool | Best For | Format | Quality | Safety | CI/CD | Cost |
|---|---|---|---|---|---|---|
| DeepEval | General LLM testing | ✅ | ✅ | ✅ | ✅ | Free |
| LangSmith | Dataset management | ✅ | ✅ | ✅ | ✅ | Freemium |
| MLflow Eval | Experiment tracking | ✅ | ✅ | ✅ | ✅ | Free |
| RAGAS | RAG pipelines | ❌ | ✅ | ❌ | ❌ | Free |
| Pydantic | Schema validation | ✅ | ❌ | ❌ | ❌ | Free |
| Custom harness | Full control | ✅ | ✅ | ✅ | ✅ | Dev cost |

---

## 10. Spec-as-Code Best Practices

### 10.1 Write Specs Before Prompts

The spec is the source of truth; the prompt is implementation detail. Starting with a spec prevents endless "prompt gardening" without clear success criteria.

### 10.2 Version Spec with Application Code

Spec changes are code changes: committed to same repository, reviewed via PRs, subject to same CI/CD, tagged in release notes.

### 10.3 One Spec Per Capability

Decompose complex behavior into focused specs (format, accuracy, safety, compliance, performance). Benefits: isolated failures, clear ownership, targeted iteration, parallel development.

### 10.4 Deterministic Evaluation Preferred

Evaluation hierarchy:
1. Direct schema validation (Pydantic) — deterministic, instant, zero cost
2. Rule-based checks — deterministic, fast
3. Classifier-based (toxicity, PII) — mostly deterministic
4. LLM-as-judge — non-deterministic, slower, costs tokens
5. Human evaluation — slowest, most expensive

### 10.5 Spec Threshold Defines "Done"

A prompt is done when it meets spec thresholds — not when it "feels right":

```python
THRESHOLDS = {
    "format_compliance": 0.98,
    "factual_accuracy": 0.90,
    "safety_compliance": 1.0,
    "edge_case_handling": 0.85,
    "regression_suite": 1.0,
}
```

### 10.6 Benchmark Diversity

| Category | Proportion | Purpose |
|---|---|---|
| Golden (common) | 50% | Verify standard behavior |
| Golden (rare) | 10% | Long-tail coverage |
| Edge cases | 15% | Test boundaries |
| Adversarial | 10% | Safety and robustness |
| Regression | 10% | Prevent recurrence |
| Ambiguous inputs | 5% | Input validation |

### 10.7 Spec Evolution

Specs evolve as requirements improve. Document changes with date, author, type, rationale, and benchmark impact. Maintain a CHANGELOG alongside each spec.

### 10.8 Regression Suite

Every production bug becomes a spec test. Regression suite size must never decrease without documented, approved reasoning.

### 10.9 Spec Review

Reviewed by: domain experts (business requirements, edge cases), engineers (executability, feasibility), compliance (regulatory audit requirements).

### 10.10 Spec Documentation

Specs should be both human-readable and machine-executable. Markdown documentation alongside executable Pydantic models or YAML specs.

---

## 11. CI/CD Integration

### 11.1 PR Check

Run spec benchmark on every PR. Block merge if compliance drops below threshold. Post results as PR comment. Compare against baseline.

### 11.2 Regression Gate

Compare new vs baseline spec compliance. Critical regression (change > 5%) blocks the PR. Warning regression (change > 2%) flags for review.

### 11.3 Release Gate

Full evaluation suite must pass before release. Required thresholds enforced programmatically. Tag releases that pass gates.

### 11.4 Production Monitoring

Sample 5-10% of production traffic. Evaluate against spec continuously. Alert on compliance drops, safety violations, drift detection. Track trends per dimension.

```python
class SpecMonitor:
    def __init__(self, spec, sample_rate=0.05):
        self.spec = spec
        self.sample_rate = sample_rate
        self.results = defaultdict(list)

    def evaluate(self, input_text, output, metadata):
        if random.random() > self.sample_rate:
            return
        try:
            self.spec.model_validate_json(output)
            self.results["format_compliance"].append(1.0)
        except:
            self.results["format_compliance"].append(0.0)
            self._alert_violation(input_text, output)

    def get_trend(self, hours=24):
        return {dim: sum(s)/len(s) if s else 0
                for dim, s in self.results.items()}
```

### 11.5 Dashboard Components

- Overall compliance gauge (single score)
- Per-dimension breakdown (bar chart)
- Trend line (7/14/30 day)
- Regression alert list (real-time)
- Production compliance (hourly)
- Benchmark health (weekly)

### 11.6 Pipeline Flow

```
PR → Lint/Type Check → Spec Benchmark → Compare Baseline
                                            ↓
                                       PASS / FAIL (warn/block)
                                            ↓
                              Deploy to Staging → Staging Validation
                                                      ↓
                                                 PASS → Release
                                                 FAIL → Rollback
```

---

## 12. Spec-Driven Development Maturity Model

### Level 1: Ad-Hoc
No formal specs. Manual evaluation by prompt engineers. No benchmarks. Regressions caught in production. No audit trail.

### Level 2: Basic Specs
Output format specs (JSON Schema/Pydantic). Manual quality checks. Small golden dataset (50-100). Basic schema validation. Thresholds defined but not enforced.

### Level 3: Spec-Driven Development
SDD for critical paths. Automated CI evaluation (format + accuracy + safety). Golden dataset (200-500) with edge cases. PR gates guard against regression. Baseline comparison. Spec versioning.

### Level 4: Comprehensive
Full spec coverage across all capabilities. Adversarial benchmarks (red teaming). Regression suite (every bug = test). Gated releases. Production monitoring with dashboards. Spec review process.

### Level 5: Continuous Evolution
Automated test generation from spec constraints. Production drift detection. Self-healing: alerts trigger automated investigation. Spec evolution driven by failure patterns. Human eval reduced to calibration.

| Level | Investment | Regression Risk | Audit Readiness |
|---|---|---|---|
| 1 (Ad-Hoc) | Minimal | High | None |
| 2 (Basic) | Low-Medium | Medium | Low |
| 3 (SDD) | Medium | Low | Medium |
| 4 (Comprehensive) | Medium-High | Very Low | High |
| 5 (Continuous) | High | Near-zero | Full |

**Typical timeline:** 3-6 months to L3, 6-12 months to L4, 12-24 months to L5.

---

## 13. Banking Context for Spec-Driven Development

### 13.1 Regulatory Specs as Executable Specifications

**MAS FEAT (Fairness, Ethics, Accountability, Transparency):**

```python
class FEATSpec:
    fairness = [
        "output_must_not_discriminate_by_age",
        "output_must_not_discriminate_by_income",
        "output_must_not_discriminate_by_employment_type",
    ]
    ethics = [
        "output_must_not_recommend_unsuitable_products",
        "output_must_not_encourage_over_borrowing",
    ]
    accountability = [
        "output_must_include_decision_rationale",
        "output_must_reference_applicable_regulations",
    ]
    transparency = [
        "rationale_must_be_in_plain_language",
        "key_factors_must_be_explicitly_listed",
        "appeal_process_must_be_communicated_on_declines",
    ]
```

**BCBS 239 (Data Lineage):** Every fact traceable to source. Source documents versioned. Transformation steps documented. Aggregation auditable and reproducible.

**EU AI Act:** Risk category classification (minimal/limited/high/unacceptable). High-risk requires: risk management, data governance, technical documentation, transparency, human oversight, accuracy/robustness/cybersecurity standards.

### 13.2 Compliance Specs

- **PII absence:** Zero tolerance for NRIC, FIN, passport, phone, email, bank account, credit card, address, DOB
- **Data residency:** Processing only in allowed jurisdictions (SG, MY)
- **Audit trail:** Every LLM call logged with timestamp, user_id, input/output hash, model/prompt/spec version, latency, token count, compliance result
- **Retention:** 7 years (MAS requirement)

### 13.3 Risk Management Specs

Model risk tier determines requirements:
- **Tier 1** (credit decisions, trading): Independent validation, explainability, bias monitoring, human-in-the-loop, monthly evaluation
- **Tier 2** (customer service summarization): Independent validation, explainability, quarterly evaluation
- **Tier 3** (internal knowledge base): Self-review sufficient

### 13.4 Use Case: Credit Decision LLM

For a Singapore-based bank's credit decision LLM:

- **Format:** CreditDecision schema with required fields: decision, credit_limit, rationale, regulations_applied, confidence_score
- **MAS FEAT:** Fairness checks on age/income/employment; transparency: rationale in plain language, key factors listed, appeal communicated
- **Accuracy:** ≥92% decision accuracy on golden set
- **Fairness:** ≤15% approval rate disparity across demographic groups; no protected attributes in decision
- **Safety:** Zero toxicity, zero PII leakage, ≤2% hallucination rate
- **Performance:** ≤2s latency, ≤1000 tokens, ≤$0.005/call
- **Rationale requirements:** Must mention income, credit score, DTI ratio, employment status; must NOT mention race, religion, gender, marital status

### 13.5 Use Case: Customer Service LLM

- Escalate on: fraud concern, account takeover, financial difficulty, complaint, regulatory query, legal threat
- Complaint handling: logged, acknowledged, resolution timeline communicated, reference number provided
- Quality: CSAT ≥ 4.0/5, first-contact resolution ≥ 70%, transfer rate ≤ 30%
- Language: response in customer's language
- Regulatory queries: disclaimer required, offer written details

---

## 14. Challenges and Limitations

### 14.1 Spec Completeness

**Challenge:** Specs can't cover all possible LLM behaviors. The combinatorial input space is infinite.
**Mitigations:** Treat specs as bounded contracts. Use property-based testing to explore space automatically. Layer specs (format → quality → safety). Continuously add production failures.

### 14.2 Spec Maintenance

**Challenge:** Stale specs create false security — they pass while actual quality degrades.
**Mitigations:** Assign ownership. Quarterly review cycles. Track spec freshness (flag if unchanged > 6 months). Spec changes trigger same CI as code changes.

### 14.3 Eval Accuracy

**Challenge:** LLM-as-judge has noise, bias (position, verbosity, self-enhancement).
**Mitigations:** Calibrate judges against human judgments. Run each eval multiple times. Prefer structured eval. Track judge accuracy. Use ensemble judges for critical evals.

### 14.4 Over-Specification

**Challenge:** Too-rigid specs constrain handling of novel cases.
**Mitigations:** Distinguish "must have" from "nice to have." Use thresholds not binary passes. Flag novel cases. Warning signs: high compliance + declining user satisfaction, robotic outputs, rigid edge case responses.

### 14.5 False Confidence

**Challenge:** Passing spec ≠ real-world quality.
**Mitigations:** Monitor production metrics alongside spec compliance. Regularly audit spec coverage. Maintain "known gaps" document. Spec compliance is necessary but not sufficient.

### 14.6 Initial Investment

**Challenge:** Setting up spec framework requires upfront effort.
**Mitigations:** Start small (one spec, one benchmark, one critical path). Use existing tools. Quantify cost of not having specs. Show early wins by catching one regression.

### 14.7 Team Skills

**Challenge:** Requires shift from prompt engineering to specification engineering.
**Mitigations:** Train teams on specification thinking. Pair prompt engineers with SWEs. Start with simple format specs. Provide templates. Make spec writing collaborative with domain experts.

---

## 15. Getting-Started Roadmap

### Week 1: First Spec

- Identify highest-risk LLM capability (pilot target)
- Write output format spec (Pydantic model)
- Create minimal benchmark (50 examples, 10+ edge cases)
- Implement schema validation, run against current prompt
- **Deliverable:** Format spec + 50 examples + baseline compliance score

### Week 2: Quality Spec

- Define quality criteria (accuracy, relevance)
- Set up evaluation method (LLM-as-judge or structured)
- Integrate evaluation into CI (run on every PR)
- Establish quality baseline

### Week 3: Edge Cases + Thresholds

- Work with domain experts on 20-30 edge cases
- Add edge cases to benchmark, re-evaluate
- Define compliance thresholds (warning + blocking)
- Implement CI warning gate (non-blocking)

### Week 4: Safety + Regression Suite

- Add safety evaluation (toxicity, PII, prompt injection)
- Create regression tests from historical failures
- Implement blocking PR gate (compliance below threshold = blocked)

### Month 2: Scale

- Expand specs to all critical capabilities
- Build adversarial benchmarks (50+ examples via red teaming)
- Implement production monitoring (sample 5-10% traffic)
- Monthly evaluation cadence established

### Month 3: Optimize

- Add property-based testing (auto-generate from constraints)
- Formalize spec review process (domain expert + quality engineer + compliance)
- Review against maturity model, set L3-L4 targets

### Effort Estimation

| Phase | Hours | Team |
|---|---|---|
| Week 1-4 setup | 56-96 | 3-4 engineers |
| Per additional capability | 28-44 | 2-3 engineers |
| Monthly ongoing | 14-26 | 1-2 engineers |

### Quick-Start Checklist

```
Week 1: ☐ Pilot capability ☐ Format spec ☐ 50 benchmarks ☐ Schema validation ☐ Baseline
Week 2: ☐ Quality criteria ☐ Eval method ☐ CI integration ☐ Quality baseline
Week 3: ☐ 20-30 edge cases ☐ Thresholds ☐ CI warning gate
Week 4: ☐ Safety eval ☐ Regression suite ☐ Blocking PR gate
Month 2: ☐ All capabilities ☐ Adversarial benchmarks ☐ Production monitoring
Month 3: ☐ Property-based tests ☐ Spec review process ☐ L3-L4 maturity
```

---

## 16. Conclusion

Spec-Driven Development transforms LLM application development from fragile and subjective to robust and measurable. Key takeaways:

1. **Specs come first** — The spec is source of truth; the prompt is implementation detail.
2. **Quality is measurable** — Replace "does it look good?" with quantifiable, executable specifications.
3. **Regressions are preventable** — Every change runs against the full benchmark. Declining compliance catches regressions before production.
4. **Gates enable velocity** — Quality gates enable faster, more confident shipping by eliminating manual evaluation bottlenecks.
5. **Start small, grow systematically** — One format spec and 50 examples. Expand to quality, safety, and behavioral specs as the practice matures.
6. **Banking is a natural fit** — Regulatory requirements (MAS FEAT, BCBS 239, EU AI Act) translate naturally into executable specifications.
7. **It's a maturity journey** — Each level compounds quality assurance. L3 (3-6 months) is achievable and provides the most value per investment.

The shift from **prompt engineering** to **specification engineering** is the next evolution in building reliable, auditable, high-quality LLM applications. The best prompt is the one that satisfies the spec.

---

*Specification engineering is not about constraining LLMs — it's about defining what success looks like in terms that both humans and machines can agree on.*
