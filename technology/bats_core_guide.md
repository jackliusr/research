# bats-core: The Definitive Guide to Testing Bash Scripts

**Bats** (short for **Bash Automated Testing System**) is the de facto standard testing framework for Bash scripts. This guide covers what bats-core is, why you should test shell code, core concepts and syntax, the official assertion libraries, advanced features, alternatives, use cases, best practices, and a full CLI/environment reference.

## 1. What is bats-core?

bats-core is a **TAP (Test Anything Protocol)-producing testing framework for Bash scripts**, written entirely in Bash itself. It gives shell scripts what JUnit gives Java and pytest gives Python: a structured way to write test cases, run them, report pass/fail, and wire results into CI.

- **Project home:** <https://bats-core.github.io/> (docs), <https://github.com/bats-core/bats-core> (source)
- **Also known as:** Bats, Bats-core, `bats` (the executable)
- **License:** MIT; **language:** 100% Bash — no runtime dependencies beyond Bash itself (plus GNU `parallel` for parallel execution)
- **Output:** TAP — a machine-readable text protocol (`ok 1 ...`, `not ok 2 ...`) consumed by CI systems, test runners, and formatters

**What it is used for:** testing anything that lives in a shell script — **shell scripts**, **CLI tools** (argument parsing, validation, output, exit codes), **Docker entrypoints** (`entrypoint.sh`), **deployment scripts**, **CI/CD pipeline scripts**, and **infrastructure automation** (Terraform wrappers, cloud-init, Ansible helpers).

**Design philosophy:** deliberately minimal. A test file is just a Bash script with `@test` blocks; assertions are plain Bash conditionals or helpers from `bats-assert`. There is no magic DSL beyond a handful of built-ins (`run`, `load`, `skip`, `setup`, `teardown`). If you can write a Bash `if`, you can write a Bats test.

## 2. History

- **2011** — Created by **Sam Stephenson** (creator of rbenv) as `sstephenson/bats`; 0.4.0 was the last release under him, then the project stalled
- **2017** — The **bats-core organization** took over maintenance, consolidating the community fork under `bats-core/bats-core`
- **2019** — **v1.0.0**: API stabilization, major performance boost, Bash 4.2+ support
- **2021** — v1.5.0 (per-test temp dirs), v1.6.0 (parallel execution)
- **2022** — v1.8.0 (suite setup/teardown, tags, JUnit formatter)
- **2023–2025** — v1.10.0, v1.11.0/v1.11.1, v1.12.0 (failure hooks), v1.13.0 (bugfixes)

The bats-core org now maintains the whole ecosystem: `bats-core`, `bats-assert`, `bats-support`, `bats-file`, `bats-extra`, `bats-detik`.

## 3. Why test Bash at all?

Shell scripts are everywhere and are often the most brittle code in an organization: **CI/CD pipelines** (build/test/deploy orchestration), **deployment scripts**, **Docker entrypoints** (the first thing every container runs), **cron jobs** (backups, cleanup, reporting), and **infrastructure automation** (IaC wrappers, provisioning).

**What bugs in shell scripts actually cost:** failed deployments (a typo in a deploy script takes down a release), data loss (`rm -rf "$DIR/"` with an unset `DIR` becomes `rm -rf /`), security vulnerabilities (unquoted variables, unsafe `eval`, missing input validation), and downtime (a broken entrypoint means containers that never start).

**Why manual testing is not enough:** shell code is written once and rarely touched, so regressions go unnoticed; "run it and see" doesn't cover error paths, missing arguments, or cleanup behavior; there is no compiler — Bash parses lazily, so syntax errors surface at runtime, in production; and untestable logic accumulates precisely where it's hardest to verify.

**What bats-core gives you:** automated regression testing (the same assertions on every change), TDD for shell scripts (write the test first, watch it fail, make it pass), CI integration (exit code 0 = pass, non-zero = fail; TAP output is machine-readable), and structured test suites (named test cases, shared setup/teardown, helper libraries).

## 4. Core concepts

### 4.1 Test file

A **test file** is a `.bats` file containing one or more test cases, executed with the `bats` command. The file is itself a Bash script (extension configurable via `BATS_FILE_EXTENSION`):

```bash
#!/usr/bin/env bats

@test "addition works" {
  run echo $((2 + 2))
  [ "$status" -eq 0 ]
  [ "$output" = "4" ]
}
```

### 4.2 Test case

A **test case** is a `@test` block with a name and a body of Bash code. Every test runs in its **own subshell**, so variables and `cd` don't leak between tests. A test **passes if it exits 0**; a non-zero exit — from a failed assertion or an uncaught error — **fails the test**. The name is printed in TAP output, so names should read as specifications.

### 4.3 Test run

```bash
bats testfile.bats              # run one file
bats test/                      # run all *.bats files in a directory
bats test/unit test/integration # multiple paths
```

### 4.4 TAP output

Bats emits **TAP (Test Anything Protocol)** — the standard text format for test results, originally created for Perl's Test::Harness:

```text
1..3
ok 1 addition works
not ok 2 subtraction works
# (in test file testfile.bats, line 7)
#   `[ "$output" = "1" ]' failed
ok 3 multiplication works
```

`ok N name` / `not ok N name` = passed/failed, followed by `#` diagnostic lines (file, line, failing command); `1..N` is the plan line declaring the test count; `skip` marks a test as skipped rather than failed. TAP is what makes Bats CI-friendly — Jenkins (TAP plugin), GitLab, and others ingest it natively, and Bats ships formatters for JUnit XML and TAP13.

### 4.5 `setup()` and `teardown()`

Per-test lifecycle hooks: **`setup()`** runs **before each test** (create temp files, set env vars, start services); **`teardown()`** runs **after each test, even on failure** (delete temp files, stop services, kill background processes).

```bash
setup() { export TEST_DIR=$(mktemp -d); export GREETING="hello"; }
teardown() { rm -rf "$TEST_DIR"; }

@test "greeting is written to file" {
  echo "$GREETING" > "$TEST_DIR/greeting.txt"
  run cat "$TEST_DIR/greeting.txt"
  [ "$output" = "hello" ]
}
```

### 4.6 `setup_file()` and `teardown_file()`

File-level hooks (Bats 1.2+): **`setup_file()`** runs **once before all tests in a file** (one-time expensive setup like building a binary or pulling an image); **`teardown_file()`** runs **once after them**. They run in a subshell, so `export` anything tests need.

### 4.7 `run` — capturing output and status

`run` executes a command and captures the result in three special variables: **`$status`** (exit code), **`$output`** (combined stdout **and** stderr, trailing newlines stripped), and **`$lines`** (array of output lines, index 0 = first).

```bash
@test "git status is clean" {
  run git status --porcelain
  [ "$status" -eq 0 ]
  [ -z "$output" ]
}
```

`run` is the standard way to invoke the thing under test. Without it, a failing command fails the test directly and its output can't be inspected.

### 4.8 Assertions

An **assertion** is any expression that must hold for the test to pass. Raw form: Bash conditionals — `[ "$status" -eq 0 ]`, `[ "$output" = "expected" ]`, `[ "${lines[0]}" = "first" ]`. On failure, Bats reports the failing expression with file and line number. For richer diagnostics, use `bats-assert` (Section 6).

### 4.9 `skip` — skipping tests

`skip "reason"` marks the current test as **skipped**: reported as `ok N # skip reason` in TAP, and it does **not** fail the run. Typical uses: **platform-specific tests** (skip unless the OS matches), **not-yet-implemented / TODO tests** (show as skipped, not failing), and **conditional skips** (skip when an optional dependency is missing).

```bash
@test "uses GNU sed" {
  if ! command -v gsed >/dev/null 2>&1 && [[ "$(uname)" == "Darwin" ]]; then
    skip "GNU sed not installed"
  fi
  run sed --version
  [ "$status" -eq 0 ]
}
```

## 5. Test file structure

### 5.1 A minimal `.bats` file

```bash
#!/usr/bin/env bats

@test "greet says hello" {
  run ./greet.sh world
  [ "$status" -eq 0 ]
  [ "$output" = "Hello, world!" ]
}
```

The shebang makes the file directly executable; it's optional when running via `bats`.

### 5.2 Loading helpers: `load`

`load` sources another Bash file into the test context: `load 'test_helper'` (loads `./test_helper.bash` relative to the test file), `load 'test_helper.bash'` (explicit extension), or `load 'lib/assertions'` (relative path). Paths resolve **relative to the test file's directory** unless absolute; `.bash` is appended automatically if missing. Helper files typically hold functions shared across test files. For installed libraries, load their entrypoint: `load 'test_helper/bats-assert/load'`.

### 5.3 Standard layout

```text
project/
├── src/
│   ├── deploy.sh
│   └── backup.sh
└── test/
    ├── test_helper.bash        # shared helpers
    ├── deploy.bats
    ├── backup.bats
    └── test_helper/            # vendored libs: bats-support, bats-assert, bats-file
```

### 5.4 Common patterns

```bash
#!/usr/bin/env bats

load 'test_helper'
load 'test_helper/bats-support/load'
load 'test_helper/bats-assert/load'

setup() { export TMPDIR_TEST=$(mktemp -d); }
teardown() { rm -rf "$TMPDIR_TEST"; }

# helper function shared by tests
make_config() {
  printf 'key=%s\n' "$1" > "$TMPDIR_TEST/config.ini"
}

@test "config reader returns configured value" {
  make_config "port=8080"
  run ./read_config.sh "$TMPDIR_TEST/config.ini"
  assert_success
  assert_output "port=8080"
}
```

## 6. Assertion libraries

Bats core ships no assertion helpers beyond raw `[ ]` — the ecosystem fills that gap. Libraries are usually vendored into `test/test_helper/` and loaded with `load`.

### 6.1 bats-assert — assertion helpers

**Repo:** <https://github.com/bats-core/bats-assert>. Load after `bats-support`:

```bash
load 'test_helper/bats-support/load'
load 'test_helper/bats-assert/load'
```

| Assertion | Meaning |
|---|---|
| `assert_success` | last `run` exited 0 |
| `assert_failure` (optionally `assert_failure 2`) | last `run` exited non-zero |
| `assert_output "text"` | `$output` equals the string exactly |
| `assert_output --partial "text"` (`-p`) | `$output` contains the substring |
| `assert_output --regex 'patt'` (`-e`) | `$output` matches the regex |
| `refute_output "text"` / `--partial "text"` | `$output` does NOT equal / contain the string |
| `assert_line "text"` | a line of `$output` equals the string (`--index N`, `--partial`, `--regex`) |
| `refute_line "text"` | no line of `$output` equals the string |
| `assert_equal a b` | two values are equal |

On failure these produce readable diagnostics (expected vs. actual) instead of a bare `[ ]` failure:

```bash
@test "script prints version" {
  run ./tool.sh --version
  assert_success
  assert_output --partial "tool version 2.0"
}
```

### 6.2 bats-support — support library

**Repo:** <https://github.com/bats-core/bats-support>. Low-level utilities that power bats-assert (rarely called directly, but must be loaded first): `batslib_decorate` (format multi-line failure output), `batslib_err` (stderr output), `batslib_count_lines`/`batslib_get_lines` (line counting/extraction), `batslib_prefix` (indent lines), `batslib_mark` (mark a specific line in diagnostics), `batslib_trim` (trim whitespace), and `batslib_print_kv_single`/`batslib_print_kv_multi` (key/value rendering).

### 6.3 bats-file — file and directory assertions

**Repo:** <https://github.com/bats-core/bats-file>. Invaluable for deploy/backup testing:

| Assertion | Checks that |
|---|---|
| `assert_file_exist <path>` / `refute_file_exist` (alias `assert_file_not_exist`) | the file exists / does NOT exist |
| `assert_file_contains <file> <pattern>` | the file contains the regex pattern |
| `assert_file_empty <file>` | the file exists and is empty |
| `assert_file_size <file> <bytes>` | exact byte size (also `--lt`, `--gt`, `--le`, `--ge`) |
| `assert_file_mode <file> <mode>` | octal mode, e.g. `assert_file_mode script.sh 0755` |
| `assert_file_executable` / `assert_file_not_executable` | executable bit |
| `assert_dir_exist <dir>` / `refute_dir_exist` | directory existence |
| `assert_symlink_to <target> <link>` | path is a symlink to target |
| `assert_owner`, `assert_group`, `assert_link_exist`, ... | and more |

```bash
@test "backup script creates a tar.gz" {
  run ./backup.sh --out "$TMP"
  assert_success
  assert_file_exist "$TMP/backup.tar.gz"
  assert_file_size "$TMP/backup.tar.gz" --gt 0
}
```

### 6.4 bats-detik — Kubernetes / e2e assertions

**Repo:** <https://github.com/bats-core/bats-detik>. Wraps `kubectl` for Kubernetes end-to-end testing: `assert_pod_status`/`assert_pod_ready` (pod readiness), `assert_service_status` (service availability), `try "cmd" "times" "delay"` (retry loops for eventual consistency), `verify "cmd"` (retrying assertion), `get_pod_name`, `run_try`. Useful for testing Kubernetes operators, Helm charts, and infrastructure code with Bats as the harness.

### 6.5 bats-mock — mocking and stubbing

**Repo:** <https://github.com/jasonkarns/bats-mock>. The de facto standard for stubbing external commands: create **mock commands** replacing real binaries (`curl`, `docker`, `git`), **stub behavior** (scripted outputs and exit codes), and **verify calls** (how many times, with which arguments).

```bash
load 'test_helper/bats-mock/load'

setup() {
  mock=$(mock_create); export GIT="$mock"; stub "$mock" "status --porcelain : echo modified"
}

@test "deploy fails on dirty working tree" {
  run ./deploy.sh
  assert_failure
  assert_output --partial "working tree is dirty"
  assert_equal "$(mock_get_call_num "$mock")" 1
}
```

### 6.6 bats-extra — additional helpers

**Repo:** <https://github.com/bats-core/bats-extra>. Grab-bag of extra assertions: `assert_contains`, `assert_json`/`assert_valid_json` (JSON assertions), `assert_no_duplicates`, random/uuid helpers, and more. Load with `load 'test_helper/bats-extra/load'`.

### 6.7 Using the libraries — load patterns

The standard recipe for the full ecosystem (order matters — bats-support first):

```bash
#!/usr/bin/env bats

load 'test_helper/bats-support/load'   # must come first
load 'test_helper/bats-assert/load'    # depends on bats-support
load 'test_helper/bats-file/load'
load 'test_helper/bats-mock/load'

@test "everything wired up" {
  assert_file_exist ./README.md
  run echo ok
  assert_success
  assert_output "ok"
}
```

Install via git submodules, `git clone` into `test/test_helper/`, or your package manager. Load paths are relative to the test file.

## 7. Advanced features

### 7.1 Parameterized tests

Bats has no built-in `@cases` syntax — parameterize with plain Bash loops and arrays:

```bash
@test "string utilities" {
  for case in "hello|5" "world!|6" "|0"; do
    run ./strlen.sh "${case%%|*}"
    assert_success
    assert_output "${case##*|}"
  done
}
```

For one TAP line per case, use a **function that emits `@test` blocks**:

```bash
case_test() {
  local input="$1" expected="$2"
  @test "strlen('$input') == $expected" {
    run ./strlen.sh "$input"
    assert_success
    assert_output "$expected"
  }
}
case_test "hello" 5
case_test "" 0
case_test "a b c" 5
```

### 7.2 Mocking and stubbing

- **Override commands with stubs** — a fake executable earlier in `PATH` wins over the real one
- **Use bats-mock** for scripted behavior, call counts, and argument logs
- **Stub external commands** — `curl`, `docker`, `git`, `aws`, `kubectl` are the usual suspects; never hit the network in unit tests
- **Control return values** — make the stub exit non-zero to exercise error paths

```bash
setup() {
  docker() { echo "Docker $*"; return 0; }   # fake docker that always "succeeds"
  export -f docker
}
```

### 7.3 Parallel test execution

```bash
bats --jobs 4 test/
```

Requires **GNU `parallel`** (or a compatible `flock`/`shlock` backend). Files run in parallel; tests within a file run serially by default. A file can opt out with `--no-parallelize` (older) or the `# bats file_tags=...` pragma (newer versions).

Caveats: **temp file collisions** (each test has its own `BATS_TEST_TMPDIR`, but shared fixtures must be created in `setup_file`, not `setup`), **port conflicts** (tests binding ports must use distinct ports), and **shared state** (env vars set in one test can race with another — keep tests independent). `setup_file`/`teardown_file` are per-file and safe; `setup_suite` runs once globally.

### 7.4 Test filters

```bash
bats --filter "deploy" test/              # run tests whose NAME matches /deploy/
bats --filter-tags "unit" test/           # run tests tagged "unit"
bats --filter "retry" --filter-tags "slow" test/   # combine filters
```

`-f, --filter <regex>` runs only tests whose name matches the regex; `--filter-tags <tags>` runs only tests with the given tags (comma-separated = AND, repeated flags = OR). Descriptive test names double as a selection mechanism.

### 7.5 Tags

Tag tests with metadata (Bats 1.8+):

```bash
@test "unit: parser handles empty input" --tag "unit" {
  run ./parser.sh ""
  assert_success
}

@test "integration: full pipeline" --tag "integration" --tag "slow" {
  run ./pipeline.sh
  assert_success
}
```

Syntax: `@test "name" --tag "tagname"` (repeatable); file-level via `# bats file_tags=foo,bar`. Filter with `--filter-tags "unit"` or `--filter-tags "unit,fast"` (AND), multiple flags for OR. Typical use: CI grouping — `unit` on every commit, `integration` nightly, `slow` on release branches.

### 7.6 Tracing and debugging

```bash
bats -x testfile.bats                     # -x / --trace: print each command (like bash -x)
bats -t testfile.bats                     # -t / --tap: raw TAP, no colors
bats -p testfile.bats                     # -p / --pretty: human-readable output
bats --print-output-on-failure test/      # dump captured output on failure
bats --show-output-of-passing-tests test/ # also show passing tests' output
bats --timing test/                       # elapsed time per test
```

Inside a test, `set -x` gives the same trace; `BATS_DEBUG=1` enables the framework's own debug logging. On failure Bats already prints the failing source line with file/line; `--print-output-on-failure` additionally dumps the full captured `$output`.

### 7.7 Pre/post hooks — the full lifecycle

| Hook | Scope | Runs |
|---|---|---|
| `setup_suite()` | whole run | once, before all files (Bats 1.8+) |
| `setup_file()` | one file | once, before that file's tests (1.2+) |
| `setup()` | one test | before each test |
| `teardown()` | one test | after each test, even on failure |
| `teardown_file()` | one file | once, after the file's tests |
| `teardown_suite()` | whole run | once, after everything (Bats 1.8+) |

```bash
setup_suite() { mkdir -p "$BATS_SUITE_TMPDIR"; export SUITE_START=$(date +%s); }
teardown_suite() { rm -rf "$BATS_SUITE_TMPDIR"; }
```

`setup_suite` runs in a separate subshell — `export` anything tests need.

### 7.8 Coverage measurement

- **kcov** — the standard code coverage tool for shell scripts (<https://github.com/SimonKagstrom/kcov>); instruments Bash and works with Bats out of the box:

```bash
kcov --include-path=src coverage bats test/   # produces coverage/index.html, coverage.json
```

- **bashcov** — Ruby-based alternative; simpler in some setups, less maintained
- **Codecov / Coveralls** — upload kcov's `coverage.json` for PR coverage gates in CI
- For shell code, prioritize covering **error paths** (the expensive failure modes) over line-count chasing

## 8. Bats vs alternatives

**vs shellcheck** — shellcheck is a **linter** (static analysis: unquoted variables, missing `set -e`, pitfalls, without running code); Bats is a **test framework** (dynamic testing: runs code, checks behavior). They are **complementary**: run shellcheck on every script in CI and Bats on the critical paths. A script can pass shellcheck and still be wrong, and vice versa.

**vs shunit2** — the older (2008-era) xUnit-style framework for POSIX sh/Bash (`setUp`/`tearDown`, `assertEquals`, `test_*` functions). **Less active, less feature-rich**, no TAP output, weaker CI story. Bats is more modern: TAP, parallel execution, formatters, active org with assertion libraries.

**vs assert.sh** — a minimal assertion library (`assert`, `assert_raises`, ...), **not a framework**: no test discovery, no TAP, no lifecycle hooks. A helper for hand-rolled test scripts.

**vs Bats + bats-assert** — bare Bats gives structure + TAP + `run`/`$status`/`$output` with raw `[ ]`; adding bats-assert gives readable assertions (`assert_success`, `assert_output --partial`) with excellent failure diagnostics. Use the full stack for real projects.

**vs pytest (shell from Python)** — you can drive shell commands from pytest (`subprocess.run`), but you lose Bash-native constructs (`run`, `$lines`), direct sourcing of Bash helpers, and testing functions *inside* a script. Choose pytest only if the logic under test is mostly Python.

**vs custom test scripts** — DIY scripts (`if [ ... ]; then echo PASS; else exit 1; fi`) work for 5 tests and break at 50: no isolation (one failure aborts the rest), no TAP, no filters, no parallel, no CI integration. Bats is structured, TAP-producing, CI-ready — setup cost is minutes.

**vs shellspec** — the main alternative (<https://shellspec.info>): an RSpec-inspired DSL for POSIX shells with `Describe`/`Context`/`It` blocks and expectations like `The output should equal ...`. More expressive for large suites; supports sh/bash/ksh/dash. But **smaller community**, fewer ecosystem libraries, steeper learning curve than Bats' minimal syntax.

**Comparison table:**

| Dimension | **Bats** | **shunit2** | **shellspec** | **Custom scripts** |
|---|---|---|---|---|
| Syntax style | Minimal `@test` blocks, Bash-native | xUnit (`test_*`, `assertEquals`) | RSpec-like DSL (`Describe`/`It`) | Whatever you hand-roll |
| TAP output | ✅ native | ❌ | ❌ (custom reporters) | ❌ |
| Assertion library | bats-assert + bats-file + bats-mock | built-in assertEquals family | built-in expectation DSL | none |
| Mocking | bats-mock, PATH stubs | manual | built-in `Mock` | manual |
| Parallel execution | ✅ `--jobs` (GNU parallel) | ❌ | ✅ | ❌ |
| CI integration | ✅ exit codes + TAP + JUnit | basic exit codes | reporters | exit codes only |
| Community / activity | ✅ large, active org | small, slow | small but active | n/a |
| Learning curve | Low (Bash + 6 built-ins) | Low–medium | Medium–high (new DSL) | High to do well |

**Bottom line:** use Bats unless you need POSIX-sh portability at scale (shellspec) or are maintaining a legacy shunit2 suite. For pure Bash, Bats is the default — which is why it's the de facto standard.

## 9. Use cases

### 9.1 Deployment scripts

Verify `deploy.sh` behavior without deploying — env var validation, file creation, dependency checks, rollback:

```bash
@test "deploy.sh requires ENV_NAME" {
  run ./deploy.sh
  assert_failure
  assert_output --partial "ENV_NAME is required"
}

@test "deploy.sh creates backup before deploy" {
  run ./deploy.sh --env staging
  assert_success
  assert_file_exist "${BACKUP_DIR}/backup.tar.gz"
}

@test "deploy.sh retries on failure" {
  run ./deploy.sh --env staging --retry 3
  assert_success
}

@test "deploy.sh rolls back when post-deploy checks fail" {
  health_check() { return 1; }; export -f health_check   # force failure
  run ./deploy.sh --env staging
  assert_failure
  assert_output --partial "rolling back"
  assert_file_not_exist "$APP_DIR/.deployed"
}
```

### 9.2 Docker entrypoints

Test `entrypoint.sh` exactly as the container runs it — command execution, env var defaults, signal handling, exit codes, health check behavior:

```bash
@test "entrypoint defaults PORT to 8080" {
  run env -u PORT ./entrypoint.sh --print-port
  assert_success
  assert_output "8080"
}

@test "entrypoint exits non-zero on missing config" {
  run env -u APP_CONFIG ./entrypoint.sh
  assert_failure
  assert_output --partial "APP_CONFIG is required"
}

@test "entrypoint forwards SIGTERM to child" {
  ./entrypoint.sh & local pid=$!
  kill -TERM "$pid"; wait "$pid" || true
  assert_output --partial "shutting down"   # graceful-shutdown log line
}
```

### 9.3 CLI tools

CLI behavior is the most natural Bats target — argument parsing, validation, output format, error messages, exit codes, help text:

```bash
@test "cli parses --output flag" {
  run ./cli.sh --output /tmp/out.txt input.txt
  assert_success
  assert_file_exist /tmp/out.txt
}

@test "cli rejects unknown flag" {
  run ./cli.sh --bogus
  assert_failure
  assert_output --partial "unknown option"
}

@test "cli --help prints usage" {
  run ./cli.sh --help
  assert_success
  assert_line --partial "Usage:" && assert_line --partial "Options:"
}
```

### 9.4 CI/CD pipeline scripts

Test pipeline glue — step ordering, artifact handling, failure handling, retry logic — stubbing `git`, `docker`, and artifact stores:

```bash
@test "pipeline skips deploy step when tests fail" {
  run ./pipeline.sh
  assert_success
  assert_output --partial "step: test -> FAILED"
  assert_output --partial "step: deploy -> SKIPPED"
}
```

### 9.5 Infrastructure automation

IaC wrapper scripts, cloud-init, Ansible helpers — anything that provisions or configures: Terraform wrappers (plan/apply/destroy flow, state handling), cloud-init scripts (**idempotency**: run twice, same result), Ansible helpers (inventory parsing, vault handling):

```bash
@test "terraform wrapper runs plan before apply" {
  run ./tf.sh apply
  assert_success
  assert_output --partial "terraform plan"
  assert_output --partial "terraform apply"
}

@test "cloud-init script is idempotent" {
  run ./cloud-init.sh; run ./cloud-init.sh
  assert_success
  assert_output "already configured"
}
```

### 9.6 Cron jobs / scheduled tasks

Cron scripts run unattended — logging and error handling are the critical contract:

```bash
@test "backup cron logs success" {
  run ./cron/backup.sh
  assert_success
  assert_file_contains "$LOG_DIR/backup.log" "backup completed"
}

@test "backup cron alerts on failure" {
  sendmail() { echo "MAIL: $*" >&2; }; export -f sendmail
  run ./cron/backup.sh --fail-simulated
  assert_failure
  assert_output --partial "MAIL:"
}
```

## 10. Best practices

- **Organize tests by script/module:** `test/` directory, one `.bats` file per script, mirroring source structure (`src/deploy.sh` → `test/deploy.bats`); shared helpers in `test/test_helper.bash` + vendored libraries under `test/test_helper/`.
- **Test naming:** names are specifications — write behavior statements (`@test "deploy.sh should fail when ENV_NAME is unset"`, `@test "parser returns an empty list for an empty file"`). Good names power `--filter` selection and make failure reports self-explanatory.
- **Keep tests independent:** no shared state (each test runs in its own subshell, but files and env vars persist); use `setup`/`teardown` for isolation; avoid global variables; never rely on execution order — parallel mode makes it non-deterministic.
- **Use temp directories:** `mktemp -d` in `setup`, `rm -rf` in `teardown` — or use Bats' **`BATS_TEST_TMPDIR`** (Bats 1.5+), a unique per-test temp dir cleaned up automatically:

```bash
@test "writes output file" {
  run ./tool.sh -o "$BATS_TEST_TMPDIR/out.txt"
  assert_file_exist "$BATS_TEST_TMPDIR/out.txt"
}
```

(`BATS_FILE_TMPDIR` per file, `BATS_SUITE_TMPDIR` per run.)

- **Prefer `run` + bats-assert over raw `[ ]`:** `run ./tool.sh; assert_success; assert_output "expected"` beats `[ "$status" -eq 0 ]; [ "$output" = "expected" ]` — better failure messages ("expected: X, actual: Y") and reads as a spec.
- **Test error paths:** the most expensive bugs live in failure handling — missing args/unset env vars, invalid input, permission errors (read-only dirs, non-executable scripts), resource exhaustion (disk full / fd limits via stubs).
- **Use fixtures:** test data files under `test/fixtures/` referenced via `BATS_TEST_DIRNAME`; mock external services; prefer deterministic data — no random ports, no wall-clock-dependent assertions.
- **CI integration:** run Bats on every push (GitHub Actions `bats/bats-action`, Makefile target, GitLab job); emit JUnit XML (`--formatter junit`) for reporting dashboards; upload coverage (kcov → Codecov); use `--jobs` in CI for speed, `--jobs 1` when debugging.
- **Version control:** commit tests with the code they test; review tests like any other code; update tests when behavior changes — a failing suite after a refactor is a feature, not a nuisance.
- **Keep tests fast:** avoid network calls (stub `curl`/`docker`/`git`); prefer unit-level tests (source the function, call it) over integration-level tests (spin up containers); run in parallel where safe; split slow e2e suites into nightly runs.

## 11. Environment and configuration

### 11.1 `BATS_*` environment variables

| Variable | Meaning |
|---|---|
| `BATS_VERSION` | Bats version string, e.g. `1.11.1` |
| `BATS_TEST_TMPDIR` | unique per-test temp dir (Bats 1.5+), auto-cleaned |
| `BATS_FILE_TMPDIR` / `BATS_SUITE_TMPDIR` | per-file / per-run temp dirs |
| `BATS_TEST_DIRNAME` | directory containing the current test file |
| `BATS_TEST_FILENAME` | full path of the current test file |
| `BATS_TEST_NAME` | name of the current test |
| `BATS_TEST_DESCRIPTION` | the `@test` description string |
| `BATS_TEST_NUMBER` | 1-based index of the current test in its file |
| `BATS_TEST_PREFIX` | optional prefix prepended to test names |
| `BATS_CWD` | directory Bats was invoked from |
| `BATS_FILE_EXTENSION` | test file extension (default `.bats`) |
| `BATS_LIB_PATH` | colon-separated search path for `load` |
| `BATS_DEBUG` | set to `1` for verbose framework debug output |

### 11.2 `bats` command reference

| Command | Effect |
|---|---|
| `bats <file|dir>...` | run tests (files or directories of `.bats` files) |
| `bats --help` / `bats --version` | usage / version |
| `bats -j N` / `--jobs N` | run files in parallel with N jobs |
| `bats -f <regex>` / `--filter <regex>` | run only tests matching the name regex |
| `bats --filter-tags <tags>` | run only tests with the given tags |
| `bats -x` / `--trace` | print each command before executing |
| `bats -t` / `--tap` | raw TAP output (no colors) |
| `bats -p` / `--pretty` | pretty human-readable output |
| `bats -c` / `--count`, `bats -l` / `--list` | count / list tests without running them |
| `bats --formatter <name>` | stdout formatter: `tap`, `tap13`, `junit`, `pretty` |
| `bats --report-formatter junit` + `--output <dir>` | write a JUnit XML report file |
| `bats --print-output-on-failure` | print captured output when tests fail |
| `bats --show-output-of-passing-tests` | also print output of passing tests |
| `bats --timing` | print elapsed time per test |
| `bats --no-parallelize` | force serial execution for this file |
| `bats --no-tempdir-cleanup` | keep temp dirs after the run (debugging) |

### 11.3 Bash version compatibility

**Requires Bash 4.2+** (project recommends 5.x); **Bash 3.x is not supported**. **macOS ships Bash 3.2** (licensing), so install a modern Bash: `brew install bash` → `/usr/local/bin/bash` (or `/opt/homebrew/bin/bash` on Apple Silicon); verify with `bats --version` / `bash --version`. The `bats/bats` Docker image bundles a recent Bash, sidestepping the issue.

### 11.4 Installation

```bash
brew install bats-core          # macOS / Linuxbrew
apt install bats                # Debian / Ubuntu (distro versions lag; prefer GitHub releases)
dnf install bats                # Fedora
apk add bats                    # Alpine
pacman -S bats                  # Arch
```

From source: `git clone https://github.com/bats-core/bats-core.git && cd bats-core && ./install.sh /usr/local` (installs to `/usr/local/bin/bats`). Docker: `docker run --rm -v "$PWD:/code" bats/bats:latest /code/test` (versioned: `bats/bats:1.11.1`). npm: `npm install -g bats-core`. Also distributed via **pre-commit hooks** (Section 13) and **mise/asdf** version managers.

## 12. Bats-core project, versions, and community

### 12.1 Project structure

The bats-core repo is a Bash project that tests itself (dogfooding): `libexec/` (main executable scripts — `bats`, `bats-format-*`, `bats-gather-tests`), `lib/` (Bash libraries — output, semver, trace), `test/` (Bats' own test suite), `docs/` (documentation, CHANGELOG, man page sources), `man/` (bats(1) man page), `install.sh`, `Dockerfile` (bats/bats image).

### 12.2 Version history highlights

| Version | Released | Highlights |
|---|---|---|
| v1.0.0 | 2019 | First stable release; performance boost, Bash 4.2+ support |
| v1.3.0 | 2021 | `BATS_FILE_EXTENSION`, TAP13 formatter, npm publishing |
| v1.5.0 | 2021 | **`BATS_TEST_TMPDIR`** per-test temp dirs |
| v1.6.0 | 2021 | **Parallel execution** (`--jobs`), `--no-parallelize` |
| v1.8.0 | 2022 | **`setup_suite`/`teardown_suite`**, **tags** + `--filter-tags`, JUnit formatter |
| v1.10.0 | 2023 | `--formatter`/`--report-formatter` polish, `BATS_LIB_PATH` |
| v1.11.0 | 2024 | v1.11.1 bugfixes (Nov 2024): load-path improvements, tempdir fixes |
| v1.12.0 | 2025 | **`bats::on_failure` hook** (runs when a test or `setup*` fails), Solaris compat |
| v1.13.0 | 2025 | Bugfix release (tempdir path shortening, `set -u` robustness, CI updates) |

**Recent features at a glance:** `setup_suite`/`teardown_suite`, parallel execution with `--jobs`, tags + `--filter-tags`, JUnit formatter, `--print-output-on-failure`, `--show-output-of-passing-tests`, `BATS_TEST_TMPDIR`, standalone formatter scripts (`bats-format-tap`, `bats-format-tap13`, `bats-format-junit`), pluggable `--formatter`, and the `bats::on_failure` hook (1.12+).

### 12.3 Community and roadmap

- **Issues & discussions:** GitHub issues for bugs/features, GitHub Discussions for questions
- **Contributing:** documented guidelines and release process (`docs/releasing.md`)
- **Ecosystem:** the bats-core org maintains bats-core, bats-assert, bats-support, bats-file, bats-extra, bats-detik; bats-mock lives with Jason Karns
- **Roadmap:** Windows support improvements, formatter/IDE integration, performance work, CI modernization
- **Adoption:** used by Docker and Kubernetes tooling, Homebrew's test suite, many CNCF projects, and countless CLI tools' CI pipelines

## 13. Integration with other tools

- **pre-commit:** add the official hook — `repo: https://github.com/bats-core/bats-core`, `rev: v1.11.1`, `hooks: [{id: bats, args: [test/]}]` — runs your suite on every commit.
- **GitHub Actions:** the official action is `bats/bats-action` (`uses: bats/bats-action@v2` with `bats-version: 1.11.1` and `args: test/`), or run the Docker image directly: `docker run --rm -v "$PWD:/code" bats/bats:v1.11.1 /code/test`. Combine with `actions/checkout@v4`.
- **GitLab CI:** `image: bats/bats:v1.11.1`, script `bats /builds/$CI_PROJECT_PATH/test/`.
- **Jenkins:** use the **TAP plugin** — `sh 'bats --tap test/ > bats.tap'` then `tap testResults: 'bats.tap'`; JUnit XML also works.
- **Makefile:** add a `test:` target running `bats test/`, a `test-unit:` target with `bats --filter-tags unit test/`, and `coverage:` with `kcov --include-path=src coverage bats test/` — then `make test` in CI and locally.
- **Codecov:** `kcov --include-path=src coverage bats test/` then `codecov -f coverage/coverage.json` for PR coverage gates.
- **Editors:** VS Code `vscode-bats` extension (syntax highlighting + test running for `.bats` files); `vim-bats` plugins for vim/neovim; JetBrains community plugins for Bash and TAP. Most editors pair Bats with shellcheck and a task runner.
- **CI systems at a glance:** GitHub Actions (bats/bats-action or Docker image), GitLab CI (bats/bats image), Jenkins (TAP plugin), CircleCI (Bash executor + JUnit XML), TeamCity (TAP/JUnit service messages), Codecov/Coveralls (kcov reports).
- **Companion tools:** **shellcheck** (static analysis alongside Bats — run both in CI), **shfmt** (formatting), the ecosystem libraries (bats-assert, bats-file, bats-mock, bats-extra, bats-detik), **GNU parallel** (runtime backend for `--jobs`), **kcov**/bashcov (coverage).

## 14. Quick reference / cheat sheet

```bash
# Run tests
bats test/                                   # whole suite
bats -j 4 test/                              # parallel
bats --filter "deploy" test/                 # by name regex
bats --filter-tags "unit" test/              # by tag
bats -x test/deploy.bats                     # trace mode

# Inside a test file
run ./script.sh --flag                       # capture $status, $output, $lines
assert_success                               # exit code 0
assert_failure                               # non-zero exit
assert_output "exact"                        # full output match
assert_output --partial "substring"          # partial match
assert_line --index 0 "first line"           # specific line
refute_output --partial "forbidden"          # negative assertion
assert_file_exist "$BATS_TEST_TMPDIR/x"      # file assertion
skip "not implemented"                       # skip a test
```

## 15. Conclusion

Bats is the de facto standard for testing Bash because it matches the domain: written in Bash, tests Bash, and speaks TAP — the lingua franca of test automation. For architects and engineers responsible for CI/CD pipelines, deployment scripts, Docker entrypoints, and infrastructure automation, Bats turns the most untestable layer of the stack into something with regression tests, coverage, and CI visibility.

**Start small:** one `.bats` file for your most critical script, `run` + `[ ]` assertions, run it in CI. **Grow deliberately:** add bats-assert, bats-file, bats-mock; parallelize with `--jobs`; add kcov coverage; split unit vs integration by tags. In an afternoon you can go from "shell scripts are untestable" to "every deploy script has a test suite."
