# Infrastructure as Code (IaC) Best Practices — Comprehensive Guide

> **Author:** Jack Liu Shurui | **Last updated:** July 2026

---

## 1. What Is IaC and Why It Matters

Infrastructure as Code (IaC) is the practice of managing and provisioning infrastructure through machine-readable definition files instead of manual processes. You write code describing your desired state; a tool makes it real.

**Why IaC Matters:**

| Benefit | Description |
|---|---|
| **Repeatability** | Same config → identical infrastructure every time. No "works on my machine" for infra. |
| **Consistency** | Dev, staging, prod built from same templates. Eliminates environment-specific drift. |
| **Version Control** | Infrastructure lives in Git alongside app code. Full history of who changed what, when, and why. |
| **Auditability** | Every change tracked via PRs, commit history, and apply logs. Meets SOC 2, PCI-DSS, HIPAA. |
| **Disaster Recovery** | Rebuild entire environment in minutes from IaC definitions. RTO drops from days to hours. |
| **Speed** | Spin up complete environments in minutes. Parallel provisioning. Self-service infra for developers. |
| **Cost Efficiency** | Destroy ephemeral environments when unused. Tag for cost allocation. Prevent orphaned resources. |

---

## 2. Core IaC Principles

### 2.1 Idempotency

Applying the same configuration multiple times produces the same result. If a resource exists and matches desired state, the tool does nothing; if it differs, it reconciles. Terraform/OpenTofu achieve this via state files. Ansible does so through module-level idempotence (check current state before making changes). Pulumi tracks state similarly with SDK-based management.

### 2.2 Declarative vs Imperative

**Declarative** describes *what* the end state should be, not *how* to get there.
- **Tools:** Terraform, OpenTofu, CloudFormation, ARM, Google Deployment Manager
- **Advantages:** Self-healing (manual destruction recreated on next apply). Simpler mental model. Better for compliance/audit.
- **When to use:** Greenfield, multi-cloud, compliance-heavy environments.

**Imperative** specifies *how* to reach the desired state, step by step.
- **Tools:** Pulumi (imperative usage), Ansible, Chef, Puppet, CDK
- **Advantages:** Greater control over ordering, conditional logic, error handling. Familiar to software engineers.
- **When to use:** Complex provisioning workflows, stateful config management, teams wanting programming abstractions.

**Recommendation:** Use declarative for provisioning (VPCs, databases, clusters). Use imperative for post-provisioning steps (installing software, configuring services). Many mature teams use both: Terraform for core infra + Ansible for config management.

### 2.3 Immutable Infrastructure

Servers are never modified after deployment — you replace them entirely.
- **AMI/Golden Images:** Build a machine image (Packer, container image) including OS, dependencies, and app. Never SSH in to patch.
- **Blue-Green Deployments:** Run two environments (Blue=current, Green=new). Route traffic to Green after validation; flip back on failure.
- **Canary Deployments:** Route small % of traffic to new version, gradually increase.

**Benefits:** Eliminates configuration drift. Simple rollback (re-deploy old image). Trivial scaling — new instances are identical copies.

### 2.4 Cattle vs Pets

Treat servers as **cattle** (numbered, disposable, replaceable) not **pets** (named, hand-configured, irreplaceable). If you're attaching EIPs to named EC2 instances and SSHing to configure them, you're treating them as pets. Use load balancers, auto-scaling groups, and immutable images instead.

### 2.5 Drift Detection and Remediation

Configuration drift = actual state diverges from desired state (code). Causes: manual console changes, emergency hotfixes, automated processes outside IaC.

**Detection tools:**
- `terraform plan` / `tofu plan` — diff between config and real infrastructure
- AWS Config — rule-based resource evaluation
- Azure Policy — Azure resource compliance
- OPA / Rego — policy-as-code engine for Terraform, K8s, REST
- CloudHealth / CloudCheckr — multi-cloud drift detection

**Remediation:**
1. **Reactive:** Re-apply (`terraform apply`) to reconcile.
2. **Preventive:** OPA/Sentinel block non-compliant changes before apply.
3. **Automated rollback:** Post-deploy drift detected → auto-rollback.
4. **Scheduled detection:** Run `terraform plan` daily, alert on diffs.

---

## 3. IaC Tools Comparison

### Tool Overviews

**Terraform / OpenTofu** — Declarative, HCL-based. 2,000+ providers. State management with remote backends + locking. Plan/apply workflow. Terraform is BSL-licensed; OpenTofu is MPL 2.0 (open-source fork). Best for multi-cloud provisioning.

**Pulumi** — Declarative + imperative. Uses TypeScript/Python/Go/C#/Java. Same provider ecosystem as Terraform (bridged providers). State managed via Pulumi Cloud or self-managed. Best for teams wanting general-purpose languages.

**AWS CloudFormation / CDK** — AWS-native. CloudFormation: declarative YAML/JSON with AWS-managed state. CDK: imperative constructs in TypeScript/Python/Go/.NET/Java, compiles to CloudFormation. Best for AWS-only shops.

**Ansible** — Imperative, push-based YAML playbooks. Agentless (SSH/WinRM). No persistent state — checks at runtime. Best for configuration management, OS-level provisioning.

**Chef / Puppet** — Agent-based, pull model. Ruby DSL (Chef) or Puppet DSL. Mature config management ecosystems. Less popular for cloud provisioning today; Terraform/OpenTofu/Pulumi dominate cloud IaC.

### Comparison Table

| Dimension | Terraform | OpenTofu | Pulumi | CloudFormation | CDK | Ansible | Chef / Puppet |
|---|---|---|---|---|---|---|---|
| **Type** | Declarative | Declarative | Decl+Imp | Declarative | Imp constructs | Imperative | Imperative/Decl |
| **Language** | HCL | HCL | TS/Py/Go/C#/Java | YAML/JSON | TS/Py/.NET/Java | YAML | Ruby / DSL |
| **State mgmt** | Remote + lock | Remote + lock | Managed/self | AWS-managed | AWS-managed | Stateless | Agent state |
| **Multi-cloud** | ✅ | ✅ | ✅ | ❌ AWS only | ❌ AWS only | ⚠️ Good | ⚠️ Moderate |
| **Config mgmt** | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| **Plan/Preview** | ✅ plan | ✅ plan | ✅ preview | ✅ Change sets | ✅ Change sets | ❌ check mode | ❌ |
| **Module eco.** | ✅ Registry (2K+) | ✅ Registry (2K+) | ✅ Registry | ❌ Limited | ✅ Construct Hub | ✅ Galaxy | ✅ Forge |
| **Open source** | ❌ BSL | ✅ MPL 2.0 | ✅ Apache 2.0 | ❌ | ❌ | ✅ GPLv3 | ✅ Apache 2.0 |
| **Learning curve** | Medium | Medium | Med-High | Medium | Medium | Low | High |
| **Best for** | Multi-cloud provisioning | Open-source IaC | Programmers, complex logic | AWS-only | AWS + devs | Config mgmt | Legacy config mgmt |

---

## 4. Project Structure Best Practices

### 4.1 Module Organization

- **Separate repo or registry for reusable modules.** Versioned, tested, documented library code.
- **Semantic versioning:** `v1.2.3` tags. MAJOR for breaking changes. Changelog required.
- **Standard module structure:**
  ```
  modules/vpc/
    main.tf      # Resources
    variables.tf  # Inputs with type constraints
    outputs.tf    # Derived values
    versions.tf   # Provider constraints
    README.md     # Description, examples, requirements
    examples/basic/
      main.tf
      outputs.tf
  ```
- **Compose modules, don't copy them.** Build higher-level modules by composing lower-level ones (e.g., "web-app" composes vpc + ecs-cluster + rds + alb).
- **Module contracts:** Publish which variables are required vs optional. Use `nullable`, `type`, `validation` blocks.

### 4.2 Repository Structure

**Monorepo** (start here for <15 people): Single source of truth, atomic changes, unified CI.
**Multi-repo** (split when teams specialize): Independent versioning, per-team ownership, smaller blast radius.

**Environment directory structure (monorepo):**
```
infra/
  global/                       # IAM, DNS, org policy
    main.tf | variables.tf | backend.tf
  accounts/
    dev/                        # Per-environment dirs
      main.tf | backend.tf | terraform.tfvars | provider.tf
    staging/
    prod/
  modules/                      # Reusable modules or separate repo
    vpc/ | ecs-service/ | rds/
  terragrunt/                   # When using Terragrunt
    dev/ | staging/ | prod/
```

**Terragrunt for DRY:** Define `provider.tf`, `backend.tf`, and common settings once. Each environment only specifies unique values via `include`.

### 4.3 Naming Conventions

- **Resource naming:** `{org}-{env}-{region}-{type}-{name}` — e.g., `acme-prod-us-east-1-ecs-api`
- **Tagging strategy:** Minimum tags — `Environment`, `Project`, `ManagedBy`, `Owner`, `CostCenter`. Use `default_tags` in providers. Enforce via policy-as-code.
- **State file naming:** `{org}-{env}-{component}.tfstate` — e.g., `acme-prod-networking.tfstate`

---

## 5. State Management

State files map real-world resources to configuration. They must be treated as sensitive.

### 5.1 Remote State + Locking

- **Never use local state for team projects.**
- **AWS:** S3 (storage) + DynamoDB (locking + consistency).
- **Azure:** Storage Account blob + Azurerm lease-based locking.
- **GCP:** GCS bucket + object versioning.
- **Terraform Cloud:** Managed backend with built-in locking.
- **Pulumi:** Cloud backend or self-managed S3/Azure/GCP.
- **OpenTofu:** Same backends + built-in client-side encryption.

**Backend config:**
```hcl
terraform {
  backend "s3" {
    bucket         = "acme-tf-state"
    key            = "prod/networking/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "tf-state-locks"
    encrypt        = true
  }
}
```

### 5.2 State Isolation

- **One state per environment.** Never share across dev/staging/prod.
- **Split by component** for large infrastructures: networking, security, databases, app services.
- **Workspaces** only for similar ephemeral environments (feature branches). Not for production isolation.

| Strategy | Isolation | Complexity | Best For |
|---|---|---|---|
| Separate S3 keys/environments | Full | Low | Standard per-environment |
| Separate backend configs | Full | Medium | Multi-account/region |
| Workspaces | Partial | Low | Ephemeral/CI environments |
| Root module per component | Full | Medium | Large infra, team boundaries |

### 5.3 Sensitive Data in State

State can contain plaintext secrets (passwords, keys, tokens).
1. `sensitive = true` — redacts CLI output but still stored in state.
2. **Encrypt state at rest** — S3 SSE-KMS, OpenTofu client-side encryption.
3. **Restrict access** — IAM policies allowing only specific roles to read state.
4. **Use external secrets** — Vault/AWS Secrets Manager data sources (never hardcoded).
5. **Audit** — S3 access logging, CloudTrail.

### 5.4 Partial Configuration

Avoid hardcoding backend config. Use partial config files or env vars:
```bash
terraform init -backend-config=backend.hcl -backend-config="dynamodb_table=prod-locks"
```

---

## 6. Security Best Practices

### 6.1 Secrets Management

**Never hardcode secrets in IaC.** Use dedicated solutions:

| Service | Integration | Notes |
|---|---|---|
| **HashiCorp Vault** | `vault_generic_secret` data source | Dynamic secrets, rotation, audit |
| **AWS Secrets Manager** | `aws_secretsmanager_secret_version` | Auto-rotation for RDS, native IAM |
| **AWS SSM Parameter Store** | `aws_ssm_parameter` | Free tier, Standard/Advanced tiers |
| **Azure Key Vault** | `azurerm_key_vault_secret` | HSM-backed, RBAC |
| **GCP Secret Manager** | `google_secret_manager_secret_version` | IAM, auto-replication |

**Do NOT:** Hardcode passwords, commit `.env`/`*.secret.tfvars`, or pass secrets unredacted in CI logs.

### 6.2 Least Privilege

- IaC roles need minimum permissions. No admin roles for CI.
- **Separate IAM roles per environment.** Dev CI can't touch prod.
- Restrict who can run `terraform apply` vs `plan`.
- Use AWS permission boundaries for roles IaC creates.

### 6.3 Policy as Code

| Tool | Language | When |
|---|---|---|
| **Sentinel** (TFC) | Sentinel DSL | During Terraform Cloud runs |
| **OPA / Rego** | Rego | Plan-time policy evaluation |
| **Checkov** | Python/YAML | Static scan of config files |
| **tfsec** | HCL analysis | Security-focused scanning |
| **Terrascan** | Rego + custom | Security best practices |
| **Conftest** | Rego | Test configs against OPA policies |

**Enforce at three stages:**
1. **Pre-plan:** Checkov/tfsec on config files in CI.
2. **During plan:** OPA/Sentinel on plan output.
3. **Post-apply:** InSpec/Checkov on running infrastructure.

### 6.4 Secret Scanning & Compliance

| Tool | Purpose |
|---|---|
| **git-secrets** | Prevent secret commits via pre-commit hooks |
| **truffleHog** | Scan git history for high-entropy secrets |
| **Gitleaks** | Git repo audit, regex + entropy detection |
| **Checkov** | CIS benchmarks for AWS/Azure/GCP, PCI-DSS, HIPAA |
| **InSpec** | Compliance-as-code — CIS, SOC 2, PCI controls |

**CI integration:** Run secret scanning on every commit and PR. Block merges on detection.

---

## 7. CI/CD for IaC

### Pipeline Stages

```
[Code Push] → [Lint] → [Validate] → [Plan] → [Security Scan]
→ [Policy Check] → [Approval Gate] → [Apply] → [Integration Test] → [Drift Check]
```

| Stage | Tool / Action | Purpose |
|---|---|---|
| **Lint** | `tflint`, `terraform fmt -check`, pre-commit | Style, formatting, syntax |
| **Validate** | `terraform validate`, `init` | Config correctness, provider compat |
| **Plan** | `terraform plan -out=tfplan` | Show changes. Save plan artifact. |
| **Security Scan** | Checkov, tfsec, Terrascan | Misconfiguration detection |
| **Policy Check** | OPA, Sentinel, Conftest | Org policy on planned changes |
| **Approval Gate** | GitHub Env, GitLab Env, Atlantis | Manual approval for prod |
| **Apply** | `terraform apply tfplan` | Execute saved plan (never fresh apply) |
| **Integration Test** | Terratest, InSpec | Verify deployed infra works |
| **Drift Check** | Scheduled `terraform plan` | Daily alert on any diff |

### Approval Gates

- **Require manual approval for production applies.** Auto-approve dev/staging.
- Policy checks must pass before approval is available.
- Review the plan in the PR (Atlantis, Terraform Cloud comments). Reviewers should check: unintended replacements, security group exposure, cost implications, state manipulation.

### Sandbox & Rollback

- **Ephemeral sandboxes:** Spin up per feature branch, destroy on merge. Use Atlantis, Spacelift, GitHub Actions with `-destroy` on PR close.
- **Drift detection:** Scheduled `terraform plan` on production. Alert on any diff.
- **Rollback strategy:** Save previous state + plan before apply. Re-apply prior config on failure. Blue-green rollback is fastest — just shift traffic.

---

## 8. Testing IaC

### 8.1 Unit Testing

| Tool | Language | Description |
|---|---|---|
| **Terratest** | Go | Most mature — call Terraform, validate outputs, teardown |
| **kitchen-terraform** | Ruby | Test Kitchen driver — converge, verify, destroy |
| **pytest-terraform** | Python | Pytest plugin with fixture-based lifecycle |
| **terraform-compliance** | Python + Gherkin | BDD for Terraform |

```go
// Terratest example
func TestVPCModule(t *testing.T) {
    opts := &terraform.Options{
        TerraformDir: "../examples/basic",
        Vars: map[string]interface{}{"vpc_cidr": "10.0.0.0/16"},
    }
    defer terraform.Destroy(t, opts)
    terraform.InitAndApply(t, opts)
    vpcID := terraform.Output(t, opts, "vpc_id")
    assert.Contains(t, vpcID, "vpc-")
}
```

### 8.2 Static Analysis

- `tflint` — Provider-specific best practices, unused declarations.
- `terraform fmt -check` — Formatting.
- `terraform validate` — Syntax/validity.
- `checkov`, `tfsec`, `terrascan` — Security + compliance.

### 8.3 Integration & Compliance Testing

- **Integration:** Spin up real infra in isolated accounts. Validate connectivity, routing, IAM scoping. Teardown everything. Run on merges to main.
- **Compliance:** InSpec controls verify CIS benchmarks post-deploy:
```ruby
control "cis-2.1.1" do
  describe aws_s3_bucket(bucket_name: 'acme-prod-data') do
    it { should have_versioning_enabled }
  end
end
```

---

## 9. Git Workflows

### 9.1 GitFlow for IaC

- **`main`** — production state. Merge via PR with review + approval.
- **`develop`** — integration branch for non-production work.
- **Feature branches** — `feat/add-rds-replica` from `develop`.
- **Hotfix branches** — `hotfix/critical-patch` from `main`.
- **Release branches** — `release/v1.2` from `develop`, merge to `main` + back to `develop`.

**PR workflow:** Create branch → local validation → open PR → CI runs plan + security scan → reviewer checks plan output → squash merge → CI applies to dev → release PR to main → manual prod approval → apply → post-deploy tests.

### 9.2 Code Review for IaC

Reviewers check:
1. **Security:** Open security groups, unencrypted data, plaintext secrets, overly permissive IAM, public S3 buckets.
2. **Cost:** Instance type sizing, NAT gateway count, provisioned IOPS, data transfer.
3. **Tagging:** Every resource has required tags. Cost allocation breaks without them.
4. **State implications:** `force_new` = resource replacement = downtime. Removed resources = destruction.
5. **Plan output:** Does the plan match what the author intended?
6. **Naming conventions:** Follow established patterns? Environment prefixes correct?
7. **Module versions:** Pinned, not `latest`. Changelog reviewed for version upgrades.

### 9.3 Tagging

- **Module releases:** `v1.2.3` semver tags trigger registry updates.
- **State snapshots:** Tag commit before major apply: `state/prod-2026-07-14-v1`.
- **Release tags:** `prod-v1.2.3` after successful deployment.

---

## 10. Module Design

### 10.1 Reusability & Structure

- Abstract enough for reuse, not so abstract that a module needs 15 optional variables.
- **One concern per module.** Don't create "full-stack" modules. Compose smaller ones.
- Standard interface: `variables.tf` (inputs with types), `outputs.tf` (derived values), `main.tf` (resources), `versions.tf` (provider constraints).
- **Document every module.** README: description, requirements, providers, inputs, outputs, runnable example(s). Use `terraform-docs` to auto-generate.

### 10.2 Versioning & Registry

- **Semantic versioning:** MAJOR (breaking), MINOR (new features, backward-compatible), PATCH (fixes, docs).
- **Changelog required.** Link migration guides for breaking changes.
- **Consumer pinning:** `version = "~> 5.0"` (compatible) or version tag in Git source ref.
- **Private registry:** Terraform Cloud, GitLab module registry, or self-hosted.
- **Cross-org consumption:** Shared Git repo with version tags. Never `ref=main` in production.

### 10.3 Testing Modules

- **Terratest:** Create real env with minimal inputs → validate outputs → verify resources → destroy cleanly.
- **Test fixtures** in `examples/` dir. Each example is a valid, runnable configuration.
- **Test combinations:** Defaults, all optional vars, minimal config, edge cases.

---

## 11. Environment Management

### 11.1 Workspaces

Use workspaces for short-lived environments (feature branches, PR previews, CI sandboxes) where environments are nearly identical. **Do NOT use workspaces** for production isolation, large infrastructures, or multi-team setups.

**Workflow:**
```bash
terraform workspace new dev
terraform workspace select dev
terraform plan       # Plan for dev (different state than prod)
terraform workspace select prod
terraform plan       # Plan for prod
```

### 11.2 Directory Structure for Complex Environments

Use directory-per-environment + component for larger infra:
```
infra/
  dev/networking/ | dev/database/ | dev/app-services/
  staging/networking/ | staging/database/ | staging/app-services/
  prod/networking/ | prod/database/ | prod/app-services/
```
**Terragrunt** eliminates repetition — define shared config once, inherit per environment.

### 11.3 Partial Configuration

Use environment-specific tfvars files:
```bash
terraform apply -var-file=../config/dev.tfvars
terraform apply -var-file=../config/prod.tfvars
```
Set CI/CD variables as `TF_VAR_*` environment variables. Never commit env-specific values.

---

## 12. Terraform-Specific Best Practices

### 12.1 State

- Remote state + locking: S3 + DynamoDB for AWS. Always `encrypt = true`.
- Workspaces for ephemeral. Directory structure for permanent.
- **Never edit state manually.** Use `terraform state mv/rm/pull/push` if needed.
- Migration: `terraform init -migrate-state` — never delete and re-import.

### 12.2 Resource Addressing

- `count` for conditional (0 or 1). `for_each` for map/set with meaningful keys.
- **Avoid resource duplication.** Use `for_each` with local maps:
```hcl
locals {
  subnets = {
    app-1 = { cidr = "10.0.1.0/24", az = "us-east-1a" }
    app-2 = { cidr = "10.0.2.0/24", az = "us-east-1b" }
  }
}
resource "aws_subnet" "this" {
  for_each          = local.subnets
  cidr_block        = each.value.cidr
  availability_zone = each.value.az
  tags = { Name = each.key }
}
```

### 12.3 Provisioners

**Avoid `local-exec` and `remote-exec`** — they operate outside Terraform's state model and are fragile. Use **user-data scripts** for EC2 bootstrapping instead. If you must use provisioners: `create_before_destroy = true`, handle `on_failure` carefully.

### 12.4 Modules

- Source from registry or Git with version pinning. Never `source = "./modules/vpc"` for reusable modules.
- Compose modules. Single responsibility per module.

### 12.5 Cross-Workspace Data

**Prefer data sources** over `terraform_remote_state`. Data sources decouple infrastructure components. Better: export values to SSM Parameter Store / Secrets Manager and read from there.

---

## 13. OpenTofu-Specific Notes

**Migration** from Terraform is straightforward: install `tofu`, alias `terraform=tofu`, run `tofu init`. Existing HCL, state, modules, and providers work unchanged.

**Features beyond Terraform OSS:**
- Client-side state encryption (built-in, no KMS needed).
- Ephemeral values (exist only during a run, never stored in state).
- Linux Foundation governance (community-driven, no single-vendor control).
- Backward compatibility guarantee with HCL, provider SDK, and state format.

**Choosing OpenTofu vs Terraform:**

| Factor | OpenTofu | Terraform (HashiCorp) |
|---|---|---|
| License | MPL 2.0 (fully open-source) | BSL 1.1 |
| Governance | Linux Foundation community | HashiCorp |
| Enterprise features | Community alternatives (Terragrunt, Spacelift) | TFC/Enterprise |
| Unique features | Client-side encryption, ephemeral values | Sentinel, Run Tasks, cost estimation |
| Provider ecosystem | Same (Terraform providers) | Same |

**Recommendation:** OpenTofu if you need open-source licensing and community governance. Terraform if you rely on TFC Enterprise features. Both work with the same HCL modules and providers.

---

## 14. Anti-Patterns

### 14.1 Manual Infrastructure Changes

**Problem:** Someone SSHes into a server, resizes RDS in the console, adds a security group rule via CLI. IaC no longer matches reality.

**Consequences:** Next `terraform apply` either reverts the change (potential downtime) or fails on collision. Configuration drift spreads like technical debt.

**Solution:** All changes through IaC. If an emergency requires manual change, backport to IaC immediately and apply.

### 14.2 Large Monolith State Files

**Problem:** 500+ resources in one state file across VPC, databases, EKS, IAM, DNS.

**Consequences:** Slow plans. High blast radius — one mistake affects everything. Team coordination nightmare — one apply at a time.

**Solution:** Split by environment and component. Each bounded context gets its own state: `prod-networking`, `prod-database`, `prod-app-services`.

### 14.3 Hardcoding Secrets

**Problem:** Passwords, API keys, and private tokens in config files.

**Consequences:** Secrets in Git history. Secrets in state files. Secrets in CI logs. Compromised forever.

**Solution:** Secrets manager (Vault, AWS Secrets Manager, Azure Key Vault) with data source references. `sensitive = true`. No exceptions.

### 14.4 Ignoring State Security

**Problem:** Public/accessible state bucket, no encryption, no access restrictions.

**Consequences:** Attackers see all resource configurations. Plaintext secrets exposed if any were stored.

**Solution:** Encrypt at rest. Restrict IAM to CI/CD + senior devops only. Enable versioning. Audit access. OpenTofu client-side encryption adds another layer.

### 14.5 No Modules / No Testing

**Problem:** Copy-pasted VPC/RDS/IAM code across projects. Changes merged with only `terraform validate`.

**Consequences:** Inconsistency. Security patches must be applied across N copies. Production incidents from untested changes.

**Solution:** Extract repeated resources into modules. Apply testing rigor: static analysis on every PR → plan review → integration tests for critical modules → post-deploy compliance checks.

### 14.6 No Version Control

**Problem:** IaC files on a developer's laptop or unreviewed branch.

**Consequences:** No history. No audit trail. Cannot roll back. No collaboration.

**Solution:** Everything in Git. Everything reviewed via PR. Tagged for releases.

### 14.7 Over-Engineering

**Problem:** 30-variable custom modules for every resource type, 4-level nesting, 12-stage CI pipeline for 20 resources.

**Consequences:** High maintenance. Low velocity. New team members can't understand it. Edge cases break the abstraction.

**Solution:** YAGNI. Use registry modules for standard resources. Custom module only when pattern repeats 3+ times. 2-3 nesting levels max. Optimize for readability.

**Rule of thumb:** If configuring a new service takes longer with your IaC than the cloud console, you've over-engineered it.

---

## Quick Reference: IaC Commandments

1. **Everything as Code** — No manual console changes. Period.
2. **One State Per Environment** — Never share state between dev and prod.
3. **Remote State + Locking** — No local state files in team projects.
4. **Pin Module Versions** — Never use `latest` in production.
5. **Secrets in Secrets Manager** — Never in code, never plaintext in state.
6. **Review Plans, Not Just Code** — The plan is what actually changes.
7. **Test Infrastructure Code** — Lint, scan, validate, plan, test, apply.
8. **Immutable Over Mutable** — Replace servers, don't patch them.
9. **Modules Over Copy-Paste** — DRY applies to infrastructure too.
10. **Least Privilege for IaC** — CI/CD roles scoped and restricted.
11. **Policy as Code** — Security and compliance at every stage.
12. **Tag Everything** — Cost allocation, ownership, environment identification.

---

*This guide is maintained as part of the research repository. Contributions welcome.*
