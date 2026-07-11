# Bash Script Testing & Automation

A comprehensive guide to writing, testing, linting, and automating bash scripts with modern tools, frameworks, and CI/CD practices.

---

## 1. Overview

### Why Test Bash Scripts?

Bash scripts run CI/CD pipelines, deployments, backups, and system automation. Despite their ubiquity, they are notoriously brittle:

- **Unset variables** — `rm -rf /$DIR` with `$DIR` empty causes data loss
- **Missing commands** — assuming `python3` where only `python` exists causes silent failure
- **Unquoted variables** — `[ $STATUS = "ok" ]` with spaces or empty values causes syntax errors
- **Word splitting** — `for f in $FILES` breaks on filenames with spaces
- **Piped errors** — `cmd1 | cmd2` — if `cmd1` fails and `cmd2` succeeds, errors are lost
- **Environment drift** — different bash versions, OS differences, tool availability between dev and CI

### Testing Categories

**Unit Testing** — Tests individual functions in isolation. Mocks external commands and filesystem. Fast and targeted.

**Integration Testing** — Tests scripts with real external dependencies (filesystem, commands, processes). Catches interface mismatches.

**End-to-End (E2E)** — Full pipeline from argument parsing through processing to output, often in Docker. Most comprehensive but slowest.

### The Challenge

Bash has **no built-in testing framework** — the ecosystem is entirely community-driven. No consensus on "the right way," documentation is scattered across GitHub, blogs, and Stack Overflow, and install methods range from `apt` to `npm` to `go install`. This guide covers the proven tools and patterns used in production.

---

## 2. Bash Testing Frameworks

### bats (Bash Automated Testing System)

The most popular bash testing framework. Uses TAP (Test Anything Protocol) output, making it CI-friendly. Maintained as **bats-core**.

```bash
sudo apt update && sudo apt install -y bats         # Debian/Ubuntu
brew install bats-core                                # macOS
npm install -g bats                                   # Always latest
```

```bash
#!/usr/bin/env bats

setup() {
  export TEST_DIR=$(mktemp -d)
}

teardown() {
  rm -rf "$TEST_DIR"
}

@test "greeting returns hello" {
  result="$(./script.sh greet "world")"
  [ "$result" == "Hello, world!" ]
}

@test "greeting fails without arguments" {
  run ./script.sh greet
  [ "$status" -eq 1 ]
  [[ "$output" == *"Usage:"* ]]
}
```

**Key Features:**
- **`run`** — Captures stdout/stderr + exit code into `$output`, `$status`, `$lines`
- **`skip`** — `skip "reason"` to conditionally skip a test
- **`setup_file` / `teardown_file`** — Run once per file (bats 1.7+)
- **Parallel execution** — `bats --jobs N` runs `.bats` files concurrently

**Community helpers:** `bats-support`, `bats-assert`, `bats-file`, `bats-mock`:

```bash
@test "validate config" {
  run ./parse-config.sh test/fixtures/good.conf
  assert_success
  assert_output --partial "Config loaded"
  assert_line --index 0 "server=localhost"
}
```

Key assertions: `assert_success`, `assert_failure [N]`, `assert_output [--partial|--regex]`, `assert_line [--index N]`, `refute_output`, `refute_line`, `assert_file_exist`, `assert_dir_exist`.

**TAP output:** `bats --formatter tap tests/` — works with any CI that understands TAP. Combined: `bats --jobs "$(nproc)" --formatter tap tests/`.

---

### shunit2

One of the oldest bash testing frameworks, porting JUnit to shell. **POSIX-compatible** (works with dash, busybox).

```bash
sudo apt install shunit2

testEquality() { assertEquals 1 1; }
testContains() { assertContains "hello world" "world"; }
. shunit2    # Must come last
```

Assertions: `assertEquals`, `assertNotEquals`, `assertNull`, `assertNotNull`, `assertContains`, `assertNotContains`, `assertTrue`, `assertFalse`.

**Downsides vs bats:** No TAP output, no parallel execution, no built-in `run`-style output capture, smaller community, less active development.

---

### zsh-test-runner / zunit

Designed for **zsh only** — leverage zsh's modern features. Not portable to bash/sh. Only for zsh-specific projects (oh-my-zsh plugins, etc.).

---

### zicher

Minimal single-file bash testing library. Best for embedded systems or when installing a full framework is overkill.

```bash
. zicher.sh
test_hello() { assert_eq "$(./greet.sh world)" "Hello, world!"; }
run_tests
```

---

## 3. Static Analysis & Linting

### ShellCheck

The single most important bash quality tool — static analysis for hundreds of common pitfalls.

```bash
sudo apt install shellcheck                           # Debian/Ubuntu
brew install shellcheck                                # macOS

shellcheck script.sh
shellcheck --severity=warning src/**/*.sh
shellcheck --format=json script.sh                     # For programmatic use
```

**Detection levels:**
- **Error:** Syntax errors, unmatched quotes
- **Warning:** Unquoted variables (`SC2086`), `useless cat` (`SC2002`), `cd` without error handling (`SC2164`), unsafe assignment masking return values (`SC2155`)
- **Info:** POSIX compatibility (`SC2039`), backtick usage (`SC2006`)
- **Style:** Readability improvements

**Configuration (`.shellcheckrc`):**

```ini
disable=SC2086,SC2034
shell=bash
source-path=src/
source-path=SHELLCHECK_SCRIPT_DIR/../lib
```

**Inline suppression:** `# shellcheck disable=SC2086`

**CI:** Pre-installed on GitHub Actions `ubuntu-latest` runners. VS Code extension available.

---

### bash -n (Syntax Check)

```bash
bash -n script.sh     # Syntax check without executing
```

Fast, zero dependencies, built-in. Only catches syntax errors — not runtime bugs, unset variables, or logic errors.

---

### shfmt (Shell Formatter)

Auto-formats shell scripts (like `gofmt` / `prettier`).

```bash
go install mvdan.cc/sh/v3/cmd/shfmt@latest
brew install shfmt

shfmt -w script.sh      # Format in-place
shfmt -d script.sh      # Diff mode (for CI — exit non-zero if unformatted)
shfmt -i 4 -ln posix    # Set indent width and language variant
```

---

## 4. Testing Techniques & Patterns

### 4.1 Mocking External Commands

**Method 1 — Function Override (simplest):**

```bash
git() { echo "MOCK: git $*"; return 0; }
```

**Method 2 — PATH Manipulation (most robust):** Intercepts all invocations, including subprocesses.

```bash
setup() {
  export ORIGINAL_PATH="$PATH"
  export MOCK_DIR=$(mktemp -d)
  cat > "$MOCK_DIR/git" <<'EOF'
#!/bin/bash
echo "MOCK: git $@" >&2
exit 0
EOF
  chmod +x "$MOCK_DIR/git"
  export PATH="$MOCK_DIR:$PATH"
}

teardown() {
  export PATH="$ORIGINAL_PATH"
  rm -rf "$MOCK_DIR"
}
```

**Method 3 — bats-mock (stub/unstub):**

```bash
setup() { load 'bats-support/load'; load 'bats-assert/load'; load 'bats-mock/load'; }

@test "deploy calls git push" {
  stub git "push origin main : echo 'MOCK'"
  run ./deploy.sh
  assert_success
  unstub git    # Verifies all stubs were called
}
```

**Best practices:** Always restore `PATH` in `teardown()`. Log mock invocations to stderr. Verify call count. Return appropriate exit codes for both success and error paths.

---

### 4.2 Testing with Temporary Directories

```bash
setup() {
  TEST_TEMP_DIR=$(mktemp -d)
  mkdir -p "$TEST_TEMP_DIR/input/valid"
  echo "data" > "$TEST_TEMP_DIR/input/valid/data.txt"
}
teardown() { rm -rf "$TEST_TEMP_DIR"; }

@test "creates output file" {
  run ./generate.sh "$TEST_TEMP_DIR/output.txt"
  assert_success
  assert_file_exist "$TEST_TEMP_DIR/output.txt"
}
```

For standalone scripts (no framework): `trap cleanup EXIT` where `cleanup() { rm -rf "$TEMP_DIR"; }`.

---

### 4.3 Testing Exit Codes

```bash
@test "returns 0 for valid input" { run ./validate.sh "valid"; [ "$status" -eq 0 ]; }
@test "returns 1 for invalid input" { run ./validate.sh ""; [ "$status" -eq 1 ]; }
@test "returns 127 for missing command" { run nonexistent-command; [ "$status" -eq 127 ]; }
```

Convention: 0=success, 1=general error, 2=misuse, 126=not executable, 127=not found, 128+=fatal signal.

---

### 4.4 Testing Output

```bash
@test "prints usage"       { run ./script.sh --help; assert_output --partial "Usage:"; }
@test "exactly 5 lines"    { run ./list-files.sh; [ "${#lines[@]}" -eq 5 ]; }
@test "header at line 0"   { run ./report.sh; assert_line --index 0 "Header"; }
@test "valid JSON output"  { run ./config.sh; echo "$output" | jq . > /dev/null 2>&1; assert_success; }
```

---

### 4.5 Testing with `set -euo pipefail`

```bash
@test "works under strict mode" {
  run bash -euo pipefail ./script.sh
  assert_success
}

@test "catches unset variable" {
  run bash -euo pipefail -c 'unset VAR; echo "$VAR"'
  assert_failure
  [[ "$output" == *"unbound variable"* ]]
}

@test "catches pipeline failure" {
  run bash -euo pipefail -c 'false | echo piped'
  assert_failure
}
```

`set -e`: exit on error. `set -u`: error on unset variables. `set -o pipefail`: fail pipeline if any command fails.

---

## 5. CI/CD Integration

### GitHub Actions

```yaml
name: Test Bash Scripts
on: [push, pull_request]
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: ShellCheck
        run: shellcheck --severity=warning src/**/*.sh tests/**/*.bats
      - name: Format check
        run: shfmt -d src/ tests/
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run tests
        run: |
          sudo apt update && sudo apt install -y bats
          bats tests/
  coverage:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: sudo apt update && sudo apt install -y bats kcov
      - run: kcov --exclude-pattern=/usr/lib coverage bats tests/
      - uses: actions/upload-artifact@v4
        with:
          name: coverage
          path: coverage/
```

### pre-commit Hooks

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/shellcheck-py/shellcheck-py
    rev: v0.10.0
    hooks:
      - id: shellcheck
        args: ["--severity=warning"]
        files: \.(sh|bash|bats)$
  - repo: https://github.com/scop/pre-commit-shfmt
    rev: v3.9.0
    hooks:
      - id: shfmt-docker
        args: ["-w", "-i", "2"]
        files: \.(sh|bash|bats)$
```

```bash
pip install pre-commit && pre-commit install
pre-commit run --all-files
```

---

## 6. Testing Project Structure

```
my-bash-project/
├── src/
│   ├── main.sh
│   └── lib/           # utils.sh, validate.sh, config.sh
├── tests/
│   ├── test_*.bats                    # Per-module unit tests
│   ├── integration/test_pipeline.bats # Integration tests
│   ├── fixtures/                      # Test fixture data (config files, sample output)
│   └── helpers/common.bash            # Shared test helpers
├── mocks/              # Mock executables (git, docker, curl)
├── .shellcheckrc
├── .pre-commit-config.yaml
├── Makefile
└── .github/workflows/test.yml
```

**Example Makefile:**

```makefile
test:          ; bats tests/
test-parallel: ; bats --jobs $(shell nproc) tests/
lint:          ; shellcheck --severity=warning src/**/*.sh tests/**/*.bats
format:        ; shfmt -w src/ tests/
format-check:  ; shfmt -d src/ tests/
coverage:      ; kcov --exclude-pattern=/usr/lib coverage bats tests/
```

---

## 7. Advanced Techniques

### 7.1 Test Coverage

**kcov** instruments bash scripts via debug trace hooks.

```bash
sudo apt install kcov
kcov coverage/ bats tests/
kcov --include-pattern=src/ coverage/ bats tests/
```

Generates HTML at `coverage/index.html` — green lines executed, red unexecuted, per-file percentages. **Limitations:** No per-test tracking, no delta reporting, performance overhead.

### 7.2 Property-Based Testing (Fuzzing)

```bash
@test "handles random strings" {
  for i in {1..100}; do
    input=$(base64 /dev/urandom | head -c $((RANDOM % 64 + 1)))
    run ./sanitize.sh "$input"
    assert_success
  done
}
```

Test **invariant properties**: "output never empty" → `[ -n "$output" ]`, "exit always 0 or 1" → `[ "$status" -eq 0 ] || [ "$status" -eq 1 ]`, "output always valid JSON" → `echo "$output" | jq .`.

### 7.3 Testing Timeouts

```bash
timeout 5 bats tests/test_slow.bats                      # Kill run if test hangs

@test "network call times out gracefully" {
  timeout 3 ./fetch.sh "http://httpbin.org/delay/10"
  assertEquals 124 $?   # timeout's exit code
}

@test "slow rebuild" {
  [ "${CI:-}" = "true" ] || skip "Only runs in CI"
  run ./rebuild.sh && assert_success
}
```

### 7.4 Testing with Docker

```bash
docker run --rm -v "$(pwd):/test" -w /test ubuntu:24.04 bash -c "
  apt update > /dev/null 2>&1
  apt install -y bats shellcheck > /dev/null 2>&1
  shellcheck src/**/*.sh tests/**/*.bats
  bats tests/
"
```

Guarantees clean, reproducible environment. Useful for testing across bash versions or OS distributions.

---

## 8. Comparison: bats vs shunit2

| Feature | bats | shunit2 |
|---|---|---|
| Style | TAP (Test Anything Protocol) | JUnit / xUnit style |
| Setup/Teardown | `setup()` / `teardown()` | `setUp()` / `tearDown()` |
| File-level setup | `setup_file()` / `teardown_file()` (1.7+) | ❌ |
| Assertions | Via helpers (bats-assert, bats-file) | Built-in (`assertEquals`, `assertNull`, etc.) |
| Output capture | Built-in `run` → `$output`/`$status`/`$lines` | Manual: `result=$(cmd)` |
| Test discovery | `@test "name" { }` | Function naming (`testEquality`) |
| TAP output | ✅ Native | ❌ Custom formatters needed |
| Parallel execution | ✅ `bats --jobs N` | ❌ Sequential only |
| CI integration | ✅ Excellent (native TAP) | ✅ Good (needs TAP wrapper) |
| Community | ✅ Largest ecosystem | ⚠️ Smaller |
| Shell compatibility | bash 4.x+, zsh | bash, zsh, ksh, dash (POSIX) |
| Active development | ✅ High (bats-core) | ⚠️ Stable, infrequent releases |
| Dependencies | bash + coreutils | bash + coreutils + `expr` |
| Learning curve | Low | Very low |

---

## 9. Recommendations

| Scenario | Tool |
|---|---|
| New project, need framework | **bats** — most popular, TAP output, parallel, best ecosystem |
| Simple suite, JUnit familiarity | **shunit2** — POSIX compatible, minimal learning curve |
| Linting / quality | **ShellCheck** — essential, run before any test framework |
| Formatting / style | **shfmt** — `-d` for CI checks, `-w` for auto-fix |
| Coverage | **kcov** — HTML reports, simple setup |
| Docker scripts | **bats in Docker** — clean environment isolation |
| Pre-commit | **pre-commit hooks** — shellcheck + shfmt together |
| CI/CD | **GitHub Actions** — pre-installed tools, TAP integration |
| Mocking | **PATH manipulation** + **bats-mock** |
| Cross-version testing | **Docker matrix** — test bash 3.2 through 5.2 |
| Large suites | **bats + parallel** — `--jobs N` for speed |
| Embedded / busybox | **shunit2** — works with minimal shells |

### Recommended Workflow

1. Write bash functions in `src/lib/*.sh`
2. Run `shfmt -w` to format
3. Run `shellcheck` at every save (editor integration)
4. Write bats tests in `tests/test_*.bats`
5. Run `bats tests/` locally (pre-commit hook)
6. Push → GitHub Actions runs shellcheck → bats → kcov
7. Merge only when all checks pass

---

## 10. Quick Start: bats

```bash
# Install
sudo apt update && sudo apt install -y bats shellcheck

# First test
cat > test_example.bats <<'EOF'
#!/usr/bin/env bats
setup() { test_dir=$(mktemp -d); }
teardown() { rm -rf "$test_dir"; }
@test "directory created" {
  mkdir -p "$test_dir/subdir"
  [ -d "$test_dir/subdir" ]
}
EOF

# Run
bats test_example.bats

# TAP format (CI)
bats --formatter tap test_example.bats

# Lint
shellcheck test_example.bats
```

You now have a complete, working bash testing setup. From here, expand by adding bats-assert for friendlier assertions, structuring your project with `src/` and `tests/`, setting up GitHub Actions with the workflow from Section 5, and adding coverage with kcov as your suite grows.

---

## Data Sources

- [bats-core](https://github.com/bats-core/bats-core) — bats documentation
- [ShellCheck](https://www.shellcheck.net) — static analysis docs
- [shunit2](https://github.com/kward/shunit2) — xUnit-style framework
- [shfmt](https://github.com/mvdan/sh) — shell formatter
- [kcov](https://simonkagstrom.github.io/kcov) — code coverage
- [GitHub Actions](https://docs.github.com/en/actions/automating-builds-and-tests) — CI for bash
- [pre-commit](https://pre-commit.com) — hook framework
