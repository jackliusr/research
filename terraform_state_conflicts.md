# Terraform State Conflicts & Race Conditions — Self-Managed Backends

**Generated:** 2026-06-08
**Scope:** Comprehensive guide to avoiding state conflicts, race conditions, and state corruption when using Terraform/OpenTofu WITHOUT a managed SaaS backend (no Terraform Cloud, Spacelift, env0, Scalr).

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Remote State Backends (Self-Managed)](#1-remote-state-backends-self-managed)
3. [State Locking Deep Dive](#2-state-locking-deep-dive)
4. [Team Workflow Patterns](#3-team-workflow-patterns)
5. [CI/CD Pipeline Serialization](#4-cicd-pipeline-serialization)
6. [Common Race Conditions & Solutions](#5-common-race-conditions--solutions)
7. [Complementary Tools](#6-complementary-tools)
8. [State Migration Strategies](#7-state-migration-strategies)
9. [Disaster Recovery](#8-disaster-recovery)
10. [Quick Reference](#9-quick-reference)

---

## Executive Summary

Terraform state conflicts happen when multiple processes attempt to modify the same state file concurrently. Without a managed SaaS platform, you need to implement your own locking, serialization, and workflow enforcement.

**Key principles:**
1. **Always use remote state** with a locking backend — never local state in teams
2. **Serialize applies** — never allow two concurrent `terraform apply` runs on the same state
3. **Plan and apply in same pipeline run** — avoid plan drift
4. **Enforce GitOps** — plan in PR, apply on merge to main; no local applies in production
5. **Enable state versioning** — always (S3 versioning, GCS object versioning, Azure blob soft delete)
6. **Automate lock cleanup** — stale locks block your team

---

## 1. Remote State Backends (Self-Managed)

### 1.1 S3 + DynamoDB (AWS) — Gold Standard

**How DynamoDB Locking Works:**
Terraform writes a lock item to a DynamoDB table using `PutItem` with a condition:
```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/networking/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-state-locks"
    encrypt        = true
  }
}
```

| Aspect | Detail |
|--------|--------|
| **Lock mechanism** | `PutItem` with `attribute_not_exists(LockID)` — atomic, distributed |
| **Partition key** | `LockID` = `{bucket}/{key}/terraform.tfstate-md5` |
| **Lock item fields** | LockID, Info (JSON: ID, version, operation, who, created, path), Created (timestamp) |
| **Stale lock** | Item remains if apply crashes; must be cleaned up manually or via automation |
| **Force unlock** | `terraform force-unlock <LOCK_ID>` — only after verifying no apply is running |

**Best Practices:**
- Enable DynamoDB **point-in-time recovery**
- One DynamoDB table per environment
- Never manually delete lock items while an apply might be running
- Set `-lock-timeout=10m` in CI/CD pipelines

**Stale lock detection script:**
```bash
aws dynamodb scan --table-name terraform-state-locks --query 'Items[*].[LockID.S,Created.N]' --output json |   python3 -c "
import sys, json, time
for item in json.load(sys.stdin):
    lock_id, created = item
    age = time.time() - float(created)
    if age > 1800:
        print(f'STALE LOCK: {lock_id} (age: {age/60:.0f} min)')
"```

### 1.2 GCS (Google Cloud Storage)

**How Locking Works:**
Uses object versioning + conditional writes for lock files:
```hcl
terraform {
  backend "gcs" {
    bucket      = "my-terraform-state"
    prefix      = "prod/networking"
    credentials = "path/to/sa-key.json"
  }
}
```

| Aspect | Detail |
|--------|--------|
| **Lock mechanism** | Writes `.tfstate.lock.info` with `x-goog-if-generation-match` condition |
| **Stale lock recovery** | `gsutil rm gs://my-bucket/.tfstate.lock.info` or `terraform force-unlock` |
| **Versioning** | Enable object versioning on the bucket for state recovery |
| **Lifecycle rules** | Auto-delete lock files older than 24 hours |

### 1.3 Azure Storage Blob

**How Lease-Based Locking Works:**
```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "terraform-state-rg"
    storage_account_name = "mytfstate"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
  }
}
```

| Aspect | Detail |
|--------|--------|
| **Lock mechanism** | Acquires a write lease on the blob (state file) |
| **Stale lock risk** | **Leases never expire** by default — this is Azure's biggest limitation |
| **Breaking a stale lease** | `az storage blob lease break --account-name mytfstate --container tfstate --blob-name prod.terraform.tfstate --lease-break-period 60` |

**Recommendation:** Prefer S3+DynamoDB over Azure Blob for reliability of lock cleanup.

### 1.4 HashiCorp Consul

```hcl
terraform {
  backend "consul" {
    address = "consul.example.com:8500"
    path    = "terraform/prod/network"
    lock    = true
  }
}
```

| Aspect | Detail |
|--------|--------|
| **Lock mechanism** | Consul sessions + KV store with Check-and-Set |
| **Stale lock handling** | **Automatic** — session TTL (default 15s) auto-releases locks on client death |
| **Best for** | Teams already running Consul; best auto-cleanup of any backend |

### 1.5 HTTP Backend (Custom Locking)

For teams that want a simple REST endpoint:

```hcl
terraform {
  backend "http" {
    address       = "https://locker.mycompany.io/state"
    lock_address  = "https://locker.mycompany.io/lock"
    unlock_address = "https://locker.mycompany.io/unlock"
    lock_method   = "PUT"
    unlock_method = "DELETE"
  }
}
```

Server must implement: return 200 success, 409 if locked, 423 if held by another.

### Backend Comparison

| Backend | Lock Type | Auto-Cleanup? | Stale Risk | Recovery Method | Best For |
|---------|-----------|--------------|------------|-----------------|----------|
| **S3+DynamoDB** | DB item condition | ❌ Manual | Low (visible items) | force-unlock, delete item | **Most teams** |
| **GCS** | Object generation | ❌ Manual | Low | Delete lock file | GCP-native |
| **Azure Blob** | Lease | ❌ No | **HIGH** (never expires) | `az blob lease break` | Azure-only; use with caution |
| **Consul** | Session + KV | ✅ Auto (TTL) | None | Session expires | Consul users |
| **HTTP** | Custom | Depends on impl | Depends | Custom endpoint | Custom infra |

---

## 2. State Locking Deep Dive

### 2.1 Lock Lifecycle

```
terraform apply
  │
  ├── Attempt to acquire lock
  │     ├── Success → continue
  │     └── Fail → retry until -lock-timeout expires
  │
  ├── Read current state
  ├── Compute and execute changes
  ├── Write new state
  └── Release lock
```

**Timeout behavior:** `-lock-timeout=5m` — waits up to 5 minutes for lock, then fails.

### 2.2 Interrupted Apply (Mid-Lock)

| Event | What happens | Risk |
|-------|-------------|------|
| **Ctrl+C (SIGINT)** | Terraform's signal handler releases the lock | Low — lock released cleanly |
| **Process killed (SIGKILL)** | Lock NOT released | Stale lock remains; manual cleanup needed |
| **Network failure during state write** | Partial state file | State corruption; restore from versioned backup |

### 2.3 Force-Unlock Risks

**ONLY use `terraform force-unlock` when you are 100% certain NO Terraform process is running against that state.**

**Deadly scenario:** Alice force-unlocks while Bob's apply is running → Bob's apply continues → second process acquires the same lock → **two concurrent writes → state corruption**.

**Safe force-unlock checklist:**
1. Check CI pipeline status (GitHub Actions, CodePipeline, Jenkins)
2. Check who the lock owner is (DynamoDB `Info` field contains user/path)
3. Verify the lock owner's machine/pod is terminated
4. Announce in team Slack before force-unlocking
5. Only then: `terraform force-unlock <LOCK_ID>`

### 2.4 Partial State Writes

**Symptoms:** `Error loading state: failed to decode...`

**Causes:**
- Terraform crashes mid-write
- Network failure during PUT to backend

**Recovery:**
```bash
# S3 — restore previous version
aws s3api get-object --bucket my-bucket --key path/terraform.tfstate   --version-id <prev-version> recovered.tfstate

# Push recovered state
terraform state push recovered.tfstate

# Verify
terraform plan
```

---

## 3. Team Workflow Patterns

### 3.1 The Release Branch Model

```
Feature Branch → PR → terraform plan (auto-comment on PR)
         ↓
    Merge to main → terraform plan (fresh) → terraform apply (serialized)
```

**Rules:**
- **One branch drives applies** — main branch only
- Plan and apply in the **same pipeline run** (seconds apart, not hours)
- **No local `terraform apply`** in production — enforce via branch rules and tooling
- All state modifications go through CI/CD

### 3.2 Workspace vs Per-Environment Backend

| Pattern | Setup | Pros | Cons |
|---------|-------|------|------|
| **Workspaces** | Single backend, state key includes workspace name | Simple configswitching | Shared lock table; accidental workspace targeting |
| **Per-environment backends** | Separate backend config per env | Complete isolation; separate locks | More infra to manage |

**Recommendation:** Per-environment backends for prod vs non-prod. Within same environment, workspaces are fine.

### 3.3 GitOps Workflow (No Terraform Cloud)

```
1. Create PR → GitHub Actions runs terraform plan
2. Plan output posted as PR comment
3. Team reviews plan in PR
4. Merge to main → GitHub Actions runs terraform plan + terraform apply (serialized)
5. On failure → rollback commit or fix forward
```

---

## 4. CI/CD Pipeline Serialization

### 4.1 GitHub Actions

```yaml
name: Terraform
on:
  pull_request:
    branches: [main]
    paths: ['infra/**']
  push:
    branches: [main]
    paths: ['infra/**']

jobs:
  plan:
    runs-on: ubuntu-latest
    steps:
      - uses: hashicorp/setup-terraform@v3
      - run: terraform init
      - run: terraform plan -out=tfplan

  apply:
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    needs: [plan]
    concurrency: terraform-apply  # ← CRITICAL: serializes all applies
    runs-on: ubuntu-latest
    steps:
      - uses: hashicorp/setup-terraform@v3
      - run: terraform init
      - run: terraform plan -out=tfplan  # fresh plan
      - run: terraform apply -auto-approve -lock-timeout=10m tfplan
```

### 4.2 GitLab CI

```yaml
terraform-apply:
  stage: deploy
  resource_group: terraform-production  # ← serializes applies
  script:
    - terraform init
    - terraform plan -out=tfplan
    - terraform apply -auto-approve tfplan
  only:
    - main
```

### 4.3 AWS CodePipeline

- **executionMode: SINGLE** (settable on the pipeline) — ensures only one execution runs at a time
- Pipeline stages are inherently serial per execution
- Pass plan file as artifact from Plan stage → Approval → Apply stage

### 4.4 Jenkins

```groovy
lock('terraform-state') {  // blocks concurrent builds using same resource
    sh 'terraform apply -auto-approve tfplan'
}
```

### 4.5 Critical: Plan Drift Prevention

**The most common race condition:**
1. Alice runs `terraform plan` at 10:00
2. Bob applies a change at 10:05
3. Alice runs `terraform apply tfplan` at 10:06 → plan is based on STALE state

**Solution:** Plan and apply in the same pipeline run within seconds. Never save a plan file and apply it hours later.

```yaml
# GOOD: plan + apply in same job, seconds apart
- run: terraform plan -out=tfplan
- run: terraform apply -auto-approve tfplan

# BAD: plan in one PR comment, apply days later from saved file
```

---

## 5. Common Race Conditions & Solutions

| Scenario | Risk | Solution |
|----------|------|----------|
| **Two devs apply simultaneously** | Second gets lock error; worst case: both crash | Concurrency groups in CI; GitOps serialization |
| **Plan based on stale state** | Unexpected updates; resource deletion | Plan + apply in same pipeline run |
| **Force-unlock while apply running** | **State corruption** | Verify process dead before force-unlock |
| **Partial apply failure** | State OK but infra partial | Re-run apply; Terraform creates missing resources |
| **State file corruption** | JSON parse errors | Restore previous version; `terraform state push` |
| **Two pipelines on same branch** | Concurrent applies to same state | `concurrency` in GitHub Actions; `resource_group` in GitLab |
| **Local apply bypasses CI lock** | State out of sync with CI | Branch protection + `.gitignore` local state; team policy |
| **DynamoDB lock table deleted** | All applies blocked | Point-in-time recovery; IaC the lock table |

### Plan Drift — The Most Common Problem

**How to prevent it:**
1. Always run `terraform plan -out=tfplan` immediately before `terraform apply tfplan` in the same CI step
2. Ensure the time between plan generation and apply is under 60 seconds
3. After apply, run `terraform plan` again to confirm no unexpected diff
4. Never manually apply a plan file created by someone else

---

## 6. Complementary Tools

### 6.1 Terragrunt (Gruntwork)

**What it solves:** Auto-initialization of remote state + dependency-aware execution ordering

```hcl
# terragrunt.hcl — auto-creates S3 bucket and DynamoDB table
remote_state {
  backend = "s3"
  generate = {
    path      = "backend.tf"
    if_exists = "overwrite_terragrunt"
  }
  config = {
    bucket         = "my-terraform-state-${get_aws_account_id()}"
    key            = "${path_relative_to_include()}/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-state-locks"
  }
}
```

**Dependency-aware DAG execution:**
```hcl
dependency "vpc" {
  config_path = "../vpc"
}
```
`terragrunt run-all apply` runs in topological order, serializing applies to the same module.

### 6.2 Atlantis (runatlantis.io)

**What it solves:** PR-driven Terraform with automatic serialization and project-level locks

```
PR opened → auto terraform plan → comment on PR with output
PR comment "atlantis apply" → queue → serialized apply
```

- **Project-level locks:** Prevents concurrent plans/applies on same project
- **Locks out manual applies:** Blocks `terraform apply` outside Atlantis
- **Self-hosted:** Deploy on ECS/EKS/EC2 — fully managed by you

### 6.3 Terramate

**What it solves:** Change detection + stack ordering for Terraform/OpenTofu

```hcl
# stack.tm.hcl
stack {
  name  = "networking"
  after = ["bootstrap"]
}

parallelism 3  # max concurrent stacks, respecting deps
```

`terramate run --changed -- terraform plan` only runs on stacks with git-detectable changes.

### 6.4 OpenTofu

**Drop-in replacement for Terraform** with identical state locking behavior. All commands, backends, and lock mechanisms are the same:
- `tofu plan` → `tofu apply` → `tofu force-unlock`
- Same `backend "s3"` config blocks
- Same DynamoDB lock table format

---

## 7. State Migration Strategies

| Migration | Command | Notes |
|-----------|---------|-------|
| **Local → Remote** | `terraform init -migrate-state` | Copies local state to backend |
| **S3 → GCS** | Change backend config + `terraform init -migrate-state` | Handles copying between backends |
| **Split state (decompose)** | `terraform state mv -state=parent.tfstate -state-out=child.tfstate module.parent.aws_vpc.main aws_vpc.main` | Move resources between states |
| **Merge states** | `terraform state mv -state=child.tfstate aws_instance.web aws_instance.web` | Move into current working state |
| **Rename resource** | `terraform state mv aws_instance.old aws_instance.new` | Safe rename in state |
| **Import resource** | `terraform import aws_s3_bucket.bucket my-bucket-name` | Add existing resource to state |

**Golden rule:** Always back up state files before any surgical operation.

---

## 8. Disaster Recovery

### 8.1 Lost State File

Recover from cloud provider resources:
```bash
# Generate code from existing resources using terracognita or terraformer
terraform import aws_instance.web i-12345
terraform import aws_s3_bucket.data my-bucket
# ... for every resource
```

**Better:** If using S3 versioning, restore the last good version:
```bash
aws s3api get-object --bucket my-bucket --key path/terraform.tfstate   --version-id <last-good-version-id> recovered.tfstate
terraform state push recovered.tfstate
```

### 8.2 State File Corruption

```bash
# 1. Detect: terraform plan fails with JSON parse error
# 2. List versions
aws s3api list-object-versions --bucket my-bucket --prefix path/to/terraform.tfstate
# 3. Restore previous version
aws s3api get-object --bucket my-bucket --key path/to/terraform.tfstate   --version-id <version> tfstate.recovered
# 4. Push recovered state
terraform state push tfstate.recovered
# 5. Verify
terraform plan
```

### 8.3 Corrupted DynamoDB Lock Table

```bash
# Clear specific stale lock
aws dynamodb delete-item --table-name terraform-state-locks   --key '{"LockID": {"S": "my-bucket/path/terraform.tfstate-md5"}}'

# OR: force-unlock (safer as it goes through Terraform's validation)
terraform force-unlock <LOCK_ID>
```

**Prevention:** Set up a scheduled Lambda to purge locks older than 2 hours:
```python
import boto3, time
dynamodb = boto3.client('dynamodb')
for item in dynamodb.scan(TableName='terraform-locks')['Items']:
    created = int(item['Created']['N'])
    if time.time() - created > 7200:  # 2 hours
        dynamodb.delete_item(TableName='terraform-locks',
                             Key={'LockID': item['LockID']})
```

---

## 9. Quick Reference

### Backend Config Templates

```hcl
# === S3 + DynamoDB (recommended) ===
terraform {
  backend "s3" {
    bucket         = "company-terraform-state"
    key            = "env/project/terraform.tfstate"
    region         = "ap-southeast-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}

# === GCS ===
terraform {
  backend "gcs" {
    bucket = "company-terraform-state"
    prefix = "env/project"
  }
}

# === Azure Storage ===
terraform {
  backend "azurerm" {
    resource_group_name  = "terraform-state-rg"
    storage_account_name = "companystate"
    container_name       = "tfstate"
    key                  = "env-project.terraform.tfstate"
  }
}
```

### Essential Commands

| Command | Purpose |
|---------|---------|
| `terraform plan -out=tfplan` | Generate plan file |
| `terraform apply -lock-timeout=10m tfplan` | Apply with lock wait |
| `terraform force-unlock <LOCK_ID>` | Break stale lock |
| `terraform state mv <from> <to>` | Move/rename resources in state |
| `terraform state push <file>` | Replace remote state |
| `terraform state pull > local.tfstate` | Download current state |
| `terraform init -migrate-state` | Migrate to new backend |

### CI/CD Serialization Cheat Sheet

| Platform | Mechanism | Config |
|----------|-----------|--------|
| **GitHub Actions** | `concurrency` group | `concurrency: terraform-apply` |
| **GitLab CI** | `resource_group` | `resource_group: terraform-prod` |
| **AWS CodePipeline** | `executionMode: SINGLE` | Pipeline property |
| **Jenkins** | `lock` step | `lock('terraform-state')` |
| **CircleCI** | custom + S3 lock | Minimal built-in support |

### Golden Rules

```
1. Remote state + locking backend → ALWAYS
2. State versioning → ALWAYS (S3/GCS versioning)
3. Serialize applies → ALWAYS (concurrency groups)
4. Plan + apply in same run → ALWAYS (prevent drift)
5. Force-unlock only after checking → ALWAYS
6. No local apply in production → ENFORCE (branch rules + Atlantis)
7. Backup state before surgery → ALWAYS
8. Automate lock cleanup → RECOMMENDED (Lambda, lifecycle rules)
```

---

*This guide is based on established Terraform best practices, HashiCorp documentation, and real-world team workflows as of mid-2026. Always refer to the official Terraform documentation for the latest backend configuration options.*
