# SBOM Generation for C, Pro*C (Embedded SQL), and COBOL Codebases

> **Author:** Jack Liu Shurui  
> **Purpose:** Research reference for SBOM generation targeting C, Pro*C, and COBOL — enterprise/mainframe languages with limited modern SCA and package manager tooling.  
> **Repo:** https://github.com/jackliusr/research

---

## 1. SBOM Background & Relevance

### 1.1 What is an SBOM?

A **Software Bill of Materials (SBOM)** is a formal, machine-readable inventory of all components, libraries, and dependencies that comprise a software artifact. Analogous to a food ingredient label, it lists every "ingredient" — direct and transitive dependencies, versions, licenses, and supplier metadata.

### 1.2 Why SBOMs Matter

Supply-chain attacks (SolarWinds 2020, Log4j 2021) showed that organizations cannot secure what they cannot inventory.

- **US EO 14028 (2021):** Mandates SBOMs for software vendors selling to the US federal government. Reinforced by OMB M-22-18 and NIST SP 800-218.
- **EU Cyber Resilience Act:** Requires documented software supply chain dependencies.
- **CISA SBOM Guidance:** Identifies SBOMs as critical infrastructure for supply chain risk.
- **Industry:** PCI DSS v4.0, FDA premarket cybersecurity guidance, and finance/insurance regulations increasingly mandate SBOM delivery.

### 1.3 SBOM Formats & Standards

| Format | Latest | ISO | Strengths |
|--------|--------|-----|-----------|
| **SPDX** | 3.0.1 (2024) | ISO/IEC 5962:2021 | Deep licensing detail, AI/ML profiles, native VEX |
| **CycloneDX** | 1.6 (2024), 1.7 (2025) | OWASP, IEC submission | Richer dep graph, crypto inventory, attestation, services |

JSON is the most common serialization for both. JSON/XML variants exist. SWID (ISO/IEC 19770-2) covers OS-level software identification.

### 1.4 Key Challenges for C/Pro*C/COBOL

Unlike ecosystems with centralized package managers (npm, pip, Maven), these languages present unique SBOM challenges:

1. **No universal package manager.** Conan/vcpkg for C/C++, but many projects use none. Pro*C and COBOL have none.
2. **Legacy build systems.** Make, CMake, Autotools, JCL — no standard lockfile.
3. **Manual dependency tracking.** `#include` and `COPY` without version pinning. Transitive deps are implicit.
4. **Static linking.** C/C++/COBOL binaries frequently statically link libraries — runtime detection is the only option.
5. **Proprietary compilers.** IBM COBOL for z/OS, Oracle Pro*C precompiler — closed-source, no SBOM awareness.
6. **Mainframe isolation.** z/OS uses datasets (PDS/PDSE), not POSIX filesystems.
7. **Mixed-language deployments.** Single executables may combine C, Pro*C, COBOL, and Assembler.

### 1.5 Why SBOM Matters for These Languages

| Sector | Reason |
|--------|--------|
| **Finance** | Basel III/IV, SOX — regulatory exams require software inventory |
| **Healthcare** | HIPAA, FDA premarket cybersecurity (medical device SBOMs) |
| **Government** | EO 14028, FedRAMP, NIST SP 800-218 |
| **Insurance / Legacy** | Risk management for 30+ year-old COBOL systems processing core business logic |
| **Mainframe Modernization** | Migration to cloud requires complete dep mapping to avoid breaking critical paths |

---

## 2. SBOM Generation for C/C++

### 2.1 Conan (JFrog) — Package Manager SBOM

**Active, well-maintained.** Best option for new C/C++ projects.

`conan sbom` generates CycloneDX JSON from a Conan lockfile. Includes package names, versions, PURLs, licenses, and dependency graph.

```bash
conan install . --lockfile-out=conan.lock
conan sbom . --format=cyclonedx-json --output=sbom.cyclonedx.json
```

**Limitations:** Requires adopting Conan. Doesn't help with vendored headers or system libraries.

### 2.2 vcpkg (Microsoft) — Package Manager SBOM

Active. Community-driven CycloneDX export support added in 2024.01.12 release. PURL support in progress (GitHub issue #30461). Emerging but not yet mature.

### 2.3 Syft (Anchore) — Filesystem/Container Scanner

**Excellent, active, open source (Apache 2.0).** Scans directories, container images, and archives for package manifests. Detects Conan lockfiles, vcpkg manifests, dpkg/rpm packages, shared objects (`.so` inspection), and some vendored headers. Outputs CycloneDX and SPDX.

```bash
syft dir:./my-project -o cyclonedx-json=sbom.json
syft my-app:latest -o cyclonedx-json=sbom.json
```

**Limitation:** Cannot discover statically linked libraries without symbol tables.

### 2.4 Trivy (Aqua Security) — Scanner with CVE Matching

**Excellent, active, open source.** Similar to Syft with added vulnerability matching. Modes: `trivy fs`, `trivy image`, binary scanning. Detects OpenSSL, libcurl, zlib, etc. via binary fingerprinting.

```bash
trivy fs ./my-project --format cyclonedx --output sbom.json
trivy image my-app:latest --format cyclonedx --output sbom.json
```

### 2.5 scancode-toolkit (nexB) — Source License Scanner

**Active, open source.** Detects licenses, copyrights, package manifests in source code. Can identify bundled third-party C code and license text. Outputs SPDX and CycloneDX.

```bash
scancode --spdx-tv output.spdx.tv --license --copyright --package ./my-c-project
```

**Limitation:** Slower than Syft/Trivy on large codebases. Not for containers/binaries.

### 2.6 Fossology + SW360

- **Fossology:** Open-source license scanner with C-specific scanners (nomos, monk). Generates SPDX.
- **SW360:** Component catalog that ingests Fossology results. Tracks components, licenses, vulnerabilities.

Heavyweight setup (needs PostgreSQL/MySQL). Best for orgs needing a centralized compliance registry.

### 2.7 OSS Review Toolkit (ORT)

**Active.** Experimental Conan analyzer reads `conanfile.py` / `conan.lock`. Outputs CycloneDX and SPDX. Only covers Conan-declared deps — not Make, CMake, or Autotools.

```bash
ort analyze -i /path/to/project -o /path/to/output -m Conan
```

### 2.8 Other Tools

- **build2:** C/C++ build toolchain with its own package manager. Niche adoption. Limited SBOM support via build reports.
- **Gradle CycloneDX plugin:** Limited C support (mainly Java/Android).
- **CMake:** No native SBOM. Workaround: `--graphviz` output converted manually.
- **cppcheck:** Static analysis only. Not SBOM-related but can inform manual inventory.
- **VEX/CSAF:** Complementary formats for vulnerability exploitability communication. Integrated in CycloneDX 1.5+/SPDX 3.0.

---

## 3. SBOM Generation for Pro*C (Embedded SQL)

### 3.1 Background

Pro*C is Oracle's embedded SQL precompiler for C. Source (`.pc`) mixes C code with `EXEC SQL` statements. The `proc` precompiler translates to C before compilation.

**Zero dedicated SBOM tooling exists for Pro*C.** Approach must be polyglot — treat as C code with an Oracle dependency overlay.

### 3.2 Key Dependencies to Capture

| Dependency | Detection Method | Criticality |
|-----------|-----------------|-------------|
| Oracle Pro*C Precompiler (`proc`) | `proc -V` in build scripts | High — SQL dialect |
| Oracle OCI libs (`libclntsh.so`) | `$ORACLE_HOME`; binary scan | High — runtime |
| Oracle DB version | `EXEC SQL` feature set | High — compatibility |
| C libraries | Standard include scanning | Medium |
| SQL includes (`EXEC SQL INCLUDE`) | Parse Pro*C source | Medium |
| Third-party C libs | Syft/Trivy on binary | Medium |

### 3.3 Approach 1: Syft/Trivy on Compiled Binaries (Recommended First Step)

Captures linked Oracle OCI libraries and C library deps. Misses precompiler version, SQL-specific deps, and `EXEC SQL INCLUDE` files.

```bash
proc iname=my_program.pc oname=my_program.c
gcc -o my_program my_program.c -lclntsh
syft file:./my_program -o cyclonedx-json=procyc_sbom.json
```

### 3.4 Approach 2: Custom Source Parser

Write a Python script to extract from Pro*C source:

```python
import re
def parse_proc_deps(text):
    return {
        "c_includes": re.findall(r'#include\s+[<"]([^>"]+)[>"]', text),
        "sql_includes": re.findall(r'EXEC\s+SQL\s+INCLUDE\s+(\w+)', text, re.I),
        "oracle_dep": "sqlca.h" in text  # marker for Oracle dependency
    }
```

Output as CycloneDX JSON using `cyclonedx-python` library.

### 3.5 Approach 3: Oracle Enterprise Inventory

Oracle Enterprise Manager / `opatch lsinventory` reports installed client versions, DB versions, OCI library versions. Combine with source parsing for completeness.

### 3.6 Approach 4: Manual Documentation

For mainframe-connected Pro*C on AIX/zLinux, document Oracle client version + C libraries in a spreadsheet, export to CycloneDX, supplement with Syft/Trivy binary scans.

### 3.7 Pro*C Dependency Tree

```
Program (.pc)
├── Oracle Pro*C Precompiler (proc)  — build-time
├── Oracle OCI Libs (libclntsh.so)   — runtime
├── Oracle Database                   — runtime connection
├── C Standard Library (glibc)        — runtime
├── Third-Party C Libs               — link-time
└── SQL Include Files                 — compile-time
```

---

## 4. SBOM Generation for COBOL

### 4.1 Background

COBOL powers most financial transactions, insurance processing, and government systems. Most runs on IBM z/OS, though GnuCOBOL provides a Linux path. Codebases are often 30+ years old.

**No mainstream SBOM tooling supports COBOL directly.**

### 4.2 IBM z/OS Tools

#### 4.2.1 IBM Dependency Based Build (DBB)

**Best option for z/OS COBOL.** Tracks build dependencies (COPY members, CALL targets) for COBOL, PL/I, Assembler. Integrates with Git/Jenkins/UrbanCode. **Does not natively export SPDX/CycloneDX** — custom Python/REXX scripts needed to convert DBB metadata to standard SBOM formats.

#### 4.2.2 IBM IDz (Developer for z/Systems)

Eclipse IDE for COBOL. Shows COPY/CALL dep trees visually. **No SBOM export.**

#### 4.2.3 IBM Wazi for VS Code

Modern VS Code extension. Dependency navigation. **No SBOM export**, but VS Code extension model enables custom plugins.

#### 4.2.4 IBM z/OSMF

System inventory of installed software (compiler versions, CICS/DB2). Useful for infrastructure-level SBOM but not application-level COBOL deps.

### 4.3 Open-Source / Cross-Compiler Approaches

#### 4.3.1 GnuCOBOL (OpenCOBOL)

Translates COBOL to C, then compiles with gcc. On Linux, scan the ELF binary with Syft/Trivy for linked C libraries (glibc, GnuCOBOL runtime). Does **not** capture COBOL-level COPY/CALL deps.

```bash
cobc -x my_program.cbl -o my_program
syft file:./my_program -o cyclonedx-json=cobol_sbom.json
```

#### 4.3.2 cobol-to-c Conversion

GnuCOBOL's `-C` flag outputs intermediate C. Once in C form, apply C SBOM tooling (limited value — generated C, not authored).

#### 4.3.3 z/OS Open Source Tools

IBM ported Python, curl, zlib, OpenSSL to z/OS. Can run custom SBOM pipelines on z/OS itself.

### 4.4 Custom COBOL Parsing

Most thorough (highest effort) approach.

**COPY statements** (COBOL's `#include`):
```python
copies = re.findall(r'COPY\s+(\w+)', source_text, re.I)
```

**CALL statements** (cross-program deps):
```python
calls = re.findall(r"CALL\s+['\"](\w+)['\"]", source_text, re.I)
```

**JCL dependencies:** Parse dataset names, `EXEC PGM=`, STEPLIB concatenations.

**Parsing tooling:**
- **ProLeap COBOL Parser** (`uwol/proleap-cobol-parser`): ANTLR4-based full COBOL grammar → AST/ASG. Java. Most complete open-source COBOL parser.
- **cobol85parser** (`ulfloe/cobol85parser`): ANTLR4 grammar for COBOL-85.

### 4.5 Enterprise Tooling

| Tool | Type | COBOL SBOM |
|------|------|------------|
| **CAST SBOM Manager** | Commercial | ✅ Visualizes COBOL deps, generates SBOM reports. Expensive. |
| **SonarQube + COBOL plugin** | Free/Commercial | ⚠️ Code quality only, no SBOM export, but informs inventory |
| **Micro Focus Visual COBOL** | Commercial | ⚠️ Some dependency tracking in IDE |
| **BMC Compuware Topaz** | Commercial | ⚠️ Dependency analysis via API; no SBOM export |
| **Fortify/CyberRes** | Commercial | ❌ Limited COBOL SCA coverage |
| **VirtualZ** | Commercial | ❌ Modernization platform; needs SBOM but doesn't generate one |

### 4.6 COBOL SBOM Pipeline

```
COBOL Source (PDS)
├── COPY members     ──┐
├── CALL subprograms ──┤── Custom Parser (ProLeap + regex)
├── JCL deps         ──┘       │
└── Compiled Binary  ── Syft/Trivy binary scan
                               ↓
                       CycloneDX/SPDX SBOM
```

---

## 5. Cross-Cutting Approaches (Works for All Three)

### 5.1 Manual + Scripted Hybrid (Recommended)

1. **Syft/Trivy on binaries/containers** — captures runtime C/C++/Oracle OCI deps (all three languages at binary level).
2. **Custom language parsers** — dep constructs per language (`#include`, `EXEC SQL INCLUDE`, `COPY`, `CALL`, JCL).
3. **Merge** — CycloneDX CLI (`cyclonedx merge`) or `cyclonedx-python` library.
4. **Validate** — `cyclonedx validate` or SPDX tools.

### 5.2 Container-Based Scanning

Package all components into one image, scan once with Syft/Trivy. Captures system and compiled deps. Does **not** capture COBOL COPY/CALL or Pro*C SQL deps — add via custom parsing.

### 5.3 SBOM Merge & Management Tools

| Tool | Purpose |
|------|---------|
| **CycloneDX CLI** | Merge, validate, convert CycloneDX docs |
| **SPDX Tools** (Java) | Validate, compare, convert SPDX |
| **Dependency-Track** (OWASP) | Platform: ingest CycloneDX SBOMs, correlate CVEs, policy enforcement |
| **NTIA SBOM Validation** | Validate against NTIA minimum elements |

### 5.4 Enterprise SCA Platforms — Summary

| Platform | C/C++ | Pro*C | COBOL | Notes |
|----------|-------|-------|-------|-------|
| **Synopsys Black Duck** | ✅ Good | ⚠️ Binary only | ⚠️ Binary only | High cost |
| **Snyk** | ✅ Good (C) | ❌ Limited | ❌ Limited | Container + manifest scanning |
| **Sonatype Nexus Lifecycle** | ✅ Good (Conan) | ❌ Limited | ❌ Limited | CI/CD pipeline focused |
| **Mend (WhiteSource)** | ✅ Good | ❌ Limited | ❌ Limited | C/C++ in build pipelines |
| **FOSSA** | ⚠️ Partial | ❌ Limited | ❌ Limited | License compliance |
| **Contrast Security** | ⚠️ Limited | ❌ N/A | ❌ N/A | IAST, not applicable to compiled binaries |

**Bottom line:** True COBOL and Pro*C source-level SCA is virtually nonexistent in commercial tools. Binary scanning via Black Duck works for identifying known open-source components in compiled output.

---

## 6. Comparison Table

| Tool / Method | C/C++ | Pro*C | COBOL | Format | Approach | Effort | Cost |
|----------------|-------|-------|-------|--------|----------|--------|------|
| **Syft** (binary/container) | ✅ Good | ✅ Good (bin) | ✅ Good (bin) | CycloneDX, SPDX | Post-build binary | Low | Free |
| **Trivy** (binary/container) | ✅ Good | ✅ Good (bin) | ✅ Good (bin) | CycloneDX, SPDX | Post-build binary | Low | Free |
| **Conan SBOM** | ✅ Good | ❌ No | ❌ No | CycloneDX | Build integration | Med | Free |
| **vcpkg SBOM** | ⚠️ Emerging | ❌ No | ❌ No | CycloneDX | Build integration | Med | Free |
| **Fossology / SW360** | ✅ Good | ⚠️ Partial | ⚠️ Partial | SPDX | Source scanning | High | Free |
| **scancode-toolkit** | ✅ Good | ⚠️ Partial | ⚠️ Partial | SPDX, CycloneDX | Source scanning | High | Free |
| **ORT** | ⚠️ Limited | ❌ No | ❌ No | SPDX, CycloneDX | Build integration | High | Free |
| **IBM DBB** | ❌ No | ❌ No | ✅ Good | Custom | z/OS build tracking | Med | Commercial |
| **IBM IDz / Wazi** | ❌ No | ❌ No | ⚠️ Visual only | None | IDE dep view | N/A | Commercial |
| **IBM z/OSMF** | ❌ No | ❌ No | ⚠️ Infra only | None | System inventory | Med | Bundled |
| **CAST SBOM** | ⚠️ Partial | ⚠️ Partial | ✅ Good | Proprietary+export | Platform scanning | High | Commercial |
| **Black Duck** | ✅ Good | ⚠️ Binary | ⚠️ Binary | Proprietary+SPDX | Multi-mode | High | $$$ |
| **Snyk** | ✅ Good (C) | ❌ Limited | ❌ Limited | CycloneDX, SPDX | CI scanning | Med | $$ |
| **Custom Parser** | ✅ Yes | ✅ Yes | ✅ Yes | Any (CycloneDX) | Custom parsing | High | Free |
| **Container Pack + Scan** | ✅ Good | ✅ Good | ✅ Good | CycloneDX | Post-build | Med | Free |
| **Manual Documentation** | ✅ Yes | ✅ Yes | ✅ Yes | Any | Manual | High | Variable |

---

## 7. Recommended Approach by Language

### 7.1 For C/C++

**New projects:** Adopt Conan → `conan sbom` (primary) + Syft on container (supplement).  
**Legacy projects:** Syft/Trivy on compiled output + scancode-toolkit for license clarity.  
**Best practice:** Conan for dep management → CycloneDX → Dependency-Track.

### 7.2 For Pro*C

1. **Syft/Trivy on compiled binary** — captures Oracle OCI and C libs.
2. **Custom parser** — `#include`, `EXEC SQL INCLUDE`, Oracle version hints (sqlca.h).
3. **Manual** — Oracle client version (from `opatch lsinventory`), Oracle DB version.
4. **Merge** — combine all three into CycloneDX via CLI.

Key insight: Most critical Pro*C deps are Oracle ecosystem components (precompiler, OCI, DB version) — well-documented in Oracle inventory tools; the gap is converting that data to open-standard SBOM format.

### 7.3 For COBOL

**On z/OS:** IBM DBB (metadata) → custom Python to convert to CycloneDX + z/OSMF for system version inventory + custom parser for COPY/CALL/JCL.  
**On Linux (GnuCOBOL):** Syft/Trivy on binary + custom parser for COPY/CALL.  
**Enterprise:** DBB + CAST + custom parser → merged CycloneDX → Dependency-Track.

### 7.4 Unified Pipeline

```
Step 1: syft dir:./artifacts -o cyclonedx-json=binary-sbom.json
Step 2: parse_c_includes.py  → c_deps.json
        parse_proc_deps.py   → proc_deps.json
        parse_cobol_deps.py  → cobol_deps.json
Step 3: Manual overrides: Oracle DB version, mainframe subsystem versions
Step 4: cyclonedx merge --input-files binary.json c.json \
                          proc.json cobol.json \
                        --output-file unified-sbom.cyclonedx.json
Step 5: Upload to Dependency-Track for continuous monitoring
```

---

## 8. Practical Implementation Notes

### 8.1 Start with Binary/Container Scanning

Deploy Syft or Trivy against build artifacts. Instantly get OpenSSL, libcurl, zlib, Oracle OCI lib versions — **highest-risk components first**.

### 8.2 "Good Enough" SBOM > None

NTIA/CISA/FDA recognize that perfect coverage for legacy languages may be infeasible. Requirements: transparency (document what you know and don't), auditability (show method), completeness over perfection (80% with clear method > 0%).

### 8.3 Focus on High-Value Dependencies

| Priority | What | Why |
|----------|------|-----|
| P0 | OpenSSL, libcurl, zlib, libxml2 | Most CVEs |
| P0 | Oracle client version (Pro*C) | Oracle security patches |
| P1 | glibc / system C library | Broad CVE impact |
| P1 | Mainframe compiler versions | Patch management |
| P2 | COBOL COPY/CALL graph | Application dep mapping |
| P2 | JCL dataset deps | Runtime environment |
| P3 | SQL includes / DB schemas | DB dependency mapping |

### 8.4 CI/CD Integration

GitHub Actions skeleton:
```yaml
steps:
  - name: Parse C/Pro*C/COBOL deps
    run: python scripts/parse_all_deps.py > source-deps.json
  - name: Generate SBOM (Syft)
    uses: anchore/sbom-action@v0
    with: { path: ./build-output/, format: cyclonedx-json }
  - name: Merge SBOMs
    run: cyclonedx merge --input-files source-deps.json binary-sbom.json
  - name: Upload to Dependency-Track
    run: curl -X POST "$DTRACK_URL/api/v1/bom" -H "X-Api-Key: $DTRACK_API_KEY" ...
```

### 8.5 Pitfalls to Avoid

1. **Relying only on Conan/vcpkg** — empty SBOM for Makefile-based projects.
2. **Forgetting system libraries** — every C program links glibc; document it.
3. **Binary scanning alone** — misses COPY/CALL/JCL deps in COBOL, SQL deps in Pro*C.
4. **Ignoring mainframe subsystems** — CICS, DB2, IMS are undetectable from binaries.
5. **One-time exercise** — SBOMs must regenerate per build. Integrate into CI/CD immediately.
6. **Not validating** — `cyclonedx validate` catches structural issues. Invalid SBOM is worse than none.

### 8.6 Regulatory Note for Regulated Industries

- **Finance (MAS, Basel, SOX):** Documented manual inventory is acceptable if auditable, repeatable, and version-controlled.
- **Healthcare (FDA):** FDA 2023 guidance accepts "best effort" for legacy COBOL in medical software.
- **Government (EO 14028):** Phased compliance allowed — start with known open-source deps, expand coverage over time.

---

## 9. Data Sources & Further Reading

### Standards & Regulatory

| Resource | URL |
|----------|-----|
| CycloneDX Spec 1.6 | https://cyclonedx.org/specification/overview/ |
| SPDX Spec 3.0 | https://spdx.dev/specifications/ |
| US EO 14028 | https://www.nist.gov/itl/executive-order-14028 |
| NTIA SBOM Minimum Elements | https://www.ntia.gov/sbom |
| CISA SBOM Guidance | https://www.cisa.gov/sbom |
| NIST SP 800-218 | https://csrc.nist.gov/publications/detail/sp/800-218/final |

### Tool Documentation

| Tool | URL |
|------|-----|
| Conan SBOM | https://docs.conan.io/2/reference/commands/sbom.html |
| Syft | https://github.com/anchore/syft |
| Trivy | https://github.com/aquasecurity/trivy |
| scancode-toolkit | https://github.com/aboutcode-org/scancode-toolkit |
| Fossology | https://www.fossology.org/ |
| SW360 | https://sw360.github.io/ |
| ORT | https://oss-review-toolkit.github.io/ort/ |
| IBM DBB | https://www.ibm.com/docs/en/dbb/latest |
| IBM Wazi | https://www.ibm.com/products/wazi-developer |
| GnuCOBOL | https://gnucobol.sourceforge.io/ |
| ProLeap COBOL Parser | https://github.com/uwol/proleap-cobol-parser |
| Dependency-Track | https://dependencytrack.org/ |
| CAST SBOM Manager | https://www.castsoftware.com/products/sbom-manager |

### Industry References

| Resource | URL |
|----------|-----|
| C/C++ SBOM Guide (sbomgenerator.com) | https://sbomgenerator.com/guides/cpp |
| C/C++ SBOM Guide (sbomify.com) | https://sbomify.com/guides/cpp/ |
| "The C Conundrum" (sbomify) | https://sbomify.com/2024/11/18/c-conundrum/ |
| SBOM Tools Comparison (sbomify) | https://sbomify.com/2026/01/26/sbom-generation-tools-comparison/ |
| Best SBOM Tools for C/C++ (RunSafe) | https://runsafesecurity.com/blog/best-sbom-tools-cpp/ |
| OpenSSF SBOM Tools | https://openssf.org/technical-initiatives/sbom-tools/ |
| z/OS DevOps Guide (IBM) | https://ibm.github.io/z-devops-acceleration-program/docs/build/ |

> **Version:** 1.0 | **Updated:** July 2026 | **Author:** Jack Liu Shurui | **Repo:** https://github.com/jackliusr/research
