# Audit as Code: A Comprehensive Guide

**Codifying Audit, Compliance, and Governance into Automated Checks, Policies, and Pipelines**

*Last updated: July 2026*

---

## Table of Contents

1. What Is Audit as Code?
2. Why It Matters
3. Core Concepts
4. Key Tools Comparison
5. Implementing Audit as Code
6. Audit as Code in CI/CD
7. Regulatory Frameworks Coverage
8. Evidence Management
9. Challenges
10. Best Practices
11. Audit as Code Maturity Model
12. Future Trends
13. Code Examples
14. Conclusion & Sources

---

## 1. What Is Audit as Code?

Audit as Code is the practice of treating audit, compliance, and governance requirements as software — version-controlled, tested, automated, and integrated into CI/CD pipelines. Instead of relying on manual checklists, spreadsheets, and point-in-time evidence collection, organisations encode regulatory mandates, security policies, and internal controls as executable code that runs continuously.

At its core, Audit as Code applies DevOps principles to governance:

- **Version control** — Policies, compliance profiles, and evidence scripts live in Git alongside application code, with full history and change tracking.
- **Automated testing** — Compliance checks run on every pull request, not just before an external audit.
- **Continuous enforcement** — Policies gate deployments in real time (e.g., OPA admission control in Kubernetes) rather than relying on periodic manual reviews.
- **Reproducible evidence** — Every run produces timestamped, hashed evidence bundles that auditors can replay and verify independently.

Audit as Code spans four domains: **policy as code** (rules and constraints), **compliance as code** (regulatory requirement tests), **infrastructure as code compliance** (IaC scanning), and **security as code** (application and runtime security tests).

---

## 2. Why It Matters

| Benefit | Description |
|:--------|:------------|
| **Continuous compliance** | Compliance posture is assessed on every deploy, not annually or quarterly. Violations are caught in minutes, not months. |
| **Faster audits** | Auditors get executable evidence bundles with hashed results instead of manual screenshots and PDFs. External audits shrink from weeks to days. |
| **Reduced manual effort** | Automated collection replaces spreadsheets, email chains, and manual evidence gathering. Teams reclaim 60–80% of compliance overhead. |
| **Real-time visibility** | Dashboards show compliance posture per framework (CIS, NIST, PCI-DSS) updated after every pipeline run or scheduled scan. |
| **Audit evidence generation** | Every scan produces a signed, timestamped evidence bundle — configuration snapshots, policy evaluation results, runtime attestation reports. |
| **Faster remediation** | When a policy violation is detected in CI/CD, the developer who introduced it gets immediate feedback. Mean time to remediate drops from weeks to hours. |
| **Shift-left compliance** | Compliance gates move from pre-audit panic to everyday development — the same shift that transformed security testing. |

---

## 3. Core Concepts

### 3.1 Policy as Code

Policy as Code (PaC) encodes rules and constraints in a declarative language that automated systems evaluate against structured data.

- **Open Policy Agent (OPA) / Rego** — Cloud-native policy engine. Rego (pronounced "ray-go") is a declarative query language purpose-built for expressing policies over hierarchical data. OPA evaluates Rego rules against JSON/YAML input for Kubernetes admission control, Terraform plan validation, API authorization, and more.
- **HashiCorp Sentinel** — Policy-as-code framework embedded in HashiCorp products (Terraform, Vault, Consul). Uses a purpose-built language; policies are enforced as part of the Terraform plan/apply lifecycle.
- **Amazon Cedar** — Open-source policy language from AWS used by Amazon Verified Permissions. Designed for fine-grained authorization policies with schema-based validation.

### 3.2 Compliance as Code

Compliance as Code translates regulatory requirements (CIS benchmarks, NIST SP 800-53, PCI-DSS) into executable tests.

- **InSpec** — Open-source compliance testing framework from Chef. Uses a Ruby-based DSL to define controls and describe expected system state. Runs against OS configurations, cloud APIs, and network services.
- **Chef Compliance** — Commercial platform built on InSpec, adding enterprise features (role-based access, reporting, multi-node orchestration).
- **cnspec** — Open-source security and compliance scanner from Mondoo. Uses MQL (Mondoo Query Language) to define policies. Ships with 100+ pre-built policies for CIS, SOC 2, HIPAA, PCI DSS, NIST.

### 3.3 Infrastructure as Code Compliance

Scanning IaC templates (Terraform, CloudFormation, ARM, Pulumi) for misconfigurations before they reach production.

- **Checkov** — Static analysis for Terraform, CloudFormation, ARM, Kubernetes, Helm, Dockerfile, and more. 1000+ built-in policies covering CIS, PCI-DSS, HIPAA, and custom policies.
- **tfsec** — Terraform-specific security scanner with predefined rules for AWS, Azure, and GCP misconfigurations. Integrated into CI/CD via GitHub Actions, GitLab CI, and pre-commit hooks.
- **Sentinel for Terraform** — HashiCorp's policy-as-code framework embedded in Terraform Cloud/Enterprise. Policies evaluated against plan output before apply.

### 3.4 Security as Code

Incorporating security testing into CI/CD pipelines as automated gates.

- **SAST (Static Application Security Testing)** — Semgrep (custom rules, pattern matching), CodeQL (GitHub's query-based analysis), SonarQube.
- **DAST (Dynamic Application Security Testing)** — OWASP ZAP, Burp Suite Enterprise — run against staging environments in CI/CD.
- **SCA (Software Composition Analysis)** — Trivy, Snyk, OWASP Dependency-Check — scan dependencies for known vulnerabilities.
- **Trivy** — All-in-one scanner covering container images, filesystems, Git repositories, Kubernetes, and IaC templates.

### 3.5 Evidence Collection Automation

Automated gathering of control evidence — configuration snapshots, runtime attestations, user access reviews.

- **Scheduled compliance scans** — InSpec/cnspec/cnspec profiles run on cron/scheduler, outputting results to S3/GCS/Azure Blob.
- **Configuration drift detection** — Scheduled scans compare current state against a known-good baseline. Drift is flagged and logged.
- **Runtime attestation** — Tools like Kubeclarity (container attestation) and TPM-backed measurements provide cryptographic proof of runtime state.
- **User access reviews** — Automated extraction of IAM role memberships, last login timestamps, privilege escalation events.

### 3.6 Audit Trails and Logging

- **Structured audit logging** — All policy evaluations, compliance scan results, and configuration changes logged as structured events (JSON) with consistent schemas.
- **Immutable logs** — Logs written to append-only storage (e.g., S3 Object Lock, Azure Immutable Blob Storage, AWS CloudTrail Lake) with cryptographic chain-of-custody.
- **Chain of custody** — Each evidence artifact hashed and linked to the previous artifact in a hash chain, provably demonstrating that no evidence has been tampered with.

---

## 4. Key Tools Comparison

| Tool | Category | Language | Key Use Cases | Frameworks |
|:-----|:---------|:---------|:--------------|:-----------|
| **OPA/Rego** | Policy engine | Rego | K8s admission control, Terraform policy, API authz | Custom policies |
| **InSpec** | Compliance testing | Ruby DSL | OS config audit, cloud compliance, regulatory profiles | CIS, NIST, PCI-DSS, SOC 2 |
| **cnspec** | Security + compliance | MQL | Infrastructure, cloud, containers, K8s compliance | CIS, SOC 2, HIPAA, NIST |
| **Checkov** | IaC scanning | Python | Terraform, CloudFormation, K8s, ARM, Helm, Docker | CIS, PCI-DSS, HIPAA, custom |
| **tfsec** | IaC scanning | Go | Terraform (AWS, Azure, GCP) | CIS, custom |
| **Prowler** | Cloud security | Python | AWS, GCP, Azure CIS benchmarks; CSPM | CIS, NIST, GDPR, HIPAA, PCI |
| **CloudSploit** | Cloud scanning | Node.js | AWS, Azure, GCP misconfigurations | CIS, custom |
| **Semgrep** | SAST | OCaml | Custom static analysis rules, code review gates | OWASP Top 10, custom |
| **CodeQL** | SAST | QL | Variant analysis, code querying | OWASP, CWE, custom |
| **Trivy** | Vulnerability scanner | Go | Containers, filesystems, Git repos, IaC, K8s | CVE, CIS, custom |
| **Sentinel** | Policy engine | Sentinel | Terraform Cloud/Enterprise policy enforcement | Custom policies |
| **Cedar** | Policy engine | Cedar | Fine-grained authorization, AWS Verified Permissions | Custom policies |
| **C/S Graph (Wiz)** | CSPM | N/A | Cloud security graph, attack path analysis | CIS, NIST, PCI, SOC 2 |

### Selection Guidance

- **Need K8s admission control?** → OPA/Rego (Gatekeeper) or Kyverno.
- **Need regulatory compliance audits (CIS, NIST, PCI)?** → InSpec or cnspec.
- **Scanning Terraform at PR time?** → Checkov (broadest coverage) + tfsec (fast, Go).
- **CIS benchmark for AWS/GCP/Azure?** → Prowler (most comprehensive open-source option).
- **Runtime container scanning?** → Trivy.
- **Custom static analysis rules for code?** → Semgrep.
- **Cloud security graph / CSPM?** → C/S Graph (Wiz) or Prowler SaaS.

---

## 5. Implementing Audit as Code

### 5.1 Write OPA Rego Policies for Kubernetes Admission Control

Policies are packaged as **Constraint Templates** (reusable Rego rules) and **Constraints** (parameterised instantiations). Example pattern:

```
OPA Gatekeeper → ConstraintTemplate (Rego) → Constraint (parameters) → AdmissionReview (input) → PASS/FAIL
```

Deploy Gatekeeper via Helm, then apply ConstraintTemplate + Constraints as Kubernetes manifests in a separate `policies/` directory — version-controlled, reviewed, and applied through GitOps (ArgoCD/Flux).

### 5.2 Implement InSpec Compliance Profiles

Each regulatory framework gets its own InSpec profile (a directory with `inspec.yml` and a `controls/` folder). Controls map one-to-one to regulation requirements:

```
profiles/
  cis-aws-foundations/
    inspec.yml
    controls/
      1.1_iam_password_policy.rb
      1.2_iam_users.rb
      2.1_s3_buckets.rb
  nist-800-53/
    controls/
      ac-1_access_control_policy.rb
      au-2_audit_events.rb
```

Run profiles against live infrastructure via `inspec exec` or integrate into CI/CD.

### 5.3 Integrate IaC Scanning into CI/CD

Add Checkov, tfsec, or Trivy as pipeline stages that run on every pull request. Misconfigurations block the build. Results are published to the PR as comments.

### 5.4 Automate Evidence Collection

- **Schedule**: CronJob (K8s) or Lambda/Cloud Function runs compliance profiles every 6–24 hours.
- **Store**: Results written to S3/GCS with versioning enabled and object lock for immutability.
- **Hash**: Each result file includes a SHA-256 hash of the raw scan output, recorded in an evidence manifest.
- **Drift detection**: Compare latest scan against previous baseline; flag deviations for manual review.

### 5.5 Create an Audit Dashboard

Build a Grafana dashboard that queries the compliance results store:

- Overall compliance score (percentage of passing controls) per framework, tracked over time.
- Drill-down: by control, by resource, by severity.
- Trend lines showing compliance improvement/regression over weeks.
- Alerts: when compliance drops below a threshold (e.g., < 95% passing), trigger PagerDuty/Slack notification.

### 5.6 Integrate with SIEM

Feed policy violations and compliance failures into the SIEM (Splunk, Elastic, Sentinel) via structured JSON logs. Correlate policy violations with other events (IAM changes, network anomalies) for contextual alerting.

---

## 6. Audit as Code in CI/CD

### Example: GitHub Actions Pipeline

```yaml
name: Audit as Code Pipeline
on: [pull_request]
jobs:
  iac-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Checkov IaC Scan
        uses: bridgecrewio/checkov-action@v12
        with:
          framework: terraform
          output_format: sarif
      - name: tfsec Scan
        uses: aquasecurity/tfsec-action@v1
        with:
          sarif: true
  policy-check:
    runs-on: ubuntu-latest
    steps:
      - name: OPA Policy Evaluation
        uses: docker://openpolicyagent/opa:latest
        with:
          args: eval --data policies/ --input tfplan.json --format pretty
              "data.terraform.deny"
  compliance:
    runs-on: ubuntu-latest
    steps:
      - name: InSpec Compliance Profile
        uses: docker://chef/inspec:latest
        with:
          args: exec profiles/cis-aws-foundations -t aws://
              --reporter json:evidence/report.json cli
  evidence-bundle:
    runs-on: ubuntu-latest
    steps:
      - name: Generate Evidence Bundle
        run: |
          tar czf evidence-$(date +%Y%m%d-%H%M).tgz evidence/
          sha256sum evidence-*.tgz > evidence-checksum.txt
      - name: Upload Evidence to S3
        uses: aws-actions/configure-aws-credentials@v4
      - run: aws s3 cp evidence-*.tgz s3://audit-evidence-bucket/
```

### Example: GitLab CI Pipeline Stage

```yaml
audit-as-code:
  stage: compliance
  image: bridgecrew/checkov:latest
  script:
    - checkov -d . --framework terraform --output junitxml > checkov-report.xml
    - inspec exec profiles/cis-aws-foundations -t aws:// --reporter json cli
    - opa eval --data policies/ --input tfplan.json "data.terraform.deny"
  artifacts:
    reports:
      junit: checkov-report.xml
    paths:
      - evidence/
```

The pipeline fails if any policy violation is found, preventing non-compliant changes from merging.

---

## 7. Regulatory Frameworks Coverage

| Framework | Tools | What's Automated |
|:----------|:------|:-----------------|
| **CIS Benchmarks (AWS)** | Prowler, InSpec, ScoutSuite | IAM, S3, logging, monitoring, networking — 100+ controls |
| **CIS Benchmarks (GCP)** | Prowler, InSpec | GKE, IAM, Cloud Storage, VPC, logging |
| **CIS Benchmarks (Azure)** | Prowler, Checkov | Azure RBAC, NSGs, Key Vault, SQL, monitoring |
| **CIS Benchmarks (K8s)** | kube-bench, InSpec, Checkov | Pod security, RBAC, etcd, API server, kubelet |
| **CIS Benchmarks (Docker)** | docker-bench-security, InSpec | Host config, daemon config, container runtime |
| **NIST SP 800-53** | InSpec, OPA, cnspec | Access control (AC), audit (AU), config management (CM) |
| **PCI-DSS** | InSpec, Checkov, OPA | Cardholder data protection, access controls, logging |
| **SOC 2** | InSpec, cnspec | Security, availability, confidentiality controls |
| **HIPAA** | Prowler, InSpec, OPA | Data encryption, access logging, backup verification |
| **GDPR** | Prowler, custom InSpec | Cloud config, data retention, access controls |
| **Singapore IM8/GCC** | OPA, InSpec, custom profiles | IAM, data-at-rest encryption, network segmentation |
| **FedRAMP** | InSpec, cnspec | AC, AU, CM, IA, SC, SI — 300+ baseline controls |

### Mapping Controls to Tests

A control-to-test mapping matrix should be maintained in a structured format (YAML/CSV):

```yaml
control: CIS-AWS-1.1
requirement: IAM password policy must enforce min 14 chars
tool: inspec
test: controls/1.1_iam_password_policy.rb
frequency: daily
framework: cis-aws-foundations-v1.5
```

---

## 8. Evidence Management

### 8.1 Automated Evidence Collection

| Collection Type | Frequency | Tool | Output |
|:----------------|:----------|:-----|:-------|
| Compliance scan | Daily | InSpec/cnspec | JSON report with per-control PASS/FAIL |
| Config drift | Hourly | CloudSploit/Prowler | Diff from baseline, severity-tagged |
| Runtime attestation | On deploy | Kubeclarity/TPM | Signed measurement, CVE list |
| Access review | Weekly | Custom scripts | IAM role memberships, last login, unused keys |
| IaC scan | Per PR | Checkov/tfsec | SARIF report, policy violations |

### 8.2 Evidence Storage

- **Immutable buckets**: S3 with Object Lock (COMPLIANCE mode), Azure Immutable Blob Storage, GCS with Bucket Lock.
- **Versioning**: Every evidence artifact is versioned — full history is preserved for audit trail completeness.
- **Tamper-evident hashing**: Each artifact carries a SHA-256 hash. A root manifest contains hashes of all artifacts, itself hashed and optionally published to a transparency log.
- **Retention**: Defined per framework (e.g., PCI-DSS requires 12 months; SOC 2 requires 6 months). Automated lifecycle policies expire evidence after the mandated period.

### 8.3 Evidence Packaging for Auditors

Automated report generation produces a single evidence bundle per audit period:

```
evidence-2026-Q2/
  manifest.json           # Index of all artifacts with hashes
  inspec-results.json     # Compliance scan results
  checkov-results.sarif   # IaC scan results
  prowler-report.html     # Cloud security assessment
  access-review.csv       # IAM access review
  config-snapshots/       # Configuration snapshots per resource
  runtime-attestation/    # Runtime measurements and signatures
  chain-of-custody.log    # Hash chain linking all artifacts
```

The bundle is uploaded to the evidence bucket, and a signed URL is shared with the auditor. The auditor can independently replay any scan using the exact profile versions recorded in `manifest.json`.

---

## 9. Challenges

| Challenge | Description | Mitigation |
|:----------|:------------|:-----------|
| **Policy coverage gaps** | Not all controls can be automated (e.g., organisational policies, physical security, subjective judgments). | Focus automation on technical controls (60–80% coverage); maintain manual evidence templates for the rest. |
| **False positives** | Scanners flag benign configurations as violations, creating alert fatigue. | Tune policies with exception lists, severity thresholds, and a false-positive review process. |
| **Policy drift** | Code changes may violate policies not covered by existing tests. | Expand test coverage iteratively; use property-based testing; maintain a control-to-test mapping matrix. |
| **Cross-functional skillset** | Requires DevOps + compliance + security + software engineering expertise, a rare combination. | Build a Centre of Excellence (CoE) with compliance engineers; pair compliance officers with DevOps engineers. |
| **Initial investment** | Writing policies and profiles for 200+ controls takes significant upfront effort. | Start with the top 20 highest-risk controls; build incrementally. Prioritise frameworks already enforced. |
| **Tool fragmentation** | No single tool covers all frameworks and environments. | Standardise on 3–4 core tools (e.g., InSpec + Checkov + OPA + Prowler) and build integration wrappers. |
| **Evidence reproducibility** | If the tool version or profile changes, old evidence may not be reproducible. | Lock tool and profile versions in CI/CD; store versions alongside evidence. |
| **Noise suppression** | Ten-thousand-passing-controls to one-failing-control ratio makes real issues invisible. | Separate "blocking" violations (severity critical/high) from "advisory" warnings (medium/low). |

---

## 10. Best Practices

1. **Start with high-risk controls** — Map your top 20 compliance risks and automate those first. Prove value, then expand coverage.
2. **Version control everything** — Policies, compliance profiles, CI/CD configurations, evidence schemas — all in Git with semantic versioning.
3. **Test policies with TDD** — Write tests for each policy in Rego (OPA test runner) or InSpec (describe blocks). Policy changes follow the same PR review cycle as application code.
4. **Combine automated checks with manual review** — Subjective controls (e.g., "segregation of duties is documented") need a manual sign-off workflow integrated into the same system.
5. **Match evidence frequency to risk** — Critical controls (access keys, encryption) checked hourly; non-critical controls (tagging conventions) checked weekly.
6. **Maintain a control-to-test mapping matrix** — Every regulatory requirement traces to a specific automated test. When regulation changes, you know exactly which tests to update.
7. **Run Audit as Code in normal CI/CD** — Compliance is a daily practice, not a pre-audit panic. Non-compliant changes should block merges just like failing unit tests.
8. **Periodically review policy effectiveness** — Quarterly review of false-positive rates, coverage gaps, and policy bypass incidents. Retire ineffective policies; add missing ones.
9. **Document policy exceptions with approval workflow** — When a policy must be overridden, require a documented exception (business justification, owner, expiry date) in a Git-managed exceptions file.
10. **Lock tool versions** — Pin scanner and profile versions in CI/CD to ensure evidence reproducibility across audit periods.
11. **Automate the auditor package** — Generate the evidence bundle and shareable auditor portal automatically at the end of each reporting period.
12. **Invest in dashboards** — A compliance dashboard visible to engineering teams drives accountability faster than quarterly reports to the board.

---

## 11. Audit as Code Maturity Model

| Level | Name | Characteristics | Tools & Practices |
|:------|:-----|:----------------|:------------------|
| **L1** | Ad-Hoc / Manual | Evidence collected via spreadsheets and email. No automation. Annual audit scramble. Screenshots as evidence. | Manual checklists, Outlook, SharePoint |
| **L2** | Automated Scanning | Scheduled scans run but outputs are not centrally managed. No CI/CD integration. Evidence is reviewed but not versioned. | Standalone Prowler, manual InSpec runs, cron jobs |
| **L3** | Codified Policies | Policies and compliance profiles in Git. Scans run in CI/CD (every PR). Evidence stored in versioned buckets. Fixes tracked. | OPA + Gatekeeper, InSpec in CI/CD, Checkov in PRs, S3/GCS versioned buckets |
| **L4** | Integrated GRC | Real-time policy enforcement (OPA admission control). Automated evidence packaging. Compliance dashboard with trend lines. SIEM integration. Exceptions workflow in Git. | OPA Gatekeeper, Grafana dashboards, automated evidence bundles, SIEM pipeline |
| **L5** | Predictive & Self-Healing | Continuous attestation (eBPF/runtime). Predictive compliance (ML flags risk before violation). Automated remediation of policy violations. AI-augmented policy generation from regulatory text. Blockchain-based immutable audit trails. | eBPF-based runtime attestation, AI-augmented policy generation (Regulatory→Rego), automated remediation (Lambda/runbooks), blockchain evidence notary |

Most organisations today are at **L2–L3**. Moving to L4 requires investment in real-time enforcement and evidence automation. L5 is emerging — early adopters in regulated fintech and government clouds are piloting continuous attestation.

---

## 12. Future Trends

### 12.1 AI-Augmented Policy Generation

Large language models are being used to translate regulatory text (e.g., a new CIS benchmark release or a revised PCI-DSS section) directly into Rego rules or InSpec controls. Human-in-the-loop review validates generated policies before deployment. Early research shows 70–80% reduction in policy authoring time.

### 12.2 Real-Time Continuous Auditing with eBPF

eBPF (extended Berkeley Packet Filter) enables runtime kernel-level monitoring without modifying application code. Tools like Cilium (for Kubernetes network policies) and Falco (runtime security) provide continuous attestation of in-flight behaviour. Combined with OPA, this enables **just-in-time compliance verification** — every system call, network connection, and file access is evaluated against policy at the moment it happens.

### 12.3 Policy as Code for AI/ML Governance

As AI regulation matures (EU AI Act, Singapore's AI Verify, NIST AI RMF), policy-as-code frameworks are extending into ML pipeline governance: model fairness rules, data provenance checks, model card generation, and bias detection thresholds encoded as OPA policies evaluated during the ML training and deployment pipeline.

### 12.4 Automated Remediation

Policy violations trigger automated remediation workflows (Lambda functions, Cloud Functions, Ansible playbooks) that restore compliance without human intervention. Remediation is logged, tested, and auditable — closing the loop from detection → notification → fix → verify.

### 12.5 Blockchain for Immutable Audit Trails

Evidence hashes published to a blockchain (Ethereum, Hyperledger, or a transparency log like Trillian/DigilCert) provide cryptographic proof that evidence existed at a specific point in time and has not been altered. This satisfies the most stringent chain-of-custody requirements.

### 12.6 Compliance-as-Code Marketplaces

Organisations are beginning to share and sell reusable compliance profiles — a "marketplace" for CIS, NIST, SOC 2, and industry-specific profiles. InSpec Supermarket (from Chef) was an early example; Mondoo's cnspec hub and community Rego policy libraries are following. This dramatically reduces the initial investment barrier.

---

## 13. Code Examples

### 13.1 OPA Rego Policy — Kubernetes Admission Control

Prevent deploying containers running as root in a Kubernetes cluster:

```rego
package kubernetes.admission

import data.kubernetes.admission

deny[msg] {
    input.request.kind.kind == "Pod"
    container := input.request.object.spec.containers[_]
    not container.securityContext.runAsNonRoot
    msg := sprintf("Container %v in %v must set runAsNonRoot=true",
        [container.name, input.request.object.metadata.name])
}

deny[msg] {
    input.request.kind.kind == "Pod"
    container := input.request.object.spec.containers[_]
    container.securityContext.runAsUser == 0
    msg := sprintf("Container %v in %v runs as root (UID 0)",
        [container.name, input.request.object.metadata.name])
}
```

### 13.2 InSpec Compliance Profile — CIS AWS IAM Controls

```ruby
# cis-aws-foundations/controls/1.1_iam_password_policy.rb
title 'CIS AWS 1.1 — IAM password policy enforces minimum password length'

control 'cis-aws-1.1' do
  impact 0.7
  title 'Ensure IAM password policy requires at least 14 characters'
  desc 'Password policies in IAM should enforce a minimum of 14 characters.'

  describe aws_iam_password_policy do
    it { should exist }
    its('minimum_password_length') { should be >= 14 }
    its('require_uppercase_characters') { should be true }
    its('require_lowercase_characters') { should be true }
    its('require_numbers') { should be true }
    its('require_symbols') { should be true }
    its('password_reuse_prevention') { should be >= 24 }
  end
end
```

### 13.3 Checkov IaC Scan — CI/CD Integration

```yaml
# .github/workflows/iac-security.yml
name: IaC Security Scan
on: [pull_request]
jobs:
  checkov-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run Checkov
        id: checkov
        uses: bridgecrewio/checkov-action@v12
        with:
          directory: terraform/
          framework: terraform
          skip_check: CKV_AWS_115  # Known exception — approved
          output_format: cli
          quiet: true
      - name: Post Results to PR
        uses: actions/github-script@v7
        if: failure()
        with:
          script: |
            const output = `⚠️ **Checkov found policy violations**
            See scan details in the workflow logs.
            [Workflow run](${context.serverUrl}/${context.repo}/actions/runs/${context.runId})`;
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: output
            });
```

---

## 14. Conclusion

Audit as Code transforms compliance from a periodic, manual, and reactive burden into a continuous, automated, and proactive practice. By treating regulatory requirements as code — versioned, tested, enforced in CI/CD, and producing reproducible evidence — organisations reduce audit cycles from weeks to days, cut manual compliance overhead by 60–80%, and gain real-time visibility into their compliance posture.

The journey follows a maturity curve: from manual spreadsheets (L1) through automated scanning (L2) and codified policies (L3) to real-time enforcement (L4) and predictive compliance (L5). Most teams should start with the top 20 highest-risk controls using OPA + InSpec + Checkov, prove value, then expand coverage iteratively.

In an era of tightening regulation (AI Act, GDPR, Singapore IM8) and increasingly complex cloud environments, Audit as Code is not just a best practice — it is becoming the expected standard for any organisation operating at scale.

### Sources

- Open Policy Agent: openpolicyagent.org/docs — Rego language reference, policy design patterns
- Chef InSpec: docs.chef.io/inspec — Compliance profile authoring guide
- Checkov: checkov.io — IaC scanning documentation and policy reference
- Prowler: docs.prowler.cloud — Cloud security assessment framework
- cnspec by Mondoo: mondoo.com/cnspec — Security and compliance as code
- CIS Benchmarks: cisecurity.org/benchmark — Benchmark definitions for AWS, GCP, Azure, K8s, Docker
- NIST SP 800-53: csrc.nist.gov/publications/detail/sp/800-53/rev-5 — Security and privacy controls
- PCI-DSS: pcisecuritystandards.org — Payment card industry data security standard
