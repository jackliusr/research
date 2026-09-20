# Cheap GPU Cloud Providers with VM Access — What the GPU-Hour Actually Costs

**Author:** Jack Liu Shurui — Solution Architect
**Last Updated:** September 2026
**Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
**Rates and access-model claims checked:** 20 September 2026
**Scope note:** every GPU rental rate here is a point-in-time reading from the provider's own pricing page on the date shown. This is the fastest-repricing corner of cloud: neoclouds have repriced H100 capacity by 30% or more inside a single quarter, spot markets move hourly, and a marketplace's "price" is a live order book rather than a tariff. Treat every figure below as an observation with a timestamp, not a quotation. Where a provider's rate or access model could not be read from its own documentation, it is marked ⚠ rather than estimated.

---

## Table of Contents

1. [Overview, Thesis and Decoder](#1-overview-thesis-and-decoder)
2. [The Access-Model Axis](#2-the-access-model-axis)
3. [The Cost Model Decomposed](#3-the-cost-model-decomposed)
4. [The Provider Line-Up](#4-the-provider-line-up)
5. [The Availability Problem](#5-the-availability-problem)
6. [The Security, Tenancy and Compliance Question](#6-the-security-tenancy-and-compliance-question)
7. [The Performance-Per-Dollar Reality](#7-the-performance-per-dollar-reality)
8. [The Usage-Pattern Fit](#8-the-usage-pattern-fit)
9. [Rent Versus Own](#9-rent-versus-own)
10. [The Regulated-Enterprise and Banking Angle](#10-the-regulated-enterprise-and-banking-angle)
11. [The Cymbal Bank Worked Example](#11-the-cymbal-bank-worked-example)
12. [The Anti-Patterns](#12-the-anti-patterns)
13. [The Claims Audit](#13-the-claims-audit)
14. [What Could Not Be Verified](#14-what-could-not-be-verified)
15. [Glossary, Cross-References and Closing Summary](#15-glossary-cross-references-and-closing-summary)

---

## 1. Overview, Thesis and Decoder

### 1.1 The thesis

**The headline price per GPU-hour is the least informative number on the page.** The real cost of renting a GPU is the rate multiplied by the hours you actually pay for — not the hours you use — plus the hidden lines (persistent storage while idle, egress, images, snapshots, reserved-capacity minimums), and the binding constraint on a real project is usually **availability, not price**. A provider can publish the cheapest H100 rate in the market and still be the wrong choice, because on the Tuesday you needed sixteen of them there were four, or because what you rented was a container and you needed a kernel module.

This is the whole reason the guide carries a VM-access qualifier in its title. "Cheap GPU cloud" is a category that today contains at least four structurally different products sharing one marketing word — *GPU*. Only one of them is a virtual machine you control. The price-marketed end of the market is disproportionately the container end.

### 1.2 What this guide is not

This is not a ranking, and it does not crown a winner. The output is three things:

* a **cost model** (§3) that decomposes what you are actually billed;
* an **access-model matrix** (§2) that says what each provider gives you and withholds;
* a **workload-keyed framework** (§8) that maps a workload class to an access model, a pricing tier and a provider class — with the reasoning exposed so you can disagree with it.

The correct answer changes with the access requirement, the utilisation, the residency constraint and the availability you can actually obtain on the day. Two of those four are not commercial, which is why a pure price table is unusable.

### 1.3 Boundary statement — what this guide owns and what it does not

This guide owns **the cost of renting GPU capacity and the access model you get for that money**. Several adjacent topics are covered better elsewhere in this repository, and this guide cross-references them rather than re-deriving them:

| Adjacent topic | Owning guide | Relationship |
|---|---|---|
| The NVIDIA learning pathway (what to learn, in what order) | `technology/gain_nvidia_experience.md` | That guide owns the curriculum. **Its stored GPU rates are stale and this guide supersedes them.** Use this guide's dated rates; use that guide's learning sequence. |
| Hyperscaler-by-hyperscaler comparison (AWS, Azure, GCP, OCI general compute) | `technology/cloud_providers_guide.md` | That guide owns the hyperscaler comparison. §4 here treats hyperscalers only briefly, as one provider class among several. |
| Performance technique (kernels, throughput, parallelism tuning) | `technology/gpu_optimization_guide.md` | That guide owns how to make a GPU go faster. This guide only prices the GPU it runs on. |
| GPU sharing and partitioning (MIG, time-slicing, HAMi) | `technology/hami_gpu_sharing_guide.md` | That guide owns how one GPU is shared between tenants. §2 here notes when a provider sells you a shared slice. |
| GPU observability and telemetry (DCGM, exporters, dashboards) | `technology/nvidia_dcgm_guide.md` | That guide owns measurement. §7 here only asserts that a host can starve a GPU, not how to instrument it. |
| Self-hosting models on your own hardware | `technology/on_prem_llm_deployment_guide.md` | That guide owns on-premises deployment. §9 here does the rent-versus-own arithmetic at the boundary. |
| Deployment architecture for AI/LLM systems | `technology/ai_llm/scalable_ai_deployment.md` | That guide owns the architecture. This guide owns the bill for the compute under it. |
| Object storage cost, egress and exit | `technology/cloud_object_storage_comparison_guide.md` | That guide owns storage cost in depth. §3 here prices only the storage lines a GPU tenant is forced to pay. |

If your question is "how do I make the model faster", you are in the wrong guide. If your question is "what will this cost me, and can I even get it", read on.

### 1.4 Decoder — the terms that decide the answer

| Term | What it actually means | Why it bites |
|---|---|---|
| **GPU-hour** | One accelerator rented for one hour. Billed per GPU, *not* per instance, on the neoclouds — a "8×H100 node at $49.24/hr" is $6.155 per GPU-hour. | Comparing a node price against a per-GPU price inflates one side by 8×. CoreWeave's own card prints a derived "Inference Single GPU" column precisely because buyers keep doing this. |
| **On-demand** | Capacity allocated to you now, cancellable now, at the highest rate. | On-demand is the *least* available tier. Nothing is reserved for you, so the rate you see is the rate you may not be able to get. |
| **Reserved / committed** | A term commitment (1 month to 3+ years) at a discount, with capacity held for you. | Commits payment before utilisation is known. Nebius advertises up to 35% off on-demand; Lambda takes committed clusters as low as $5.54/GPU-hr against a $4.29 on-demand 1× rate. Discounts are real; so is the minimum. |
| **Spot / interruptible / preemptible** | You bid for spare capacity at a large discount and accept that the provider may reclaim it, often on a short notice window. | Verified on CoreWeave's own card: an 8×H100 node is $49.24/hr on-demand and $19.71/hr spot — about 60% off. The discount is the compensation for the reclaim right, not a pricing mistake. |
| **Preemption** | The mechanism by which a spot instance is taken back. | Preemption is fine for a checkpointed job and fatal for a stateful service. The rate is not the risk; the restartability of your workload is. |
| **Access model** | *How* you are given the GPU: true VM, restricted container, managed notebook, or serverless endpoint. | The discriminator of this entire guide. See §2. |
| **Bare metal** | A whole physical server, no hypervisor between you and the silicon. | The most control and the least elasticity; you pay for the box whether or not the GPU is busy. |
| **Neocloud** | A GPU-first provider (CoreWeave, Lambda, Nebius, Crusoe, Together) that sells clusters rather than a broad service catalogue. | Better price-per-GPU-hour, thinner everything else: fewer regions, fewer compliance artefacts, less integration surface. |
| **Marketplace** | An aggregator listing capacity owned by many third-party hosts (Vast.ai, RunPod's community tier, TensorDock, Salad). | Cheapest headline rates; the tenancy model is the weakest, and the host is frequently a stranger. See §6. |
| **Persistent volume** | A disk that survives the instance being stopped or destroyed. | It is billed while the GPU is not. RunPod publishes a higher idle rate than running rate for volume disk, which makes the point sharply. |
| **Egress** | Data leaving the provider's network. | Free at Lambda and (per its own pricing/FAQ) at CoreWeave; metered at Nebius for object-storage egress ($0.015/GiB). A free-egress provider can be cheaper overall on a dataset-heavy job at a higher GPU rate. |
| **Quota** | A cap on how much of a resource you may allocate, imposed by the provider (or by your own procurement). | Quota is a *product* constraint that looks like a *billing* constraint. Getting it raised takes days. |
| **Instance family** | The hardware configuration behind a rate: GPU model, VRAM, vCPU count, RAM, local NVMe, interconnect. | Two instances at the same GPU rate are not the same product. This is the "cheap GPU with a slow disk" trap in §7. |
| **MIG** | Multi-Instance GPU: one physical GPU partitioned into isolated slices, each sold or shared as its own device. | MIG is how a provider sells you "48GB" at $1.09/hr (RunPod lists Pro 6000 MIG 48GB and MIG 24GB tiers). You get VRAM and compute fractions, not the full card, and you do not get the interconnect. |

### 1.5 How to read the figures in this guide

Every rate below is followed by its source page and the date the page was read. Where a provider bills a whole node, the per-GPU figure is shown as arithmetic on the provider's node price and labelled as such. Where a provider advertises a discount structure without a public number, the number is marked ⚠ and no figure is invented.

---

## 2. The Access-Model Axis

This is the section the entire guide exists to serve. If you take one thing away, take this: **the word "GPU" in a price list does not tell you what you are renting.** Four structurally different products are sold under it, and the cheap end of the market skews to the restrictive end.

### 2.1 The four access models, defined precisely

**(a) True virtual machine.** You get a bootable VM with root, an SSH endpoint, a block device, and the right to install your own kernel modules and drivers. The hypervisor (or a full-metal allocation) sits between you and other tenants; the GPU is passed through (VFIO/PCI passthrough or SR-IOV). On this model you can `modprobe`, install a custom NVIDIA driver, run Docker/containerd yourself, run Kubernetes yourself, mount your own filesystem, and set your own scheduler.

**(b) Container with a restricted runtime.** You get a container image running inside the provider's orchestration. You typically get SSH or a web terminal, and often a persistent volume, but the kernel is the *host's* kernel, shared with other tenants. You cannot load a kernel module, cannot change the driver, cannot run privileged/`--cap-add` workloads, and cannot bring your own scheduler. This is the single most common shape in the cheap tier.

**(c) Managed notebook / jobs workspace.** A Jupyter-style environment on pre-baked images. Convenient for exploration, structurally incapable of being a production endpoint: no SSH you control, no root, no custom kernel, no scheduler of your own.

**(d) Serverless endpoint.** You deploy a handler or a container with a cold-start contract and are billed by the second for workers. The strongest "no infrastructure" story and the weakest control story: no host access at all, no persistent kernel state, no multi-node, and pricing is per worker-hour of the provider's choosing.

The practical test is not "can I run `nvidia-smi`" — everything passes that. The test is:

1. Can I load a kernel module (`insmod`/`modprobe`)?
2. Can I replace the NVIDIA driver version without asking the provider?
3. Can I run my own container runtime or Kubernetes, and schedule my own pods?
4. Is there a volume that survives instance termination?
5. Can I join more than one node into a single job with a fast interconnect?

A true VM answers five of five. A restricted container answers at most two. A notebook answers one. A serverless endpoint answers none of them and is not supposed to.

### 2.2 What each model gives and withholds

| Capability | (a) True VM | (b) Restricted container | (c) Managed notebook | (d) Serverless endpoint |
|---|---|---|---|---|
| Root / SSH you control | Yes | Partial (shell, no root) | No | No |
| Custom kernel / kernel modules | Yes | **No** | No | No |
| Own NVIDIA driver version | Yes | **No** | No | No |
| Own container runtime / Kubernetes | Yes | **No** (nested) | No | No |
| Own scheduler (Slurm, K8s, Ray head) | Yes | No | No | No |
| Persistent volume | Yes | Usually | Usually | Provider-managed only |
| Multi-node cluster with fast interconnect | Yes (where offered) | Limited | No | No |
| Billed when you are not using it | Yes | Yes (volume + idle) | Yes (workspace) | No (scale to zero) |
| Fastest time-to-first-GPU | Minutes | Minutes | Seconds | Minutes |

### 2.3 Verified per-provider access-model matrix

Read from each provider's own pricing page and documentation on **20 September 2026**. "Root/SSH" means the provider documents that you get a shell you control on a machine-level instance; "custom kernel+drivers" means the provider's own docs describe VM/root access such that installing or replacing the driver is possible. Where a provider's page is silent, the cell is ⚠ rather than a guess.

| Provider | Primary access model | Root/SSH | Custom kernel + drivers | Persistent volume | Multi-node | Source URL | Date checked |
|---|---|---|---|---|---|---|---|
| Lambda | True VM (instance) + managed clusters | Yes | Yes | Yes (attached SSD, plus filesystem) | Yes (1-Click Clusters, 16–2,000+ GPU) | https://lambda.ai/pricing | 20 Sep 2026 |
| CoreWeave | True VM / bare-metal-backed instances, K8s-native | Yes | Yes | Yes (block + shared filesystem) | Yes (NVL72, HGX 8-GPU nodes; K8s) | https://www.coreweave.com/pricing | 20 Sep 2026 |
| Nebius | True VM (GPU instances) + managed K8s/Slurm | Yes | Yes | Yes (block, shared, WEKA) | Yes (Managed Kubernetes, Soperator/Slurm) | https://nebius.com/prices | 20 Sep 2026 |
| RunPod — Pods | Container on dedicated GPU instance (some templates); SSH available | Partial | ⚠ No (host kernel) | Yes (volume disk, network storage) | Yes (Clusters product) | https://www.runpod.io/pricing | 20 Sep 2026 |
| RunPod — Serverless | Serverless endpoint (workers) | No | No | Provider-managed | No | https://www.runpod.io/pricing | 20 Sep 2026 |
| Vast.ai | Marketplace: per-host, mostly container-oriented | Partial (host-dependent) | ⚠ Host-dependent | Yes (per-instance storage) | Limited (reserved/cluster offerings) | https://vast.ai/pricing | 20 Sep 2026 |

This matrix is deliberately short because §4 fills it out provider by provider. The important observation for the *access* question is already visible in it: **the providers that market hardest on price (marketplaces, serverless) are the ones where the answer to "root, kernel module, my own driver" is "no" or "depends on the host".**

### 2.4 Consequences per workload class

| Workload class | Minimum access model | Why |
|---|---|---|
| Driver/kernel development, GPU operator work, custom MIG configuration | True VM (a) | Needs `modprobe`, driver swaps, privileged containers. Container-only providers are structurally disqualified. |
| Multi-node distributed training | True VM (a) with fast fabric | Needs a shared scheduler and RDMA-class interconnect; serverless and notebooks cannot form the cluster. |
| Single-node fine-tune on pre-baked images | Container (b) is sufficient | No kernel work, no custom driver; a persistent volume for checkpoints is the only requirement. |
| Interactive exploration / notebook demos | Notebook (c) | Convenience beats control; nothing to schedule. |
| Steady, spiky inference API | Serverless (d) *or* true VM with autoscaling | Serverless wins if load is bursty and latency-tolerant; a VM wins if you need a pinned driver, a warm cache, or predictable tail latency. |
| Batch evaluation over a fixed dataset | Container (b) or true VM (a), interruptible tier | Restartable by construction, so the spot discount is nearly free money. |

### 2.5 The honest summary of this section

A provider that leads with price is usually selling you model (b), (c) or (d). That is not a criticism — for a large fraction of real workloads a container or an endpoint is the right product and the VM would be over-engineering. But it means the question "which is the cheapest GPU cloud with VM access" cannot be answered by sorting a price column, because a meaningful share of the cheap column is not VM access at all. Filter on §2.2's five tests first; price the survivors second.

---

## 3. The Cost Model Decomposed

### 3.1 The line items

A GPU bill is not one number. It is at least these, and most published comparisons price only the first.

| Line | Billed per | Notes and verified examples |
|---|---|---|
| Compute (the GPU-hour) | GPU-hour or node-hour, at a billing granularity | See 3.2. |
| Persistent volume / volume disk | GB-month | Billed while the instance is **stopped or idle**. RunPod's card lists volume disk at $0.10/GB/mo running and $0.20/GB/mo idle (checked 20 Sep 2026) — idleness costs *more* per GB than running. |
| Container/instance disk | GB-month | RunPod lists container disk at $0.10/GB/mo (20 Sep 2026). |
| Network / shared filesystem | GiB-month | Nebius lists a shared filesystem at $0.08/GiB/mo and a WEKA filesystem at $0.10/GiB/mo (20 Sep 2026). |
| Object storage | GiB-month + egress | Nebius lists object storage Standard at $0.0147/GiB/mo with egress at $0.015/GiB (20 Sep 2026). |
| Egress | GiB transferred out | Free at Lambda ("no egress fees" on its pricing page, 20 Sep 2026) and free for Nebius compute networking, but metered on Nebius object-storage egress. Cross-ref `technology/cloud_object_storage_comparison_guide.md` for the storage-side treatment. |
| Image / registry | GB-month | Frequently bundled, occasionally a separate line; check before assuming. |
| Snapshot | GB-month | The classic forgotten line. |
| Public IP | per IP-hour | Nebius lists public IPs as free (20 Sep 2026); others do not. |
| Support / SLA tier | % of spend | Enterprise support is usually a percentage uplift, not a fixed fee. |

### 3.2 Billing granularity — the quiet ranking-shifter

Billing granularity converts an identical rate into different bills:

* **Per-second** — Vast.ai states per-second billing with "no minimum hours, no rounding up" and RunPod's pods and clusters are per-second billed (both checked 20 Sep 2026). For a job that runs 11 minutes, you pay for 11 minutes.
* **Per-minute** — rounds a 61-second job up to two minutes. Immaterial for long jobs, material for evaluation sweeps that launch thousands of short-lived pods.
* **Hourly minimum** — rounds an 11-minute job up to a full hour. A 6× multiplier on the same underlying rate. If a marketing table shows a provider "cheaper per GPU-hour" with an hourly minimum against a per-second competitor, the ranking can invert on any job shorter than an hour.

The rule: **compare effective cost per job, not cost per GPU-hour.** For long training runs the two coincide; for the burst and evaluation workloads in §3.4 they do not.

### 3.3 Pricing tiers

| Tier | Mechanism | Verified example (all 20 Sep 2026) |
|---|---|---|
| On-demand | Immediate, cancellable, highest rate, no capacity guarantee | CoreWeave 8×H100 node $49.24/hr on-demand |
| Reserved / committed | Term commitment, discounted, capacity held | Nebius advertises up to 35% off on-demand for multi-month reservations; Lambda committed clusters are lower per GPU-hour than 1× on-demand ($6.16/GPU-hr at 16 GPUs vs $4.29 on-demand 1×) |
| Spot / interruptible / preemptible | Discount in exchange for the provider's right to reclaim | CoreWeave 8×H100 spot $19.71/hr vs $49.24 on-demand — a 60% discount; Nebius HGX H100 preemptible $2.15 vs $4.50 on-demand per GPU-hour; Vast.ai states its interruptible tier is "50%+ cheaper" |
| Serverless / per-worker | Billed by worker-second with scale-to-zero | RunPod serverless H100 listed at $4.79/hr per worker |

The spot discount is real and large. It is also a **contract term you must be able to honour**: preemption does not care that your job was at epoch 900 of 1000 with no checkpoint.

### 3.4 Worked arithmetic — three workloads

**All figures below are illustrative arithmetic on rates checked on 20 September 2026.** They are not quotes and they are not predictions of your bill. The purpose is to show *where the ranking changes* once hidden lines enter.

#### Workload A — multi-day fine-tune

Profile: 8×H100 for 72 hours (3 days), on a dataset that must be moved in (~500 GiB) and a checkpoint set moved out (~100 GiB per checkpoint, 5 checkpoints).

| Provider / tier | Compute arithmetic | Storage + transfer lines | Illustrative total |
|---|---|---|---|
| Lambda, on-demand 8×H100 | 8 GPUs × $3.99/GPU-hr × 72 h = **$2,298.24** | Egress free; 22 TiB local SSD included; no bucket metered (Lambda card, 20 Sep 2026) | **≈ $2,298** |
| CoreWeave, on-demand 8×H100 node | $49.24/node-hr × 72 h = **$3,545.28** | No egress/NAT/intra-region charge claimed; storage extra if you keep data beyond the node | **≈ $3,545 + storage** |
| CoreWeave, spot 8×H100 node | $19.71/node-hr × 72 h = **$1,419.12** | Same, **but** assume at least one preemption/restart per day | **≈ $1,419 + rework** |
| Nebius, on-demand HGX H100 | 8 × $3.85/GPU-hr × 72 h = **$2,217.60** | Object egress $0.015/GiB × 500 = $7.50 in + checkpoints out; shared FS $0.08/GiB/mo | **≈ $2,225 + FS** |
| RunPod pod, 8 × H100 SXM | 8 × $3.49/hr × 72 h = **$2,010.24** | Volume disk $0.10/GB/mo; container disk $0.10/GB/mo | **≈ $2,010 + disk** |

Where the ranking changes: spot and per-second marketplaces fall below every on-demand rate, and two providers with free egress (Lambda, CoreWeave) beat a nominally cheaper GPU rate once a large dataset is moved in and checkpoints are moved out repeatedly.

#### Workload B — continuous small inference service

Profile: one H100-class GPU held 24/7 for 730 hours in a month; model weights refreshed weekly (~50 GiB per refresh); volume 200 GB held permanently.

| Provider / tier | Compute arithmetic | Hidden lines | Illustrative monthly total |
|---|---|---|---|
| Nebius H100 on-demand | $3.85 × 730 = **$2,810.50** | 200 GB block volume at $0.053/GiB/mo ≈ $11.20; weight egress $0.015/GiB × 200 ≈ $3.00 | **≈ $2,825** |
| CoreWeave H100 (per-GPU inference line $6.16/hr) | $6.16 × 730 = **$4,496.80** | Storage extra | **≈ $4,497 + storage** |
| Lambda H100 SXM 1 GPU at 1× rate | $4.29 × 730 = **$3,131.70** | Egress free; SSD included | **≈ $3,132** |
| RunPod serverless H100 worker | $4.79/hr × 730 = **$3,496.70** *if pinned* | Scale-to-zero changes this entirely — a spiky service pays only for active workers | **highly load-dependent** |

Where the ranking changes: a genuinely bursty service makes the serverless column collapse (you pay for a fraction of 730 hours), while a flat one makes serverless the most expensive per hour on the page. **Utilisation, not rate, decides.** That is the thesis of §1 in arithmetic form.

#### Workload C — burst evaluation batch

Profile: 4×A100-80GB for 40 minutes, run twice a week (≈ 5.33 hours/month of GPU time). This is where billing granularity does the work.

| Provider / tier | Billing granularity | Arithmetic | Illustrative monthly total |
|---|---|---|---|
| Lambda A100 80GB | Hourly-equivalent on-demand | 4 × $2.79 × 5.33 h = **$59.47** (assumes no hourly-minimum rounding penalty) | **≈ $59** |
| CoreWeave A100 node | Hourly node billing | $21.60/node-hr × 5.33 h = **$115.13** | **≈ $115** |
| Nebius HGX H100 preemptible (A100-class unavailable; substitute smaller card) | Per-second, preemptible | 4 × $2.15 × 5.33 h = **$45.84**, restarts plausible | **≈ $46 + rework** |
| RunPod A100 SXM pod | Per-second | 4 × $1.59 × 5.33 h = **$33.90** | **≈ $34** |
| Vast.ai interruptible | Per-second, live market | Host-set rate; ⚠ live price not fixed on the pricing page — the page presents an order book, not a tariff | **⚠ host-dependent** |

Where the ranking changes: the per-second providers win a short burst by construction; an hourly-minimum provider with a lower headline rate loses on any job under an hour. And the whole column is small — a monthly evaluation batch costs less than one engineer-hour, which is the argument for not spending two weeks choosing a provider for it.

### 3.5 The worked-arithmetic rule

Run the arithmetic with storage, egress and idleness *included*, at your utilisation, at your granularity. Then check availability (§5) before you commit to anything, because a rate you cannot obtain is not a rate.

---

## 4. The Provider Line-Up

Organised by provider *class*, because class predicts most of what a buyer cares about: the access model, the certification set, the availability pattern and the failure mode. Rates below are the ones this guide could read from the provider's own page on **20 September 2026**. Anything unreadable is ⚠ — never estimated.

A reminder of the house rule for this market: **rates in this table are a photograph, not a price list.** If you are reading this in the future, re-read the source URLs.

### 4.1 Hyperscalers (brief — cross-ref the cloud providers guide)

AWS, Azure, Google Cloud and OCI all sell GPU capacity, and all of them sell it as true VM instances (model (a)) inside a broader catalogue. `technology/cloud_providers_guide.md` owns that comparison; this guide does not re-derive it. Three observations that are specific to the *GPU* question:

1. **The hyperscaler premium buys integration, not silicon.** A hyperscaler GPU instance is the same H100 or B200 as a neocloud's. What differs is the surrounding estate: IAM, VPC, managed Kubernetes, existing enterprise agreements, support, audit evidence, regional footprint, and the procurement relationship a regulated buyer already has.
2. **Hyperscaler GPU capacity is quota-gated.** You cannot simply order a hundred GPUs; you request a quota increase and wait. That wait is the single most common reason projects that "have the budget" slip a quarter.
3. **Hyperscaler list prices for high-end accelerators are the highest in this guide's table**, which is precisely why the neocloud tier exists. Whether the premium is worth paying is a §8 question, not a price question.

### 4.2 Neoclouds

GPU-first providers that sell clusters rather than a broad service catalogue. Stronger access model (true VMs), better price-per-GPU-hour, thinner compliance surface.

#### CoreWeave

* **Access model:** True VM / bare-metal-backed instances, Kubernetes-native; 8-GPU HGX nodes and GB200/GB300 NVL72 systems. Root/SSH documented; multi-node is the point of the product.
* **GPU catalogue and rates (checked 20 Sep 2026, [coreweave.com/pricing](https://www.coreweave.com/pricing))** — the card prices *whole nodes*, so the per-GPU figure is arithmetic on the node price and is labelled as such:

| Instance | On-demand (node-hr) | Spot (node-hr) | Derived per-GPU-hr |
|---|---|---|---|
| NVIDIA HGX H100 (8 GPU) | $49.24 | $19.71 | ≈ $6.16 |
| NVIDIA HGX H200 (8 GPU) | $50.44 | $20.93 | ≈ $6.31 |
| NVIDIA HGX B200 (8 GPU) | $68.80 | $34.11 | ≈ $8.60 |
| NVIDIA A100 (8 GPU) | $21.60 | $9.65 | ≈ $2.70 |
| NVIDIA L40S (8 GPU) | $18.00 | $7.88 | ≈ $2.25 |
| NVIDIA L40 (8 GPU) | $10.00 | $6.27 | ≈ $1.25 |
| NVIDIA GB200 NVL72 (41 superchips) | $42.00 | N/A | — (§1.4: 2 superchips/node, 4 GPUs total per listed unit) |
| NVIDIA GH200 (1 GPU) | $6.50 | N/A | $6.50 |

* **Availability reality:** the spot column exists only for some SKUs (GB200 NVL72 spot is listed N/A); B300 and RTX PRO 6000 standard-memory rows show "Contact sales" rather than a price, i.e. those are negotiated, not self-serve.
* **Egress:** CoreWeave states there are no charges for egress, NAT, or intra-region transfer on its cost-calculator copy (checked 20 Sep 2026). Treat as a provider claim with a source, not an audited fact.
* **SLA / compliance:** CoreWeave publishes for Gartner MQ recognition and enterprise terms; enterprise-grade claims should be read from its trust/security pages, not assumed here. ⚠ Specific certification list not verified on the date checked.
* **Honest weaknesses:** node-granular pricing means a single-GPU experiment is inefficient; the "Inference Single GPU" column is explicitly restricted to CoreWeave inference-platform customers (its own footnote 2), so you cannot freely buy a 1/8th slice.
* **Residency footprint:** North America and Europe tables published separately; the same SKUs are priced identically across both regions on the date checked.

#### Lambda

* **Access model:** True VM instances with self-serve, first-come access; plus "1-Click Clusters" for multi-node. VM with root, attached local SSD, and filesystem storage.
* **Rates (checked 20 Sep 2026, [lambda.ai/pricing](https://lambda.ai/pricing))** — per GPU-hour, 8× configuration:

| GPU | VRAM/GPU | 8× price/GPU-hr | 4× / 2× / 1× price/GPU-hr |
|---|---|---|---|
| NVIDIA B200 SXM6 | 180 GB | $6.69 | $6.79 / $6.89 / $6.99 |
| NVIDIA H100 SXM | 80 GB | $3.99 | $4.09 / $4.19 / $4.29 |
| NVIDIA A100 SXM | 80 GB | $2.79 | — |
| NVIDIA A100 40GB (PCIe) | 40 GB | $1.99 | $1.99 / $1.99 / $1.99 |
| NVIDIA A6000 | 48 GB | — | $1.09 / $1.09 / $1.09 |
| NVIDIA A10 | 24 GB | — | — / — / $1.29 |
| NVIDIA GH200 | 96 GB | — | — / — / $2.29 |
| NVIDIA Tesla V100 | 16 GB | $0.79 | — |

* **Cluster tier:** 1-Click Clusters from 16 to 2,000+ GPUs; H100 at $6.16/GPU-hr (16 GPU), $5.85 (64), $5.54 (256); B200 at $9.86 (16), $9.36 (64), $8.87 (256+). Note the pattern: **committed cluster capacity is priced above the 1× on-demand self-serve rate per GPU-hour** at small sizes, because it is reserved availability with a term attached, not a discount off the hourly rate. Do not read the cluster number as a discount.
* **Egress:** "No egress fees" stated on the pricing page (checked 20 Sep 2026).
* **Availability reality:** self-serve access is explicitly labelled "first-come", i.e. there is no capacity guarantee behind the on-demand price.
* **Honest weaknesses:** ⚠ published SLA terms and certification list were not read from Lambda's own legal pages on the date checked. Lambda is a well-known NVIDIA partner cloud, but "known NVIDIA partner" is not a SOC 2 report.

#### Nebius

* **Access model:** True VM GPU instances plus Managed Kubernetes and Soperator (Slurm on Kubernetes), both listed as free of charge on the pricing page — a genuine differentiator for teams that want a scheduler without building one.
* **Rates (checked 20 Sep 2026, [nebius.com/prices](https://nebius.com/prices))**, per GPU-hour:

| Instance | Preemptible | On-demand |
|---|---|---|
| NVIDIA HGX B300 | $4.30 | $7.85 |
| NVIDIA HGX B200 | $3.95 | $7.15 |
| NVIDIA HGX H200 | $2.45 | $4.50 |
| NVIDIA HGX H100 | $2.15 | $3.85 |
| NVIDIA RTX PRO 6000 | $0.95 | $1.80 |
| NVIDIA L40S (Intel CPU) | from $0.90 | from $1.82 |
| NVIDIA L40S (AMD CPU) | from $0.74 | from $1.55 |
| NVIDIA GB300 NVL72 | — | Contact us |
| NVIDIA GB200 NVL72 | — | Contact us |

* **Storage and network lines (same source, same date):** shared filesystem $0.0800/GiB/mo; WEKA filesystem $0.1000/GiB/mo; object storage Standard $0.0147/GiB/mo with egress $0.0150/GiB; block volumes $0.0530 (no replication) / $0.0710 (erasure coding) / $0.1180 (3× mirroring) per GiB/mo; local SSD $0.065/GiB/mo; **networking egress and ingress free; public IP free**.
* **Commitment:** "up to 35% less than on-demand" for reserving large clusters for multiple months (provider claim, same page).
* **Honest weaknesses:** the headline H100 on-demand rate ($3.85/GPU-hr) is competitive but the L40S rows say "from", i.e. configuration-dependent; and the region footprint is narrower than a hyperscaler's.

#### Crusoe

* **Access model:** GPU instances with a stated "lightweight virtualization" design, managed Kubernetes at $0.10/cluster-hour, spot + on-demand + reserved.
* **Rates (checked 20 Sep 2026, [crusoe.ai/cloud/pricing](https://www.crusoe.ai/cloud/pricing))**, per GPU-hour on-demand: H200 (141GB HGX) **$4.29**; H100 (80GB HGX) **$3.90**; A100 80GB SXM **$2.30**; A100 80GB PCIe **$2.00**; L40S (48GB) **$1.50**; AMD MI300X (192GB) **$3.45**. GB200 NVL72, B200 HGX and AMD MI355X are all "Contact sales" — no public rate.
* **Spot:** the "Current spot" column exists on the page but every readable row resolves to "Contact sales" — so ⚠ **Crusoe's spot discount could not be read as a number on the date checked.** Do not assume the CoreWeave discount ratio transfers.
* **Storage lines:** persistent disks $0.08/GiB/mo; shared disks $0.07/GiB/mo; container registry $0.10/GiB/mo; object storage $0.06/GiB/mo. CPU: $0.04/vCPU-hr general-purpose, $0.09/vCPU-hr storage-optimized.
* **Managed inference add-on:** self-serve dedicated deployments list H100 at $5.50/hr and H200 at $6.00/hr — **higher than the same GPUs as raw instances ($3.90 / $4.29)**, which is the price of the managed endpoint layer, stated explicitly rather than buried.
* **Honest weaknesses:** the spot column being contact-sales-only means the discount structure is unverifiable from public pages; a buyer cannot pre-plan a spot strategy from the website.

#### Together AI

* **Access model:** a model-platform provider first (serverless inference, provisioned throughput, dedicated inference, fine-tuning) with GPU Clusters and a Sandbox as the compute products. The compute tier is the newer surface; the platform is the core.
* **Rates:** ⚠ **Together's GPU cluster per-hour rates were not readable from the pricing page as fetched on 20 Sep 2026** — the page's GPU Clusters section did not render its rate table, and its announcement feed mentions "on-demand B200s now available on Together GPU Clusters" without a published price. Together's *inference* pricing is public and per-token (e.g. its own card shows Llama 3.3 70B at $1.04 per 1M tokens), which is the part of the product this guide would route around anyway (§9, managed inference).
* **Honest weakness:** for a buyer whose question is "what is the VM-hour", Together is currently unpriceable from public pages — which for a procurement record is itself a finding, not a gap to paper over.

#### Other neoclouds surfaced by the research

* **DataCrunch / Verda (Finland-based, EU)** — true VM GPU instances, NVLink models available, instant clusters with InfiniBand up to 144 GPUs self-service, serverless containers with scale-to-zero. Rates (checked 20 Sep 2026, [datacrunch.io/pricing](https://datacrunch.io/pricing)), per hour on-demand / spot:

| GPU | On-demand | Spot |
|---|---|---|
| GB300 SXM6 288GB | $9.06 | $4.53 |
| B300 SXM6 268GB | $7.88 | $3.94 |
| B200 SXM6 180GB | $6.49 | $3.24 |
| H200 SXM5 141GB | $4.37 | $2.19 |
| H100 SXM5 80GB | $3.35 | $1.67 |
| A100 SXM4 80GB | $1.74 | $0.8675 |
| A100 SXM4 40GB | $1.26 | $0.6320 |
| RTX PRO 6000 96GB | $1.86 | $0.9300 |
| L40S 48GB | $1.45 | $0.7265 |
| RTX A6000 48GB | $0.6040 | $0.3020 |
| Tesla V100 16GB | $0.1760 | $0.0880 |

  Reserved discounts published as a schedule rather than a quote: 1 month 2% off, 3 months 3%, 6 months 4%, 1 year 8%, 2 years 25% (same source, same date). Storage: NVMe and shared NVMe $0.2000/GiB/mo, container registry $0.2000/GiB/mo. **"SOC 2 and GDPR compliant" is stated by the provider on its own pricing page** — that is the claim, at that date; the trust centre at trust.verda.com is where the report would be read. Serverless containers list H100 at $3.68/hr on-demand, *above* the $3.35 instance rate — the managed layer premium again, stated openly.
* **Hyperstack (NexGen Cloud, UK-headquartered, UK/EU data)** — true VM GPU instances, **billed per minute**, egress and ingress free, Kubernetes master node free, public IP $0.00672043/hr, SSV storage $0.000096774/GB/hr. Rates (checked 20 Sep 2026, [hyperstack.cloud/gpu-pricing](https://www.hyperstack.cloud/gpu-pricing)), per GPU-hour:

| GPU | On-demand | Reservation from | Spot VM |
|---|---|---|---|
| NVIDIA H200 SXM | $3.99 | $2.79 | — |
| NVIDIA H100 SXM | $3.20 | $2.72 | — |
| NVIDIA H100 NVLink | $2.60 | $1.82 | — |
| NVIDIA H100 PCIe | $2.50 | $1.75 | $2.00 |
| NVIDIA B200 | $6.00 | $5.10 | — |
| NVIDIA B300 | $7.40 | — | — |
| NVIDIA RTX PRO 6000 SE | $1.85 | $1.30 | $1.48 |
| NVIDIA A100 SXM | $1.60 | $1.36 | — |
| NVIDIA A100 NVLink | $1.40 | $0.98 | — |
| NVIDIA A100 PCIe | $1.35 | $0.95 | $1.08 |
| NVIDIA L40 | $1.00 | $0.70 | $0.80 |
| NVIDIA A6000 | $0.50 | $0.35 | $0.40 |
| NVIDIA A4000 | $0.15 | $0.11 | — |

  Hyperstack displays a **SOC 2 Type 2** mark on its own pricing page (checked 20 Sep 2026). It also carries a fraud warning about an impersonating domain — a useful reminder to verify the vendor domain before entering payment details for *any* GPU provider.
* **Voltage Park, Fluidstack, Genesis Cloud** — all named in the brief as candidates. ⚠ **Rates and access models for these three were not readable from their own pages on 20 September 2026** (Genesis Cloud's pricing page failed to render; the others were not verifiable within the research budget). They are listed here as *unverified candidates*, which is exactly what §14 exists for. Do not cite a voltage-park or fluidstack rate from this guide.
* **Scaleway (France, EU)** — true VM GPU instances in PAR-1/PAR-2 with block storage and scratch NVMe. Readable rates (checked 20 Sep 2026, [scaleway.com/en/pricing/gpu](https://www.scaleway.com/en/pricing/gpu/)): L4-1-24G **€0.79/hr**, L4-2-24G €1.58/hr, L4-4-24G €3.15/hr, L4-8-24G €6.30/hr (prices before tax). ⚠ **The L40S, H100 and B300-SXM instance families render as "not available in this zone — available in PAR-2" from the default zone view, so their hourly rates could not be read on the date checked.** That is a real finding: the interesting GPUs are zone-restricted, and the price depends on which zone you can get.

### 4.3 Marketplaces

Aggregators listing capacity owned by third parties. Structurally the cheapest end of the market and structurally the weakest on tenancy, guarantees and paper. The access model is where they must be read most carefully.

#### Vast.ai

* **Access model:** a marketplace with 40+ data centres and per-price-set-by-supply-and-demand. Its pricing page states the platform rate explicitly: **"Prices set by supply and demand … Prices are set by the market, not by Vast."** That sentence is the most important thing on the page — there is no tariff to quote, only an order book. Instances are created from templates and reached over SSH or Jupyter (its docs index has categories for Instances, Rental Types, Jupyter & SSH, Billing, Security, Technical, Networking — all checked 20 Sep 2026).
* **Rental types (checked 20 Sep 2026, [vast.ai/pricing](https://vast.ai/pricing))**:

| Tier | Provider's stated terms |
|---|---|
| On-demand | "Guaranteed uptime", per-second billing, no interruptions, spin up/down anytime |
| Interruptible | "50%+ cheaper", preemptible/may be reclaimed, "ideal for fault-tolerant workloads", checkpoint and resume |
| Reserved | "Up to 50% off", 1/3/6-month terms, guaranteed capacity, volume discounts |

* **Rates:** ⚠ **no per-GPU number is quoted here on purpose.** The card prices are live and change continuously; the page's own copy says prices vary by availability, performance and provider settings. Any figure a guide printed from this source would be wrong by the time it was read.
* **Honest weaknesses:** the host is a third party, the discount tier is preemptible by design, and the platform's own copy concedes that price is set by competition between hosts — which means price, quality and tenancy all vary listing to listing. §6 covers what that means for data.

#### RunPod

* **Access model — verified from RunPod's own docs (checked 20 Sep 2026, [docs.runpod.io/pods/overview](https://docs.runpod.io/pods/overview)):** Pods are containers. You choose a template or bring "custom containers" from a registry, and connect by SSH, JupyterLab, VS Code/Cursor or a web proxy. The docs list explicit limitations, and they are decisive for §2's five tests: **"Docker Compose is not supported: Runpod runs Docker for you, so you cannot spin up your own Docker instance"**; **UDP connections are not supported (TCP and HTTP only)**; **Windows is not supported**. That is a restricted container (model (b)), not a VM — even though it feels like one from the shell.
* **Two clouds, two tenancy models (same docs page):** **Secure Cloud** "operates in T3/T4 data centers, providing high reliability and security for enterprise and production workloads"; **Community Cloud** "connects individual compute providers to users through a vetted, secure peer-to-peer system, with competitive pricing options." The price difference between the two is the tenancy difference, and it is not labelled as such on a pricing comparison.
* **Compute rates (checked 20 Sep 2026, [runpod.io/pricing](https://www.runpod.io/pricing); page states "Updated September 13, 2026"))**, per hour, pods:

| GPU | Community/Secure listed rate |
|---|---|
| B300 (288 GB) | $7.89 |
| B200 (180 GB) | $6.79 |
| H200 (141 GB) | $4.59 |
| H100 SXM (80 GB) | $3.49 |
| H100 PCIe (80 GB) | $2.89 |
| A100 SXM / A100 PCIe (80 GB) | $1.59 |
| L40S (48 GB) | $1.09 |
| Pro 6000 MIG 48GB | $1.09 |
| RTX 6000 Ada (48 GB) | $0.84 |
| A40 (48 GB) | $0.49 |
| L4 (24 GB) | $0.49 |
| RTX 4090 (24 GB) | $0.74 |
| RTX A5000 (24 GB) | $0.27 |

* **Billing granularity — a documented inconsistency worth flagging:** the pricing page offers a per-hour/per-second toggle, while the pods overview page states pods "are billed by the minute with no fees for ingress/egress". Read the provider's own two pages together and the honest conclusion is: **granularity is sub-hour, but which sub-hour unit applies depends on the product and possibly the console setting.** Both figures came from RunPod's own material on 20 Sep 2026.
* **Storage rates (same pricing page, same date):** container disk $0.10/GB/mo; **volume disk $0.10/GB/mo running, $0.20/GB/mo idle**; network storage standard $0.07/GB/mo (under 1 TB) or $0.05/GB/mo (over 1 TB); high-performance network storage $0.14/GB/mo. The idle rate being double the running rate is the single cleanest illustration of §3.1's "a stopped instance still costs" line.
* **Serverless:** worker-hour rates published (H100 $4.79, H200 $5.93, B200 $8.64, B300 $9.98, A100 $2.72, L4-class $0.69) — consistently above the pod rate for the same GPU.
* **Clusters:** H200 SXM $4.31/hr, A100 SXM $1.79/hr; H100 SXM, L40S, B200 contact sales. Reserved clusters are contract-priced.
* **Honest weaknesses:** container-only (no custom kernel/driver, no nested Docker, no UDP), two-cloud tenancy split that is easy to overlook, and enterprise paper (SLA, MSA, certifications) is not something its public pricing page asserts.

#### TensorDock, Salad

* **TensorDock:** ⚠ its pricing URL returned 404 on 20 Sep 2026, so neither its rates nor its access model were verified. Named in the brief, marked unverified here.
* **Salad (SaladCloud)** — a genuinely different architecture, and stated plainly by the provider: Community Cloud "runs on GPUs that already exist in homes and small businesses worldwide. Owners are paid for idle time, and a competitive marketplace keeps prices near the cost of electricity rather than the cost of a data center" (checked 20 Sep 2026, [salad.com/pricing](https://salad.com/pricing)). This is the furthest possible point from a dedicated-VM tenancy model.

| GPU | High | Medium | Low | Lowest | Monthly (Lowest × 730 h) |
|---|---|---|---|---|---|
| RTX 5090 (32 GB) | $0.500 | $0.417 | $0.333 | $0.250 | $182.50 |
| RTX 4090 (24 GB) | $0.330 | $0.273 | $0.217 | $0.160 | $116.80 |
| RTX 5080 (16 GB) | $0.260 | $0.223 | $0.187 | $0.150 | $109.50 |
| RTX 3090 (24 GB) | $0.170 | $0.143 | $0.117 | $0.090 | $65.70 |
| RTX 4070 (12 GB) | $0.150 | $0.123 | $0.097 | $0.070 | $51.10 |

  Additional classes from **$0.015 per GPU-hour at Lowest priority** (provider's own statement). Priority tiers, stated terms: **High** — "never preempted by other workloads. Still subject to node disconnection"; **Medium/Low** — may be preempted by higher-priority groups, "minimum run before reallocation is roughly 30 minutes"; **Lowest (Batch)** — "lowest-cost tier … for retryable batch work and stateless inference behind a queue". Billing is per second while an instance is *running*; allocation, image download and cold start are free. CPU is $0.005/vCPU-hr + $0.001/GB RAM/hr. No free trial. **The provider publishes live schedulable-GPU counts** (data dated 17 Sep 2026 on the page: RTX 4090 16 at high priority, RTX 3090 288 at high priority, RTX 5080 97 at high priority, etc.) — an unusual and honest disclosure of the availability problem in §5.
* **Honest weaknesses:** consumer-grade GPUs in consumer-grade premises; the High tier protects you from other *workloads* but explicitly not from a *node disconnecting*; and no compliance story is asserted on the pricing page.

### 4.4 Bare metal and European regional providers

This class matters for two reasons: true bare metal gives the strongest access model, and European providers give a residency footprint that does not require a data-transfer analysis.

#### Hetzner (Germany/Finland)

* **Access model — verified from the provider's own product page (checked 20 Sep 2026, [hetzner.com/dedicated-rootserver/gex131](https://www.hetzner.com/dedicated-rootserver/gex131/)):** dedicated GPU servers with **"Full root access"**, IPMI remote management, VNC installation, image installer, rescue system, stateless firewall, vSwitch, **no minimum contract term**, cancellation "immediately". That answers §2's five tests with a clean yes on root, kernel, drivers and own scheduler.
* **GPU catalogue:** GEX45 — NVIDIA RTX PRO 4000 Blackwell SFF, 24 GB GDDR7, Intel Core i5-13500, 64 GB RAM, located HEL1 (Helsinki), positioned "for AI inference". GEX131 — NVIDIA RTX PRO 6000 Blackwell Max-Q, 96 GB GDDR7 ECC, Intel Xeon Gold 5412U (24 cores), 256 GB DDR5 ECC upgradable to 768 GB, 2×960 GB NVMe upgradable to 4×3.84 TB, **1 Gbit/s guaranteed port, unlimited traffic**, located HEL1 and FSN1 (Falkenstein). The FAQ on the family page answers the obvious question directly: **Hetzner does not offer H100 GPU servers** — "our GEX servers use NVIDIA RTX GPUs" — and **each server has one GPU and cannot be configured with multiple GPUs**.
* **Rates:** ⚠ the per-month and per-hour figures on both pages render client-side and did not resolve on the date checked, so **no Hetzner price is quoted in this guide.** What is verified is the price *structure*: a monthly rate, an hourly rate, and a one-off setup fee. Use the configurator.
* **Residency and certification:** the provider states GDPR-compliant hosting with "customer master data is not transferred to third countries", 100% green electricity, and data centres certified **DIN ISO/IEC 27001** in Germany and Finland. A DPA under GDPR Article 28 is offered as an optional feature. These are the provider's claims on its own page, at that date.
* **Honest weaknesses:** no H100/H200/B200 at all; one GPU per server; a single 1 Gbit/s port with an exception on the 10G uplink addon (usage above 20 TB charged per TB, outgoing only, unlimited-inbound otherwise) — fine for inference, limiting for multi-node training; and a dedicated server is *always running*, so idle GPU time is 100% billable.

#### OVHcloud (France/EU)

* **Access model:** public-cloud virtual machine instances (General Purpose b3/b2, Compute Optimised c3/c2, Memory Optimised r3) with local NVMe and per-instance public bandwidth. Standard VM semantics.
* **Rates (checked 20 Sep 2026, [ovhcloud.com/en/public-cloud/prices](https://www.ovhcloud.com/en/public-cloud/prices/))** — CPU instance examples that bound the floor: b3-8 (8 GB / 2 vCore / 50 GB NVMe) $0.0605/hr; c3-16 (16 GB / 8 vCore) $0.2154/hr; c3-256 (256 GB / 128 vCore) $3.447/hr. Monthly figures are estimated at 730 hours.
* **Two forward-looking structural facts from the same page, both dated in the provider's own copy:** **"Starting 1 October 2026, local storage and the IPv4 address will no longer be included in the price of b3/c3/r3 instances and will be billed separately"** — a price change that hits exactly the hidden-line category in §3.1 — and a Savings Plan discounting structure (12 months −15%, up to 30% off instances and Managed Rancher per the page's own copy).
* **Rates for GPU instances:** ⚠ not extracted on the date checked; the page as fetched priced CPU families. Do not carry a GPU figure from a third party into this guide.

#### Other European / regional options

* **Gcore** — ⚠ the `/cloud/gpu` path returned a 404 on 20 Sep 2026; the site's navigation describes a "GPU Cloud" and "AI Cloud" product and a "Digital Sovereignty" solution, but no rate was readable. Unverified; do not cite.
* **Paperspace (dba DigitalOcean)** — the platform layer is public: Free tier with **auto-shutdown (12-hour limit)** and 5 GB storage; Pro at $8/month with configurable auto-shutdown and 15 GB storage; Growth at $39/month with 50 GB storage and "high-end instances"; team tiers T0/T1/T2 at $0/$12/user/month/contact-sales, with **persistent storage overage at $0.29/GB** and documented notebook-running limits per tier. Prices "plus utilization costs on paid instances" — ⚠ **the per-GPU hourly rates sit behind the Compute/Usage pricing tab and were not readable on the date checked.** Note the auto-shutdown design: it exists precisely because unattended notebooks are a known cost leak (§12).
* **Modal** — serverless endpoint model (see §4.5).
* **Fly.io** — ⚠ no GPU rate was readable on its public pricing page on 20 Sep 2026. What the page does verify is the *platform* pricing structure around a GPU app: stopped Machines are charged only for rootfs at $0.15/GB per 30 days; **Fly Volumes $0.15/GB-month, "charged for volumes that you create, whether they are attached to a Machine or not, including when an attached Machine is stopped"**; volume snapshots $0.08/GB-month with the first 10 GB free (charges introduced 1 January 2026, first invoiced February 2026 per the page's own notice); egress $0.02/GB North America/Europe, $0.04/GB Asia-Pacific/Oceania/South America, $0.12/GB Africa/India; private cross-region transfer $0.006/GB (NA/EU); static egress IP $0.005/hr; Machine reservation blocks at a 40% discount. Fly does not support UDP-equivalent constraints to RunPod, but it is a platform-as-a-service, so treat the access model as (b)/(d), not (a), until its GPU family is verified.

### 4.5 Serverless and managed-endpoint providers

The access model is (d): no host access, no kernel, no scheduler of your own. The pricing model is the reason to use them.

* **Modal — rates verified (checked 20 Sep 2026, [modal.com/pricing](https://modal.com/pricing))**, per second, and the per-hour figure is arithmetic shown here for comparability only:

| GPU | $/second (provider's unit) | ≈ $/hour (arithmetic) |
|---|---|---|
| NVIDIA B300 | $0.001972 | ≈ $7.10 |
| NVIDIA B200 | $0.001736 | ≈ $6.25 |
| NVIDIA H200 SXM | $0.001261 | ≈ $4.54 |
| NVIDIA H100 SXM5 | $0.001097 | ≈ $3.95 |
| NVIDIA RTX PRO 6000 | $0.000842 | ≈ $3.03 |
| NVIDIA A100 80 GB | $0.000694 | ≈ $2.50 |
| NVIDIA A100 40 GB | $0.000583 | ≈ $2.10 |
| NVIDIA L40S | $0.000542 | ≈ $1.95 |
| NVIDIA A10 | $0.000306 | ≈ $1.10 |
| NVIDIA L4 | $0.000222 | ≈ $0.80 |
| NVIDIA T4 | $0.000164 | ≈ $0.59 |

  Modifiers that materially change the bill, all stated by the provider on the same page: **region selection multiplies base prices by 1.15–1.75×**, and **"non-preemptible execution" costs 3× base prices** — i.e. Modal's default execution is preemptible, and buying out that risk is a published 3× surcharge. Volumes are $0.09/GiB/mo with 1 TiB/mo free per the page. Plans: Starter $0 with $30/month free compute, 3 seats, 10 GPU concurrency; Team $250/month with $100/month free compute and 50 GPU concurrency; Enterprise custom with audit logs, SAML SSO and HIPAA, and committed-spend transactions through the AWS and GCP marketplaces. **SOC 2 is listed as a feature on all tiers; HIPAA compatibility and audit logs appear in the enterprise column.**
* **Crusoe Managed Inference and Together** — both sell endpoints per token or per model unit (§4.2); the per-token price is not comparable to a GPU-hour and should not be.
* **Baseten, RunPod public endpoints, and other token-priced platforms** — the brief names Baseten; ⚠ **no Baseten GPU-hour rate was verified in this research pass.** Its product shape (managed inference) places it in model (d).

### 4.6 Chinese providers

Relevant for two distinct reasons: absolute price competitiveness, and — more decisively for most regulatory readers — **where the data physically sits and what law applies to it.**

* **Alibaba Cloud (Elastic GPU Service)**, rates read from the provider's own product page (checked 20 Sep 2026, [alibabacloud.com/en/product/gpu/pricing](https://www.alibabacloud.com/en/product/gpu/pricing)):
  * GPU-accelerated compute-optimised instances: gn8is (NVIDIA L20 ×8, 48 GB ×8) **from $1,279.25/month**; gn7i (A10 ×4, 24 GB ×4) **from $676.49/month**; gn6e (V100 ×8, 32 GB ×8) **from $1,319.93/month**; gn6v (V100 ×8, 16 GB ×8) **from $884.54/month**; gn6i (T4 ×4) **from $466.34/month**; gn5 (P100 ×8) **from $520.9/month**.
  * Product families documented on the same page: **GPU-accelerated ECS bare-metal instance family** (true bare metal), **vGPU-accelerated instances**, and software tools **AIACC-Training, AIACC-Inference, FastGPU (cluster deployment) and cGPU (GPU splitting/isolation for containers)**.
  * The purchase links in the provider's own page point at `cn-beijing`, `cn-wulanchabu` and `cn-zhangjiakou` zones — i.e. **a mainland-China footprint on the pages surfaced for this guide**. Hourly rates are ⚠ not shown on that page (monthly list prices only); the linked ECS pricing list is where hourly would be read.
* **Tencent Cloud, Huawei Cloud, Volcano Engine (ByteDance)** — all named in the brief as providers of GPU capacity. ⚠ **No rate or access-model claim for these three was verified from their own documentation within this research pass**, so none is asserted. They are recorded in §14 as unverified candidates.
* **AutoDL** — a Chinese GPU rental marketplace for researchers. ⚠ **No price or access-model claim was verifiable from its own pages in this pass.** Unverified; do not cite.
* **The consequence that matters regardless of price:** for a regulated buyer, a mainland-China region means the training data, the weights and the telemetry are subject to Chinese law and cross-border transfer rules on top of your own. That is a §10 gate, not a §3 line item, and it eliminates the option before price is discussed. For an unregulated team with Chinese operations, the same fact is an advantage (proximity, price) rather than a blocker.

### 4.7 Provider-class summary

| Class | Typical access model | Price position | Availability | Compliance surface | Right for |
|---|---|---|---|---|---|
| Hyperscalers | True VM (a) | Highest | Quota-gated, predictable once granted | Strongest (already in your procurement) | Production, regulated, integrated estates |
| Neoclouds | True VM / bare metal (a) | Low–mid per GPU-hour | Capacity crunch; spot reclaim | Provider-specific, verify per claim | Training at scale, cost-sensitive production |
| Marketplaces | Container or host-dependent (b) | Lowest | Live order book, varies hourly | Weakest; often none published | Experiments, fault-tolerant batch |
| Bare metal / EU regional | True bare metal / VM (a) | Predictable monthly, no hourly elasticity | You own the box; availability is a stock question | Strong on residency (EU/GDPR/ISO) | Steady inference, residency-bound work |
| Serverless / endpoints | Endpoint (d) | Per-second, scales to zero | Effectively elastic | Varies; enterprise tiers carry the good paper | Spiky or unpredictable inference |

The table's last column is the point: **each class is right for something and wrong for something else, and no class is right for everything.** That is why §8 is a decision framework rather than a ranking.

---

## 5. The Availability Problem

### 5.1 The statement this section exists to make

**A listed rate is not a guarantee of access.** Every rate in §4 is a price at which capacity *can* be sold to you. None of them is a promise that capacity exists at the moment you need it, in the region you need it, at the quantity you need. In a market where demand has outrun supply on the newest accelerators, availability is frequently the binding constraint and price is only the second constraint — which is exactly the reversal of the assumption most price-comparison exercises start from.

Three pieces of evidence from the providers themselves, all read on 20 September 2026:

1. **Lambda labels its self-serve on-demand tier "first-come"** and routes everyone else to a sales conversation for committed clusters. A first-come model means the rate is real but the capacity allocation is a race.
2. **Salad publishes live schedulable-GPU counts** — on the date its page carried data dated 17 Sep 2026, showing 16 schedulable RTX 4090s at high priority and 288 RTX 3090s. That is a provider being unusually honest about how thin the top of its catalogue is.
3. **Provider cards increasingly carry "Contact sales" where a price used to be** (CoreWeave's GB300 NVL72 and standard-memory RTX PRO 6000; Crusoe's entire spot column, GB200, B200 and MI355X; Nebius's GB300/GB200 NVL72; Scaleway's H100/L40S/B300 outside PAR-2). "Contact sales" is the market's way of saying *this SKU is allocated, not sold*.

### 5.2 What an interruption actually does, per tier

Preemption is an abstraction until it hits a running job. Concretely, per rental tier, using each provider's own wording:

| Tier / provider | What the provider says happens | Consequence for a running job |
|---|---|---|
| Vast.ai Interruptible | "You set a bid price … Can be stopped by higher bids." The docs are explicit about the failure mode: "Your instance is stopped (killing running processes)" and "Instance may wait long to resume." | The job dies, unsaved state is lost, and **resuming is not immediate** — you wait to win the bid again. |
| CoreWeave Spot | Listed spot rates ($19.71/hr for an 8×H100 node against $49.24 on-demand) with no published notice window on the pricing page. | Assume reclaim is possible at any time; the 60% saving is the payment for that. |
| Nebius Preemptible | Published preemptible column ($2.15 vs $4.50 per H100 GPU-hour). | Same shape: a discount in exchange for the provider's reclaim right. |
| Salad Medium/Low | "May be preempted by higher-priority groups. Minimum run before reallocation is roughly 30 minutes." Highest priority is the one that "protects from preemption by other workloads". | A ~30-minute floor is a guaranteed granularity, not a guarantee. Salad's **High** tier is explicitly "never preempted by other workloads — still subject to node disconnection", i.e. even the top tier can lose a *node*, and reallocation is automatic but disruptive. |
| Modal (default) | Non-preemptible execution is priced at **3× base prices**. | The default *is* preemptible; buying the risk out is a published surcharge, which is the cleanest price signal in the whole market about what interruption actually costs. |
| On-demand anywhere | No interruption by the provider. | The rate you paid for the guarantee. Capacity is still subject to provider-side incidents. |

The design conclusion: **preemption is a workload property, not a pricing property.** A job that checkpoints every five minutes and can resume from a checkpoint pays almost nothing for spot. A stateful inference service with in-memory session state cannot use spot at any discount, because the discount does not cover the outage.

### 5.3 Reservation vs on-demand vs spot vs marketplace queueing

| Mechanism | Who bears the capacity risk | What you pay for it |
|---|---|---|
| Reserved / committed | The provider holds the capacity for you | A term, and usually an upfront or minimum-spend commitment. Nebius advertises up to 35% off; DataCrunch publishes a ladder (1 month 2% → 2 years 25%); Hyperstack publishes "reservation from" rates below on-demand |
| On-demand | You do, per hour | The on-demand rate, plus the risk that the instance is not there when you want it |
| Spot / interruptible | You, always, with no notice guarantee | A large discount — CoreWeave ≈60% off H100 on-demand, Vast "50%+" cheaper on the pricing page and "saves 50-80%" in its docs, Hyperstack's spot column visibly below on-demand |
| Marketplace | You, twice — from the host disappearing and from the host's price moving | The lowest rate in the market, in exchange for both risks |
| Serverless | The provider, up to its concurrency limits | Per-second pricing and, on Modal, a 1.15–1.75× region multiplier and a 3× non-preemptible surcharge |

Note what reservation buys and does not buy. It converts an availability risk into a cost risk: you have traded the possibility of not getting capacity for the certainty of paying for capacity. **If your utilisation forecast is wrong, reservation turns a shortage into a bill.**

### 5.4 What a team must measure before committing

Six measurements, in this order, before signing anything:

1. **Actual utilisation of the fleet you already have.** Not requested capacity — *used* GPU-seconds divided by paid GPU-seconds. If nobody can produce this number, no commitment should be signed (§12, anti-pattern 8).
2. **Job restart time, measured.** Take a representative training job, kill it mid-flight, and time how long it takes to resume from the last checkpoint. If that number is hours, the spot discount is smaller than it looks.
3. **Obtainability, tested at your quantity and region.** Actually try to launch the node count you need, on the provider, on three separate days. A rate you cannot buy is a press release.
4. **Queue behaviour under load.** For marketplaces: how long did the interruptible instance wait to be re-scheduled after losing a bid? The docs say "may wait long to resume" — measure how long *for your job*.
5. **Quota headroom, in writing.** For hyperscalers and for any provider with an account-level cap: what is the ceiling, and who raises it?
6. **The cost of the fallback.** Running the same job on the second-choice provider, including re-uploading the dataset and re-building the image. A fallback that costs two engineer-days is not a fallback.

Only after those six does a rate comparison mean anything. Before them, it is a comparison of menus in a restaurant that may be closed.

---

## 6. The Security, Tenancy and Compliance Question

### 6.1 Isolation is a spectrum, and the access model tells you where you are

The four access models in §2 map almost exactly onto four isolation strengths:

| Model | Isolation mechanism | Who else is on the hardware | Practical exposure |
|---|---|---|---|
| True VM / bare metal (a) | Hypervisor or no hypervisor at all; GPU passed through | Possibly nobody (bare metal); otherwise other VMs that cannot see yours | Strongest. Your kernel, your driver, your disk, your keys. |
| Restricted container (b) | Namespaces + cgroups on a **shared host kernel** | Other tenants' containers on the same kernel | Strong in practice, but the isolation boundary is the host kernel and the host's operator |
| Managed notebook (c) | Provider-managed container on provider-chosen images | Other workspaces, typically | You do not control the image, the host, or the persistence |
| Serverless endpoint (d) | Provider's runtime, provider's network | Opaque by design | You cannot assess it; you trust the enterprise tier's paperwork |

Two verified illustrations of the two ends:

* **Vast.ai, from its own security FAQ (checked 20 Sep 2026):** "Clients are isolated in unprivileged Docker containers and only have access to their own data. Each container is completely separate from others on the same host machine with: separate namespaces and cgroups, network isolation, file system isolation, process isolation." Its compliance page adds: "Every workload runs in an unprivileged Docker container, isolated from other tenants … no shared filesystems between tenants … Data is destroyed immediately when a client deletes an instance."
* **The same FAQ, one question later — the honest part:** asked how data is protected *from providers*, Vast.ai answers "Provider security varies significantly: Tier 4 datacenters have extensive physical and operational security; **individual hosts may have less formal security measures**." Its recommended mitigations are: use Secure Cloud certified providers only, encrypt sensitive data at rest, don't store credentials in instances, use external key management.

That second quote is the most valuable sentence in this guide for a security reviewer. **The marketplace's own documentation tells you that the isolation guarantee applies to other tenants, not to the host operator.** If the host can see your volume, a marketplace is not a place for unencrypted regulated data — and no amount of container isolation changes that.

### 6.2 Tenancy models actually on offer

| Tenancy model | Where it appears | What it means |
|---|---|---|
| Dedicated bare metal | Hetzner GEX (one physical server, full root), hyperscaler bare-metal families, Alibaba ECS bare-metal GPU family | No co-tenant on the silicon. You also own the patching. |
| Dedicated VM / passthrough | Lambda, CoreWeave (NA/EU), Nebius, DataCrunch, Hyperstack, Scaleway, OVHcloud | Strong isolation; still a provider-managed host under your VM. |
| Shared host, container-level isolation | RunPod Pods, Vast.ai (all tiers at container level), Salad | Namespace/cgroup isolation on a kernel you do not control. |
| Vetted data-centre tier within a marketplace | Vast.ai **Secure Cloud** | Same container isolation, but the *facility* has been vetted: minimum 5 flagship-class GPU servers, signed Data Processing Agreements with Vast.ai, due diligence on facility security, ownership and business identity, per its compliance page. |
| Peer-to-peer consumer hosts | RunPod **Community Cloud** ("individual compute providers … a vetted, secure peer-to-peer system"); Salad ("GPUs that already exist in homes and small businesses worldwide") | The weakest tenancy model on the market, sold at the lowest price. Not a defect — a disclosed trade. |

### 6.3 Certifications each provider actually publishes

**Rule applied here: only what the provider itself claims, in its own words, with a date.** No certification is inferred from partnership, size, or another provider's claim.

| Provider | What the provider itself publishes (checked 20 Sep 2026) | Source |
|---|---|---|
| Nebius | SOC 2 Type II (**with HIPAA**), SOC 3, ISO 27001, ISO 27018, ISO 22301, ISO 27032, ISO 27701, ISO 27799, **ISO 42001** (AI management systems), NIS 2 alignment, **DORA** alignment, CSA STAR Level 1. Isolation claimed as VPC segmentation, InfiniBand traffic segregation, per-tenant Kubernetes clusters isolated at the VM level. At-rest encryption "by default", with an explicit carve-out for Network SSD non-replicated and IO M3 disks. | [nebius.com/trust-center](https://nebius.com/trust-center) |
| Vast.ai | **SOC 3** (available on request) and **SOC 2 Type 2** (completed; report under NDA). **HIPAA** supported on the Secure Cloud tier with BAAs "for qualifying customers". Secure Cloud datacentre partners' certifications "may include" ISO 27001, ISO 20000-1, ISO 22301, ISO 14001, SOC 1/2/3, HIPAA, HITRUST, PCI DSS, NIST frameworks — **but the same page states these are "encouraged … but are not strictly required"**. | [vast.ai/compliance](https://vast.ai/compliance) |
| CoreWeave | Aligns its programs with SOC 2, ISO 27001, ISO 27017, ISO 27018; detailed evidence is requested through its Compliance Documentation Portal, "including policies, control descriptions, and (where available) third-party assessments". Note the wording — *alignment with frameworks* plus a portal, not a badge on the pricing page. | [docs.coreweave.com/security/trust-compliance/compliance-programs](https://docs.coreweave.com/security/trust-compliance/compliance-programs) |
| Hyperstack | Displays a **SOC 2 Type 2** mark on its own pricing page. | [hyperstack.cloud/gpu-pricing](https://www.hyperstack.cloud/gpu-pricing) |
| DataCrunch / Verda | States **"SOC 2 and GDPR compliant"** on its own pricing page; its trust centre is at trust.verda.com. | [datacrunch.io/pricing](https://datacrunch.io/pricing) |
| Hetzner | **GDPR-compliant hosting**, "customer master data is not transferred to third countries", data centres certified **DIN ISO/IEC 27001** in Germany and Finland, DPA under GDPR Art. 28 available. 100% green electricity. | [hetzner.com/dedicated-rootserver/gex131](https://www.hetzner.com/dedicated-rootserver/gex131/) |
| Modal | **SOC 2** listed on all plans; **HIPAA compatibility**, audit logs and SAML SSO in the Enterprise column. | [modal.com/pricing](https://modal.com/pricing) |
| Lambda | ⚠ No certification claim was readable from Lambda's own pricing or legal pages in this pass. Not asserted here. | — |
| RunPod | ⚠ No certification claim readable from its pricing or pods docs; the docs describe Secure Cloud as "T3/T4 data centers … high reliability and security for enterprise and production workloads", which is a description, not an attestation. | [runpod.io/pricing](https://www.runpod.io/pricing) |
| Salad, Scaleway, OVHcloud, Gcore, Paperspace, Fly.io, Together, Crusoe, Alibaba Cloud, Tencent, Huawei, Volcano Engine | ⚠ Not verified from provider-owned pages in this pass. Do not assert any certification for these in a procurement document on the strength of this guide. | — |

### 6.4 Data at rest and in transit

Verified specifics rather than blanket claims:

* **Vast.ai:** "All API and console traffic is encrypted in transit via TLS 1.2+" (compliance page). SSH is described as encrypted by default and Jupyter as HTTPS "with self-signed certificates" (security FAQ) — note that last detail: a self-signed certificate is encrypted but not authenticated by a public CA, which matters if you expose anything beyond your own SSH tunnel.
* **Nebius:** storage encrypted at rest by default, with an explicitly named exception for certain non-replicated/IO disks; TLS for data in transit; and its own footnote states that "logging, monitoring and encryption are shared responsibilities … customers must configure observability and choose between encrypted or faster unencrypted storage". Read that sentence carefully — **on at least one GPU cloud, encryption is a customer choice, and the *faster* option is the unencrypted one.**
* **Marketplaces generally:** the security advice is client-side by construction (encrypt before upload, external key management, destroy instances when done). That is the correct advice and also the honest admission that the platform cannot make the host trustworthy.

For the storage-side treatment of encryption, key management and egress, see `technology/cloud_object_storage_comparison_guide.md`.

### 6.5 The questions a bank must ask before putting data or weights on a marketplace host

Asked in order; a "no" on any of the first four should stop the conversation, regardless of rate.

1. **Who is the host, legally?** Not the marketplace — the *host*. Name, entity, jurisdiction, and the contract you have with them. If the answer is "the marketplace's terms of service govern", that is not an enterprise MSA and there is no negotiated liability cap, no audit right, and no named data processor. (Vast.ai's Secure Cloud tier does at least require signed Data Processing Agreements with datacentre partners and due diligence on ownership and business identity — that is a meaningful difference from an individual host.)
2. **Can the host read the volume?** Assume yes unless the data is encrypted with keys the host never sees. The platform's own FAQ says provider security "varies significantly".
3. **Where does the data physically sit, and which law applies?** Marketplace capacity is distributed across 40+ data centres; your dataset may land in a jurisdiction your DPO has not approved. There is no residency control unless the platform exposes one.
4. **What happens at termination — and can you prove it?** "Data is destroyed immediately when a client deletes an instance" is a claim, not a certificate of destruction. For regulated data you need the processor's documented deletion process, not a sentence on a marketing page.
5. **What is the incident-notification obligation, and in what timeframe?** DORA-style regimes put hard clocks on this for financial entities. A marketplace ToS typically contains no such commitment.
6. **Is there an audit right?** Can you or your auditor inspect the controls, at the host, on request? On dedicated-VM and bare-metal providers this is negotiable; on a peer-to-peer host it is not.
7. **Is model weight a regulated asset in your jurisdiction?** Weights derived from customer data may be subject to the same handling obligations as the data. Where the weights are *stored* is therefore part of the model-risk record, not just an ops detail.
8. **Concentration and exit.** If this host disappears next week, what is the recovery path and what does it cost in egress and engineer time? (§9 and §10.)

The worked version of these gates appears in §11, where they are the reason two otherwise-cheapest options are eliminated before price is discussed.

---

## 7. The Performance-Per-Dollar Reality

### 7.1 The point of this section, stated plainly

A price per GPU-hour is only comparable between two hosts if the GPU inside them delivers the same work per hour. It frequently does not, because the host around the GPU — CPU, RAM, disk, network, interconnect, thermals, driver — is part of the product. **A per-hour rate compares poorly when the host starves the GPU.** The cheapest instance on the page can be the most expensive per unit of useful work, and getting that wrong is a §12 anti-pattern.

### 7.2 The components that decide useful work per dollar

| Component | Why it changes $/unit-of-work | Evidence available in this guide's research |
|---|---|---|
| **GPU generation and memory** | A B200 with 180 GB and a 40 GB A100 are not substitutable for the same model. Insufficient VRAM forces sharding, which multiplies both cost and communication. | Lambda's own catalogue spans 16 GB (V100, $0.79/GPU-hr) to 180 GB (B200, $6.69) — an 8.5× price range and an 11× memory range in one product list (20 Sep 2026). |
| **Interconnect: NVLink vs InfiniBand vs Ethernet** | Model-parallel and multi-node training are bounded by fabric bandwidth and latency, not by FLOPs. | Nebius states per-tenant Kubernetes clusters are isolated at the VM level *and* that traffic segregation is enforced **at the InfiniBand layer** — fabric is a first-class product there. DataCrunch sells "instant clusters" with **InfiniBand interconnect** explicitly as the feature. Hyperstack publishes a technical note on **"RDMA, RoCE and Performance Benchmarks in SR-IOV"** — all provider-published (all checked 20 Sep 2026). |
| **PCIe vs SXM form factor** | SXM parts carry the NVLink fabric; PCIe parts do not, and the same GPU name appears in both. | Lambda and Hyperstack both price the *same* H100 and A100 in PCIe and SXM/NVLink variants at different rates — Hyperstack lists H100 SXM at $3.20 and H100 NVLink at $2.60 and H100 PCIe at $2.50; Azure-class naming conventions aside, the point is that the variant is a separate SKU with a separate price. |
| **Host CPU and RAM per GPU** | A data pipeline that cannot feed the GPU leaves it idle. | CoreWeave's and Lambda's cards both print vCPUs and system RAM **per GPU** precisely because the ratio is a purchasing decision, not a footnote. Lambda's 8×H100 instance carries 208 vCPUs and 1,800 GiB RAM; its 1×H100 PCIe instance carries 26 vCPUs and 225 GiB. |
| **Local disk type and size** | Dataset loading at startup is paid GPU time. | Lambda bundles **22 TiB SSD** on its 8×H100 instance and **512 GiB–1 TiB** on smaller ones — a 20×+ difference in local capacity at the same GPU rate. A job whose dataset exceeds local disk streams from network storage, which then becomes the bottleneck. |
| **Network throughput for data loading** | Model weights are large; a per-hour GPU rate that spends its first 20 minutes downloading is a rate you are paying while doing nothing. | OVHcloud prices public bandwidth *per instance tier* from 250 Mbit/s guaranteed (b2-7) to 20 Gbit/s (b3-512) — bandwidth is a line item in the product, not a constant. Hyperstack guarantees 1 Gbit/s on its GPU hosts. Hetzner guarantees 1 Gbit/s and charges only above 20 TB on the 10G uplink addon. |
| **Thermals and consumer-grade hardware** | Consumer cards throttle; datacentre cards do not (as much). | Vast.ai's own technical FAQ lists **"Thermal throttling — some consumer GPUs throttle"** and **"PCIe bandwidth — Multi-GPU setups may be limited"** among the reasons training is slower than expected (20 Sep 2026). |
| **Pre-installed software stack** | A curated stack with a matched driver/CUDA/PyTorch combination saves hours of setup — once. After that it is worth nothing, and it can hurt if it pins you to a version. | RunPod's docs describe official templates that remove the need to install PyTorch and JupyterLab yourself; Vast.ai's templates let you "add Docker run arguments" and note that the CUDA version "depends on the Docker image". Lambda ships its Lambda Stack. |
| **Host kernel, nesting and privileges** | Some workloads need privileged containers, custom drivers or a specific kernel version. | RunPod: "Runpod runs Docker for you, so you cannot spin up your own Docker instance." Vast.ai: "Docker-in-Docker is disabled for security." Both are correct security choices that nonetheless remove a capability (all checked 20 Sep 2026). |

### 7.3 The cheap-GPU-with-a-slow-disk trap

The trap in one sentence: **you are billed for the GPU while the disk is the bottleneck.**

The pattern is easy to fall into because the comparison surface — the GPU column — looks excellent:

1. A marketplace listing offers an RTX 4090 at $0.16/GPU-hr (Salad's Lowest tier, verified 20 Sep 2026).
2. The job needs to read a 400 GB embedding table per epoch.
3. The host's disk path cannot deliver it at the rate the GPU can consume it, so the GPU idles at 30% utilisation while the clock runs.
4. Effective cost is now $0.16 / 0.30 ≈ $0.53 per *useful* GPU-hour — above a datacentre-class H100 at a far higher sticker rate that actually keeps the GPU fed.

Nothing in that arithmetic is exotic. It is the reason §5.4's measurement list and this section exist: **measure achieved GPU utilisation, not advertised clock speeds.** Utilisation achieved is available locally from `nvidia-smi` or a proper telemetry stack — `technology/nvidia_dcgm_guide.md` owns the instrumentation, and `technology/gpu_optimization_guide.md` owns the fix.

A corollary for the multi-GPU case: Vast.ai's FAQ lists "try a different provider" among the debugging steps when an instance will not start, which is the marketplace reality in one line — **the same product name can be a different machine on every host, and the marketplace itself says so.**

### 7.4 Every benchmark, labelled by who measured it

**Rule: a benchmark number without a named measurer is an advertisement.** Benchmarks are listed here with the measurer named, and none is used as evidence of a ranking.

| Benchmark / source | **Who measured it** | What it is good for | What it is not |
|---|---|---|---|
| **MLPerf Training, Training: HPC, Inference: Datacenter, Storage, Endpoints, Client, Edge, Mobile, Tiny** | **MLCommons** working groups, with results submitted by member and non-member organisations under a signed agreement and a defined benchmark tripod (fixed model, fixed dataset, defined allowable changes) | The closest thing to a neutral, reproducible cross-system comparison; the model/dataset tripod is designed to "enable fair comparison of competing systems". MLPerf **Storage** is specifically about how fast storage can supply training data — the trap in §7.3, benchmarked | It measures systems on the benchmark's workloads. Your model, your data pipeline and your utilisation are not the benchmark. Provider-and-configuration coverage is uneven. |
| **DLPerf** (Vast.ai) | **Vast.ai** itself — its docs define DLPerf as "our scoring function that estimates performance for typical deep learning tasks", predicting iterations/second for tasks like ResNet50, and publish example scores (V100 ≈21, 2080 Ti ≈14, 1080 Ti ≈10) | Comparing two *listings* on the same marketplace, on a common scale, when you cannot benchmark them yourself | It is a provider's own scoring function, optimised for common DL tasks and explicitly "less accurate for unusual compute patterns" and "not optimized for non-ML workloads" — the docs say to benchmark specialised workloads yourself. |
| **Provider performance-benchmark blogs** (e.g. Hyperstack's A100 NVLink vs H100 SXM LLM-inference benchmark, A100 PCIe vs SXM, L40 vs RTX A6000; DataCrunch's published GPU benchmarks and R&D posts) | **The provider**, authored and dated by named staff (Hyperstack's are attributed and dated 2024–2025 on its own site) | Useful for understanding *which* hardware variant suits which workload, and for learning the methodology a team should reuse | Vendor-authored. A provider benchmarking its own catalogue is not an independent test. Reproduce the method on your own workload before you rely on the conclusion. |
| **Your own numbers** | **You** | The only measurement that decides your bill: achieved GPU utilisation, tokens/second at your sequence length, steps/hour for your model, cost per useful GPU-hour | Costs engineer time. There is no substitute. |

### 7.5 The rule this section produces

Compare **cost per unit of work you actually need**, using your own measurement, on the same job: cost per training step, cost per million tokens served, cost per evaluation completed. Where that number is unavailable, the second-best proxy is rate × measured utilisation — and the worst possible metric is the headline rate alone, which is what §3 and this section exist to replace.

---

## 8. The Usage-Pattern Fit

### 8.1 The decision framework

The framework maps a **workload class** to an **access model** (from §2), a **pricing tier** (from §3.3) and a **provider class** (from §4.7). It is a framework, not a ranking: the right answer for one row is frequently the wrong answer for the row below it.

| Workload class | Minimum access model | Pricing tier | Provider class | Reasoning |
|---|---|---|---|---|
| **Learning / dev sandbox** | Container (b) is enough | On-demand, per-second | Marketplace or neocloud | Restartability is irrelevant, cost is tiny, and the point is access to a GPU *now*. Cross-ref `technology/gain_nvidia_experience.md` for the learning pathway itself — note that guide's stored rates are stale and superseded by §4 here. |
| **Fine-tuning job** | True VM (a) preferred; container (b) workable if no kernel/driver work is needed | Committed or on-demand; interruptible with checkpointing | Neocloud or EU regional for production; marketplace for experiments | Needs a persistent volume for checkpoints and a real filesystem. Multi-day runs make the spot discount material *if* the job checkpoints. |
| **Sustained inference serving** | True VM (a) | Reserved or on-demand | EU regional, neocloud, or hyperscaler | A stateful service cannot use spot. Per-hour cost dominates because the GPU is running continuously; utilisation is high, so the lowest *rate* for the required GPU wins — and residency may overrule that (§10). |
| **Large training run (multi-node)** | True VM / bare metal (a) with a fast fabric | Committed — reservation is the only way to be sure the cluster is there | Neocloud or hyperscaler | Interconnect and multi-node scheduling dominate. Marketplace hosts cannot form the cluster, and serverless cannot either. This is the one row where the hyperscaler premium is often justified by quota certainty alone. |
| **Evaluation batch** | Container (b) or true VM (a) | Interruptible / spot | Marketplace | Restartable by construction. This is the workload that should *always* run on the cheapest interruptible tier. |
| **Short burst (< 1 hour)** | Any | Per-second on-demand | Serverless or per-second marketplace | Billing granularity decides (§3.2). An hourly minimum can multiply the cost of the same job several-fold. |

### 8.2 How to read the table

Three rules fall out of it:

1. **The access requirement eliminates first, the price decides second.** If the workload needs a kernel module, the marketplace column is gone before any rate is read. Half of all bad GPU-cloud decisions are access-model mistakes dressed up as price comparisons.
2. **Restartability is what makes the discount usable.** Spot, interruptible and Lowest-priority tiers are cheap because they can be taken away. If your job cannot resume from a checkpoint, that discount is not available to you at any price — you are an on-demand customer who has not admitted it yet.
3. **Utilisation, not rate, decides the monthly bill for anything that runs continuously.** §3.4's Workload B shows the ranking inverting on this alone.

### 8.3 When the hyperscaler premium IS worth paying

Honest conditions — this guide does not argue against hyperscalers, it argues against paying the premium by accident:

* **You already have a committed spend agreement.** If the money is committed, the marginal decision is not "which provider is cheaper" but "which provider can I draw against an existing commitment", and an existing commitment makes a nominally higher list price cheaper than new cash.
* **Integration is the workload.** If the GPU job must sit inside an existing VPC, IAM, logging, key-management and network estate, the integration work you avoid is real engineer time — frequently more than the rate delta for a modest job.
* **Compliance evidence is a hard requirement.** Where the audit right, the certification set, the incident-notification clock and the DPA must be negotiated rather than accepted, the smaller providers may simply not be able to sign (§6.5).
* **Support escalation matters.** If the job is business-critical and you need a named escalation path with a contractual response time, that is a paid feature and hyperscalers sell it best.
* **Quota certainty is the product.** For a large multi-node run, being *guaranteed* capacity is worth more than a lower rate on capacity you might not get (§5.3).

### 8.4 When the premium is NOT worth paying

* **The workload is a self-contained job on pre-baked images.** A container, a dataset and a script are not integrated with anything.
* **The GPU is the whole product.** Pure matrix multiplication does not benefit from your provider's managed databases.
* **The job is interruptible.** The discount available outside the hyperscalers (CoreWeave's spot H100 node at $19.71 against $49.24 on-demand; Vast.ai's stated 50–80% interruptible saving) is not matched by hyperscaler spot for the same SKU in most cases.
* **The experiment is disposable.** For a two-day pilot, the cost of *procuring* access inside a large organisation can exceed the compute cost of the pilot itself.

### 8.5 The framework in one line

**Access model is a filter, restartability is an enabler, utilisation is the multiplier, and compliance is the gate.** Price enters only after the first, third and fourth are known.

---

## 9. Rent Versus Own

### 9.1 The four routes, not two

"Rent or own" is usually framed as a binary. It is not. There are four routes, and one of them involves no GPUs at all:

| Route | What you buy | Fixed vs variable | Access model |
|---|---|---|---|
| **Rented GPU capacity** | GPU-hours from any provider in §4 | Almost entirely variable | Whatever §2 says for that provider |
| **Owned workstation GPU** | A box under a desk | Almost entirely fixed; power and noise are real costs | Full control, no multi-node, no datacentre |
| **Owned node, colocated** | Hardware you buy, housed in a third-party facility | Fixed hardware + monthly colo | Full control, real fabric possible, you own the ops |
| **On-premises cluster** | Hardware, facility, power, cooling, staff | Heavily fixed, plus headcount | Full control; `technology/on_prem_llm_deployment_guide.md` owns this route |
| **Managed inference API (no GPUs)** | Tokens or model units | Purely variable, zero infrastructure | Model (d); see §9.5 |

### 9.2 Break-even arithmetic, with the assumptions exposed

**Every number in this sub-section is illustrative arithmetic with invented capital and opex parameters.** The parameters are stated so you can replace them; the *shape* of the answer is what transfers.

Illustrative parameters: capital **$45,000** for a single 96 GB-class GPU box, all-in; colocation plus bandwidth **$350/month**; draw **1.2 kW**; electricity **$0.20/kWh**; 8,760 hours in a year.

* Illustrative annual operating cost: **$6,302**
* Break-even against a rental rate, if you could resell the capacity at that rate continuously:
  * at a **$1.85/GPU-hr** rental equivalent (a real, verified on-demand rate: Hyperstack's RTX PRO 6000 SE, §4.2) → **4.54 years** to recover the capital
  * at a **$1.30/GPU-hr** reserved equivalent (same source, reservation column) → **8.85 years**
  * at a **$0.50/GPU-hr** rate (marketplace territory) → **never** — the operating cost alone exceeds the rental equivalent
  * at a **$3.99/GPU-hr** rate (a datacentre-class H100 rate, Lambda on-demand, §4.2) → **1.57 years**

The spread from 1.57 years to "never" across rates that all exist in the market on the same day is the whole point: **the rent-versus-own answer is not a fact about GPUs, it is a fact about which rental rate you would actually be substituting against.** Compare an owned box against a $0.50/GPU-hr marketplace rate and owning loses forever. Compare it against $3.99 and owning pays back in under two years *at 100% utilisation*.

Utilisation then does the rest. Effective cost per *paid* GPU-hour of owning the illustrative box:

| Horizon | 25% utilisation | 50% utilisation | 100% utilisation |
|---|---|---|---|
| 2 years | $13.15 | $6.58 | $3.29 |
| 3 years | $9.73 | $4.86 | $2.43 |
| 5 years | $6.99 | $3.49 | $1.75 |

Read the diagonal: **owning only beats a cheap rental rate at high utilisation over a long horizon** ($1.75/GPU-hr at 5 years and 100% utilisation, against a verified $1.85 on-demand rental). At 25% utilisation over two years it costs $13.15 per useful GPU-hour — multiples of any rental rate in §4. This is the same utilisation logic as §3.4's Workload B, applied to capital instead of rates.

### 9.3 What the arithmetic leaves out, and why it usually decides

The table above is a hardware comparison. The real comparison includes:

* **People.** A rented GPU needs an account. An owned node needs an OS, drivers, firmware, a scheduler, monitoring, patching, spares, a rack contract and someone on call. That headcount cost is frequently larger than the GPU cost and it is *fixed* — you pay it at 5% utilisation too.
* **Obsolescence.** The rental market reprices a GPU generation down as the next one ships (§4 alone spans V100 to B300). An owned box does not reprice; it just gets slower relative to what you could rent for the same money.
* **Time-to-capacity.** Buying takes weeks-to-months (quote, PO, delivery, install). Renting takes minutes. For a pilot, the procurement lead time is usually the dominant cost — not the rate.
* **Capacity flexibility.** Owned capacity is a single number. A rented fleet can be 4 GPUs today and 32 next week, which matters more than any rate when the workload is uncertain.
* **The hidden benefit of owning: it is always there.** No quota, no capacity crunch, no price change, no residency question. For a workload that must run continuously and cannot tolerate an availability failure, this is worth more than the arithmetic shows — and it is the honest reason a lot of teams own hardware despite the numbers.

### 9.4 The rule

**Break-even = capital ÷ (rental rate × hours) + operating cost ÷ (rental rate × hours), with utilisation as the multiplier on the whole thing.** Compute it with your utilisation, your horizon, and the specific rental rate you would substitute. Then add the people cost, which usually overwhelms the answer. The honest conclusion is that for most teams the answer is determined by *how much of the cost is people rather than silicon* — and renting shifts that cost to someone else at a margin the neoclouds are quite willing to earn.

### 9.5 The route that needs no GPUs at all

Before any of the above, ask whether you need to rent or own an accelerator. A large share of what teams run self-hosted inference for is available as a managed API at a per-token price with no capacity risk, no driver management, no residency problem to solve yourself, and no fixed cost. §4.2 and §4.5 both show the pattern: providers publish per-token inference prices (Crusoe's serverless inference card lists input/output/cached token prices per model; Together's card is entirely per-token) while their self-serve GPU deployments cost more per hour than the raw instance ($5.50/hr for an H100 endpoint against $3.90 for the same H100 as an instance, Crusoe, 20 Sep 2026).

When to use it: spiky or low-volume inference, workloads where the model is available and the data classification permits an external call, and anything where the engineering cost of operating an endpoint exceeds the token bill. When not to: hard residency constraints, proprietary weights you will not send anywhere, latency requirements that need a warm accelerator, or volumes high enough that your own reserved GPU is cheaper per token.

Cross-references for the surrounding architecture rather than the compute: `technology/ai_llm/scalable_ai_deployment.md` owns the deployment architecture; `technology/openshift_ai_alternatives_guide.md`, `technology/charmed_kubeflow_vs_openshift_ai_guide.md`, `technology/nutanix_enterprise_ai_vs_openshift_ai_guide.md` and `technology/ai_llm/vertex_ai_vs_gemini_guide.md` cover the platform layer; `technology/finops_guide.md` and `technology/capacity_sizing_guide.md` cover the sizing and cost-governance disciplines this section only sketches.

---

## 10. The Regulated-Enterprise and Banking Angle

This section is the reason a guide about cheap GPU access needs a section that mostly says "cheap is not the criterion". It contains no recommendation about any institution; it lists the gates.

### 10.1 Data residency — where the GPU actually sits

The GPU's physical location determines the legal regime that applies to the data on its disk, the telemetry around it, and the weights produced from it. The provider classes differ sharply:

| Provider class | Residency control you actually have | Verified specifics |
|---|---|---|
| Hyperscalers | Regions and zones, chosen explicitly; cross-border transfer is a configuration decision | Covered in `technology/cloud_providers_guide.md` |
| Neoclouds | A small number of named regions; some publish NA and EU tables separately | CoreWeave publishes separate NORTH AMERICA and EUROPE price tables (20 Sep 2026), so a region choice exists and is explicit |
| EU regional / bare metal | Datacentre-level control, with contractual residency | Hetzner: "customer master data is not transferred to third countries", data centres certified DIN ISO/IEC 27001 in Germany and Finland, and a GDPR Art. 28 DPA available. Scaleway and OVHcloud are French providers pricing French zones (PAR-1/PAR-2) |
| Marketplaces | **Effectively none.** Capacity is distributed across 40+ data centres and the listing is chosen per-job | Vast.ai's own FAQ tells the user to configure residency-relevant protections themselves: encrypt at rest, use external key management, prefer Secure Cloud |

For a regulated buyer, the middle two rows are usually acceptable and the last row usually is not. That is a gate, applied before price.

### 10.2 Vendor risk and concentration

* **Counterparty size and stability.** Some providers in §4 are large, listed, and heavily financed — CoreWeave's page carries its own Gartner MQ announcement and the company's funding news is on its own site; Crusoe's pricing page carries a $3.9B Series F announcement. Others are small enough that a single funding round or a single anchor tenant determines their existence. Both facts are visible on the pricing pages, which is a useful reminder that *the vendor's own marketing page is a risk-disclosure document if you read it that way*.
* **Concentration risk is a financial-regulation concept, and it applies here.** Using one small provider for all GPU capacity is outsourcing concentration. A second provider that requires a different image format, a different scheduler and a different storage layout is not really a fallback (§5.4, item 6).
* **The exit cost is part of the risk.** Egress is free at some providers (Lambda, Hyperstack, CoreWeave per its own copy; Nebius for compute networking) and metered at others (Nebius object-storage egress at $0.015/GiB). A provider with free egress is materially easier to leave, which reduces the risk of using it. Cross-ref `technology/cloud_object_storage_comparison_guide.md` for the exit-cost discipline.

### 10.3 The procurement reality: a marketplace's terms of service are not an enterprise MSA

This is the single most misunderstood point in the whole area. Specifically, a marketplace ToS typically gives you:

* no negotiated liability cap, indemnity or service credit;
* no audit right over the host that runs your workload;
* no contractual incident-notification clock (a DORA-style regime imposes notification *duties on you*, which makes the absence of a provider commitment a direct compliance problem, not a commercial one);
* no named data processor at the host level, unless the platform has signed a DPA with that host (§6.2 shows Vast.ai's Secure Cloud tier does require signed DPAs — that is the exception that proves the rule);
* no change-of-control or exit assistance clause.

An enterprise MSA is a document your legal and risk functions can put in front of an auditor. A ToS is a click. **The difference between them is not cost — it is existence.** Where a provider *will* sign an MSA (the neoclouds and the EU regionals generally will), and where it will not (peer-to-peer marketplace hosts), determines which class you can use for regulated work regardless of the rate.

### 10.4 Evidence and audit expectations

An auditor asking "where does this model's training data live, and who can access it" needs a chain of evidence, not a statement. That chain looks like: the provider's certification report (read the report, not the badge — §6.3 records that some providers publish reports under NDA and others publish "alignment"), the DPA, the residency commitment, the access-control design, the log-retention period, and a documented deletion process. Where any link is missing — most commonly the host-level DPA and the audit right — the chain breaks and the workload cannot be placed there.

Two practical notes from the research:

* **"Alignment with a framework" is not a certification.** CoreWeave's own page says it aligns its programs with SOC 2, ISO 27001, ISO 27017 and ISO 27018, and that evidence is accessed through its Compliance Documentation Portal. That is a reasonable, honest disclosure — and it is *not* the same statement as "we hold a current SOC 2 Type II report", which is what Vast.ai states for itself and what Hyperstack displays as a badge. Read the exact wording (§6.3) rather than the logo.
* **Where the report is available under NDA, the cost is a legal review, not money.** Budget for it in the procurement timeline.

### 10.5 Model-risk and AI-regulation implications

The choice of where training data and weights are held is no longer an infrastructure footnote:

* **Regulatory breadth is increasing.** Nebius publishes **ISO 42001** (AI management systems) alignment among its certifications and states alignment with **NIS 2** and **DORA** — evidence that AI-specific and financial-sector-specific frameworks are now part of a GPU cloud's compliance surface, not a separate conversation.
* **Weights derived from regulated data may inherit its handling obligations.** If a fine-tune is trained on customer data, where the resulting weights sit and who can access them is a question your model-risk function will ask. A marketplace host is a poor answer.
* **The AI-regulation "where" question is asked more often than the "how fast" question.** Being able to answer "EU data centres, GDPR-compliant, ISO 27001-certified, DPA in place" is worth more in a review than a cheaper rate that cannot be described in those terms.

### 10.6 The honest statement, and its counterpart

**For a bank's production AI workload, the cheapest provider is rarely chosen.** Not because banks are irrational about cost, but because the cheapest provider is structurally the one that cannot supply the audit right, the residency commitment, the incident-notification clock or the named processor — and those are conditions of doing the work at all, not preferences. Where a bank does use a low-cost provider for production, it is typically after the provider has grown into an MSA, a certification set and a residency commitment.

**For experimentation, the cheapest provider often is chosen, and that is correct.** A research sandbox, an evaluation batch, a model-selection experiment: these carry no regulated data, produce no production artefact, and the decision they inform is worth far more than the compute costs. Using a marketplace for that work is not a shortcut around governance — it is the correct application of governance, provided the sandbox genuinely contains no regulated data and nobody quietly promotes a marketplace-trained artefact into production without re-doing it properly.

The failure mode is not using a cheap provider. It is **not knowing which of the two situations you are in.** §11 is the worked version of that distinction.

---

## 11. The Cymbal Bank Worked Example

**Cymbal Bank is fictional.** It is used here to work the decision end-to-end: requirements, gates, arithmetic, and a split recommendation. Every rate is a real rate from §4 with the source date shown; every cost figure is **ILLUSTRATIVE ARITHMETIC** on those rates for a stated workload, not a quote.

### 11.1 The situation

Cymbal Bank's AI team has two jobs to place this quarter:

| # | Workload | Profile | Data classification | Duration |
|---|---|---|---|---|
| **A** | Fine-tuning pilot | 8 GPUs, ~60 GPU-hours one week of piloting (including a restart), 300 GiB dataset in, three 40 GiB checkpoints out (120 GiB total) | Internal, but derived from a **customer-transaction sample** | One week, then repeated monthly if it works |
| **B** | Small inference service | 1 GPU continuously, 730 GPU-hours/month, 150 GB persistent volume for weights and cache, model refreshed weekly | Serves **live customer-facing requests** — highest classification | Ongoing |
| **C** | Evaluation batch | 4 A100-class GPUs for ~40 minutes, twice a week (≈5.33 GPU-hours/month) | **Public benchmark dataset only** | Recurring, disposable |

### 11.2 What each workload requires from the access model

| # | Access requirement | Minimum model (§2) | Reason |
|---|---|---|---|
| A | Persistent volume for checkpoints; the ability to install the team's pinned driver/CUDA stack because the model needs a specific flash-attention build | **True VM (a)** | Kernel-level driver work is on the critical path. Container-only providers are eliminated — this is the §2 filter applied *before* any price is read. |
| B | A pinned, reproducible driver and a warm accelerator; no kernel work once built; a persistent volume; a stable endpoint address | **True VM (a)** preferred, container (b) acceptable if nothing privileged is needed | The service must not restart under someone else's schedule (§5.2) |
| C | Nothing persistent; restartable by construction; runs against public data | **Container (b) or marketplace** | The §8 framework's "evaluation batch" row: this workload should always run on the cheapest interruptible tier |

### 11.3 The residency and tenancy gates — applied before price

These gates eliminate options *outright*, and they are applied first for exactly that reason:

| Gate | A (fine-tune) | B (service) | C (evaluation) | Effect |
|---|---|---|---|---|
| Customer-derived data on the host's disk | Yes — the transaction sample is on the volume | Yes — live requests | **No** — public benchmark data | C is eligible for the cheapest class; A and B are not |
| Weights become a production artefact | Yes (feeds B) | Yes | No — discarded | A inherits B's handling standard |
| Named data processor / DPA required | Yes | Yes | Not required | Peer-to-peer hosts (RunPod Community Cloud, Salad) are eliminated for A and B |
| Audit right and incident-notification clock required | Yes (DORA-style) | Yes | No | Provider must be able to sign (§10.3) |
| Region must be EU/approved jurisdiction | Yes | Yes | Not required | Geographically unconstrained marketplaces are eliminated for A and B |
| Certified datacentre or bare metal acceptable | Either, if certified | Either | Either | — |

**Result of the gates:** for A and B, the cheapest classes in §4 — the marketplaces — are eliminated regardless of their rates. For C, they remain fully eligible. This is the split, and it was determined before a single rate was compared.

### 11.4 The availability check (what a real team would do first)

Per §5.4, before committing: Cymbal's platform team attempts to launch 8×H100 on its first-choice provider on three separate days, records the success rate and the time-to-allocation, and confirms the account's quota ceiling in writing. The illustrative outcome: two of three attempts succeed within 20 minutes, one fails at the requested 8-GPU size and succeeds at 4. **That single observed failure is why the recommendation below includes a second provider rather than one** — not because the second is cheaper, but because the first is not always available.

### 11.5 The arithmetic, with hidden lines (all figures ILLUSTRATIVE on rates checked 20 Sep 2026)

**Workload A — fine-tune pilot, 8 GPUs × 60 GPU-hours.** *ILLUSTRATIVE, based on each provider's own published rate on 20 Sep 2026.*

| Option | Compute | Storage (prorated 60 h) | Egress | Illustrative total | Gate result |
|---|---|---|---|---|---|
| Hyperstack, 8× H100 SXM @ $3.20/GPU-hr | 8 × $3.20 × 60 = **$1,536.00** | SSV, included in compute path | Free (provider states egress/ingress free) | **≈ $1,536** | Eligible |
| DataCrunch, 8× H100 SXM5 @ $3.35/GPU-hr | **$1,608.00** | NVMe 500 GiB @ $0.20/GiB/mo → $8.22 | — | **≈ $1,616** | Eligible |
| RunPod pod, 8× H100 SXM @ $3.49/GPU-hr | **$1,675.20** | Volume disk 500 GB @ $0.10/GB/mo → $4.11 | Free ingress/egress (docs) | **≈ $1,679** | **Eliminated by access model** — container, no custom driver (§11.2) |
| Nebius, 8× HGX H100 @ $3.85/GPU-hr | **$1,848.00** | Block volume 500 GiB @ $0.053/GiB/mo → $2.18; checkpoint egress 120 GiB @ $0.015 → $1.80 | $1.80 | **≈ $1,852** | Eligible |
| Lambda, 8× H100 SXM @ $3.99/GPU-hr | **$1,915.20** | Local SSD included | Free (provider states no egress fees) | **≈ $1,915** | Eligible |
| CoreWeave, 8×H100 node @ $49.24/node-hr | $49.24 × 60 = **$2,954.40** | Extra if retained | Free per provider copy | **≈ $2,954** | Eligible |
| CoreWeave **spot** node @ $19.71/node-hr | **$1,182.60** | Extra if retained | Free per provider copy | **≈ $1,183 + rework risk** | Eligible **only if the job checkpoints** — it does, so this is the cheapest eligible option on the page |
| Marketplace (Vast.ai interruptible, Salad Lowest) | ⚠ live/host-set; salad's RTX 4090 Lowest is $0.160/GPU-hr but is a **consumer card with no NVLink** | n/a | n/a | **not priced — gate fails** | **Eliminated by tenancy gate** |

**The ranking changes twice here, which is the point of the exercise.** The headline-cheapest number on the page (a marketplace rate) is not eligible. The second cheapest (CoreWeave spot) is eligible *only because the job checkpoints*, and it carries a rework cost if a restart lands mid-epoch. The cheapest *unconditional* option is Hyperstack at $1,536 — and note that this is less than the arithmetic total of the CoreWeave on-demand node by nearly half, while the Lambda total sits 25% above it purely on rate, with storage and egress lines roughly washing out.

**Workload B — inference service, 1 GPU × 730 hours, 150 GB volume.** *ILLUSTRATIVE, same date and sources.*

| Option | Compute | Volume/month | Illustrative monthly | Annualised | Gate result |
|---|---|---|---|---|---|
| Hyperstack, H100 SXM @ $3.20/GPU-hr | **$2,336.00** | SSV 150 GB @ $0.000096774/GB/hr × 730 ≈ $10.60 | **≈ $2,347** | **≈ $28,159** | Eligible |
| DataCrunch, H100 SXM5 @ $3.35/GPU-hr | **$2,445.50** | NVMe 150 GiB @ $0.20/GiB/mo = $30.00 | **≈ $2,476** | ≈ $29,706 | Eligible |
| RunPod pod, H100 SXM @ $3.49/GPU-hr | **$2,547.70** | Volume 150 GB @ $0.10 running = $15.00 (**$30.00 if left idle**) | **≈ $2,563** | ≈ $30,753 | Container model; acceptable only if no privileged/driver requirement — Cymbal's does not need one, but the incident-notification and MSA gates do apply |
| Nebius, HGX H100 @ $3.85/GPU-hr | **$2,810.50** | Block volume 150 GiB @ $0.053 = $7.95 | **≈ $2,818** | ≈ $33,821 | Eligible |
| Lambda, H100 SXM 1× @ $4.29/GPU-hr | **$3,131.70** | Local SSD included | **≈ $3,132** | ≈ $37,580 | Eligible |
| CoreWeave, inference single-GPU H100 @ $6.16/hr | **$4,496.80** | Extra | **≈ $4,497** | **≈ $53,962** | Eligible — and ≈$25,800/year more expensive than the cheapest eligible option for the same GPU generation |

The illustrative spread between the cheapest and most expensive *eligible* option for the same class of GPU and the same 730 hours is **≈$2,150/month, ≈$25,800/year** — purely on rate plus a storage line, for a workload where the hardware is comparable. That is the number a price-comparison exercise is actually trying to find, and it only becomes visible after the gates are applied.

**Workload C — evaluation batch, 4 GPUs × 5.33 hours/month.** *ILLUSTRATIVE, same date.*

| Option | Arithmetic | Illustrative monthly |
|---|---|---|
| Salad, 4× RTX 4090 @ $0.160/GPU-hr (Lowest, per-second while running) | 4 × 0.160 × 5.33 | **≈ $3.41** |
| Salad, 4× RTX 4090 @ $0.330/GPU-hr (High priority) | 4 × 0.330 × 5.33 | **≈ $7.04** |
| RunPod A100 SXM pod @ $1.59/hr (per-second) | 4 × 1.59 × 5.33 | **≈ $33.90** |

The whole of Workload C costs **under the price of one engineer-hour per month** in every row. This is the honest reason not to spend two weeks choosing a provider for it: **the decision cost exceeds the compute cost.** Pick the per-second one, put a queue in front of it, and move on.

### 11.6 The recommendation, including the split

**A. Fine-tune pilot.** Run it on a compliant true-VM provider — Hyperstack or DataCrunch are the illustrative least-cost eligible options at $1,536 and $1,616 respectively; Lambda and Nebius are the more expensive eligible options at $1,915 and $1,852. **Use CoreWeave spot only if the checkpointing is demonstrably working and the team has budgeted the restart** — it is the cheapest number on the page and the most expensive one to get wrong. Do **not** run this workload on a marketplace, even though it is cheaper: the dataset is customer-derived and the resulting weights feed a production service.

**B. Inference service.** Put it on the same compliant provider class as A, on a reserved or committed term if the utilisation forecast supports it (§5.3), and keep the volume on the provider's native storage rather than rebuilding a storage layer. The illustrative delta between the cheapest and the most expensive eligible option is ≈$25,800/year for the same GPU generation — worth the procurement effort, and worth paying a modest premium over if the cheaper provider cannot meet the incident-notification and audit gates.

**C. Evaluation batch.** Run it on the marketplace/per-second tier. It is public data, it is disposable, and the monthly cost is single-digit dollars to low double digits. This is exactly the workload marketplaces exist for.

**The split, and what it costs in governance and ops overhead.** The split decision is real and it is not free:

| Overhead of the split | Cost |
|---|---|
| Two provider relationships instead of one | Two sets of credentials, two billing processes, two sets of terms, two contacts for support |
| Two image/artifact paths | The evaluation image and the production image must be kept separate so that nothing built in the sandbox is promoted by accident |
| Governance continuity | A written rule, not an implicit one: **no artifact, weight, dataset or container built in the marketplace tier is ever promoted into production without being rebuilt and re-run on the compliant tier.** This rule is the single control that makes the split safe, and it needs an owner. |
| Cost attribution | Two bills to charge to the right cost centre; a marketplace bill on a corporate card may be harder to attribute than an invoiced contract |
| Data-leakage risk | The genuine one. A well-meaning engineer with a "sample" of transaction data and a cheap GPU account is how the split turns into an incident. The control is classification at the point of upload, not a policy document. |

**What the split buys:** experimentation at marketplace prices, production at compliant-provider prices, and a clear boundary between them. **What it costs:** one written rule, one owner, and the discipline to keep two environments genuinely separate. In this worked example the split is worth doing — the evaluation workload is trivially cheap and completely separable, and the production workload's requirements make the marketplace unusable anyway.

### 11.7 What this example does not decide

It does not decide the provider. Hyperstack, DataCrunch, Lambda, Nebius, Crusoe and CoreWeave are all eligible under the gates as applied here; the arithmetic shows a spread, and the availability check of §11.4 might eliminate the cheapest. **It also does not decide reserved versus on-demand for B** — that requires the utilisation measurement in §5.4 item 1, which this example assumes rather than supplies. That assumption is exactly the kind of thing that makes a real decision different from a worked example, and it is stated here rather than hidden.

---

## 12. The Anti-Patterns

Each entry: **symptom → cause → guardrail.** These are the failure modes this guide's rate tables, access-model matrix and cost model exist to prevent.

### 12.1 Rate comparison that ignored egress

* **Symptom.** A provider is chosen on the lowest GPU-hour rate; the first month's invoice is materially higher than the comparison spreadsheet predicted.
* **Cause.** Egress was treated as a constant. It is not: Hetzner includes unlimited traffic on its GPU servers (with the 10G-uplink exception), Hyperstack and Lambda state egress/ingress free, CoreWeave states no egress/NAT/intra-region charges, and by contrast Nebius meters object-storage egress at $0.015/GiB. Fly.io charges $0.02/GB in NA/EU but $0.12/GB in Africa/India. A dataset-heavy workload moved twice can exceed the rate delta between providers.
* **Guardrail.** Put egress, ingress and cross-region transfer in the comparison spreadsheet as explicit rows, from each provider's own pricing page, with the date. Cross-ref `technology/cloud_object_storage_comparison_guide.md` for the exit-cost discipline.

### 12.2 Persistent volume left attached to a stopped instance

* **Symptom.** A monthly bill with little or no GPU compute on it.
* **Cause.** Storage is billed whether or not the GPU is running, and on at least one provider the idle rate is *higher* than the running rate: RunPod lists volume disk at $0.10/GB/mo running and **$0.20/GB/mo idle** (20 Sep 2026). Fly.io states the same principle in its own words: volumes are "charged for volumes that you create, whether they are attached to a Machine or not, including when an attached Machine is stopped" ($0.15/GB-month). Nebius bills block volumes from $0.0530/GiB/mo regardless of attachment status.
* **Guardrail.** A lifecycle policy with a maximum unattached-volume age, enforced by the platform rather than by intent; and a monthly report of storage spend with zero attached compute.

### 12.3 Spot instance running a job that cannot restart

* **Symptom.** A training run that "keeps dying" and whose cost is quietly close to on-demand despite using spot pricing.
* **Cause.** Preemption kills running processes without warning (Vast.ai's own docs: the instance "is stopped (killing running processes)" and "may wait long to resume"), and a job that cannot resume from a checkpoint re-does work at full price. The discount is real — CoreWeave's H100 node is $19.71/hr spot against $49.24 on-demand — but it is only earned if the workload can absorb the interruption.
* **Guardrail.** Checkpoint frequency as a design requirement, not an afterthought; a measured restart time (§5.4 item 2); and a rule that spot is used only for workloads whose restart cost is below the discount.

### 12.4 Marketplace listing assumed compliant

* **Symptom.** A production or customer-data workload running on a host whose operator nobody has assessed, and a compliance answer that says "we use a certified platform" when the platform's own documentation says provider security "varies significantly".
* **Cause.** Confusing platform-level assurances with host-level tenancy. Vast.ai's Secure Cloud tier does require datacentre partners to sign DPAs and holds SOC 2 Type 2 itself, but its own compliance page notes that partner certifications such as ISO 27001 or SOC 2 are "encouraged … but are not strictly required".
* **Guardrail.** Apply §6.5's eight questions before data moves. Treat "the platform is certified" and "the host that runs my job is in scope" as two different claims.

### 12.5 Cheap provider with no capacity at the moment of need

* **Symptom.** A launch that fails, or succeeds at 4 GPUs when 8 were needed, on the day the experiment was supposed to start.
* **Cause.** A listed rate is not an allocation. Lambda labels its on-demand tier "first-come"; CoreWeave's spot column is N/A for some SKUs; Scaleway's H100/L40S/B300 render as unavailable outside PAR-2; Salad publishes daily evidence that some classes have fewer than ten schedulable units.
* **Guardrail.** Test obtainability at your exact quantity and region on multiple days before you commit (§5.4 item 3); keep a funded second-provider account with a tested image path; and reserve for anything with a deadline.

### 12.6 Dev pod billed for a month because nobody stopped it

* **Symptom.** The largest line on the invoice belongs to nobody's project.
* **Cause.** No auto-shutdown. This is well understood enough that providers ship the fix and document it: Paperspace's Free tier carries an "Auto-shutdown (12-hour limit)" and its paid tiers advertise "configurable" auto-shutdown; Modal's whole pricing model is that you "never pay for idle resources".
* **Guardrail.** Default auto-shutdown on every non-production instance, spend alerts at the account level, and a named owner for every long-lived instance.

### 12.7 Dataset moved twice at egress cost

* **Symptom.** Two egress charges for one dataset, plus a re-download time that appears in the job duration.
* **Cause.** The dataset was staged into provider A, the experiment moved to provider B, and the job was run where the data was not. Egress is sometimes free (Lambda, Hyperstack, CoreWeave per its own copy) and sometimes metered (Nebius object-storage egress at $0.015/GiB; Fly.io from $0.02 to $0.12/GB by region) — so the cost of being disorganised varies by provider and is never zero in engineer time.
* **Guardrail.** Decide the provider before staging the data; keep the canonical copy in the cheapest-to-egress location; and treat "where does the data live" as a prerequisite of "where does the job run".

### 12.8 Commitment signed before utilisation was measured

* **Symptom.** A reserved-capacity bill with visible idle GPU time, defended on the grounds that "we needed the capacity".
* **Cause.** Reservation converts an availability risk into a cost risk (§5.3). Discounts are genuinely attractive — Nebius advertises up to 35% off for multi-month commitments, DataCrunch publishes 8% at one year and 25% at two years — but they only pay if the utilisation materialises.
* **Guardrail.** Measure used-versus-paid GPU-hours first (§5.4 item 1). If that number does not exist, no commitment gets signed. Start with the shortest term available and a published discount ladder rather than the largest headline discount.

### 12.9 The meta-pattern

Every entry above has the same root cause: **treating the rate as the decision.** The rate is one variable in a cost model (§3), constrained by an access model (§2) and an availability reality (§5), and gated by compliance (§6). The nine anti-patterns are what happens when the analysis stops at the first of those.

---

## 13. The Claims Audit

**All claims in this guide were checked on 20 September 2026.** This market reprices faster than any other part of cloud — H100 capacity has moved by 30% or more inside a single quarter, spot markets move hourly, and a marketplace "price" is a live order book rather than a tariff. Re-read the source URLs before relying on any figure.

Verdict legend: **Verified** — the claim was read on the provider's own page at the date shown. **Flagged** — the claim is the provider's own assertion, or an approximation/substitution requiring care. **Rejected** — a claim encountered in research that this guide declines to carry.

### 13.1 Verified

| Claim | Source | Source date | Source quality |
|---|---|---|---|
| **CoreWeave** 8×H100 node: $49.24/hr on-demand, $19.71/hr spot; H200 $50.44/$20.93; B200 $68.80/$34.11; A100 $21.60/$9.65; L40 $10.00; L40S $18.00; GB200 NVL72 $42.00 (spot N/A); GH200 $6.50; inference single-GPU H100 $6.16 (footnote-restricted to inference customers); NA and EU tables both read | coreweave.com/pricing | 20 Sep 2026 | Provider's own pricing page |
| **Lambda** per-GPU rates: B200 SXM6 $6.69 (8×), H100 SXM $3.99, A100 SXM 80GB $2.79, A100 40GB $1.99, V100 $0.79; 1× H100 SXM $4.29; cluster H100 $6.16/$5.85/$5.54 at 16/64/256 GPUs; B200 cluster $9.86/$9.36/$8.87; "no egress fees"; on-demand self-serve described as "first-come" | lambda.ai/pricing | 20 Sep 2026 | Provider's own pricing page |
| **Nebius** per-GPU-hour: HGX H100 $3.85 on-demand / $2.15 preemptible; H200 $4.50/$2.45; B200 $7.15/$3.95; B300 $7.85/$4.30; RTX PRO 6000 $1.80/$0.95; L40S from $1.82/$0.90 (Intel) and $1.55/$0.74 (AMD); GB300/GB200 NVL72 contact-us. Storage: shared FS $0.0800/GiB/mo; WEKA $0.1000; object Standard $0.0147 with egress $0.0150/GiB; block volumes $0.0530/$0.0710/$0.1180; local SSD $0.065; compute egress and ingress free; public IP free; commitment discount "up to 35%" | nebius.com/prices | 20 Sep 2026 | Provider's own pricing page |
| **Nebius** certifications: SOC 2 Type II (with HIPAA), SOC 3, ISO 27001, 27018, 22301, 27032, 27701, 27799, 42001, NIS 2 alignment, DORA alignment, CSA STAR Level 1; at-rest encryption by default with named disk exceptions | nebius.com/trust-center | 20 Sep 2026 | Provider's own trust centre |
| **Crusoe** per-GPU-hour on-demand: H200 $4.29, H100 $3.90, A100 80GB SXM $2.30, A100 80GB PCIe $2.00, L40S $1.50, MI300X $3.45; GB200/B200/MI355X contact-sales; spot column contact-sales only; storage $0.08/$0.07/$0.10/$0.06 per GiB/mo; managed K8s $0.10/cluster-hr; self-serve H100 endpoint $5.50/hr, H200 $6.00/hr | crusoe.ai/cloud/pricing | 20 Sep 2026 | Provider's own pricing page |
| **RunPod** pod rates: B300 $7.89, H200 $4.59, B200 $6.79, H100 SXM $3.49, H100 PCIe $2.89, A100 $1.59, L40S $1.09, RTX 4090 $0.74, A5000 $0.27; serverless worker rates above the pod rate for the same GPU (H100 $4.79); storage container $0.10/GB/mo, volume $0.10 running / **$0.20 idle**, network standard $0.07/$0.05, high-perf $0.14; clusters H200 $4.31, A100 $1.79 | runpod.io/pricing (page states "Updated September 13, 2026") | 20 Sep 2026 | Provider's own pricing page |
| **RunPod** access model: containers and custom containers, but "Docker Compose is not supported", "UDP connections are not supported", "Windows is not supported"; Secure Cloud = T3/T4 datacentres, Community Cloud = "individual compute providers … peer-to-peer"; pods "billed by the minute" | docs.runpod.io/pods/overview | 20 Sep 2026 | Provider's own documentation |
| **Vast.ai** tiers: on-demand "guaranteed uptime" per-second; interruptible "50%+ cheaper", preemptible, bid-price model, "saves 50-80%" in docs, "instance is stopped (killing running processes)", "may wait long to resume", no 24-hour limit like GCE preemptible; reserved "up to 50% off" at 1/3/6 months; 40+ data centres; 68+ GPU types; "Prices are set by the market, not by Vast" | vast.ai/pricing and docs.vast.ai/…/faq/rental-types | 20 Sep 2026 | Provider's own pricing page and documentation |
| **Vast.ai** isolation and compliance: "unprivileged Docker containers", namespaces/cgroups, network/file-system/process isolation, "no shared filesystems between tenants", data destroyed on deletion — **and the concession that "Provider security varies significantly … individual hosts may have less formal security measures"**; SOC 3 on request, SOC 2 Type 2 completed (report under NDA), HIPAA on Secure Cloud with BAAs; Secure Cloud partners must sign DPAs and hold ≥5 flagship-class GPU servers, but their ISO 27001/SOC 2 certifications are "encouraged … but are not strictly required" | vast.ai/compliance and docs.vast.ai/…/faq/security | 20 Sep 2026 | Provider's own compliance page and documentation |
| **Vast.ai** technical constraints: Docker-in-Docker disabled; CUDA version depends on the Docker image; instance storage deleted on destroy while volumes persist at additional cost; slower-than-expected training attributed to CPU bottleneck, network I/O, thermal throttling and PCIe bandwidth | docs.vast.ai/…/faq/technical | 20 Sep 2026 | Provider's own documentation |
| **DataCrunch / Verda** per-hour on-demand/spot: GB300 $9.06/$4.53, B300 $7.88/$3.94, B200 $6.49/$3.24, H200 $4.37/$2.19, H100 SXM5 $3.35/$1.67, A100 80GB $1.74/$0.8675, RTX PRO 6000 $1.86/$0.9300, L40S $1.45/$0.7265, A6000 $0.6040/$0.3020, V100 $0.1760/$0.0880; NVMe $0.20/GiB/mo; reservation ladder 2%/3%/4%/8%/25% at 1/3/6/12/24 months; serverless H100 $3.68/hr; instant clusters with InfiniBand up to 144 GPUs; "SOC 2 and GDPR compliant" stated on the page | datacrunch.io/pricing | 20 Sep 2026 | Provider's own pricing page and its own compliance claim |
| **Hyperstack** per-GPU-hour on-demand / reservation-from: H200 $3.99/$2.79; H100 SXM $3.20/$2.72; H100 NVLink $2.60/$1.82; H100 PCIe $2.50/$1.75 (spot $2.00); B200 $6.00/$5.10; B300 $7.40; RTX PRO 6000 SE $1.85/$1.30 (spot $1.48); A100 SXM $1.60/$1.36; A100 NVLink $1.40/$0.98; A100 PCIe $1.35/$0.95 (spot $1.08); L40 $1.00/$0.70 (spot $0.80); A6000 $0.50/$0.35 (spot $0.40); A4000 $0.15/$0.11; per-minute billing; egress/ingress free; public IP $0.00672043/hr; SSV $0.000096774/GB/hr; Kubernetes master free; SOC 2 Type 2 displayed; publishes a fraud warning about an impersonating domain | hyperstack.cloud/gpu-pricing | 20 Sep 2026 | Provider's own pricing page |
| **Scaleway** L4 instances €0.79/hr (1 GPU) → €6.30/hr (8 GPU), before tax; PAR-1 default view shows L40S, H100 and B300-SXM families as PAR-2-only with no price rendered | scaleway.com/en/pricing/gpu | 20 Sep 2026 | Provider's own pricing page |
| **Hetzner** GEX131: RTX PRO 6000 Blackwell Max-Q, 96 GB GDDR7 ECC, Xeon Gold 5412U, 256 GB ECC (to 768 GB), 2×960 GB NVMe, 1 Gbit/s guaranteed, unlimited traffic, HEL1/FSN1; GEX45: RTX PRO 4000 Blackwell SFF, 24 GB, HEL1. "Full root access", no minimum contract term, no H100 offered, one GPU per server and no multi-GPU configuration; DIN ISO/IEC 27001 datacentres; "customer master data is not transferred to third countries"; GDPR Art. 28 DPA available | hetzner.com/dedicated-rootserver/gex131 and …/matrix-gpu | 20 Sep 2026 | Provider's own product pages |
| **OVHcloud** public-cloud rates: b3-8 $0.0605/hr, c3-16 $0.2154/hr, c3-256 $3.447/hr; monthly estimated at 730 h; from 1 October 2026 local storage and IPv4 billed separately for b3/c3/r3; Savings Plan 12 months −15%, up to 30% off instances and Managed Rancher | ovhcloud.com/en/public-cloud/prices | 20 Sep 2026 | Provider's own pricing page |
| **Salad** per-GPU-hour by priority: RTX 5090 $0.500/$0.417/$0.333/$0.250; RTX 4090 $0.330/$0.273/$0.217/$0.160; RTX 5080 $0.260/$0.223/$0.187/$0.150; RTX 3090 $0.170/$0.143/$0.117/$0.090; classes from $0.015 at Lowest; per-second while running; allocation, image download and cold start free; High "never preempted by other workloads" but subject to node disconnection; Medium/Low minimum run ≈30 minutes; live schedulable counts published (page data dated 17 Sep 2026); CPU $0.005/vCPU-hr + $0.001/GB RAM/hr; community GPUs "in homes and small businesses worldwide" | salad.com/pricing | 20 Sep 2026 | Provider's own pricing page |
| **Modal** per-second: B300 $0.001972, B200 $0.001736, H200 $0.001261, H100 SXM5 $0.001097, RTX PRO 6000 $0.000842, A100 80GB $0.000694, A100 40GB $0.000583, L40S $0.000542, A10 $0.000306, L4 $0.000222, T4 $0.000164; volumes $0.09/GiB/mo with 1 TiB free; region selection 1.15–1.75× base; non-preemptible execution 3× base; SOC 2 on all tiers, HIPAA and audit logs in Enterprise; committed spend usable via AWS/GCP marketplaces | modal.com/pricing | 20 Sep 2026 | Provider's own pricing page |
| **Fly.io** platform prices: volumes $0.15/GB-month "whether or not attached … including when an attached Machine is stopped"; snapshots $0.08/GB-month (first 10 GB free; charges from 1 Jan 2026); stopped Machines charged rootfs only at $0.15/GB per 30 days; egress $0.02/GB NA/EU, $0.04/GB APAC/Oceania/South America, $0.12/GB Africa/India; private cross-region $0.006/GB (NA/EU); static egress IP $0.005/hr; reservation blocks 40% discount; Fly Kubernetes $75/cluster/month | fly.io/docs/about/pricing | 20 Sep 2026 | Provider's own documentation |
| **Paperspace (dba DigitalOcean)** platform tiers: Free (auto-shutdown 12-hour limit, 5 GB storage, public projects), Pro $8/mo (configurable auto-shutdown, 15 GB), Growth $39/mo (50 GB, high-end instances), T0 $0, T1 $12/user/mo, T2 contact sales; persistent storage overage $0.29/GB; NVIDIA Elite partner badge | paperspace.com/pricing | 20 Sep 2026 | Provider's own pricing page |
| **Alibaba Cloud** GPU families and monthly list prices: gn8is L20×8 from $1,279.25/mo, gn7i A10×4 from $676.49/mo, gn6e V100×8 $1,319.93/mo, gn6v V100×8 $884.54/mo, gn6i T4×4 $466.34/mo, gn5 P100×8 $520.9/mo; GPU bare-metal and vGPU families documented; AIACC-Training/Inference, FastGPU and cGPU tools; purchase links point at cn-beijing, cn-wulanchabu, cn-zhangjiakou | alibabacloud.com/en/product/gpu/pricing | 20 Sep 2026 | Provider's own product page |
| **CoreWeave compliance programs**: alignment with SOC 2, ISO 27001, ISO 27017, ISO 27018; evidence via the Compliance Documentation Portal | docs.coreweave.com/security/trust-compliance/compliance-programs (page last modified 10 June 2026) | 20 Sep 2026 | Provider's own documentation |
| **MLPerf** suites and governance: model/dataset/rules "tripod" defined by MLCommons working groups; suites include Training, Training HPC, Inference Datacenter/Edge/Mobile/Tiny, Storage, Endpoints, Client, AlgoPerf, Ailuminate | mlcommons.org/benchmarks | 20 Sep 2026 | Benchmark owner's own site |
| **Vast.ai DLPerf**: provider's own scoring function, ResNet50-style iterations/second, example scores V100 ≈21 / 2080 Ti ≈14 / 1080 Ti ≈10; explicitly less accurate for unusual compute patterns and non-ML workloads | docs.vast.ai/…/faq/rental-types | 20 Sep 2026 | Provider's own documentation |
| **Hyperstack benchmark publications**: provider-authored and dated performance-benchmark articles, including A100 NVLink vs H100 SXM5 LLM inference (20 May 2025), A100 PCIe vs SXM (12 Dec 2024), L40 vs RTX A6000 (4 Apr 2025) and "RDMA, RoCE and Performance Benchmarks in SR-IOV" (1 Jul 2024) | hyperstack.cloud/technical-resources/performance-benchmarks | 20 Sep 2026 | Provider-authored (vendor-measured) |

### 13.2 Flagged — real, but requiring care

| Claim | Why flagged | Date |
|---|---|---|
| CoreWeave "no egress, NAT or intra-region transfer charges" | Read from CoreWeave-adjacent calculator copy rather than an unambiguous line on the pricing table itself. Treat as a provider claim; confirm in your contract. | 20 Sep 2026 |
| Lambda "no egress fees" | Provider's own statement on its pricing page. Straightforward, but still a provider claim rather than an audited term. | 20 Sep 2026 |
| RunPod billing granularity | The pricing page offers a per-second toggle; the docs page says pods are "billed by the minute". Both are RunPod's own material. Confirm in the console/invoice for your product. | 20 Sep 2026 |
| CoreWeave "Inference Single GPU" per-GPU prices | CoreWeave's own footnote restricts these to its inference-platform customers, so they are not a general self-serve rate. | 20 Sep 2026 |
| Modal per-hour figures | Modal prices per **second**; every per-hour figure in §4.5 is this guide's arithmetic and is labelled as such. | 20 Sep 2026 |
| Per-GPU figures derived from node prices (CoreWeave, RunPod clusters) | Derived by division. Labelled as arithmetic; the provider sells the node. | 20 Sep 2026 |
| AWS/Azure/GCP/OCI GPU list prices | Deliberately not quoted here; `technology/cloud_providers_guide.md` owns the hyperscaler comparison and this guide does not restate rates it did not read. | — |
| All §3.4, §9.2 and §11.5 totals | **Illustrative arithmetic** on verified rates. They are not quotes and not predictions. | 20 Sep 2026 |
| Hetzner and Scaleway GPU prices | Hetzner's per-month/per-hour values render client-side; Scaleway's H100/L40S/B300 rates are hidden behind a zone selector. Structure verified, numbers ⚠. | 20 Sep 2026 |
| "Up to 35%" (Nebius), "up to 50%" (Vast.ai reserved), "50%+" (Vast.ai interruptible), "never preempted" (Salad High) | Provider-stated maxima and absolutes. Discount ceilings are rarely the discount you get. | 20 Sep 2026 |

### 13.3 Rejected — claims this guide declines to carry

| Claim encountered | Why rejected |
|---|---|
| Any GPU rate sourced from a comparison blog, an aggregator, or a "GPU price index" site | Rates must come from the provider's own page with a date. Aggregators were used only to *locate* provider pages, never as the source of a figure. |
| Any GPU rate carried from another repository guide | `technology/gain_nvidia_experience.md` holds NVIDIA-experience material including rates; those are stale and this guide supersedes them for pricing. No figure in §3, §4, §9 or §11 is inherited from another guide. |
| "Provider X is SOC 2 certified" where the provider does not say so | Never asserted. §6.3 lists only what each provider publishes, and marks the rest ⚠. |
| Benchmark comparisons used to rank providers | Rejected. §7.4 names the measurer of every benchmark and uses none of them to crown a winner. |
| Derived "effective H100 price" league tables | Rejected — the workloads behind them are not stated, the storage and egress lines are excluded, and the preemption terms are ignored. |
| Any claim that a marketplace host is "iso 27001-certified" because the *platform* says partner certifications "may include" ISO 27001 | Rejected as a misuse of the source. The provider's own wording is that such certifications are "encouraged … but are not strictly required". |
| Any statement that a bank should choose a particular provider | Rejected. This guide produces a cost model, an access-model matrix and a framework, not a recommendation (§1.2, §14). |

### 13.4 The audit's own conclusion

The access-model claims in §2 and §6 are the strongest in this guide: they were read from provider *documentation* — RunPod's own list of what it does not support, Vast.ai's own FAQ on what its isolation does and does not cover, Hetzner's own "Full root access" — and documentation is the honest place to find limits. The **rates are the weakest class of fact in this guide**, because they are the fastest-moving, and every one of them is a timestamped observation rather than a quotation. Re-read them before you commit anything.

---

## 14. What Could Not Be Verified

Stated plainly, because a guide that hides its gaps is worse than one that names them. Everything below is marked ⚠ where it appears in the body.

| # | Item | Status | Why | Effect on the guide's conclusions |
|---|---|---|---|---|
| 1 | **Hetzner GEX45/GEX131 prices** | ⚠ Not verified | The per-month, per-hour and setup-fee values render client-side and did not resolve on either page on 20 Sep 2026. The price *structure* (monthly + hourly + one-off setup) is verified; the numbers are not. | No Hetzner figure appears anywhere in this guide. Hetzner's role in §4.4 and §6 is as an access-model and residency example, not a price example. |
| 2 | **Scaleway L40S, H100 and B300-SXM hourly rates** | ⚠ Not verified | The default PAR-1 zone view renders them as "not available in this zone — available in PAR-2" without prices. | §4.4 quotes only the L4 rates that were readable. The finding that the interesting GPUs are zone-restricted is carried as a finding, not as a price. |
| 3 | **OVHcloud GPU instance prices** | ⚠ Not verified | The prices page as fetched priced the CPU families (b3/b2/c3/c2/r3); GPU families were not in the extracted content. | §4.4 uses OVHcloud for the VM access model and the 1 October 2026 storage/IPv4 billing change, not for a GPU rate. |
| 4 | **Together AI GPU cluster per-hour rates** | ⚠ Not verified | The pricing page's GPU Clusters section did not render a rate table; the announcement feed advertises on-demand B200 clusters with no published price. | Together appears as a provider whose compute tier is not publicly priceable. Its per-token inference prices are public and are treated as such. |
| 5 | **Vast.ai per-GPU rates** | ⚠ Deliberately not carried | Rates are live, supply-and-demand set; the provider itself states prices vary by availability, performance and provider settings. | §4.3 explains the rental tiers and mechanics in the provider's own words and states no number. This is a considered position, not a research failure. |
| 6 | **Voltage Park, Fluidstack, Genesis Cloud rates and access models** | ⚠ Not verified | Genesis Cloud's pricing page failed to render; the others were not verifiable from provider-owned pages within the research budget. | All three are named in §4.2 as unverified candidates. **No rate from any of them appears in this guide.** |
| 7 | **TensorDock rates and access model** | ⚠ Not verified | The pricing URL returned 404 on 20 Sep 2026. | Named in §4.3 as unverified; no figure carried. |
| 8 | **Gcore GPU rates and access model** | ⚠ Not verified | The `/cloud/gpu` path returned 404; the navigation confirms GPU Cloud and AI Cloud products exist, without a rate. | Named in §4.4 as unverified; no figure carried. |
| 9 | **Paperspace per-GPU hourly rates** | ⚠ Not verified | The pricing page's Compute/Usage tab content was not readable; only the platform-plan layer (Free/Pro/Growth/T0/T1/T2) was extracted. | §4.4 carries the platform tiers, the auto-shutdown design and the $0.29/GB storage overage, and states explicitly that the GPU rates were not readable. |
| 10 | **Fly.io GPU rates** | ⚠ Not verified | No GPU line was present in the pricing content fetched. The platform charges around a GPU app (volumes, snapshots, egress, IPs) are verified. | §4.4 carries the platform charges and treats Fly's access model as (b)/(d) pending verification. |
| 11 | **Baseten GPU-hour or per-token rates** | ⚠ Not verified | Not read from a provider-owned page in this pass. | Named in §4.5 as a managed-endpoint provider with no rate asserted. |
| 12 | **Tencent Cloud, Huawei Cloud, Volcano Engine GPU rates, access models and certifications** | ⚠ Not verified | Not read from provider-owned documentation in this pass. | §4.6 names them as providers of GPU capacity and asserts nothing else about them. |
| 13 | **AutoDL prices and access model** | ⚠ Not verified | Not readable from the provider's own pages within this pass. | Named in §4.6 as unverified; no figure carried. |
| 14 | **Lambda's published SLA terms and certification list** | ⚠ Not verified | Not read from Lambda's legal/trust pages in this pass. | §4.2 and §6.3 state that no certification claim was verified for Lambda and decline to assert one. |
| 15 | **CoreWeave's specific certification statuses (as distinct from framework alignment)** | ⚠ Partially verified | The docs page states alignment with SOC 2, ISO 27001, ISO 27017 and ISO 27018 and points to the Compliance Documentation Portal; the certification status PDF is redacted and the portal requires a request. | §6.3 records the provider's exact wording and flags it as alignment-plus-portal rather than a badge. |
| 16 | **RunPod's certification status** | ⚠ Not verified | Not asserted on its public pricing or pods docs. | §6.3 records the Secure Cloud description as a description, not an attestation. |
| 17 | **Scaleway, OVHcloud, Gcore, Salad, Together, Crusoe, Alibaba Cloud certification claims** | ⚠ Not verified | Not read from provider-owned trust pages in this pass. | §6.3 explicitly instructs the reader not to assert any certification for these on the strength of this guide. |
| 18 | **A published preemption notice window for CoreWeave spot and Nebius preemptible** | ⚠ Not verified | Neither provider's pricing page states a notice period on the date checked. | §5.2 states the absence honestly and advises assuming reclaim can occur at any time. |
| 19 | **Exact per-second vs per-minute billing granularity across every marketplace product** | ⚠ Partially verified | Where a provider publishes two different statements (RunPod's pricing page and docs), both are recorded rather than one being chosen. | §3.2 and §4.3 flag the inconsistency. The ranking caveat in §3.2 stands regardless. |
| 20 | **Independently measured performance of these providers on a common workload** | ⚠ Not available | No third-party, same-workload benchmark covering this provider set was located. MLPerf covers systems, not this commercial shortlist; the rest are vendor-authored. | §7.4 names the measurer of every benchmark and §7.5 concludes that your own measurement is the only one that decides your bill. |
| 21 | **Spot discount percentages for providers whose spot column is contact-sales only (Crusoe, some CoreWeave SKUs, Nebius NVL72)** | ⚠ Not verified | The column exists but resolves to "Contact sales". | §4.2 explicitly warns against assuming one provider's discount ratio transfers to another. |
| 22 | **Any rate for a provider not named in this guide** | Not attempted | Out of scope. | The §4.7 class table is the generalisation; the specific providers are the worked examples. |

**What this list means in practice.** The guide is strong on access models, cost structure, availability mechanics and governance gates — the parts that are read from documentation — and deliberately incomplete on rates, where it carries only what it could read from a provider's own page on the date shown, and marks everything else ⚠. If you need a number that is not in §4, the correct action is to open the provider's pricing page yourself, note the date, and treat it as an observation.

---

## 15. Glossary, Cross-References and Closing Summary

### 15.1 Glossary

| Term | Definition as used in this guide |
|---|---|
| **Access model** | *How* a provider gives you an accelerator: true VM (a), restricted container (b), managed notebook (c), or serverless endpoint (d). The discriminator of this guide (§2). |
| **Bare metal** | A whole physical server with no hypervisor between you and the silicon. Strongest access model, least elasticity, and you own the patching. |
| **Billing granularity** | The unit a provider rounds to: per-second, per-minute, or hourly minimum. Decides the cost of any job shorter than an hour (§3.2). |
| **Bid price** | The price you set on a marketplace's interruptible tier; a higher bid takes your instance away (Vast.ai's model, §5.2). |
| **Cold start** | The delay before a serverless worker begins serving. Free on some providers (Salad explicitly does not bill allocation, download or cold start); billed on others. |
| **Committed / reserved capacity** | Capacity held for you against a term commitment, at a discount (Nebius up to 35%; DataCrunch 2–25% by term; Hyperstack "reservation from" rates). Converts availability risk into cost risk (§5.3). |
| **Derived per-GPU rate** | This guide's arithmetic dividing a provider's node price by its GPU count. Always labelled as arithmetic (§1.5). |
| **DLPerf** | Vast.ai's own deep-learning performance score for comparing listings on its platform. Provider-defined, not an independent benchmark (§7.4). |
| **Egress** | Data leaving the provider's network. Free at some providers, metered at others (Nebius $0.015/GiB for object storage; Fly.io $0.02–$0.12/GB by region). A first-class line in the cost model (§3.1). |
| **GPU-hour** | One accelerator for one hour. On neoclouds, priced per GPU and often sold per node; the two are not interchangeable (§1.4). |
| **Instance family** | The configuration behind a rate: GPU model, VRAM, vCPUs, RAM, local storage, interconnect. Two instances at the same GPU rate are not the same product (§7.2). |
| **Interconnect** | The fabric between GPUs and nodes: NVLink within a node, InfiniBand or Ethernet between nodes. Bounds multi-node and model-parallel work (§7.2). |
| **MIG (Multi-Instance GPU)** | One physical GPU partitioned into isolated slices, each sold or shared as a device (RunPod lists Pro 6000 MIG 48GB and MIG 24GB tiers). You get VRAM and compute fractions, not the full card or the fabric. |
| **Neocloud** | A GPU-first provider selling clusters rather than a broad catalogue (CoreWeave, Lambda, Nebius, Crusoe, Together). Better rate, thinner everything else. |
| **Marketplace** | An aggregator listing capacity owned by third parties (Vast.ai, RunPod Community, TensorDock, Salad). Lowest rates, weakest tenancy (§6.2). |
| **MSA (Master Services Agreement)** | The negotiated contract a regulated buyer needs. A marketplace terms-of-service is not one, and the difference is existence rather than cost (§10.3). |
| **On-demand** | Immediate, cancellable, highest rate, no capacity guarantee — and the tier where a listed price is least likely to be obtainable (§5.1). |
| **Preemption** | The provider reclaiming an instance. Harmless with checkpointing, fatal for stateful services (§5.2). |
| **Persistent volume** | Storage that survives instance termination, and that is billed while the instance is stopped or idle — at a higher rate than while running on at least one provider (§3.1, §12.2). |
| **Quota** | An account-level cap on allocation. A product constraint that presents as a billing one, and the most common cause of a slipped schedule (§4.1). |
| **Residency** | Where the data and the GPU physically sit, and therefore which law applies. A gate, applied before price (§10.1). |
| **Spot / interruptible / preemptible** | The discounted tier that grants the provider a reclaim right. The discount is compensation for the reclaim, not a pricing error (§1.4, §5.2). |
| **SXM vs PCIe** | GPU form factors. SXM parts carry the NVLink fabric; PCIe parts do not. Priced as separate SKUs (§7.2). |
| **Tenancy** | Who else is on the hardware, under what isolation, and who operates the host. Container isolation protects you from other tenants; it does not protect you from the host operator (§6.1). |
| **Utilisation** | Used GPU-seconds divided by paid GPU-seconds. The multiplier on every cost model in this guide (§3.4, §9.2). |
| **VM access** | Root, SSH, your own kernel and drivers, your own container runtime and scheduler. §2.1's five questions are the test. |

### 15.2 Cross-references

| Guide | It owns | This guide's relationship to it |
|---|---|---|
| `technology/gain_nvidia_experience.md` | The NVIDIA learning pathway | **Its stored rates are stale; §4 of this guide supersedes them.** Use that guide for what to learn, this one for what it costs. |
| `technology/cloud_providers_guide.md` | The hyperscaler comparison | §4.1 here is deliberately brief and defers to it. |
| `technology/gpu_optimization_guide.md` | Performance technique — kernels, throughput, parallelism | §7 here identifies that a host can starve a GPU; that guide owns the fix. |
| `technology/hami_gpu_sharing_guide.md` | GPU sharing and partitioning (MIG, time-slicing, HAMi) | §1.4 defines MIG as a billing concept (you may be sold a slice); that guide owns the mechanics. |
| `technology/nvidia_dcgm_guide.md` | GPU observability and telemetry | §7.3 requires achieved-utilisation measurement; that guide owns the instrumentation. |
| `technology/on_prem_llm_deployment_guide.md` | Self-hosting on your own hardware | §9 here does the rent-versus-own arithmetic at the boundary and defers the on-prem route to it. |
| `technology/ai_llm/scalable_ai_deployment.md` | Deployment architecture for AI/LLM systems | §9.5 points to it for the architecture above the compute line. |
| `technology/cloud_object_storage_comparison_guide.md` | Storage cost, egress and exit | §3.1, §6.4, §10.2 and §12.1 defer to it for the storage-side treatment. |
| `technology/finops_guide.md`, `technology/capacity_sizing_guide.md` | Cost governance and sizing discipline | §9.4 points to them for the disciplines this guide sketches. |
| `technology/openshift_ai_alternatives_guide.md`, `technology/charmed_kubeflow_vs_openshift_ai_guide.md`, `technology/nutanix_enterprise_ai_vs_openshift_ai_guide.md`, `technology/ai_llm/vertex_ai_vs_gemini_guide.md` | The AI platform layer | §9.5 points to them for the platform decision around the GPU. |

### 15.3 Closing summary

The cheapest GPU cloud provider with VM access is not a fact, it is a function of four variables: **what access model the workload requires, what the workload's utilisation will be, what the capacity actually available on the day will be, and which jurisdiction the data may sit in.** Change any one of them and the answer changes — which is why this guide is a cost model (§3), an access-model matrix (§2) and a workload-keyed framework (§8), and not a league table.

The three claims worth carrying out of it:

1. **The headline GPU-hour rate is the least informative number on the page.** The bill is the rate times the hours you pay for — including storage while stopped, egress, images and snapshots — at the granularity the provider rounds to, and at the utilisation you actually achieve. §3.4, §11.5 and the rent-versus-own arithmetic in §9.2 all show the same thing: the ranking changes when the hidden lines and the granularity enter.
2. **The access model is the discriminator, and the cheap end of the market is disproportionately container-only.** A price-marketed provider frequently cannot give you root, a kernel module, your own driver, your own scheduler or a multi-node cluster. Filter on §2.2's five tests before reading any price column.
3. **A listed rate is not a guarantee of access.** Availability is often the binding constraint, "Contact sales" increasingly replaces a price, and the discount tiers (spot, interruptible, Lowest priority) are cheap precisely because the capacity can be taken back. Measure utilisation, measure restart time, test obtainability at your quantity, then sign.

The market reprices faster than any other part of cloud, every figure in this guide is a point-in-time reading from a provider's own page on 20 September 2026, and where a rate could not be read it is marked ⚠ rather than estimated. Verify the rates; keep the model. Because what a GPU costs per hour is a headline, and what the GPU-hour actually costs.
