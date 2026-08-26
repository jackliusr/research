# Shell Testing Frameworks — shUnit2 vs Bats vs bash_unit vs assert.sh vs bashaspec Head-to-Head

> **Author:** Jack Liu Shurui — Solution Architect at Cymbal Bank, Singapore
> **Context:** Technology Research — shell-automation testing series; the dedicated **five-plus-way head-to-head** on shell testing frameworks: the two mainstream veterans (shUnit2, Bats/bats-core), the lightweight contenders (bash_unit, assert.sh), the RSpec-style family (bashaspec, shpec, ShellSpec), and the bats-assert ecosystem. Where the umbrella guide asks *"which framework should I use?"* and the Bats guide goes deep on the winner, this guide answers *"how do they actually differ, and how do I choose?"*
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **Primary Sources:** the GitHub repositories and READMEs of shunit2/kward (kward/shunit2), bats (sstephenson/bats), bats-core (bats-core/bats-core, bats-assert, bats-support, bats-file), bash_unit (bash-unit/bash_unit), assert.sh (lehmannro/assert.sh), bashaspec (d10n/bashaspec), ShellSpec (shellspec/shellspec), shpec (rylnd/shpec), bats-mock (jasonkarns/bats-mock); the GitHub API (repository creation dates); testanything.org (TAP history and specifications); the sibling guides in this repository (see "How to Read This Guide")
> **Last Updated:** August 2026

---

## Table of Contents

1. [Overview — The Five-Plus Frameworks](#1-overview--the-five-plus-frameworks)
   - 1.1 The Frameworks
   - 1.2 The Three Lineages: xUnit, TAP, BDD
   - 1.3 The Overview Table
   - 1.4 Verification Notes
2. [Framework Profiles](#2-framework-profiles)
   - 2.1 shUnit2 — The xUnit Veteran
   - 2.2 Bats / bats-core — The TAP Standard
   - 2.3 bash_unit — The Lightweight Single-File
   - 2.4 assert.sh — The Minimal Assertion Library
   - 2.5 bashaspec — The Verification Caveat
   - 2.6 ShellSpec — The Modern BDD Framework
   - 2.7 shpec — The Other RSpec-Style Contender
   - 2.8 The Profile Table
3. [Feature Matrix](#3-feature-matrix)
   - 3.1 The Styles: xUnit, TAP, BDD
   - 3.2 The Capabilities
   - 3.3 The Matrix Table
4. [Ecosystems](#4-ecosystems)
   - 4.1 The Bats Ecosystem: bats-support, bats-assert, bats-file, bats-mock
   - 4.2 Everything Else
   - 4.3 The Ecosystem Table
5. [Pros and Cons](#5-pros-and-cons)
   - 5.1 Per Framework
   - 5.2 The Pros-Cons Table
6. [Decision Matrix](#6-decision-matrix)
   - 6.1 The Criteria and the Weights
   - 6.2 The Weighted Scoring
   - 6.3 The Decision Table
   - 6.4 What If the Weights Change?
7. [Worked Example — One Suite, Six Harnesses](#7-worked-example--one-suite-six-harnesses)
   - 7.1 The Scenario: An MFT Payment-File Validation Script
   - 7.2 The Suite in shUnit2
   - 7.3 The Suite in Bats
   - 7.4 The Suite in bash_unit
   - 7.5 The Suite in assert.sh
   - 7.6 The Suite in bashaspec
   - 7.7 The Suite in ShellSpec
   - 7.8 The Outputs Compared
   - 7.9 The Lessons
8. [Summary — One Page](#8-summary--one-page)
9. [Glossary](#9-glossary)
10. [Claims Status, References and Disclaimer](#10-claims-status-references-and-disclaimer)

### How to Read This Guide

This is the dedicated **head-to-head on shell testing frameworks**, in the same genre as the sibling head-to-heads [huggingface_vs_csghub_guide.md](huggingface_vs_csghub_guide.md) and the Nutanix-vs-OpenShift precedent: comparison tables, a weighted decision matrix, a worked example, and a verdict. It deliberately **deepens the comparison dimension** — it does not re-derive single-framework depth, which lives in the siblings:

- **The umbrella (THE sibling)** — [bash_script_testing_automation.md](bash_script_testing_automation.md) is the 547-line umbrella covering frameworks, linting (ShellCheck), formatting (shfmt), mocking patterns, CI/CD, coverage (kcov), and project structure. Its **§8 is a two-way "bats vs shunit2" table** (style, setup/teardown, assertions, output capture, discovery, TAP, parallel, CI, community, shell compatibility, activity, dependencies, learning curve) and its **§9 is a scenario→tool recommendations table**. This guide is the *five-plus-way* expansion of §8: every row of that table is revisited here across all six frameworks, and every §9 scenario is scored in §6.
- **The Bats depth (THE sibling)** — [bats_core_guide.md](bats_core_guide.md) is the 743-line dedicated Bats guide: history (§2), core concepts (§4), the assertion libraries (§5), coverage (§7), and its own "Bats vs alternatives" section (§8: vs shellcheck, shunit2, assert.sh, shellspec, pytest, custom scripts). Cross-reference it for anything Bats-specific; this guide only summarises Bats and positions it against the field. Its §8 characterisations of shunit2 ("2008-era"), assert.sh ("not a framework"), and shellspec ("RSpec-inspired DSL") are the sibling's independent verdicts, which this guide corroborates from primary sources.
- **The TDD discipline** — the dev-tooling cluster's [test-driven-development_guide.md](test-driven-development_guide.md): the worked example in §7 is written RED-GREEN-REFACTOR style (each suite starts with a failing test for the bug being fixed). The frameworks are the harnesses; TDD is the discipline.
- **The banking context** — the MFT/payment-file scenario in §7 deliberately mirrors the transfer-automation scripts guarded by shell tests in the bank series ([../banking/nets_software_systems_guide.md](../banking/nets_software_systems_guide.md) and siblings): payment-format validation, currency/amount checks, checksums, file transfer pre-checks. [../management/business_case_development_guide.md](../management/business_case_development_guide.md) supplies the adoption-cost lens applied in §6.4.

**Note on verification.** This guide was researched in August 2026. Claims are marked **Verified** (confirmed against primary sources during research — repository pages, READMEs, the GitHub API, testanything.org), **Reported** (widely cited but not independently confirmed), or **flagged** inline where a fact could not be pinned down. The two flags the reader should carry forward: (1) **assert.sh's author is Robert Lehmann, not "Robert Böhnke"** as sometimes cited; (2) **"bashaspec" as an RSpec-style framework could not be verified against any current primary source** — see §2.5. Where this guide inherits a fact from a sibling, it cites the sibling rather than re-verifying.

**Reader paths.** *Architect choosing a framework for a new repo* — read §1, §3, §6, then §7's lessons. *Engineer standing up tests for legacy ops scripts* — read §2.3, §2.4, §7.4, §7.5. *Regulated-bank CI owner* — read §3.3 (CI row), §4, §6. *The one-paragraph answer* — §8, which is deliberately self-contained.

---

## 1. Overview — The Five-Plus Frameworks

### 1.1 The Frameworks

Bash has **no built-in testing framework** — the ecosystem is entirely community-driven (Verified — the umbrella guide's §1 states this plainly, and the six projects below all predate and postdate each other with no standardisation effort). The frameworks compared here:

1. **shUnit2** — the xUnit veteran (JUnit/PyUnit ported to the Bourne shell), maintained by Kate Ward, ~2008-era lineage, POSIX-compatible.
2. **Bats / bats-core** — the Bash Automated Testing System: TAP output, created by Sam Stephenson in 2011, community-forked as bats-core in 2017. The de facto standard and the largest ecosystem.
3. **bash_unit** — a lightweight **single-file** framework (assertions + a built-in `fake` mocking primitive), authored by Pascal Grange, first committed mid-2011, popular in France-adjacent and Homebrew circles.
4. **assert.sh** — a minimal assertion library ("test-driven development in the Bourne again shell") by Robert Lehmann, 2009: two assertion functions, zero framework machinery.
5. **bashaspec** — *flagged*: the name as an RSpec-style `describe`/`it` framework **could not be verified** against any current primary source (§2.5); the only live "bashaspec" on GitHub (d10n/bashaspec, 2020) is a TAP-compliant, `test_*`-function library. It is profiled as documented, with the caveat made explicit.
6. **ShellSpec** — the modern BDD framework (2019, Koichi Nakashima): full-featured — coverage, mocking, parameterized tests, parallel — for bash, ksh, zsh, dash and all POSIX shells.
7. **shpec** — the other RSpec-style contender (2013, rylnd): `describe`/`it` with assert-matchers and command stubbing.
8. **The bats ecosystem** — bats-support, bats-assert, bats-file, bats-mock: the helper libraries that make bare Bats production-grade (§4).

Plus the surrounding tools this guide does **not** profile (cross-referenced, not compared): **ShellCheck** (static analysis — a complement, not a competitor), **shfmt** (formatting), **kcov** (coverage), **zunit/zsh-test-runner** (zsh-only), **zicher** (minimal single-file; profiled in the umbrella's §2), and **pytest-from-Python** (driving shell via `subprocess` — see bats_core_guide §8).

### 1.2 The Three Lineages: xUnit, TAP, BDD

Every framework in this comparison descends from one of three stylistic lineages, and the lineage predicts most of what the framework gives and costs you:

- **xUnit (shUnit2)** — JUnit's shape ported to shell: test functions discovered by name, `setUp`/`tearDown` lifecycle, a family of `assertEquals`-style macros, and a human-oriented dot/summary report. **What you get:** familiarity for anyone who has written JUnit; POSIX portability (runs under dash, busybox, ksh). **What you give up:** no TAP, no structured output capture, sequential execution.
- **TAP (Bats, bash_unit's `-f tap`, assert.sh's lineage, bashaspec's d10n incarnation)** — the **Test Anything Protocol**, born inside Perl's test harness: earliest recorded `t/TEST` commit by **Larry Wall dated January 30, 1988** (Perl 1.0 era; the protocol is "around since 1988" per testanything.org — Verified). TAP is a plain-text stream — `1..N` plan line, `ok N` / `not ok N` lines, `#` diagnostics — that **decouples test reporting from presentation**: any TAP consumer (Jenkins TAP plugin, GitLab, prove, formatters) can read any TAP producer. **What you get:** machine-readable output, CI-friendliness, ecosystem interoperability. **What you give up:** nothing much — TAP is a floor, not a ceiling; every modern harness layers prettier output on top.
- **BDD (bashaspec-as-described, shpec, ShellSpec)** — RSpec's shape ported to shell: `describe`/`it` (or `Describe`/`It`) blocks that read as specifications, matcher-based assertions ("the output should equal…"), and stubbing primitives. **What you get:** test-as-spec readability, rich failure messages, behavior-driven framing that maps to requirements. **What you give up:** a steeper DSL learning curve than Bats' "just Bash plus six built-ins" (the bats_core_guide §8 makes exactly this point about ShellSpec).

The family tree matters for adoption: an xUnit shop adopts shUnit2 fastest; a CI-platform team adopts Bats fastest; a product-requirements team adopts a BDD harness fastest.

### 1.3 The Overview Table

The one-table answer to "what is each framework, in one breath" — expanding the umbrella guide's §8 two-way table to the full field. The umbrella's row-by-row detail (setup naming, output capture, discovery, TAP, parallel, CI, community, shell compatibility, activity, dependencies, learning curve) is the authoritative comparison of the two mainstream frameworks; this table adds the four other players and the ecosystem:

| Aspect | shUnit2 | Bats / bats-core | bash_unit | assert.sh | bashaspec (as documented) | ShellSpec |
|---|---|---|---|---|---|---|
| **One-line identity** | JUnit for the Bourne shell | The Bash Automated Testing System | Single-file "enterprise edition" for professionals | Two-assertion TDD-in-bash library | TAP-compliant test_*-function library (d10n; see §2.5 flag) | Full-featured BDD for all POSIX shells |
| **Stylistic lineage** | xUnit | TAP | TAP (optional) / custom | TAP-flavoured custom | TAP | BDD (RSpec-inspired) |
| **First commit** | ~2008 (Reported; repo on GitHub since 2015, Google Code before) | Dec 28, 2011 (Verified) | Jul 13, 2011 (Verified) | Nov 15, 2009 (Verified) | Jul 23, 2020 (Verified — d10n) | Feb 5, 2019 (Verified) |
| **Primary maintainer** | Kate Ward | bats-core org (fork of Sam Stephenson's Bats) | Pascal Grange | Robert Lehmann | d10n | Koichi Nakashima |
| **Test discovery** | `test*` function names | `@test "name" { }` blocks | `test*` function names | manual sequence + `assert_end` | `test_*` functions, run by file/glob | `Describe`/`It` blocks |
| **Output** | human summary (+ unreleased JUnit XML) | TAP native + pretty + JUnit formatters | human, TAP via `-f tap` | per-suite statistics, `-v` verbose | TAP-compliant | human spec output + reporters |
| **License** | Apache-2.0 (repo LICENSE; Verified) | MIT-style (Verified) | GPL-3.0 (Verified) | LGPLv3 (Verified) | MIT (d10n; Verified) | MIT (Verified) |
| **GitHub stars (Aug 2026)** | ~1.7k (Verified) | ~6.2k (Verified) | ~635 (Verified) | ~491 (Verified) | ~37 (Verified) | ~1.4k (Verified) |
| **Ecosystem** | small (none official) | largest (bats-assert/support/file/mock, detik, action) | pre-commit hook, Homebrew/nixpkgs/AUR | stub.sh (external, plays well) | none | kcov integration, getoptions family |
| **Cross-ref** | umbrella §2, §8 | umbrella §8, §9; bats_core_guide (all) | — | bats_core_guide §8 ("not a framework") | — | bats_core_guide §8 ("main alternative") |

### 1.4 Verification Notes

- **shUnit2's "~2008" lineage is Reported, not directly verified.** The GitHub repository was created June 26, 2015 (Verified — GitHub API) and the README documents that the project predates GitHub (it lived on Google Code; it was "originally developed to provide a consistent testing solution for log4sh" — Verified, README). The sibling bats_core_guide §8 independently dates it "2008-era". We therefore use "~2008 (Reported; corroborated by the sibling guide)".
- **Bats dates are Verified** from the GitHub API (sstephenson/bats created Dec 28, 2011) and the bats-core README (forked Tuesday, September 19, 2017 at commit 0360811 via `git clone --bare` + `git push --mirror`; original archived April 29, 2021; "© 2011–2016 Sam Stephenson"; "© 2017–2024 bats-core organization").
- **TAP's origin is Verified** from testanything.org's own history page: "The protocol has been around since 1988," first commit `840163b` by Larry Wall dated Jan 30, 1988, implemented as Perl 1.0's `t/TEST`. (The "1987" date sometimes cited refers to the Perl 1.0 release era; the protocol's earliest recorded artifact is 1988.)
- **bashaspec's RSpec-style identity is NOT verified** — see §2.5 for the full caveat.
- Star counts are point-in-time (Aug 2026) and drift; they are cited as rough adoption signals only.

---

## 2. Framework Profiles

### 2.1 shUnit2 — The xUnit Veteran

**Identity:** "shUnit2 is a **xUnit based unit test framework for Bourne based shell scripts**, and it is designed to work in a similar manner to JUnit, PyUnit, etc." (Verified — README, kward/shunit2). It is the oldest framework in this comparison and the only one that is genuinely POSIX-first: tested against **sh, bash, dash, ksh, mksh, and zsh** (zsh requires `setopt shwordsplit`; Verified — README), and reported working on Ubuntu, macOS, FreeBSD, Solaris 8/9/10, and Cygwin.

**Origin:** written by **Kate Ward** (kward) to give **log4sh** — her shell logging framework, "similar to log4j" — a consistent cross-shell test bed, after repeated "works under /bin/bash on Linux, breaks under /bin/sh on Solaris" incidents (Verified — README's Background section). The project migrated from Google Code to GitHub (repo created 2015); last tagged release **v2.1.8, March 29, 2020** (Verified — GitHub releases), with commits still landing in 2026 (Verified — commit history).

**Shape of a test suite** (the umbrella guide §2's example, verbatim style):

```bash
#!/bin/sh
testEquality() { assertEquals 1 1; }
testContains() { assertContains "hello world" "world"; }
. ./shunit2    # Must come last — this line runs the suite
```

- **Discovery:** any function whose name starts with `test` is a test case; `setUp`/`tearDown` run around each (the umbrella's §8 row: `setUp()`/`tearDown()` vs Bats' `setup()`/`teardown()` — casing differs).
- **Assertions:** `assertEquals`, `assertNotEquals`, `assertNull`, `assertNotNull`, `assertContains`, `assertNotContains`, `assertTrue`, `assertFalse` (Verified — umbrella §2 and shunit2 docs); plus failure/skip helpers (`fail`, `startSkipping`, `endSkipping`).
- **Output capture is manual:** `result=$(cmd)` — there is no `run`-style capture built in (umbrella §8 row), which the assert.sh README also criticises ("extra care when crafting test cases with many subprocess invocations… you have to fall back to shell features").
- **Output:** human-oriented summary (`Ran 2 tests… OK`); no native TAP. A **JUnit XML formatter** exists in the source tree but is documented in the README as "still unreleased" (Verified — README "Generating test results in JUnit format").
- **Parallel execution:** none — suites run sequentially (umbrella §8 row).
- **Why it survives:** zero dependencies beyond the shell itself (the umbrella §8 row adds `expr`), POSIX portability that no other framework here matches except ShellSpec, and a learning curve of minutes for anyone who has seen JUnit. Debian/Ubuntu ship it (`apt install shunit2`).

### 2.2 Bats / bats-core — The TAP Standard

**Identity:** "Bash Automated Testing System" — a **TAP-producing testing framework for Bash**, written entirely in Bash (Verified — repo, and bats_core_guide §1). It gives shell scripts what JUnit gives Java: named test cases, lifecycle hooks, structured results, CI wiring.

**Origin — the 2011 birth and the 2017 fork (Verified):**
- **December 28, 2011** — **Sam Stephenson** (creator of rbenv) publishes `sstephenson/bats`. Bats 0.4.0 is the last release under him; development stalls (there was an initial call for maintainers — issue #150 — but write access to the original repo could not be obtained; Verified — bats-core README's "Why was this fork created?").
- **September 19, 2017** — the **bats-core organization** forks the project (`git clone --bare` + `git push --mirror` at commit 0360811) to keep it alive. This is the "Bats vs bats-core" question in one line: **Bats is the 2011 original (now archived, read-only since April 29, 2021); bats-core is the community-maintained continuation everyone actually uses.** All modern features below are bats-core's.
- **2019** — v1.0.0: API stabilization, major performance boost, Bash 4.2+ support (Verified — bats_core_guide §2 history). Current line: **v1.14.0 released July 2026** (Verified — repo commits/CHANGELOG).

**Shape of a test suite:**

```bash
#!/usr/bin/env bats

setup() { export TEST_DIR=$(mktemp -d); }
teardown() { rm -rf "$TEST_DIR"; }

@test "greeting returns hello" {
  result="$(./script.sh greet world)"
  [ "$result" == "Hello, world!" ]
}

@test "greeting fails without arguments" {
  run ./script.sh greet
  [ "$status" -eq 1 ]
  [[ "$output" == *"Usage:"* ]]
}
```

- **Discovery:** `@test "name" { }` blocks; every test runs in its own subshell (isolation by construction — bats_core_guide §4.2).
- **The `run` built-in:** captures stdout/stderr into `$output`, exit code into `$status`, line-split into `$lines` — the output-capture row where Bats beats shUnit2 (umbrella §8).
- **Lifecycle:** `setup`/`teardown` per test; `setup_file`/`teardown_file` per file (bats 1.x; the bats guide dates file-level hooks to 1.2+).
- **Output:** TAP natively, plus formatters (`--formatter tap|pretty|junit`); `--jobs N` for parallel file execution; `skip`, `--filter`, tags, failure hooks in modern releases.
- **Assertions:** bare Bats has none — you assert with `[ ]` or `[[ ]]`; the **bats-assert/bats-file/bats-mock ecosystem** (§4) supplies the readable assertion layer. This is the umbrella §8's "Assertions: via helpers (bats-assert, bats-file)" row.
- **Why it won:** TAP-native CI integration, the biggest community (~6.2k stars, 2,041 commits), the helper ecosystem, and the minimal "just Bash" syntax. The umbrella §9 recommends it for essentially every new-project scenario.

### 2.3 bash_unit — The Lightweight Single-File

**Identity:** "**bash_unit** — bash unit testing enterprise edition framework for professionals!" (Verified — README, bash-unit/bash_unit; the tagline is tongue-in-cheek). A **single-file framework** — the whole runner is one `bash_unit` script — where **test cases are functions starting with `test`**, run with stack traces pointing at the exact source file and line on failure.

**Origin:** repo created **July 13, 2011** (Verified — GitHub API), authored by **Pascal Grange** (pgrange), now under the bash-unit organization. *French origin (Reported):* the author is French, the README's GitHub Actions example links to the **French** GitHub documentation (`docs.github.com/fr/actions` — Verified), and the companion "getting started" project lives on GitLab under `pgrange` (Verified — README links). None of this is a formal "origin" statement, so the French-origin claim is flagged Reported-but-plausible.

**Shape of a test suite:**

```bash
#!/bin/bash
test_valid_payment() {
  assert_equals 0 "$(validate_payment_file test/fixtures/valid.pay)"
}
test_rejects_negative_amount() {
  assert_status_code 1 validate_payment_file test/fixtures/negative.pay
}
setup() { :; }   # optional per-test setup
```

- **Assertions (Verified — README):** `fail`, `assert`, `assert_fail`, `assert_status_code`, `assert_equals`, `assert_not_equals`, `assert_matches`, `assert_not_matches`, `assert_within_delta`, `assert_no_diff`, `skip_if` — a genuinely rich macro set for a single file.
- **Mocking is built in:** `fake <command> [replacement]` replaces any command (even builtins) for the duration of a test — the README shows faking `ps` with a function, with the `coproc` trick for capturing what the fake was called with (Verified — README's extensive "how to run tests" section). This is the one framework besides ShellSpec with **first-class mocking in the core**.
- **CLI (Verified — README):** `-p pattern` filter tests, `-s pattern` skip, `-r` randomise order within a file, `-f tap` for TAP output, `-q` quiet mode.
- **Distribution:** `install.sh` one-liner, Homebrew (`brew install bash_unit`), nixpkgs, Arch AUR, a **pre-commit hook** (`repo: https://github.com/bash-unit/bash_unit`, id `bash-unit`), and documented **GitHub Actions and GitLab CI** snippets (Verified — README).
- **Why it's notable:** the best "small but complete" story — one file, no install ceremony, real assertions, real mocking, TAP when you need it. 635 stars and a 2026-active commit history (Verified — pushed Feb 2026).

### 2.4 assert.sh — The Minimal Assertion Library

**Identity:** "**assert.sh** is test-driven development in the Bourne again shell" (Verified — README, lehmannro/assert.sh). Version 1.1, author **Robert Lehmann**, LGPLv3. Not a framework in the discovery/lifecycle sense — the bats_core_guide §8 says it plainly: *"a minimal assertion library (`assert`, `assert_raises`, …), **not a framework**: no test discovery, no TAP, no lifecycle hooks. A helper for hand-rolled test scripts."* This guide agrees, and profiles it anyway because it is the canonical answer to "I just want assertions, nothing else."

**⚠ Verification flag:** the task brief named the author "Robert Böhnke"; the primary source says **Robert Lehmann** (Verified — README header: "Author: Robert Lehmann"). The Böhnke attribution could not be found in any source consulted and appears to be a misattribution — this guide uses Lehmann.

**Shape of a test suite:**

```bash
. ./assert.sh
assert "echo test" "test"                 # command + expected stdout
assert_raises "false" 1                   # command + expected exit code
assert_end examples                       # closes the suite, prints stats
```

- **The entire API (Verified — README reference):** `assert <command> [stdout] [stdin]` — run a command, compare its stdout (supports `\n` escapes); `assert_raises <command> [exitcode] [stdin]` — check the exit code; `assert_end [suite]` — close and print statistics; `skip` / `skip_if <command>` — skip the next test, unconditionally or on a precondition.
- **Runner options (Verified):** `-v/--verbose`, `-x/--stop` (stop on first failure), `-i/--invariant` (no timing), `-d/--discover` (collect only), `-c/--continue` (don't touch exit code); env-var equivalents `$DEBUG`, `$STOP`, `$INVARIANT`, `$DISCOVERONLY`, `$CONTINUE`. The suite's exit status is set automatically to reflect failures.
- **Origin:** repo created **November 15, 2009** (Verified — GitHub API); last push January 2022 (Verified) — effectively dormant, though "done" for its scope.
- **Companions (Verified — README's Related projects):** **stub.sh** (jimeh/stub.sh) — "fake binaries and bash builtins… supports mocking features such as expecting a certain number of invocations and plays well with assert.sh"; the README also positions itself against ShUnit, shUnit2, tap-functions, and bats ("Another TAP producer with syntactic sugar… depends on `errexit` environments (set -e)").
- **Why it exists:** modifying process state (stdin, argv, environment) is awkward in most languages and natural in shell — so "why don't we run the tests in your shell?" (Verified — README's Use case). For 5–20 quick checks of an ops script with no framework ceremony, this is still a defensible choice; for anything with real structure, pick a framework.

### 2.5 bashaspec — The Verification Caveat

**The flag, stated plainly:** the brief asked for "bashaspec — the RSpec-style describe/it framework". **Research could not verify any current primary source for an RSpec-style framework named bashaspec.** Specifically:

- `github.com/jmervine/bashaspec` returns **404** (the account is Joshua Mervine; no such repository exists or ever appears in the GitHub API, and the Wayback Machine has **zero snapshots** of it — Verified via the Wayback CDX API).
- npm has no `bashaspec` package (Verified — registry 404).
- The **only** live GitHub project named bashaspec is **d10n/bashaspec** (created July 23, 2020, ~37 stars, MIT — Verified via GitHub API and README): a "Short, self-contained, and actually useful unit testing library for bash, zsh, ksh, dash, and sh" — **TAP-compliant, driven by `test_*` functions, with `before_all`/`after_all`/`before_each`/`after_each` hooks, under 100 lines, vendor-able by copying one file.** It is explicitly *not* RSpec-style: no `describe`/`it`, no matcher DSL — "Assertions: no need to reinvent the wheel; just use the `test` or `[[ ]]` commands" (Verified — README).
- The RSpec-style (`describe`/`it`) niche in shell testing is in fact occupied by **shpec (2013)** and **ShellSpec (2019)** — both verified below.

**Consequence for this guide:** the bashaspec profile below documents the framework **as it exists** (the d10n library), the decision matrix scores that documented reality, and the worked example (§7.6) writes the suite in that style. Readers who came looking for an RSpec-style "bashaspec" should read §2.6–§2.7 instead — that is where the `describe`/`it` behaviour lives in the real world.

**Profile (d10n/bashaspec, as documented — Verified):** source the library after defining `test_*` functions; run `./bashaspec.sh` to discover all spec files in the directory (or run a single `*_spec.sh` file). Passing-test output is hidden; failures show the failing command's output; verbose and tiny output modes; TAP-compliant; works with `dash` — tests can be written in any POSIX-compatible shell. Strengths: genuinely tiny, readable-at-a-glance, POSIX-wide. Limits: no mocking primitive, no test filtering beyond file selection, no lifecycle beyond the four hooks, no parallel, no CI formatters beyond TAP.

### 2.6 ShellSpec — The Modern BDD Framework

**Identity:** "A **full-featured BDD unit testing framework** for dash, bash, ksh, zsh and **all POSIX shells** that provides first-class features such as code coverage, mocking, parameterized test, parallel execution and more… released in 2019" (Verified — README, shellspec/shellspec). Author **Koichi Nakashima** (ko1nksm), MIT, repo created **February 5, 2019** (Verified — GitHub API), ~1.4k stars, 1,976 commits. The README's "Inspired frameworks" list (Verified) is a who's-who of BDD: **RSpec, Jest, Mocha, Jasmine, Ginkgo, JUnit 5**.

**Shape of a test suite:**

```bash
Describe 'validate_payment_file'
  It 'accepts a valid payment file'
    When call validate_payment_file test/fixtures/valid.pay
    The status should be success
    The output should equal "OK"
  End
End
```

- **The BDD layer (Verified — README/shellspec.info):** `Describe`/`Context`/`It` blocks; the "syntax sugar" layer turns `When call …` / `The output should …` / `The status should be success` into an executable specification. There is also a plain `expect`-style layer for non-DSL users.
- **First-class features (Verified):** code coverage via **kcov** integration, **mocking** (`mock`/`stub` helpers in the `stub/` directory), **parameterized tests**, **parallel execution**, random ordering, per-file/per-test hooks, and a `--sandbox` mode. CI-friendly reporters (tap, junit, documentation).
- **Why it matters:** it is the only framework that combines BDD syntax **and** POSIX-wide portability **and** enterprise features — the bats_core_guide §8's verdict: *"the main alternative… more expressive for large suites; supports sh/bash/ksh/dash. But smaller community, fewer ecosystem libraries, steeper learning curve."* For a bank with Solaris/AIX ksh boxes, it is the serious Bats alternative.

### 2.7 shpec — The Other RSpec-Style Contender

**Identity:** "**shpec** — Test your shell scripts!" (Verified — README, rylnd/shpec). Repo created **January 27, 2013** (Verified — GitHub API), ~386 stars, POSIX-shell core. The README describes it as "similar to other BDD frameworks like RSpec, Jasmine, and mocha"; the two main constructs are **`describe`/`end`** (grouping) and **`it`/`end`** (an individual test), and test files are *sourced into* shpec so any shell command is available.

**Matchers (Verified — README):** binary — `equal`, `unequal`, `gt`, `lt`, `glob`, `no_glob`, `grep`, `no_grep`, `egrep`, `no_egrep`; unary — `present`, `blank`, `file_present`, `file_absent`, `symlink`, `test`; plus **custom matchers** auto-loaded from `shpec/matchers/*.sh` (the README's `still_alive` example pings a host and asserts on `$?`). **Stubbing** via `stub_command name [body]` / `unstub_command`. Runs with `shpec [files]`; integrates with `entr` for watch mode.

shpec is included as the "and etc." sixth framework because it is the historical bridge between the RSpec idea and shell — and because the unverifiable "bashaspec" description (describe/it) matches *this* project's shape almost exactly. Its limits: no TAP output, no built-in parallel, no coverage, and maintenance has been quiet since ~2022 (Verified — last push Dec 2022). For new work, ShellSpec is the actively-maintained version of the same idea.

### 2.8 The Profile Table

| Framework | Origin (Verified unless flagged) | Style | Distribution | Current status (Aug 2026) |
|---|---|---|---|---|
| **shUnit2** | ~2008 (Reported; Google Code → GitHub 2015), Kate Ward | xUnit | `apt install shunit2`, GitHub, distro packages | Stable/legacy; last release v2.1.8 (Mar 2020), occasional commits |
| **Bats** (original) | Dec 28, 2011, Sam Stephenson | TAP | GitHub (npm `bats`, distros) | **Archived** Apr 29, 2021 — read-only |
| **bats-core** | Sep 19, 2017 community fork | TAP | `apt`/`brew`/`npm`, Docker, Homebrew tap | Active; v1.14.0 (Jul 2026), ~6.2k stars |
| **bash_unit** | Jul 13, 2011, Pascal Grange | Custom + TAP (`-f tap`) | `install.sh`, Homebrew, nixpkgs, AUR, pre-commit | Active (pushed Feb 2026), ~635 stars |
| **assert.sh** | Nov 15, 2009, Robert Lehmann | Assertion library (TAP-flavoured custom) | wget/`bpkg` single file | Dormant (last push Jan 2022), ~491 stars |
| **bashaspec** | Jul 23, 2020, d10n (⚠ see §2.5) | TAP, `test_*` functions | single vendorable file | Small/niche, ~37 stars |
| **ShellSpec** | Feb 5, 2019, Koichi Nakashima | BDD (`Describe`/`It` DSL) | `install.sh`, Homebrew, Docker | Active, ~1.4k stars |
| **shpec** | Jan 27, 2013, rylnd | BDD (`describe`/`it` + matchers) | curl installer, antigen (zsh) | Quiet since 2022, ~386 stars |


---

## 3. Feature Matrix

### 3.1 The Styles: xUnit, TAP, BDD

The single most consequential difference between these frameworks is the **stylistic contract** each one makes with the test author — it determines how tests read, how failures are reported, and how much DSL you must learn.

- **xUnit style (shUnit2).** Test cases are functions; assertions are macros that never abort the suite (each failure is recorded, the next test still runs). Familiar to anyone who has written JUnit; the umbrella guide's §8 row-by-row comparison with Bats is the reference treatment of this style's trade-offs.
- **TAP style (Bats, bash_unit `-f tap`, assert.sh's output, d10n bashaspec).** Tests produce a `1..N` plan plus `ok N`/`not ok N` lines. TAP is the interoperability layer: *"It decouples the reporting of errors from the presentation of the reports"* (Verified — testanything.org). Every CI system worth naming consumes TAP natively or via a plugin; producers and consumers need not share a language (Verified — testanything.org).
- **BDD style (ShellSpec, shpec; the unverified "bashaspec" description).** `describe`/`it` blocks plus matcher-based assertions make the suite read as an executable requirements document. ShellSpec's `Describe`/`It`/`Context` + `The output should …` is the fullest expression; shpec's `assert equal`/`assert file_present` is the lighter one.

The lineage also predicts the failure-message quality: bare Bats gives you "`[ "$x" == "y" ]` failed at line N"; bats-assert and the BDD matchers give you "expected: X, actual: Y". For a regulated-bank audit trail, the latter is materially better — the worked example's outputs (§7.8) show the difference.

### 3.2 The Capabilities

**Assertions.** What you can say, and how readable the failure is:
- shUnit2: rich built-in macro set (`assertEquals`, `assertContains`, `assertTrue`…).
- Bats: none built in — raw `[ ]`/`[[ ]]` or the bats-assert library.
- bash_unit: ten built-in assertion functions plus `fail`; regex (`assert_matches`), delta (`assert_within_delta`), and diff (`assert_no_diff`) variants are unusual for a single file.
- assert.sh: exactly two — `assert` (stdout) and `assert_raises` (exit code). By design.
- bashaspec (d10n): none — "just use the `test` or `[[ ]]` commands" (Verified — README).
- ShellSpec: the expectation DSL (`The status should be success`, `The output should equal`…) plus a plain `expect` layer.

**Mocking.** The ability to replace external commands and verify their usage:
- Bats has no core mocking; the ecosystem provides it — PATH-manipulation stubs by hand (umbrella §4.1), `bats-mock`'s `stub`/`unstub` (umbrella §4.1 Method 3), or `bats-detik`.
- bash_unit: **`fake` is built into the core** — replace any command, even builtins, for the duration of a test (Verified — README, with worked `ps` and `coproc` examples).
- ShellSpec: first-class `mock`/`stub` helpers (Verified — README and `stub/` directory).
- shpec: `stub_command`/`unstub_command` (Verified — README).
- assert.sh: none inside; the README points to **stub.sh** as the companion ("plays well with assert.sh" — Verified).
- shUnit2: none — you hand-roll PATH or function overrides (the umbrella's §4.1 Method 1/2 patterns work with any framework, but shUnit2 offers no helper).
- bashaspec (d10n): none documented.

**Fixtures & lifecycle.** Per-test and per-suite setup/teardown:
- shUnit2: `setUp`/`tearDown` per test (umbrella §8 row).
- Bats: `setup`/`teardown` per test, `setup_file`/`teardown_file` per file (bats 1.x; umbrella §8 row "1.7+" vs bats guide "1.2+" — the practical point is file-level hooks exist in modern bats), per-test temp dirs via `$BATS_TEST_TMPDIR` (bats guide §2: v1.5.0).
- bash_unit: `setup` per test (README's faking examples use it to clean `/tmp` state).
- bashaspec (d10n): `before_all`/`after_all`/`before_each`/`after_each` — the four-hook full set (Verified — README).
- ShellSpec: `BeforeEach`/`AfterEach`/`BeforeAll`/`AfterAll` blocks inside `Describe` (Verified — docs).
- assert.sh: none — you manage state inline; the "fixtures" row is where assert.sh's "not a framework" status bites (bats_core_guide §8's phrasing).

**CI integration.** Exit codes, TAP, JUnit, CI snippets:
- Bats: native TAP + JUnit formatter + exit-code contract — the best-in-class story (umbrella §8 row "Excellent (native TAP)").
- bash_unit: documented GitHub Actions and GitLab CI workflows plus a pre-commit hook (Verified — README).
- ShellSpec: reporters (tap/junit/documentation), CI badge matrix across Ubuntu/macOS/FreeBSD/NetBSD/OpenBSD/DragonFly/Solaris (Verified — README badges).
- shUnit2: exit codes only, or the unreleased JUnit-XML generator; TAP needs a wrapper (umbrella §8 row).
- assert.sh: exit code reflects suite status automatically (Verified — README) — enough for a CI step, nothing more.
- bashaspec (d10n): TAP-compliant output — consumable by any TAP-aware CI step.

**Coverage.** Line coverage of the script under test:
- ShellSpec: first-class **kcov** integration (`--kcov` options, `.codecov.yml` in-repo; Verified).
- Bats: kcov works well (`kcov coverage bats tests/` — umbrella §5/§7.1), plus bashcov; but it is external tooling, not framework-integrated.
- shUnit2 / bash_unit / assert.sh / bashaspec: no integrated coverage story; kcov or nothing. (bash_unit's README mentions no coverage; it is not a criticism — single-file scope.)

**Parallel execution.**
- Bats: `--jobs N` runs `.bats` files in parallel (needs GNU `parallel`; umbrella §8 row; bats guide §2: v1.6.0).
- ShellSpec: parallel execution built in (Verified — README).
- bash_unit: `-r` randomises within a file; files run sequentially (Verified — README).
- shUnit2, assert.sh, bashaspec, shpec: sequential only.

### 3.3 The Matrix Table

Feature × framework, with notes. "✅" = first-class, "🟡" = via ecosystem/external, "❌" = absent. This table is the §8 umbrella table generalised: every row the umbrella compared for bats-vs-shunit2 is here for all six.

| Feature | shUnit2 | Bats/bats-core | bash_unit | assert.sh | bashaspec (d10n) | ShellSpec |
|---|---|---|---|---|---|---|
| **Style** | xUnit | TAP | custom + TAP (`-f tap`) | assertion lib | TAP | BDD (`Describe`/`It`) |
| **Test discovery** | `test*` names | `@test` blocks | `test*` names | manual + `assert_end` | `test_*` names | `Describe`/`It` blocks |
| **Assertion macros** | ✅ rich built-in | 🟡 via bats-assert | ✅ 10 built-in | ✅ 2 (`assert`, `assert_raises`) | ❌ raw `[[ ]]` | ✅ expectation DSL |
| **Output capture** | ❌ manual `$( )` | ✅ `run` → `$status/$output/$lines` | ✅ via assert helpers | ✅ `assert` compares stdout | ❌ manual | ✅ `When call`/`The output` |
| **Setup/teardown** | ✅ `setUp`/`tearDown` | ✅ `setup`/`teardown` (+file-level) | ✅ `setup` | ❌ | ✅ 4 hooks (before/after, all/each) | ✅ `BeforeEach`/`AfterEach`/… |
| **Mocking** | ❌ hand-rolled | 🟡 bats-mock / PATH stubs | ✅ built-in `fake` | 🟡 stub.sh (external) | ❌ | ✅ first-class `mock`/`stub` |
| **Fixtures/temp dirs** | 🟡 manual | ✅ per-test tmpdir, fixture helpers | 🟡 manual | ❌ | 🟡 manual | ✅ parameterized + fixture dirs |
| **TAP output** | ❌ (JUnit XML unreleased) | ✅ native | ✅ `-f tap` | 🟡 custom-ish | ✅ TAP-compliant | 🟡 tap reporter |
| **JUnit/CI formats** | 🟡 unreleased XML | ✅ junit formatter | ❌ (TAP only) | ❌ | ❌ | ✅ junit reporter |
| **Parallel** | ❌ | ✅ `--jobs N` | ❌ (`-r` random only) | ❌ | ❌ | ✅ built-in |
| **Coverage** | ❌ | 🟡 kcov/bashcov (external) | ❌ | ❌ | ❌ | ✅ kcov integrated |
| **Filter/skip** | 🟡 skip helpers | ✅ `--filter`, `skip` | ✅ `-p`/`-s` patterns, `skip_if` | ✅ `skip`/`skip_if` | ❌ | ✅ filters + `skip` |
| **CI docs** | 🟡 generic | ✅ first-class | ✅ GH Actions + GitLab + pre-commit | 🟡 exit codes | 🟡 TAP | ✅ reporters + OS matrix |
| **POSIX shells** | ✅ sh/bash/dash/ksh/zsh | 🟡 bash 4.2+ (zsh partial) | 🟡 bash-centric | ✅ Bourne-again family | ✅ sh/dash/ksh/zsh | ✅ all POSIX |
| **Learning curve** | very low | low | low | minimal | minimal | medium-high (DSL) |

*Notes: (1) The "POSIX shells" row is where the umbrella §8's "bash 4.x+, zsh" vs "bash, zsh, ksh, dash (POSIX)" contrast extends across the field — shUnit2 and ShellSpec are the portability pair, bash_unit sits between (bash-native syntax, but the runner is bash-friendly), Bats is bash-only by design. (2) "Coverage" for Bats is external but well-trodden — the umbrella's §7.1 kcov recipe is the canonical one. (3) The learning-curve row quotes the bats_core_guide §8 verdict on ShellSpec ("Medium–high (new DSL)") and the umbrella §8 on shUnit2 ("Very low").*

---

## 4. Ecosystems

### 4.1 The Bats Ecosystem: bats-support, bats-assert, bats-file, bats-mock

The reason "Bats vs the field" is not a fair fight is the **bats-core organization's helper stack** — the umbrella guide's §2 calls them "Community helpers" and the bats_core_guide dedicates a full section to them. Verified inventory (GitHub API + READMEs, Aug 2026):

- **bats-support** (bats-core/bats-support; originally ztombol/bats-support, Feb 2016) — "Supporting library for Bats test helpers": shared output-formatting machinery (the `--` failure banner style) that bats-assert builds on. The load order in every bats file: `load 'bats-support/load'; load 'bats-assert/load'` (umbrella §4.1 Method 3).
- **bats-assert** (bats-core/bats-assert; forked from ztombol/bats-assert, Feb 2016; CC0-1.0) — "Common assertions for Bats": `assert`/`refute`, `assert_equal`/`assert_not_equal`, `assert_success`/`assert_failure`, `assert_output`/`refute_output` (with `--partial`, `--regexp`, `--index`), `assert_line`/`refute_line`, `assert_regex`, and the newer stderr family (`assert_stderr`, `assert_stderr_line`) (Verified — README's full assertion API). Failure output is formatted and sent to stderr so assertions work outside `@test` blocks (Verified — README).
- **bats-file** (bats-core/bats-file; originally ztombol/bats-file, 2016; repo in org since Sep 2018) — "Common filesystem assertions for Bats": `assert_file_exist`, `assert_dir_exist`, `assert_file_contains`, `assert_file_empty`, `assert_file_size` and friends (Verified — README; umbrella §2 lists the key ones).
- **bats-mock** (jasonkarns/bats-mock, Nov 2015) — "Mocking/stubbing library for BATS": `stub`/`unstub` with call-count verification; the umbrella §4.1 Method 3 uses it for "deploy calls git push" (Verified — umbrella; `unstub` verifies all stubs were called).
- Also in the org: **bats-detik** (2019, "e2e tests of applications in Kubernetes" style helpers), **bats-extra**, **bats-vscode**, **bats-action** (GitHub Action), **homebrew-bats-core** (Verified — org repo list).

The lineage matters: bats-assert and bats-support were written by **Zoltán Tömböl** (ztombol) and migrated into the bats-core org — the standard move of a community-standardising on one umbrella. Nothing comparable exists for shUnit2 (no official helper org), bash_unit (single-file by design), assert.sh (stub.sh is the only named companion), or ShellSpec (whose features are in-core instead of in-satellites).

### 4.2 Everything Else

- **shUnit2 extras:** the repo ships `test_runner`, `shunit2_test_helpers`, and JUnit-XML support in the source tree (Verified — repo file listing) — but there is no third-party ecosystem to speak of; the "ecosystem" is the distro packages and its 21 contributors (Verified).
- **bash_unit ecosystem:** pre-commit hook, Homebrew formula, nixpkgs package, AUR package, a getting-started GitHub project and a getting-started GitLab project (Verified — README). Pragmatic, small, complete.
- **ShellSpec ecosystem:** kcov integration and the author's companion libraries — **getoptions** (option parser), **readlinkf**, **portable-echo** (Verified — README's related projects) — plus Docker images and CI across eight OSes (Verified — README badges).
- **shpec ecosystem:** the wiki examples page, `entr` watch-mode integration, custom matchers via `shpec/matchers/*.sh`, an antigen zsh plugin (Verified — README).
- **TAP consumers (the horizontal ecosystem):** because Bats, bash_unit (`-f tap`), and bashaspec emit TAP, they all plug into the same consumers — Jenkins' TAP plugin, GitLab, `prove`, custom formatters (testanything.org's consumers list). This is the one ecosystem *all* TAP frameworks share, and it is why "no native TAP" is shUnit2's heaviest row in §3.3.

### 4.3 The Ecosystem Table

| Framework | Official/primary helpers | Origin of helpers (Verified) | Distribution channels | Adjacent tooling |
|---|---|---|---|---|
| **Bats** | bats-support, bats-assert, bats-file (bats-core org); bats-mock (jasonkarns) | ztombol 2016 (assert/support), ztombol 2016 → org 2018 (file), jasonkarns 2015 (mock) | apt/brew/npm, Docker, GitHub Action (bats-action), Homebrew tap | bats-detik, bats-vscode, kcov/bashcov, pre-commit hooks |
| **shUnit2** | none official | — | distro packages, GitHub releases | `test_runner` (in-repo), unreleased JUnit XML |
| **bash_unit** | none needed (all-in-one) | — | install.sh, Homebrew, nixpkgs, AUR | pre-commit hook, GH Actions/GitLab CI snippets |
| **assert.sh** | stub.sh (jimeh) — external companion | 2012-era (Reported) | wget single file, bpkg | none |
| **bashaspec (d10n)** | none | — | single vendorable file | none |
| **ShellSpec** | in-core (mocking, coverage, parameterized) | — | install.sh, Homebrew, Docker | kcov, getoptions, readlinkf, portable-echo |

*Takeaway:* ecosystem is the single strongest predictor of long-term framework ROI. The bats stack has four maintained satellite libraries, an org, a GitHub Action, and a Homebrew tap; everything else ranges from "deliberately self-contained" (bash_unit, bashaspec) to "nothing official" (shUnit2, assert.sh). For a bank standardising across many repos, the bats stack's consistency — same load order, same failure-banner format, same CI wiring everywhere — is a real governance win.

---

## 5. Pros and Cons

### 5.1 Per Framework

**shUnit2 — pros.** POSIX-everywhere portability (sh/bash/dash/ksh/zsh, Solaris to Cygwin — Verified README); xUnit familiarity and a very low learning curve (umbrella §8); zero-dependency operation; stable for fifteen-plus years; Debian/Ubuntu packaging; assertions built in (no helper install step).

**shUnit2 — cons.** No TAP (the umbrella §8's "needs TAP wrapper" row); manual output capture; no parallel; no mocking helpers; ecosystem effectively frozen; releases rare (last tag 2020); the JUnit-XML story is documented-but-unreleased (Verified README). For new suites in a CI-driven shop, it is hard to justify over Bats; its niche is embedded/busybox/POSIX-only environments (umbrella §9 row) and legacy suites already written in it.

**Bats/bats-core — pros.** Native TAP + JUnit formatter + exit-code contract = the best CI story in shell testing (umbrella §8); `run` output capture; per-test subshell isolation; file-level hooks and per-test temp dirs; `--jobs` parallel; the bats-assert/support/file/mock ecosystem; the largest community (6.2k stars, active org, 2026 releases); minimal syntax ("if you can write a Bash `if`, you can write a Bats test" — bats_core_guide §1).

**Bats/bats-core — cons.** Bash-only (bash 4.2+; no dash/ksh portability — umbrella §8 row); bare Bats has no assertion macros (the ecosystem is effectively mandatory for readable suites); parallel is file-level only; mocking requires bats-mock or hand-rolled PATH stubs; the "original Bats vs bats-core" naming confusion persists in old blog posts and some distro packages.

**bash_unit — pros.** One file, no install ceremony (install.sh/brew/nixpkgs/AUR); a genuinely rich built-in assertion set; **`fake` mocking in the core** — unique at this weight class; TAP output on demand; stack traces with file:line on failure (Verified README); documented GH Actions/GitLab CI + pre-commit hook; still actively maintained (Feb 2026 push).

**bash_unit — cons.** Bash-centric (no real POSIX story); no parallel (only `-r` randomisation); no coverage integration; smaller community (635 stars) and docs that assume you read the README's long "how to run tests" essay; the "enterprise edition" tagline undersells a tool that is actually a well-scoped single-file framework.

**assert.sh — pros.** Two functions to learn; zero boilerplate; auto exit-status (CI-usable with `$?`); `skip_if` preconditions are genuinely handy; works under `set -e`/`set -u` (Verified changelog — 1.1 fixed both); stub.sh pairing gives a mocking story.

**assert.sh — cons.** Not a framework — no discovery, no lifecycle, no TAP, no parallel (bats_core_guide §8's verdict, with which this guide agrees); dormant since 2022; output-capture only via command substitution in your own variables; suites do not scale past a couple of dozen checks.

**bashaspec (d10n) — pros.** ~100 lines you can read in an afternoon and vendor into any repo; POSIX-wide (bash/zsh/ksh/dash/sh); TAP-compliant; four lifecycle hooks; MIT.

**bashaspec (d10n) — cons.** No assertion library, no mocking, no filtering, no parallel, no coverage; a 37-star niche project; **and the name carries the §2.5 identity problem** — teams adopting "bashaspec" for RSpec-style behaviour will be surprised, because the RSpec-style niche is actually shpec/ShellSpec territory.

**ShellSpec — pros.** The only framework combining BDD syntax, POSIX-wide portability, and enterprise features (coverage via kcov, mocking, parameterized tests, parallel — all first-class); superb failure messages from the expectation DSL; active single-author maintenance with a disciplined coding style (Verified README); 2019-era design that learned from RSpec/Jest/Mocha/Jasmine/Ginkgo/JUnit 5 (Verified README's inspired list).

**ShellSpec — cons.** Steepest learning curve of the field (new DSL — bats_core_guide §8); smaller community and ecosystem than Bats; the DSL can feel heavy for tiny scripts (the "just Bash" argument that made Bats popular); spec-file syntax is not Bash-the-language anymore.

**shpec — pros.** The lightest true `describe`/`it` experience; custom matchers are a nice extension point; POSIX core; works for bash scripts despite being shell-agnostic.

**shpec — cons.** No TAP, no parallel, no coverage; effectively unmaintained since 2022; matcher vocabulary is thin compared to ShellSpec's DSL; for new projects ShellSpec is the maintained successor of the same idea.

### 5.2 The Pros-Cons Table

| Framework | The pros in one line | The cons in one line |
|---|---|---|
| **shUnit2** | POSIX-everywhere xUnit with built-in assertions and a zero learning curve | Frozen ecosystem, no TAP, no parallel, manual capture — legacy-comfort only |
| **Bats/bats-core** | TAP-native, CI-first, isolated, parallel, with the only real helper ecosystem | Bash-only, assertions live in satellites, mocking needs assembly |
| **bash_unit** | One file with ten assertions, built-in `fake` mocking, TAP, CI snippets | Bash-centric, no parallel/coverage, small community |
| **assert.sh** | Two functions, zero ceremony, exit-status honesty | Not a framework — no discovery/lifecycle/TAP; dormant |
| **bashaspec (d10n)** | Tiny, vendorable, POSIX-wide, TAP-compliant | No assertions/mocking/filtering; niche; name-identity caveat (§2.5) |
| **ShellSpec** | BDD + POSIX + coverage + mocking + parallel, all first-class | New DSL to learn; smaller ecosystem; overkill for tiny scripts |
| **shpec** | Lightest describe/it with custom matchers | Dormant, no TAP/parallel/coverage — superseded by ShellSpec |

---

## 6. Decision Matrix

### 6.1 The Criteria and the Weights

Scoring the frameworks the way a solution architect in a regulated bank would — this is the umbrella §9's recommendations table turned into arithmetic, and it applies the business-case discipline of the management series' [business_case_development_guide.md](../management/business_case_development_guide.md) (weighted scoring over gut feel). Criteria and weights (sum = 100):

| # | Criterion | Weight | Why (banking/CI context) |
|---|---|---|---|
| 1 | CI integration | 20 | Shell tests exist to gate pipelines; TAP/JUnit/exit-code contract is the deliverable |
| 2 | Assertion quality | 15 | Audit-grade failure messages ("expected X, got Y") beat "line 7 failed" |
| 3 | Mocking | 15 | Payment/MFT scripts touch `sftp`, `curl`, `cksum` — external commands must be faked safely |
| 4 | Ecosystem & maintenance | 15 | A framework you standardise on will outlive the current team |
| 5 | POSIX portability | 10 | Legacy Solaris/AIX ksh boxes still run bank ops scripts |
| 6 | Fixtures & lifecycle | 10 | Payment files, temp dirs, cleanup — every test needs a sandbox |
| 7 | Learning curve | 10 | Ops engineers adopt what they can read in an afternoon |
| 8 | TAP output | 5 | Interoperability with Jenkins/GitLab TAP consumers (already weighted in #1; kept separate for the TAP-native vs wrapper distinction) |

### 6.2 The Weighted Scoring

Scores are 1–5 per criterion, justified against the §3 matrix (5 = first-class, 4 = strong via minimal assembly, 3 = workable, 2 = weak, 1 = absent). Weighted score = Σ(score × weight/100).

| Criterion (weight) | shUnit2 | Bats | bash_unit | assert.sh | bashaspec | ShellSpec |
|---|---|---|---|---|---|---|
| CI integration (20) | 3 | **5** | 4 | 3 | 3 | 4 |
| Assertion quality (15) | 4 | 4 (5 w/ bats-assert) | 4 | 2 | 2 | **5** |
| Mocking (15) | 2 | 4 (bats-mock/PATH) | **5** (built-in `fake`) | 2 (stub.sh) | 1 | **5** |
| Ecosystem & maintenance (15) | 2 | **5** | 3 | 2 | 1 | 3 |
| POSIX portability (10) | **5** | 2 | 3 | 4 | **5** | **5** |
| Fixtures & lifecycle (10) | 4 | 4 | 3 | 1 | 3 | **5** |
| Learning curve (10) | **5** | **5** | 4 | **5** | **5** | 3 |
| TAP output (5) | 1 | **5** | 4 | 2 | 4 | 3 |
| **Weighted total** | **3.25** | **4.45** | **3.80** | **2.60** | **2.70** | **4.20** |

The arithmetic in full (so the numbers are checkable):
- **Bats:** 20×5 + 15×4 + 15×4 + 15×5 + 10×2 + 10×4 + 10×5 + 5×5 = 100+60+60+75+20+40+50+25 = **430/100 = 4.45**
- **ShellSpec:** 20×4 + 15×5 + 15×5 + 15×3 + 10×5 + 10×5 + 10×3 + 5×3 = 80+75+75+45+50+50+30+15 = **420/100 = 4.20**
- **bash_unit:** 20×4 + 15×4 + 15×5 + 15×3 + 10×3 + 10×3 + 10×4 + 5×4 = 80+60+75+45+30+30+40+20 = **380/100 = 3.80**
- **shUnit2:** 20×3 + 15×4 + 15×2 + 15×2 + 10×5 + 10×4 + 10×5 + 5×1 = 60+60+30+30+50+40+50+5 = **325/100 = 3.25**
- **bashaspec:** 20×3 + 15×2 + 15×1 + 15×1 + 10×5 + 10×3 + 10×5 + 5×4 = 60+30+15+15+50+30+50+20 = **270/100 = 2.70**
- **assert.sh:** 20×3 + 15×2 + 15×2 + 15×2 + 10×4 + 10×1 + 10×5 + 5×2 = 60+30+30+30+40+10+50+10 = **260/100 = 2.60**

Ranking: **Bats 4.45 > ShellSpec 4.20 > bash_unit 3.80 > shUnit2 3.25 > bashaspec 2.70 > assert.sh 2.60.** The ranking is stable under small perturbations (top three hold unless a criterion dominates); the score spread between Bats and ShellSpec is the CI+ecosystem advantage, and the spread between the top three and the rest is the assertion/mocking/ecosystem gap.

### 6.3 The Decision Table

The verdict, mapped back onto the umbrella's §9 scenario table (each scenario row now names its weighted winner):

| Scenario (from umbrella §9) | Weighted winner | Runner-up | Why |
|---|---|---|---|
| New project, need framework | **Bats** (4.45) | ShellSpec (4.20) | CI + ecosystem dominate for greenfield Bash |
| Simple suite, JUnit familiarity | **shUnit2** (3.25) | bash_unit (3.80 overall; shUnit2 for the JUnit shape) | Legacy-comfort pick; honest score is mid-pack |
| Embedded / busybox / POSIX-only | **ShellSpec** (4.20) | shUnit2 (3.25) | Portability 5s, but ShellSpec brings coverage/mocking |
| Minimal-dependency single file | **bash_unit** (3.80) | assert.sh (2.60) | `fake` mocking + TAP beat assert.sh's two functions |
| Mocking-heavy suites (MFT/sftp/curl) | **bash_unit** / **ShellSpec** (5 on mocking) | Bats (4 w/ bats-mock) | Built-in `fake` is the least-assembly path |
| Large suites | **Bats** (parallel) | ShellSpec (parallel) | `--jobs` + JUnit at scale |
| Spec-style / requirements-driven | **ShellSpec** (4.20) | shpec | The maintained BDD option |
| Just assertions, nothing else | **assert.sh** (2.60) | — | Honest scope: it wins only on "least ceremony" |
| TAP-only CI consumers | **Bats** (native) | bash_unit / bashaspec | Zero-wrapper TAP |

### 6.4 What If the Weights Change?

The matrix is stable under reasonable weight shifts — the top three stay the top three unless a criterion dominates — but three scenarios flip it, and each flip is a real bank context:

- **POSIX mandate (portability weight 10 → 30, taken from CI).** If the estate includes Solaris/AIX ksh boxes or busybox appliances and "bash-only is a hard no", **ShellSpec (4.65)** overtakes **Bats (4.25)** — the only framework with BDD + portability + coverage. shUnit2 rises to 3.65 but still trails, because its lack of TAP and mocking hurts even in pure-POSIX shops.
- **Minimal-footprint mandate (ecosystem 15 → 5, learning curve 10 → 20).** Audit/ops teams that vendor everything and hate external dependencies push **bash_unit (4.10)** within a whisker of Bats (4.25) — and past it if CI weight also drops. The honest reading: bash_unit is the best "we will not install anything" answer.
- **Regulatory audit-trail mandate (assertion quality 15 → 25).** **ShellSpec (4.50)** edges past bare **Bats (4.45)** on failure-message quality; with bats-assert already in the stack Bats holds the lead at 4.60. The practical conclusion: *if you standardise on Bats, standardise on bats-assert with it* — the assertion-quality gap between bare Bats and the rest of the field is the one thing the ecosystem fixes for free.

Adoption-cost lens (cross-ref the management series' business-case discipline): Bats and bash_unit are free-and-instant (apt/brew/install.sh, no config file); ShellSpec costs a DSL learning curve but no infra; shUnit2 costs nothing and buys nothing new. The matrix's real message is the umbrella's §9 in numbers: **for new Bash work, Bats; for POSIX-mixed estates, ShellSpec; for vendorable single-file needs, bash_unit; for legacy suites, shUnit2; for "two assertions and out", assert.sh; for the spec-shaped project, ShellSpec again.**



---

## 7. Worked Example — One Suite, Six Harnesses

### 7.1 The Scenario: An MFT Payment-File Validation Script

The familiar context (cross-ref the bank series — [../banking/nets_software_systems_guide.md](../banking/nets_software_systems_guide.md) and the transfer-automation scripts in the banking cluster): an ops script that **pre-validates a payment instruction file before an MFT (Managed File Transfer) push to a clearing system**. Bad files must be caught before they leave the bank, so this script is exactly the kind of thing a regulated shop wraps in tests.

`validate_payment_file.sh` — a deliberately small script with real rules. Format: pipe-delimited records; the first line is a header, the rest are payment records, and the last line carries a checksum:

```
HDR|2026-08-25|PAY|2
PAY|SG-12345678|100.50|SGD|REF-0001
PAY|SG-87654321|2500.00|USD|REF-0002
CHK|817
```

Rules: header date must be `YYYY-MM-DD`; the header's record count must match the number of `PAY` lines; amounts must be positive with at most two decimals; currency codes must be in a known ISO-4217 list; the `CHK` value must equal (sum of all ASCII codes in the file body) mod 1000 — a stand-in for a real file-level checksum (MD5/SHA-256 would be used in production; a deterministic toy keeps the demo readable). Exit 0 + `OK` on success; error message on stderr + exit 1 on failure; exit 2 for a missing file.

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

The suite to write in every framework — six tests (the TDD habit from the dev-tooling cluster's test-driven-development_guide.md: each test names the rule it guards):

1. `accepts a valid payment file` → exit 0, output `OK`
2. `rejects a bad header date` → non-zero, stderr mentions "bad header date"
3. `rejects a negative amount` → non-zero
4. `rejects an unknown currency` → non-zero, stderr mentions "unknown currency"
5. `rejects a checksum mismatch` → non-zero, stderr mentions "checksum mismatch"
6. `rejects a missing file` → exit 2

Fixtures: `test/fixtures/valid.pay` (the four lines above), and per-test variants built in `setup`/`setUp` temp dirs.

### 7.2 The Suite in shUnit2

xUnit shape: `test*` functions, `setUp`/`tearDown`, built-in assertions, `. ./shunit2` last.

```bash
#!/bin/sh
# test_validate_payment_file.sh  (shUnit2)
. ./validate_payment_file.sh

FIXTURE_DIR=""
setUp() {
  FIXTURE_DIR=$(mktemp -d)
  cat > "$FIXTURE_DIR/valid.pay" <<'EOF'
HDR|2026-08-25|PAY|2
PAY|SG-12345678|100.50|SGD|REF-0001
PAY|SG-87654321|2500.00|USD|REF-0002
CHK|817
EOF
}
tearDown() { rm -rf "$FIXTURE_DIR"; }

testAcceptsValidFile() {
  output=$(validate_payment_file "$FIXTURE_DIR/valid.pay")
  assertEquals 0 $?
  assertEquals "OK" "$output"
}

testRejectsBadHeaderDate() {
  sed 's/2026-08-25/2026-13-40/' "$FIXTURE_DIR/valid.pay" > "$FIXTURE_DIR/bad_date.pay"
  validate_payment_file "$FIXTURE_DIR/bad_date.pay" 2>/dev/null
  assertNotEquals 0 $?
}

testRejectsNegativeAmount() {
  sed 's/100.50/-100.50/' "$FIXTURE_DIR/valid.pay" > "$FIXTURE_DIR/neg.pay"
  validate_payment_file "$FIXTURE_DIR/neg.pay" 2>/dev/null
  assertNotEquals 0 $?
}

testRejectsUnknownCurrency() {
  sed 's/|SGD|/|XYZ|/' "$FIXTURE_DIR/valid.pay" > "$FIXTURE_DIR/ccy.pay"
  err=$(validate_payment_file "$FIXTURE_DIR/ccy.pay" 2>&1 >/dev/null)
  assertNotEquals 0 $?
  assertContains "$err" "unknown currency"
}

testRejectsChecksumMismatch() {
  sed 's/CHK|817/CHK|000/' "$FIXTURE_DIR/valid.pay" > "$FIXTURE_DIR/chk.pay"
  validate_payment_file "$FIXTURE_DIR/chk.pay" 2>/dev/null
  assertNotEquals 0 $?
}

testRejectsMissingFile() {
  validate_payment_file "$FIXTURE_DIR/nope.pay" 2>/dev/null
  assertEquals 2 $?
}

. ./shunit2   # must be last — runs the suite
```

Observe: output capture is manual (`output=$(...)` then `assertEquals`), stderr is swallowed with `2>/dev/null` when uninteresting, and there is no TAP — the report is shUnit2's `Ran 6 tests… OK` summary. The xUnit shape is instantly readable to a Java team; the capture discipline is the cost.

### 7.3 The Suite in Bats

TAP shape with the ecosystem: `@test` blocks, `run` capture, `bats-assert` for readable assertions (cross-ref the bats_core_guide §5 and the umbrella §2 for the load-order boilerplate).

```bash
#!/usr/bin/env bats
# validate_payment_file.bats  (bats-core)
load 'bats-support/load'
load 'bats-assert/load'

setup() {
  TEST_DIR=$(mktemp -d)
  cat > "$TEST_DIR/valid.pay" <<'EOF'
HDR|2026-08-25|PAY|2
PAY|SG-12345678|100.50|SGD|REF-0001
PAY|SG-87654321|2500.00|USD|REF-0002
CHK|817
EOF
}
teardown() { rm -rf "$TEST_DIR"; }

@test "accepts a valid payment file" {
  run ./validate_payment_file.sh "$TEST_DIR/valid.pay"
  assert_success
  assert_output "OK"
}

@test "rejects a bad header date" {
  sed 's/2026-08-25/2026-13-40/' "$TEST_DIR/valid.pay" > "$TEST_DIR/bad_date.pay"
  run ./validate_payment_file.sh "$TEST_DIR/bad_date.pay"
  assert_failure
  assert_output --partial "bad header date"
}

@test "rejects a negative amount" {
  sed 's/100.50/-100.50/' "$TEST_DIR/valid.pay" > "$TEST_DIR/neg.pay"
  run ./validate_payment_file.sh "$TEST_DIR/neg.pay"
  assert_failure
}

@test "rejects an unknown currency" {
  sed 's/|SGD|/|XYZ|/' "$TEST_DIR/valid.pay" > "$TEST_DIR/ccy.pay"
  run ./validate_payment_file.sh "$TEST_DIR/ccy.pay"
  assert_failure
  assert_output --partial "unknown currency"
}

@test "rejects a checksum mismatch" {
  sed 's/CHK|817/CHK|000/' "$TEST_DIR/valid.pay" > "$TEST_DIR/chk.pay"
  run ./validate_payment_file.sh "$TEST_DIR/chk.pay"
  assert_failure
  assert_output --partial "checksum mismatch"
}

@test "rejects a missing file" {
  run ./validate_payment_file.sh "$TEST_DIR/nope.pay"
  assert_failure 2
}
```

**Mocking in Bats** (the MFT angle — the script's checksum could call an external tool in production; here we stub it). PATH-stub per the umbrella §4.1 Method 2, or bats-mock per Method 3:

```bash
@test "uses cksum for the file digest" {
  MOCK_DIR=$(mktemp -d)
  cat > "$MOCK_DIR/cksum" <<'EOF'
#!/bin/bash
echo "12345 0 fake"
EOF
  chmod +x "$MOCK_DIR/cksum"
  PATH="$MOCK_DIR:$PATH" run ./validate_payment_file.sh "$TEST_DIR/valid.pay"
  rm -rf "$MOCK_DIR"
  assert_success
}
```

Bats' advantages on display: `run` + `$status`/`$output` capture, `--partial` output matching, failure exit-code precision (`assert_failure 2`), and per-test subshell isolation. Run: `bats test/`; parallel: `bats --jobs 4 test/`; CI: `bats --formatter tap` or `--formatter junit`.

### 7.4 The Suite in bash_unit

Single-file shape: `test*` functions, built-in assertions, and the `fake` primitive — the mock is **in the core**, no helper install.

```bash
#!/bin/bash
# test_validate_payment_file.sh  (bash_unit)
. ./validate_payment_file.sh

setup() {
  FIXTURE_DIR=$(mktemp -d)
  printf 'HDR|2026-08-25|PAY|2\nPAY|SG-12345678|100.50|SGD|REF-0001\nPAY|SG-87654321|2500.00|USD|REF-0002\nCHK|817\n' \
    > "$FIXTURE_DIR/valid.pay"
}

test_accepts_valid_file() {
  assert_equals "OK" "$(validate_payment_file "$FIXTURE_DIR/valid.pay")"
}

test_rejects_bad_header_date() {
  sed 's/2026-08-25/2026-13-40/' "$FIXTURE_DIR/valid.pay" > "$FIXTURE_DIR/bad.pay"
  assert_status_code 1 validate_payment_file "$FIXTURE_DIR/bad.pay"
}

test_rejects_negative_amount() {
  sed 's/100.50/-100.50/' "$FIXTURE_DIR/valid.pay" > "$FIXTURE_DIR/neg.pay"
  assert_status_code 1 validate_payment_file "$FIXTURE_DIR/neg.pay"
}

test_rejects_unknown_currency() {
  sed 's/|SGD|/|XYZ|/' "$FIXTURE_DIR/valid.pay" > "$FIXTURE_DIR/ccy.pay"
  assert_status_code 1 validate_payment_file "$FIXTURE_DIR/ccy.pay"
}

test_rejects_checksum_mismatch() {
  sed 's/CHK|817/CHK|000/' "$FIXTURE_DIR/valid.pay" > "$FIXTURE_DIR/chk.pay"
  assert_status_code 1 validate_payment_file "$FIXTURE_DIR/chk.pay"
}

test_rejects_missing_file() {
  assert_status_code 2 validate_payment_file "$FIXTURE_DIR/nope.pay"
}
```

**Mocking in bash_unit** — the differentiating demo. Fake `cksum` for one test (the README's documented pattern, simplified):

```bash
test_uses_cksum_for_digest() {
  fake cksum 'echo "12345 0 fake"'
  assert_equals "OK" "$(validate_payment_file "$FIXTURE_DIR/valid.pay")"
}
```

Run: `./bash_unit test_validate_payment_file.sh`; filter: `-p 'currency'`; TAP for CI: `-f tap`. Failure output includes the stack trace with `file:line` — e.g. `doc:10:test_rejects_unknown_currency()` — which is the single-file framework's answer to IDE navigation.

### 7.5 The Suite in assert.sh

The honest two-function shape. There is no discovery — tests run top-to-bottom and `assert_end` closes the suite — and no fixture hook, so the fixture file is prepared inline. This is the "5–20 quick checks" tier (bats_core_guide §8's "not a framework" verdict, demonstrated).

```bash
#!/bin/bash
# tests_validate.sh  (assert.sh)
. ./assert.sh
. ./validate_payment_file.sh

FIXTURE_DIR=$(mktemp -d)
trap 'rm -rf "$FIXTURE_DIR"' EXIT
printf 'HDR|2026-08-25|PAY|2\nPAY|SG-12345678|100.50|SGD|REF-0001\nPAY|SG-87654321|2500.00|USD|REF-0002\nCHK|817\n' \
  > "$FIXTURE_DIR/valid.pay"

assert "validate_payment_file '$FIXTURE_DIR/valid.pay'" "OK"                       # stdout equals
assert_raises "validate_payment_file '$FIXTURE_DIR/valid.pay'" 0                    # exit code 0
sed 's/2026-08-25/2026-13-40/' "$FIXTURE_DIR/valid.pay" > "$FIXTURE_DIR/bad.pay"
assert_raises "validate_payment_file '$FIXTURE_DIR/bad.pay'" 1
sed 's/100.50/-100.50/' "$FIXTURE_DIR/valid.pay" > "$FIXTURE_DIR/neg.pay"
assert_raises "validate_payment_file '$FIXTURE_DIR/neg.pay'" 1
sed 's/|SGD|/|XYZ|/' "$FIXTURE_DIR/valid.pay" > "$FIXTURE_DIR/ccy.pay"
assert_raises "validate_payment_file '$FIXTURE_DIR/ccy.pay'" 1
sed 's/CHK|817/CHK|000/' "$FIXTURE_DIR/valid.pay" > "$FIXTURE_DIR/chk.pay"
assert_raises "validate_payment_file '$FIXTURE_DIR/chk.pay'" 1
assert_raises "validate_payment_file '$FIXTURE_DIR/nope.pay'" 2
assert_end "validate_payment_file"
```

Note the stderr assertions are simply not expressible — `assert` only compares stdout, `assert_raises` only the exit code. The suite is genuinely compact (ten checks in eleven lines of tests) and the exit status propagates to CI via `$?`. But the moment you need per-test sandboxes, partial-output matching, or stderr verification, you have left assert.sh's design envelope — which is exactly the envelope boundary the matrix in §3.3 draws.

### 7.6 The Suite in bashaspec

As documented (d10n/bashaspec — see the §2.5 caveat): TAP-compliant, `test_*` functions, raw `[[ ]]` assertions, four lifecycle hooks. The suite is a *spec file* — `bashaspec.sh` runs every `*_spec.sh` in the directory, or you run the file directly after sourcing the library. The fixture file is prepared in `before_all` and cleaned in `after_all`.

```bash
#!/bin/bash
# validate_payment_file_spec.sh  (d10n/bashaspec)
. ./validate_payment_file.sh

FIXTURE_DIR=""
before_all() {
  FIXTURE_DIR=$(mktemp -d)
  printf 'HDR|2026-08-25|PAY|2\nPAY|SG-12345678|100.50|SGD|REF-0001\nPAY|SG-87654321|2500.00|USD|REF-0002\nCHK|817\n' \
    > "$FIXTURE_DIR/valid.pay"
}
after_all() { rm -rf "$FIXTURE_DIR"; }

test_accepts_valid_file() {
  out=$(validate_payment_file "$FIXTURE_DIR/valid.pay")
  [[ "$out" = "OK" ]] || return 1
}
test_rejects_bad_header_date() {
  sed 's/2026-08-25/2026-13-40/' "$FIXTURE_DIR/valid.pay" > "$FIXTURE_DIR/bad.pay"
  validate_payment_file "$FIXTURE_DIR/bad.pay" 2>/dev/null || return 0
  return 1
}
test_rejects_negative_amount() {
  sed 's/100.50/-100.50/' "$FIXTURE_DIR/valid.pay" > "$FIXTURE_DIR/neg.pay"
  validate_payment_file "$FIXTURE_DIR/neg.pay" 2>/dev/null || return 0
  return 1
}
test_rejects_unknown_currency() {
  sed 's/|SGD|/|XYZ|/' "$FIXTURE_DIR/valid.pay" > "$FIXTURE_DIR/ccy.pay"
  validate_payment_file "$FIXTURE_DIR/ccy.pay" 2>/dev/null || return 0
  return 1
}
test_rejects_checksum_mismatch() {
  sed 's/CHK|817/CHK|000/' "$FIXTURE_DIR/valid.pay" > "$FIXTURE_DIR/chk.pay"
  validate_payment_file "$FIXTURE_DIR/chk.pay" 2>/dev/null || return 0
  return 1
}
test_rejects_missing_file() {
  validate_payment_file "$FIXTURE_DIR/nope.pay" 2>/dev/null
  [[ $? -eq 2 ]] || return 1
}
. ./bashaspec.sh
```

The pattern `cmd || return 0; return 1` is the raw-`[[ ]]` idiom the README itself recommends ("use return codes to identify failed assertions"). It works, it is POSIX-wide, and it is unapologetically unassisted — which is the honest profile of this framework, and a reminder that **anyone who wanted the RSpec-style `describe`/`it` experience under the name "bashaspec" will find it in shpec (§2.7) or ShellSpec (next).**

### 7.7 The Suite in ShellSpec

BDD shape — the same six tests as an executable specification (the bats_core_guide §8's "RSpec-inspired DSL" verdict, in practice):

```bash
# spec/validate_payment_file_spec.sh  (ShellSpec)
Describe 'validate_payment_file'
  setup() {
    FIXTURE_DIR=$(mktemp -d)
    printf 'HDR|2026-08-25|PAY|2\nPAY|SG-12345678|100.50|SGD|REF-0001\nPAY|SG-87654321|2500.00|USD|REF-0002\nCHK|817\n' \
      > "$FIXTURE_DIR/valid.pay"
  }
  cleanup() { rm -rf "$FIXTURE_DIR"; }

  It 'accepts a valid payment file'
    When call validate_payment_file "$FIXTURE_DIR/valid.pay"
    The status should be success
    The output should equal "OK"
  End

  It 'rejects a bad header date'
    sed 's/2026-08-25/2026-13-40/' "$FIXTURE_DIR/valid.pay" > "$FIXTURE_DIR/bad.pay"
    When call validate_payment_file "$FIXTURE_DIR/bad.pay"
    The status should be failure
    The stderr should include "bad header date"
  End

  It 'rejects a negative amount'
    sed 's/100.50/-100.50/' "$FIXTURE_DIR/valid.pay" > "$FIXTURE_DIR/neg.pay"
    When call validate_payment_file "$FIXTURE_DIR/neg.pay"
    The status should be failure
  End

  It 'rejects an unknown currency'
    sed 's/|SGD|/|XYZ|/' "$FIXTURE_DIR/valid.pay" > "$FIXTURE_DIR/ccy.pay"
    When call validate_payment_file "$FIXTURE_DIR/ccy.pay"
    The status should be failure
    The stderr should include "unknown currency"
  End

  It 'rejects a checksum mismatch'
    sed 's/CHK|817/CHK|000/' "$FIXTURE_DIR/valid.pay" > "$FIXTURE_DIR/chk.pay"
    When call validate_payment_file "$FIXTURE_DIR/chk.pay"
    The status should be failure
    The stderr should include "checksum mismatch"
  End

  It 'rejects a missing file with exit 2'
    When call validate_payment_file "$FIXTURE_DIR/nope.pay"
    The status should equal 2
  End

  It 'mocks cksum for the file digest'
    mock cksum 'echo "12345 0 fake"'
    When call validate_payment_file "$FIXTURE_DIR/valid.pay"
    The status should be success
  End
End
```

Run: `shellspec` (discovers `spec/`); parallel: `shellspec --jobs 4`; coverage: `shellspec --kcov`; CI: `shellspec --format tap` or `--format junit`. Note what the DSL buys: `The stderr should include …` is a first-class assertion here (a row where ShellSpec beats shUnit2, assert.sh, and bashaspec outright), and `mock` is in-core. The cost is visible too: the file is the longest of the six and no longer looks like Bash.

### 7.8 The Outputs Compared

The same failing test (`rejects an unknown currency`) as reported by each harness — the §3.1 "lineage predicts failure-message quality" claim, demonstrated:

```text
# shUnit2
testRejectsUnknownCurrency
ASSERT:expected:<1> but was:<0>  ... FAILED
Ran 6 tests. FAILED (failures=1)

# Bats (bare)
not ok 4 rejects an unknown currency
# (in test file validate_payment_file.bats, line 38)
#   `run ./validate_payment_file.sh "$TEST_DIR/ccy.pay"' failed

# Bats + bats-assert
not ok 4 rejects an unknown currency
# -- output does not contain substring --
#   substring : unknown currency
#   output    : OK
# --

# bash_unit
Running test_rejects_unknown_currency ... FAILURE
  expected status code [1] but was [0]
  test_validate_payment_file.sh:38:test_rejects_unknown_currency()

# assert.sh
test #7 "validate_payment_file ...ccy.pay" failed:
        program terminated with code 0 instead of 1
6 of 7 examples tests failed in 0.021s.

# bashaspec (d10n)
not ok 5 - test_rejects_unknown_currency
# (failure output shown for the failing test)

# ShellSpec
1 example, 0 failures
  ✘ rejects an unknown currency
    expected stderr to include "unknown currency"
    but stderr was empty
```

Read the column: bare Bats tells you *which line failed*; bats-assert tells you *what was expected and what was actually seen*; bash_unit adds the *source file and line of the test*; ShellSpec restates the *spec sentence* alongside the mismatch. For a payment-file pipeline where a colleague in ops must diagnose a red build at 3 a.m., the difference between "output does not contain substring" and "line 38 failed" is the difference between a fix in minutes and a fix in hours.

### 7.9 The Lessons

1. **The framework shapes the suite's spine, not its coverage.** Every harness above expresses the same six rules — the tests that exist are a product of the script's risk profile, not the framework. The TDD discipline (test-driven-development_guide.md) transfers unchanged: write the failing test for the bug first, watch it fail in the harness's own voice, then make it pass. What changes is only the *grammar* of that loop.
2. **Assertion grammar is the real differentiator.** `assert_status_code 1` (bash_unit), `assert_raises … 1` (assert.sh), `assert_failure 2` (Bats), `The status should be failure` (ShellSpec) — same intent, four ergonomics. The stderr-matching row is where the field splits: ShellSpec and bats-assert can assert on stderr content; shUnit2, assert.sh, and bashaspec cannot without shell gymnastics. In a bank, error-message assertions are where regression tests earn their keep.
3. **Isolation is a feature, not an implementation detail.** Bats runs each test in a subshell (bats_core_guide §4.2); the function-based frameworks share one process, so a stray `cd` or unset variable in one test can poison the next. shUnit2 mitigates with `setUp`/`tearDown` discipline; assert.sh and bashaspec rely entirely on author discipline. This is the hidden cost behind "minimal framework".
4. **Mocking maturity predicts testability ceiling.** The MFT script's external commands (`cksum`, and in real life `sftp`/`curl`) are trivially faked in bash_unit (`fake`), ShellSpec (`mock`), and Bats (PATH stubs / bats-mock); they are DIY in shUnit2 and absent-by-design in assert.sh and bashaspec. The umbrella's §4.1 mocking patterns work with any framework — but the frameworks with primitives make the *right* thing the *easy* thing.
5. **Output capture is the quiet killer.** Bats' `run`/`$status`/`$output` and ShellSpec's `When call` make "run command, inspect result" a one-liner. shUnit2's manual `output=$(...)` + `assertEquals` works but silently collapses stdout and stderr unless you're careful; assert.sh's `assert` can't see stderr at all. For CLI validation scripts, this row alone justifies a framework.
6. **The ecosystem converts Bats from "good" to "standard".** The same six tests with bats-assert read better than the bare versions, and the load-order boilerplate (`load 'bats-support/load'; load 'bats-assert/load'`) is the same in every repo — which is precisely what a governance standard wants (umbrella §6's project template already assumes it).
7. **Verification discipline pays.** Two of the task brief's assumptions did not survive contact with primary sources (assert.sh's author; the RSpec-style "bashaspec"), and one date is now exact (Bats' Dec 28, 2011 first commit). The lesson for this series: repo pages, the GitHub API, and testanything.org beat blog posts and memory.

---

## 8. Summary — One Page

**The question:** six ways to test the same shell script — which harness do you standardise on?

**The one-table answer:**

| If you… | Use | Because |
|---|---|---|
| Write Bash for CI platforms and want the biggest safety net | **Bats + bats-assert + bats-file (+ bats-mock)** | TAP-native, parallel, isolated, the only real ecosystem (4.45) |
| Run scripts on POSIX-mixed estates (Solaris/AIX ksh, busybox) or want spec-style suites | **ShellSpec** | BDD + POSIX + coverage + mocking + parallel, all in core (4.20) |
| Want one vendorable file with real assertions and mocking, nothing to install | **bash_unit** | `fake` in-core, ten assertions, TAP on demand (3.80) |
| Maintain a legacy suite or must support embedded sh | **shUnit2** | POSIX-everywhere xUnit, zero deps (3.25) |
| Need two assertions for a quick ops check | **assert.sh** | Least ceremony that still reports honestly (2.60) |
| Want a tiny TAP library to vendor | **bashaspec (d10n)** | ~100 lines, four hooks, POSIX-wide (2.70; see §2.5) |
| Want `describe`/`it` and nothing heavier | **shpec** | The lightest BDD, but dormant — prefer ShellSpec (n/a) |

**The final word — the right harness.** The "right" shell testing harness is the one whose *grammar* matches your team and whose *plumbing* matches your estate. For a bank's new Bash work, that is **Bats with the bats-assert stack** — it wins the weighted matrix (4.45 vs ShellSpec's 4.20) on CI integration and ecosystem, and its "just Bash" syntax is the lowest-friction standard a distributed ops team can adopt; the umbrella guide's §9 workflow (write functions → format → lint → test → CI) is unchanged, and its §8 two-way table is this guide's §3 in miniature. Reach for **ShellSpec** when POSIX portability or spec-style readability outweighs ecosystem size — it is the only framework that combines both with first-class coverage. Keep **shUnit2** for legacy suites and embedded shells, **bash_unit** where "one file, no install" is a hard constraint, **assert.sh** for the smallest honest suites, and **bashaspec** — as documented, not as rumoured — for a vendorable TAP micro-harness. The frameworks differ in grammar and plumbing; the discipline that makes them valuable — write the failing test first, keep the fixtures small, mock every external command, assert on the error messages, wire the exit code into CI — is the same in all six.

---

## 9. Glossary

- **shUnit2** — The xUnit-style unit-test framework for Bourne-based shells, maintained by Kate Ward (~2008-era lineage, now on GitHub as kward/shunit2). Tests are `test*` functions; assertions are `assertEquals`-family macros; `. ./shunit2` runs the suite.
- **Bats** — "Bash Automated Testing System", created by Sam Stephenson on December 28, 2011 (Verified — GitHub API). The original `sstephenson/bats` is archived (April 29, 2021); everything modern lives in bats-core.
- **bats-core** — The community-maintained continuation of Bats, forked September 19, 2017 (Verified — README). The de facto standard shell testing framework: `@test` blocks, TAP output, `run` capture, parallel `--jobs`, formatters.
- **TAP** — Test Anything Protocol; a plain-text stream (`1..N` plan, `ok`/`not ok` lines, `#` diagnostics) that decouples test reporting from presentation. Born in Perl's test harness; earliest recorded commit January 30, 1988 by Larry Wall (Verified — testanything.org history).
- **Test Anything Protocol** — the full name of TAP; "started life as part of the test harness for Perl but now has implementations in C, C++, Python, PHP, Perl, Java, JavaScript, Go, Rust, and others" (Verified — testanything.org).
- **bash_unit** — A lightweight, single-file bash testing framework authored by Pascal Grange (repo created July 13, 2011, Verified). `test*` functions, ten built-in assertions, built-in `fake` mocking, `-f tap` output, documented GitHub Actions/GitLab CI integration.
- **assert.sh** — A minimal bash assertion library by Robert Lehmann (repo created November 15, 2009, Verified; LGPLv3). API: `assert`, `assert_raises`, `assert_end`, `skip`, `skip_if`. *Not* "Robert Böhnke" (flagged — see §2.4).
- **bashaspec** — Ambiguous name, flagged: the only current GitHub project is d10n/bashaspec (2020), a TAP-compliant, ~100-line, POSIX-wide `test_*`-function library with `before_all`/`after_all`/`before_each`/`after_each` hooks. An RSpec-style `describe`/`it` framework under this name could not be verified (§2.5).
- **ShellSpec** — A full-featured BDD unit-testing framework for all POSIX shells (2019, Koichi Nakashima, MIT). `Describe`/`It` DSL, first-class mocking, kcov coverage, parameterized tests, parallel execution.
- **shpec** — A BDD-style shell testing framework (2013, rylnd): `describe`/`it` blocks, assert matchers, `stub_command`/`unstub_command`, custom matchers. Dormant since ~2022; superseded by ShellSpec for new work.
- **xUnit** — The family of test frameworks modelled on JUnit/SUnit: test cases discovered by naming convention, assertion macros, setUp/tearDown lifecycle, aggregate reporting. shUnit2 is the shell member of the family.
- **BDD** — Behaviour-Driven Development; a testing style (popularised by RSpec) where suites read as specifications: `describe` a behaviour, `it` behaves thus. ShellSpec and shpec are the shell members.
- **describe / it** — The BDD block keywords: `describe` groups related behaviours, `it` names one specific behaviour under test.
- **assertion** — A check that a condition holds; in these frameworks, a macro/function/matcher that records a failure (and usually prints expected-vs-actual) without aborting the run.
- **mock** — A stand-in for a real command/function that records or scripted its behaviour, used to isolate the code under test from external dependencies (sftp, curl, cksum). bash_unit's `fake`, ShellSpec's `mock`, bats-mock's `stub`.
- **fixture** — Pre-arranged input data or environment (sample payment files, temp directories) that a test runs against; usually created in setup hooks and destroyed in teardown.
- **CI** — Continuous Integration: automated pipelines that build and test every change. Shell testing frameworks integrate via exit codes, TAP streams, and JUnit XML.
- **coverage** — The proportion of the script's lines/branches executed by the test suite, measured by tools like kcov; integrated in ShellSpec, external-but-standard for Bats.
- **parallel** — Running independent test files concurrently to cut wall-clock time: `bats --jobs N`, ShellSpec's `--jobs`; absent in the function-based frameworks.
- **harness** — The test runner machinery (discovery, lifecycle, reporting) that a framework provides around your assertions; "the right harness" is this guide's framing for choosing that machinery deliberately.
- **MFT** — Managed File Transfer; the secure, audited file-transfer channel banks use to exchange payment and reference files with clearing systems and counterparties. The worked example's validation script guards an MFT push.

---

## 10. Claims Status, References and Disclaimer

### Claims status

- **Verified (primary sources, August 2026):** Bats first commit Dec 28, 2011 (GitHub API, sstephenson/bats); bats-core fork Sept 19, 2017 at commit 0360811 and original's archival April 29, 2021 (bats-core README); bash_unit repo created July 13, 2011 (GitHub API) and all README claims (assertions, `fake`, options, CI snippets, French docs links); assert.sh created Nov 15, 2009, author Robert Lehmann, LGPLv3 (GitHub API + README); ShellSpec created Feb 5, 2019, author Koichi Nakashima, MIT, feature list (GitHub API + README); shpec created Jan 27, 2013, matchers and stubbing (GitHub API + README); d10n/bashaspec created July 23, 2020, TAP-compliance and hook set (GitHub API + README); bats-assert/bats-support/bats-file provenance and the bats-core org inventory (GitHub API); TAP's Perl origin, 1988 earliest commit, and version history (testanything.org history page); shUnit2's xUnit identity, log4sh origin, POSIX shell list, last release v2.1.8 (kward/shunit2 README + releases); star counts and push dates (GitHub API, point-in-time).
- **Reported (flagged):** shUnit2's "~2008" start (project predates GitHub; corroborated by bats_core_guide §8's "2008-era"); bash_unit's French origin (author's language, French docs links — plausible, not formally documented); stub.sh's creation era.
- **Not verified (flagged):** any RSpec-style "bashaspec" framework (§2.5 — the name as described could not be found in GitHub, npm, or the Wayback Machine); the "Robert Böhnke" attribution for assert.sh (§2.4 — the primary source says Robert Lehmann).

### References

- kward/shunit2 — https://github.com/kward/shunit2
- sstephenson/bats (archived original) — https://github.com/sstephenson/bats
- bats-core/bats-core — https://github.com/bats-core/bats-core ; docs — https://bats-core.readthedocs.io/
- bats-core/bats-assert, bats-core/bats-support, bats-core/bats-file; jasonkarns/bats-mock; ztombol originals — https://github.com/bats-core , https://github.com/jasonkarns/bats-mock
- bash-unit/bash_unit — https://github.com/bash-unit/bash_unit (originally pgrange/bash_unit)
- lehmannro/assert.sh — https://github.com/lehmannro/assert.sh
- d10n/bashaspec — https://github.com/d10n/bashaspec
- shellspec/shellspec — https://github.com/shellspec/shellspec ; https://shellspec.info/
- rylnd/shpec — https://github.com/rylnd/shpec
- Test Anything Protocol — https://testanything.org/ (home, history, TAP version 14 specification)
- Sibling guides: technology/bash_script_testing_automation.md (§2 frameworks, §8 bats-vs-shunit2, §9 recommendations); technology/bats_core_guide.md (dedicated Bats guide, §8 alternatives); technology/test-driven-development_guide.md (TDD discipline); ../banking/nets_software_systems_guide.md and the banking cluster (MFT/payment context); ../management/business_case_development_guide.md (adoption-cost/weighted-decision discipline)

### Disclaimer

This guide is technology research, not an audit or a procurement recommendation. Framework facts were verified against primary sources in August 2026; star counts, versions, and maintenance activity drift, and the decision matrix encodes this author's weights — change the weights, change the winner (see §6.4). The worked-example script is a teaching vehicle with a deliberately toy checksum; production payment validation should use real message standards, real digests, and the bank's own control framework.
