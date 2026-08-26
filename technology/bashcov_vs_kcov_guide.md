# Bash Code-Coverage Tools — bashcov vs kcov Head-to-Head

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Technology Research — shell-automation testing series; the dedicated **head-to-head on the two bash code-coverage tools**: bashcov (the Ruby gem that traces via `BASH_XTRACEFD` and reports SimpleCov-style) and kcov (the C++ coverage engine, born from Bcov, that covers compiled programs, Python and Bash via ptrace). Where the umbrella guide asks *"how do I measure coverage of my shell tests?"* and the shell-testing comparison merely names the tools in its coverage row, this guide answers *"how do bashcov and kcov actually differ, and which one should a bank's automation estate use?"*
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** the GitHub repositories and READMEs of bashcov (infertux/bashcov — the Ruby gem, its `lib/bashcov/xtrace.rb`, `lib/bashcov/runner.rb` and `USAGE.md`) and kcov (SimonKagstrom/kcov — the README, the man page `doc/kcov.1`, `src/engines/ptrace.cc`, `src/engines/bash-helper.sh`, `src/engines/ptrace_sys.hh`, the repo layout); the kcov web page (simonkagstrom.github.io/kcov); the sibling guides in this repository (see "How to Read This Guide").
> **Last Updated:** August 2026

---

## Table of Contents

1. [Overview — The Two Tools](#1-overview--the-two-tools)
   - 1.1 The Tools
   - 1.2 The Coverage Row in the Framework Comparison
   - 1.3 The Overview Table
   - 1.4 Verification Notes
2. [Profiles](#2-profiles)
   - 2.1 bashcov — The Ruby Gem That Listens to Bash's Own Trace
   - 2.2 kcov — The C++ Engine That Watches Through ptrace
   - 2.3 The Profile Table
3. [Coverage Dimensions](#3-coverage-dimensions)
   - 3.1 Line Coverage
   - 3.2 Function Coverage
   - 3.3 Branch Coverage
   - 3.4 The Dimensions Table
4. [Outputs](#4-outputs)
   - 4.1 bashcov — SimpleCov HTML and Resultset JSON
   - 4.2 kcov — LCOV-Style HTML, Cobertura XML, Generic JSON
   - 4.3 The Output Table
5. [Integration](#5-integration)
   - 5.1 CI Integration
   - 5.2 Codecov and Coveralls
   - 5.3 The Integration Table
6. [Pros and Cons](#6-pros-and-cons)
   - 6.1 bashcov
   - 6.2 kcov
   - 6.3 The Pros-Cons Table
7. [Decision — When to Use Which](#7-decision--when-to-use-which)
   - 7.1 The Decision
   - 7.2 The Decision Table
   - 7.3 What If the Context Changes?
8. [Worked Example — One Script, Both Tools](#8-worked-example--one-script-both-tools)
   - 8.1 The Scenario: The Familiar MFT Payment-File Script
   - 8.2 The Coverage Runs: bashcov
   - 8.3 The Coverage Runs: kcov
   - 8.4 The Two Reports Compared
   - 8.5 The Lessons
9. [Summary — One Page](#9-summary--one-page)
10. [Glossary](#10-glossary)
11. [Claims Status, References and Disclaimer](#11-claims-status-references-and-disclaimer)

### How to Read This Guide

This is the dedicated **coverage-tool head-to-head**, in the same genre as the sibling head-to-heads ([huggingface_vs_csghub_guide.md](huggingface_vs_csghub_guide.md) and the shell-framework comparison): profiles, a comparison matrix, a worked example, and a verdict. It deliberately **deepens the coverage dimension** — it does not re-derive the testing-framework content, which lives in the siblings:

- **The framework comparison (THE sibling)** — [shell_testing_frameworks_comparison_guide.md](shell_testing_frameworks_comparison_guide.md) is the 1,084-line head-to-head on shUnit2/Bats/bash_unit/assert.sh/bashaspec/ShellSpec. Its feature matrix carries the **coverage row** that names this guide's subjects — shUnit2 ❌, Bats 🟡 kcov/bashcov (external), bash_unit ❌, assert.sh ❌, bashaspec ❌, ShellSpec ✅ kcov integrated (§1.3 below quotes the row in full). Its §7 worked example — the `validate_payment_file.sh` MFT payment-file script and its six-test suite — is **the same suite this guide covers with both tools** (§8). Cross-reference it for the harness side of "who runs the tests"; this guide is about "who measures what the tests touched".
- **The umbrella (THE sibling)** — [bash_script_testing_automation.md](bash_script_testing_automation.md) is the 547-line umbrella covering frameworks, linting, mocking, CI/CD, **coverage (kcov)** and project structure. Its §7.1 Test Coverage is the two-command kcov recipe (`sudo apt install kcov`; `kcov coverage/ bats tests/`), and its GitHub Actions workflow snippet (§6) runs `kcov --exclude-pattern=/usr/lib coverage bats tests/` and uploads the artifact — the exact pattern §8 of this guide generalises. It also names the limitation set this guide verifies in detail: no per-test tracking, no delta reporting, performance overhead.
- **The Bats angle (THE sibling)** — [bats_core_guide.md](bats_core_guide.md) covers Bats' own coverage limits (§7) — Bats has **no native coverage**; coverage comes from exactly the two tools compared here. Cross-reference it for the Bats-native-limits side of the argument.
- **The TDD discipline** — the dev-tooling cluster's [test-driven-development_guide.md](test-driven-development_guide.md): coverage tools are the *measuring stick* of the RED-GREEN-REFACTOR loop — they tell you whether the failing test you wrote actually drives the line you meant to fix. §8's worked example applies that discipline: the suite is unchanged, only the measurement is new.
- **The banking context** — the MFT/payment-file scenario in §8 deliberately mirrors the transfer-automation scripts guarded by shell tests in the bank series ([../banking/nets_software_systems_guide.md](../banking/nets_software_systems_guide.md) and siblings): payment-format validation, currency/amount checks, checksums, file-transfer pre-checks. [../management/business_case_development_guide.md](../management/business_case_development_guide.md) supplies the adoption-cost lens applied in §7.3 (a Ruby runtime vs a compiled C++ binary is a real dependency decision for a change-controlled estate).

---

## 1. Overview — The Two Tools

### 1.1 The Tools

For a long time, measuring "how much of my shell script did my tests actually execute" was the awkward gap in shell testing: the frameworks (shUnit2, Bats, bash_unit — see the framework comparison) are harnesses that run tests and report pass/fail, but none of them measures coverage. Two tools filled that gap from opposite ends of the implementation spectrum:

**bashcov** — *"a code coverage analysis tool for Bash"* (Verified — README, infertux/bashcov). It is a **Ruby gem** (`gem install bashcov`, MIT license, created in **2012** and maintained by Cédric Félizard (infertux) for over a decade — Verified — README). Its trick: it does **not** instrument your script. Instead it launches your script under Bash with `xtrace` (the `set -x` debug mode) enabled, points the trace stream at itself through the **`BASH_XTRACEFD`** file descriptor, and **parses the trace output** — each trace line carries `LINENO`, `BASH_SOURCE`, `PWD` and `OLDPWD`, so bashcov can count how many times every line of every executed script ran. The counts are then handed to **SimpleCov**, the de-facto Ruby coverage library, which renders the familiar SimpleCov-style HTML report and merges results across multiple test-suite runs automatically. It is **Bash-only by design**: no compiled programs, no Python — if it doesn't run under Bash, bashcov doesn't see it.

**kcov** — *"Code coverage tool for compiled programs, Python and Bash which uses debugging information to collect and report data without special compilation options"* (Verified — repo description, SimonKagstrom/kcov). It is a **C++ code-coverage engine**, originally a fork of **Bcov** by Thomas Neumann (Verified — README, kcov web page), written by **Simon Kagstrom**, GPL-2.0, with **827 stars / 127 forks / 2,202 commits** and **v43** released **July 23, 2024** (Verified — GitHub repo page, Aug 2026). For compiled ELF/Mach-O binaries it reads **DWARF debugging information** to find every basic block, then **traces the program with ptrace**, planting a breakpoint on each block and recording every block that traps — no recompilation, no special compiler switches. For scripts it has dedicated engines: a **Python engine** and a **Bash engine** (which itself offers two collection methods — the `PS4` trace-variable method or the `DEBUG` trap — mirroring bashcov's mechanism from inside a compiled binary). Its scope is therefore the widest of any tool in this series: **compiled languages, Python and Bash** in one invocation, on FreeBSD, Linux and macOS.

One sentence each: **bashcov is a Ruby trace-parser that rides Bash's own debug output and reports through SimpleCov; kcov is a native ptrace-based coverage engine that treats Bash as just one more of the three language families it can cover.**

### 1.2 The Coverage Row in the Framework Comparison

The sibling comparison's feature matrix resolves the question "who provides coverage for each harness?" in a single row (Verified — shell_testing_frameworks_comparison_guide.md, §3.3, line 363):

```
| **Coverage** | ❌ | 🟡 kcov/bashcov (external) | ❌ | ❌ | ❌ | ✅ kcov integrated |
|              shUnit2    Bats                           bash_unit   assert.sh   bashaspec   ShellSpec
```

Reading it against the two tools compared here:

- **Bats** gets a 🟡 because coverage is **external** — you wrap the Bats run with bashcov (`bashcov -- bats tests/`) or kcov (`kcov coverage/ bats tests/`). That wrapper pattern is exactly what §8 demonstrates.
- **ShellSpec** gets a ✅ because it ships **kcov integration built in** — ShellSpec runs kcov for you (cross-ref the framework comparison §2.6); it does not bundle bashcov, which is itself a signal about which tool the ecosystem has standardised on for Bash.
- **shUnit2, bash_unit, assert.sh, bashaspec** have no coverage story of their own — with them, the *only* path to coverage is wrapping the whole suite run with one of these two tools (or ShellSpec's kcov mode).
- The 🟡/external status of both tools is the reason this guide exists: the coverage tool is a **wrapper around the test runner**, not a feature of it — so the choice of coverage tool is independent of the choice of harness, and can be made once for the whole estate.

### 1.3 The Overview Table

| Aspect | bashcov | kcov |
|---|---|---|
| **What it is** | Ruby gem — "code coverage analysis tool for Bash" | C++ coverage engine — "code coverage tool for compiled programs, Python and Bash" |
| **Scope** | Bash scripts only | Compiled ELF/Mach-O binaries, Python, Bash/sh (FreeBSD, Linux, macOS) |
| **Mechanism** | Bash `xtrace` + `BASH_XTRACEFD` trace stream parsed in Ruby | ptrace engine + software breakpoints on DWARF-discovered basic blocks; PS4/DEBUG-trap engines for Bash |
| **Reporting** | SimpleCov-style HTML (+ SimpleCov resultset JSON) | LCOV-style HTML, Cobertura XML, generic JSON |
| **Language of implementation** | Ruby (gem; depends on SimpleCov) | C++ (~54% of repo), with Python/JS tooling |
| **License** | MIT | GPL-2.0 |
| **Origin** | Created 2012, Cédric Félizard (infertux); repo now at infertux/bashcov | Fork of Bcov (Thomas Neumann), by Simon Kagstrom; repo at SimonKagstrom/kcov |
| **Activity (Aug 2026)** | single-maintainer project, decade-old (adoption beyond this not fully verifiable this session) | v43 (Jul 2024), 827 stars, 2,202 commits, 71 contributors, 16 releases |
| **CI/upload story** | via the SimpleCov ecosystem (`.simplecov`, resultset JSON, community formatters) | first-class: doc/coveralls.md, doc/codecov.md, doc/jenkins.md, doc/sonarqube.md, doc/github.md, doc/gitlab.md, `--coveralls-id` |
| **Cross-ref (this series)** | framework comparison coverage row; umbrella §7.1 | framework comparison coverage row; umbrella §7.1 + §6 CI snippet; bats_core_guide §7; ShellSpec's built-in integration |

### 1.4 Verification Notes

Verified against primary sources in this session: bashcov's README, `lib/bashcov/runner.rb`, `lib/bashcov/xtrace.rb` and `USAGE.md` (the `BASH_XTRACEFD` + `PS4` mechanism, the SimpleCov dependency, the 2012 origin, the MIT license); kcov's README, `doc/kcov.1` man page (options, output formats, `--bash-parser`, `--bash-method`), `src/engines/ptrace.cc` (breakpoint patching), `src/engines/bash-helper.sh` (its own `PS4` + `set -x` recipe), `src/engines/ptrace_sys.hh`, the repo page (stars, license, languages, releases), and the kcov web page (Bcov lineage, feature list). Two claims were **flagged** rather than confirmed: (1) the common "ptrace/**debug-register**" description of kcov — the current source shows **software breakpoints (instruction patching), not hardware debug registers** (§2.2); (2) **bashcov's adoption numbers** (gem downloads, stars) — the web-search backend was unavailable this session, so only repo-verified facts are stated. Anything else asserted below is sourced or marked unverified.

### 1.5 The Series Map

Where this guide sits in the shell-automation series — and what each sibling contributes to the coverage question:

| Guide | Role in the coverage story |
|---|---|
| [shell_testing_frameworks_comparison_guide.md](shell_testing_frameworks_comparison_guide.md) | names the tools in its coverage row; supplies the worked-example script and six-test suite that §8 measures |
| [bash_script_testing_automation.md](bash_script_testing_automation.md) | the umbrella — coverage as one practice among lint/format/mock/CI; the kcov GitHub Actions recipe |
| [bats_core_guide.md](bats_core_guide.md) | Bats has no native coverage — the external-wrapper story for the series' most-used harness |
| [test-driven-development_guide.md](test-driven-development_guide.md) | the discipline the measurement serves: red lines drive the next RED test |
| [../banking/nets_software_systems_guide.md](../banking/nets_software_systems_guide.md) + bank series | the MFT/payment-file domain the worked example mirrors |
| [../management/business_case_development_guide.md](../management/business_case_development_guide.md) | the adoption-cost lens for choosing a Ruby runtime vs a native binary |
| **this guide** | the dedicated bashcov-vs-kcov head-to-head: profiles, dimensions, outputs, integration, decision, measured worked example |

---

## 2. Profiles

### 2.1 bashcov — The Ruby Gem That Listens to Bash's Own Trace

**Identity:** "Bashcov is a **code coverage analysis tool for Bash**" (Verified — README). Ruby gem, MIT, `gem install bashcov`; created in **2012** (Verified — README links the "created in 2012" claim to commit `f65e65e`), maintained since by **Cédric Félizard** (infertux), who also maintains the build on GitHub Actions and GitLab CI and asks for Liberapay sponsorship — the profile of a long-lived, single-maintainer, community-funded project. The repo lives at `infertux/bashcov`; the older `bashcov/bashcov` org URL now returns 404 (Verified — the project moved to the maintainer's account).

**The mechanism — `BASH_XTRACEFD` and the `PS4` contract (Verified — `lib/bashcov/runner.rb`, `lib/bashcov/xtrace.rb`, `USAGE.md`).** bashcov runs your command with Bash's debug mode switched on, and *reads the debug trace*:

1. It adds `xtrace` to the inherited `SHELLOPTS` (`with_xtrace_flag` in runner.rb) — the equivalent of `set -x`.
2. It builds a **`PS4`** prompt string that expands, on every command Bash executes, to a random-UUID-delimited record of `+BASHCOV>` + `${LINENO-}` + `${BASH_SOURCE-}` + `${PWD-}` + `${OLDPWD-}` (the UUID delimiter prevents collisions with script content).
3. It opens an `IO.pipe` and sets **`BASH_XTRACEFD`** to the pipe's write end, so the xtrace stream — normally stderr — flows into a Ruby-managed pipe instead. (When run as root, Bash 4.4+ refuses to inherit `PS4` from the environment, so bashcov writes a `BASH_ENV` temp file exporting `PS4` — and warns that root is not recommended.)
4. A Ruby thread reads the pipe, splits each trace record on the delimiter, and increments a per-file, per-line hit counter (`@files[script][index] += 1` in xtrace.rb). `cd`/`pushd`/`popd` are tracked through **PWD/OLDPWD stacks** because `BASH_SOURCE` alone is not reliably absolute — USAGE.md documents this as the "gory details": a reverse scan of the PWD stack finds the first `$PWD/$BASH_SOURCE` combo that names an existing file, a heuristic that is "susceptible to false positives" but performs well in practice.
5. A lexer (`lib/bashcov/lexer.rb`) then marks which lines are *relevant* (executable) so blank lines and comments are not counted against you, and SimpleCov filters decide which files enter the report.

Because the trace records every line hit, **bashcov's core unit is the line hit**. It needs no parsing of your script to *collect* data — it only parses afterwards to know which lines count.

**The PS4 contract, in code.** The heart of the mechanism is the `PS4` construction in `lib/bashcov/xtrace.rb` (Verified — quoted from source):

```ruby
PREFIX = "BASHCOV>"                       # marker that identifies bashcov's trace lines
FIELDS = %w[${LINENO-} ${BASH_SOURCE-} ${PWD-} ${OLDPWD-}]
def delimiter; @delimiter ||= SecureRandom.uuid; end   # random field separator
def make_ps4(*fields)                     # "+BASHCOV><uuid>$LINENO<uuid>$BASH_SOURCE..."
  fields.reduce(DEPTH_CHAR + PREFIX) { |memo, field| memo + delimiter + field } + delimiter
end
```

and the runner wires it to Bash's own trace plumbing (`lib/bashcov/runner.rb`, Verified):

```ruby
env["BASH_XTRACEFD"] = fd.to_s            # xtrace stream -> Ruby-managed pipe
ENV["SHELLOPTS"] = (existing_flags | ["xtrace"]).join(":")   # set -x, inherited
# then: Process.spawn env, *@command ... ; Thread.new { @xtrace.read }
```

The `-` suffix on each variable (`${LINENO-}`) is deliberate: if a variable fails to expand on a line, the trace still parses, and the parser skips records where `LINENO` came back literal (Verified — xtrace.rb `parse_hit!`). The random UUID delimiter exists so a script containing the delimiter string can't corrupt the stream — and bashcov's own warning says exactly that: parse errors surface as "Aborting early; coverage report will be incomplete" (Verified — runner.rb rescue path).

**The SimpleCov-style reporting.** "It uses the **SimpleCov** coverage library to generate HTML reports. SimpleCov gets installed automatically when you install Bashcov" (Verified — README). Output lands in `./coverage/` with `coverage/index.html` as the entry point — the classic SimpleCov layout (file list with green/red percentages, per-file line views with hit counts in the gutter). SimpleCov's `.simplecov` config file in the project root is honoured, `SimpleCov.command_name` is controlled via `--command-name` or `BASHCOV_COMMAND_NAME`, and — the feature the README leads with — **"Bashcov automatically takes care of this by caching and merging results when generating reports"** across shUnit2, Bats, bash_unit, assert.sh, bashaspec, etc.: run the whole test suite under bashcov once per framework, and the report merges them into one picture of the blank spots. (RVM is cited as a real-world demo target.)

**Usage (Verified — README):**

```bash
gem install bashcov
bashcov ./script.sh
bashcov --skip-uncovered ./script.sh
bashcov -- ./script.sh --some --flags          # -- separates bashcov's own flags
bashcov --skip-uncovered -- ./script.sh --some --flags
```

Flags (Verified — `lib/bashcov.rb`): `--skip-uncovered` (don't report files never executed), `--mute` (suppress script output), `--bash-path PATH`, `--root PATH` (project root), `--command-name NAME`; env vars `BASHCOV_COMMAND_NAME`, `BASHCOV_BASH_PATH`.

**The constraints.** Bash-only: a script run by `/bin/sh` or another shell is outside its model (it tracks whatever `BASH_SOURCE` says, but the tracing contract is Bash's). It needs Ruby on the machine. It is a wrapper — "`./script.sh` can be a mere Bash script or typically your CI script" — so it traces *everything* the command spawns under Bash, which is exactly why wrapping a whole test suite works.

### 2.2 kcov — The C++ Engine That Watches Through ptrace

**Identity:** "Kcov is a FreeBSD/Linux/Mac OS code coverage tester for **compiled languages, Python and Bash**" (Verified — README). Written by **Simon Kagstrom** (simon.kagstrom@gmail.com); "Kcov was originally a **fork of Bcov** [by Thomas Neumann], but has since evolved to support a large feature set in addition to that of Bcov" (Verified — README, kcov web page). GPL-2.0 (the repo moved to SPDX license expressions in 2024 — Verified — repo). C++ is the implementation language (~53.8% of the repo, with JavaScript 14%, Python 11.5%, C 9.8% — Verified — repo languages panel). **v43** (July 23, 2024) is the latest of 16 releases; 2,202 commits, 71 contributors, 827 stars, 24 watchers (Verified — repo page, Aug 2026). Packaging: `apt install kcov` on Debian/Ubuntu (the umbrella's recipe), distro man page (written by Michael Tautschnig for Debian), an official Docker image `kcov/kcov` for releases since v31 (Verified — README), and GitHub Actions builds of tagged releases.

**The mechanism — DWARF + ptrace + software breakpoints (Verified — `src/engines/ptrace.cc`, `ptrace_sys.hh`; man page).** For compiled binaries:

1. kcov parses the executable's **DWARF debugging information** to map every source line to its machine addresses and derive the **basic blocks** — no `--coverage`-style compiler flags needed; plain `-g` debug info suffices (hence "without special compilation options"). The `--verify` flag even double-checks that breakpoints land on instruction boundaries, "to catch problems where the compiler generates bad DWARF data" (Verified — man page).
2. It forks the target, disables ASLR in the child, calls `PTRACE_TRACEME`, and pins both processes to the same CPU (`disable_aslr()`, `trace_me()`, `tie_process_to_cpu()` — Verified — ptrace_sys.hh).
3. It **plants a breakpoint at the start of every basic block** by *patching the instruction stream*: `pokeWord()` writes `0xcc` (INT3) on x86-64, `0x7fe00008` (`tw`) on PowerPC, `0x00100073` (`ebreak`) on RISC-V, an undefined instruction on ARM/AArch64 (Verified — `arch_setupBreakpoint()` in ptrace.cc). Every trap is a "this block executed" event; the block is then single-stepped and the original instruction restored (`singleStep()`, `skipInstruction()`, `arch_clearBreakpoint()`).
4. Forks and new threads are followed automatically (`follow_fork`, `follow_child`), and shared libraries — including those opened via `dlopen` — are covered transparently (Verified — web page feature list; `--skip-solibs` exists to turn that off for speed).

**A verification flag:** the widely-repeated "ptrace/**debug-register** mechanism" phrasing is **not accurate for the current implementation**. The engine source shows software breakpoints achieved by **instruction patching** (the `0xcc`/`ebreak`/`tw` table below), not hardware debug registers (DR0–DR7). kcov *is* genuinely ptrace-based; the debug-register claim is the unverified part and is corrected here on the authority of the source. (Hardware registers would have been a plausible design — that is likely how the phrasing entered secondary summaries — but the code patched instructions instead.)

**The breakpoint table, per architecture (Verified — `arch_setupBreakpoint()` in `src/engines/ptrace.cc`):**

| Architecture | Patched instruction | Purpose |
|---|---|---|
| x86 / x86-64 | `0xcc` (INT3) | classic software breakpoint opcode |
| PowerPC | `0x7fe00008` (`tw`) | trap word |
| ARM / AArch64 | undefined instruction (`0xfedeffe7` / `0xd4200000`) | SIGILL trap (kcov maps SIGILL to a breakpoint event) |
| RISC-V | `0x00100073` (`ebreak`) | explicit breakpoint instruction |
| LoongArch | `break 0x4` (`0x002a0004`) | breakpoint |
| SPARC64 | `ta 0x01` (`0x91d02001`) | trap always |

The trap is handled by the ptrace loop in `waitEvent()`: a `SIGTRAP`/`SIGILL`/`SIGSTOP` stop at a known breakpoint address is treated as a "this basic block executed" event, the block is single-stepped, and the original instruction is restored with `arch_clearBreakpoint()` before execution continues (Verified — ptrace.cc `continueExecution()`/`clearBreakpoint()`). `--skip-solibs` skips shared libraries for speed; `--verify` re-checks that breakpoints sit on instruction boundaries.

**The script engines — Bash and Python.** kcov did not start as a Bash tool; the Bash engine came later and sits beside the ptrace engine in `src/engines/` (Verified — repo layout):

- `bash-engine.cc` + **`bash-helper.sh`** — the helper is four lines: `PS4='kcov@${BASH_SOURCE}@${LINENO}@'` then `set -x` — i.e. kcov's Bash mode uses **the same PS4/xtrace idea as bashcov**, implemented in C++ with its own trace-fd plumbing (`bash-tracefd-cloexec.c`, `--bash-tracefd-closeexec`). The man page exposes the choice: **`--bash-method` PS4 | DEBUG** — the DEBUG alternative uses Bash's `DEBUG` trap (`bash-helper-debug-trap.sh`) instead of xtrace.
- `--bash-parser=PARSER` sets which bash binary parses scripts (default `bash`); `--bash-handle-sh-invocation` LD_PRELOADs an execve redirector so `/bin/sh` scripts get executed as bash ("does not work well on some systems", so it defaults off — Verified — man page); `--bash-parse-files-in-dir` and `--bash-dont-parse-binary-dir` control which sibling scripts get parsed into the report.
- Python gets the same treatment: `python-engine.cc` + `python-helper.py`, `--python-parser`.

So the scope line — **"compiled programs, Python and Bash"** — is not marketing: it is three real engines in one binary (plus a Mach-O engine for macOS and a kernel/system-mode engine for full-system instrumentation of a sysroot — Verified — repo layout, man page `--system-record`).

**Usage (Verified — README, man page):**

```bash
kcov /path/to/outdir executable [args]            # HTML written continuously
kcov --include-pattern=src/ coverage/ bats tests/  # the umbrella's CI pattern
kcov --exclude-pattern=/usr/include --include-pattern=part/of/path,other/path /path/to/outdir executable
kcov --merge /tmp/merged /tmp/kcov-output1 /tmp/kcov-output2   # merge prior runs
kcov --collect-only ... && kcov --report-only ...              # split collect/report
```

Other notable flags (Verified — man page): `--pid` (trace an existing PID), `--limits low,high` (25,75 default colour thresholds), `--exclude-line`/`--exclude-region` (mark lines as non-code), `--clean` (don't accumulate across runs), `--dump-summary`, `--coveralls-id`, `--cobertura-only`, `--output-interval MS` (default 5000), `--strip-path`/`--replace-src-path`.

**The behaviour.** kcov **accumulates** data from multiple runs and multiple programs into one output directory (a `[merged]` view shows the union — Verified — man page); the HTML is regenerated continuously so long-running programs can be watched "live" (Verified — web page; default write interval 5 s); lines that map to multiple instrumentation points (e.g. inlined functions) show as `1/3` in the gutter — one of three points executed (Verified — man page).

### 2.3 The Profile Table

| Dimension | bashcov | kcov |
|---|---|---|
| **Origin** | Ruby gem by Cédric Félizard (infertux), created 2012; repo infertux/bashcov | C++ engine by Simon Kagstrom, fork of Bcov (Thomas Neumann); repo SimonKagstrom/kcov |
| **Language / runtime** | Ruby gem; requires Ruby + SimpleCov | Compiled C++ (GPL-2.0); `apt install kcov`, Docker `kcov/kcov`, or cmake/ninja build |
| **Scope** | Bash scripts only | Compiled ELF/Mach-O binaries, Python, Bash/sh; FreeBSD, Linux, macOS |
| **Mechanism** | Bash `xtrace` via `SHELLOPTS`; `PS4` with `LINENO`/`BASH_SOURCE`/`PWD`/`OLDPWD`; trace captured through `BASH_XTRACEFD` pipe; parsed in Ruby | ptrace engine with software breakpoints (INT3/`tw`/`ebreak`) on DWARF basic blocks + single-step; separate Bash engine (`PS4` or `DEBUG` trap) and Python engine |
| **Collection granularity** | per-line hit counts | per-basic-block (compiled) / per-line (Bash, Python) |
| **Merging** | built-in via SimpleCov caching/merging across suite runs | built-in accumulation across runs/programs; `--merge` for prior runs; `[merged]` view |
| **License** | MIT | GPL-2.0 |
| **Maturity signal** | single maintainer, decade+ old, SimpleCov ecosystem | v43 (Jul 2024), 827★, 71 contributors, distro-packaged, ShellSpec integration |

---

## 3. Coverage Dimensions

The classic coverage hierarchy is **line → function → branch**: did this line run, did this function ever get called, did both sides of this `if` run? For shell scripts the honest starting point is that **both tools are line-coverage tools at heart** — their deepest verified dimension is per-line hit counts. The finer dimensions are partial, implicit, or absent, and this section says exactly which.

### 3.1 Line Coverage

The primary dimension for both — and the only one bashcov reports:

- **bashcov:** the xtrace stream yields a hit count per `LINENO` per `BASH_SOURCE` (Verified — xtrace.rb `parse_hit!`), the lexer then marks which lines are relevant (executable vs blank/comment), and SimpleCov renders covered/uncovered lines with counts. This is textbook line coverage.
- **kcov:** for Bash/Python, the same per-line model (the `PS4` record is `kcov@${BASH_SOURCE}@${LINENO}@` — Verified — bash-helper.sh). For compiled code it is finer than lines: **basic-block coverage**, so a line with several blocks (a `goto`, an inlined call) can show `1/3` executed (Verified — man page). Block-level is *at least* as good as line-level for spotting gaps — an unexecuted block inside a line is visible where a pure line tool would show the line as "covered" if any part of it ran.

### 3.2 Function Coverage

- **bashcov:** no function metric. SimpleCov's model is files → lines; there is no per-function percentage in the report. You *infer* function coverage by eye from red function bodies.
- **kcov:** no first-class per-function percentage in the HTML/JSON for scripts either (the generic JSON reports per-file percentages and line counts — Verified — man page). What kcov adds is the block-level view: a function that never ran is a contiguous red region, and a partially-run function shows exactly which of its blocks are dead. For compiled code, DWARF function boundaries are available in the model, but the shipped reports stay at file/line granularity.

Honest verdict: **function coverage is not a first-class metric in either tool.** Neither emits a "functions covered: 3/5" number for a Bash script. If you need that, the closest built-in signal is kcov's block-granular red regions; anything more requires post-processing the trace/JSON.

### 3.3 Branch Coverage

- **bashcov:** no branch dimension. A line inside an un-taken `else` simply shows zero hits, which is how you *spot* an uncovered branch — but the tool never counts branches, and it cannot distinguish "line never executed" from "line executed but this branch not taken" for multi-branch lines.
- **kcov:** no branch-percentage metric either, but its compiled-mode **basic-block model is branch-aware by construction**: every branch target is its own block with its own breakpoint, so an unexecuted block *is* an unexecuted branch edge. For Bash mode, both collection methods (PS4/DEBUG) reduce to line hits, so branch coverage is implicit exactly as with bashcov.
- **Neither tool** reports a "branches taken: N/M" figure for shell scripts — that claim, occasionally seen in secondary summaries, is **flagged unverified**. For a regulated shop, the practical consequence is the same as the umbrella's: coverage gaps show up as red lines, and the *meaning* of a red line ("this branch never tested") is read by a human, not computed by the tool.

### 3.4 The Dimensions Table

| Dimension | bashcov | kcov |
|---|---|---|
| **Line coverage** | ✅ first-class — per-line hit counts via xtrace; lexer marks executable lines | ✅ first-class — per-line for Bash/Python; per-basic-block for compiled code (finer than lines, `1/3` notation) |
| **Function coverage** | ❌ no metric (infer from red bodies) | 🟡 no explicit % for scripts; block-granular dead regions make function gaps visible |
| **Branch coverage** | ❌ no metric; uncovered branch bodies show as uncovered lines | 🟡 implicit for compiled code (block = branch edge); line-level only for Bash |
| **Per-test attribution** | ❌ merged whole-suite picture only | ❌ no per-test tracking (flag — umbrella §7.1 names this limitation) |
| **What a red line means** | line never executed across all wrapped suites | line (or block) never executed across all accumulated runs |
| **Coverage discipline tie-in** | RED-GREEN-REFACTOR measuring stick: a red line means no test drove it (cross-ref test-driven-development_guide.md) | same — and ShellSpec exposes it per-test via its kcov integration (cross-ref the framework comparison) |

### 3.5 What the Lab Measured

The worked example (§8) was executed for real, and it put numbers on every row of this section. The honest headline: **for Bash, both tools delivered line coverage and nothing finer, exactly as the table above claims** — the measured agreement is documented in §8.4. Two dimension-level observations from the run are worth pre-empting here:

- **The line dimension has artifacts.** Both tools reported the two `done < "$file"` loop-closing lines as uncovered in *every* run, because the tracing contract simply never emits a record for the `done` keyword. The line-coverage number (92.1% vs 91.9%) is therefore a slight *under*-statement of true coverage — and a coverage gate that treats every red line as a real gap will over-report. Red lines need reading (§8.5, lesson 4).
- **The branch dimension is where tests live.** The six-test suite's real untested rule — the unknown-record `*)` case — was the one genuinely red executable line, while the same-line branch arms the suite *also* never exercises (missing-HDR, count-mismatch) were invisible because their lines were hit from the happy path. The measured takeaway: with these tools, branch rules must be tested explicitly; the coverage report will not police them (verified in §8.2's line table).

---

## 4. Outputs

### 4.1 bashcov — SimpleCov HTML and Resultset JSON

- **HTML (SimpleCov-style):** `./coverage/index.html` — "It will create a directory named `./coverage/`, you may open `./coverage/index.html` to browse the coverage report" (Verified — README). The familiar SimpleCov layout: an index of files with green/red coverage percentages, and per-file pages with hit counts in the gutter and uncovered lines highlighted. SimpleCov's colour thresholds and groups are configurable via the project's `.simplecov` file (Verified — README, USAGE.md).
- **JSON:** SimpleCov's standard resultset file (`coverage/.resultset.json`) — the machine-readable side of the same data, and the basis of SimpleCov's cross-run merging (Verified — SimpleCov's documented merging behaviour, referenced by bashcov's USAGE.md: "SimpleCov's result-merging behavior").
- **LCOV:** *not produced natively* — flagged. bashcov ships the SimpleCov formatters; LCOV for bashcov means adding a community SimpleCov formatter (e.g. `simplecov-lcov`) through `.simplecov`. The task brief's "LCOV/JSON/HTML" for bashcov therefore resolves as: HTML ✅ native, JSON ✅ native (resultset), LCOV 🟡 via ecosystem formatter only.
- Demo reports are published for the bundled test app and for RVM (Verified — README links to infertux.github.io/bashcov).

### 4.2 kcov — LCOV-Style HTML, Cobertura XML, Generic JSON

- **HTML (LCOV-style):** the output directory "will contain **lcov-style HTML output** generated continuously while the application runs" (Verified — README). Layout: `out-dir/executable-name/index.html`, per-file pages with green/red lines, `1/3` block notation, a `[merged]` link for the union of all covered programs, low/high coverage colour thresholds via `--limits` (default 25/75 — Verified — man page). The HTML regenerates on a timer (`--output-interval`, default 5000 ms) so long runs are watchable live.
- **Cobertura XML:** written as `out-dir/executable-name/cobertura.xml` (Verified — man page) — the format Jenkins' Cobertura plugin consumes; `--cobertura-only` emits just that file, which the man page recommends for editors like VS Code coverage gutters.
- **JSON:** "Kcov generates a very generic json file which includes the overall percent covered for a single command and the count of lines instrumented and covered. It also includes a summary of each source file with a percentage and line counts" — written to `out-dir/executable-name/coverage.json` and aimed at GitLab CI's coverage report (Verified — man page).
- **LCOV note:** "LCOV-style HTML" is the README's own phrasing — kcov mimics the *look* of lcov's HTML report; it does not emit `.info` tracefiles natively. Same shape of caveat as bashcov's, from the other direction.

### 4.3 The Output Table

| Format | bashcov | kcov |
|---|---|---|
| **HTML** | ✅ SimpleCov-style, `coverage/index.html` | ✅ LCOV-style, `out-dir/<exec>/index.html`, continuously regenerated |
| **JSON** | ✅ SimpleCov resultset (`coverage/.resultset.json`, merged across runs) | ✅ generic `coverage.json` (overall %, lines instrumented/covered, per-file summary) |
| **LCOV** | 🟡 only via community SimpleCov formatter (e.g. simplecov-lcov) | 🟡 "lcov-style" HTML is the native form; no `.info` tracefiles |
| **Cobertura XML** | ❌ (not part of SimpleCov's core output) | ✅ `cobertura.xml` per executable, Jenkins-ready, `--cobertura-only` |
| **Summary to stdout** | 🟡 SimpleCov prints a coverage summary at exit | ✅ `--dump-summary` |
| **Merged view** | ✅ automatic cache+merge across wrapped suites | ✅ accumulation + `--merge` + `[merged]` union page |
| **CI-consumable out of the box** | 🟡 resultset JSON + ecosystem formatters | ✅ Cobertura (Jenkins), JSON (GitLab), `--coveralls-id` |

---

## 5. Integration

### 5.1 CI Integration

Both tools are **wrappers**: they execute the test command themselves, so any CI that can run a shell command can run them — the difference is how much of the CI story is built in vs assembled by you.

- **bashcov** is CI-agnostic: `bashcov -- bats tests/` in a GitHub Actions step, a GitLab job, or a Jenkins shell step all work identically, and its automatic merge means you can wrap *each* framework's suite and get one combined report. There is no bashcov-maintained CI action or uploader; the integration surface is the SimpleCov ecosystem (`.simplecov`, resultset JSON, formatters).
- **kcov** ships a documented integration suite: `doc/coveralls.md` (Travis + Coveralls), `doc/codecov.md` (Travis + Codecov), `doc/jenkins.md` (Cobertura plugin), `doc/sonarqube.md`, `doc/github.md`, `doc/gitlab.md` (the JSON report feeds GitLab's coverage widget), and `doc/vscode.md` for editor coverage gutters (Verified — README list). The umbrella's §6 workflow is the canonical GitHub Actions shape: `sudo apt install -y bats kcov` → `kcov --exclude-pattern=/usr/lib coverage bats tests/` → `actions/upload-artifact@v4` (Verified — bash_script_testing_automation.md §6). kcov dogfoods this: its own README carries Coveralls and Codecov badges and it has a Coverity Scan build (Verified — README).

### 5.2 Codecov and Coveralls

- **kcov → Coveralls:** `--coveralls-id=<id>` uploads directly ("Upload data to coveralls.io using secret repo_token or Travis CI service job ID"; the ID is treated as a repo_token if ≥ 32 chars — Verified — man page), with the Travis-era recipe in `doc/coveralls.md`. kcov's own repo uses exactly this badge.
- **kcov → Codecov:** documented Travis recipe in `doc/codecov.md` (Verified — README); the repo carries a Codecov badge. The upload path is the standard Codecov uploader fed from kcov's output (the Cobertura or JSON).
- **bashcov → Codecov/Coveralls:** no native uploader flag. The path is the SimpleCov ecosystem: resultset JSON / a SimpleCov formatter (LCOV for Codecov's uploader, the coveralls-ruby gem for Coveralls) wired through `.simplecov` and the CI step. Workable — and the *same* path Ruby projects already use — but it is assembly, not a flag. Flagged: neither bashcov's README nor USAGE.md documents a first-party Codecov/Coveralls recipe, so the specific formatter combination is presented as ecosystem practice, not as a bashcov-verified claim.
- **ShellSpec shortcut (cross-ref):** if the harness is ShellSpec, its built-in kcov integration runs the coverage step for you — the framework comparison's ✅ in the coverage row is the one place the upload story is someone else's problem entirely.

### 5.3 The Integration Table

| Integration | bashcov | kcov |
|---|---|---|
| **GitHub Actions** | 🟡 plain wrapper step (`bashcov -- bats tests/`); no first-party action | ✅ wrapper step + doc/github.md; umbrella §6 canonical workflow |
| **GitLab CI** | 🟡 generic job | ✅ doc/gitlab.md; `coverage.json` feeds the coverage widget |
| **Jenkins** | 🟡 generic job; resultset/HTML as artifacts | ✅ doc/jenkins.md + Cobertura XML via the Cobertura plugin |
| **SonarQube** | ❌ not documented | ✅ doc/sonarqube.md |
| **Coveralls** | 🟡 via SimpleCov/coveralls-ruby ecosystem | ✅ `--coveralls-id`, doc/coveralls.md, self-dogfooded |
| **Codecov** | 🟡 via SimpleCov formatter (e.g. LCOV) | ✅ doc/codecov.md, self-dogfooded |
| **VS Code coverage gutters** | 🟡 via formatter | ✅ doc/vscode.md + `--cobertura-only` |
| **Docker** | 🟡 needs a Ruby image | ✅ official `kcov/kcov` image (releases since v31) |

### 5.4 The CI YAML, Side by Side

The same GitHub Actions shape, both tools — the kcov side is the umbrella's §6 workflow verbatim (Verified), the bashcov side is the equivalent wrapper step:

```yaml
# bashcov flavour: needs Ruby, then wrap the suite
  coverage:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: sudo apt update && sudo apt install -y ruby-full
      - run: gem install bashcov
      - run: bashcov --skip-uncovered -- bats tests/        # SimpleCov HTML + .resultset.json
      - uses: actions/upload-artifact@v4
        with: { name: coverage, path: coverage/ }

# kcov flavour: apt binary, wrap the suite, filter the noise (the umbrella's recipe)
  coverage:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: sudo apt update && sudo apt install -y bats kcov
      - run: kcov --exclude-pattern=/usr/lib coverage bats tests/
      - uses: actions/upload-artifact@v4
        with: { name: coverage, path: coverage/ }
```

The pipeline differences that matter for a release gate: kcov's step produces `coverage.json`/`cobertura.xml` that Jenkins/GitLab/Codecov consumers can parse natively, while bashcov's step needs a formatter (`simplecov-lcov` via `.simplecov`) to emit a machine-checkable artifact — the §4/§5 difference, now visible in YAML.

---

## 6. Pros and Cons

### 6.1 bashcov

**Pros**

- **Zero-instrumentation elegance:** it reuses Bash's own `xtrace` machinery — your scripts are executed exactly as-is, no source rewriting, no debug-register tricks, no ASLR games (Verified — runner.rb).
- **Automatic merging across suites:** wrap shUnit2 *and* Bats *and* bash_unit, and the SimpleCov cache merges everything into one report — the README's headline feature, and the exact multi-framework estate pattern of the shell-testing series (Verified — README).
- **SimpleCov familiarity:** for anyone who has seen a Ruby coverage report, the HTML is instantly readable; `.simplecov` gives Ruby-grade configurability (filters, groups, thresholds).
- **Simple to run:** one gem, one wrapper command, `./coverage/index.html` — lowest ceremony of the two (Verified — README).
- **MIT license** — no GPL considerations for a bank's toolchain (Verified).
- **Tracks `cd`/`pushd`/`popd` correctly** via the PWD/OLDPWD stacks — a genuine edge bash coverage tools trip on (Verified — USAGE.md).

**Cons**

- **Bash-only scope:** nothing compiled, nothing Python, no `/bin/sh`-as-sh story (its `--bash-path` swaps which bash; it cannot follow a dash script).
- **Ruby runtime dependency** on every machine that runs coverage — for a change-controlled estate that is a new runtime to approve, install and patch (cross-ref the business-case adoption lens).
- **Root caveat:** running as root is warned against and handled with a `BASH_ENV` workaround (Bash 4.4+ PS4 inheritance bug — Verified — runner.rb) — a real constraint in containerised CI where the default user is root.
- **Single-maintainer bus factor** and no first-party CI uploaders (flag: adoption/community metrics beyond the repo could not be verified this session; the 2012 origin, MIT license and decade of maintenance are verified).
- **Line coverage only:** no function/branch metrics (§3), and the "gory details" heuristics (PWD-stack path resolution) are acknowledged to be susceptible to false positives (Verified — USAGE.md).
- **Performance:** `PS4` expands on *every* command in debug mode; USAGE.md itself notes subshells in PS4 are slow (bashcov avoids them for that reason). Expect real overhead on large suites (unquantified — flagged).

### 6.2 kcov

**Pros**

- **One tool, three languages:** compiled ELF/Mach-O binaries, Python and Bash in a single binary — the only coverage tool in this series that can measure a *mixed* estate (Verified — README).
- **No recompilation:** DWARF-based collection "without special compilation options" (Verified — repo description) — crucial for vendor binaries where you cannot recompile.
- **Block-level granularity** for compiled code (finer than lines), plus accumulated/multi-program runs and a merged union view (Verified — man page).
- **First-class CI integration:** Coveralls/Codecov/Jenkins/SonarQube/GitLab/GitHub/VS Code all documented; `--coveralls-id` is a one-flag upload (Verified — README, man page).
- **Packaging:** apt package, official Docker image, distro man page, GitHub Actions release builds (Verified) — install friction is low.
- **Live output:** continuously regenerated HTML for long-running programs, `--output-interval` tunable (Verified — web page, man page).
- **Two Bash collection methods** (PS4 or DEBUG trap) plus `--bash-parser` and the LD_PRELOAD `/bin/sh` redirector — more Bash coverage knobs than bashcov has (Verified — man page).
- **Active project:** v43 (2024), 71 contributors, 2,202 commits — and ShellSpec, the framework comparison's coverage-integrated harness, standardises on it (Verified).

**Cons**

- **Mechanism weight:** ptrace + breakpoint patching + CPU pinning + ASLR disabling is a lot of machinery; the `--verify` option exists because bad DWARF can break it (Verified — man page); it can be flaky in hardened/containerised environments (flagged — the man page's own caveats on the LD_PRELOAD redirector and CLOEXEC options are evidence).
- **Bash support is the junior citizen:** the Bash engine is newer than the ptrace engine, per-line only, and its helper (`bash-helper.sh`) is a four-line `PS4` + `set -x` — functionally the same tracing contract as bashcov, without bashcov's PWD-stack path handling or SimpleCov merging semantics (Verified — repo layout).
- **GPL-2.0 license** — a consideration for proprietary toolchain distribution that MIT's bashcov does not raise.
- **Output noise without filters:** `/usr/include` and system libraries are covered by default; the `--include-pattern`/`--exclude-pattern`/`--exclude-line`/`--exclude-region` filters are effectively mandatory for script projects (Verified — README, umbrella §7.1).
- **No per-test attribution, no delta reporting** (Verified-limitation named by the umbrella §7.1).
- **Build-from-source on some platforms** (cmake/ninja, DWARF/ELF deps, macOS entitlements files in the repo) — apt/Docker smooth this on Linux, less so elsewhere.

### 6.3 The Pros-Cons Table

| Criterion | bashcov | kcov |
|---|---|---|
| **Scope** | ✅ Bash only (simple) | ✅ compiled + Python + Bash (wide) |
| **Install friction** | ✅ one gem (needs Ruby) | ✅ apt/Docker (needs native build or package) |
| **Report quality** | ✅ SimpleCov HTML, familiar, merged across suites | ✅ LCOV-style HTML + Cobertura + JSON, live updates |
| **CI integration** | 🟡 via SimpleCov ecosystem | ✅ documented first-class + `--coveralls-id` |
| **Language coverage** | ❌ Bash only | ✅ 3 families |
| **License** | ✅ MIT | 🟡 GPL-2.0 |
| **Bash-specific depth** | ✅ PWD-stack cd handling, suite merging | 🟡 PS4/DEBUG choice, `/bin/sh` redirector, but line-only and no merge semantics |
| **Granularity** | 🟡 line only | ✅ block-level (compiled), line-level (scripts) |
| **Maintenance risk** | 🟡 single maintainer | ✅ 71 contributors, v43, distro-packaged |
| **Runtime constraints** | 🟡 Ruby runtime; root caveat | 🟡 ptrace/CPU-pinning machinery; filter flags required |

---

## 7. Decision — When to Use Which

### 7.1 The Decision

**Choose bashcov when:** the estate is **Bash-only**, Ruby is already (or cheaply) available, you want the **lowest-ceremony wrapper**, and you value the **automatic merge across multiple frameworks** — the exact shape of this series' shell-testing estate (shUnit2 suite + Bats suite + bash_unit suite, one combined SimpleCov picture). It is also the pick when MIT licensing matters or when SimpleCov familiarity among the team is a real asset.

**Choose kcov when:** the estate is **mixed** (shell scripts guarding binaries, Python sidecars — the realistic bank-automation shape: a Bash orchestrator, a Python ETL, a compiled helper), when you need **CI-native artifacts** (Cobertura for Jenkins, JSON for GitLab, `--coveralls-id` for Coveralls), when you want **one installed tool** with apt/Docker packaging rather than a Ruby runtime, or when the harness is **ShellSpec** (whose built-in coverage is kcov).

**The tie-breaker for a regulated shop:** for *pure Bash*, both do line coverage by the same underlying trick (PS4 + xtrace). bashcov is the better *Bash* citizen (suite merging, cd handling, SimpleCov quality); kcov is the better *platform* (scope, integration, maintenance). If the coverage requirement is "the scripts" — the umbrella's ask — either works; if it is "the scripts **and** the things they call" — a bank's MFT scripts calling a compiled transfer agent — only kcov sees both.

### 7.2 The Decision Table

| Situation | Pick | Why (verified basis) |
|---|---|---|
| Bash-only estate, multiple test frameworks | **bashcov** | automatic cache+merge across suites; SimpleCov report; MIT; one gem |
| Mixed Bash + Python + compiled binaries | **kcov** | only one of the two covers all three (repo scope line) |
| Jenkins/SonarQube/GitLab CI pipelines | **kcov** | Cobertura XML, sonarqube.md, gitlab.md JSON — all documented |
| Coveralls or Codecov upload wanted | **kcov** (one flag) / bashcov (ecosystem assembly) | `--coveralls-id` vs SimpleCov formatters |
| ShellSpec harness | **kcov** (built-in) | framework comparison coverage row: ✅ kcov integrated |
| Ruby runtime unacceptable / hardened CI | **kcov** | apt/Docker binary vs `gem install` |
| Lowest ceremony, familiar Ruby-style reports | **bashcov** | `bashcov -- bats tests/` → `coverage/index.html` |
| GPL avoidance in proprietary distribution | **bashcov** | MIT vs GPL-2.0 |
| Vendor binary, no recompilation possible | **kcov** | DWARF-based, "no special compilation options" |
| Change-controlled toolchain, minimal deps | **kcov** (single native tool) | apt package + Docker image; one dependency |

### 7.3 What If the Context Changes?

- **Adoption cost (cross-ref ../management/business_case_development_guide.md):** bashcov's cost is a Ruby runtime to approve and patch; kcov's is a native binary plus, for scripts, its filter flags to learn. In a bank's change-controlled estate the Ruby runtime is the heavier approval; kcov's apt package is closer to "another yum/apt package".
- **If the estate later adds Python or compiled components**, bashcov silently stops covering the new code — kcov's scope line becomes the differentiator and the migration cost is one wrapper change.
- **If coverage becomes a release gate,** kcov's CI-native artifacts and `--dump-summary` make the gate plumbing easier; bashcov needs a formatter step to produce a machine-checkable number beyond the HTML.

---

## 8. Worked Example — One Script, Both Tools

### 8.1 The Scenario: The Familiar MFT Payment-File Script

The same script and suite from the shell-testing comparison's §7 — this guide's companion — covered here by both tools. The context (cross-ref the bank series — [../banking/nets_software_systems_guide.md](../banking/nets_software_systems_guide.md) and the transfer-automation scripts): an ops script that **pre-validates a payment instruction file before an MFT (Managed File Transfer) push to a clearing system**. Bad files must be caught before they leave the bank — exactly the kind of script a regulated shop wraps in tests *and then measures*.

`validate_payment_file.sh` — reproduced from shell_testing_frameworks_comparison_guide.md §7.1 (the canonical copy lives there; only the parts coverage cares about are annotated here):

```bash
#!/usr/bin/env bash
# validate_payment_file.sh — pre-MFT payment file validation
set -u

CURRENCIES="SGD USD EUR JPY AUD GBP"

validate_header() {
  local line="$1" date_str count
  [[ "$line" == HDR\|* ]] || { echo "missing HDR record" >&2; return 1; }
  date_str="${line#HDR|}"; date_str="${date_str%%|*}"
  [[ "$date_str" =~ ^[0-9]{4}-[0-9]{2}-[0-9]{2}$ ]] || { echo "bad header date: $date_str" >&2; return 1; }
}

validate_amount() {
  local amt="$1"
  [[ "$amt" =~ ^[0-9]+(\.[0-9]{1,2})?$ ]] && [ "$amt" != "0" ] && [ "$amt" != "0.00" ] \
    || { echo "bad amount: $amt" >&2; return 1; }
}

validate_currency() {
  local ccy="$1"
  [[ " $CURRENCIES " == *" $ccy "* ]] || { echo "unknown currency: $ccy" >&2; return 1; }
}

validate_checksum() {
  local file="$1" body_sum=0 expected sum=0 c
  expected=$(tail -n1 "$file" | cut -d'|' -f2)
  while IFS= read -r line; do
    [[ "$line" == CHK\|* ]] && continue
    for (( i=0; i<${#line}; i++ )); do
      c="${line:$i:1}"; sum=$(( sum + $(printf '%d' "'$c") ))
    done
  done < "$file"
  body_sum=$(( sum % 1000 ))
  [ "$body_sum" -eq "$expected" ] || { echo "checksum mismatch: got $body_sum want $expected" >&2; return 1; }
}

validate_payment_file() {
  local file="$1" line hdr_count pay_count=0 ccy amt
  [ -f "$file" ] || { echo "no such file: $file" >&2; return 2; }
  local -i lineno=0
  while IFS= read -r line; do
    lineno=$((lineno + 1))
    case "$line" in
      HDR\|*) hdr_count="${line##*|}"; validate_header "$line" || return 1 ;;
      PAY\|*) pay_count=$((pay_count + 1))
              amt="${line#PAY|}"; amt="${amt#*|}"; amt="${amt%%|*}"
              ccy="${line#PAY|}"; ccy="${ccy#*|}"; ccy="${ccy#*|}"; ccy="${ccy%%|*}"
              validate_amount "$amt" || return 1
              validate_currency "$ccy" || return 1 ;;
      CHK\|*) validate_checksum "$file" || return 1 ;;
      *) echo "unknown record type on line $lineno: $line" >&2; return 1 ;;
    esac
  done < "$file"
  [ "$pay_count" -eq "$hdr_count" ] || { echo "record count mismatch: header $hdr_count, found $pay_count" >&2; return 1; }
  echo "OK"
}

if [ "${BASH_SOURCE[0]}" = "$0" ]; then validate_payment_file "$1"; fi
```

The suite (from the framework comparison §7 — six tests, each guarding one rule): (1) accepts a valid file → exit 0 + `OK`; (2) rejects a bad header date; (3) rejects a negative amount; (4) rejects an unknown currency; (5) rejects a checksum mismatch; (6) rejects a missing file → exit 2. Fixtures: `test/fixtures/valid.pay` plus per-test variants. In this guide the suite is assumed to exist in Bats (`test/`); the harness is irrelevant to the measurement — that is the point of the wrapper design (cross-ref the coverage row: Bats 🟡 kcov/bashcov external).

**The lab.** For this guide the scenario was **actually executed** (not just simulated): kcov 43+dfsg-2 and bashcov 3.3.0 (Ruby 3.3.8, Bash 5.3) were installed on a fresh Ubuntu container, the script and a six-scenario driver (`run_suite.sh` — the six tests from §7 of the framework comparison, invoking the script with per-test fixtures) were created in `/tmp/coverage-lab`, and the suite ran **6/6 passing** before any coverage tool touched it. One lab note: the framework comparison's canonical fixture carries `CHK|817`, but the checksum algorithm in the script computes **85** for that body — the sibling's fixture value is illustrative, not computed. The lab's `valid.pay` therefore uses `CHK|085` (numeric `-eq` comparison makes the leading zero harmless) so the happy-path test is self-consistent. Verified: `bash run_suite.sh` → "SUITE: 6/6 passed".

**Before running either tool, predict the gaps** (this is the TDD-habit payoff, cross-ref test-driven-development_guide.md): no test feeds an *unknown record type* (`*)` case) — that line should be red; no test produces a *first line that is not HDR* (the "missing HDR record" arm) — but that arm shares its line with the happy-path prefix check, so *line* coverage may still show it green; the *record count mismatch* check shares its line with the happy-path count comparison — same same-line blindness; and test 5 *does* exercise the failed-checksum arm, so the checksum function should be mostly green. The interesting question the run answers is which of these predictions the tools can actually *show* — and what else shows red that nobody predicted.

### 8.2 The Coverage Runs: bashcov

Install and wrap the suite (Verified commands from the bashcov README, run for real in the lab):

```bash
gem install bashcov                    # bashcov 3.3.0 on Ruby 3.3.8
bashcov --skip-uncovered -- ./run_suite.sh
```

A usage subtlety learned the hard way: bashcov *prepends its own bash* to your command, so you pass the **script**, not `bash script` — `bashcov -- ./run_suite.sh` (passing `bash run_suite.sh` made it try to execute `/bin/bash bash` and fail with "cannot execute binary file"). The same rule means the canonical `bashcov -- bats tests/` works because the `bats` entry point *is* a bash script.

What happens mechanically (Verified — runner.rb): bashcov spawns the script with `xtrace` in `SHELLOPTS`, the `PS4` records `LINENO|BASH_SOURCE|PWD|OLDPWD` for every command, the trace streams through `BASH_XTRACEFD` into the Ruby parser, and the six child invocations of `validate_payment_file.sh` — separate bash processes spawned by the driver — are traced too, because they inherit the environment. Repeat for each framework in the estate and bashcov merges them automatically:

```bash
bashcov --skip-uncovered -- shunit2_tests.sh     # if the shUnit2 suite lives too
bashcov --skip-uncovered -- bash_unit_tests.sh   # and a bash_unit suite
```

**The measured result** (this session, not a guess): the run printed

```
Coverage report generated for /bin/bash ./run_suite.sh to /tmp/coverage-lab/coverage.
Line Coverage: 91.8% (56 / 61)
```

and `coverage/.resultset.json` breaks it down: `validate_payment_file.sh` **35/38 lines (92.1%)**, `run_suite.sh` 21/23 (91.3%). The per-line map of the script (from the resultset, lines are source lines):

| Source line | Content | bashcov |
|---|---|---|
| 52 | `*) echo "unknown record type on line $lineno: $line" >&2; return 1 ;;` | **MISS — the real gap** |
| 33 | `done < "$file"` (checksum loop) | MISS — **xtrace artifact** |
| 54 | `done < "$file"` (main loop) | MISS — **xtrace artifact** |
| 55 | `[ "$pay_count" -eq "$hdr_count" ] \|\| { echo "record count mismatch"... }` | HIT x1 — error arm invisible |
| 9 | `[[ "$line" == HDR\|* ]] \|\| { echo "missing HDR record"... }` | HIT x5 — error arm invisible |
| 35 | `[ "$body_sum" -eq "$expected" ] \|\| { echo "checksum mismatch"... }` | HIT x4 — arm exercised by test 5 |
| 59 | `if [ "${BASH_SOURCE[0]}" = "$0" ]; then validate_payment_file "$1"; fi` | HIT x12 |

Reading the red lines: **line 52 is the predicted real gap** — no test feeds a garbage record, and bashcov says so loudly. Lines 33 and 54 were *not* predicted — they are the loop-closing `done` lines, and they are red in every run because Bash's xtrace simply never emits a separate trace record for the `done` keyword of a `while ... done` compound (the loop's commands are traced, its closing keyword is not). This is the classic **line-attribution artifact** of trace-based coverage: the line is instrumented-relevant to the lexer but unreachable by the tracer. And the predictions about same-line arms were confirmed: lines 9, 55 and 35 show green single hit-counts that cannot distinguish the executed condition from the unexecuted error arm — exactly the branch-blindness §3.3 predicted.

One caveat on the flag: `--skip-uncovered` hides files that never ran at all — with it, only `run_suite.sh` and `validate_payment_file.sh` appear; without it, every `.sh` under the project root is listed at 0%. For a focused script-level report, `--skip-uncovered` is the right default (Verified — option semantics in `lib/bashcov.rb`).

### 8.3 The Coverage Runs: kcov

Install and wrap (Verified commands from the kcov README and the umbrella's §6 workflow, run for real in the lab):

```bash
sudo apt install kcov                   # kcov 43+dfsg-2
kcov --include-pattern=validate_payment_file.sh,run_suite.sh kcov-out ./run_suite.sh
```

What happens mechanically (Verified — engines): kcov execs `./run_suite.sh`; because the target is a bash script, the **Bash engine** kicks in (`bash-engine.cc`), injecting `PS4='kcov@${BASH_SOURCE}@${LINENO}@'` + `set -x` (bash-helper.sh) and reading the trace through its own trace-fd machinery; the six child invocations of `validate_payment_file.sh` inherit the trace fd and are covered too. The output directory shows the engine's working parts directly: `bash-helper.sh`, `libbash_execve_redirector.so`, `libbash_tracefd_cloexec.so` and a `data/` tree beside the per-executable report directory `run_suite.sh.ca002d4a896b19cb/` containing `index.html`, `coverage.json`, `cobertura.xml` and `codecov.json`.

**The measured result** (this session): `coverage.json` reports **overall 91.67%** — `validate_payment_file.sh` **34/37 lines (91.89%)**, `run_suite.sh` 21/23 (91.30%). The per-line map, decoded from the report's data file, is **line-for-line identical to bashcov's** for the script's uncovered set: the same line 52 (`*)` unknown-record — real gap), the same lines 33 and 54 (`done < "$file"` — artifacts), the same HIT x1 on the count-mismatch line 55, the same HIT x12 on the guard line 59. Even the hit counts agree down to the digit on the hot lines (line 30: x302 in both; line 31: x438 in both). One marginal difference: bashcov counts line 17 (`[[ "$amt" =~ ... ]] && [ "$amt" != "0" ] ...`) as covered while kcov omits it from its report entirely (35/38 vs 34/37 denominators) — a line-attribution difference at the margin between two independent parsers, not a coverage disagreement.

Two kcov-specific findings from the lab are worth their weight:

1. **The filter is not just about `/usr/lib`.** Run kcov without `--include-pattern` in a *clean* directory and it still reports exactly 2 files — the two scripts. The noise the umbrella warns about appears when the output directory (or other artifacts) sits *inside the scanned directory*: kcov "parses the directory of the binary for other scripts" by default (`--bash-dont-parse-binary-dir` turns it off), and in the first lab run the previous run's own HTML/JS artifacts were picked up as "scripts" and reported at 0.00% — kcov was measuring its own output. Keep `kcov-out` outside the source tree, or pass the filter.
2. **The `--bash-method=DEBUG` variant under-traces child processes.** `kcov --bash-method=DEBUG --include-pattern=validate_payment_file.sh kcov-debug ./run_suite.sh` in a clean directory produced `validate_payment_file.sh 0.00%` — the DEBUG trap method traced the driver but not the child script (the trap does not survive the child exec the way the inherited PS4/trace-fd contract does in this environment). The default **PS4 method is the reliable choice for suites that spawn child scripts**; treat DEBUG as an experiment for single-script targets.

Also verified in the lab: the `--include-pattern` comma-list form (`validate_payment_file.sh,run_suite.sh`) works as documented, and the report's `[merged]`/accumulation structure is present (`kcov-merged` view directory).

### 8.4 The Two Reports Compared

| Aspect | bashcov report | kcov report |
|---|---|---|
| **Headline (measured)** | Line Coverage **91.8% (56/61)**; script **35/38 = 92.1%** | **91.67%** overall; script **34/37 = 91.89%**; driver 21/23 = 91.30% |
| **Uncovered set (measured)** | lines 33, 52, 54 (52 = real gap; 33/54 = `done` artifacts) | lines 33, 52, 54 — **identical** |
| **Hit counts** | e.g. line 30 ×302, line 31 ×438 | same ×302 / ×438 — agreement to the digit |
| **Margin difference** | line 17 counted covered (35/38) | line 17 omitted from report (34/37) |
| Entry point | `coverage/index.html` (SimpleCov) | `kcov-out/run_suite.sh.<hash>/index.html` |
| Machine-readable | `.resultset.json` (SimpleCov format) | `coverage.json` + `cobertura.xml` + `codecov.json` |
| Filtering | SimpleCov filters / `--skip-uncovered` | `--include-pattern` etc.; also keep output dir out of the scanned tree |
| Same gaps shown? | yes | **yes — line-for-line identical** |

The two tools' reports agree on the *science* (which lines ran — down to the digit) and differ on the *packaging* (who renders, merges and ships it). When two independent implementations of the same tracing contract produce identical uncovered-line sets, that agreement *is* the validation.

### 8.5 The Lessons

1. **The wrapper design pays off twice.** Neither tool touched a single line of the script or the suite; both wrapped `run_suite.sh` unchanged. The framework comparison's 🟡 "external coverage" row is not a weakness — it is the feature that lets you add measurement to an existing estate in one command.
2. **Both tools agree on the gaps — to the digit.** The six-test suite leaves the unknown-record `*)` case (line 52) uncovered; both reports say so, along with identical hit counts on the hot lines (×302, ×438). *Two independent implementations of line tracing (Ruby PS4-parser vs C++ trace-fd engine) painting the same lines red is the strongest possible evidence the measurement is right.*
3. **Coverage finds the missing test, not the missing code.** The red line 52 is not a bug in `validate_payment_file.sh`; it is a missing test for "a payment file with a garbage record" — a real-world corruption case for an MFT feed. The next RED test is written against the red line (cross-ref test-driven-development_guide.md).
4. **Red lines need reading, not trusting.** The lab's other two red lines (33, 54) are `done < "$file"` — trace-attribution artifacts that would be red in *any* run, with any suite, including a perfect one. A coverage gate that fails on "uncovered lines" without a human reading them would fail your scripts on lines that can never be covered. Budget this when you set thresholds.
5. **Line coverage is blind to same-line arms — and both tools share the blindness.** The "missing HDR record" arm (line 9), the "record count mismatch" arm (line 55) and the "bad amount" arm all sit on lines that are green from the happy path. Neither tool can split them. This is §3.3 in the flesh: for shell scripts, treat coverage as a gap-finder for *lines*, and write tests for branch rules explicitly.
6. **The tool choice shows up in the plumbing, not the picture.** The line map is identical; what differs is the runtime (Ruby gem vs apt binary), the artifacts (resultset JSON vs Cobertura/JSON), the filter gotchas (output-dir pollution vs `--skip-uncovered`) and the Bash-engine knobs (kcov's DEBUG-trap caveat vs bashcov's PS4 robustness). For the pure-Bash picture, pick on plumbing; for a mixed estate, kcov's scope decides.
7. **Measure the merged suite, not one framework.** A single harness's coverage is a lower bound on the truth; bashcov's merge (or kcov's accumulation) across shUnit2 + Bats + bash_unit is what produces the estate-level blank-spot map the series has been driving toward.

---

## 9. Summary — One Page

**The two tools.** bashcov (infertux/bashcov, 2012, MIT, Ruby gem) is a *Bash trace-parser*: it runs your command with `xtrace` on, reads the trace through `BASH_XTRACEFD`, counts line hits from `PS4`-expanded `LINENO`/`BASH_SOURCE` records, and reports SimpleCov-style HTML, auto-merging across every wrapped test suite. kcov (SimonKagstrom/kcov, GPL-2.0, C++, fork of Bcov) is a *native coverage engine*: for compiled binaries it reads DWARF, forks the target under ptrace and plants software breakpoints (patched instructions — INT3 on x86, `ebreak` on RISC-V) on every basic block; for Bash and Python it runs dedicated script engines (PS4 or DEBUG-trap for Bash); it emits LCOV-style HTML, Cobertura XML and JSON, with documented Coveralls/Codecov/Jenkins/SonarQube/GitLab integration.

**The dimensions.** Line coverage is first-class in both (kcov's compiled mode is finer — basic-block). Function and branch coverage are **not** first-class metrics in either for shell scripts — gaps show as red lines, read by humans.

**The decision.** Pure-Bash estate, multi-framework, Ruby acceptable, MIT preferred → **bashcov**. Mixed estate (scripts + Python + binaries), CI-native artifacts, Jenkins/GitLab/Coveralls, ShellSpec harness, no-Ruby constraint → **kcov**.

**The worked example.** The MFT payment-file script from the framework comparison, covered by both tools in a real lab run (bashcov 3.3.0 and kcov 43, Ubuntu, six-test suite 6/6 green): bashcov reported **91.8% (56/61)** overall and the script at **35/38 (92.1%)**; kcov reported **91.67%** overall and the script at **34/37 (91.89%)** — with **line-for-line identical uncovered sets** (line 52, the untested unknown-record case = the real gap; lines 33/54, `done` keywords = trace artifacts) and hit counts agreeing to the digit (×302, ×438). Different plumbing (resultset JSON vs Cobertura/JSON; gem vs apt; merge vs accumulate), identical science.

**The final word — the coverage of the scripts.** For the scripts that guard a bank's payment flows, coverage is the audit trail of "which rules the tests actually exercised". Both tools deliver that audit trail for Bash; kcov delivers it for everything around the scripts too. Measure the scripts with either, measure the estate with kcov — and treat every red line as the next test to write.

---

## 10. Glossary

- **bashcov** — a Ruby gem (MIT, created 2012, Cédric Félizard/infertux) that measures Bash script coverage by parsing Bash's own `xtrace` output (captured via `BASH_XTRACEFD`) and reporting through SimpleCov.
- **kcov** — a C++ coverage engine (GPL-2.0, Simon Kagstrom, fork of Bcov) covering compiled programs (via DWARF + ptrace + software breakpoints), Python and Bash; outputs LCOV-style HTML, Cobertura XML and JSON.
- **coverage** — in this guide, *test coverage*: the fraction of a program's code that a test run actually executed; used as a gap-finder for the test suite, not as a pass/fail number in itself.
- **code coverage** — the general practice of measuring which lines/blocks/functions/branches of a program were executed during a test run; line, function and branch coverage are its classic dimensions.
- **line coverage** — the proportion of executable source lines executed at least once; the primary (and for these tools, the only first-class) dimension for Bash.
- **function coverage** — the proportion of functions called at least once; not reported as a metric by bashcov or kcov for shell scripts.
- **branch coverage** — the proportion of branch outcomes (both sides of an `if`/`case`) taken; implicit in kcov's compiled-mode basic blocks, not computed for Bash by either tool.
- **BASH_XTRACEFD** — a Bash variable that redirects the `xtrace` (debug-trace) output away from stderr to the given file descriptor; bashcov's capture channel; kcov has its own trace-fd equivalent (with a CLOEXEC option for children).
- **xtrace** — Bash's debug mode (`set -x` / `SHELLOPTS=xtrace`): prints each expanded command (prefixed by `PS4`) as it executes; the raw material both tools parse for line hits.
- **PS4** — the Bash variable expanded as the prefix of each xtrace line; bashcov sets it to expand `LINENO`, `BASH_SOURCE`, `PWD`, `OLDPWD`; kcov's bash helper sets `PS4='kcov@${BASH_SOURCE}@${LINENO}@'`.
- **DEBUG trap** — Bash's `trap ... DEBUG` hook that runs before every command; kcov's alternative Bash collection method (`--bash-method DEBUG`).
- **ptrace** — the Linux/BSD process-tracing syscall family (PTRACE_TRACEME, PTRACE_CONT, ...) kcov uses to observe and control the target process.
- **debug register** — x86 hardware breakpoint registers (DR0–DR7); **flagged**: kcov is often described as using them, but the verified source uses software breakpoints (instruction patching) instead.
- **DWARF** — the debugging-information format kcov parses to map source lines to machine addresses and basic blocks, enabling coverage without special compiler options.
- **basic block** — a straight-line run of instructions with one entry and one exit; kcov's compiled-mode coverage unit.
- **LCOV** — a popular coverage toolchain/report style (originally for GCC's gcov); kcov emits "lcov-style HTML"; neither tool natively emits LCOV `.info` tracefiles.
- **JSON** — JavaScript Object Notation; bashcov's `.resultset.json` (SimpleCov) and kcov's `coverage.json` are machine-readable coverage outputs.
- **HTML** — the human-readable coverage reports: SimpleCov's `coverage/index.html` (bashcov) and kcov's per-executable `index.html`.
- **Cobertura** — the XML coverage-report format consumed by Jenkins' Cobertura plugin; kcov writes `cobertura.xml` natively.
- **SimpleCov** — the de-facto Ruby code-coverage library; bashcov's reporting engine (HTML + resultset JSON + merging).
- **Ruby gem** — a packaged Ruby library; bashcov is distributed as `gem install bashcov`.
- **C (C++)** — kcov's implementation language (~54% C++ in the repo), relevant to its distribution as a compiled binary.
- **Python** — the second script language kcov covers natively via its Python engine (and a common sidecar language in bank automation estates).
- **CI** — Continuous Integration; both tools are wrappers that run inside CI jobs; kcov documents GitHub/GitLab/Jenkins integration.
- **Codecov** — a coverage-hosting/PR-commenting service; kcov documents integration (doc/codecov.md), bashcov reaches it via SimpleCov formatters.
- **Coveralls** — a coverage-hosting service; kcov uploads via `--coveralls-id` (doc/coveralls.md).
- **MFT** — Managed File Transfer, the channel a bank uses to push payment files to clearing systems; the worked example's context.
- **script** — in this series, a Bash script (e.g. `validate_payment_file.sh`); the subject of coverage in this guide.

---

## 11. Claims Status, References and Disclaimer

**Verification status.** Facts marked (Verified) were checked against primary sources in this session: the bashcov README/source (`runner.rb`, `xtrace.rb`, `lib/bashcov.rb`, `USAGE.md`); the kcov README, `doc/kcov.1` man page, `src/engines/ptrace.cc`, `src/engines/bash-helper.sh`, `src/engines/ptrace_sys.hh`, repo page and web page. **Additionally, the worked example (§8) was executed for real** in this session: bashcov 3.3.0 and kcov 43+dfsg-2 were installed and run against `validate_payment_file.sh` with a six-test driver on Ubuntu (Bash 5.3), and the headline numbers, per-line maps, hit counts, filter behaviour and the `--bash-method=DEBUG` under-tracing finding are all measured output, not estimates. **Flagged/unverified:** kcov's "debug-register" mechanism (source shows software breakpoints — corrected in §2.2); bashcov adoption metrics (web-search backend unavailable this session; only repo-verified facts stated); the DEBUG-trap under-tracing and the identical-hit-count findings are environment-specific (Ubuntu, Bash 5.3, kcov 43) and may differ elsewhere; "performance overhead" magnitudes for both tools.

**References.**

- bashcov — github.com/infertux/bashcov (README, USAGE.md, lib/bashcov/runner.rb, lib/bashcov/xtrace.rb, lib/bashcov.rb); demo reports at infertux.github.io/bashcov
- kcov — github.com/SimonKagstrom/kcov (README, doc/kcov.1, src/engines/*, ChangeLog, releases v43); simonkagstrom.github.io/kcov (feature list, Bcov lineage)
- Bcov (kcov's ancestor) — bcov.sourceforge.net, by Thomas Neumann
- SimpleCov — github.com/colszowka/simplecov (merging, resultset, formatters)
- Sibling guides — shell_testing_frameworks_comparison_guide.md (§1.2 coverage row, §7 worked-example suite), bash_script_testing_automation.md (§6 CI workflow, §7.1 Test Coverage), bats_core_guide.md (§7 Bats coverage limits), test-driven-development_guide.md (§7 RED-GREEN-REFACTOR), ../banking/nets_software_systems_guide.md (MFT context), ../management/business_case_development_guide.md (adoption lens)

**Disclaimer.** This guide was researched from public sources in August 2026; versions, stars and options change. The worked example's headline numbers are real measured output from one Ubuntu/Bash 5.3 container (kcov 43+dfsg-2, bashcov 3.3.0) and will shift with versions, shell builds and fixture content — re-run on your own checkout before adopting a coverage gate. Nothing here is confidential; it is public research for engineering judgement, not a compliance or audit opinion.

