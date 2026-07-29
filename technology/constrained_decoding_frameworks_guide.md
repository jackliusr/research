# Constrained Decoding Frameworks

**Guaranteeing LLM Output Conformance to Schemas, Grammars, and Formats**

> A comprehensive technical guide to libraries, techniques, and serving engines that enforce
> structured output from large language models — covering logit masking, grammar-constrained
> generation, response-based validation, inference-engine integration, and production selection
> criteria for enterprise deployments.

---

## 1. What Is Constrained Decoding?

Constrained decoding is a technique that guarantees LLM outputs conform to a predefined
format by **masking invalid tokens during generation**. At each decoding step, the framework
computes a mask over the vocabulary that only permits tokens valid under the target
constraint (JSON schema, regex, context-free grammar), then samples only from the allowed
subset. This delivers **100% format compliance** — a fundamental capability for production
systems.

### How It Works

```
Tokens + constraint → compute mask → mask logits (set invalid to -inf) → sample → next token
```

| Technique | Compliance | Failure mode |
|-----------|-----------|--------------|
| Prompt engineering | ~60–90% | Invalid output silently |
| Post-processing + retry | Best-effort | May never converge; expensive loops |
| Fine-tuning | Not guaranteed | Costly; reduces flexibility |
| **Constrained decoding** | **100%** | Slower generation; no format failures |

Constrained decoding is the only technique providing a **mathematical guarantee** of format
compliance, because the mask operates on the raw token distribution before sampling.

---

## 2. Why Constrained Decoding Matters

**Production reliability.** Tool calling demands valid JSON arguments — one unclosed brace
crashes the chain. SQL generation must produce executable queries. Code generation must
compile.

**Latency.** Without constrained decoding, every generation requires parse + retry loops,
each adding seconds. Constrained decoding eliminates retries: output is valid on the first
pass.

**Safety.** Malformed output can leak internal prompt structure or system instructions.
Constrained decoding ensures output boundaries are always respected.

**Compliance (banking).** Regulatory reporting to MAS, HKMA, ECB requires adherence to
published schemas (XSD, JSON Schema). Trade confirmations and settlement instructions
(SWIFT MT/MX, FpML, ISO 20022) demand guaranteed formats. Self-hosted models with
constrained decoding keep sensitive data on-premises while producing compliant output.

---

## 3. The Core Mechanism: Logit Masking

During autoregressive generation at step *t*:

1. Model produces logit vector **ℓ** ∈ ℝ<sup>|V|</sup> over vocabulary *V*.
2. Mask function *M*(*s*, *c*) returns binary mask given parser state *s* and constraint *c*.
3. Apply mask: **ℓ′** = **ℓ** ⊙ *M*(*s*, *c*), masked positions set to -∞.
4. Sample: *x*<sub>t</sub> ~ softmax(**ℓ′**).
5. Advance parser state, repeat.

**Formally:** *V*<sub>valid</sub>(*p*, *C*) = { *v* ∈ *V* | ∃ *s* : *p* + *v* + *s* ∈ *L*(*C*) }

The naive check — testing every token (32K–128K) against the constraint per step — is
prohibitively expensive. All practical frameworks use precomputed structures (FSMs, PDAs,
cached masks) for O(1) or O(log |V|) per step.

---

## 4. Mask Computation Approaches

### Regex-Derived FSM

Compile regex → NFA (Thompson construction) → DFA → FSM. At step *t*, outgoing transitions
from current FSM state define valid next characters, then mapped to vocabulary tokens.

**Limitation:** Regular languages only — cannot model JSON's recursive nesting.

### Context-Free Grammar (CFG) Derived Automaton

Compile CFG/BNF → push-down automaton (PDA) or augmented parser. Partial parser (Earley,
LR(1), GLR) tracks stack state; a token is valid if appending it doesn't cause a parse
error. Handles nested structures, recursive patterns, cardinality constraints.

### JSON Schema to Automaton

JSON Schema → intermediate grammar → automaton. Supports: required/optional fields, nested
objects, arrays, enums, string patterns, numeric ranges, type constraints, `$ref`/`$defs`,
`allOf`/`anyOf`/`oneOf`, `if/then/else`. Output is not just syntactically valid JSON — it
is **semantically valid** against the schema (types, enums, ranges, patterns).

### Tool Calling Schema

OpenAI function definitions → JSON Schema → automaton. Handles required vs. optional args,
nested parameters, enums, array item schemas, `additionalProperties: false`. Foundation
of reliable LLM agent tool use.

---

## 5. The Speed–Quality Trade-off

```text
Prompt-based (~0% overhead, ~60–90% reliable)
    → Response-based (retry latency, best-effort)
        → Hybrid (~99%, selective mask)
            → Grammar-constrained (2–20% overhead, 100% reliable)
```

| Factor | Low overhead | High overhead |
|--------|-------------|---------------|
| Constraint complexity | Simple enum (3 values) | Deeply nested JSON with `$ref` chains |
| Vocabulary | 32K tokens | 128K+ tokens |
| Batch size | Single sequence | Large batch (mask per sequence) |
| Cache hit rate | Repeated schema, many steps | One-shot schema, short gen |

| Framework | Simple schema | Complex schema |
|-----------|--------------|----------------|
| Outlines | 2–5% | 5–15% |
| XGrammar | <1% | 2–5% |
| Guidance | 5–10% | 10–20% |
| llama.cpp GBNF | 1–3% | 3–8% |

**Hybrid approaches** (emerging): constrain only critical structural tokens (braces, commas)
while allowing free generation for content; or constrain the first few tokens then fall back
to prompt-based for the body. Trades ~99.5% reliability for <1% overhead.

---

## 6. Two Architectural Approaches

### Grammar-Constrained (Token-Level Masking)

Compiled automaton tracks valid next tokens per step; mask applied before sampling.
**Frameworks:** Outlines, XGrammar, llama.cpp GBNF, Guidance, LMQL.

| Strengths | Weaknesses |
|-----------|-----------|
| 100% guarantee | Requires logit access |
| Works with any logit-exposing model | Per-step latency overhead |
| Per-token enforcement | Complex inference engine integration |

### Response-Based (Post-Hoc Validation + Retry)

Generate freely → parse → validate → retry on failure. **Frameworks:** Instructor, Marvin,
LangChain validators.

| Strengths | Weaknesses |
|-----------|-----------|
| Works with any API (no logit access) | No guarantee (best-effort) |
| Easy integration | Retry latency: each failure = full gen cycle |
| Works with proprietary models | May never converge for complex schemas |
| Simple validation logic | Higher cost (pay for failed gens) |

---

## 7. Grammar-Constrained Frameworks

### 7.1 Outlines

**Repository:** [github.com/dottxt-ai/outlines](https://github.com/dottxt-ai/outlines)

Reference implementation for grammar-constrained generation. Compiles JSON Schema, regex,
CFG to FSM; interleaves with tokenizer for per-step token-level masking.

**Architecture:** Constraint → compile to FSM → map character transitions to vocabulary
token IDs → per-step: get FSM state, look up valid token IDs, mask logits, sample, advance.

**Supported constraints:** JSON Schema (Draft 2020-12: `$ref`, `$defs`, allOf/anyOf/oneOf,
if/then/else, numeric ranges, string patterns, enums), regex, CFG (BNF), Python type
annotations (Pydantic → JSON Schema), OpenAPI function calling.

**Integrations:** Hugging Face Transformers, vLLM, llama.cpp, custom backends via
`LogitsProcessor` interface.

| Strengths | Weaknesses |
|-----------|-----------|
| Most mature FSM library (since 2023) | FSM compilation adds 5–30s startup |
| Broadest constraint type support | FSM size grows with schema complexity |
| Strong documentation, active community | Integration depth varies by backend |
| Modular, extensible design | Per-step lookup grows with state count |

**Best for:** Teams needing reliable structured outputs from open-weight models; production
systems requiring JSON Schema compliance with mature library support.

```python
import outlines
from pydantic import BaseModel, constr

class TradeConfirmation(BaseModel):
    trade_id: constr(pattern=r"^T\d{10}$")
    counterparty: str
    notional: float
    currency: constr(min_length=3, max_length=3)

model = outlines.models.transformers("NousResearch/Hermes-3-Llama-3.1-8B")
generator = outlines.generate.json(model, TradeConfirmation)
result = generator("Generate a trade confirmation for a USD 5M swap.")
```

---

### 7.2 XGrammar

**Repository:** [github.com/xlang-ai/xgrammar](https://github.com/xlang-ai/xgrammar)
**Paper:** [arxiv.org/abs/2411.15100](https://arxiv.org/abs/2411.15100)

Efficient structured generation engine from CMU/Shanghai Jiao Tong. Uses CFG-based
approach with adaptive token mask caching and push-down automaton for nested structures.
**Over 80x speedup on H100 for Llama-3.1** compared to naive FSM approaches.

**Architecture:** CFG (from JSON Schema/BNF) → PDA construction → adaptive token mask
caching (keyed by state + context_hash) → context-aware mask reuse across sequences →
per-step: check cache → hit (90%+ for repeated structures): apply cached mask; miss:
compute, cache, apply.

**Key innovations:**
1. **Adaptive Token Mask Caching:** mask reuse across sequences with same schema —
   cache hit rate >90% for common JSON patterns, reducing overhead to near zero.
2. **Push-Down Automaton:** stack-based tracking of nesting depth — avoids FSM state
   explosion for deeply nested structures.
3. **C++ Core + Python Bindings:** hot path in C++; ergonomic Python API.

**Benchmarks (Llama-3.1-8B, H100):**

| Approach | Tokens/s | Overhead |
|----------|---------|----------|
| No constraint | 2,400 | — |
| XGrammar | 2,350 | ~2% |
| Naive FSM | 290 | ~88% |

**Strengths:** Extremely fast; efficient caching; production serving design; native vLLM
and SGLang integration; handles deeply nested schemas without FSM state explosion.

**Weaknesses:** Newer project, smaller community/docs; primarily JSON/text focused.

**Best for:** High-throughput production serving with structured outputs; vLLM or SGLang
users needing speed.

```python
from xgrammar import Grammar
grammar = Grammar.from_json_schema({
    "type": "object",
    "properties": {
        "trade_id": {"type": "string", "pattern": "^T\\d{10}$"},
        "status": {"type": "string", "enum": ["NEW", "AMENDED", "CANCELLED"]},
        "notional": {"type": "number"},
        "currency": {"type": "string", "minLength": 3, "maxLength": 3}
    },
    "required": ["trade_id", "status", "notional", "currency"]
})
# Used automatically with vLLM's guided decoding backend
```

---

### 7.3 Guidance

**Repository:** [github.com/guidance-ai/guidance](https://github.com/guidance-ai/guidance)

Token-level control through hand-crafted grammars using a Handlebars-like template DSL.
Stateless controllers integrate with model internals; supports generation, masking,
variable binding, tool calls, and multi-modal.

**Key features:** Template-based grammar DSL (`{{gen 'field'}}`, `{{select}}`, `{{#each}}`,
`{{#if}}`); **token healing** (backtracks one position to fix broken token boundaries —
unique to Guidance); role-based chat templates; variable binding.

| Strengths | Weaknesses |
|-----------|-----------|
| Token healing prevents tokenization bugs | Template DSL verbose for simple schemas |
| Readable, debuggable template DSL | Slower than FSM/PDA approaches |
| Excellent for conditional generation | Less suited for pure JSON Schema use |
| Multiple backend support | Requires learning template DSL |

**Best for:** Complex generation patterns needing fine-grained control; applications where
token healing matters (code generation); templates with conditional sections.

```python
import guidance
trade_gen = guidance(
    "{{#system~}}You are a trade generator.{{~/system}}\n"
    "{{#user~}}Generate a {{trade_type}} trade{{~/user}}\n"
    "{{#assistant~}}\n"
    '{"trade_id": "{{gen "trade_id" pattern="T[0-9]{10}"}}",\n'
    ' "type": "{{select "type" options=["FX_SPOT","FX_FWD","IR_SWAP"]}}",\n'
    ' "notional": {{gen "notional" pattern="[0-9]+(\\.[0-9]+)?"}}}\n'
    "{{~/assistant}}"
)
model = guidance.models.LlamaCpp("/path/to/model.gguf")
result = model + trade_gen(trade_type="FX Forward")
```

---

### 7.4 LMQL

**Repository:** [github.com/eth-sri/lmql](https://github.com/eth-sri/lmql)

Declarative, SQL-like query language for LLM generation constraints. Developed at ETH
Zurich. Integrates constraints into generation with logit masking and beam search.

**Architecture:** Declarative query → query compiler → constrained generation engine
(logit masking + beam search + confidence-based branching) → multi-model support.

| Strengths | Weaknesses |
|-----------|-----------|
| Unique declarative approach | Steep learning curve (custom language) |
| Supports complex constraints beyond JSON | Smaller community |
| Beam search + confidence branching | Less focused on JSON/structured output |
| Strong academic backing | Documentation less comprehensive |

**Best for:** Academic research on constrained generation; complex multi-constraint
scenarios beyond simple JSON schema.

```python
@lmql.query
def generate_trade():
    '''lmql
    argmax """
        {"trade_id": "[ID]", "type": "[TYPE]", "currency": "[CUR]"}
    """
    from "NousResearch/Hermes-3-Llama-3.1-8B"
    where
        len(TOKENS(ID)) == 10 and STOPS_AT(ID, '"') and
        TYPE in ["FX_SPOT", "FX_FWD", "IR_SWAP"]
    '''
```

---

### 7.5 llama.cpp GBNF (GGML BNF)

Grammar-based constrained decoding built into llama.cpp. BNF-like DSL (GBNF), compiled
to FSM, integrated into llama.cpp's sampler. Zero external dependencies.

**GBNF Example:**
```gbnf
root  ::= "{" ws trade_id ws "," ws type ws "," ws notional ws "}"
trade_id ::= "\"T" digit{10} "\""
type   ::= "\"FX_SPOT\"" | "\"FX_FWD\"" | "\"IR_SWAP\""
digit  ::= [0-9]
ws     ::= [ \t\n]*
```

| Strengths | Weaknesses |
|-----------|-----------|
| Zero external dependencies | GBNF is BNF subset (less expressive than CFG) |
| Compiled to native C++ (fast) | Only works with llama.cpp/GGML ecosystem |
| Works with any GGUF model | Less documentation for custom grammars |
| Excellent CPU/Apple Silicon performance | No PyTorch backend support |

**Best for:** llama.cpp users; offline/edge deployment on CPU/Apple Silicon.

---

## 8. Structured Generation via Inference Engines

### 8.1 vLLM Guided Decoding

**Documentation:** [docs.vllm.ai](https://docs.vllm.ai)

Built-in structured output support via guided decoding with XGrammar (default) or
Outlines backends. OpenAI-compatible `response_format` API.

**Supported formats:**
- `response_format: {"type": "json_object"}` — any valid JSON.
- `response_format: {"type": "json_schema", "json_schema": {...}}` — schema-constrained JSON.
- `response_format: {"type": "regex", "regex": "..."}` — regex-constrained text.
- `response_format: {"type": "grammar", "grammar": "..."}` — CFG-constrained text.

| Strengths | Weaknesses |
|-----------|-----------|
| Production-grade serving | Requires running vLLM (GPU server) |
| XGrammar backend for high throughput | Integration depth varies by backend |
| OpenAI-compatible API (drop-in replacement) | Some JSON Schema features differ across backends |
| Backward compatible; active dev | |

**Best for:** Production deployments needing high-throughput structured output; teams
already on vLLM.

### 8.2 SGLang Structured Generation

**Repository:** [github.com/sgl-project/sglang](https://github.com/sgl-project/sglang)

Inference engine designed for agent/structured output workloads. Native XGrammar
integration, first-class structured generation, **RadixAttention** prefix caching.

**Key features:** Constrained decoding with XGrammar; JSON Schema, regex, function
calling support; Python-based frontend (SGLang program); RadixAttention for shared
prefix KV cache reuse (40–60% reuse for repeated schemas).

| Metric | vLLM | SGLang |
|--------|------|--------|
| JSON Schema throughput | ~1,200 req/s | ~1,800 req/s |
| Tool-calling p50 latency | 45ms | 32ms |

| Strengths | Weaknesses |
|-----------|-----------|
| Fastest structured gen (native XGrammar) | Newer; smaller track record |
| RadixAttention for repeated schemas | Fewer integrations than vLLM |
| Designed for agent/tool-calling workloads | Docs improving but not yet vLLM parity |
| Growing adoption in agent ecosystem | |

**Best for:** Maximum throughput structured output serving; agent-heavy workloads with
shared schema prefixes.

---

## 9. Managed API Structured Outputs

Major providers now offer server-side constrained decoding — no client-side library needed.

### OpenAI

`response_format` with `strict: true`. Server-side constrained decoding guarantees JSON
Schema compliance.

```python
response = client.chat.completions.create(
    model="gpt-4o",
    response_format={
        "type": "json_schema",
        "json_schema": {
            "name": "trade",
            "strict": True,
            "schema": {
                "type": "object",
                "properties": {
                    "trade_id": {"type": "string", "pattern": "^T\\d{10}$"},
                    "type": {"type": "string", "enum": ["FX_SPOT", "FX_FWD"]},
                    "notional": {"type": "number"},
                    "currency": {"type": "string", "minLength": 3, "maxLength": 3}
                },
                "required": ["trade_id", "type", "notional", "currency"],
                "additionalProperties": False
            }
        }
    }
)
```

**Availability:** GPT-4o, GPT-4o-mini, o1, o3.

### Google Gemini

`response_mime_type: "application/json"` + `response_schema`:

```python
genai.GenerativeModel("gemini-2.0-flash", generation_config={
    "response_mime_type": "application/json",
    "response_schema": {
        "type": "OBJECT",
        "properties": {"trade_id": {"type": "STRING"}, ...},
        "required": ["trade_id", ...]
    }
})
```

### Anthropic Claude

Tool use for structured output — repurpose `tool_choice: {"type": "tool", "name": "..."}`
with the desired schema as the tool's `input_schema`. Extended JSON mode with schema
in system prompt is a lighter alternative.

### Other Providers

| Provider | Mechanism |
|----------|-----------|
| Mistral | `response_format: {"type": "json_object"}`, function calling |
| Perplexity | `response_format` parameter |
| Grok (xAI) | Function calling |
| Together AI | `response_format` with JSON schema |
| Fireworks | `response_format` with JSON schema |

### When to Use Provider Structured Outputs

**Advantages:** No client infrastructure; vendor-maintained; zero client latency overhead;
access to best proprietary models.

**Disadvantages:** Vendor lock-in; less mask control; may not support all JSON Schema
features (`$ref`, `if/then/else`); higher cost at volume; data leaves your infrastructure.

---

## 10. Response-Based / Validation-Based Frameworks

### 10.1 Instructor

**Repository:** [github.com/instructor-ai/instructor](https://github.com/instructor-ai/instructor)

Wraps any LLM client with Pydantic validation and automatic retry. Sends schema as prompt,
parses response, validates, retries on failure with error feedback.

**Architecture:** Pydantic model → prompt with schema instructions → LLM → parse →
validate → success (return model) or failure (retry with error message).

**Features:** Multi-language (Python, TypeScript, Go, Ruby, Elixir); works with any LLM API;
streaming support; async support; rich validation (field validators, custom validators);
iterable support (extract lists of objects).

| Strengths | Weaknesses |
|-----------|-----------|
| API-agnostic (any provider) | No guarantee (best-effort) |
| Best DX (Pydantic → structured output) | Retry latency for complex schemas |
| Multi-language SDKs | May never converge for complex schemas |
| Active community | Higher cost (failed generations) |

**Best for:** Teams using managed API providers; prototyping; simple to moderately
complex schemas.

```python
from pydantic import BaseModel, Field
from instructor import from_openai
from openai import OpenAI

class TradeConfirmation(BaseModel):
    trade_id: str = Field(pattern=r"^T\d{10}$")
    type: str
    notional: float
    currency: str = Field(min_length=3, max_length=3)

client = from_openai(OpenAI())
trade = client.chat.completions.create(
    model="gpt-4o",
    response_model=TradeConfirmation,
    messages=[{"role": "user", "content": "Generate a USD 10M FX swap"}]
)
```

---

### 10.2 DSPy

**Repository:** [github.com/stanfordnlp/dspy](https://github.com/stanfordnlp/dspy)

Programmatic LLM programming framework (Stanford NLP) with structured output via typed
signatures. Uses compile-time optimization to improve prompt quality for structured
output.

**Key features:** Typed signatures (`input_field: type → output_field: type`); JSON output
via type annotations; DSPy optimizer (auto-prompt improvement via few-shot selection,
instruction tuning).

| Strengths | Weaknesses |
|-----------|-----------|
| Part of larger DSPy ecosystem | Not a dedicated structured output library |
| Compiler optimizes prompts | Less control over JSON schema details |
| Good for multi-step pipelines | Learning curve for DSPy |

**Best for:** Teams using DSPy; applications needing prompt optimization for structured
output in complex pipelines.

```python
class GenerateTrade(dspy.Signature):
    """Generate a structured trade confirmation."""
    request = dspy.InputField()
    trade = dspy.OutputField(desc="JSON with trade_id, type, notional, currency")
```

---

### 10.3 Marvin

**Repository:** [github.com/PrefectHQ/marvin](https://github.com/PrefectHQ/marvin)

Python library for structured data extraction via type hints and Pydantic validation.
Focused on extraction from unstructured text.

| Strengths | Weaknesses |
|-----------|-----------|
| Simple, clean API | Less flexible for generation use cases |
| Good for extraction | Smaller community |
| Minimal boilerplate | Less active development |

**Best for:** Simple data extraction from text; teams in Prefect ecosystem.

```python
from marvin import extract
from pydantic import BaseModel
class Trade(BaseModel):
    trade_id: str; type: str; notional: float; currency: str
trades = extract(text, target=Trade, model="gpt-4o")
```

---

## 11. Comparison Table

| Feature | Outlines | XGrammar | Guidance | LMQL | llama.cpp GBNF | Instructor | DSPy | Marvin |
|---------|----------|----------|----------|------|----------------|------------|------|--------|
| **Approach** | Grammar-constrained | Grammar-constrained | Grammar-constrained | Grammar-constrained | Grammar-constrained | Response-based | Response-based | Response-based |
| **Constraint types** | JSON Schema, Regex, CFG, Pydantic, OpenAPI | JSON Schema, Regex, CFG | Template DSL, Regex, select | Custom query lang, Regex | GBNF (BNF subset) | Pydantic, Zod | Typed signatures | Type hints |
| **API integration** | Transformers, vLLM, llama.cpp | vLLM, SGLang, Python | Transformers, llama.cpp, vLLM | Transformers, OpenAI, distributed | llama.cpp only | Any LLM API | Any LLM (DSPy) | OpenAI, any API |
| **Latency overhead** | 2–15% | <1–5% | 5–20% | Varies | 1–8% | Retry-dependent | Retry-dependent | Prompt-dependent |
| **Guarantee** | 100% | 100% | 100% | 100% | 100% | Best-effort | Best-effort | Best-effort |
| **Learning curve** | Moderate | Moderate | Moderate | Steep | Moderate | Low | Moderate-High | Low |
| **Token healing** | No | No | Yes | No | No | No | No | No |
| **Best for** | Reliable structured output | High-throughput production | Complex templates | Academic research | Edge/CPU deployment | Any API, flexibility | Multi-step LLM | Simple extraction |

---

## 12. Selection Guide

### Scenario 1: Open-Weight Model, Production, High Throughput

**→ XGrammar + vLLM or SGLang**

Fastest constrained decoding (<2% overhead). Native XGrammar integration, continuous
batching, PagedAttention, OpenAI-compatible API. SGLang adds RadixAttention for shared
schema prefix reuse. Requires GPU memory for model + KV cache.

### Scenario 2: Open-Weight Model, Quick Prototyping

**→ Outlines**

Simplest FSM-based library. Works out of the box with Transformers, covers all common
constraint types (JSON Schema, regex, CFG, Pydantic). Accept 5–30s startup cost for
FSM compilation; migrate to XGrammar+vLLM for production.

### Scenario 3: Edge / CPU Deployment

**→ llama.cpp + GBNF**

Zero external dependencies. Works on CPU, Apple Silicon, Raspberry Pi. GGUF model format
required. GBNF DSL is a BNF subset — adequate for JSON and most structured formats.

### Scenario 4: Managed API (OpenAI / Gemini)

**→ Provider's built-in structured output**

Use `response_format` with `strict: true` (OpenAI) or `response_schema` (Gemini). Zero
client overhead, guaranteed compliance, vendor-maintained. Accept vendor lock-in and
data leaving infrastructure.

### Scenario 5: Any API, Maximum Flexibility

**→ Instructor + Pydantic**

Define Pydantic models once, switch providers with config change. Works with OpenAI,
Anthropic, Gemini, local models. Best-effort only — monitor retry rates in production.

### Scenario 6: Complex Generation Templates

**→ Guidance**

Token-level control with conditional sections, variable binding, token healing. Best
when generation requires fine-grained control beyond simple schema compliance.

### Scenario 7: Academic Research on Constraints

**→ LMQL**

Declarative query language supports complex multi-constraint scenarios. Beam search
and confidence branching for exploration. Accept steeper learning curve.

### Scenario 8: Enterprise Banking

**→ Multi-strategy approach**

Banking demands regulatory compliance, data sovereignty, and auditability — no single
approach suffices.

| Layer | Approach | Rationale |
|-------|----------|-----------|
| **API-based models** | Provider structured output (OpenAI `response_format strict`, Google `response_schema`, Anthropic `tool_use`) | Simplest operation; vendor-maintained compliance |
| **Self-hosted models** | Outlines/XGrammar + vLLM | Full control; data sovereignty for sensitive data |
| **OpenAI models** | `response_format` with `strict: true` | Most reliable API option; guaranteed JSON Schema |
| **Anthropic Claude** | `tool_use` mechanism | Anthropic's supported structured output pathway |
| **Fallback** | Instructor + Pydantic | For providers without native structured output |

**Banking-specific considerations:**
- **Audit logging:** Log every generation request + output + schema + model version +
  framework version.
- **Schema versioning:** JSON Schemas must be version-controlled and governance-approved.
- **Monitoring:** Track constraint violation rates, retry rates, generation latency;
  alert on anomalies.
- **Testing:** Validate against representative schemas before production deployment.
- **Disaster recovery:** Fallback plan for framework bugs or incompatibilities.

---

## 13. Decision Flowchart

```
Need 100% guaranteed compliance?
├── No → Using managed API?
│        ├── Yes → Provider structured output (simplest)
│        └── No  → Instructor + Pydantic (most flexible)
└── Yes → Have logit access?
          ├── No → Accept best-effort?
          │       ├── Yes → Instructor + Pydantic
          │       └── No  → Switch to open-weight model with logit access
          └── Yes → Deployment target?
                    ├── Production, high throughput  → XGrammar + vLLM / SGLang
                    ├── Quick prototyping            → Outlines
                    ├── Edge / CPU / Apple Silicon   → llama.cpp + GBNF
                    ├── Complex templates            → Guidance
                    └── Academic research            → LMQL
```

---

## 14. Summary

Constrained decoding has matured rapidly:

- **2023:** Outlines pioneered FSM-based constrained decoding for open-weight models.
- **2024:** XGrammar introduced efficient PDA-based generation with mask caching
  (80x speedup). vLLM and SGLang added native structured output. OpenAI and Google
  launched server-side structured outputs.
- **2025:** Constrained decoding is a standard feature across major inference engines
  and API providers. The question is not _whether_ to use it, but _which_ framework
  fits your deployment.

For banking and regulated industries, constrained decoding is not optional — it is a
required capability for any LLM application producing structured output. The multi-strategy
approach (provider structured output for API models, XGrammar/Outlines for self-hosted,
Instructor as fallback) provides the robustness, compliance, and flexibility that
enterprise deployments demand.

---

> **Author:** Jack Liu Shurui — Solution Architect, Crédit Agricole CIB
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Path:** `technology/constrained_decoding_frameworks_guide.md`
> **Part of:** LLM/AI Infrastructure Series (fine-tuning frameworks, LLM optimization,
> enterprise AI gateway guides)
