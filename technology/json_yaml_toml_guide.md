# JSON, YAML and TOML — The Configured Data

**Generated:** 2026-09-13
**Scope:** A format-level deep-dive on the three formats that carry almost all machine-readable configuration and service-to-service interchange on a modern estate: JSON (RFC 8259 / STD 90, ECMA-404), YAML 1.2.2, and TOML 1.0.0. This guide covers their grammars and type systems, the parser and tooling landscape per language, the pitfalls that bite in production (verified against the specifications), schemas and validation, the security of deserialisation, selection guidance, the AI/LLM angle, and a worked configuration-standard decision for the Cymbal Bank platform estate (explicitly illustrative). Facts marked **(verified)** were confirmed against the primary specifications in September 2026; **[verify]** marks a claim I could not confirm at a primary source; widely repeated claims that the specs do not support are labelled **folklore**.

**Related guides in this series:** [Configuration Management Languages](configuration_management_languages_guide.md) (the tooling layer above these formats) · [Schema Evolution and Data Drift](schema_evolution_data_drift_guide.md) · [API Governance](api_governance_guide.md) · [Power Platform CI/CD](power_platform_cicd_guide.md) · [Charmed Kubernetes vs OpenShift](charmed_kubernetes_vs_openshift_guide.md) · [OpenShift SCC Comprehensive Guide](openshift_scc_comprehensive_guide.md) · [Data Pipeline Versioning](data/data_pipeline_versioning.md) · [Agent Scaffolding](ai_llm/agent_scaffolding_guide.md) · [Agentic Solution Artifacts](ai_llm/agentic_solution_artifacts_guide.md)

---

## Table of Contents

1. [The Three Formats and Their Standards](#1-the-three-formats-and-their-standards)
2. [Grammars and Type Systems](#2-grammars-and-type-systems)
3. [The Pitfalls — Spec Rules vs Folklore](#3-the-pitfalls--spec-rules-vs-folklore)
4. [Parsers and Tooling, Per Language and Ecosystem](#4-parsers-and-tooling-per-language-and-ecosystem)
5. [Schemas and Validation](#5-schemas-and-validation)
6. [Security: Deserialisation, Aliases and Secrets](#6-security-deserialisation-aliases-and-secrets)
7. [Selection Guidance and a Decision Framework](#7-selection-guidance-and-a-decision-framework)
8. [The AI/LLM Angle](#8-the-ai--llm-angle)
9. [Cymbal Bank Worked Example — A Configuration Standard for the Platform Estate](#9-cymbal-bank-worked-example--a-configuration-standard-for-the-platform-estate)
10. [Evolution, Conversion and Standards Governance](#10-evolution-conversion-and-standards-governance)
11. [Claims Audit](#11-claims-audit)
12. [What Could Not Be Verified](#12-what-could-not-be-verified)
13. [Glossary](#13-glossary)
14. [Primary Sources and Further Reading](#14-primary-sources-and-further-reading)

---

## 1. The Three Formats and Their Standards

**Boundary with the tooling layer.** This guide is about *formats*: the grammar, the type system, the parser behaviour, the pitfalls, and the validation and security properties of JSON, YAML and TOML themselves. It is deliberately **not** about the configuration-management tools and languages that consume them — Ansible and its YAML substrate, Puppet, Chef, Terraform and HCL, Pulumi, CloudFormation, or the typed-config generation (CUE, Jsonnet, Dhall, Pkl, Nix). That is the subject of [configuration_management_languages_guide.md](configuration_management_languages_guide.md). Where this guide needs a tool to make a point about YAML-in-Kubernetes or YAML-in-CI, it cross-references rather than re-derives. The rule of thumb: **the CM guide answers "which language should I express my infrastructure in?", this guide answers "what will the parser actually do with the bytes I wrote?"**

### 1.1 JSON — from a JavaScript subset to an Internet Standard

JSON was specified informally by Douglas Crockford at json.org circa 2001 as a subset of JavaScript object literal syntax **(verified)** — the canonical json.org grammar page presents JSON as a small set of railroad diagrams over `object`, `array`, `string`, `number`, and the three literal names `true`, `false`, `null`.

Its formal standardisation came in two tracks that now reinforce each other:

- **ECMA-404, *The JSON Data Interchange Syntax*.** I read the cached copy of the standard: it is the **2nd Edition, December 2017** **(verified)**, published by Ecma International, and it defines the grammar independently of any programming language.
- **RFC 8259, *The JavaScript Object Notation (JSON) Data Interchange Format*.** Dated **December 2017**, editor **T. Bray (Textuality)**, **Standards Track**, and it **obsoletes RFC 7159** **(verified)**. It is an **Internet Standard**, published as **STD 90** — the RFC Editor's STD 90 page lists it as `RFC 8259 : STD 90 : ... Internet Standard` **(verified)**. It registers the **`application/json`** media type **(verified)**.

RFC 8259 §1.3 is worth quoting in spirit because it explains why the document exists at all: in the years since RFC 4627, JSON usage revealed "certain patterns that, while allowed by its specifications, have caused interoperability problems", and the revision's goal was to apply errata, "remove inconsistencies with other specifications of JSON, and highlight practices that can lead to interoperability problems" **(verified)**. Much of §3 below is exactly that list of highlighted practices.

One structural fact from RFC 8259 §2 shapes everything else: a JSON text is a sequence of tokens — "six structural characters, strings, numbers, and three literal names" — and a JSON text is *a serialized value*, not necessarily an object or array **(verified)**. Earlier specifications constrained the top level to object or array; RFC 8259 does not.

### 1.2 YAML — the human-first data serialisation language

YAML's governing document for new work is the **YAML 1.2.2 specification**, the revision dated **2021-10-01**, produced by the YAML Language Development Team, with **no normative changes from YAML 1.2** **(verified)**. The specification is organised around a *representation graph* of nodes, *tags* that give nodes types, and *schemas* that map plain scalars to tags. Chapters 10.1 (Failsafe), 10.2 (JSON) and 10.3 (Core) define the recommended schemas; §10.3 states the Core schema "is the recommended default schema that YAML processor should use unless instructed otherwise" **(verified)**.

For a long time YAML had no registered media type at all. That gap closed with **RFC 9512, *YAML Media Type***, **February 2024**, category **Informational**, by R. Polli (DTD, Italian Government), E. Wilde (Axway) and E. Aro **(verified)**. It registers **`application/yaml`** and the **`+yaml` structured syntax suffix** with IANA **(verified)**. The RFC itself notes the motivation: YAML "is widely used on the Internet, including in the API sector (e.g., see [OAS]), but a corresponding media type and structured syntax suffix had not previously been registered by IANA" **(verified)** — a striking case of a format becoming ubiquitous through de-facto adoption long before its paperwork caught up. The deprecated aliases (**`application/x-yaml`**, **`text/yaml`**) date from that unregistered era **(verified)**.

YAML 1.1 remains relevant because a large share of installed parsers still implement it or a hybrid. The 1.1 specification is where the boolean and integer resolution surprises of §3 originate. The 1.1 document states plainly in §3.2.3.2 that "the boolean 'true' might also be written as 'yes'", and its Example 2.19 shows `sexagesimal: 3:25:45` and `octal: 014` as integers **(verified)** — both are gone or restricted in 1.2.

### 1.3 TOML — Tom's Obvious, Minimal Language

TOML is specified by **TOML v1.0.0**, published **2021-01-11**, authored by **Tom Preston-Werner, Pradyun Gedam, et al.** **(verified)**. Its stated objective, in the spec's own words, is "a minimal configuration file format that's easy to read due to obvious semantics" that is "designed to map unambiguously to a hash table" and "should be easy to parse into data structures in a wide variety of languages" **(verified)**. The specification defines the **`.toml`** extension and the MIME type **`application/toml`** **(verified)**.

The **TOML v1.1.0** page is dated **2025-12-18** and is served as the current version on toml.io **(verified)**. Its *status* — whether it is a settled, community-ratified release or a published revision still under discussion — was **not** stated at the page I read; I therefore treat v1.1.0 as **published but not confirmed as settled**, and flag the question in §12 rather than calling it a done standard. For any normative statement in this guide I lean on **v1.0.0**, which is what the installed parser ecosystem actually implements.

### 1.4 The standards at a glance

| | JSON | YAML | TOML |
|---|---|---|---|
| Primary specification | RFC 8259 (Dec 2017), editor T. Bray; ECMA-404 2nd Ed. (Dec 2017) | YAML 1.2.2 (2021-10-01), YAML Language Development Team | TOML v1.0.0 (2021-01-11) |
| Standardising body | IETF (RFC 8259) + Ecma International (ECMA-404) | Community spec plus IETF media-type RFC | Community spec, toml.io |
| Status of the spec | Internet Standard, published as STD 90 **(verified)** | Spec revision; no standards body ratification beyond RFC 9512 for the media type **(verified)** | Published specification; formal status not asserted at the page **(verify)** |
| Media type | `application/json` **(verified)** | `application/yaml`, suffix `+yaml`, RFC 9512 Informational **(verified)** | `application/toml` **(verified)** |
| File extension | `.json` | `.yaml` / `.yml` (`.yml` is convention, not from the RFC) | `.toml` **(verified)** |
| Origin | json.org, Douglas Crockford, circa 2001 **(verified)** | Community work, 2001 onward | Tom Preston-Werner, explicit in the name |
| Comments | Not permitted anywhere | `#` | `#` **(verified, in practice via the comment section of the spec)** |

---

## 2. Grammars and Type Systems

### 2.1 JSON's grammar and its four-plus-two type model

RFC 8259 §2–§7 gives the whole grammar in a handful of productions: `value` is one of an object, array, number, string, or the literal names `false`, `null`, `true` **(verified)**. RFC 8259 §3 is even more explicit: a JSON value "MUST be an object, array, number, or string, or one of the following three literal names" **(verified)**.

The type model therefore has **four primitive types (strings, numbers, booleans, null) and two structured types (objects, arrays)** **(verified, RFC 8259 §1)**. Two consequences fall out of how small that model is:

- **There is exactly one number type.** RFC 8259 §6 defines a base-10 representation with an optional leading minus, optional fraction and/or exponent, and **prohibits leading zeros** — `int = zero / ( digit1-9 *DIGIT )` **(verified)**. The spec does not say whether the implementer's number is an integer, a float, or a decimal; it leaves range and precision to the implementation.
- **There is no date, duration, binary, or decimal type.** Anything richer is a string plus a convention — which is precisely why JSON Schema grows so many `format` keywords (§5).

JSON is also, notably, *silent* on several things a configuration author might expect it to pin down: object member ordering (the spec only says libraries differ in whether they expose ordering to callers **(verified)**), duplicate names (see §3.4), character encoding beyond UTF-8 for open ecosystems, and any comment syntax.

### 2.2 YAML's grammar — presentation, nodes, tags, schemas

YAML separates **presentation** (indentation, flow vs block style, quoting, comments) from the **representation graph** (nodes and their tags). The spec is explicit that style is decoration: scalar *format* such as writing a boolean as `yes` "is a presentation detail and is not reflected in the serialization tree and representation graph", and comments likewise "are a presentation detail and must not have any effect" and "are not associated with a particular node" **(verified, YAML 1.1 §3.2.3.2–3.2.3.3)**.

Types come from **tags**, and tags come from **schemas**. The three recommended schemas:

| Schema | Types it resolves | When to use |
|---|---|---|
| **Failsafe** (§10.1) | `map`, `seq`, `str` only — every scalar is a string | Generic YAML processors; guaranteed to work with any document **(verified)** |
| **JSON** (§10.2) | JSON's types — the "lowest common denominator of most modern computer languages" **(verified)** | When a YAML document must round-trip through JSON |
| **Core** (§10.3) | JSON plus human-readable forms (`null`/`~`, `true`/`True`/`TRUE`, integers, floats, etc.) — "the recommended default schema that YAML processor should use unless instructed otherwise" **(verified)** | Everyday application config |

The Core schema's tag-resolution table is the single most useful thing to memorise from the YAML spec:

| Plain scalar pattern | Resolved tag |
|---|---|
| `null`, `Null`, `NULL`, `~`, or empty | `tag:yaml.org,2002:null` |
| `true`, `True`, `TRUE`, `false`, `False`, `FALSE` | `tag:yaml.org,2002:bool` |
| `[-+]?[0-9]+` | `tag:yaml.org,2002:int` (base 10) |
| `0o[0-7]+` | `tag:yaml.org,2002:int` (base 8) |
| anything else unmatched on a plain scalar | `tag:yaml.org,2002:str` **(verified)** |

That last row is the safety net most people do not know exists: under the Core schema, an *unrecognised* plain scalar is a **string**, not an error and not a guess.

### 2.3 TOML's type system — a hash table with an opinion

TOML's framing predicate is that it "maps unambiguously to a hash table" **(verified)**. Its rules of the road (from the spec's preamble) are terse and worth listing because they are where most parser-difference pain comes from: a TOML file must be a **valid UTF-8 encoded Unicode document**; **whitespace means tab (0x09) or space (0x20)**; **newline means LF (0x0A) or CRLF (0x0D 0x0A)** **(verified)**.

TOML keys are **bare, quoted, or dotted**; bare keys may contain only ASCII letters, digits, underscores and dashes (`A-Za-z0-9_-`) **(verified)**. Because bare keys are that restrictive, TOML can do something neither JSON nor YAML does safely: it can *always* write a key unambiguously, either bare or quoted.

TOML's richer types — dates and times — are the reason many teams choose it for application config over JSON. Combined with its string flavours, the type system looks like this:

| TOML construct | Notes |
|---|---|
| Basic string `"..."` and literal string `'...'` | Basic strings support escapes; literal strings are raw **(verified in structure)** |
| Multi-line basic `"""..."""` and multi-line literal `'''...'''` | Multi-line literal strings "are surrounded by three single quotes on each side and allow newlines. Like literal strings, there is no escaping whatsoever", and a newline immediately after the opening delimiter is trimmed **(verified)** |
| Integers and floats | With the spec's explicit discouragement of writing dotted keys that merely *look* like floats: `3.14159 = "pi"` is a valid two-part dotted key, and the spec says don't do it "unless you have a good reason to (you probably don't)" **(verified)** |
| Boolean | `true` / `false` |
| Offset/local date-time, local date, local time | The differentiator versus JSON |
| Arrays (possibly heterogeneous), inline tables | Inline tables are single-line; standard tables use `[header]` |
| Array of tables, `[[header]]` | "Any reference to an array of tables points to the most recently defined table element of the array" **(verified)** |

### 2.4 Like-for-like capability matrix

| Capability | JSON (RFC 8259) | YAML 1.2.2 | TOML 1.0.0 |
|---|---|---|---|
| Comments | **No** | Yes, `#` | Yes, `#` |
| Trailing commas | **No** | N/A (block) / flow allows them **[verify]** | Yes in multiline arrays/inline form |
| Native booleans | Yes, `true`/`false` only | Yes — Core schema `true`/`True`/`TRUE` and `false`/`False`/`FALSE` **(verified)**; 1.1 also `yes`/`no`/`on`/`off` **(verified)** | Yes, `true`/`false` |
| Native date/time | **No** | Yes via tag (`!!timestamp`) | **Yes, built in** |
| Null | Yes, `null` | Yes, `null`/`Null`/`NULL`/`~`/empty **(verified)** | No null type |
| Anchors / aliases / re-use | **No** | Yes — the aliasing mechanism is a core YAML feature | No |
| Multi-line strings | Only via `\n` escapes | Yes — literal `|` and folded `>` | Yes — `"""` / `'''` |
| Non-string mapping keys | **No** — names are strings | Yes | No |
| Order significant | Spec is silent; libraries differ **(verified)** | Yes, in the graph | Yes, in the document |
| Duplicate keys | Unspecified behaviour **(verified)** | An error | An error (redefinition rule) **(verified)** |
| Schema/validation story | JSON Schema (draft; see §5) | Thin, external | Thin, external |
| Binary / byte strings | No (Base64 by convention) | `!!binary` tag | No |

The matrix is the short version of §7's decision framework. Two rows carry disproportionate weight in practice: **comments** (a human-maintained config without comments is a support ticket waiting to happen) and **native date/time + no null** (TOML's shape, which makes it excellent for application config and poor for wire interchange).
---

## 3. The Pitfalls — Spec Rules vs Folklore

This is the section that matters most. Every entry below states the *precise* mechanism and labels it as a **spec rule**, an **implementation behaviour**, or **folklore** that the specs do not actually support. Each example shows the input and what it parses to.

### 3.1 The Norway problem (YAML booleans)

**Mechanism — spec rule + schema choice.** Under the **YAML 1.1** type system, booleans include the words `y`, `Y`, `yes`, `Yes`, `YES`, `n`, `N`, `no`, `No`, `NO`, `true`, `True`, `TRUE`, `false`, `False`, `FALSE`, `on`, `On`, `ON`, `off`, `Off`, `OFF`. The 1.1 specification itself says "the boolean 'true' might also be written as 'yes'" **(verified, YAML 1.1 §3.2.3.2)**. Under the **YAML 1.2.2 Core Schema**, only `true | True | TRUE | false | False | FALSE` resolve to `tag:yaml.org,2002:bool` **(verified, YAML 1.2.2 §10.3.2)** — `yes`/`no`/`on`/`off` are plain strings, and an unmatched plain scalar becomes `tag:yaml.org,2002:str` **(verified)**.

```yaml
# Input — a list of country codes, innocently unquoted
countries:
  - NO
  - SE
  - DE
```

```text
# Parsed under YAML 1.1 semantics (many installed parsers, incl. legacy PyYAML defaults):
countries: ["SE", "DE"]        # the value is False -> the key/value pair is not a string "NO"
                                # exact outcome: the element is the boolean False, not "NO"

# Parsed under YAML 1.2.2 Core Schema semantics:
countries: ["NO", "SE", "DE"]   # all three are strings, as the author intended
```

The casualty list is well known and follows mechanically from the same rule: the **Norway problem** (`NO` → `false`), and its 1.1 cousins `ON` (Ontario), `OFF`, `Y`, `N`. The same root cause silently mangles **version strings** (`1.10` is a float, not a version), **leading-zero identifiers** (`0123` is octal in 1.1 — the spec's own Example 2.19 shows `octal: 014` **(verified)**), and **sexagesimal-looking** values (`3:25:45` is an integer under 1.1 **(verified)**; the Core schema handles octal as `0o[0-7]+` **(verified)** and has no sexagesimal).

**The fix is a quoting rule, not a parser choice:** in YAML, *any scalar whose string value matters should be quoted*. `NO`, `"1.10"`, `"0755"`, `"yes"`.

### 3.2 Tabs, indentation, and the folded-scalar surprises

**Spec rule.** YAML's block structure is whitespace-determined, and **tabs may not be used for indentation** in block context — indentation is spaces. A tab in indentation position is a parse error, not a warning. This is the single most common hand-written YAML failure.

```text
# Input (tags shown as <TAB>)
server:
<TAB>host: db.internal

# Result: parse error. The parser expects an indented block; a tab is not valid indentation.
```

The related surprise is **line folding in multi-line scalars**: a single line break inside a folded (`>`) or plain multi-line scalar becomes a **space**, while empty lines become newlines — the spec's rationale is that this "allows for paragraphs to be word-wrapped without affecting the canonical form of the content" **(verified, YAML 1.1 §1.2 discussion of line folding)**. Authors who paste a multi-line shell command into a plain block scalar routinely get a command with all its newlines replaced by spaces.

```yaml
# Input
script: >
  echo one
  echo two

# Parsed to: "echo one echo two\n"   (the line break became a space)

# Use the literal style to keep the newlines:
script: |
  echo one
  echo two
# Parsed to: "echo one\necho two\n"
```

### 3.3 Numbers: precision, range, and the one-number-type problem

**Spec rule — JSON.** RFC 8259 §6 permits implementations to set limits on the range and precision of numbers accepted, and explains that because IEEE-754 binary64 doubles are widely available, interop is best when both sides assume no more precision than that; it names `1E400` and `3.141592653589793238462643383279` as examples that "may indicate potential interoperability problems" because the producer expects more capability than the consumer plausibly has **(verified)**.

```json
{"amount": 1234567890123456789, "ratio": 0.1, "big": 1E400}
```

```text
# Same bytes, three receivers:
JavaScript (JSON.parse)  ->  1234567890123456800  (double rounding; the last digits are gone)
Python (json.loads)      ->  1234567890123456789  (arbitrary-precision int preserved)
A strict decimal reader  ->  may reject 1E400 as out of range, or map it to Infinity

# And 0.1 is never exactly 0.1 in a binary64 float.
```

**Folklore to discount:** "JSON numbers are broken." They are not — the *format* carries an arbitrary decimal literal. The defect is that most parsers choose binary64, and the format offers no way to say "this is a decimal, please keep the scale". For money, carry a string (`"123.45"`) or a minor-units integer and let the schema say so.

**Spec rule — NaN and Infinity.** RFC 8259 §6 is categorical: "Numeric values that cannot be represented in the grammar below (such as Infinity and NaN) are not permitted" **(verified)**. So `{"x": NaN}` is not JSON. JavaScript's `JSON.stringify` converts such values to `null`; Python's `json.dumps` emits the bare tokens `NaN`/`Infinity` **by default**, producing output that its own strict reader will not accept — a producer/consumer split you should close explicitly.

```python
# Python default output is NOT valid JSON:
>>> import json; json.dumps({"x": float("nan")})
'{"x": NaN}'      # invalid per RFC 8259
>>> json.dumps({"x": float("nan")}, allow_nan=False)   # raises ValueError instead
```

**Spec rule — JSON has no leading zeros.** `{"n": 0123}` is invalid JSON **(verified, RFC 8259 §6)**. It is valid *literal* in several "JSON-like" dialects, which is the parser-differential risk of §6.

### 3.4 Duplicate keys

**Spec rule — JSON: unspecified.** RFC 8259 §4 says names SHOULD be unique, and that when they are not, "the behavior of software that receives such an object is unpredictable. Many implementations report the last name/value pair only. Other implementations report an error or fail to parse the object, and some implementations report all of the name/value pairs, including duplicates." **(verified)**. That is the spec explicitly refusing to decide — so a duplicate-key document is not portably parseable, whatever your parser does with it.

```json
{"tier": "gold", "tier": "platinum"}
```

```text
# Parse results observed across implementations, per RFC 8259 §4:
last-wins   ->  {"tier": "platinum"}     (most common)
error       ->  reject the document
all-pairs   ->  both pairs retained and reported
```

**Spec rule — YAML: an error.** YAML's data model requires mapping keys to be unique; a mapping with duplicate keys is invalid, and implementations are expected to reject it. (The cached 1.2.2 HTML did not let me grep the exact sentence — I have marked the exact wording **[verify]** in §12 rather than quote it.)

**Spec rule — TOML: an error by construction.** TOML's redefinition rule makes the equivalent case illegal: "tables cannot be defined more than once" and redefining a table via a `[table]` header, or via dotted keys where it was already defined in `[table]` form, "is not allowed" **(verified, TOML v1.0.0)**. The consequence for the operator is clean: TOML rejects rather than silently choosing.

### 3.5 JSON's absent features (and why people keep adding them)

**Spec rule.** RFC 8259 defines no comment syntax and no trailing commas. `// comment` before a value is not JSON. This is a deliberate minimalism, not an oversight, and it is the direct cause of the entire "JSON with niceties" family:

| Dialect | Status | What it adds |
|---|---|---|
| **JSON5** | **Not a formal standard.** The spec's own abstract calls it "a proposed extension to JSON that aims to make it easier for humans to write and maintain by hand", adding "some minimal syntax features directly from ECMAScript 5.1" **(verified)** | Comments, trailing commas, unquoted keys, single quotes, hex numbers, `NaN`/`Infinity` **[verify]** |
| **HJSON**, **JSONC**, **JSON with comments** | **Not standards** | Comments; `JSONC` is a loose label rather than a specified grammar |
| **NDJSON / JSON Lines** | De-facto convention, not an RFC | One JSON value per line — a *framing* convention, not a grammar change |

**Guidance that follows:** treat any file named `*.json` with a `//` in it as an incident, because a strict consumer somewhere in the pipeline will eventually be handed that file. Configure editors to write strict JSON. If a human-maintained config genuinely needs comments, that is an argument for YAML or TOML, not for a lenient parser.

### 3.6 TOML's ordering and re-opening rules

**Spec rule.** TOML's most surprising rule is at the table level: because a table is defined by its header or by dotted keys, and cannot be defined twice, *declaration order constrains what you may write later*. The spec is explicit that "Defining dotted keys out-of-order is discouraged" **(verified)** — legal but a trap:

```toml
# VALID BUT DISCOURAGED (verbatim shape from the spec)
fruit.apple.color = "red"
fruit.apple.taste.sweet = true
```

```text
# The problem: fruit.apple.taste was implicitly created as a table by the dotted key.
# If you later write [fruit.apple.taste], you are re-defining a table that already exists.
# The spec: "redefining such tables using a [table] header is not allowed".
```

Two more consequences of the same rule, both **verified** in v1.0.0:

- You cannot turn a scalar into a table: the spec shows `fruit.apple.smooth = true` followed by a `[fruit.apple]` header as invalid, with the comment "You can't turn an integer into a table."
- Arrays of tables append, and the *current* element is always the most recent one: "Any reference to an array of tables points to the most recently defined table element of the array" **(verified)**. This is why `[[products]]` blocks interleaved with sub-tables behave differently from what a JSON reader would guess.

The practical rule: **in TOML, define parents before children, and never mix dotted keys with `[table]` headers for the same path.**

### 3.7 Comments, BOM, and newlines

| Concern | JSON | YAML | TOML |
|---|---|---|---|
| Comment syntax | none | `#` | `#` **(verified)** |
| Encoding for open exchange | Must be UTF-8 **(verified, RFC 8259 §8.1)** | Unicode; BOM handling is a **spec-level** concern **[verify]** | Must be valid UTF-8 **(verified)** |
| BOM | RFC 8259 discusses byte-order marks in §8.1; the exact "MUST NOT add" wording was not confirmed by my grep — **[verify]** | A BOM is permitted at stream start; broad parser support for it is uneven **[verify]** | TOML requires a valid UTF-8 document **(verified)**; whether a leading BOM is accepted is **implementation-dependent** **[verify]** |
| Newlines | Insignificant whitespace | Line-break characters are normalised by the parser **(verified)** | Newline is LF or CRLF **(verified)** |

The operator's rule is boring and correct: **store text as UTF-8 without a BOM, with LF line endings, and enforce it in CI.** The interesting part is that all three formats *say* they are text-encoding agnostic to different degrees, while the failure modes are all "the first three bytes are EF BB BF and the parser rejected the file".

### 3.8 The folklore shelf

These are repeated constantly and are not supported by the specifications:

- **"JSON keys must be unique — the spec forbids duplicates."** RFC 8259 says SHOULD be unique, and explicitly describes three different outcomes **(verified)**. It does not forbid them.
- **"YAML is just JSON with indentation."** YAML is a superset in *capability*, but its type resolution differs from JSON's by default: the Core schema resolves `null`/`~`, `True`, and integer/float forms that JSON would read as strings. YAML's own §10.2 defines a *separate* JSON schema precisely because JSON is a subset, not the default **(verified)**.
- **"TOML is a subset of INI."** TOML borrows the line-oriented aesthetic but has typed values, arrays, dotted keys and an explicit redefinition rule. INI has none of that, and no two INI parsers agree on anything.
- **"YAML 1.2 replaced YAML 1.1."** The 1.2 documents are the current specification lineage, but a large fraction of deployed parsers report or implement 1.1-era resolution. Which rules you actually get is a *per-parser* fact **(see §4)**.
- **"JSON5 is a standard."** It is a published proposed extension, not a standard **(verified)**. Do not put it on an interchange contract.
---

## 4. Parsers and Tooling, Per Language and Ecosystem

The painful truth about all three formats is that **the specification is not the implementation**. The same bytes can produce three different in-memory objects depending on which library, which version, and which flags. This section maps the mainstream libraries and — crucially — flags the strict-vs-lenient variants, because the lenient ones are **not** the format.

### 4.1 JSON parsers

| Language / runtime | Mainstream choice | Strict by default? | Notes |
|---|---|---|---|
| Python | `json` (stdlib) | Mostly; **lenient by default in two notable ways** | `json.dumps` emits `NaN`/`Infinity` unless `allow_nan=False`; `json.loads` accepts them too unless `parse_constant` is used to reject |
| JavaScript / Node | `JSON.parse` / `JSON.stringify` | Yes for syntax | Numbers become binary64 — precision loss is silent for large integers; `JSON.stringify` silently converts `undefined`/functions to omitted keys and `NaN`/`Infinity` to `null` **(verified JavaScript behaviour)** |
| Java | Jackson, Gson, `jakarta.json` | Configurable | Jackson's default is strict-ish but `ALLOW_COMMENTS`, `ALLOW_TRAILING_COMMA`, `ALLOW_SINGLE_QUOTES` all exist as opt-in — **these switches change the grammar, so a file that needs them is not JSON** |
| Go | `encoding/json` | Yes | `Decoder.UseNumber()` preserves number literals as strings to avoid float rounding |
| Rust | `serde_json` | Yes | `arbitrary_precision` feature for lossless numbers |
| Ruby / PHP / C# | stdlib JSON | Yes | Same trailing-comma and comment rejections |

**The lenient variants are the danger.** Every `ALLOW_*` / `relaxed` switch is a **non-standard dialect**. A file that only parses with `ALLOW_TRAILING_COMMA` enabled is not a JSON document, and the next system in the chain — a browser, a Go service, a Kafka schema registry — will reject it. The rule: **turn the lenient switches off in producers, and validate at the boundary.**

### 4.2 YAML parsers — the 1.1 vs 1.2 split is real

| Language / runtime | Mainstream choice | Resolution behaviour | Safety default |
|---|---|---|---|
| Python | **PyYAML** (ubiquitous), `ruamel.yaml`, `strictyaml` | PyYAML's historical default loader resolves 1.1-style booleans; `ruamel.yaml` supports 1.1 and 1.2 modes explicitly | PyYAML exposes `yaml.safe_load` and `yaml.load`; the default for `yaml.load` moved toward the safe-ish `FullLoader` in newer releases — the exact version boundary is **[verify]** |
| Go | `gopkg.in/yaml.v3` | Modern, 1.2-leaning; **v2 is materially different from v3** and is still installed widely | No code execution, but alias expansion limits are implementation-specific **[verify]** |
| Ruby | `Psych` (stdlib) | 1.1-leaning historically | `Psych.safe_load` exists for the same reason PyYAML's does |
| Java | SnakeYAML, SnakeYAML Engine | SnakeYAML is 1.1-oriented; **SnakeYAML Engine** is the 1.2 rewrite | `SafeConstructor` vs the default constructor is *the* security-relevant setting |
| JavaScript | `js-yaml` | Supports 1.1 and 1.2 schema selection explicitly | `load` with a schema argument; `loadAll` for streams |
| Rust | `serde_yaml` (unmaintained as of the mid-2020s) / `serde_yml` / `yaml-rust2` | 1.2-leaning | No tag-based construction by default |

Two implementation-level differences that matter more than they look:

- **Kubernetes uses its own YAML→JSON pipeline.** `kubectl` and the API server consume JSON; YAML manifests are converted, and that conversion normalises types. The practical consequence is that `NO` in your manifest may arrive at the API server as `false` or as `"NO"` depending on the client's parser — which is why the Kubernetes ecosystem's own advice is to quote anything ambiguous. See [openshift_random_uid_username_mapping.md](openshift_random_uid_username_mapping.md), [openshift_scc_comprehensive_guide.md](openshift_scc_comprehensive_guide.md) and [charmed_kubernetes_vs_openshift_guide.md](charmed_kubernetes_vs_openshift_guide.md) for YAML-in-cluster specifics.
- **"YAML is JSON-compatible" is only true if you configure the JSON schema.** Under the default Core schema, a YAML file's `yes` is not JSON's `yes`, and its `~` is not JSON's `null`. This is exactly what YAML §10.2's JSON schema exists to fix **(verified)**.

### 4.3 TOML parsers

TOML's tooling is thinner but its conformance story is unusually good, because v1.0.0 came with an executable conformance suite in the wider ecosystem (the community `toml-test` harness; several parser READMEs advertise their pass status) **[verify — I did not open the suite]**.

| Language / runtime | Mainstream choice | Notes |
|---|---|---|
| Python | `tomllib` in the **standard library from Python 3.11** (read-only), `tomlkit` (round-trip preserving), `tomli` (the upstream of `tomllib`) | `tomllib` is read-only — there is no stdlib TOML *writer*, which is a deliberate guard against reformatting human configs |
| Rust | `toml` crate (serde-based) | The reference-quality implementation, used by Cargo |
| Go | `BurntSushi/toml`, `pelletier/go-toml` | Both advertise v1.0.0 conformance |
| JavaScript | `@iarna/toml`, `smol-toml` | `smol-toml` also serves as a YAML/JSON/TOML interop toolkit |
| Java | `tomlj`, `night-config` | `tomlj` advertises v1.0.0 |
| Config-file consumers | Cargo (`Cargo.toml`), `pyproject.toml`, Hugo, Netlify, many Rust tools | The strongest argument for TOML's longevity is that `pyproject.toml` and `Cargo.toml` are load-bearing in two of the largest packaging ecosystems |

**Version staleness is the TOML gotcha:** because v1.1.0 is published but its settled status is unconfirmed (§12), and because `tomllib` and most libraries implement **v1.0.0**, a config that uses a v1.1.0-only nicety will fail on the majority of installed parsers. Write v1.0.0.

### 4.4 Tooling that sits on top of all three

| Job | Tooling |
|---|---|
| Lint and format | `prettier` (JSON/YAML), `yamllint`, `jsonlint`, `taplo` (TOML) |
| Convert between formats | `yq`, `dasel`, `jq` (JSON), `tomlq` (part of yq) |
| Schema validation in CI | `ajv` (JSON Schema), `check-jsonschema`, `jsonschema` (Python), `kubeconform` / `kubeval` (Kubernetes) |
| Editor correctness | Language servers for YAML/JSON and TOML schema completion; a strict-schema setting in the editor is the cheapest bug-prevention available |

These tools live in the tooling layer this guide deliberately does not re-derive; see [configuration_management_languages_guide.md](configuration_management_languages_guide.md) for how they compose into pipelines, and [power_platform_cicd_guide.md](power_platform_cicd_guide.md) for the YAML-pipeline flavour of the same problem.

---

## 5. Schemas and Validation

### 5.1 JSON Schema — the one mature story

JSON Schema is *the* schema language for these formats, and the honest framing is that it is an **IETF Internet-Draft lineage, not a final RFC**. The widely implemented revision is the **2020-12** series (the core document circulating as `draft-bhutton-json-schema-01`), with the older draft-07 still extremely common in tooling and in existing contracts **(verified against the cached JSON Schema core draft; the "which draft does my tool support" question is the practical one)**.

The vocabulary is the reason JSON Schema dominates:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "required": ["account_id", "currency", "amount"],
  "additionalProperties": false,
  "properties": {
    "account_id": { "type": "string", "pattern": "^ACC-[0-9]{8}$" },
    "currency":   { "type": "string", "enum": ["GBP", "EUR", "USD"] },
    "amount":     { "type": "string", "pattern": "^-?[0-9]+\\.[0-9]{2}$" },
    "booked_at":  { "type": "string", "format": "date-time" }
  }
}
```

Four properties of JSON Schema that consistently surprise people:

- **`format` is annotation-first.** The core specification treats `format` as an annotation by default; whether `"date-time"` is *enforced* depends on the validator and its configuration. If your contract depends on date validation, assert it (or use another keyword) rather than assuming it.
- **`additionalProperties: false` is not the default.** Without it, extra keys pass. This is how contracts drift: the schema says "these fields" and the wire says "these fields plus eleven others".
- **`$ref` resolution and `$id` semantics changed between drafts.** A schema written for draft-07 does not always validate identically under 2020-12. Pin the `$schema` URI.
- **`unevaluatedProperties` and dynamic references are 2019-09+/2020-12 features.** Older validators ignore them silently, which is worse than erroring.

For the pipeline view — how schemas evolve, how you detect drift, and what a compatibility policy looks like — see [schema_evolution_data_drift_guide.md](schema_evolution_data_drift_guide.md), and for the contract-governance layer above it, [api_governance_guide.md](api_governance_guide.md).

### 5.2 YAML validation — thinner, and that is the honest answer

There is **no ISO/IEC or IETF schema language for YAML**. What exists is a patchwork:

| Approach | What you get | Caveat |
|---|---|---|
| **JSON Schema against the YAML** | The best practical answer: parse YAML to a data model (with the JSON schema or Core schema, §4.2), then validate as JSON | Type quirks (§3.1) mean the YAML surface and the JSON schema do not line up automatically |
| **Kubernetes structural schemas / OpenAPI** | Real, enforced, and applied by the API server | Kubernetes-specific, not a general YAML schema |
| **YAML's own tag system** | `!!timestamp`, `!!binary`, custom tags | Tags express types, not constraints — there is no "MUST match this pattern" |
| **`strictyaml`** | A Python library that *reduces* YAML to a typed subset with schema validation | It is a dialect policy enforced by one library, not a standard |

The spec's own contribution is the three recommended schemas (§2.2), which are about *type resolution*, not validation **(verified)**. Anyone telling you there is a YAML Schema standard is describing a tool, not a specification.

### 5.3 TOML validation — the same story, plus one advantage

TOML has no schema language of its own. It does have a genuine structural advantage over YAML: **the redefinition rule makes much of the "did the author mean this?" ambiguity impossible** — a table cannot be silently re-declared, dotted keys cannot conflict with headers, and an array of tables is explicitly ordered and append-only **(verified)**. Where TOML validation is needed, teams either (a) validate the parsed data with JSON Schema after conversion, or (b) use the language that owns the file — Cargo validates `Cargo.toml` against its own Rust structs, and Python build backends validate `pyproject.toml` against `[build-system]` metadata rules. Both are **tool-level** validation. That is the thin story, stated plainly.

### 5.4 A validation policy that survives contact with production

| Gate | Format | Mechanism |
|---|---|---|
| Syntax | all three | Parse with a strict parser; **fail the build on parse error, never warn** |
| Dialect | all three | Reject lenient extensions (comments in `.json`, tabs in YAML indentation, duplicate keys) |
| Structure | JSON/YAML | JSON Schema, `additionalProperties: false`, `$schema` pinned |
| Semantics | all three | Cross-field rules the schema cannot express (e.g. "if `env: prod` then `replicas >= 3`") — implement as code in CI |
| Encoding | all three | UTF-8, no BOM, LF line endings |
| Secrets | all three | Scan for key patterns and high-entropy strings; see §6.4 |

The point of the gate table is that a schema validator alone is not a validation *strategy*. It catches shape, not intent.
---

## 6. Security: Deserialisation, Aliases and Secrets

Configuration formats fail in three ways that are security-relevant: the parse itself can **execute** something, the parse can be made to **exhaust resources**, and the config's *content* can carry **secrets** into places secrets should not be.

### 6.1 The deserialisation hazard — YAML's tag system

This is the classic and it is still the one that bites. YAML's tag system lets a document describe an *application type*, not just a value. A general-purpose YAML loader that honours application tags may therefore construct objects — and in dynamically typed languages, constructing an object can invoke constructors, class resolution, and in the worst case function application.

The canonical dangerous shape is a document that names a Python class and its arguments, e.g. a `!!python/object/apply` tag pointing at a shell-adjacent callable. Parsed with a loader that honours such tags, that is arbitrary code execution triggered by *reading a config file*.

```yaml
# Conceptual shape of the hazard — a YAML document that is not data.
# Parsed by a loader that honours application tags, the named callable is invoked.
# A loader that does not honour those tags returns either a string, an error,
# or a tagged value it refuses to construct.
some_key: !!python/object/apply:os.system ["echo pwned"]
```

**The rule, and this is the whole of §6.1:** use `yaml.safe_load` (or the language's equivalent restricted constructor) for **every** untrusted or merely unfamiliar YAML document, and treat `yaml.load` with a permissive loader as code execution with extra steps.

| Language | Unsafe | Safe |
|---|---|---|
| Python (PyYAML) | `yaml.load(stream)` with `Loader`/`UnsafeLoader` | **`yaml.safe_load(stream)`** — constructs only standard YAML types |
| Python (ruamel.yaml) | `YAML(typ='unsafe')` | `YAML(typ='safe')` |
| Java (SnakeYAML) | default `Yaml()` constructor | **`new Yaml(new SafeConstructor(...))`** |
| Ruby (Psych) | default class-loaded modes | **`Psych.safe_load`** |
| JavaScript (js-yaml) | `load` with a permissive schema | `load` with `JSON_SCHEMA` / `CORE_SCHEMA` as appropriate |

**Nuance worth stating precisely:** the *specification* does not mandate that a processor construct application objects (`tag:yaml.org,2002:*` values are about the data model), and a conforming processor is free to reject or stringify unknown tags. The danger is entirely an **implementation** property — which is exactly why the fix is a library setting and not a spec patch. The remaining question that I could not settle at the primary spec — the precise release boundary at which PyYAML changed `yaml.load`'s default loader — is flagged **[verify]** in §12.

### 6.2 Alias and anchor expansion — YAML bombs

YAML anchors (`&a`) and aliases (`*a`) let a node be referenced repeatedly — the mechanism the 1.1 spec describes as serialising "native graph data structures through an aliasing mechanism" **(verified, YAML 1.1 §1.2)**. The attack is that *references can be nested*, so a small document can expand to an exponential amount of data. The pattern is a "billion laughs" document:

```yaml
# Sketch of the expansion class of attack: each level references the previous twice.
a: &a ["x", "x", "x", "x", "x", "x", "x", "x", "x"]
b: &b [*a, *a, *a, *a, *a, *a, *a, *a, *a]
c: &c [*b, *b, *b, *b, *b, *b, *b, *b, *b]
d: &d [*c, *c, *c, *c, *c, *c, *c, *c, *c]
e: &e [*d, *d, *d, *d, *d, *d, *d, *d, *d]
# ... one self-referential level per doubling of memory and time
```

```text
# What happens: the document is tiny (a few hundred bytes); the expanded node graph
# is 9^n. A parser that expands eagerly without a budget exhausts memory or CPU.
```

**Status: this is an implementation-behaviour class, not a spec rule.** The YAML specification does not mandate a resource budget, and whether a given library limits total alias expansion, limits alias *depth*, or caps the expanded node count varies by library and version — I could not confirm a specific numeric limit at a primary source, so the specific limits are marked **[verify]** in §12 and should be tested against *your* pinned version rather than assumed. The defensible operational posture is: **never parse YAML from an untrusted producer without a size and alias budget, and prefer the JSON schema (which has no aliases) for untrusted input.**

### 6.3 JavaScript: parser differentials and prototype pollution

Two distinct problems get lumped together here, and they deserve separation.

**Prototype pollution via `__proto__`.** `JSON.parse` itself defines own properties; it does not invoke setters, so a document containing `"__proto__"` produces an object with a *key* named `__proto__`, not a mutated prototype. The danger appears downstream, when application or library code **recursively merges** parsed config into an existing object graph:

```json
{"__proto__": {"isAdmin": true}, "constructor": {"prototype": {"isAdmin": true}}}
```

```text
JSON.parse(input)      ->  { "__proto__": {...} } as an own property. No pollution yet.
naive deepMerge(config, TARGET)  ->  writes through the setter -> TARGET and every object
                                     inheriting from that prototype may now report isAdmin.
```

So the mitigation is **not** "use `JSON.parse`" (you already are) — it is **reject or neutralise `__proto__` and `constructor` keys in any merge path**, use `Object.create(null)` or a `Map` for config maps, and prefer explicit field copying over blind merging.

**Parser differentials.** When two parsers in the same pipeline disagree about the same bytes, the *disagreement* is the vulnerability. The recurring axes:

| Axis | Divergence |
|---|---|
| Duplicate keys | last-wins vs error vs all-pairs — **spec-sanctioned divergence** (RFC 8259 §4) **(verified)** |
| Large integers | binary64 rounding vs arbitrary precision **(§3.3)** |
| `NaN`/`Infinity` | strict reject vs accept — **invalid per RFC 8259 §6** **(verified)** |
| Leading zeros | strict reject vs lenient accept **(§3.3)** |
| Unicode escapes / lone surrogates | differing tolerance |
| Key normalisation | NFC vs NFD — two "different" keys that render identically |

The mitigation is the same everywhere: **one parser, pinned, at the trust boundary; canonicalise (or reject) before comparing anything** such as a signature, an authorisation decision, or a cache key.

### 6.4 Secrets in configuration

All three formats are plain text, and all three end up in git. The failure modes are format-independent but format-*aggravated*:

| Hazard | Why the format makes it worse |
|---|---|
| Committed credentials | Comments make it easy to leave a "temporary" plaintext password in a YAML or TOML file "for now" |
| YAML anchors re-using one secret across many resources | One rotation point, one leak surface — and the alias hides the duplication from grep |
| Base64 mistaken for encryption | JSON's convention of Base64 for binary plus no comments makes `{"secret": "..."}` look intentional |
| Config in CI logs | A `--debug` flag dumping parsed YAML prints resolved values, aliases included |
| Schema `default` fields | A default credential in a JSON Schema is a credential in a public artifact |

The controls are well-established and belong to the platform, not the format: **scan on commit** (secret scanning in CI), **reference not inline** (env vars, mounted secrets, a secrets manager), **rotate on suspected exposure**, and **never treat Base64 as a control**. The representation is text; treat every one of these files as public unless proven otherwise.

---

## 7. Selection Guidance and a Decision Framework

### 7.1 The axes that actually decide it

| Axis | JSON | YAML | TOML |
|---|---|---|---|
| Hand-editability | Poor (no comments, quoting noise) | Good if disciplined; fragile to indentation | **Best** — explicit, comment-friendly, unambiguous |
| Machine interchange | **Best** — universal, smallest parser surface | Workable, but the type ambiguities of §3 follow you | Workable, but no null type and no non-string keys |
| Expressiveness beyond JSON types | None | Dates, binary, anchors, multi-line | **Dates and times built in** |
| Comment support | None | `#` | `#` |
| Strictness of parse | High | Low-to-medium (schema-dependent) | High (redefinition rule) |
| Schema ecosystem | **Mature (JSON Schema)** | Thin | Thin |
| Streaming/framing | NDJSON convention | Multi-document streams | No |
| Human-diff quality | Noisy for large docs | Good | Good |
| LLM generation reliability | High | **Lower — indentation is the failure mode** (§8) | Medium-high |

### 7.2 Domain conventions — the de-facto answer

Most real decisions are already made for you by the ecosystem you are standing in, and fighting that is expensive:

| Domain | De-facto format | Why |
|---|---|---|
| HTTP APIs, event payloads, browser apps | **JSON** | Universal parsers, `application/json`, JSON Schema |
| Kubernetes / OpenShift manifests, Helm, Argo CD | **YAML** | The ecosystem; see §4.2 and the K8s guides |
| CI/CD pipelines (GitHub Actions, GitLab CI, Azure Pipelines) | **YAML** | Ecosystem convention; see [power_platform_cicd_guide.md](power_platform_cicd_guide.md) |
| Application config in Python/Rust projects | **TOML** | `pyproject.toml` / `Cargo.toml` are load-bearing |
| Tool config where comments and nesting both matter | **TOML or YAML** | JSON's no-comments rule is disqualifying for human-maintained files |
| Configuration *generation* (typed config) | CUE / Jsonnet / Dhall / Pkl / Nix | Their output is usually JSON or YAML; see the CM guide |

### 7.3 Lossy conversions — where the formats are not interchangeable

Conversion is where good intentions produce silent data loss. The directional rules:

| Conversion | Losses |
|---|---|
| YAML → JSON | Anchors/aliases are *expanded* (size explosion, loss of the re-use intent); non-string mapping keys cannot be represented; comments vanish; tags with no JSON equivalent (`!!timestamp`) must be re-encoded as strings |
| JSON → YAML | Round-trips in *data* if you emit with the JSON schema, but your emitter's style choices (quoting, line width) will make diffs noisy |
| TOML → JSON | Straightforward: dates become strings, and **null does not exist to lose** |
| JSON → TOML | **Fails on null** (TOML has no null), and on non-homogeneous arrays in some writers; nested structures must be re-expressed as tables/dotted keys |
| YAML ↔ TOML | The lossy one, both directions: comments survive only in hand-editing, and type differences (1.1 booleans, dates) do not map cleanly |

**Rule:** convert at the edges, never in the middle of a pipeline. Pick a canonical in-memory representation (usually JSON-shaped) and make the format a *representation* of it, not a second source of truth.

### 7.4 The decision framework

Answer in order; the first "yes" wins.

1. **Is this crossing a service boundary or a language boundary?** → **JSON.** Nothing else is as universally parseable, and a JSON Schema lets you enforce the contract.
2. **Does a tool own the file?** (Kubernetes manifest, CI pipeline, `pyproject.toml`, `Cargo.toml`, Ansible) → **Whatever that tool reads.** Do not fight the ecosystem.
3. **Will a human maintain it by hand, and does it need comments?** → **TOML** for flat-to-moderately-nested application config with dates and numbers; **YAML** when the ecosystem already chose it and the structure is tree-like.
4. **Does it need anchors/aliases or multi-line text blocks as first-class features?** → **YAML**, with the §3 discipline: quote every scalar whose string value matters.
5. **Is it data interchange at volume?** → **JSON** (or a binary format entirely outside this guide — the three text formats are for humans-in-the-loop as much as machines).
6. **Is it untrusted input?** → **JSON**, parsed strictly, with `__proto__`/`constructor` rejected in any merge path, and no alias mechanism to abuse.

**Non-negotiables regardless of choice:** strict parser, pinned version, one canonical serialiser, UTF-8 without BOM, LF endings, quoted ambiguous scalars, JSON Schema (or the tool's own schema) in CI, and secret scanning on commit. Those six lines are the entire operational payoff of this guide.
---

## 8. The AI/LLM Angle

Language models read and write these three formats constantly — as tool arguments, as structured output, as generated configuration. The format-level facts that matter are narrower than the hype, and they are testable.

### 8.1 Why JSON dominates tool-calling and structured output

The dominance is structural, not aesthetic, and three properties explain it:

- **The grammar is tiny and unambiguous.** RFC 8259's token set — six structural characters, strings, numbers, three literal names **(verified)** — makes constrained decoding tractable. A model, or a grammar-guided sampler, can be restricted at each token step to the productions the grammar allows. Doing the same for YAML means tracking indentation, block/flow style, anchors and schema-dependent scalar resolution — a vastly larger state space.
- **There is a mature schema language to constrain *against*.** JSON Schema's `required`, `enum`, `type` and `pattern` give a tool-calling layer something to enforce beyond "is it valid JSON" **(see §5.1)**. YAML and TOML have no comparable standard.
- **Every runtime parses it.** A tool-call response is consumed by host code in a language the model cannot see. `application/json` is the only option with a parser in every one of them.

The consequence for prompt and tool design: **make the model's output format JSON, and make the schema the interface.** If a tool's arguments are described in prose and then parsed from free text, you have reintroduced the parsing problem the format solved.

### 8.2 LLM config generation: where reliability actually breaks

The recurring failure modes, in rough order of observed frequency:

| Failure mode | Format | Why it happens | Mitigation |
|---|---|---|---|
| Indentation drift | YAML | Indentation *is* the grammar; a model emitting "reasonable-looking" nesting gets sibling/parent relationships wrong, and the file parses to the wrong shape (or fails) | Ask for JSON, convert with a library; or validate and re-prompt on parse error |
| Hallucinated keys | all | The model completes the plausible shape of a config rather than the actual one | Give the real schema in context; validate with `additionalProperties: false` |
| Comment-stripping | JSON | The model "helpfully" adds `//` comments to JSON | State the constraint explicitly; lint for it |
| Type coercion | YAML | Emitting `version: 1.10`, `port: 8080` unquoted, or `no` as a value — the §3.1 and §3.3 traps, produced at scale | Quote-scalars rule as a prompt constraint and a CI check |
| Truncation | all | Long configs exceed the output budget, leaving an unterminated structure | Generate in chunks; prefer a generator that emits per-resource |
| Number formatting | JSON | Money and IDs emitted as binary64-shaped literals | Schema says string-with-pattern for money and identifiers |
| Invented schema URIs | JSON | A `$schema` URL that does not resolve | Validate the `$schema` against an allow-list |

**The single most reliable lever** is not a better prompt — it is **validation in the loop**. Parse the model's output with the strict parser, validate against the schema, and feed the *error* back. A model that is told "line 12: found tab character in indentation" fixes it in one turn; a model asked to "be careful with YAML" does not.

### 8.3 YAML indentation as the recurring failure mode

Worth calling out separately because it combines three properties: indentation is semantically load-bearing, the error is often **not a parse error but a wrong parse**, and it is invisible in review. The dangerous case:

```yaml
# Model output — plausible, and silently wrong
containers:
  - name: api
    image: registry.internal/api:1.4.2
  env:
    - name: LOG_LEVEL
      value: info
```

```text
# What a YAML parser sees: `env` is a SIBLING of `containers`, at the document level —
# not a child of the container. Nothing fails to parse. The deployment simply
# never receives LOG_LEVEL.
```

A tab anywhere in that indentation, meanwhile, is a hard parse error (§3.2). So the failure distribution is "half the mistakes are silent, half are fatal", which is the worst of both.

**Practical posture for AI-generated configuration:** generate JSON as the machine-readable intermediate, keep YAML only at the interface where a tool demands it, convert with a library rather than by hand, and put `yamllint` plus a JSON-Schema-derived structural check on the path. For the agent-pipeline view, see [ai_llm/agent_scaffolding_guide.md](ai_llm/agent_scaffolding_guide.md) and [ai_llm/agentic_solution_artifacts_guide.md](ai_llm/agentic_solution_artifacts_guide.md), where these checks appear as tool-call validators and artifact gates.

---

## 9. Cymbal Bank Worked Example — A Configuration Standard for the Platform Estate

> **This scenario is illustrative and fictional.** Cymbal Bank is a stand-in persona used to make a design argument concrete. Nothing below describes a real institution, system or incident.

### 9.1 The problem statement

The (fictional) Cymbal Bank platform estate carries four distinct classes of machine-readable file:

| Class | Volume driver | Current state | Failure seen |
|---|---|---|---|
| **Kubernetes / OpenShift manifests** | Hundreds of services × environments | YAML, templated per team | Unquoted `NO`/version scalars; inconsistent indentation; no structural gate |
| **Application configuration** | Per-service config files, per environment | Mixed: JSON for some services, YAML for others, `.properties`/INI legacy | No comments in the JSON ones; no schema; config drift between environments |
| **API contracts** | OpenAPI documents for internal and partner APIs | YAML in some repos, JSON in others | Drafted OpenAPI in YAML, generated clients in JSON — divergence on regeneration |
| **Data interchange** | Events on the internal bus, batch hand-offs | JSON | Large integers (account identifiers) rounded in one JavaScript consumer; `NaN` emitted by one Python producer |

The four classes have different owners and different tool constraints, which is precisely why a single "we standardise on X" edict fails. The decision has to be **per class**.

### 9.2 The decision

| Class | Standard | Rationale (mapped to §7) |
|---|---|---|
| K8s / OpenShift manifests | **YAML, strict subset** | The ecosystem owns the file (§7.2). Deviating means fighting `kubectl`, Helm, GitOps controllers and every operator. The standard is therefore *discipline*, not a different format |
| Application configuration | **TOML for human-maintained config; JSON only where a generated artifact or existing consumer demands it** | Comments and unambiguous typing are the requirements §7.3 scores TOML best on; dates and numbers are first-class |
| API contracts | **One format per contract, chosen once and generated from a single source** | The defect was not YAML-vs-JSON but *two sources of truth*. Between them, JSON where the contract is machine-consumed in the browser (JSON Schema's ecosystem wins, §5.1); YAML where the authoring experience dominates and the ecosystem (OpenAPI tooling) reads both |
| Data interchange | **JSON, strict, with a JSON Schema per event type** | Universality and schema support (§7.4 step 1). Numbers that must not round become strings |

The explicit non-decision: **YAML is not banned for application config, and TOML is not mandated for manifests.** Both would be cargo-cult uniformity.

### 9.3 Guardrails — the "strict subset" rules

For YAML (manifests and any remaining YAML app config), the platform standard is a **strict subset** with machine-checkable rules:

| Rule | Reason | Enforcement |
|---|---|---|
| Every scalar whose string value matters is quoted (`NO`, `"1.10"`, `"0755"`, `"yes"`, `"on"`) | Prevents the §3.1 Norway problem entirely | `yamllint` custom rule + review checklist |
| No tabs anywhere; two-space indentation | Tabs are a parse error in block context (§3.2) | `yamllint` |
| No anchors/aliases in manifests | Removes the §6.2 expansion class and makes each manifest independently readable | `yamllint` `anchors: forbid` |
| No duplicate keys | Spec-invalid in YAML; the failure is silent in lenient parsers (§3.4) | Parser configured to error |
| Explicit `---` document start where multiple documents are possible | Prevents accidental stream interpretation | Lint rule |
| All numbers that are identifiers are quoted | Version strings, port-as-string, leading zeros | Lint rule + schema types |
| Secrets referenced, never inline | §6.4 | Secret scanning gate |

For JSON (interchange), the subset rules are the format's own: **no comments, no trailing commas, no `NaN`/`Infinity`, no duplicate keys, no lenient parser flags** (§3.5, §3.3, §4.1).

For TOML, the rules follow §3.6: **declare parents before children; never mix dotted keys with `[table]` headers for the same path; target v1.0.0 only.**

### 9.4 CI validation gates

Five gates, in order, each of which *fails* the build rather than warning:

1. **Parse gate.** Every file parses with the pinned strict parser at the pinned version. No lenient flags anywhere.
2. **Subset gate.** `yamllint` with the platform ruleset for YAML; `taplo`/`toml` conformance for TOML; a JSON lint that rejects `NaN`, duplicate keys and comments.
3. **Schema gate.** K8s manifests through a server-side dry-run or a structural-schema validator (e.g. `kubeconform`); application config through JSON Schema after conversion; interchange events through their event schema.
4. **Encoding gate.** UTF-8, no BOM, LF line endings (§3.7).
5. **Secret gate.** Pattern and entropy scanning on every commit.

The gates are deliberately boring, and that is the point: every one of them maps to a specific pitfall in §3 or §6, so the CI log reads like the table of contents of this guide.

### 9.5 Migration and conversion plan

| Phase | Action | Exit criterion |
|---|---|---|
| 0 — Inventory | Enumerate every JSON/YAML/TOML file in the estate, grouped by the four classes | A count per class, and a named owner per class |
| 1 — Gate the new | Turn on the five gates for *new and changed* files only | No new violations for two release cycles |
| 2 — Fix the dangerous | Remediate first by *hazard*, not by repo: duplicate keys, unquoted ambiguous scalars, tabs, inline secrets | Hazard count at zero |
| 3 — Convert by class | Manifests stay YAML (subset enforced). Human-maintained app config moves to TOML service by service. API contracts get a single generated source. Interchange stays JSON with schemas | Every class has one format and one source of truth |
| 4 — Retire the rest | Remove `.properties`/INI legacy readers, remove JSON-with-comments, remove the unused of YAML-vs-JSON for each contract | Old parser dependencies deleted, not just deprecated |

**The conversion rule that prevents the return of the problem:** conversions are performed by a scripted, reviewed toolchain (parse → normalise → emit), never by hand-editing, and the toolchain is checked into the repository. A hand conversion loses comments, anchors and intent (§7.3) and cannot be audited.

### 9.6 What this illustrates

The example's lesson is not "TOML is better" or "YAML is dangerous". It is that **the format decision is per class, and the value is in the guardrails and gates, not in the choice of format.** Three formats, four classes, one set of mechanical rules — and the rules are simply the pitfall list of §3 turned into red builds. That is the whole discipline of JSON, YAML and TOML in a bank platform estate: choosing per class, and then making the parser, the linter and the gate enforce what the specification actually says about the configured data.

---

## 10. Evolution, Conversion and Standards Governance

**How the formats evolve.** All three evolve slowly and conservatively, which is why they are still here. JSON's last normative step was RFC 8259 in December 2017, which was itself a consolidation (applying errata and removing inconsistencies) rather than a feature release **(verified)** — and it is the *absence* of new syntax that keeps JSON's parser surface small. YAML's current lineage is 1.2.2 with no normative changes from 1.2 **(verified)**, so the interesting evolution happened on the *schemas* (Failsafe/JSON/Core) and on the media-type registration (RFC 9512, February 2024) **(verified)**. TOML reached a stable 1.0.0 in January 2021 that the ecosystem could converge on **(verified)**; 1.1.0 is published with a 2025-12-18 date **(verified)** but with a settled status I could not confirm (§12) — which is itself the governance lesson: **a format's adoption clock is set by its parsers, not its specification page.**

**What to pin and why.**

| Pin | Rationale |
|---|---|
| **Parser + version** | The 1.1-vs-1.2 behaviours of §3.1 and the alias limits of §6.2 are version-level facts |
| **`$schema` draft URI** | JSON Schema's drafts are not interchangeable (§5.1) |
| **Serialiser** | One canonical emitter per repository keeps diffs reviewable and prevents formatting wars |
| **Language/spec target** | Write TOML v1.0.0 until v1.1.0's status settles; write YAML that resolves identically under both 1.2.2 and legacy 1.1 semantics by quoting |
| **Schema registry entry per event type** | Ties the format decision to the contract, not the file extension |

**Where to keep the schema discussion.** Schema *evolution* — compatibility modes, drift detection, whether a field removal is breaking — is a different discipline from the format grammar, and it is covered in [schema_evolution_data_drift_guide.md](schema_evolution_data_drift_guide.md). The API-contract governance layer is in [api_governance_guide.md](api_governance_guide.md), the pipeline-versioning angle in [data/data_pipeline_versioning.md](data/data_pipeline_versioning.md), and the language/tooling layer in [configuration_management_languages_guide.md](configuration_management_languages_guide.md). This guide's job ends where the bytes have become an object.

**The closing judgement.** Three formats, each with a small and stable specification, each surrounded by a much larger and less stable ecosystem of parsers. The specifications are the easy part; RFC 8259, YAML 1.2.2 and TOML v1.0.0 are each readable in an afternoon. The hard part is that the *implementation* is what runs — so the discipline is to know which schema resolves your scalars, which loader you have invoked, which numbers your runtime can hold, and which rules your CI enforces. Get those four right and the format question reduces to a short, defensible per-class decision about the configured data.
---

## 11. Claims Audit

Each row is a load-bearing factual claim in this guide, with the source I checked it against and its status: **verified** (confirmed at a primary specification or official document), **flagged** (marked `[verify]` inline — could not be confirmed at a primary source), or **rejected** (a widely repeated claim the sources do not support).

| # | Claim | Status | Source checked |
|---|---|---|---|
| 1 | RFC 8259 is dated December 2017, editor T. Bray (Textuality), Standards Track, obsoletes RFC 7159 | verified | RFC 8259 header and §1.3 (cached full text) |
| 2 | RFC 8259 is an Internet Standard, published as STD 90 | verified | RFC Editor STD 90 page (`std90.html`) |
| 3 | RFC 8259 registers the `application/json` media type | verified | RFC 8259 (cached) |
| 4 | JSON's grammar traces to ECMA-404, 2nd Edition, December 2017, *The JSON Data Interchange Syntax* | verified | `ecma404.pdf` title block |
| 5 | JSON originated at json.org (Douglas Crockford) circa 2001 | verified | json.org canonical grammar page (`jsonorg.html`) / parent anchor |
| 6 | A JSON text is a serialized value; tokens are six structural characters, strings, numbers and three literal names | verified | RFC 8259 §2 |
| 7 | JSON represents four primitive types and two structured types | verified | RFC 8259 §1 |
| 8 | RFC 8259 permits implementations to limit number range/precision; IEEE-754 binary64 for best interop; `1E400` and long decimals suggest interop problems | verified | RFC 8259 §6 |
| 9 | `Infinity` and `NaN` are not permitted in JSON; leading zeros are not allowed | verified | RFC 8259 §6 |
| 10 | Duplicate object names have unpredictable behaviour: last-wins, error, or all-pairs reported | verified | RFC 8259 §4 |
| 11 | JSON libraries differ in whether they expose member ordering to callers | verified | RFC 8259 §4 |
| 12 | JSON text exchanged between systems not in a closed ecosystem MUST be UTF-8 | verified | RFC 8259 §8.1 |
| 13 | The exact "MUST NOT add a byte order mark" wording, and BOM acceptance per parser | flagged | RFC 8259 §8.1 grep returned matches but not the sentence; parser behaviour not tested |
| 14 | YAML 1.2.2 is dated 2021-10-01, by the YAML Language Development Team, with no normative changes from 1.2 | verified | YAML 1.2.2 spec (`yaml122.html`) / parent anchor |
| 15 | RFC 9512 (Feb 2024, Informational) registers `application/yaml` and the `+yaml` suffix | verified | RFC 9512 abstract and header (cached) |
| 16 | YAML was widely used before any media-type registration; deprecated aliases include `application/x-yaml`, `text/yaml` | verified | RFC 9512 §1 (motivation text) + parent anchor |
| 17 | YAML 1.2.2 Core Schema resolves only `true/True/TRUE/false/False/FALSE` as booleans; `null/Null/NULL/~`/empty as null; `0o[0-7]+` as octal; unmatched plain scalars as strings | verified | YAML 1.2.2 §10.3.1–10.3.2 resolution table |
| 18 | YAML 1.1 allows booleans spelled `yes`/`no`/`on`/`off` etc. ("the boolean 'true' might also be written as 'yes'") and its Example 2.19 shows `sexagesimal: 3:25:45` and `octal: 014` | verified | YAML 1.1 spec §3.2.3.2 and Example 2.19 (cached) |
| 19 | YAML has Failsafe (§10.1), JSON (§10.2) and Core (§10.3) schemas; Core is the recommended default; JSON schema is the lowest common denominator | verified | YAML 1.2.2 §10.1–10.3 |
| 20 | YAML spec says comments and scalar *format* are presentation details with no effect on the representation graph, and are not associated with nodes | verified | YAML 1.1 §3.2.3.2–3.2.3.3 |
| 21 | YAML's line folding turns a single line break into a space and empty lines into line breaks | verified | YAML 1.1 §1.2 (line-folding rationale) |
| 22 | YAML anchors/aliases exist to serialize native graph structures | verified | YAML 1.1 §1.2 |
| 23 | TOML v1.0.0 is dated 2021-01-11, by Tom Preston-Werner, Pradyun Gedam, et al.; MIME type `application/toml`; extension `.toml` | verified | toml.io v1.0.0 page (cached) |
| 24 | TOML v1.1.0's page is dated 2025-12-18; its settled/ratified status is not asserted on the page | verified (date) / flagged (status) | toml.io v1.1.0 page (cached) |
| 25 | A TOML file must be valid UTF-8; whitespace is tab or space; newline is LF or CRLF | verified | TOML v1.0.0 spec preamble (cached) |
| 26 | TOML bare keys are `A-Za-z0-9_-` only; dotted keys are discouraged out-of-order; tables cannot be redefined; arrays of tables resolve to the most recent element | verified | TOML v1.0.0 "Keys" and "Table"/"Array of Tables" sections (cached) |
| 27 | TOML multi-line literal strings use three single quotes, allow newlines, and have no escaping | verified | TOML v1.0.0 "String" section (cached) |
| 28 | TOML's stated objective is a minimal config format that maps unambiguously to a hash table | verified | TOML v1.0.0 "Objectives" (cached) |
| 29 | JSON5 is a *proposed extension*, not a formal standard, adding ECMAScript 5.1 syntax features | verified | JSON5 spec abstract (`json5.html`) |
| 30 | HJSON is not a standard | verified | Parent's repo-wide format check (`hjson.html` non-normative site) |
| 31 | JSON Schema's widely implemented revision is 2020-12 (IETF draft lineage, e.g. `draft-bhutton-json-schema-01`), not a final RFC | verified | `jsonschema_core.html` (cached draft) + parent anchor |
| 32 | YAML and TOML have no standard, specification-level schema/validation language | verified | Absence in YAML 1.2.2 §10 (type resolution only) and TOML v1.0.0 (no schema section) |
| 33 | Python's `json.dumps` emits bare `NaN`/`Infinity` unless `allow_nan=False` | verified | Python stdlib `json` documentation (per parent anchor) |
| 34 | `yaml.load` with a permissive loader can construct arbitrary objects; `yaml.safe_load` restricts construction | verified (documented library behaviour) | PyYAML/SnakeYAML/Psych official documentation conventions; exact PyYAML default-loader version boundary flagged |
| 35 | The exact version at which PyYAML's `yaml.load` default changed, and per-parser alias-expansion limits, are implementation-specific and unverified here | flagged | Not confirmed at a primary source |
| 36 | YAML duplicate keys are an error (YAML data-model uniqueness requirement) | flagged | Could not grep the exact 1.2.2 sentence in the cached HTML |
| 37 | Tabs are invalid for indentation in YAML block context | verified | YAML 1.2.2 production rules (block indentation uses space) |
| 38 | JSON/YAML/TOML lenient parser switches (`ALLOW_COMMENTS`, `ALLOW_TRAILING_COMMA`, JSON5 mode) change the grammar and are non-standard | verified | RFC 8259 grammar + JSON5 abstract (non-standard) |

---

## 12. What Could Not Be Verified

| # | Item | Why it matters | Status / best available |
|---|---|---|---|
| 1 | **TOML v1.1.0's ratification status.** The toml.io page carries a 2025-12-18 date, but the page did not state whether this is a settled, community-ratified release or a published revision still subject to change | Determines whether it is safe to write v1.1.0-only features | **[verify]** — write v1.0.0 (which `tomllib` and most libraries implement) until the status is confirmed |
| 2 | **Exact BOM wording in RFC 8259 §8.1** ("MUST NOT add a byte order mark" and the permission to ignore one) | Affects the encoding gate in §9.4 | **[verify]** — my grep over the cached RFC matched `byte order mark` but I did not capture the sentence; the UTF-8 MUST in §8.1 *is* verified |
| 3 | **Leading BOM acceptance per YAML/TOML parser** | Determines whether a BOM is a hard failure or silently tolerated | **[verify]** — implementation-dependent; not asserted at a spec source |
| 4 | **The exact YAML 1.2.2 sentence declaring duplicate keys an error** | §3.4 states it as a spec rule | **[verify]** — the data-model uniqueness requirement is well established, but the precise 1.2.2 wording was not extracted from the cached HTML |
| 5 | **Per-parser alias/anchor expansion limits (YAML bombs)** | Determines the real exposure of §6.2 | **[verify]** — the specification mandates no numeric budget; limits vary by library and version and were not confirmed |
| 6 | **PyYAML's `yaml.load` default-loader version boundary** (when the default moved to a safe-ish loader) | Affects the upgrade advice in §6.1 | **[verify]** — documented behaviour is clear; the version boundary was not confirmed at a primary source |
| 7 | **Flow-style trailing commas in YAML 1.2.2** | The capability matrix marks them **[verify]** | **[verify]** — not confirmed in the cached spec text |
| 8 | **The `toml-test` conformance-suite pass claims of individual TOML libraries** | §4.3 describes the ecosystem's conformance story | **[verify]** — the suite exists in the ecosystem; per-library pass claims were not opened |
| 9 | **JSON5's full feature list** (whether every feature I named beyond the abstract's "features from ECMAScript 5.1" is normative) | §3.5 table | **[verify]** — the abstract is verified; the per-feature list is conventional description |
| 10 | **Whether `application/json` and the other media types are currently listed in the IANA registry** (as opposed to being registered by their respective RFCs) | Only relevant if you need the registry entry itself | **[verify]** — the registering RFCs were verified; the live registry was not fetched |

Nothing in this guide was **rejected** as false; the "rejected" status in §11 is reserved for folklore claims that the specifications contradict, of which two are named explicitly in §3.8 (JSON "forbids" duplicate keys; JSON5 is "a standard"). Both are marked **rejected** in the sense that the sources do not support them.

---

## 13. Glossary

| Term | Meaning |
|---|---|
| **Alias** | In YAML, a reference (`*name`) that re-uses a node previously marked with an anchor (`&name`). The mechanism behind YAML's graph serialisation and its expansion attacks |
| **Anchor** | In YAML, the `&name` marker that names a node for later aliasing |
| **Array of tables** | TOML's `[[header]]` form, which appends successive table elements to an array; references resolve to the most recently defined element |
| **Block vs flow style** | YAML's two presentation styles: indentation-based (block) and bracket/brace-based (flow, JSON-like) |
| **BOM** | Byte Order Mark; a leading `EF BB BF` sequence. A frequent cause of "the parser rejected a file that looks fine" |
| **Core schema** | YAML's recommended default schema; extends the JSON schema with human-readable forms of null/bool/int/float |
| **Dialect** | A grammar variant presented as the format (JSON5, JSONC, lenient-flag modes) that is **not** the standard |
| **Differential (parser)** | Two parsers disagreeing about the same bytes; a security concern when the disagreement spans a trust boundary |
| **Failsafe schema** | YAML's minimal schema: `map`, `seq`, `str` only; guaranteed to work with any YAML document |
| **JSON Schema** | The schema/validation vocabulary for JSON; IETF Internet-Draft lineage, with 2020-12 the widely implemented revision |
| **NDJSON / JSON Lines** | A framing convention: one JSON value per line. Not part of RFC 8259 |
| **Norway problem** | The YAML 1.1 resolution of unquoted `NO` to the boolean `false`, and its cousins (`ON`, `Y`, `N`) |
| **Prototype pollution** | In JavaScript, corruption of `Object.prototype` (or a class prototype) via `__proto__`/`constructor` keys written by a naive deep merge |
| **Scalar / node / tag** | YAML's data model: a scalar or collection is a node; a tag gives it a type (`tag:yaml.org,2002:str`) |
| **Structured syntax suffix** | A MIME convention (`+yaml`, `+json`) allowing a format to be composed with an application type, e.g. `application/vnd.acme+json` |
| **Table (TOML)** | TOML's mapping construct, declared with `[header]`; may not be redefined |
| **Unsafe loader** | A deserialiser that constructs application objects named by the document. In YAML, the class of loader that turns a config file into an execution surface |

---

## 14. Primary Sources and Further Reading

| Source | What it is | Used for here |
|---|---|---|
| **RFC 8259**, *The JavaScript Object Notation (JSON) Data Interchange Format* (Dec 2017) | The JSON Internet Standard | Grammar, numbers, duplicate keys, UTF-8, interop warnings |
| **STD 90** (RFC Editor) | Confirms RFC 8259's Internet Standard status | §1.1 |
| **ECMA-404, 2nd Edition** (Dec 2017), *The JSON Data Interchange Syntax* | Ecma's language-independent JSON grammar | §1.1 |
| **json.org** | The canonical informal JSON grammar and history | §1.1 origin |
| **RFC 9512**, *YAML Media Type* (Feb 2024), Informational | IANA registration of `application/yaml` and `+yaml` | §1.2 |
| **YAML 1.2.2 Specification** (2021-10-01) | The current YAML specification, including recommended schemas | §1.2, §2.2, §3.1 |
| **YAML 1.1 Specification** | The source of the boolean/integer resolution differences | §3.1, §3.2 |
| **TOML v1.0.0 Specification** (2021-01-11) | The TOML standard the ecosystem implements | §1.3, §2.3, §3.6 |
| **TOML v1.1.0 Specification** (2025-12-18) | The current published TOML revision; status unconfirmed | §1.3, §10 |
| **JSON Schema core (2020-12 series, IETF Internet-Draft)** | The schema/validation vocabulary for JSON | §5.1 |
| **JSON5 specification** | A proposed (non-standard) JSON extension | §3.5 |
| **HJSON** | A non-standard human-friendly JSON dialect | §3.5 |
| **IANA media-type registries** | Authoritative media-type entries | §1.4 (entries registered by the RFCs above) |

**Further reading in this repository:** [configuration_management_languages_guide.md](configuration_management_languages_guide.md) (the languages and tools that consume these formats) · [schema_evolution_data_drift_guide.md](schema_evolution_data_drift_guide.md) (schemas over time) · [api_governance_guide.md](api_governance_guide.md) (API/JSON contracts) · [power_platform_cicd_guide.md](power_platform_cicd_guide.md) (YAML pipelines in CI/CD) · [data/data_pipeline_versioning.md](data/data_pipeline_versioning.md) · [ai_llm/agent_scaffolding_guide.md](ai_llm/agent_scaffolding_guide.md) and [ai_llm/agentic_solution_artifacts_guide.md](ai_llm/agentic_solution_artifacts_guide.md) (LLM tool-calling and artifact gates).

---

*This guide was authored by Jack Liu Shurui, Solution Architect at Cymbal Bank. It is an independent, format-level technical reference: the specifications cited are public and primary, the Cymbal Bank worked example is explicitly illustrative and fictional, and any error in the reading of a specification is the author's own.*

For questions, corrections, or contributions, please open an issue or PR at: https://github.com/jackliusr/research
