# GitHub Copilot Data & Cache Usage — Comprehensive Guide

> **Author:** Jack Liu Shurui, Solution Architect, Crédit Agricole CIB (Singapore)
> **Date:** July 2026 | **Classification:** Internal Use

---

## 1. Overview & Scope

GitHub Copilot is an AI-powered coding assistant (VS Code, JetBrains, Visual Studio, Xcode). This guide covers: what data leaves the developer's machine, what is retained and for how long, local cache/index artifacts, policy controls across tiers, compliance posture for regulated industries, and VS Code settings to minimise telemetry.

---

## 2. What Data Copilot Collects

### 2.1 Prompts (Inputs)

A *prompt* is the contextual bundle sent when the user pauses typing (completions), opens Copilot Chat, invokes agent mode, or requests a code review/PR summary.

**The prompt bundle includes:**

| Component | Details |
|-----------|---------|
| Active file content | Full text of the open editor tab |
| Cursor + surrounding code | ~2,000 tokens of adjacent code |
| File metadata | Path, name, extension, language |
| Chat history | Current session conversation history |
| Workspace index | Embeddings from `@workspace` / `#codebase` |
| Open tabs | Contents of other open editor tabs |
| Terminal output (agent) | Shell output, diffs, tool results in agent mode |
| Git metadata | Branch, commit hash, remote URL |

> Prompts are **transmitted in real-time** over TLS 1.3. Whether they are *retained* depends on subscription tier and telemetry settings (see Section 5).

### 2.2 Suggestions (Outputs)

Generated suggestions are returned over TLS 1.3. **Not stored** for Business/Enterprise (zero-retention for completions and standard chat unless telemetry is enabled). Retained for telemetry on Individual plans with opt-in.

### 2.3 Telemetry Data

Telemetry is distinct from prompt content — it captures **operational and usage metrics**:

| Category | Examples |
|----------|----------|
| **Suggestion events** | Shown, accepted, dismissed, or manually opened |
| **Language breakdown** | Which languages/file types Copilot is used with |
| **Performance** | Time-to-first-suggestion, round-trip latency |
| **Errors** | API errors, extension crashes, network failures |
| **Feature usage** | Completions, chat, agent mode, PR review, CLI |
| **Model selection** | Which model is used (GPT-4o, Claude Sonnet, Gemini, etc.) |
| **Platform info** | VS Code version, OS version, extension version |
| **Anonymous ID** | Pseudonymous identifier for aggregation |

**Telemetry collection by tier:**
- **Individual:** Collected by default; opt-out available.
- **Business/Enterprise:** Not collected without explicit authorisation (contractual prohibition).
- **Retention:** Aggregate metrics — 90 days. Raw interaction data varies by tier.

### 2.4 Agent Mode & Coding Agent Data

Agentic features introduce additional data flows:
- Terminal commands Copilot executes on the developer's machine.
- File creation/modification diffs.
- Git operations (branch, commit, PR creation).
- PR review context: full diffs, issue descriptions, repo metadata.

> **⚠️ Zero-retention applies to real-time completions and standard chat only.** Asynchronous, stateful operations (agent mode, Copilot CLI, PR review) may retain data for session continuity.

---

## 3. What Copilot Does NOT Collect

| Data Type | Collected? | Notes |
|-----------|-----------|-------|
| `.gitignore`-matched files | ❌ | Never indexed or sent |
| VS Code `files.exclude` files | ❌ | Also excluded |
| Content-excluded paths | ❌ | Path-based rules block reading, indexing, transmission |
| OS env variables | ❌ | Never read |
| `.env` files (unopened) | ❌ | **Caution**: if open in editor and not excluded, contents are sent |
| Credential stores / secret managers | ❌ | Never accessed |
| SSH keys, GPG keys, tokens | ❌ | Never read |
| Binary files | ❌ | Images, audio, video, compiled binaries |
| Large files (>~100 KB) | ❌ | Truncated or skipped |
| Private repo code (B/E) | ❌ | Contractually prohibited from model training |

**⚠️ Caveat:** If a `.env` or credentials file is open in the editor and not in `.gitignore` or a content exclusion rule, its contents will be included in the prompt context. *Always* exclude such files.

---

## 4. Data Flow Architecture

### 4.1 Complection & Chat Flow

**Completions:** `IDE pause → Extension bundles prompt (file content, cursor, tabs) → TLS 1.3 → GitHub Copilot Proxy → Auth → Model → Response → IDE`

**Chat:** `User query + editor context + session history + @workspace results → Proxy (content exclusion + dup detection) → Model → Response → IDE`

- Data flows through GitHub's infrastructure (Microsoft Azure).
- **EU Data Boundary** available for Enterprise customers (processing and storage within EU).
- Sub-processors disclosed in Trust Center.

---

## 5. Enterprise vs Individual Controls

### 5.1 Tier Comparison

| Control | Free / Pro | Pro+ | Business | Enterprise |
|---------|-----------|------|----------|------------|
| Data used for model training (default) | ✅ Yes (opt-out) | ✅ Yes (opt-out) | ❌ No (contractual) | ❌ No (contractual) |
| IP indemnity | ❌ | ❌ | ✅ | ✅ |
| Content exclusion | ❌ | ❌ | ✅ | ✅ |
| Duplication detection filter | ❌ | ❌ | ✅ | ✅ |
| Org-level policy controls | ❌ | ❌ | ✅ | ✅ |
| Audit logs / usage reports | ❌ | ❌ | Basic | Full + SIEM |
| DPA | ❌ | ❌ | ✅ | ✅ |
| SOC 2 / ISO 27001 reports | ❌ | ❌ | On request | On request |
| EU Data Boundary | ❌ | ❌ | ✅ | ✅ |
| SAML/SSO | ❌ | ❌ | ✅ | ✅ |

### 5.2 The April 2026 Policy Change

Effective **24 April 2026**, GitHub updated interaction data usage:
- **Free, Pro, Pro+:** Interaction data (prompts, suggestions, context) is collected by **default** for model training. Opt-out via GitHub settings.
- **Business/Enterprise:** Existing contractual protections remain. Data is **not** used for training.
- Opt-out path: `https://github.com/settings/copilot` → uncheck *"Allow GitHub to use my code snippets from the code editor for product improvements."*

### 5.3 Key Enterprise Policy Controls

1. **Duplication Detection Filter** — Blocks suggestions matching ≥150 characters of public GitHub code.
2. **Repository Exclusion** — Restrict Copilot from entire repos.
3. **Content Exclusions** — Path-based rules at repo or org level.
4. **Policy Toggles** — Enable/disable features per user group.
5. **Seat Management** — Assign and revoke Copilot access.

---

## 6. Cache Behavior & Local Storage

### 6.1 Local Embeddings Index

When using `@workspace` / `#codebase`, VS Code builds a **local semantic index**.

**Storage location:**

| OS | Path |
|----|------|
| Linux | `~/.config/Code/User/workspaceStorage/<hash>/GitHub.copilot-chat/` |
| macOS | `~/Library/Application Support/Code/User/workspaceStorage/<hash>/GitHub.copilot-chat/` |
| Windows | `%APPDATA%\Code\User\workspaceStorage\<hash>\GitHub.copilot-chat\` |

**Contents:**
- `workspace-chunks.json` — Text chunks with embedding vectors.
- SQLite databases keyed by file URI and content version.
- In-memory LRU cache for hot embeddings (~few MB).

**What is indexed:**
- All text files except those matched by `.gitignore`, `files.exclude`, or content exclusion rules.
- Files up to ~100 KB (larger files truncated or skipped).
- Index rebuilt on workspace reload or file changes.

### 6.2 Cache Types Summary

| Cache | Location | Purpose | Retention |
|-------|----------|---------|-----------|
| Embedding vector cache | `GitHub.copilot-chat/` SQLite | Avoid recomputing embeddings | Until rebuilt or cleared |
| Completion context cache | In-memory | Recent tabs and edit history | Session lifetime |
| Chat session history | In-memory + extension storage | Conversation continuity | Session (cleared on restart) |
| Extension state | VS Code global storage | Auth tokens, configuration | Persistent |

### 6.3 Managing Local Caches

**Clear local index:**
1. Close VS Code.
2. Delete `GitHub.copilot-chat` directory under workspace storage.
3. Restart VS Code — index rebuilds on next `@workspace` query.

**Clear all Copilot extension data:**
- VS Code command palette → `Developer: Reload with Extensions Disabled`
- Delete `~/.config/Code/User/workspaceStorage/`
- Or use: `GitHub Copilot: Clear Local Index` (if available)

### 6.4 Server-Side Cache

- **Proxy cache:** Completions cached at proxy layer (minutes TTL, not cross-tenant).
- **Telemetry pipeline:** Aggregate metrics stored up to 90 days.
- **Zero retention (Business/Enterprise):** Prompts and suggestions not retained for model training under DPA.

---

## 7. VS Code Settings for Privacy & Telemetry Control

### 7.1 Global Telemetry Level

```jsonc
// settings.json
"telemetry.telemetryLevel": "off"   // or "basic" / "all"
```

- `"off"` — No telemetry sent (crash reporting may still be active).
- `"basic"` — Basic usage data only.
- `"all"` — Full telemetry including prompt metadata.

> ⚠️ Verify Copilot Chat respects this setting — check the Copilot output channel for telemetry logs after disabling.

### 7.2 Copilot-Specific Telemetry Controls

```jsonc
// settings.json
"github.copilot.telemetry": false,
"github.copilot.chat.telemetry": false,
"github.copilot.advanced.disableTelemetry": true,
"github.copilot.advanced.disableSnippetCollection": true,
"github.copilot.advanced.debug.showDebugOverride": false
```

### 7.3 Per-Language Controls

```jsonc
// .vscode/settings.json (workspace-level)
{
    "github.copilot.enable": {
        "python": true,
        "typescript": true,
        "java": true,
        "sql": true,
        "csharp": true,
        "*": false    // Disable for all other languages
    },
    "github.copilot.editor.enableAutoCompletions": true,
    "github.copilot.chat.enabled": true
}
```

### 7.4 Enterprise-Enforced Policy

Managed VS Code installations can enforce via:
- `--disable-telemetry` CLI flag.
- Group Policy (Windows) / `plist` (macOS).
- Installation-level `settings.json` under the Code install directory.
- MDM/vscode-policy for centrally managed settings.

### 7.5 Complete Privacy-First Template

```jsonc
{
    "telemetry.telemetryLevel": "off",
    "github.copilot.enable": { "*": true },
    "github.copilot.editor.enableAutoCompletions": true,
    "github.copilot.chat.enabled": true,
    "github.copilot.telemetry": false,
    "github.copilot.chat.telemetry": false,
    "github.copilot.advanced.disableTelemetry": true,
    "github.copilot.advanced.disableSnippetCollection": true,
    "github.copilot.advanced.debug.showDebugOverride": false,
    "extensions.autoCheckUpdates": false,
    "extensions.autoUpdate": false,
    "workbench.enableExperiments": false,
    "update.mode": "manual"
}
```

---

## 8. Compliance Concerns for Banking & Finance

### 8.1 Code Leakage Risk

**Core concern:** When developers type code with Copilot enabled, fragments of proprietary source code — business logic, DB schemas, API endpoints, auth flows — are transmitted to GitHub's servers.

**Risk scenarios:**

| Scenario | Risk | Notes |
|----------|------|-------|
| Prompt leakage via completions | Medium | Code context sent with every request |
| Model training ingestion | High (Indiv.) / None (Enterprise) | Enterprise has contractual prohibition |
| Chat sharing sensitive context | Medium | Context from open files transmitted in prompts |
| Agentic workflow file access | Med-High | Agent mode can read, modify, commit files |
| Secret exposure via suggestions | Medium | ~6.4% leakage rate in AI-assisted repos |

### 8.2 Regulatory Context

Operating under GDPR (data minimisation, purpose limitation, storage limitation), BCBS 239 (risk data aggregation), AMF/ACPR (outsourcing guidelines), and EU AI Act (transparency).

**Requirements:** Data minimisation, purpose limitation to code completion only, no unnecessary retention, EU data residency preference, SOC 2/ISO 27001 vendor due diligence, and EU AI Act right to explanation.

### 8.3 Mitigating Code Leakage Risk

**Organisational controls:**
1. **Use Copilot Enterprise** — Never Individual/Free tier for bank code.
2. **Execute a DPA** — Ensures no training on bank code, EU data residency, zero retention, right to audit.
3. **Content exclusions** for all repos containing sensitive IP.
4. **Duplication detection filter** enabled.
5. **Enforce VS Code settings** — Telemetry disabled, agent terminal auto-approval off (see Section 7.5).
6. **Mandatory code review** for Copilot-suggested code.
7. **Secret scanning** (GitGuardian, GitHub secret scanning) in CI/CD.

### 8.4 Secret Leakage Research

AI-assisted repos show ~6.4% secret leakage rates (~40% higher than traditional dev). Exposed secrets include AWS credentials, DB passwords, API tokens, SSH keys. **Root cause:** prompting Copilot in files with hardcoded credentials. **Mitigation:** pair secret scanning with content exclusions for secret-matching file patterns.

### 8.5 Terminal Auto-Approval (Agent Mode)

```jsonc
// Mandatory for banking environments
"github.copilot.chat.agent.terminalAutoApprove": false,
// Log all agent-initiated terminal commands for audit
```

**Disable agent mode entirely if not required:**
```jsonc
"github.copilot.chat.agent.enabled": false
```

---

## 9. Content Exclusion & .copilotignore

### 9.1 Purpose

Content exclusion prevents Copilot from reading, indexing, or using specified files as context — more powerful than `.gitignore` because it blocks access even if the file is open in the editor.

### 9.2 Configuration (Per-Repository)

`.github/copilot-exclusions.yml`:
```yaml
paths:
  - "**/secrets/**"
  - "**/credentials/**"
  - "**/*.pem"
  - "**/*.key"
  - "**/.env*"
  - "**/vault/**"
  - "**/compliance/**"
  - "**/PCI-DSS/**"
  - "**/SWIFT/**"
  - "src/internal-security/**"
```

### 9.3 Configuration (Organisation/Enterprise)

GitHub Settings → Copilot → Content Exclusion:
```yaml
paths:
  - "org/repo-a/src/regulatory/**"
  - "org/repo-b/**/secrets/**"
```

### 9.4 Limitations

- **Agentic workflows** may bypass content exclusion in some configurations — verify with testing.
- Exclusion is path-based, not content-based — a file moved to a non-excluded path becomes accessible.
- Does not prevent a developer from manually pasting sensitive code into a chat prompt.

### 9.5 Verification

Check the "GitHub Copilot" output channel in VS Code for exclusion messages. Query `@workspace` about an excluded file — it should return no results. Monitor org audit logs for access attempts.

---

## 10. IP Indemnity & Legal Protections

### 10.1 Coverage

Copilot **Business and Enterprise** plans include IP indemnity:
- If a Copilot suggestion matches (or is alleged to match) third-party copyrighted code, GitHub assumes legal responsibility.
- Covers copyright infringement claims and associated legal costs.
- **Requires duplication detection filter to be enabled** (blocks suggestions matching ≥150 characters of public code).

### 10.2 Conditions

1. Subscription tier: Business or Enterprise (not Free/Pro/Pro+).
2. Duplication detection must be enabled.
3. Suggestion used substantially as generated (minimal modification).
4. Prompt notice to GitHub of any claim.
5. Customer cooperation in defence.

### 10.3 Exclusions

- Code the developer writes that happens to infringe.
- Suggestions from non-GitHub-managed model endpoints.
- Developer modifications that introduce infringement.
- Use in violation of terms or applicable law.

### 10.4 Data Protection Agreement (DPA)

Available for Business/Enterprise customers, providing:
- GDPR Art. 28 compliance (data processor obligations).
- Contractual guarantee: customer data not used for model training.
- Data processing scope limited to providing Copilot service.
- Sub-processor list + change notification.
- Data incident response commitments.
- Return/deletion of data upon termination.

---

## 11. GitHub Copilot Trust Center & Certifications

### 11.1 Copilot Trust Center

The authoritative resource for privacy, security, and compliance documentation: **https://copilot.github.trust.page**

Includes FAQ on data usage, security architecture, feature-specific transparency docs, and compliance reports.

### 11.2 Certifications & Reports

| Certification | Scope | Availability |
|--------------|-------|-------------|
| **SOC 2 Type I** | Copilot Business (IDE completions, chat, CLI, Mobile) | Published June 2024; available to Enterprise customers |
| **SOC 2 Type II** | Ongoing operational effectiveness | On request |
| **ISO/IEC 27001:2013** | GitHub ISMS (incl. Copilot) | Certified |
| **ISO/IEC 27701** | Privacy information management | Extension to ISO 27001 |
| **CSA STAR Level 2** | Cloud security + ISO 27001 attestation | Available |
| **EU SCCs** | Standard Contractual Clauses for data transfers | Included in DPA |

### 11.3 Key URLs

| Resource | URL |
|----------|-----|
| Copilot Trust Center | https://copilot.github.trust.page |
| GitHub Trust Center | https://github.com/trust-center |
| GitHub Privacy Statement | https://docs.github.com/en/site-policy/privacy-policies |
| DPA | Available via Enterprise account management |

---

## 12. Recommended Configuration for Regulated Environments

### 12.1 Minimum Viable Configuration (Banking-Compliant)

**GitHub organisation level:**
- [ ] Assign **Enterprise** seats only (never Individual/Pro).
- [ ] Enable **duplication detection filter**.
- [ ] Configure **content exclusions** for all sensitive repos.
- [ ] Review and accept the **DPA**.
- [ ] Enable **EU Data Boundary** for EU-based users.
- [ ] Enable **audit logging** for Copilot access.

**VS Code workspace level (`.vscode/settings.json`):**
- [ ] `"telemetry.telemetryLevel": "off"`
- [ ] `"github.copilot.telemetry": false`
- [ ] `"github.copilot.chat.telemetry": false`
- [ ] `"github.copilot.advanced.disableTelemetry": true`
- [ ] `"github.copilot.advanced.disableSnippetCollection": true`
- [ ] `"github.copilot.chat.agent.terminalAutoApprove": false`

**Repository level:**
- [ ] Strict `.gitignore` covering secrets and credentials.
- [ ] `.github/copilot-exclusions.yml` for sensitive paths.
- [ ] Secret scanning + SAST in CI/CD.

### 12.2 Audit Checklist

| Item | Status |
|------|--------|
| Enterprise subscription in use | □ |
| DPA signed | □ |
| Content exclusions for regulated repos | □ |
| Duplication detection enabled | □ |
| EU data boundary configured | □ |
| VS Code telemetry disabled via policy | □ |
| Agent terminal auto-approval disabled | □ |
| Secret scanning integrated in CI/CD | □ |
| Code review policy covers AI-suggested code | □ |
| Developer training on Copilot data handling | □ |

---

## 13. References

### Official Docs
- **Copilot Trust Center:** https://copilot.github.trust.page
- **GitHub Trust Center:** https://github.com/trust-center
- **GitHub Privacy Statement:** https://docs.github.com/en/site-policy/privacy-policies
- **Content Exclusion:** https://docs.github.com/en/copilot/concepts/context/content-exclusion
- **2026 Data Policy Update:** https://github.blog/news-insights/company-news/updates-to-github-copilot-interaction-data-usage-policy/
- **Exclude Content:** https://docs.github.com/en/copilot/how-tos/configure-content-exclusion/exclude-content-from-copilot
- **VS Code Telemetry:** https://code.visualstudio.com/docs/configure/telemetry
- **Copilot Metrics:** https://docs.github.com/en/copilot/reference/metrics-data
- **Copilot Indexing:** https://docs.github.com/en/copilot/customizing-copilot/indexing-repositories-for-copilot-chat
- **SOC 2 / ISO Compliance:** https://github.blog/changelog/2024-06-03-github-copilot-compliance-soc-2-type-1-report-and-iso-iec-270012013-certification-scope/
- **Workspace Context:** https://code.visualstudio.com/docs/agents/reference/workspace-context

### Community & Analysis
- **Copilot Privacy (GH Discussion):** https://github.com/orgs/community/discussions/183412
- **MS Security Blog:** https://techcommunity.microsoft.com/blog/azuredevcommunityblog/demystifying-github-copilot-security-controls-easing-concerns-for-organizational/4468193
- **GitGuardian Analysis:** https://blog.gitguardian.com/github-copilot-security-and-privacy/
- **Enterprise Compliance Guide:** https://ai-coding-assistants-explained.dev/docs/copilot/enterprise/
- **Local Index Storage:** https://github.com/orgs/community/discussions/152490
- **Privacy-First VS Code Config:** https://paulsorensen.io/github-copilot-vscode-privacy/
- **Telemetry Config:** https://wisechecker.com/vs-code-github-copilot-telemetry-configuration-explained/
- **Data Leak Prevention:** https://adamtheautomator.com/stop-github-copilot-leaking-enterprise-data-2/
- **Secret Leakage Research:** https://www.mintmcp.com/blog/github-copilot-security-risks

### Regulatory
- **BCBS 239:** https://www.bis.org/publ/bcbs239.htm
- **EU AI Act:** https://artificialintelligenceact.eu/

---

> **Document Version:** 1.0 | **Last Updated:** July 2026
> **Author:** Jack Liu Shurui, Solution Architect, Crédit Agricole CIB, Singapore
>
> *This document is for informational purposes and does not constitute legal advice. Organisations should consult their legal and compliance teams before adopting AI coding assistants in regulated environments.*
