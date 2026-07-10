# Vulnerability Scanning for C, Pro*C (Embedded SQL), and COBOL Codebases

> **Author:** Jack Liu Shurui  
> **Purpose:** Research reference for vulnerability scanning (SAST, SCA, DAST, binary, and mainframe-specific) targeting C, Pro*C, and COBOL — enterprise/mainframe languages with limited modern tooling.  
> **Repo:** https://github.com/jackliusr/research  
> **Related:** [SBOM Generation for C/Pro*C/COBOL](./sbom_c_proc_cobol.md) — covers bill-of-materials generation; this document covers scanning/audit tooling.  
> **Date:** July 2026

---

## 1. The Challenge Overview

C, Pro*C (embedded SQL in C for Oracle), and COBOL power core banking, insurance, government tax, air traffic control, and healthcare systems — often on codebases 20–50 years old. Modern vulnerability scanning tooling (SAST, SCA, DAST) overwhelmingly targets web-centric languages (JS, Python, Java, Go) and treats these languages as second-class citizens.

### Key Challenges

| Challenge | C | Pro*C | COBOL |
|-----------|---|-------|-------|
| Proprietary compilers | GCC/Clang (standard) | Oracle Pro*C precompiler required | IBM z/OS Enterprise COBOL, Micro Focus |
| Embedded SQL | N/A | `EXEC SQL` — invisible to C SAST | DB2 embedded SQL in COBOL |
| Static linking | Common — no package-manager SBOM | Oracle OCI libs statically linked | LE/370 runtime, CICS stubs |
| No centralised package manager | Conan/vcpkg modern; legacy uses manual vendoring | Oracle libs from install media | z/OS datasets, no registry |
| Mainframe isolation | N/A | May run on Linux/HP-UX | z/OS isolated from CI/CD |
| Code age | 10–30 years | 15–35 years | 20–50+ years |
| Regulatory pressure | PCI-DSS, HIPAA, DORA | SOX, Basel III, PCI-DSS | SOX, Basel III, PCI-DSS, DORA |

### Why Scanning Matters

- **Finance:** PCI-DSS (card data), SOX (financial controls), Basel III (capital adequacy), DORA (EU Digital Operational Resilience Act, effective Jan 2025)
- **Healthcare:** HIPAA (protected health information)
- **Government:** US Executive Order 14028, NIST SP 800-218 (SSDF)
- **General:** Auditors now expect documented vulnerability management programs even for COBOL and Pro*C

### Categories of Scanning

- **SAST** (Static Application Security Testing): Source code analysis without execution — best for injection, buffer overflow, memory safety issues early in the SDLC.
- **SCA** (Software Composition Analysis): Known vulnerabilities in open-source/third-party libraries; requires an SBOM or manifest.
- **DAST** (Dynamic Application Security Testing): Sends crafted inputs at a running target; language-agnostic.
- **Binary/Container scanning:** Post-build analysis of compiled artifacts for vulnerable libraries (glibc, OpenSSL, zlib). Works regardless of source language.
- **Mainframe-specific:** z/OS security assessment covering RACF/ACF2, JCL privilege analysis, dataset access controls, CICS transaction security.

---

## 2. Vulnerability Scanning for C/C++

C and C++ have the richest tooling of the three languages. Multiple commercial and open-source SAST tools exist, SCA tools can scan binaries, and runtime memory debugging is mature.

### 2.1 SAST for C

| Tool | Type | Capabilities | Accuracy | Cost | Best For |
|------|------|-------------|----------|------|----------|
| **CodeQL** (GitHub) | SAST | Custom query language; C/C++ native. Buffer overflow, use-after-free, injection, memory leaks — semantic analysis (AST + data-flow + control-flow) | Very high — industry-leading | Free (public), $49/committer/mo (private) | Organisations on GitHub; best-in-class C SAST |
| **SonarQube + C plugin** | SAST + Quality | CWE-mapped rules, OWASP Top 10, MISRA. C plugin is commercial (Developer Edition+) | High for common patterns; higher FP than CodeQL | Free (CE, no C plugin); $150/yr Dev Ed | Unified quality + security platform |
| **Clang Static Analyzer** | SAST (compiler) | Path-sensitive inter-procedural. Null deref, leaks, use-after-free, buffer overflow (limited) | Very good for its scope | Free (LLVM/Clang) | CI gating on memory safety; complements commercial tools |
| **Coverity** (Synopsys) | SAST | Deep inter-procedural path analysis. Complex use-after-free, race conditions, tainted flows, resource leaks | Industry-leading; lowest FP | $$$$ ($25–100K+/yr) | Safety-critical (ISO 26262, DO-178C); high-security environments |
| **PVS-Studio** | SAST | OWASP Top 10, MISRA, 64-bit portability, low FP | Very good | $$ ($1.5–5K/yr/dev) | Teams wanting pragmatic commercial tool without Coverity price |
| **Cppcheck** | SAST | Buffer overflow, leaks, use-after-free, STL misuse. Intra-procedural only | Moderate; misses complex cases | Free (GPL) | Low-cost CI gating for legacy code |
| **Infer** (Meta) | SAST | Separation logic-based. Null safety, resource leaks, memory safety | Good for its scope | Free (MIT) | Teams on a budget wanting industrial-grade memory analysis |
| **Flawfinder** | SAST (lexical) | Greps for dangerous functions (`strcpy`, `sprintf`, `system`). No CFG/DFA | Low — purely lexical | Free (GPL) | Quick reconnaissance pass only |
| **RATS** | SAST (lexical) | Similar to Flawfinder; C/C++, Python, Perl, PHP | Low — unmaintained | Free | Historical; prefer Flawfinder if lexical pass needed |

### 2.2 SCA for C

SCA for C is harder than for managed languages because: no standard manifest (`package.json`/`pom.xml`); static linking embeds library code; legacy code vendored third-party code manually; many CVEs affect system libraries (glibc, OpenSSL).

| Tool | Type | Capabilities | Cost | Best For |
|------|------|-------------|------|----------|
| **Black Duck** (Synopsys) | SCA | Scans source + binaries (signature matching). 5M+ component KB. CVEs, licences | $$$$ | Enterprise open-source compliance + vuln management |
| **Snyk** | SCA | Container/binary scanning + Conan/vcpkg support | Free tier; $25–100/dev/mo | Teams already using Snyk for other languages |
| **Trivy** (Aqua) | SCA/Container | Scans filesystems, containers, repos, SBOMs. OS packages + Conan/vcpkg. CycloneDX/SPDX output. ELF binary scanning | Free (Apache 2.0) | **Single easiest win** — works on compiled output regardless of language |
| **Grype + Syft** (Anchore) | SCA/Container | Grype scans for vulns; Syft generates SBOMs (CycloneDX/SPDX). Covers OS + language packages | Free (Apache 2.0) | Dual coverage with Trivy; feed SBOMs into Dependency-Track |
| **Mend** (WhiteSource) | SCA | C/C++ via binary scanning + Conan/vcpkg. Licence + vuln detection | $$$ | Licence compliance + vuln detection |
| **FOSSA** | SCA | Limited C/C++ — Conan support | Free tier; $150+/mo | Teams already using FOSSA |
| **Dependency-Track** (OWASP) | SBOM Platform | Ingests CycloneDX/SPDX SBOMs; polls NVD, GHSA, OSV; policy-based alerting | Free (Apache 2.0) | Continuous vuln monitoring after initial SBOM pass |

### 2.3 DAST / Runtime for C

| Tool | Type | What It Catches | Integration |
|------|------|----------------|-------------|
| **Valgrind** (Memcheck) | Runtime | Memory leaks, use-after-free, invalid reads/writes (10–50x slowdown) | Wrap test runner: `valgrind --tool=memcheck ./program` |
| **ASan / LSan / UBSan** | Compiler-instrumented | Use-after-free, buffer overflow, memory leaks, undefined behaviour (~2x slowdown) | Compile with `-fsanitize=address -fno-omit-frame-pointer` |
| **libFuzzer** | Fuzzing | Coverage-guided, in-process. Best for specific code paths | Clang-only; write fuzz targets |
| **AFL** (American Fuzzy Lop) | Fuzzing | Coverage-guided. Best for file-parsing code | Works with GCC/Clang; QEMU mode for binaries |
| **Honggfuzz** | Fuzzing | Hardware feedback (Intel BTS/PT). Performance-sensitive | Persistent mode; CLI integration |

---

## 3. Vulnerability Scanning for Pro*C

Pro*C is embedded SQL in C — source code with `EXEC SQL` and `EXEC ORACLE` statements. The Oracle precompiler (`proc`) translates these into pure C with OCI library calls.

**No dedicated Pro*C vulnerability scanner exists.** SAST tools understand *either* C or SQL — not both in the same file. Pro*C falls exactly in the gap.

### What SAST Tools See vs. What Matters

| Element | Visible to C SAST? | Visible to SQL SAST? | Security Concern |
|---------|-------------------|---------------------|------------------|
| C code | ✅ Yes | ❌ No | Buffer overflows, memory leaks |
| `EXEC SQL SELECT ...` | ❌ No (preprocessor directive) | ❌ No (not pure SQL) | SQL injection via concatenation |
| `:host_variable` bind | ✅ Yes (C variable) | ❌ No | Safe if bound; injection if concatenated |
| OCI calls (post-preprocessing) | ✅ Yes | ❌ No | OCI API misuse |
| `EXEC ORACLE OPTION` | ❌ No | ❌ No | Precompiler configuration issues |
| C string literals inside EXEC SQL | ❌ No | ❌ No | Hardcoded credentials |

### SAST Approaches

**Preprocess → Run C SAST (primary approach):**
1. Run Oracle `proc` precompiler on `.pc` files → `.c` files
2. Run C SAST (CodeQL, SonarQube, Coverity, Clang SA) on generated `.c`
3. Catches: C-level bugs, memory safety, OCI API misuse
4. Misses: SQL injection within `EXEC SQL` blocks — precompiler translates SQL to OCI function calls, not analysable SQL text

**CodeQL on original .pc source:** Partially parses C parts; SQL injection queries will not fire on `EXEC SQL` blocks.

**SonarQube + C plugin:** Same limitation; may flag embedded SQL as syntax errors.

### SQL-Specific Scanning

| Tool | Type | Pro*C Support | Cost |
|------|------|---------------|------|
| **Checkmarx** | SAST | Some embedded SQL support; custom query packs for Pro*C | $$$ |
| **Fortify** (Micro Focus/OpenText) | SAST | Some Pro*C pattern recognition; taint flows through EXEC SQL (moderate accuracy) | $$$ |
| **HCL AppScan** | SAST | Some Oracle Pro*C support | $$$ |
| **Custom regex scanner** | Pattern-based | **Recommended** — scans for string concat in EXEC SQL, `EXECUTE IMMEDIATE`, hardcoded credentials | Dev effort |

### Oracle Database Security Assessment

- **Oracle Audit Vault & Database Firewall:** Monitors SQL traffic, detects injection attempts
- **Oracle Data Safe:** DB security assessment, user risk analysis, sensitive data discovery
- **Enterprise Manager — Security Assessment:** CIS benchmarks, STIGs
- **Critical Patch Updates (CPU):** Quarterly — ensure Oracle DB version is current

#### Custom Regex Scanner for Pro*C SQL Injection

```python
import re, sys

patterns = [
    # String concatenation in SQL — user input directly embedded
    (r'EXEC\s+SQL[^;]*?(?:strcat|sprintf|snprintf|\+\s+)[^;]*?;',
     'String concatenation in EXEC SQL — SQL injection risk'),
    # WHERE clause with potentially tainted host variables
    (r'EXEC\s+SQL[^;]*?\bWHERE\b[^;]*?(:\w+\s*(?:=|LIKE)\s*:\w+)',
     'Dynamic WHERE clause — verify bind variable usage'),
    # Dynamic SQL (EXECUTE IMMEDIATE)
    (r'EXEC\s+SQL\s+EXECUTE\s+IMMEDIATE',
     'Dynamic SQL execution — high risk if not using bind variables'),
    # Format specifier in SQL string literal
    (r'EXEC\s+SQL[^;]*?\'[^;]*?\%[sd][^;]*?;',
     'Format specifier in SQL string — potential injection'),
    # Hardcoded credentials
    (r'(?:password|passwd|pwd)\s*=\s*["\']',
     'Hardcoded credential in source'),
]
```

This catches the most dangerous patterns that C SAST tools miss — not a replacement for proper SAST, but essential gap-filling.

### Best Approach for Pro*C (Summary)

| Step | Tool | What It Catches | Priority | Effort |
|------|------|-----------------|----------|--------|
| 1 | Oracle `proc` → CodeQL/SonarQube on .c | C-level bugs, memory safety, OCI misuse | 🔴 High | Medium (build change) |
| 2 | Custom regex scanner on .pc files | SQL injection in EXEC SQL blocks | 🔴 High | Low (Python script) |
| 3 | Trivy/Grype on compiled binary | Library CVEs (OCI libs, glibc) | 🟡 Medium | Low (CI step) |
| 4 | Oracle DB Security Assessment | DB CVEs, misconfigurations | 🟡 Medium | Medium (DBA time) |
| 5 | Manual code review of EXEC SQL blocks | Complex injection, business logic | 🟢 Recommended | High (expert time) |

---

## 4. Vulnerability Scanning for COBOL

COBOL remains the backbone of financial, insurance, and government systems — running on IBM z/OS mainframes or via GnuCOBOL on Linux/Windows. **No mainstream SAST/SCA tool directly supports COBOL source analysis for security vulnerabilities.**

### IBM z/OS Security Tools (Platform-Level, Not Code-Level)

| Tool | Purpose | Does It Scan COBOL Code? |
|------|---------|-------------------------|
| **IBM RACF** | Access control, authentication, dataset permissions | ❌ No — runtime access control |
| **CA/Broadcom ACF2 / Top Secret** | z/OS access control | ❌ No |
| **IBM ADDI** | Code structure, call graphs, dependency analysis | ⚠️ Structural analysis only; limited security |
| **IBM Data Risk Manager** | Data security classification | ❌ No |
| **IBM QRadar** | SIEM — monitors runtime events | ❌ No |
| **CA Symantec Mainframe Security** | Network security, encryption | ❌ No |

### Enterprise SAST Tools with Limited COBOL Support

| Tool | COBOL Capabilities | Accuracy | Cost | Best For |
|------|-------------------|----------|------|----------|
| **CAST** | COBOL structural analysis, program deps, data flows. Limited security rules (hardcoded creds, dataset access) | Good for structure; limited for security | $$$ | **Best available option** for COBOL — portfolio-wide quality + limited security |
| **SonarQube + COBOL plugin** | Code quality checks (dead code, naming, comments). Very limited security rules (hardcoded creds) | Low for security | Free | Basic code hygiene metrics |
| **Fortify** (Micro Focus/OpenText) | Some COBOL vuln patterns — buffer overflows in PIC clauses, hardcoded creds | Low–moderate | $$$ | Existing Fortify customers extending to COBOL |
| **Checkmarx** | Limited COBOL support — injection flaws, buffer overflows where they map | Low–moderate | $$$ | Existing Checkmarx customers |
| **HCL AppScan** | Limited COBOL support | Low | $$$ | Existing HCL/IBM ecosystem |
| **Veracode** | COBOL source packaging → upload → scan (limited rules) | Limited | $$$$ | Existing Veracode customers |

### GnuCOBOL (OpenCOBOL) Workaround

1. Compile COBOL to C: `cobc -x program.cbl`
2. Run C SAST tools on generated C code
3. Run Trivy/Grype on ELF binary for library vulns

**Catches:** Memory safety in COBOL data structures, buffer overflows, library CVEs (libcob, glibc, GMP)
**Misses:** Business logic security (authorisation bypass, privilege escalation), SQL injection via embedded SQL, JCL privilege issues, dataset access control
**Limitations:** GnuCOBOL ≠ IBM Enterprise COBOL; z/OS-specific features (CICS, IMS/DB, DB2) may not compile; generated C is dense and noisy for SAST

### COBOL-Specific Vulnerability Patterns

No single tool catches these — understanding them is essential:

1. **Buffer overflow in PIC clauses (CWE-120):** COBOL truncates silently on `MOVE`, but data loss can feed downstream C/Java/DB systems
2. **Unvalidated file/record access (CWE-22, CWE-285):** `ACCEPT WS-FILE-NAME` → `OPEN INPUT WS-FILE-NAME` — no validation of user-supplied filenames
3. **Privilege escalation via JCL (CWE-269):** APF-authorised libraries, PROC parameter injection, DD statement overrides. z/OS operational concern — needs system programmer review
4. **SQL injection via COBOL-DB2 embedded SQL (CWE-89):** Same fundamental problem as Pro*C — no tool covers embedded SQL in COBOL
5. **Untrusted CALL target (CWE-470):** `CALL WS-PGM-NAME` with user-supplied program name invokes arbitrary programs
6. **Hardcoded credentials (CWE-798):** `VALUE 'ADMIN001'` / `VALUE 'SECRET42'` in DATA DIVISION — extremely common in legacy COBOL

### Custom COBOL Security Scanning (Most Practical Approach)

Python script using ANTLR grammar or regex patterns on COBOL source:

```python
import re, sys

cobol_patterns = [
    # ACCEPT from user/command-line (CWE-20)
    (r'ACCEPT\s+\w+\s+(?:FROM\s+(?:COMMAND-LINE|SYSIN))',
     'User input via ACCEPT — verify validation and sanitisation'),
    # Dynamic CALL with variable target (CWE-470)
    (r'CALL\s+\w+\s+[A-Z]',  # CALL WS-PGM-NAME
     'Dynamic CALL with variable — restrict module list'),
    # String concatenation in embedded SQL (CWE-89)
    (r'EXEC\s+SQL[^;]*?(?:STRING|CONCATENATE)[^;]*?;',
     'String concatenation in embedded SQL — SQL injection risk'),
    # File I/O without SECURITY clause (CWE-285)
    (r'(?:OPEN|READ|WRITE)\s+\w+\s*(?!.*SECURITY)',
     'File I/O without SECURITY clause — verify access controls'),
    # Hardcoded credentials in VALUE clauses (CWE-798)
    (r'VALUE\s+[\'"]?(?:password|passwd|pwd|userid|secret)',
     'Potential hardcoded credential'),
    # Large PIC X clauses — potential downstream overflow
    (r'PIC\s+X\(\d{4,}\)',
     'Large fixed-length string — verify no overflow downstream'),
]

def scan_cobol(filepath):
    with open(filepath) as f:
        content = f.read()
    for pattern, desc in cobol_patterns:
        for m in re.finditer(pattern, content, re.IGNORECASE):
            line = content[:m.start()].count('\n') + 1
            print(f"{filepath}:{line} — {desc}\n  → {m.group()[:80]}")
```

| Pattern | Risk | Detection |
|---------|------|-----------|
| `ACCEPT ... FROM COMMAND-LINE / SYSIN` | Unvalidated user input | Regex or ANTLR |
| `CALL <variable>` (dynamic module) | Arbitrary program execution | Regex or ANTLR |
| String concat in `EXEC SQL ... END-EXEC` | SQL injection | Regex |
| File I/O without SECURITY clause | Unauthorised dataset access | Regex + AST |
| `VALUE` with credential keywords | Hardcoded credentials | Regex |
| `PIC X(nnnn)` (large strings) | Buffer overflow in downstream systems | Regex |

**cobol-check** (Open Mainframe Project) — unit testing framework for COBOL. Not a scanner, but useful for security regression tests (input validation, boundary conditions, authorisation checks).

---

## 5. Cross-Cutting Tools & Approaches

| Tool / Approach | C/C++ | Pro*C | COBOL | Category | Cost | Notes |
|-----------------|-------|-------|-------|----------|------|-------|
| **CodeQL** | ✅ Excellent | ⚠️ Partial (C parts) | ❌ No | SAST | Free / $ | Custom QL packs for C patterns |
| **SonarQube + C plugin** | ✅ Good | ⚠️ Partial (C parts) | ⚠️ Limited (community) | SAST | Free / $ | C plugin commercial |
| **Coverity** | ✅ Excellent | ⚠️ Partial | ❌ No | SAST | $$$$ | Gold standard safety-critical C |
| **PVS-Studio** | ✅ Good | ⚠️ Partial | ❌ No | SAST | $$ | Low FP |
| **CAST** | ⚠️ Limited | ⚠️ Limited | ✅ Best option | Quality + Security | $$$ | Best COBOL structural analysis |
| **Fortify** | ✅ Good | ⚠️ Partial | ⚠️ Limited | SAST | $$$ | Weak COBOL coverage |
| **Checkmarx** | ✅ Good | ⚠️ Partial (embedded SQL) | ⚠️ Limited | SAST | $$$ | Some Pro*C SQL injection detection |
| **Veracode** | ✅ Good | ⚠️ Limited | ⚠️ Limited | SAST + SCA | $$$$ | COBOL source packaging supported |
| **Trivy** | ✅ Good (binaries) | ✅ Good (binaries) | ✅ Good (binaries) | SCA/Container | Free | Easiest win — works on compiled output |
| **Grype + Syft** | ✅ Good | ✅ Good | ✅ Good | SCA/Container | Free | Pair with Dependency-Track |
| **Black Duck** | ✅ Good | ⚠️ Partial | ❌ No | SCA | $$$$ | Signature matching for vendored C |
| **Snyk** | ✅ Good (containers) | ⚠️ Partial | ❌ No | SCA | Free / $ | Container scanning for C libs |
| **Valgrind / ASan** | ✅ Excellent | ✅ Good (post-proc) | ❌ No | Runtime/Memory | Free | Run in test pipeline |
| **Fuzzing** (libFuzzer, AFL) | ✅ Excellent | ✅ Possible (on preprocessed C) | ❌ No | Runtime | Free | Finds unknown C bugs |
| **Oracle DB Security** | ❌ No | ✅ Recommended | ⚠️ If using DB2 | Database | Included | DBA assessment + CPU patching |
| **z/OS RACF/ACF2** | ❌ No | ❌ No | ✅ Runtime access control | Platform | Included | Essential for z/OS, not code scanning |
| **Custom parser script** | ✅ Possible | ✅ Recommended | ✅ Recommended | Security patterns | Dev effort | **Necessary** for Pro*C and COBOL |

---

## 6. Recommended Approach by Language

### For C

```
Month 1-2                          Month 3-6                       Month 6+
┌──────────────────────┐    ┌──────────────────────┐    ┌──────────────────────┐
│ CodeQL / SonarQube C │    │ ASan in CI           │    │ Coverity/Checkmarx   │
│ Trivy on containers  │ →  │ Fuzzing (libFuzzer)  │ →  │ Custom QL packs      │
│ Clang Static Analyzer│    │ Dependency-Track     │    │ Full SBOM pipeline   │
└──────────────────────┘    └──────────────────────┘    └──────────────────────┘
```

1. **SAST:** CodeQL (free for public repos) or SonarQube + C plugin
2. **SCA:** Trivy/Grype on container images — free, CI-friendly
3. **Memory safety:** Clang/GCC ASan + Valgrind in test pipeline — mandatory
4. **Fuzzing:** libFuzzer or AFL for critical code paths (parsers, network, crypto)
5. **CI:** GitHub Actions / GitLab CI with CodeQL + Trivy + ASan

### For Pro*C

```
Month 1-2                          Month 3-6                       Month 6+
┌──────────────────────┐    ┌──────────────────────┐    ┌──────────────────────┐
│ Oracle proc → C      │    │ Custom regex scanner │    │ Checkmarx/Fortify    │
│ CodeQL on C output   │ →  │ for SQL injection    │ →  │ Oracle Audit Vault   │
│ Trivy on binary      │    │ EXEC SQL patterns    │    │ Full DB assessment   │
│ Manual EXEC SQL rev. │    │ Dynamic SQL detection│    │ DAST on running app  │
└──────────────────────┘    └──────────────────────┘    └──────────────────────┘
```

1. Preprocess with Oracle `proc` → run C SAST on generated .c
2. Custom Python script scanning .pc for SQL injection patterns (string concat in EXEC SQL, `EXECUTE IMMEDIATE`)
3. Trivy/Grype on compiled binary for library CVEs (OCI libs, glibc)
4. Oracle Database security assessment (Audit Vault, Data Safe, CPU patching)
5. Manual code review of EXEC SQL blocks as backup

### For COBOL

```
Month 1-2                          Month 3-6                       Month 6+
┌──────────────────────┐    ┌──────────────────────┐    ┌──────────────────────┐
│ Trivy on binaries    │    │ Custom ANTLR/regex   │    │ CAST for COBOL       │
│ Manual JCL review    │ →  │ scanner for COBOL    │ →  │ RACF/ACF2 audit      │
│ cobol-check for      │    │ ACCEPT, CALL, FILE,  │    │ Fortify (if budget)  │
│ security regression  │    │ SQL, hardcoded creds │    │                      │
└──────────────────────┘    └──────────────────────┘    └──────────────────────┘
```

1. **Primary:** CAST — most comprehensive enterprise option for COBOL structural + limited security analysis
2. **Binary:** GnuCOBOL compile → Trivy/Grype for library vulns
3. **Custom:** Python script (ANTLR grammar or regex) scanning for: ACCEPT, CALL with variable, SQL concatenation, FILE I/O without SECURITY, hardcoded credentials
4. **JCL review:** Manual inspection for privilege escalation, APF-authorised libraries, DD overrides
5. **z/OS:** RACF/ACF2 audit for dataset permissions, program control, runtime access
6. **Testing:** cobol-check for security regression tests

---

## 7. Practical Implementation Strategy

### Month 1–2: "Good Enough"

| Action | Tools | Effort | Impact |
|--------|-------|--------|--------|
| Binary/container scanning | Trivy + Grype | Low (add to build) | High — catches known library CVEs regardless of language |
| Basic SAST on C/Pro*C | CodeQL or SonarQube C plugin | Medium (build changes) | Medium — C-level bugs; misses SQL injection |
| Document known security patterns | Internal wiki + manual review | Medium (expert time) | High — reference for auditors |
| ASan in C test suites | Clang/GCC `-fsanitize=address` | Low (compiler flags) | High — catches memory bugs during dev |

*Effort: 1 security engineer × 2 months (~500K–1M LOC codebase)*

### Month 3–6: "Level Up"

| Action | Tools | Effort | Impact |
|--------|-------|--------|--------|
| Custom Pro*C SQL injection scanner | Python regex script | Low–Med | High — fills the C SAST gap |
| Custom COBOL security scanner | Python + ANTLR grammar | Medium | High — biggest gap in toolkit |
| CI/CD integration | GitHub Actions / GitLab CI | Medium | Medium — automation reduces human error |
| ASan + clang-tidy in CI | Compiler sanitizers | Low | Medium — more memory issues caught |
| Dependency-Track setup | Dependency-Track + Syft | Medium | Medium — new CVE notifications |
| Fuzzing critical C paths | libFuzzer / AFL | Med–High | High — finds unknown bugs |

*Effort: 1 security engineer × 4 months*

### Month 6+: "Enterprise"

| Action | Tools | Effort | Impact |
|--------|-------|--------|--------|
| CAST for COBOL structural analysis | CAST | High (procurement + integration) | Med–High — portfolio-wide COBOL view |
| Coverity/Checkmarx for deep C/Pro*C | Coverity / Checkmarx | High (procurement + integration) | High — catches nuanced bugs |
| IBM RACF/ACF2 audit | z/OS security team | High (mainframe expertise) | High — addresses runtime access control |
| Oracle DB Security Assessment | Oracle Audit Vault / Data Safe | Medium (DBA time) | Medium — reduces database-layer risk |

*Effort: 2 security engineers + platform team ongoing*

---

## 8. Reality Check

| Statement | Verdict |
|-----------|---------|
| "Full SAST coverage for C codebases" | ✅ Yes — CodeQL/Coverity/SonarQube cover C well |
| "Full SAST coverage for Pro*C" | ❌ **No** — SQL injection in EXEC SQL is invisible to C SAST |
| "Full SAST coverage for COBOL" | ❌ **No** — no mainstream tool properly supports COBOL security scanning |
| "Binary scanning covers everything" | ❌ **No** — catches library CVEs but not app logic flaws |
| "Custom scripts are necessary" | ✅ **Yes** — for Pro*C SQL injection and COBOL vulnerability patterns |
| "Manual review is avoidable" | ❌ **No** — especially for COBOL and complex Pro*C SQL blocks |
| "Auditors will accept documented limitations" | ✅ **Yes** — if you thoroughly document toolchain + manual review process |

### High-Risk Patterns to Prioritise

**Pro*C (risk order):** ① SQL injection via string concatenation in EXEC SQL ② Buffer overflow in C parts ③ Oracle DB CVEs ④ Hardcoded DB credentials ⑤ Dynamic SQL (`EXECUTE IMMEDIATE`) with unvalidated input

**COBOL (risk order):** ① JCL privilege escalation / authorisation bypass ② Hardcoded credentials ③ Improper file/dataset access ④ SQL injection via COBOL-DB2 embedded SQL ⑤ Dynamic CALL with untrusted program name ⑥ Buffer overflow in PIC clauses feeding downstream systems

**C (risk order):** ① Memory corruption (use-after-free, buffer overflow — CWE-416/122/121) ② Injection (command injection `system/popen`, format string — CWE-78/134) ③ Integer overflow leading to corruption (CWE-190/680) ④ Unsafe deserialisation / parser bugs ⑤ Race conditions (CWE-362/364)

### Key Takeaways

1. **Perfect coverage is impossible** for Pro*C and COBOL — no single tool, no combination covers everything
2. **Binary scanning is the easiest win** — Trivy/Grype on compiled output works regardless of source language, catches known CVEs in shared libraries
3. **Custom scripts are necessary** for Pro*C and COBOL source-level analysis — invest early
4. **Document everything** for auditors — regulators care more about a documented, systematic approach than perfect coverage
5. **Layer defences:** SAST + binary scanning + custom scripts + manual review + runtime monitoring (RACF, Oracle Audit Vault, SIEM)
6. **Start with binary scanning** — one day to add Trivy to a build pipeline; immediately identifies known CVEs in the software supply chain

---

## 9. Data Sources & References

### Tool Documentation
- CodeQL: https://codeql.github.com/docs/ | SonarQube: https://rules.sonarsource.com/ | Trivy: https://trivy.dev/ | Grype/Syft: https://github.com/anchore/grype | Dependency-Track: https://dependencytrack.org/ | CAST: https://www.castsoftware.com/ | Coverity: https://www.synopsys.com/software-integrity/security-testing/static-analysis-sast.html | Veracode COBOL: https://docs.veracode.com/r/c_compilation_cobol | Fortify: https://www.microfocus.com/en-us/cyberres/application-security/static-code-analyzer | Checkmarx: https://checkmarx.com/ | HCL AppScan: https://www.hcl-software.com/appscan | IBM RACF: https://www.ibm.com/products/zos-security-server | IBM ADDI: https://www.ibm.com/products/application-discovery-and-delivery-intelligence | Oracle Audit Vault: https://www.oracle.com/security/database-security/audit-vault-database-firewall/ | Oracle Data Safe: https://www.oracle.com/database/data-safe/ | Oracle CPU: https://www.oracle.com/security-alerts/ | Clang Static Analyzer: https://clang-analyzer.llvm.org/ | Valgrind: https://valgrind.org/ | libFuzzer: https://llvm.org/docs/LibFuzzer.html | AFL: https://github.com/google/AFL | cobol-check: https://github.com/openmainframeproject/cobol-check | COBOL ANTLR grammar: https://github.com/antlr/grammars-v4/tree/master/cobol

### Standards & Compliance
- CVE/NVD: https://nvd.nist.gov/ | OWASP Top 10: https://owasp.org/www-project-top-ten/ | OWASP C/C++: https://owasp.org/www-project-c-top-10/ | PCI-DSS: https://www.pcisecuritystandards.org/ | SOX: https://www.sec.gov/about/about-securities-laws | HIPAA: https://www.hhs.gov/hipaa/for-professionals/security/ | Basel III: https://www.bis.org/basel_framework/ | DORA: https://www.eiopa.europa.eu/digital-operational-resilience-act-dora_en | EO 14028: https://www.whitehouse.gov/briefing-room/presidential-actions/2021/05/12/executive-order-on-improving-the-nations-cybersecurity/ | NIST SP 800-218: https://csrc.nist.gov/publications/detail/sp/800-218/final | CIS Benchmarks: https://www.cisecurity.org/cis-benchmarks

---

*This document complements the [SBOM Generation for C/Pro*C/COBOL](./sbom_c_proc_cobol.md) research reference. The SBOM document covers bill-of-materials generation for supply chain transparency; this document covers vulnerability scanning and security assessment tooling. Together they cover the security tooling landscape for enterprise/mainframe codebases.*
