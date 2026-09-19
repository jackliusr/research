# Cloud Object Storage Compared — The Cost of the Exit

**Author:** Jack Liu Shurui — Solution Architect
**Last Updated:** September 2026
**Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
**Prices, allowances, durability figures and compliance claims checked:** 19 September 2026
**Regions priced:** AWS `us-east-1` (US East, N. Virginia); Google Cloud `us-central1` (Iowa); Azure Blob (US dollar tables, locally-redundant storage); Oracle Cloud Infrastructure list pricing (worldwide); Cloudflare R2, Backblaze B2, Wasabi, DigitalOcean Spaces (all default list pricing). Every object-storage price is region-dependent and every provider above states so on its own pricing page; the numbers here are one region's worth, not a global truth.

---

## Table of Contents

1. [Overview, Thesis and Decoder](#1-overview-thesis-and-decoder)
2. [The Cost Model Decomposed](#2-the-cost-model-decomposed)
3. [Worked Arithmetic: Six Workload Archetypes](#3-worked-arithmetic-six-workload-archetypes)
4. [The Egress Axis](#4-the-egress-axis)
5. [The Provider Line-Up](#5-the-provider-line-up)
6. [The Feature Matrix That Actually Decides](#6-the-feature-matrix-that-actually-decides)
7. [The S3-Compatible API Reality](#7-the-s3-compatible-api-reality)
8. [Durability, Availability and the SLA](#8-durability-availability-and-the-sla)
9. [Lock-In and Exit Analysis](#9-lock-in-and-exit-analysis)
10. [Performance and Access-Pattern Reality](#10-performance-and-access-pattern-reality)
11. [The Regulated-Enterprise and Banking Angle](#11-the-regulated-enterprise-and-banking-angle)
12. [The Cymbal Bank Worked Example](#12-the-cymbal-bank-worked-example)
13. [The Anti-Patterns](#13-the-anti-patterns)
14. [The Claims Audit](#14-the-claims-audit)
15. [What Could Not Be Verified](#15-what-could-not-be-verified)
16. [Glossary, Cross-References and Closing Summary](#16-glossary-cross-references-and-closing-summary)

---

## 1. Overview, Thesis and Decoder

### 1.1 The thesis

In object storage the price per GB is the least important number on the page. Every serious provider lands within a factor of roughly three of every other provider on the storage line, and the storage line is the one number every buyer already looks at. The money is in the cost *structure* — what is charged per what unit, what free allowance exists, what minimum duration applies, what retrieval fee sits behind an archive class, what a request costs, and above all what leaving costs. And the mechanism that turns cost structure into lock-in is egress: the price you pay to take your own data out.

This guide is therefore not a ranking. It is a cost model, a feature matrix and a framework. There is no winner to crown, because the correct answer changes with the access pattern, the residency constraint, and the exit obligation — and two of those three are not commercial.

### 1.2 Why the cheapest-looking number is the most dangerous one

Three structural facts drive the whole analysis, all of them confirmed on the vendors' own pages on 19 September 2026:

1. **The storage line is tiered differently across vendors.** AWS publishes three volume tiers for S3 Standard ($0.023/GB-month for the first 50 TB, $0.022 for the next 450 TB, $0.021 above 500 TB, per the AWS Price List API) and Azure publishes the same three-tier shape for Hot. Google publishes a single hourly rate per class. Cloudflare R2, Backblaze, Wasabi and DigitalOcean publish a single flat per-GB rate. A tier boundary is a cliff, and a volume commitment is a bet on your own growth.
2. **The unit is not the same unit.** AWS, Google and Azure bill storage in binary gibibytes (AWS states "1 GB is 2^30 bytes … also known as a gibibyte"; Azure states the same). Backblaze, Wasabi and Cloudflare quote decimal GB/TB (Backblaze: "$6.95 / TB / mo"; Wasabi: "$7.99 TB/mo ($0.0078 GB/mo)"). The same physical estate therefore produces a bill that differs by about 9.6% before any rate difference at all — 500 TiB is 549,756 decimal GB but only 512,000 GiB.
3. **The number that decides is usually not on the pricing table you read first.** It is the retrieval fee, the minimum storage duration, the per-object monitoring fee or the egress rate, and none of those live on the headline.

### 1.3 Decoder — the terms that carry the money

| Term | What it is | Why it costs or saves money |
| --- | --- | --- |
| **Bucket** | A container namespace for objects, created in a region, with a globally unique name | A bucket is the unit of region choice, policy, logging, replication and often of billing reporting. Some providers cap bucket counts (R2: 1,000,000 per account; DigitalOcean Spaces: 100 by default per the Spaces limits page) |
| **Object** | Immutable blob + key + metadata, addressed by key | Object *count* is a billing dimension in more places than buyers expect: minimum billable object sizes, per-object monitoring fees, per-1,000-object index charges |
| **Key** | The object's name; the only "path" object storage has | Key layout determines list performance, partition parallelism (AWS documents 3,500 PUT/s and 5,500 GET/s *per prefix*) and lifecycle rule expressiveness |
| **Storage class / tier** | A service level for durability, availability, latency, retrieval time and minimum duration | Tiers are where the vendor encodes the trade: cheap storage with retrieval fees and minimum durations, or expensive storage with none |
| **Durability** | Modelled probability that stored data is not lost over a year. AWS, Google, Azure (LRS/ZRS), Backblaze and Wasabi all publish 11 nines on some or all classes | It measures hardware-level loss only. It does not cover deletion, ransomware, account compromise, billing suspension or the vendor ceasing to exist |
| **Availability** | The fraction of time the service answers requests successfully; this is what the SLA actually underwrites | Availability is per-class: Google commits 99.95% for Standard in multi/dual-region, 99.9% regional, and only 99.0% for Nearline/Coldline/Archive in a region |
| **Request class** | Vendors sort operations into classes (A/B; Tier1/Tier2; write/read/list) with very different prices | Backblaze and Wasabi charge nothing for API requests; R2 charges $4.50 per million Class A; AWS charges $0.005 per 1,000 PUT-class requests. For small-object workloads this line can exceed the storage line |
| **Egress** | Data transferred out to the internet, or between regions/AZs | The lock-in mechanism. R2, Wasabi and Backblaze market zero or capped egress; hyperscalers bill it by volume tier |
| **Retrieval fee** | A per-GB charge for reading data back out of a cold class, on top of the egress charge | Google: $0.01/GiB Nearline, $0.02 Coldline, $0.05 Archive. AWS: $0.01/GB Glacier Flexible standard retrieval, $0.02/GB Glacier Deep Archive standard retrieval |
| **Minimum storage duration** | A minimum billable lifetime per object, charged pro-rata if you delete earlier | AWS Standard-IA/One Zone-IA 30 days, Glacier Instant Retrieval and Glacier Flexible 90 days, Glacier Deep Archive 180 days; Google Nearline 30 / Coldline 90 / Archive 365; Azure Cool 30 / Cold 90 / Archive 180; R2 Infrequent Access 30 days; Wasabi 90 days by default |
| **Free allowance** | Included usage before billing starts | R2: 10 GB-month + 1M Class A + 10M Class B; Cloud Storage Always Free: 5 GB-months Standard + 5,000 Class A + 50,000 Class B; Backblaze: first 10 GB free; Oracle: first 10 TB/month of outbound transfer free; AWS/Azure: first 100 GB/month of internet egress free |
| **S3-compatible API** | A service that implements (some of) the Amazon S3 REST API | A spectrum, not a boolean. It lowers migration cost materially and does not make providers interchangeable — see §7 |
| **Multipart upload** | Splitting a large object into parts uploaded in parallel, then completed | The mechanism that makes large-object throughput possible; part-size floors and completion request charges vary. R2 caps parts at 10,000 and object size at 5 TiB; Oracle caps at 10,000 parts, 50 GiB per part, 10 TiB per object |
| **Object Lock** | WORM protection: retain-until-date and/or legal hold on a version | The capability that decides whether a provider can host a regulatory archive at all. Which regimes each vendor's own documentation claims is in §6 |

### 1.4 Boundary — what this guide owns, and what it deliberately does not

This guide owns: the decomposition of object-storage cost; the egress axis and the zero-egress challengers with their conditions; the cross-provider feature matrix of capabilities that actually decide a migration; durability/availability/SLA interpretation; the lock-in and exit analysis; migration mechanics; and the selection framework.

Three near-neighbour guides in this repository already own adjacent ground, and this guide cites rather than re-derives them:

- [`technology/s3_architecture_guide.md`](s3_architecture_guide.md) (748 lines) owns Amazon S3's data model and internals, durability and consistency mechanics, the storage classes in depth, and S3 security and performance design. This guide uses S3 rates and structures as *inputs to a cross-provider comparison* and does not restate its internals.
- [`technology/cloud_object_storage_lakehouse_guide.md`](cloud_object_storage_lakehouse_guide.md) (1008 lines) owns lakehouse architectures, the on-premises/private S3-compatible field in its §5 (MinIO, Dell ObjectScale, IBM COS, NetApp StorageGRID, Ceph RGW, Pure FlashBlade, VAST) and the API-compatibility tiering in its §13. This guide **extends** the provider line into the commercial challengers and does not repeat its hyperscaler storage-class tables.
- [`technology/dell_objectscale_guide.md`](dell_objectscale_guide.md), [`technology/storagegrid_guide.md`](storagegrid_guide.md) and [`technology/cephfs_alternatives_guide.md`](cephfs_alternatives_guide.md) own their own products.

In scope and not covered elsewhere in the repo: the egress economy, the challenger cost structures, exit cost modelling and the selection framework.

### 1.5 Method and honesty rules used throughout

- Every rate, allowance, durability figure and compliance claim in this guide was read off a vendor-controlled page (pricing page, price-list API, SLA, product documentation or compliance page) on **19 September 2026**, and the URL is given where the figure appears.
- Where a vendor's pricing page renders its rate tables with client-side JavaScript (AWS and Azure both do), the guide uses the vendor's own machine-readable price source — for AWS, the AWS Price List API file `pricing.us-east-1.amazonaws.com/offers/v1.0/aws/AmazonS3/current/us-east-1/index.json`, last modified 18 September 2026.
- **Storage structure is verified separately from rate.** Every provider subsection states the structure (what is charged per what unit, allowances, minimum durations, retrieval fees) as a first-class fact, because structure survives a price change and a rate does not.
- No figure is carried over from memory, from another guide in this repository, or from a third-party comparison article. Where a figure could not be read, it is marked ⚠ and listed in §15.
- **All worked arithmetic in §3, §9 and §12 is illustrative arithmetic**, built from rates checked on 19 September 2026, on stated workload assumptions. It is a model, not a quote.

---

## 2. The Cost Model Decomposed

### 2.1 Four cost components, and the question each one asks

Object storage bills decompose into four components. Every vendor prices a different subset, in a different shape, and that shape — not the rate — is what a buyer has to model.

| # | Component | Unit it is charged on | The question to ask |
| --- | --- | --- | --- |
| 1 | **Storage** | GB-month or GiB-month; sometimes tiered by volume | Is the rate flat, tiered, or capacity-committed? Which unit — decimal or binary? Is there a minimum monthly spend? |
| 2 | **Requests / operations** | Per 1,000 or per 10,000, by class | Which class is my workload in? Am I read-heavy, write-heavy or list-heavy? Is listing charged? |
| 3 | **Egress** | Per GB transferred out | What is included, what triggers a charge, and what waives it? (§4) |
| 4 | **Retrieval and minimum-duration penalties** | Per GB retrieved; per GB-month pro-rated for early deletion | Does my tier have a retrieval fee, a minimum duration, a minimum billable object size, or a per-object index fee? |

### 2.2 How vendors structure component 1 (storage)

| Structure | Providers observed on 19 Sep 2026 | What it means commercially |
| --- | --- | --- |
| Multi-tier volume pricing on the same class | AWS S3 Standard ($0.023 first 50 TB / $0.022 next 450 TB / $0.021 over 500 TB); Azure Hot ($0.0184 / $0.0177 / $0.0170); DigitalOcean Spaces (250 GiB included, $0.02/GiB beyond) | Effective rate improves with scale; the benefit is invisible below the first boundary |
| Flat rate per class, per region | Google Cloud Storage (hourly rates × class); Oracle OCI ($0.0255/GB-month Standard, $0.01 Infrequent Access, $0.0026 Archive) | Trivially predictable; the only lever is which class you put data in |
| Single flat rate, no classes | Cloudflare R2 ($0.015 Standard, $0.01 IA); Backblaze B2 ($6.95/TB/month); Wasabi ($7.99/TB/month) | The whole optimisation is "don't use archive classes because we don't have any"; storage cost is flat and egress becomes the deciding line |
| Reserved / committed capacity | Azure Storage Reserved Capacity (100 TB or 1 PB blocks, 1- or 3-year); Backblaze annual capacity commitments; Wasabi Reserved Capacity Storage (1/3/5-year); Oracle, AWS and Google via private pricing | Discount in exchange for a forward commitment — a lock-in instrument in its own right (§9) |

### 2.3 How vendors structure component 2 (requests)

The observed classes are genuinely different, not cosmetic:

| Provider | Write-class price | Read-class price | List price |
| --- | --- | --- | --- |
| AWS S3 (Standard) | $0.005 per 1,000 PUT/COPY/POST/LIST | $0.0004 per 1,000 GET/all other | charged at the PUT rate |
| AWS S3 Glacier Flexible / Deep Archive | $0.03 per 1,000 PUT | $0.0004 per 1,000 GET | PUT rate |
| Google Cloud Storage (Standard, regional) | Class A $0.005 per 1,000 | Class B $0.0004 per 1,000 | Class A |
| Google Archive class | Class A $0.05 per 1,000 | Class B $0.05 per 1,000 | Class A |
| Azure Blob Hot (LRS table) | $0.05 per 10,000 writes | $0.004 per 10,000 reads; $0.05 per 10,000 list/create-container | listed separately |
| Azure Blob Archive | $0.10 per 10,000 writes | $5 per 10,000 reads ($50 high priority) | $0.05 per 10,000 |
| Oracle OCI Object Storage | $0.0034 per 10,000 requests (single class for all requests) | same | same |
| Cloudflare R2 | Class A $4.50 per million ($9.00 IA) | Class B $0.36 per million ($0.90 IA); deletes free | Class A |
| Backblaze B2 | free (Class A, B, C); Class D outbound calls $0.004 per 10,000 after 2,500/day free | free | free |
| Wasabi | free, subject to the free-API-request policy | free | free |

Two conclusions follow immediately and both are structural, not rate-level. First, a workload that writes or lists heavily pays a per-request tax on hyperscalers and pays nothing on Backblaze or Wasabi. Second, cold classes invert the read price: Azure Archive reads cost $5 per 10,000 versus $0.004 on Hot — a factor of 1,250 — and Google Archive charges read-class operations at the same $0.05 per 1,000 as its write class.

### 2.4 How vendors structure component 3 (egress)

| Shape | Providers | Structural note |
| --- | --- | --- |
| Volume-tiered per-GB egress, first 100 GB/month free | AWS, Azure, Google | The rate falls as you use more, which is precisely the wrong incentive for an exiting customer: you lose the discount exactly when you leave |
| Free egress, no conditions on volume, conditions on *service* | Cloudflare R2 | Free when egressing directly from R2 via the S3 API, Workers API or `r2.dev`; connecting other metered Cloudflare services can cost (§4.3) |
| Free egress up to a multiple of stored data, then a low per-GB rate | Backblaze B2 | Free up to 3× average monthly storage; beyond that $0.01/GB; unlimited free egress to partner CDNs and compute (naming Fastly, Cloudflare, bunny.net, CacheFly, CoreWeave, Equinix Metal, Vultr, phoenixNAP) |
| Free egress, subject to a fair-use policy | Wasabi | Egress and API requests are unpriced, but the policy reserves the right to limit or suspend a use case that imposes an "unreasonable burden" (§4.4) |
| Free first 10 TB/month, then geography-dependent | Oracle OCI | Outbound originating in North America/Europe/UK: first 10 TB/month free, then $0.0085/GB; APAC/Japan/South America $0.025/GB; Middle East/Africa $0.05/GB |
| Included transfer allowance per subscription | DigitalOcean Spaces | 1 TiB outbound included per $5/month subscription; $0.01/GiB beyond; free in listed Spaces→Droplets and VPC-private paths |

### 2.5 How vendors structure component 4 (retrieval and minimum-duration penalties)

| Provider | Minimum billable object size | Minimum storage duration | Retrieval fee |
| --- | --- | --- | --- |
| AWS | 128 KB for Standard-IA, One Zone-IA and Glacier Instant Retrieval; 40 KB per-object metadata overhead for Glacier Flexible and Deep Archive | 30 d (IA/ZIA), 90 d (Glacier IR, Glacier Flexible), 180 d (Deep Archive) | $0.01/GB SIA/ZIA; $0.03/GB Glacier IR; $0.01/GB Glacier Flexible standard / $0.00 bulk; $0.02/GB Deep Archive standard / $0.0025 bulk |
| Google | none published for Standard; Archive read operations are charged at Class B $0.05 per 1,000 | 30 d Nearline, 90 d Coldline, 365 d Archive | $0.01/GiB Nearline, $0.02 Coldline, $0.05 Archive |
| Azure | none published for Hot/Cool/Cold/Archive | 30 d Cool, 90 d Cold, 180 d Archive | $0.01/GB Cool, $0.03 Cold, $0.02 Archive ($0.10 Archive high priority) |
| Cloudflare R2 | none published | 30 d Infrequent Access | $0.01/GB on Infrequent Access |
| Backblaze B2 | none — "No minimum file size or storage duration fees" | none | none |
| Wasabi | 4 KB (files below 4 KB are billed as 4 KB) | 90 days by default (30-day policy available; applies to objects uploaded after the change) | none |
| DigitalOcean Spaces | 4 KiB standard; 128 KiB Cold | Cold 30 days | Cold $0.01/GiB, waived up to average daily usage |
| Oracle OCI | none published | none published | $0.01/GB on Infrequent Access |

The minimum-storage-duration mechanism is a *penalty*, not a rule: delete early and you are billed as if you had not. AWS states it plainly — objects deleted before the minimum "incur the normal storage usage charge plus a pro-rated charge for the remainder of the minimum storage duration", and the early-delete SKUs are priced at $0.0125/GB-month (SIA), $0.01 (ZIA), $0.004 (Glacier Instant Retrieval) and $0.0036 (Glacier). Wasabi's equivalent is the Timed Deleted Storage charge: store an object on day 1, delete it on day 16 under a 90-day policy, and you are billed 15 days of Timed Active Storage plus 75 days of Timed Deleted Storage.

### 2.6 What the challenge is actually about

The cheapest storage rate on the board is AWS Glacier Deep Archive at $0.00099/GB-month — approximately $1.01 per TiB-month, against $7.64/TiB-month for Backblaze B2 and $8.58/TiB-month for Wasabi. That is roughly a 7.6× spread in favour of the archive class. The same archive class charges $0.02/GB to read your data back — about $20.48 per TiB — and imposes a 180-day minimum. Read more than about a third of a percent of the estate per month, repeatedly, and the spread collapses; read more than ~32% of the estate in a month and the archive class loses to the flat-rate challenger on retrieval alone. That crossover is the single most useful calculation in this guide, and it is developed in §3.

---

## 3. Worked Arithmetic: Six Workload Archetypes

**Every figure in this section is ILLUSTRATIVE ARITHMETIC**, computed from rates checked on 19 September 2026 on the vendor pages cited in §5, against the workloads defined below. They are models of a cost structure, not quotations. Prices are regional (US East N. Virginia for AWS, us-central1 for Google Cloud, the US-dollar tables for Azure, worldwide list for Oracle and the challengers). Storage is billed in binary GiB by AWS, Google and Azure, and in decimal GB or TB by Backblaze, Wasabi and R2; the arithmetic below uses each vendor's own unit, so the tables are internally consistent but the unit difference is real and worth ~9.6% on a 500 TiB estate.

### 3.1 The six archetypes and their assumptions

| Archetype | Estate | Monthly read/egress | Objects | Writes/requests per month | What it tests |
| --- | --- | --- | --- | --- | --- |
| (a) Compliance archive | 500 TiB | 256 GiB retrieved | 512,000 × 1 GB | 50k GET | Archive-class rate, minimum duration, per-object metadata overhead |
| (b) Backup/restore estate | 200 TiB | 4,096 GiB | 2,097,152 × 100 MB | 2M GET, 500k PUT, ~5% churn | Warm/cool class + retyping churn + minimum-duration exposure |
| (c) Media/report distribution | 100 TiB | 204,800 GiB egress (200% of estate) | 10 MB average | 20M GET, 50k PUT | Egress economics and the CDN path |
| (d) Analytics data lake | 1 PiB | 52,429 GiB | 8.4M × 128 MB | 10M GET, 1M PUT | Large-scale flat storage, request volume, per-object fees |
| (e) AI training corpus | 300 TiB | 921,600 GiB (3 full passes/month) | 307,200 × 1 GB | 1M GET, 200k PUT | Sustained high-egress reads; egress caps and multiple-of-storage allowances |
| (f) Logs / small objects | 50 TiB | 500M GET | 10,000,000,000 × ~5 KiB | 200M PUT | Minimum billable object size, per-1,000-object fees, request-class pricing |

### 3.2 (a) Compliance archive, 500 TiB, near-zero reads

| Provider and class | Illustrative monthly cost | Structure that drives it |
| --- | --- | --- |
| AWS S3 Glacier Deep Archive | **$512** | $0.00099/GB-mo + 40 KB/object metadata + $0.02/GB standard retrieval + 180-day minimum |
| Google Cloud Storage Archive | $655 | $0.0012/GiB-mo (derived from the published hourly rate) + $0.05/GiB retrieval + $0.05/1,000 read-class ops + 365-day minimum |
| Azure Blob Archive | $1,059 | $0.002/GB-mo + $0.02/GB retrieval + $5/10,000 archive reads + 180-day minimum |
| Oracle OCI Archive | $1,429 | $0.0026/GB-mo, no published minimum duration |
| Backblaze B2 | $3,821 | $6.95/TB-month flat, no retrieval fee, no minimum duration |
| Wasabi Hot Cloud Storage | $4,288 | $7.99/TB-month flat, no retrieval fee, 90-day minimum duration |
| Cloudflare R2 Infrequent Access | $5,500 | $0.01/GB-mo + $0.01/GB retrieval + 30-day minimum, egress free |
| DigitalOcean Spaces Cold Storage | Not a candidate at this scale | 100 buckets and 250 GiB included per subscription make it an operational non-starter for 500 TiB |

**Crossover.** Archive classes win on the storage line by 7–8×. They lose on retrieval. Glacier Deep Archive at $1.01/TiB-month versus Backblaze at $7.60/TiB-month is a $6.59/TiB-month spread; at $0.02/GB standard retrieval ($20.48/TiB), retrieving about **32% of the estate in a month** consumes the entire annual advantage. Against Google Archive ($1.23/TiB-month storage, $0.05/GiB retrieval) the crossover is at about **12.5% of the estate per month**. If the archive is genuinely write-once-read-never, the cold class is cheaper. If it is read on demand for audit, eDiscovery or replay, the flat-rate challenger frequently wins — and it wins without a retrieval fee line to explain to a CFO.

### 3.3 (b) Backup/restore estate, 200 TiB, 2% read, 5% monthly churn

| Provider and class | Illustrative monthly cost |
| --- | --- |
| Azure Blob Cool | **$2,096** |
| Google Cloud Storage Nearline | $2,096 |
| Oracle OCI Infrequent Access | $2,241 |
| Cloudflare R2 Infrequent Access | $2,246 |
| AWS S3 Standard-IA | $2,604 |
| Backblaze B2 | $1,528 |
| Wasabi Hot Cloud Storage | $1,715 |
| AWS S3 Standard (for contrast) | $4,560 |

The interesting result is not who is cheapest but how flat the field is: five hyperscaler cool-class configurations land within 25% of each other. The differentiator is the *retyping churn*. Backup data is rewritten, and every rewrite in a minimum-duration class re-arms the 30-, 90- or 180-day clock. AWS charges $0.0125/GB-month pro-rata for Standard-IA objects deleted before 30 days; Azure charges Cool early deletion at the Cool rate; Wasabi bills 90 days of Timed Deleted Storage. On a 200 TiB estate turning over 5% a month (10 TiB), a class with a 30-day minimum that is fed data older than 30 days pays nothing extra, while an archive class with a 180-day minimum fed the same churn pays a penalty on every cycle. **The minimum duration is a line item that scales with churn**, and backup estates are the churn-heaviest workload in most organisations.

### 3.4 (c) Media/report distribution, 100 TiB served, 200 TiB/month egress

| Provider and configuration | Illustrative monthly cost | Of which egress |
| --- | --- | --- |
| Google Cloud Storage Standard + internet egress | $23,151 | $21,094 |
| AWS S3 Standard + internet egress | $16,438 | $14,126 |
| Azure Blob Hot + internet egress | $15,870 | $14,014 |
| DigitalOcean Spaces Standard (CDN included) | $4,086 | included allowance then $0.01/GiB |
| Oracle OCI Standard + outbound beyond 10 TB free | $3,658 | — |
| Cloudflare R2 Standard | $1,657 | $0 |
| Wasabi Hot Cloud Storage | $858 | $0 |
| Backblaze B2 | $764 | $0 (204,800 GiB is inside the 3× allowance of 329,853 GB) |
| AWS S3 Standard, CDN-fronted | $2,312 | $0 from S3 to CloudFront ($0.00/GB), CloudFront egress not priced here |

This table is the thesis in one block. The same 100 TiB of data, at the same access pattern, costs **20× more on the hyperscaler egress path than on a flat-rate challenger** — and the storage line is nearly identical across the field. Note the honest counterweight: AWS prices data transfer from S3 to CloudFront at **$0.00/GB**, so a distribution architecture fronted by a CDN moves the egress bill out of the object-storage budget and into the CDN budget. That is a genuine mitigation, not a dodge — and it is also why the same mitigation exists for challengers, several of which waive egress to partner CDNs entirely. Egress economics for distribution therefore have three states, not two: billed per GB (hyperscaler direct), waived to a CDN (both hyperscalers and challengers), or never billed (R2, Wasabi, and Backblaze inside its 3× allowance). CDN egress rates are outside the scope of what was verified here; see §15.

### 3.5 (d) Analytics data lake, 1 PiB, 5% read, 10M GET + 1M PUT

| Provider and class | Illustrative monthly cost |
| --- | --- |
| Backblaze B2 | **$7,825** |
| Wasabi Hot Cloud Storage | $8,782 |
| Cloudflare R2 Standard | $16,897 |
| Azure Blob Hot | $18,229 |
| Google Cloud Storage Standard | $20,980 |
| AWS S3 Standard | $22,592 |
| AWS S3 Intelligent-Tiering | $22,613 (including a $20/month monitoring fee at this object size) |
| Oracle OCI Standard | $28,714 |

The AWS Intelligent-Tiering result carries a warning for the small-object archetype: the monitoring fee is charged **per object**, $0.0025 per 1,000 objects per month. At 1 PiB with 128 MB objects it is a rounding error ($20). At 10 billion objects it is $25,000 per month, which is more than the storage itself (§3.7). The same feature is cheap and expensive depending on a parameter that is not a price.

### 3.6 (e) AI training corpus, 300 TiB, three full passes per month (921,600 GiB egress)

| Provider and configuration | Illustrative monthly cost |
| --- | --- |
| Backblaze B2 | **$2,292** (the 3× allowance is 989,560 GB, so 921,600 GiB of egress is inside it) |
| Wasabi Hot Cloud Storage | $2,573 |
| Cloudflare R2 Standard | $4,949 |
| Oracle OCI Standard + outbound | $11,128 |
| Azure Blob Hot + egress | $55,328 |
| AWS S3 Standard + egress | $56,777 |
| Google Cloud Storage Standard + egress | $84,584 |

Two structural observations. First, the challengers are not merely cheaper — they are *categorically* different, because the egress charge that dominates the hyperscaler bill does not exist in their model. Second, Backblaze's 3× allowance is the most fragile of the challenger models: at three training passes per month the allowance is consumed exactly, and a fourth pass starts billing at $0.01/GB. The "free egress" claim is a claim about a *ratio to stored data*, and a training workload with high pass counts is precisely the workload that runs at that ratio.

### 3.7 (f) Logs and small objects, 50 TiB across 10 billion objects

| Provider and class | Illustrative monthly cost | What breaks |
| --- | --- | --- |
| Wasabi Hot Cloud Storage | **$319** | billed at the 4 KB minimum per object = 40,960 GB against an actual 54,976 GB |
| Backblaze B2 | $382 | no minimum object size, no request charges |
| Oracle OCI Standard | $1,640 | single request rate for all operations |
| Cloudflare R2 Standard | $1,896 | Class A at $4.50/million is significant against 200M writes |
| Azure Blob Hot | $2,142 | per-10,000 operation pricing |
| Google Cloud Storage Standard | $2,224 | Class A $0.005/1,000 against 200M writes |
| AWS S3 Standard | $2,378 | $0.005/1,000 PUT-class requests |
| AWS S3 Glacier Instant Retrieval | $6,083 | 128 KB minimum billable object size |
| AWS S3 Standard-IA | $16,459 | 128 KB minimum billable object size: 10bn objects × 128 KB = 1.31 PB billed |
| AWS S3 Intelligent-Tiering | $27,378 | $0.0025 per 1,000 objects per month monitoring fee = **$25,000/month, $300,000/year**, larger than the storage it optimises |

This is the archetype where the price-per-GB is not merely uninformative, it is actively misleading. The estate is 50 TiB and the cheapest bill is $319 while the most expensive configuration of the same data on the same vendor is $27,378 — an 85× span driven entirely by the structure of per-object charges.

### 3.8 The crossovers, stated plainly

| Claim | Crossover | Direction |
| --- | --- | --- |
| Cold archive class versus flat-rate challenger | Retrieve ~32% of the estate per month against Glacier Deep Archive ($0.02/GB retrieval), ~27% against Azure Archive, ~12.5% against Google Archive | Above the crossover the flat-rate challenger wins; below it the archive class wins on storage and on nothing else |
| Random-read latency tier versus archive | Any workload with a first-byte expectation below hours | Archive is disqualified regardless of price |
| Egress-free challenger versus hyperscaler | Any workload egressing more than ~0.2–0.7× the stored estate per month once retrieval fees are counted | The winner flips from hyperscaler to challenger; the flip is driven by egress, not storage |
| Flat-rate challenger versus hyperscaler hot class | Never, on the storage line: $7.64–$16.49/TiB versus $21.62–$23.55/TiB for S3 Standard | The hyperscaler hot class does not win on price at any scale in these models; it wins on features, ecosystem and compliance depth |
| Per-object fee versus minimum billable object size | Objects below ~5 KB on a provider with a 4 KB floor; objects below 128 KB on a Provider with a 128 KB floor | Cost becomes a function of object *count*, not bytes stored; estimate both |
| Request-class pricing versus free requests | Workloads above ~10^8 operations per month | Backblaze and Wasabi's free-request model changes the ranking of the small-object archetype entirely |

### 3.9 The winner flips — worked demonstration

Take the same 100 TiB estate and change only the egress ratio:

| Egress as % of estate per month | AWS S3 Standard + egress | Backblaze B2 | Cloudflare R2 | Who wins |
| --- | --- | --- | --- | --- |
| 0% (write-only archive) | $2,312 (storage + requests) | $764 | $1,657 | Backblaze |
| 1% (1,024 GiB) | ~$2,395 | $764 | $1,657 | Backblaze |
| 10% (10,240 GiB) | ~$3,225 | $764 | $1,657 | Backblaze |
| 50% (51,200 GiB) | ~$6,913 | $764 | $1,657 | Backblaze |
| 200% (204,800 GiB) | $16,438 | $764 | $1,657 | Backblaze |

The flip does not happen between the challengers and AWS S3 *Standard*, because the challengers beat S3 Standard on storage alone. The flip happens when the comparison moves to a **hyperscaler archive or warm class**: swap AWS S3 Standard for S3 Glacier Deep Archive and the picture inverts at low egress (see §3.2), then re-inverts as soon as retrieval volume crosses the fee. That is the honest formulation of the thesis: the storage line decides between classes, and egress decides between providers.

---

## 4. The Egress Axis

### 4.1 Why egress is priced at all

Bandwidth is not free to the provider, but nothing in the published pricing tables suggests egress is priced at cost. Three observations from the checked pages support that reading:

1. **The egress rate exceeds the storage rate by a large multiple.** AWS S3 Standard storage is $0.023/GB-month; egress to the internet is $0.09/GB for the first 10 TB. Transferring one GB out costs about four months of storing that GB in the hottest class — and about 91 months of storing it in Glacier Deep Archive at $0.00099/GB-month.
2. **The rate decreases with your dependence and disappears after the free tier.** AWS and Azure and Google all publish descending volume tiers ($0.09 → $0.085 → $0.07 → $0.05 per GB at AWS). An exiting customer is a customer in the *lowest* usage month of their history, which is exactly when the per-GB price is highest.
3. **The only two structural exceptions are vendors whose business model does not depend on trapping data.** Cloudflare R2, Wasabi and Backblaze (within its 3× allowance) publish no egress line. All three are challengers competing for exactly the workloads that hyperscalers monetise through egress.

### 4.2 The challengers' egress model, with conditions

**Cloudflare R2 — "no charges for egress bandwidth for any storage class."** The condition is embedded in footnote 1 of the R2 pricing page, and it is narrower than the marketing line: "Egressing directly from R2, including via the Workers API, S3 API, and `r2.dev` domains does not incur data transfer (egress) charges and is free. If you connect other metered services to an R2 bucket, you may be charged by those services." Where the claim does **not** apply:

- Any metered Cloudflare service you place in front of, or beside, R2 (Workers, CDN, Image Resizing, Stream) bills on its own schedule. R2's zero-egress claim is about R2's own egress, not about the architecture around it.
- The `r2.dev` development endpoint is not a production path: Cloudflare's own limits page states it "is not intended for production usage and has a variable rate limit applied", throttling beyond "hundreds of requests/second" with `429` responses, and that bandwidth may also be throttled on that endpoint.
- R2's free tier does not extend to Infrequent Access, and IA carries both a 30-day minimum storage duration and a $0.01/GB retrieval fee — so IA is an egress-free class with a *read* charge, which is the same trap in a different shape.

**Backblaze B2 — "Free egress up to 3× storage."** This is a ratio, not an unlimited grant: "free egress up to 3x their average monthly storage … Egress beyond 3x is just $0.01/GB." Backblaze states the average is computed on byte-hours over the billing cycle. Where it does not apply: workloads whose monthly read volume exceeds three times the stored estate (see §3.6, where a three-epoch training workload sits exactly on the line); and any workload that has been architected so that the *effective* stored estate is kept small while egress is large — a caching or transcoding tier in front of B2 shrinks the storage denominator. Backblaze's second, genuinely unlimited channel is partner-mediated: unlimited free egress "when downloading to or through partner content delivery networks (CDNs) and compute services, including Fastly, Cloudflare, bunny.net, CacheFly, CoreWeave, Equinix Metal, Vultr, and phoenixNAP". That is a real and useful waiver, conditional on using one of those partners, and it is a *dependency*, not a neutrality: your egress is free only while you stay inside that list.

**Wasabi — "No charge for egress or API requests."** The condition here is a fair-use policy rather than a volume cap. Wasabi's pricing FAQ states the free egress and free API request policies are "designed for use cases where you store your data with Wasabi, you access this data at a reasonable rate, and your use case does not impose an unreasonable burden on our service", and that "If your use case exceeds the guidelines of our free API request policy on a regular basis, we reserve the right to limit or suspend your service." Where it does not apply: a workload that wants to serve the data as a public distribution origin at high request rates and high concurrency is the profile the policy is written to exclude. Wasabi's egress claim is therefore **unpriced but not unconditional**, and the condition is qualitative — which is the hardest kind of condition to underwrite in a design review.

**Oracle OCI — the free allowance model.** Oracle does not claim zero egress. It publishes the largest free allowance of any hyperscaler on the checked page — first 10 TB/month free for outbound originating in North America, Europe and the UK, then $0.0085/GB — which for estates below roughly 10 TB/month of egress is functionally free, and for estates above it is the cheapest hyperscaler per-GB rate on the board (about 9.4% of AWS's first-tier rate). The condition is the allowance threshold and the origin geography: APAC/Japan/South America origins pay $0.025/GB and Middle East/Africa $0.05/GB beyond the same 10 TB.

**DigitalOcean Spaces — allowance inside a subscription.** 1 TiB/month of outbound transfer is included in the $5/month subscription, shared across buckets; beyond it, $0.01/GiB. Free paths are enumerated and geography-specific (Spaces in NYC3 to Droplets in NYC1/NYC2/NYC3, and similar pairs in SGP1, SFO2/SFO3, AMS3, FRA1, SYD1, BLR1, ATL1, LON1, TOR1), and private VPC-local DNS routing avoids counting against the allowance. Where the claim does not apply: any traffic that leaves the enumerated metros, and any traffic that resolves through an external DNS resolver instead of DigitalOcean's, which the pricing documentation states routes to the public endpoint and bills at the standard rate.

### 4.3 Free-allowance structures compared

| Provider | Free egress allowance | Shape of the allowance |
| --- | --- | --- |
| AWS | First 100 GB/month to the internet, aggregated globally | Absolute monthly volume |
| Azure | First 100 GB/month | Absolute monthly volume |
| Google Cloud Storage | 100 GB/month from North America to each Cloud data-transfer destination (Australia and China excluded) as part of Always Free | Absolute monthly volume, origin-restricted |
| Oracle OCI | First 10 TB/month (per origin geography group) | Absolute monthly volume — 100× the hyperscaler norm |
| Backblaze B2 | 3× average monthly storage, computed on byte-hours | **Multiple of stored data** — scales with the estate |
| Cloudflare R2 | Unlimited on R2's own egress paths | Unbounded, service-conditional |
| Wasabi | Unlimited, subject to fair-use policy | Unbounded, policy-conditional |
| DigitalOcean Spaces | 1 TiB/month per subscription, with enumerated free paths | Subscription allowance, path-specific |
| Azure (data-move-off credit) | Azure's bandwidth page states Azure "offers free egress for customers leaving Azure when taking their data out of the Azure infrastructure via the internet to switch to another cloud provider or an on-premises data center", on request beyond the first 100 GB/month | Exit-specific, claim-based |

The storage-multiple allowance is the most interesting design. An absolute allowance is easy to model and does not scale; a multiple-of-storage allowance scales with the estate, which means a large write-only archive gets a large free egress budget for the day it needs one. It also means the allowance *collapses* the moment the estate shrinks — a fact worth remembering if the exit plan is "copy everything out, then delete".

### 4.4 Invisible egress — the charges that are not labelled egress

The bill-of-materials for an exit is incomplete without these, and none of them appear in a "data transfer out to internet" line:

| Path | What it costs | Source checked 19 Sep 2026 |
| --- | --- | --- |
| AWS inter-region transfer | $0.02/GB from US East (N. Virginia) to any other region; $0.01/GB to US East (Ohio) | AWS S3 pricing page (worked examples); AWS RRS page region table |
| AWS S3 → CloudFront | $0.00/GB (a waiver, not a charge) | AWS RRS page data-transfer table |
| Google inter-region replication | $0.02/GiB for dual-regions and multi-regions in North America and Europe, $0.08 Asia and Oceania; turbo replication $0.04/$0.11/$0.12 | Cloud Storage pricing page |
| Google data transfer within Cloud | Free when the source and destination are in the same location; otherwise $0.02–$0.14/GiB by location pair | Cloud Storage pricing page |
| Azure inter-region | $0.02/GB between regions within North America or within Europe; $0.08 within Asia/Oceania/MEA; $0.05/GB North America→other continents | Azure bandwidth pricing page |
| Azure service-to-service in-region | Free; across regions, outbound is charged at the normal rate | Azure bandwidth pricing page |
| DigitalOcean Spaces → Droplet | Counts against the *Droplet's* outbound allowance (inbound to Spaces is free), not against Spaces | Spaces pricing documentation |
| Cross-AZ / cross-region replication traffic | Billed under replication SKUs and as data transfer; the replication configuration is the trigger | AWS S3 pricing page structure; Google inter-region replication section |
| CDN cache-fill | Google waives Cloud Storage data-transfer charges for Cloud CDN and Media CDN but states "cache fill charges may apply" | Cloud Storage pricing page, specialty network services |
| Object-service exit charges | Migration tooling itself can be charged: R2's Super Slurper and Sippy are free as products, but the *source* bucket may incur charges as objects are copied out, and R2 Class A operations are billed | R2 pricing page, data-migration pricing |

### 4.5 The argument: egress is lock-in

Put the pieces together and the mechanism is visible.

1. **Egress is a tax on the one action that ends the relationship.** Every other cost line rewards use; egress punishes departure. A customer who never leaves never pays it, so it is invisible during procurement and enormous during exit.
2. **Egress cost scales with the thing you are trying to move.** Storage costs scale with the estate too, but you can shrink the estate by deleting data. You cannot shrink the data you need to move.
3. **The egress price is highest at the moment you need it most.** Volume tiers invert for an exiting customer: the last month is the smallest month, and therefore the most expensive per GB.
4. **The waiver structure re-creates the dependency.** S3 → CloudFront at $0.00/GB, and Backblaze's partner list, both replace a per-GB charge with a requirement to stay inside a particular ecosystem. That is a cheaper dependency, not an absence of one.
5. **The comparison competitors make is storage-versus-storage, which is the line that matters least.** "We are 80% cheaper than the hyperscalers" is a storage-line claim. The competitive claim that never gets made is the exit-cost claim, because the challengers win that one outright and the hyperscalers cannot match it.

The honest counter-argument, which the design review must include: for many workloads egress is *not* a lock-in at all, because the data is cold and the retrieval is rare. A write-once regulatory archive in Glacier Deep Archive at $512/month for 500 TiB has an egress exposure that a competent architect can budget precisely, and no vendor change would save meaningful money on the storage line either. Egress is lock-in for workflows with **read amplification** — distribution, analytics, AI training, restores — and it is close to irrelevant for workflows without it. The judgement is per workload, and §12 applies it.

---

## 5. The Provider Line-Up

One compact subsection per provider: the commercial model, the tier/class structure, the pricing **structure** with dated sourced rates, API compatibility, notable compliance and feature capabilities, operational limits, and the honest weaknesses. All rates checked **19 September 2026**. Region priced is stated per provider; every one of these providers states on its own pricing page that prices are region-dependent.

### 5.1 Amazon S3 (cross-referenced, priced here for comparison)

- **Model.** Pay per GB-month, per request, per GB retrieved, per GB transferred; no minimum charge ("There is no minimum charge" — S3 pricing page).
- **Tier/class structure.** S3 Standard, Intelligent-Tiering, Standard-IA, One Zone-IA, Express One Zone, Glacier Instant Retrieval, Glacier Flexible Retrieval, Glacier Deep Archive. The classes themselves are covered in depth in [`s3_architecture_guide.md`](s3_architecture_guide.md); this guide uses their prices only as comparison inputs.
- **Structure and rates (us-east-1; AWS Price List API, file last modified 18 Sep 2026).** Standard $0.023/GB-mo first 50 TB, $0.022 next 450 TB, $0.021 over 500 TB. Standard-IA $0.0125, One Zone-IA $0.01, Glacier Instant Retrieval $0.004, Glacier Flexible Retrieval $0.0036, Glacier Deep Archive $0.00099, Express One Zone $0.11. Requests: $0.005/1,000 PUT/COPY/POST/LIST and $0.0004/1,000 GET/other on Standard; Glacier writes $0.03/1,000; Standard-IA writes $0.01/1,000 and reads $0.001/1,000. Retrieval: $0.01/GB SIA and ZIA, $0.03/GB Glacier IR, $0.01/GB Glacier Flexible standard and $0.00 bulk, $0.02/GB Deep Archive standard and $0.0025 bulk. Intelligent-Tiering monitoring $0.0025 per 1,000 objects per month. Egress: first 100 GB/month free, then $0.09/GB (first 10 TB), $0.085 (next 40 TB), $0.07 (next 100 TB), $0.05 (above 150 TB). Transfer to other AWS regions $0.02/GB; to CloudFront $0.00/GB. Early deletion charged at $0.0125 (SIA), $0.01 (ZIA), $0.004 (Glacier IR) and $0.0036 (Glacier) per GB-month pro-rata.
- **API compatibility.** The reference implementation; private S3-compatible services are compared against it (see the lakehouse guide's §13 tiering).
- **Compliance/feature capabilities.** Object Lock is WORM with retain-until-date and legal hold; AWS states Object Lock "has been assessed by Cohasset Associates for use in environments that are subject to SEC 17a-4, CFTC, and FINRA regulations" (S3 User Guide, Object Lock). Versioning, lifecycle, replication, Batch Operations, Inventory, Access Grants, SSE-KMS/SSE-C/DSSE, FIPS endpoints, PCI DSS validation (S3 User Guide, "PCI DSS compliance").
- **Durability/availability claimed.** "Designed to provide 99.999999999% durability and 99.99% availability of objects over a given year" for Standard storage (S3 User Guide, Data protection). Object Lock, versioning and replication are separate protections.
- **SLA.** Monthly Uptime Percentage commitment 99.9% for S3 Standard/Express One Zone/Glacier Flexible/Deep Archive and 99.0% for Intelligent-Tiering/Standard-IA/One Zone-IA/Glacier IR, with service credits of 10%/25%/100% of the affected storage class charges. The SLA states plainly that credits "will not entitle you to any refund or other payment" and that "your sole and exclusive remedy for any unavailability … is the receipt of a Service Credit". Claims must be filed by the end of the second billing cycle after the incident, with request logs.
- **Operational limits.** 3,500 PUT/s and 5,500 GET/s per prefix, per the S3 consistency page — now a documented floor rather than the old random-prefix folklore.
- **Honest weaknesses.** The most expensive request and egress structure in the comparison; the deepest feature surface, which means the largest number of dependencies to unwind at exit; and egress that becomes a direct line on the P&L for read-amplified workloads. Nothing about S3's durability or availability is a weakness.

### 5.2 Google Cloud Storage

- **Model.** Pay per GiB stored (hourly-prorated), per operation class, per GiB retrieved, per GiB transferred; Always Free allowance.
- **Tier/class structure.** Standard, Nearline, Coldline, Archive, plus Rapid (zonal) and Rapid Cache. Regional, dual-region, multi-region and zonal locations.
- **Structure and rates (us-central1; Cloud Storage pricing page).** Published as hourly per-GiB: Standard $0.000027397/GiB-hour; Nearline $0.000013699; Coldline $0.000005479; Archive $0.000001644; Rapid Cache storage $0.0001233; Rapid $0.000150685. My arithmetic converts to monthly at 730 hours (Standard ≈ $0.0200/GiB-month, Nearline ≈ $0.0100, Coldline ≈ $0.0040, Archive ≈ $0.0012) — the conversion is mine, the hourly figures are Google's. Operations, regional buckets: Standard Class A $0.005/1,000 and Class B $0.0004/1,000; Nearline A $0.01, B $0.001; Coldline A $0.02, B $0.01; Archive A $0.05, B $0.05. Retrieval: Nearline $0.01/GiB, Coldline $0.02, Archive $0.05. Minimum durations: Nearline 30 days, Coldline 90, Archive 365. Egress: $0.12/GiB to 10 TiB, $0.11 to 150 TiB, $0.08 above (worldwide excluding Asia and Australia); China $0.23/$0.22/$0.20; Australia $0.19/$0.18/$0.15. Inter-region replication $0.02/GiB (North America, Europe), $0.08 (Asia, Oceania); turbo replication $0.04/$0.11/$0.12. Autoclass management fee $0.0025 per 1,000 objects per 30 days. Tags $0.005 per bucket-month. Always Free: 5 GB-months Standard, 5,000 Class A, 50,000 Class B, 100 GB North America transfer.
- **API compatibility.** Native JSON/XML/gRPC APIs plus an S3-interoperable XML API surface; strong consistency is documented for the service.
- **Compliance/feature capabilities.** **Bucket Lock** — a bucket retention policy that can be locked permanently, retroactively applies to existing objects, and on locking applies a project lien that blocks project deletion; Google states Bucket Lock "can help with regulatory and compliance requirements, such as those associated with FINRA, SEC, and CFTC". Separate **Object Retention Lock** and event-based holds operate per object. Detailed audit logging mode is referenced as part of the same compliance story. CMEK and key-destruction protections interact with locked buckets (key versions cannot be destroyed while locked objects are inside their retention window).
- **Durability/availability claimed.** "Designed for at least 99.999999999% (11 9's) annual durability, regardless of storage class and location type". Regional buckets store across at least two zones; dual/multi-region buckets store across at least two geographic places, with default replication targeting 99.9% of new objects within one hour and 100% within 12 hours.
- **SLA.** 99.95% for Standard in multi/dual-region; 99.9% for Standard regional or Nearline/Coldline/Archive in multi/dual-region; 99.0% for Nearline/Coldline/Archive regional. Financial credits 10%/25%/50%, capped at 50% of the monthly bill for the affected service. Credits must be requested within 30 days.
- **Operational limits.** Archive read operations are charged at the Class B *Archive* rate of $0.05 per 1,000 — the same as its write class — which makes an Archive read-heavy pattern expensive in a way the storage line hides.
- **Honest weaknesses.** The most expensive internet egress at the top of the range among the hyperscalers compared here ($0.12/GiB first tier versus $0.09 at AWS); a 365-day Archive minimum duration, the longest on the board; and a separate Object Lock *product family* (Bucket Lock, Object Retention Lock, object holds) whose interaction takes real study before a compliance claim should be made.

### 5.3 Azure Blob Storage

- **Model.** Pay per GB-month by tier and redundancy, per operation (write/read/list, priced per 10,000), per GB retrieved, per GB egress; optional reserved capacity.
- **Tier/class structure.** Hot, Cool, Cold, Archive, plus Premium block blob; redundancy LRS, ZRS, GRS/RA-GRS, GZRS/RA-GZRS.
- **Structure and rates (US-dollar tables, locally-redundant storage; Blob Storage pricing page).** Hot $0.0184/GB first 50 TB, $0.0177 next 450 TB, $0.0170 over 500 TB; Cool $0.01; Cold $0.0036; Archive $0.002; Premium $0.15. Writes per 10,000: Hot $0.05, Cool $0.10, Cold $0.18, Archive $0.10; list/create-container $0.05. Reads per 10,000: Hot $0.004, Cool $0.01, Cold $0.10, Archive $5 (high priority $50). Retrieval per GB: Cool $0.01, Cold $0.03, Archive $0.02 ($0.10 high priority). Early deletion: Cool 30 days, Cold 90 days, Archive 180 days. Archive rehydration to Hot or Cool "typically takes up to 15 hours". Reserved capacity: 100 TB at $1,545/month (1-year, Hot) down to $1,244/month (3-year); 1 PB Hot at $15,050/month (1-year) or $11,963/month (3-year). Egress (bandwidth pricing page): first 100 GB/month free, then $0.087/GB (next 10 TB), $0.083 (next 40 TB), $0.07 (next 100 TB), $0.05 (next 350 TB) routed over the Microsoft premium global network; the lower-cost transit-ISP routing option is $0.08/$0.065/$0.06/$0.04. Inter-region: $0.02/GB within North America or within Europe.
- **API compatibility.** Blob REST API with an S3-interoperability surface avoided as the primary path; Azure's own SDKs and Data Lake Storage Gen2 hierarchical namespace are the idiomatic interfaces.
- **Compliance/feature capabilities.** Immutable storage for Blob supports time-based retention policies (minimum 1 day, maximum 146,000 days) and legal holds, at container level or version level. Microsoft states it retained "Cohasset Associates, to evaluate immutable storage for blobs", and that "Cohasset validated that immutable storage, when used to retain blobs in a WORM state, meets the relevant storage requirements of CFTC Rule 1.31(c)-(d), FINRA Rule 4511, and SEC Rule 17a-4(f)". Microsoft also states a time-based retention policy **must be locked** to be in a compliant immutable state for SEC 17a-4(f), and that locked policies can be extended but not shortened (container-level policies allow at most five increases). Retention policy audit logging exists per container; version-level policy changes are not audited. Immutability is incompatible with point-in-time restore and last-access tracking.
- **Durability/availability claimed.** LRS "at least 99.999999999% (11 9s)" and ZRS "at least 99.9999999999% (12 9s)" durability over a given year (Data redundancy documentation). Azure also states the limitation bluntly: "Redundancy protects against hardware failure, not against data-modifying operations", and all replicas reflect the same current state, so deletions and overwrites propagate to every copy.
- **SLA.** Storage account SLA is published separately as a support-legal document; the exact commitment and credit table were not read for this guide (see §15).
- **Operational limits.** Default 40,000 requests/second per storage account in the listed major regions and 20,000 elsewhere; default egress 200 Gbps in listed regions, 50 Gbps elsewhere; 250 storage accounts per region per subscription by default, 5 PiB default capacity. The Archive tier is not supported on ZRS, GZRS or RA-GZRS accounts.
- **Honest weaknesses.** The tier matrix (five tiers × six redundancy options × region) is the most complex pricing surface here, which makes an accurate quote harder than it needs to be; archive read operations priced at $5 per 10,000 make Archive unsuitable for anything but bulk restore; and the Archive rehydration time of up to 15 hours is an availability property that no per-GB rate discloses.

### 5.4 Oracle Cloud Infrastructure Object Storage

- **Model.** Pay per GB-month by tier, per request (single rate), per GB outbound beyond the free allowance.
- **Tier/class structure.** Standard, Infrequent Access, Archive.
- **Structure and rates (OCI price list).** Object Storage Standard $0.0255/GB-month; Infrequent Access $0.01/GB-month with a $0.01/GB retrieval charge; Archive $0.0026/GB-month; Object Storage requests $0.0034 per 10,000 requests — a *single* request class for all operations, which is structurally unusual and simplified relative to the A/B and Tier1/Tier2 models elsewhere. Outbound data transfer: first 10 TB/month free for North America/Europe/UK origins, then $0.0085/GB; APAC/Japan/South America $0.025/GB; Middle East/Africa $0.05/GB. Always Free includes up to 10 TB/month outbound transfer; Object Storage is Always Free-eligible (exact Always Free object-storage GB not verified — §15).
- **API compatibility.** Oracle publishes both a native Object Storage API and a documented **Amazon S3 Compatibility API**, plus a Swift API used with Oracle RMAN.
- **Compliance/feature capabilities.** Strong consistency is documented as a service characteristic: "When a read request is made, Object Storage always serves the most recent copy of the data that was written to the system." Server-side AES-256 encryption, retention rules, lifecycle policy rules, replication policies, pre-authenticated requests, and object versioning are documented capabilities.
- **Durability/availability claimed.** Oracle's overview describes redundant storage across several servers with checksum monitoring and automatic repair, and points to its Object Storage FAQ for durability specifics; **no numeric durability figure was extracted from the Oracle pages read for this guide** (§15).
- **SLA.** Not read for this guide; note that Oracle's own documentation classifies request-rate capacity as "a Service Level Objective only" (§15).
- **Operational limits.** Default request-rate capacity of 12,000 read / 3,000 write / 2,000 list RPS in three-availability-domain regions, and 5,000 / 3,000 / 2,000 elsewhere, with increases available on request for tenants consuming several petabytes. Maximum object size 10 TiB, maximum part size 50 GiB, maximum 10,000 parts, PutObject limit 50 GiB, 4,000 bytes of metadata per object. One namespace per root compartment, immutable and region-scoped in name.
- **Honest weaknesses.** Standard is the most expensive flat storage rate in this comparison at $0.0255/GB-month; the 10 TiB object ceiling is the lowest of the hyperscalers here; the durability claim could not be verified numerically and should not be repeated without it; and the low-billed-rate, high-free-allowance egress model is a genuinely strong position on the egress axis that is under-marketed relative to AWS and Azure.

### 5.5 Cloudflare R2

- **Model.** Pay per GB-month stored, per million operations in two classes, per GB retrieved from Infrequent Access; **no egress charge on R2's own paths**.
- **Tier/class structure.** Standard and Infrequent Access only — there is no archive class, so the R2 optimisation problem is "keep everything hot and pay $0.015" or "use IA and accept a 30-day minimum plus retrieval fees".
- **Structure and rates (R2 pricing page, last updated 7 Aug 2026).** Storage $0.015/GB-month Standard, $0.01/GB-month IA. Class A $4.50 per million (IA $9.00); Class B $0.36 per million (IA $0.90); deletes and aborts free; data retrieval on IA $0.01/GB; egress free subject to footnote 1. Free tier: 10 GB-month, 1M Class A requests, 10M Class B requests — **Standard only**, and the page warns the free tier "does not apply to Infrequent Access storage". Billing unit rounding is stated explicitly: usage is **rounded up to the next billing unit** ("If you have used 1.1 GB-month, you will be billed for 2 GB-month"), and storage is billed on the average of *peak* daily storage over a 30-day period.
- **API compatibility.** S3 API plus Workers API; the S3 endpoint is the documented high-throughput path, and Cloudflare's own limits page directs high-throughput work away from its REST API, which is capped at 1,200 requests per five minutes per account.
- **Compliance/feature capabilities.** Bucket locks implement retention rules per prefix with duration, until-date or indefinite conditions, up to 1,000 rules, strictest-wins when rules overlap, and lock rules take precedence over lifecycle rules. Encryption at rest is AES-256 (GCM preferred) with keys managed by Cloudflare; TLS in transit. On compliance certifications the R2 data-security page states only that readers should visit the Cloudflare Trust Hub — **R2's own documentation does not assert specific frameworks for R2**, and this guide does not assert any on its behalf.
- **Durability/availability claimed.** No numeric durability figure was published on the R2 pages read (pricing, limits, consistency, bucket locks, data security) — §15.
- **SLA.** Not read for this guide — §15.
- **Operational limits.** 1,000,000 buckets per account; 50 bucket-management operations per second per bucket; object size 5 TiB; 5 GiB single-part uploads and 4.995 TiB multipart with 10,000 parts; object key 1,024 bytes; metadata 8,192 bytes; one concurrent write per second to the same object name; `r2.dev` public access rate-limited and explicitly not for production.
- **Honest weaknesses.** Class A at $4.50 per million is close to AWS's $5.00 per million — the zero-egress advantage does not extend to the write path; the free tier excludes IA; billing rounds up to the next whole unit; there is no archive class and no published durability figure; and the strongest compliance claims in the platform are made at Cloudflare level, not at R2 level, which is exactly the kind of gap an auditor will ask about.

### 5.6 Backblaze B2

- **Model.** Pay per TB/month stored, with free egress up to a multiple of storage; API transactions free.
- **Tier/class structure.** Single always-hot class ("Always-Hot Cloud Storage"), plus a separate higher-throughput product, B2 Overdrive, at $15/TB/month with unlimited free egress to any destination and a multi-petabyte commitment requirement.
- **Structure and rates (B2 pricing page and transaction-pricing page).** $6.95/TB/month pay-as-you-go or annual commitment ("billed based on the amount of data stored per byte-hour over the last month at a rate of $6.95/TB/30-days"); first 10 GB always free; "No minimum file size or storage duration fees"; free egress up to 3× average monthly storage, then $0.01/GB; unlimited free egress through named partner CDN and compute services (Fastly, Cloudflare, bunny.net, CacheFly, CoreWeave, Equinix Metal, Vultr, phoenixNAP); Class A, B and C transactions free for pay-as-you-go customers; Class D (outbound calls such as event notifications) $0.004 per 10,000 after the first 2,500 per day.
- **API compatibility — the native/S3 split.** Backblaze exposes three APIs: the S3-Compatible API, the B2 Native API, and a Partner API. The S3-Compatible API's own documentation lists the following as **not currently supported**: ACLs, IAM Roles, Object Tagging, Website Configuration, and browser-based uploads to pre-signed URLs using `POST`. Object-level ACLs do not exist — a `GetObjectAcl` call returns the *bucket's* ACL — and `PutBucketAcl` accepts only "private" or "public-read". `x-amz-tagging` is explicitly rejected on `PutObject`/`CopyObject`. This is the clearest illustration in this guide that S3 compatibility is a spectrum: a well-known application can be 95% portable and still fail on a policy or tagging call.
- **Compliance/feature capabilities.** Object Lock and legal hold are documented for both the S3-compatible and native API paths. Backblaze's compliance page claims SOC 2 Type 2, HIPAA (BAA available on request), PCI-DSS (with card processing via Stripe), GDPR and UK GDPR, CCPA/CPRA, GovRAMP Progressing Snapshot, TX-RAMP Provisional, ISO 27001 (stated as held by the data centres it operates in), TPN Blue Shield, HECVAT, VPAT and Internet2 participation. Backblaze is a publicly traded company (BLZE), which is directly relevant to §8's going-concern question.
- **Durability/availability claimed.** Backblaze states its Vaults architecture "calculates at 99.999999999% (11 x 9s) annual durability", built on 20 storage pods per vault with Reed-Solomon erasure coding that can reconstruct a file from any 17 of 20 shards.
- **SLA.** A B2 SLA document was not located and read for this guide — §15.
- **Operational limits.** No published per-bucket request ceilings on the pages read; the S3-compatible surface's missing features are the real limits (no IAM roles, no object tagging, no website configuration).
- **Honest weaknesses.** The free-egress claim is a ratio that a high-pass AI or media workload can exhaust; the 3× allowance shrinks if the stored estate shrinks; the S3-compatible surface has documented gaps that will break specific SDK calls; the flat single-class model gives you no cost lever for cold data; and the enterprise feature surface (IAM model, tagging, website configuration) is thinner than the hyperscalers'.

### 5.7 Wasabi

- **Model.** Flat per-TB/month, capacity only. "You pay just one low rate for capacity only." No egress or API request fees, subject to policy.
- **Tier/class structure.** Effectively one class (Hot Cloud Storage). Wasabi also sells Cloud NAS at $9.99/TB/month and, uniquely, markets "Covert Copy" — hidden buckets with enforced multi-user authentication for ransomware recovery.
- **Structure and rates (Wasabi pricing page and pricing FAQ).** $7.99/TB/month ($0.0078/GB-month) pay-as-you-go in North America, EMEA and APAC; Reserved Capacity Storage purchased in 1-, 3- or 5-year increments; ingress free; egress free; API requests free — all three marked "subject to Wasabi's free egress and API request policies". The invoice line items are explicit: Timed Active Storage, Timed Deleted Storage, Data Transfer In (shown, not charged), Data Transfer Out (shown, not charged), API Requests (shown, not charged), and Minimum Active Storage — "In the event the Timed Active Storage charge is less than the monthly minimum, a charge for the difference between Wasabi's monthly minimum and the monthly Timed Active Storage is applied". **The value of that monthly minimum was not extracted from the pages read (§15).**
- **Minimum duration.** 90 days by default under the policy documented on 14 Sep 2026, with a 30-day policy available; a change from 90 to 30 days "applies only to objects uploaded after the change is made" and triggers a mid-cycle invoice. Wasabi's own comparison is candid: against "the AWS 23-day billing plan retention, if you delete the 100 TB of storage in less than 23 days, Wasabi will be more expensive than AWS S3."
- **Minimum billable object size.** 4 KB. "If you store a 2 KB file with Wasabi, you will be charged as if it were a 4 KB file."
- **API compatibility.** S3-compatible, with documentation for validated integrations; Wasabi ties its free-API policy to using validated applications.
- **Compliance/feature capabilities.** Wasabi's trust centre claims data centres that are "SOC-2 compliant and certified for ISO 27001, and PCI-DSS", and states "Compliance with ISO 27001, HIPAA, CJIS, GDPR/UK GDPR controller obligations, and applicable SEC standards". Object Lock (immutability) is documented, as are multi-user authentication for sensitive actions such as Object Lock changes, SSO, and encryption options limited to client-side encryption and SSE-C — note that **SSE-KMS-style vendor-managed key custody is not among Wasabi's listed options**, which matters in §11. Immutability pricing is stated plainly: "Any storage (immutable or not) will be charged as active storage."
- **Durability claimed.** Designed for "11x9s (99.999999999%) of data durability"; AES-256 encryption at rest, FIPS-197.
- **SLA.** The SLA page as fetched contains navigation and a link to a "February 21, 2024" reference PDF; the operative terms were not read — §15.
- **Operational limits.** Versioning and bucket logging increase billable storage (documented in the pricing FAQ); free API requests are policy-bound rather than technical; and Wasabi reserves the right to limit or suspend a service whose load it deems unreasonable.
- **Honest weaknesses.** The 90-day minimum duration is the second-longest on the board and applies to *every* object including overwrites ("the original copy of foo.pdf will transition from active storage to deleted storage … 29 or 89 days of deleted storage"); the 4 KB minimum billable size penalises log and IoT workloads; the free egress policy is qualitative and therefore not contractually quantified the way a rate is; key custody options stop at SSE-C; and the monthly minimum charge is a commitment floor that flat-rate marketing tends to omit.

### 5.8 DigitalOcean Spaces

- **Model.** Subscription with an included allowance, plus per-GiB overage on storage and transfer; separate Cold Storage class.
- **Tier/class structure.** Standard Storage and Cold Storage; includes a CDN at no additional cost.
- **Structure and rates (Spaces pricing page and Spaces pricing documentation).** $5.00 per month base subscription including 250 GiB of storage across up to 100 buckets and 1,024 GiB (1 TiB) of outbound transfer; additional storage $0.02/GiB; additional outbound transfer $0.01/GiB; Cold Storage $0.007/GiB-month with $0.01/GiB retrieval "waived for data retrieved up to your average daily Cold Storage usage for the month"; Cold early deletion/update $0.007/GiB with the first 250 GiB per month free. Cold Storage has a 30-day minimum retention; objects below 128 KiB billed as 128 KiB and every cold read carries a 128 KiB minimum retrieval charge. Standard minimum billable object size is 4 KiB "rounded up. Storage includes both data and metadata." Incomplete multipart uploads older than 30 days are deleted automatically.
- **API compatibility.** Ceph-based "compatible with a large subset of the S3 RESTful API", per DigitalOcean's own documentation — an honest self-description of a subset rather than a claim of equivalence.
- **Compliance/feature capabilities.** Object versioning (API-only), access logs (S3 API or Terraform only, and CDN and origin logs share one target folder), CDN with custom endpoints. **No Object Lock or WORM capability is documented on the Spaces features or limits pages read**, which disqualifies Spaces from any WORM archive requirement.
- **Durability/availability claimed.** Spaces runs on Ceph for redundancy and fault tolerance; Cold Storage "provides the same durability as Spaces Standard Storage". No numeric durability figure was published on the pages read.
- **SLA.** Cold Storage is documented at a 99.5% SLA; the Standard Storage SLA was not read — §15.
- **Operational limits.** This is where Spaces is structurally different: **800 total operations per second per new bucket** (older buckets, lower limits), up to 100 buckets and 200 access keys per account, 100 million unversioned objects or 50 million versioned objects per bucket, `PUT` up to 5 GB, multipart to 5 TB, and the hard blockers — per-bucket access keys are incompatible with S3 bucket policies, Cold buckets support no CDN, no bucket policies, no CORS and no intelligent tiering, `CopyObject` does not work between Standard and Cold, and you cannot use a third-party CDN alongside the built-in one.
- **Honest weaknesses.** At the scale of the other archetypes in §3 this is not a candidate — 100 buckets and 250 GiB included make a 100 TiB estate an exercise in bucket arithmetic, and the 800 ops/second per bucket ceiling is an order of magnitude below what a log or analytics workload will need. Its genuine strengths are price transparency, a bundled CDN, and a low entry cost.

### 5.9 MinIO (self-hosted, cross-referenced)

- **Model.** Software subscription, not metered storage. No per-GB rate is published: MinIO offers an AIStor Free tier (single-node), Enterprise Lite (for capacity below 400 TiB) and Enterprise, all by quotation.
- **Tier/class structure.** Not applicable — one software-defined pool; tiering is your own hardware decision.
- **Pricing structure.** Quoted. What is published is the tiering *boundary* (Enterprise Lite below 400 TiB, Enterprise above) and the support model (Lite includes a SUBNET subscription with health diagnostics; Enterprise adds proactive 24/7/365 direct-to-engineer support with a stated <4 hour SLA and a "Panic Button").
- **API compatibility.** Fully S3-compatible by design, with additional protocols (S3 Express, SFTP, Iceberg Catalog, Delta Sharing), per the MinIO pricing page.
- **Compliance capabilities claimed.** MinIO's pricing page lists FIPS 140-3, SOC 2, a penetration test report, ISO 27001, SEC 17a-4(f) compliant and FINRA 4511(c)-(d) compliant, alongside encryption, KMS, immutability and a data firewall.
- **Operational limits.** Your hardware. That is simultaneously the strength and the entire operational burden. The on-premises field — MinIO, Dell ObjectScale, IBM COS, NetApp StorageGrid, Ceph RGW, Pure FlashBlade and VAST — is owned by [`cloud_object_storage_lakehouse_guide.md`](cloud_object_storage_lakehouse_guide.md) §5, and this guide does not repeat it.
- **Honest weaknesses.** Zero egress cost is not a vendor promise; it is a capital expense, an operational team and a datacentre. The correct comparison for a self-hosted option is not $/TB but fully-loaded $/TB including hardware refresh, staff, replication bandwidth and the opportunity cost of the team. It does, however, make one thing structurally possible that no cloud provider can match: the egress line item disappears from the architecture entirely, and with it the lock-in mechanism of §4.

### 5.10 Providers this brief did not name, surfaced by the research

| Provider | What it is | Why it belongs in the line-up |
| --- | --- | --- |
| **Backblaze B2 Overdrive** | A higher-throughput product on the B2 foundation at $15/TB/month with **unlimited free egress to any destination** and a multi-petabyte commitment requirement | It is the only "zero egress with no destination condition" offer found at any price in this research, and it is sold as a throughput product, not a storage product — the economics only work at AI/HPC scale (`backblaze.com/cloud-storage/b2-overdrive`) |
| **Wasabi Cloud NAS** | A Cloud NAS product at $9.99/TB/month with the same free egress and free API request structure | A price-per-TB product for file-protocol workloads that are not classic object storage; included here because buyers compare it against object storage on the storage line alone, which is the error this guide exists to prevent |
| **Wasabi Covert Copy** | Hidden buckets with enforced multi-user authentication and immutability | An immutability *architecture* rather than a retention feature; relevant to §11 because it changes the ransomware exposure model that Object Lock alone does not address |
| **Cloudflare R2 Data Catalog / Super Slurper / Sippy** | Catalog, bulk migration and incremental migration products layered on R2 | Migration tooling is part of the cost of the exit; Super Slurper and Sippy are free as products but charge Class A operations, and the source bucket may bill the reads (`developers.cloudflare.com/r2/pricing/`) |

---

## 6. The Feature Matrix That Actually Decides

Storage capacity is a commodity. The following capabilities are not, and they are the ones that decide whether a migration is possible at all — as distinct from whether it is cheap.

### 6.1 The matrix

Legend: ✅ documented on the vendor's own pages as read; ◐ partial or qualified in the vendor's own documentation; ✖ not documented on the pages read for this guide; — not applicable. Everything in this matrix reflects what the vendor itself claims on 19 September 2026, not what a third party says about it.

| Capability | AWS S3 | Google Cloud Storage | Azure Blob | Oracle OCI | Cloudflare R2 | Backblaze B2 | Wasabi | DO Spaces | MinIO AIStor |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Object Lock / WORM | ✅ retention + legal hold | ✅ Bucket Lock + Object Retention Lock + holds | ✅ time-based retention + legal hold | ✅ retention rules | ✅ bucket locks (up to 1,000 rules, prefix-scoped, indefinite supported) | ✅ Object Lock + legal hold | ✅ Object Lock | ✖ not documented on Spaces pages read | ✅ immutability |
| Financial-services regime claim (as the vendor states it) | ✅ "assessed by Cohasset Associates for use in environments subject to SEC 17a-4, CFTC, FINRA" | ✅ Bucket Lock "can help with … FINRA, SEC, and CFTC" | ✅ Cohasset validated CFTC 1.31(c)-(d), FINRA 4511, SEC 17a-4(f); policy must be locked | ✖ none asserted in the pages read | ✖ R2 docs point to the Cloudflare Trust Hub only | ◐ Object Lock documented; no SEC/FINRA-specific assessment claimed | ◐ "applicable SEC standards" claimed in trust centre prose | ✖ | ✅ "SEC 17a-4(f) compliant", "FINRA 4511(c)-(d) compliant" |
| Versioning | ✅ | ✅ (and noncurrent-version charges) | ✅ | ✅ | ✅ (implied by lock, documented as feature) | ✅ file versions | ✅ (costs extra storage) | ◐ API-only | ✅ |
| Lifecycle rules | ✅ | ✅ | ✅ | ✅ | ✅ (locks override lifecycle) | ✅ | ✅ | ◐ (control-panel limits; recursive changes need third-party clients) | ✅ |
| Replication | ✅ (CRR/SRR, MRAP) | ✅ (default, turbo, cross-bucket) | ✅ (redundancy options + object replication) | ✅ replication policies | ✖ not documented on the pages read | ✅ cloud replication | ✅ Object Replication | ✖ | ✅ active-active, sync, async, multi-site |
| Consistency | ✅ strong read-after-write and list | ✅ strong consistency documented | ✅ (strong, versioned blobs) | ✅ "always serves the most recent copy" | ✅ strong read-after-write, metadata, delete, list; IAM changes eventually consistent (~1 min) | ◐ not asserted on pages read | ◐ not asserted on pages read | ◐ not asserted on pages read | ✅ |
| Inventory / audit facility | ✅ S3 Inventory, Batch Operations, Access Grants | ✅ Storage Intelligence insights datasets + detailed audit logging | ✅ retention policy audit log, Activity log, resource logs | ✅ Object Storage metrics, logs, work requests | ◐ bucket locks list/get; no inventory product documented | ◐ bucket access logs, Enterprise console reports, anomaly alerts | ✅ bucket logging (billable) | ◐ access logs via S3 API/Terraform only | ✅ audit trail |
| Event notifications | ✅ | ✅ (Pub/Sub, object change notifications) | ✅ | ✅ | ◐ not documented on pages read | ✅ event notifications + webhooks | ◐ | ✖ | ✅ |
| IAM / policy model | ✅ IAM + bucket policies + Access Grants | ✅ IAM + bucket policies + ACLs + org policy | ✅ Entra ID/RBAC + SAS + no public-by-default | ✅ IAM policies + compartments + network sources | ✅ API tokens with granular permissions; account-scoped | ◐ application keys with capability sets; **no IAM roles over S3 API** | ✅ bucket and object policies, IAM-style roles, MUA for sensitive actions | ◐ per-bucket keys incompatible with S3 bucket policies | ✅ |
| Encryption and key custody | ✅ SSE-S3, SSE-KMS, SSE-C, DSSE, FIPS endpoints | ✅ Google-managed, CMEK, CSEK, key-destruction protections on locked buckets | ✅ Microsoft-managed, customer-managed keys, infrastructure double encryption | ✅ AES-256 with keys created at upload, KMS | ◐ AES-256 at rest with keys managed by Cloudflare; no BYOK documented | ✅ server-side encryption + application keys | ◐ client-side and SSE-C only in the stated options | ◐ HTTPS + control-panel keys; no key-custody options documented | ✅ KMS, key management server |
| Published durability | ✅ 11 nines (Standard et al.) | ✅ 11 nines (all classes/locations) | ✅ 11 nines LRS / 12 nines ZRS | ✖ not extracted | ✖ | ✅ 11 nines (Vaults) | ✅ 11 nines | ✖ | — |
| Published SLA commitment | ✅ 99.9% (99.0% for some classes) | ✅ 99.95% / 99.9% / 99.0% by class and location | ◐ published separately; not read here | ✖ not read here | ✖ not read here | ✖ not located | ◐ PDF referenced, not read | ◐ 99.5% Cold Storage documented | ◐ support SLA, not storage uptime |
| Data-residency depth | ✅ many regions + sovereign cloud | ✅ regions, dual, multi | ✅ many regions + sovereign/gov | ✅ regions + gov | ◐ jurisdictional data-location controls (not priced in this guide) | ◐ US and EU regions | ✅ named regions | ◐ limited metros | — |
| Support model | tiered paid plans | tiered paid plans | tiered paid plans | included + premium | tiered paid plans | add-on implementation and premium support | Basic (PAYG) or Premium (RCS) | ticket-based | SUBNET <4 h (Enterprise) |

### 6.2 The rows that bite

**Object Lock is the gate, and the paperwork is the lock.** Four providers make a financial-services regime claim in their own documentation (AWS, Google, Azure, MinIO), and three of those claims rest on a third-party assessment by the same firm, Cohasset Associates. Two further providers (Backblaze, Wasabi) document Object Lock without claiming a specific regime assessment. The distinction matters for §11: "we support WORM" and "our WORM has been assessed against SEC 17a-4(f)" are different sentences, and only the second one is useful in a regulatory archive design. Azure's documentation adds a detail that is easy to miss and expensive to discover late: an *unlocked* time-based retention policy provides immutability protection but is **not** in a compliant state for SEC 17a-4(f) — the policy must be locked.

**Enterprise key custody narrows the field more than egress does.** Of the challengers, Wasabi's documented encryption options are client-side encryption and SSE-C; Backblaze documents server-side encryption; R2 documents AES-256 with keys managed by Cloudflare and no BYOK path. None of the challengers' pages read here document a cloud-KMS integration equivalent to SSE-KMS or CMEK. If the control framework requires keys in the organisation's own HSM, this row eliminates options regardless of price — a non-commercial constraint deciding a commercial question, which is exactly the point of §11.

**Inventory and audit facilities are the exit's best friend.** The ability to enumerate every object with metadata, checksums and storage class — S3 Inventory, Google's Storage Intelligence insight datasets, Azure's activity and resource logs — is what makes a differential migration and a post-migration verification possible. Providers without an inventory facility force the exit team to list buckets over the API, which is slower, costs requests, and is harder to reconcile.

**Event notifications determine whether the estate can be kept in sync.** A migration designed around a one-month freeze is a different project from one designed around continuous replication with change feeds. Where notification support is undocumented on the pages read (R2, DO Spaces), a migration must either freeze writes or poll.

**Consistency guarantees have converged, and the date matters.** AWS moved S3 to strong read-after-write consistency for all GET, PUT and LIST operations, plus tag, ACL and metadata operations, across all regions and for existing objects, at no charge, announced in the AWS News Blog post "Amazon S3 Update – Strong Read-After-Write Consistency" in **December 2020** (the post states the change was effective "immediately"; the page as fetched does not print a publication date, so the December 2020 date is flagged in §15). That single change removed the last structural reason to maintain consistency workarounds such as EMRFS Consistent View or S3Guard for data-lake workloads, and it shifted the expectation bar: Google, Azure and Oracle all document strong consistency today, and a provider that could not offer it would now be disqualified from analytics use cases on those grounds alone rather than on performance grounds.

**The IAM model is the most under-appreciated lock-in.** Bucket policies and IAM identities are the crown jewels of an object estate — they encode who may read what, and they are the artefact most likely to be provider-specific. Backblaze's S3-compatible surface explicitly does not support IAM roles. DigitalOcean's per-bucket access keys are incompatible with S3 bucket policies, so a bucket uses one model or the other. A migration plan that lists data but not policy is not a migration plan.

---

## 7. The S3-Compatible API Reality

### 7.1 What compatibility actually buys

The S3 REST API is the closest thing object storage has to a lingua franca, and the value is real: the same SDK, the same CLI, the same `rclone` configuration, the same Terraform provider and the same IAM vocabulary can be pointed at a second endpoint. For the 80% of operations that are `PutObject`, `GetObject`, `HeadObject`, `DeleteObject`, `ListObjectsV2` and multipart upload, compatibility reduces a migration from a re-architecture to a configuration change.

What it buys specifically:

- **A second destination you can test against today.** You can write the migration tooling once and run it against two endpoints in parallel, which is the cheapest possible rehearsal for the exit test in §9.
- **Application portability for the common path.** Backup products, data-protection suites and analytics engines that speak S3 gain a second supported target without code change.
- **A pricing lever.** A credible second endpoint is the only mechanism that reliably produces a commercial response from a hyperscaler, because it turns a conversation about discounts into a conversation about switching costs.

### 7.2 Where it breaks, per provider

| Provider | Documented compatibility gaps, from the vendor's own material |
| --- | --- |
| Backblaze B2 | S3-Compatible API does not currently support **ACLs, IAM Roles, Object Tagging, Website Configuration, or browser-based uploads to pre-signed URLs using POST**. Object-level ACLs do not exist (`GetObjectAcl` returns the bucket ACL); `PutBucketAcl` accepts only private/public-read; `x-amz-tagging` is explicitly rejected on `PutObject`/`CopyObject`; `x-amz-tagging-directive` is accepted but ignored |
| DigitalOcean Spaces | "Compatible with a large subset of the S3 RESTful API" (Ceph RGW). Per-bucket access keys are incompatible with S3 bucket policies; `list-objects-v2` pagination is not supported by the Spaces API; access logs only via S3 API or Terraform; no bucket tags; CDN `PUT` via pre-signed URL capped at ~8,100 KiB |
| Cloudflare R2 | S3 API is the high-throughput path (the REST API is capped at 1,200 requests/5 min), object size 5 TiB, 10,000 parts, one concurrent write per second per key, key length 1,024 bytes, metadata 8,192 bytes |
| Oracle OCI | Publishes a separate "Amazon S3 Compatibility API" alongside its native Object Storage API; limits include a 50 GiB per-object PutObject ceiling even though the maximum object size is 10 TiB, so large objects must use multipart |
| Google Cloud Storage | XML API interoperable surface; the JSON API is the richer interface, and features such as Bucket Lock and turbo replication management are restricted to the JSON API (turbo replication "cannot be managed through the XML API") |
| Azure Blob | The S3-compatible story is the weakest of the hyperscalers; Blob's idiomatic surface is its own REST API, with hierarchical namespace (ADLS Gen2) semantics layered on top |
| Wasabi | S3-compatible with validated integrations; the free-API-request policy is explicitly tied to using applications Wasabi has validated, which makes an unvalidated custom client a policy risk as well as a technical one |
| MinIO | Deliberately implements the S3 API as its primary interface and extends it (S3 Express, SFTP, Iceberg Catalog, Delta Sharing) — the on-prem field is covered in the lakehouse guide's §5 and §13 |

### 7.3 The operations and SDK behaviours that differ

These are the failure modes that a "we support S3" claim does not cover, in the order they tend to surface:

1. **Permissions semantics.** S3 bucket policies and IAM roles have no equivalent on several challengers. A policy document will be rejected, or worse, silently ignored. Backblaze's absence of IAM roles and DigitalOcean's mutual exclusion between per-bucket keys and bucket policies are both documented, vendor-stated facts.
2. **Tagging and metadata.** Object tagging is absent on Backblaze's S3 surface and unsupported on Spaces. If lifecycle, cost allocation or access control depends on tags, the design must move to key prefixes before migration, not after.
3. **ACLs.** Object-level ACLs are effectively gone from the challengers. Any code path that calls `PutObjectAcl` or `GetObjectAcl` needs replacing with policy-based authorisation.
4. **Presigned URL semantics.** `POST`-based browser uploads to presigned URLs are not supported by Backblaze's S3-compatible API. The `PUT` variant works. This breaks a common web-upload pattern with no error until a browser hits it.
5. **Event-notification shapes.** S3 event notifications, Google Pub/Sub notifications and Backblaze event notifications are structurally different products; a notification-driven pipeline is a re-integration, not a config change.
6. **Multipart thresholds and part sizes.** R2 caps parts at 10,000 and objects at 5 TiB; Oracle caps parts at 10,000 with 50 GiB per part and a 50 GiB `PutObject` ceiling; DigitalOcean caps presigned CDN payloads at ~8,100 KiB. Multipart tuning that worked at the origin will need re-tuning.
7. **Listing and pagination.** `list-objects-v2` pagination is documented as unsupported by the Spaces API. Any inventory reconciliation built on paginated listing needs a per-provider code path.
8. **Error codes and retry semantics.** Object storage retries are usually implemented against `503 Slow Down` with exponential backoff (DigitalOcean documents exactly this). Retry logic tuned on one provider's throttling behaviour may be too aggressive or too passive on another.
9. **Strong consistency expectations.** Now that AWS, Google, Azure and Oracle all document strong consistency, code that was written against eventual-consistency semantics is no longer a problem — but code written *for* a provider that lacked it may contain workarounds (existence-check tables, S3Guard-style layers) that should be removed, not ported.
10. **Free-tier and rounding behaviour.** R2 rounds usage up to the next billing unit and excludes IA from the free tier. A test migration that fits in a free tier on one provider can generate a bill on another.

### 7.4 The honest conclusion

API compatibility lowers migration cost; it does not make providers interchangeable. The correct mental model is a **spectrum with three tiers**, consistent with the lakehouse guide's §13:

- **Tier 1 — functional interchangeability for the common path.** Data-movement operations and simple object CRUD behave the same. A migration tool can be written once.
- **Tier 2 — configuration-different.** Limits, part sizes, list semantics, request-class pricing and notification shapes differ, and require per-provider configuration and testing.
- **Tier 3 — capability-absent.** IAM roles, object tagging, object ACLs, website configuration, POST-presigned uploads, inventory products, KMS integration and Object Lock regime assessments are simply not there on some providers. These are the ones that make a migration a project.

The practical test for a design review is not "is the target S3-compatible?" but "which of my ten most important API calls are Tier 3 on the target?" That question has a verifiable answer, and the vendors' own documentation is where it lives.

---

## 8. Durability, Availability and the SLA

### 8.1 What a durability figure means, and what it excludes

Eleven nines of durability is a statement about the *modelled annual probability of hardware-induced data loss* under the provider's own redundancy design. AWS states Standard is "designed to provide 99.999999999% durability"; Google states Cloud Storage is "designed for at least 99.999999999% (11 9's) annual durability, regardless of storage class and location type"; Azure states LRS offers "at least 99.999999999% (11 9s)" and ZRS "at least 99.9999999999% (12 9s)"; Backblaze's Vaults architecture "calculates at 99.999999999% (11 x 9s) annual durability"; Wasabi is "designed for 11x9s … with redundancy and repair".

The figure excludes, by construction:

- **Deletion.** Azure says it outright: "All replicas reflect the same current state: deletions and overwrites are applied to all copies simultaneously. Redundancy protects against hardware failure, not against data-modifying operations." Eleven nines protects you from a failed disk and does nothing to protect you from a `DeleteObject` call, a misconfigured lifecycle rule, or an object-lock policy that was never locked.
- **Account compromise.** Ransomware against a storage account is a *credential* problem. Nothing in a durability figure addresses it. This is the gap that Object Lock, legal hold, multi-user authentication and Wasabi's Covert Copy architecture exist to close, and it is why immutability belongs in a durability conversation at all.
- **Billing suspension.** DigitalOcean states that accounts with unresolved billing issues may lose access to Spaces, and that a payment failure can result in lost access "while billing remains unresolved". Microsoft states that if a bill goes unpaid with an active retention policy, "normal data retention policies apply as stipulated in the terms and conditions of your contract". Durability does not survive non-payment.
- **Vendor failure.** No provider prices this, and no durability figure covers it. See §8.4.
- **Region destruction.** Google is explicit: data that has not finished replicating before a region becomes unavailable "is inaccessible until the downed region comes back online", and "could potentially be lost in the very unlikely case of physical destruction of the region". Azure's LRS documentation is equally blunt: if a disaster such as fire or flooding occurs in the datacentre, "all replicas of a storage account that uses LRS might be lost or unrecoverable".

### 8.2 Availability SLAs and the credit model

| Provider | Commitment | Credit structure | Notable terms |
| --- | --- | --- | --- |
| AWS S3 | 99.9% monthly uptime (Standard, Express One Zone, Glacier Flexible, Deep Archive); 99.0% for Intelligent-Tiering, Standard-IA, One Zone-IA, Glacier IR | 10% / 25% / 100% of affected class charges | Credits apply only against future payments; "will not entitle you to any refund or other payment"; "sole and exclusive remedy" is the credit; must claim within two billing cycles with logs |
| Google Cloud Storage | 99.95% Standard multi/dual-region; 99.9% Standard regional or Nearline/Coldline/Archive multi/dual-region; 99.0% Nearline/Coldline/Archive regional | 10% / 25% / 50%, **capped at 50% of the monthly bill** | Must notify support within 30 days; credits applied within 60 days; reseller arrangements change who receives them |
| Azure Blob | Published in a separate SLA document; not read for this guide | — | — |
| Oracle OCI | Not read for this guide | — | Oracle documents request-rate capacity as "a Service Level Objective only" |
| Cloudflare R2 | Not read for this guide | — | — |
| Backblaze B2 | Not located for this guide | — | — |
| Wasabi | Referenced as a PDF (21 Feb 2024 reference version on the SLA page); not read | — | — |
| DigitalOcean Spaces | Cold Storage documented at 99.5%; Standard not read | — | — |

Three structural facts about every service credit, and they are the same at every provider:

1. **A credit is a discount on the next bill, not compensation.** AWS states credits apply only against future payments and do not entitle you to a refund. Google caps total credits at 50% of the affected monthly bill.
2. **The ceiling is the *storage class* charge, not the business impact.** A 100% credit on a $20,000/month object-storage bill is a fraction of the cost of a failed regulatory submission or a distribution outage. Service credits are a remedy for a billing event, not for an incident.
3. **Credits are claimed, not granted.** Both AWS and Google require the customer to file a claim with evidence inside a deadline. An organisation that does not monitor its own availability and keep request logs will not receive the credit it is entitled to.

The design conclusion is that the SLA is a procurement artefact and not a resilience mechanism. The resilience mechanism is architecture: replication across regions, immutability for the data that must not be destroyed, an independent copy outside the primary provider, and a restore test that actually runs.

### 8.3 Single-AZ versus multi-AZ

The classes differ in zone redundancy, and the class you choose silently decides your exposure:

- AWS documents that Standard, Intelligent-Tiering, Standard-IA and all three Glacier classes store objects "on multiple devices across a minimum of three Availability Zones", while **S3 One Zone-IA stores data redundantly across multiple devices within a single Availability Zone**. One Zone-IA is priced at $0.01/GB-month against Standard-IA's $0.0125 — a 20% saving for the loss of zone-level durability.
- Azure's ZRS copies data synchronously across three or more zones; LRS stays in one physical datacentre and "protects your data against drive, server, and rack failures" but not against a datacentre-level event.
- Google's regional buckets store across at least two zones and tolerate the loss of any one; zonal (Rapid) buckets are locally redundant and Google states that "locally redundant storage is most suitable for data that can be replaced or reconstructed".
- Oracle's Object Storage is regional, storing "redundantly across several storage servers".

Single-zone classes are legitimate and frequently correct — for reproducible data. They are indefensible for a system of record, and no durability figure in a marketing table will tell you which class you bought. The class name will.

### 8.4 The going-concern question

This is the part of the analysis that commercial comparison guides omit and that a bank's operational-resilience function will not.

The smaller challengers are, structurally, single-business companies whose product is storage. Backblaze is a publicly traded company (BLZE) and states its financial position through investor relations; its compliance page also states that it is listed as a Progressing Product under GovRAMP and holds TX-RAMP Provisional status — the latter a state-level authorisation. Wasabi, Cloudflare and DigitalOcean are likewise going concerns with published reporting obligations of their own. But the *exposure* is different from a hyperscaler's, and the honest framing is:

- **A hyperscaler's storage service is not going to be discontinued while you use it.** The risk to manage is commercial (price increases, feature deprecation, egress policy), not existential.
- **A specialist provider's storage service is its entire business.** That is a strength — focus, cost structure, and no conflicting incentive to push you onto compute — and it is also a concentration risk, because the same fact means the product line's discontinuation is the company's discontinuation.
- **The mitigations are contractual and architectural, not analytical.** Contractual: an escrow or data-return clause with a defined format and timeline, a termination-for-convenience right on the provider's side that triggers an exit window, or a "source code and data return" undertaking. Architectural: a second copy in a second provider, and a restore path that has been exercised.

What this changes in the cost model is that the second copy is not insurance you can defer. A design that holds 100% of a regulated estate with a single specialist challenger has taken an uninsurable position, whatever the price per TB says. The correct shape for a risk-averse estate is usually one primary and one independent copy in a different provider, which — as §12 shows — improves the cost model rather than degrading it, for most archetypes.

---

## 9. Lock-In and Exit Analysis

### 9.1 What actually locks you in, in order of strength

| Mechanism | Why it holds | How to neutralise it |
| --- | --- | --- |
| **Egress economics** | The cost of the exit scales with the size of the estate, and the rate is highest in the smallest month | Negotiate an exit egress waiver or a bulk-transfer credit at signature time; price the exit before you commit; prefer providers with capped or zero egress for read-amplified workloads |
| **Proprietary capabilities with no equivalent** | Object Lock assessed against a regime, inventory products, IAM model, event notification shapes, lakehouse catalogs | Keep the proprietary layer thin and the data layer standard; treat any proprietary feature as a one-way door in the design record |
| **The IAM and policy model** | Bucket policies, roles and identities encode the access model and are the least portable artefact | Generate policy from a provider-neutral source of truth; test the whole policy set against the target during the rehearsal |
| **The ecosystem built on top** | Compute, analytics engines, catalogs, orchestration and CI all bind to the storage endpoint and its adjacent services | Keep consumers behind an abstraction (an S3 endpoint name that can be re-pointed, or a gateway) and avoid hard-coding vendor ARNs or service roles into applications |
| **Commercial commitments** | Azure reserved capacity (1- and 3-year blocks from 100 TB), Backblaze annual capacity commitments with prepayment at contract start, Wasabi 1/3/5-year Reserved Capacity, hyperscaler private pricing | Model the break fee: Azure states reserved capacity can be cancelled with an early-termination fee of 12% of the remaining balance |
| **Operational familiarity** | Runbooks, monitoring, cost tooling and staff skills | Costed into the migration estimate explicitly; this is usually the largest indirect exit cost and the least modelled |

### 9.2 What makes exit feasible

- **A standard API on both ends** (§7) — the migration tool is written once.
- **Standard formats** — objects are bytes with a key and HTTP metadata; a manifest, checksums and a key list are provider-neutral artefacts.
- **Reproducible tooling** — `rclone`, `s5cmd`, `mc`, `aws s3 sync` against a custom endpoint, or the provider's own migrator. R2's Super Slurper and Sippy, Backblaze's assisted migration and Universal Data Migration, Oracle's Data Transfer Service appliance and Wasabi's Wasabi Ball all exist and are priced or quoted on their pricing pages, which means the exit is a purchasable service rather than an impossibility.
- **An inventory reconciliation capability** — you must be able to enumerate both ends and diff them, with checksums, and this is an argument for providers with an inventory product.
- **A written exit test** — the only credible proof, see §9.5.

### 9.3 The exit cost model

An exit has five cost lines. Most business cases include only the first, and it is usually not the largest.

| Line | What it contains | How to size it |
| --- | --- | --- |
| **Data transfer** | Egress charges from the source provider (or the waiver), plus any appliance shipping or network upgrade | Estate size × source egress rate, computed on your provider's actual volume tiers, including the fact that a large one-off transfer changes your tier |
| **Request charges at both ends** | Source reads/listings; destination writes; multipart completions; any per-1,000-object fee | Object count × (source GET + destination PUT); for 10 billion objects this line dwarfs the transfer line on providers that charge per request |
| **Rehydration and tiers** | Moving data out of an archive class requires a restore, which costs a retrieval fee, takes hours to days, and *creates a temporary second copy billed at a warmer rate* (AWS states the restored copy is charged at S3 Standard rates for the duration you choose) | Estate size × retrieval fee + temporary duplicate storage × duration |
| **Tooling and time** | Migration tool licence, contractors, dual-running cost, reconciliation effort, and the write-freeze window's business cost | Usually equal to or greater than the transfer cost for a first migration |
| **Reversal risk** | The cost of the migration *back* if the new provider disappoints, which is the same five lines in reverse and is never zero | Model it. A migration whose reversal cost was never modelled is a one-way door presented as a decision |

### 9.4 Design rules that keep an exit possible

1. **Keep the data plane standard and the control plane thin.** Objects, keys and metadata are portable; policies, locks and inventory products are not. Anything proprietary should be justified in writing as a one-way door.
2. **Own your key material where the regulator requires it.** Client-side encryption makes the data portable and the provider irrelevant to confidentiality; it also makes key rotation your problem. Choose consciously.
3. **Replicate to a second provider, not just a second region.** A second region in the same provider is availability insurance, not exit insurance.
4. **Delete the egress surprise by pricing it at design time.** Compute the full-egress number for the estate and record it next to the monthly storage number in the design document, so the exit cost is a known figure rather than a discovery.
5. **Avoid the minimum-duration landmine in the design, not in the migration.** Data you will churn (backup, logs, scratch) belongs in a no-minimum class — a hot class, or a provider with no minimum — and data you will keep belongs in the cold class. Getting this backwards is the single most common cost error in object storage.
6. **Keep the object count sane.** Object count is a billing dimension on several providers and a migration complexity on all of them. Compacting small objects before a migration can pay for itself.
7. **Write the exit test into the contract and the calendar.** An annual restic restore of a statically meaningful sample, into a different provider, with checksum verification and a measured RTO. Both the vendor-management and operational-resilience content elsewhere in this repository applies here, condensed to one sentence: an untested restore path is an assumption, not a control.

### 9.5 The only credible proof

An architecture diagram showing a second provider is not evidence of exit-readiness. Evidence consists of:

- a **dated restore/re-host test** performed into the target provider,
- a **checksum reconciliation** across both estates,
- **measured** elapsed time and cost, against a stated objective,
- a **documented failure** list of the API calls and policies that did not port,
- and the identity of the person accountable for the next test.

Everything else — compatibility matrices, letters from vendors, architectural diagrams — is a claim about a system, and claims about systems are falsified by tests. The exit test is the object-storage equivalent of the backup restore test, and it fails for the same reason: nobody schedules it until an incident schedules it for them.

---

## 10. Performance and Access-Pattern Reality

### 10.1 The four access patterns, and what each one actually stresses

Cost is not the only axis on which providers differ, and for some workloads it is not the deciding one. The access pattern decides which performance property matters, and the properties are not the same property.

| Access pattern | What it stresses | Which documented limit decides | Which cost component follows |
| --- | --- | --- | --- |
| **Distribution** (media, reports, public assets) | Sustained outbound bandwidth; concurrent GETs; CDN cache-fill | Egress path and CDN integration; account-level bandwidth ceilings | Egress, overwhelmingly |
| **Analytics / data lake** | Throughput per prefix or bucket; listing; metadata operations; consistent reads | Per-prefix request rate; consistency model; object size vs part size | Storage + requests; egress only if the lake is read from outside the provider |
| **AI training corpus** | Sustained multi-pass reads at high aggregate rate; large sequential reads | Egress rate and egress *allowance shape*; platform throughput | Egress, and the allowance shape decides it (§3.6) |
| **Logs / small objects / IoT** | Operations per second at high cardinality; per-object billing floors | Per-bucket ops ceiling; minimum billable object size; `PUT` payload ceiling | Requests + per-object floors (§3.7) |

The point of the table is that a provider can be excellent at one and poor at another, and a single "performance" comparison obscures this. A provider with a generous egress allowance and a low per-bucket ops ceiling is a good archive and a bad log store.

### 10.2 Documented limits — the numbers a design may rely on

The rows below are **vendor-documented operational limits**, not benchmarks. They are drawn from the same pages cited in §5 and are the kind of statement a design review can act on without an argument about measurement methodology.

| Provider | Documented request-rate limit | Documented size / part limits | Documented egress or account ceiling |
| --- | --- | --- | --- |
| AWS S3 | 3,500 `PUT`/`COPY`/`POST`/`DELETE` and 5,500 `GET`/`HEAD` per second **per prefix**; a documented floor, not a ceiling to be afraid of (S3 consistency/performance documentation) | Standard multipart practice; Express One Zone is a separate class | No published account-level ceiling |
| Google Cloud Storage | Class- and location-dependent; the JSON API is the richer interface, and some features are JSON-API-only | Per-object and part limits documented per class | No published account-level ceiling |
| Azure Blob | **40,000 requests/second per account** in the listed major regions, **20,000** elsewhere (Blob scalability targets) | Per-tier object limits documented per redundancy option | **200 Gbps egress** default in listed regions, **50 Gbps** elsewhere; 250 storage accounts per region per subscription; 5 PiB default capacity |
| Oracle OCI | **12,000 read / 3,000 write / 2,000 list RPS** in three-availability-domain regions; **5,000 / 3,000 / 2,000** elsewhere; increases on request at multi-petabyte scale | Maximum object 10 TiB; maximum part 50 GiB; `PutObject` ceiling 50 GiB; 10,000 parts; 4,000 bytes of metadata | Request-rate capacity is documented as an SLO, not an SLA |
| Cloudflare R2 | REST API capped at **1,200 requests per five minutes per account**; the S3 endpoint is the high-throughput path; the `r2.dev` endpoint throttles past "hundreds of requests/second" with `429` responses and is explicitly not for production | Object 5 TiB; 5 GiB single-part; 10,000 parts; key 1,024 bytes; metadata 8,192 bytes; **one concurrent write per second to the same object name**; 50 bucket-management ops/second/bucket | No published account-level ceiling |
| Backblaze B2 | No published per-bucket request ceiling on the pages read | No minimum file size or storage duration | No published account-level ceiling |
| Wasabi | Free API requests are **policy-bound**, not technical: the policy reserves the right to limit or suspend a use case deemed an unreasonable burden | 4 KB minimum billable object size | None published; egress is policy-conditional rather than rate-limited |
| DigitalOcean Spaces | **800 total operations per second per new bucket** (older buckets lower) — an order of magnitude below what a log workload needs | `PUT` up to 5 GB; multipart to 5 TB; 100 buckets and 200 access keys per account; 100M unversioned / 50M versioned objects per bucket | 1 TiB/month outbound included in the $5 subscription, then $0.01/GiB |

The R2 row is the one that most often surprises: an architecture chosen for its egress economics can be defeated by a limit that has nothing to do with egress. The DigitalOcean row is the mirror image: a genuinely attractive price per GiB behind an ops ceiling that restricts the workload to small estates.

### 10.3 Latency and throughput — and who says so

Every latency and throughput figure in this space is someone's measurement, and the three sources are not equivalent:

- **Vendor-documented limits** (§10.2) — verifiable, contractual or near-contractual, and the only class a design should hard-depend on at this commit.
- **Vendor-published benchmarks and product claims** — for example AWS's positioning of S3 Express One Zone around single-digit-millisecond latency for a single-AZ class, or Google's stated throughput advantage for its gRPC interface over HTTP for analytics workloads. These are the vendor's own claims about its own product. They are usually directionally right and they are not independent evidence; the S3 Express One Zone class is priced at $0.11/GB-month in the check performed here (§5.1), roughly five times S3 Standard, so the latency tier is a *purchased* property.
- **Independent measurement** — third-party benchmarks and published research. None were run or collected for this guide; any figure in this class would be a claim about someone else's test conditions. Treat a table of such numbers as a starting hypothesis to be re-tested against your own access pattern, never as a procurement input.

Two cost-relevant performance facts belong here rather than in §8. First, **retrieval is a latency tier, not just a fee**: Azure documents that rehydration from the Archive tier to Hot or Cool "typically takes up to 15 hours", and AWS's Glacier tiers are documented with restoration times from one to five minutes (Expedited) through 3–5 hours (Standard) to 5–12 hours (Bulk) for Flexible, and up to 12 hours Standard / 48 hours Bulk for Deep Archive. A fee schedule that prices retrieval at $0.02/GB does not disclose that the restore takes half a day; the availability property is the one that disqualifies a class for a workload, and the price is secondary. Second, **the smallest object you can economically store is larger than the object you have**: 4 KB at Wasabi and DigitalOcean Standard, 128 KiB at DigitalOcean Cold, 128 KB for AWS Standard-IA, One Zone-IA and Glacier Instant Retrieval. In the small-object archetype of §3.7 the AWS Standard-IA configuration bills 1.31 PB of "storage" for 50 TiB of data — a 26× inflation caused entirely by a floor that appears in a footnote rather than in a pricing table.

### 10.4 The small-object problem, stated once

Small objects break object storage in three different ways at once, and only one of them is a price:

1. **Per-request pricing** turns a per-byte problem into a per-object problem. At 200M writes/month, Google's Class A at $0.005 per 1,000 costs $1,000 and AWS's $0.005 per 1,000 costs the same — against zero on Backblaze and Wasabi.
2. **Minimum billable object sizes** bill you for bytes you do not have (§10.3).
3. **Per-object monitoring and management fees** can exceed the storage line entirely: AWS Intelligent-Tiering's $0.0025 per 1,000 objects per month is $25,000/month at ten billion objects (§3.5, §3.7).

The structural fix is compaction — aggregating small objects into larger ones and maintaining an index — and the structural cost of compaction is that the index becomes part of the estate that must be migrated, checksummed and reconciled. A compaction decision made for cost reasons creates a migration dependency that did not exist before it.

### 10.5 Multipart upload, and where the tuning does not port

Multipart upload is the one operation that is both universal and differently specified everywhere, which is why it appears in §7.3 as a failure mode and here as a performance property. What matters in a design:

- **Part count and size ceilings differ per provider** (R2: 10,000 parts / 5 TiB object; Oracle: 10,000 parts / 50 GiB per part / 50 GiB `PutObject` ceiling; DigitalOcean: multipart to 5 TB, presigned CDN payloads capped near 8,100 KiB). Part sizes tuned on one provider can exceed another's per-part ceiling or its `PutObject` ceiling and force a rewrite of the upload path.
- **Concurrency is capped per key, not per bucket, on at least one provider** — R2 documents one concurrent write per second to the same object name. A parallel-upload design that multiplexes to a single key will throttle there and not on the hyperscalers.
- **Incomplete multipart uploads are billed by some providers and auto-reaped by others.** DigitalOcean deletes incomplete multipart uploads older than 30 days automatically; elsewhere, an aborted-but-unreaped upload is billable storage that appears in no inventory unless a lifecycle rule was written for it. An exit plan that ignores orphaned parts undercounts the estate.

The honest summary is that multipart is where "S3-compatible" is most nearly true for the common case and most likely to bite in production, because the tuning parameters that make it fast are the ones that differ.

### 10.6 CDN integration — where the egress economics actually land

For every distribution workload, the object-storage provider is rarely the last hop. That matters because the egress bill can be moved, and moving it is a real architectural decision rather than a pricing dodge. There are three states, and they are not equivalent:

| State | Mechanism | Where the cost lands | Evidence from the pages checked 19 Sep 2026 |
| --- | --- | --- | --- |
| **Billed per GB, directly from the origin** | Objects served straight from the object-storage endpoint | Object-storage egress line | AWS $0.09/GB, Google $0.12/GiB, Azure $0.087/GB at the first tiers |
| **Waived from origin to CDN** | Origin→CDN transfer at no charge; the CDN's own egress is billed separately | CDN egress line — moved, not eliminated | AWS prices S3→CloudFront at **$0.00/GB**; Google waives Cloud Storage data-transfer charges for Cloud CDN and Media CDN but states "cache fill charges may apply"; DigitalOcean Spaces **bundles a CDN** at no additional cost; Backblaze grants unlimited free egress to named partner CDNs |
| **Never billed at origin** | R2 on its own egress paths; Wasabi subject to fair use; Backblaze inside its 3× storage allowance | No egress line at all, within the stated conditions | §4.2 |

Two consequences follow, and both belong in a design document. First, **the mitigation is available to nearly everyone** — the hyperscalers waive origin-to-CDN transfer just as the challengers waive partner-CDN egress — so a comparison that shows a 20× spread on the raw direct-egress line (§3.4) is a comparison of operating models, not of intrinsic capability. Second, **CDN egress is not free, and CDN egress rates were not verified for this guide** (§15). A cost model that moves the traffic to a CDN and declares victory has moved the unknown to a different line item. The correct statement for a design review is that a CDN-fronted distribution architecture collapses the *origin* egress differential and replaces it with a CDN contract, whose commercial terms are a separate negotiation with a separate counterparty — and, if that CDN is the same vendor as the storage challenger, a new dependency of exactly the kind §4.5 warns about.

---

## 11. The Regulated-Enterprise and Banking Angle

Financial institutions are not simply large buyers of object storage. They are buyers for whom several of the axes above are constrained by rules that no price can move, and this section states what those constraints are and which providers survive them. **No bank is named anywhere in this guide; the worked example in §12 is fictional.**

### 11.1 Data residency and sovereignty

Residency is the first gate because it eliminates options rather than merely re-ranking them. An estate that must remain in a named jurisdiction restricts the candidate set to providers with a region in that jurisdiction *and* — importantly — a control on where metadata, replicas and backup copies live. The distinction that catches organisations out is between residency (where the data is stored) and **sovereignty** (who can be compelled to disclose it, and under whose law the provider operates). A provider with a local region but a foreign parent and foreign support access satisfies the first and not the second.

From the matrix in §6.1, the data-residency depth row reads: AWS ✅ many regions plus sovereign cloud; Google ✅ regions, dual and multi-region; Azure ✅ many regions plus sovereign and government; Oracle ✅ regions plus government; Wasabi ✅ named regions; Backblaze ◐ **US and EU regions**; Cloudflare R2 ◐ jurisdictional data-location controls, not priced in this guide; DigitalOcean Spaces ◐ limited metros. For an estate with a hard residency requirement outside North America and Europe, that row alone removes Backblaze from consideration, and R2 requires the jurisdictional-control question to be settled — which this research did not settle (§15). On-premises options (MinIO and the field owned by the lakehouse guide's §5) satisfy residency by construction and are the only options that satisfy it without a contractual argument.

A related and frequently missed point: **replication crosses borders**. Inter-region replication is a billed transfer (§4.4) and it is also a residency event. A design that satisfies residency at the primary region and replicates to a second region for durability has created a second residency question, and the durability argument is not a sufficient answer to it.

### 11.2 Audit and evidence — immutability is a control, not a feature

The regulatory-archive requirement is the one that most often decides a provider, and it is decided by paperwork as much as by capability. From §5 and §6.1:

| Provider | WORM capability as the vendor documents it | Regulatory regime the vendor itself claims |
| --- | --- | --- |
| AWS S3 | Object Lock, retain-until-date and legal hold | "Assessed by Cohasset Associates for use in environments that are subject to SEC 17a-4, CFTC, and FINRA regulations" |
| Google Cloud Storage | Bucket Lock, Object Retention Lock, event-based holds | Bucket Lock "can help with regulatory and compliance requirements, such as those associated with FINRA, SEC, and CFTC" |
| Azure Blob | Time-based retention (1 day to 146,000 days) and legal holds, at container or version level | Cohasset validated immutable storage "meets the relevant storage requirements of CFTC Rule 1.31(c)-(d), FINRA Rule 4511, and SEC Rule 17a-4(f)" |
| Oracle OCI | Retention rules documented | None asserted in the pages read |
| Cloudflare R2 | Bucket locks: up to 1,000 rules, prefix-scoped, duration / until-date / indefinite, strictest-wins, locks override lifecycle | R2 documentation points to the Cloudflare Trust Hub only |
| Backblaze B2 | Object Lock and legal hold documented on both API paths | No regime-specific assessment claimed |
| Wasabi | Object Lock documented; multi-user authentication for lock changes | "applicable SEC standards" claimed in trust-centre prose |
| DigitalOcean Spaces | **No Object Lock or WORM capability documented on the pages read** | None — eliminates Spaces from any WORM archive requirement |
| MinIO AIStor | Immutability documented | "SEC 17a-4(f) compliant", "FINRA 4511(c)-(d) compliant" |

Two sentences in that table are load-bearing. **"We support WORM" and "our WORM has been assessed against SEC 17a-4(f)" are not the same sentence**, and only the second one is useful in a regulatory-archive design; three of the four regime claims rest on an assessment by the same firm, which is a detail an auditor may or may not care about but a designer should at least know. **Azure's documented nuance is the one that costs money late**: a time-based retention policy that has not been *locked* provides immutability protection but is not in a compliant immutable state for SEC 17a-4(f), and the warning is stated in Microsoft's own documentation. An archive built on unlocked policies has the appearance of compliance without the property.

The evidence facilities matter as much as the lock. An archive needs to be able to demonstrate, years later, what was written, when, by whom, and that it has not changed: that is Object Lock plus versioning plus an audit log (S3 CloudTrail data events, Google's detailed audit logging, Azure's activity and resource logs and retention-policy audit log, Wasabi's bucket logging, MinIO's audit trail). Retention-policy audit logging itself is uneven — Azure documents retention-policy audit logging at container level while stating that version-level policy changes are not audited. A control that is not logged is a control that cannot be evidenced.

### 11.3 Third-party concentration and the exit-plan obligation

Object storage is rarely a standalone vendor relationship in a bank; it is usually a material third-party service, and material third parties now carry obligations that are independent of price:

- **A documented exit plan with a defined timeframe** — which is what §9 is, in cost-model form. The regulator's question is not "could you leave?" but "what is the plan, who owns it, and when was it last tested?"
- **Audit and access rights** over the provider, and notification of subcontractor changes — relevant where the provider's own supply chain (data centres, CDNs, migration tooling) is part of the service.
- **Concentration risk**, which is the §8.4 question formalised: a single provider holding the whole estate, or the whole estate of a critical function, is a concentration. The mitigation is a second copy in a second provider, and §8.4 already noted the counter-intuitive result that for most archetypes this *improves* the cost model rather than degrading it — because the second copy goes to a challenger, not to a second hyperscaler.
- **Dependency on a single legal entity's survival**, which for a specialist storage provider is the going-concern question of §8.4 in regulatory clothing. Contractual mitigations — a data-return clause with a defined format and timeline, escrow, an exit window triggered by termination for convenience — are the means by which a specialist provider becomes acceptable to a risk function.

### 11.4 Encryption and key custody

This is the constraint that most often surprises a team that has already chosen a provider on price. If the control framework requires that the organisation hold and control the keys — BYOK or HYOK, with key material inside the institution's own HSM and key destruction as a documented capability — then the candidate set narrows to providers that document a cloud-KMS integration with customer-managed keys.

From §6.1: AWS documents SSE-S3, SSE-KMS, SSE-C and DSSE with FIPS endpoints; Google documents Google-managed keys, CMEK and CSEK, with key-destruction protections interacting with locked buckets (a key version cannot be destroyed while locked objects are inside their retention window); Azure documents Microsoft-managed keys, customer-managed keys and infrastructure double encryption; Oracle documents AES-256 with keys created at upload plus KMS. Of the challengers, Wasabi's documented options are **client-side encryption and SSE-C**; Backblaze documents server-side encryption and application keys; R2 documents AES-256 with keys managed by Cloudflare and **no BYOK path** on the pages read.

The practical resolution is that client-side (application-level) encryption is a legitimate way to satisfy the requirement *independently of the provider* — it makes the data portable and the provider irrelevant to confidentiality — at the cost of making key management, rotation and crypto-shredding the institution's own problem, and at the cost of giving up server-side features that operate on plaintext. That is a genuine trade, not a workaround, and it should be recorded as one.

### 11.5 The blunt statement

**A cheap-egress provider with a weak compliance posture is not a saving.** The arithmetic that makes a challenger attractive is an arithmetic on cost lines; the gates above are gates on eligibility. If a provider cannot be named in the regulatory-archive design because it claims no regime assessment, has no documented KMS integration, or cannot evidence retention-policy auditing, then its $7.99/TB is not a price — it is a price for a different service, and the comparison with the hyperscaler was never like-for-like. The correct use of the cheap-egress challengers is on the workloads where the gates do not bind: derived analytics, distribution of published material, backup copies of reproducible data, and the *second copy* in a two-provider design. That is precisely the shape §12 arrives at, and it is arrived at from the gates rather than from the price list.

---

## 12. The Cymbal Bank Worked Example

> **This scenario is fictional and every figure in it is illustrative.** Cymbal Bank is an invented institution used across this repository as the only bank persona. The providers, prices and structures are real and dated 19 September 2026 per §5; the workloads, the estate sizes and the recommendation are constructed for the exercise. Nothing here is a quotation, and no real institution's estate is described.

### 12.1 The estate

Cymbal Bank's object-storage estate has grown organically across three workloads that have almost nothing in common except that they are all objects:

| Workload | Size | Growth | Access pattern | Retention | What it is |
| --- | --- | --- | --- | --- | --- |
| **Regulatory archive** | 500 TiB | ~15%/year | Write-once, read rarely; audit-driven bursts | 10 years, immutability required | Trade records, statements, correspondence, execution evidence |
| **Analytics data lake** | 1 PiB | ~30%/year | ~5% read per month; multi-engine (Spark, Trino) | 3–7 years | Derived and curated datasets, feature tables, model inputs |
| **Report and media distribution** | 100 TiB | ~10%/year | Heavy outbound: ~200% of the estate per month | Published artefacts, mixed | Customer-facing statements, regulatory publications, marketing media |

Two constraints come from outside the storage team and they are the ones that decide the outcome. First, **regulatory records must remain in the home jurisdiction** (a jurisdiction in Asia-Pacific; the residence requirement is contractual and legal, not architectural). Second, **regulatory records must be held in a WORM state that the bank can evidence against the retention regime it is subject to**, which in practice means a provider that asserts a third-party assessment against that regime in its own documentation.

### 12.2 The gates that eliminate options, before any arithmetic

The order matters. Price is applied last.

| Gate | What it is | Who it eliminates |
| --- | --- | --- |
| **Residency** | Regulated records must remain in-country; replication and metadata must not leave | **Backblaze** — its documented regions are US and EU, which fails an APAC residency requirement outright (§6.1, §11.1). **DigitalOcean Spaces** — limited metros. **Cloudflare R2** — jurisdictional data-location controls exist but were not verified for this guide (§15), so R2 is held pending evidence rather than accepted |
| **WORM with a regime assessment** | The vendor must claim a third-party assessment, not merely a lock feature | **Backblaze, Wasabi, R2** — all document Object Lock; none claims a regime assessment (§11.2). **DigitalOcean Spaces** — no Object Lock documented at all |
| **Key custody** | Keys must be under the bank's control (own HSM, with destruction capability) | The same three challengers unless client-side encryption is adopted (§11.4) |
| **Operational ceiling** | The archive is read in audit bursts; the lake runs multi-engine analytics | **DigitalOcean Spaces** — 800 ops/second per bucket is below the analytics workload's requirement (§10.2) |

The archive therefore has **four surviving candidates: AWS S3, Google Cloud Storage, Azure Blob, and a self-hosted MinIO deployment in the bank's own datacentres** (the on-premises field is owned by the lakehouse guide's §5). The analytics lake and the distribution workload carry no regulatory-regime requirement, so for them the residency gate alone applies — which still removes Backblaze, and still holds R2 pending evidence, but leaves Wasabi and the hyperscalers in play.

### 12.3 The arithmetic, per workload per provider

All figures are monthly, illustrative, and derived from the rates and structures in §5 under the assumptions in §3.1. The archive is priced on each provider's archive class; the lake on each provider's standard/hot class; distribution on each provider's standard class including its own egress or allowance rules.

| Workload | AWS | Google | Azure | Oracle | Cloudflare R2 | Backblaze B2 | Wasabi |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Archive, 500 TiB (cold class) | **$512** (Glacier Deep Archive) | $655 (Archive) | $1,059 (Archive) | $1,429 (Archive) | $5,500 (IA) | $3,821 | $4,288 |
| Data lake, 1 PiB (standard class) | $22,592 | $20,980 | $18,229 | $28,714 | $16,897 | **$7,825** | $8,782 |
| Distribution, 100 TiB @ 200% egress | $16,438 | $23,151 | $15,870 | $3,658 | $1,657 | **$764** | $858 |
| **Single-provider total** | **$39,542** | $44,786 | $35,158 | $33,801 | $24,054 | $12,410 | $13,928 |
| **Annual** | **$474,504** | $537,432 | $421,896 | $405,612 | $288,648 | $148,920 | $167,136 |

Three observations before the recommendation:

1. **The cheapest single-provider answer is not eligible.** Backblaze is the cheapest total at $12,410/month and it fails two gates — residency and the regime assessment. **The price list cannot see the gates, and that is exactly why the gates are applied first.**
2. **The archive line is a rounding error and the distribution line is the bill.** On AWS the archive is $512/month and distribution is $16,438/month — 32× larger for one fifth of the data. Any conversation about §12 that starts with the archive class's $0.00099/GB rate is a conversation about 1% of the money.
3. **Oracle is the interesting hyperscaler outlier.** It is the most expensive on the lake line ($28,714) and by far the cheapest hyperscaler on distribution ($3,658) because of the 10 TB/month free-outbound allowance. That is the egress axis in one row: the same vendor is worst and best on the same estate depending on which workload you look at.

### 12.4 The egress exposure — what leaving would cost

The design document must carry the exit number next to the monthly number (§9.4, rule 4). Computing it on the full estate from each candidate:

| Provider | Full-egress cost for the 500 TiB archive | Notes |
| --- | --- | --- |
| AWS S3 | **~$29,400** | Tiered: $0.09 first 10 TB, $0.085 next 40 TB, $0.07 next 100 TB, $0.05 above 150 TB |
| Azure Blob | ~$29,379 | Effectively identical structure and total; the two hyperscalers are within 0.1% of each other on this line |
| Google Cloud Storage | **~$45,670** | The highest of the three; its first-tier rate is $0.12/GiB against AWS's $0.09/GB |
| Cloudflare R2 | $0 | Within the zero-egress condition (§4.2) |
| Wasabi | $0 | Subject to the fair-use policy (§4.2) |
| Backblaze B2 | $0 | The 3× allowance is ~1.65 PB against a 549,756 GB (decimal) estate — well inside it |

The number that should stop the conversation is the ratio: **leaving AWS costs ~$29,400, which is 58× one month's archive storage charge of $507.** The archive costs Cymbal $6,144 a year to keep and would cost $29,400 to move once. That is the lock-in mechanism of §4.5 expressed as two numbers a CFO can compare, and it is the single most useful sentence in this section: *the exit costs five years of storage.*

The distribution workload carries a second, larger exposure in the opposite direction. Serving 200 TiB/month from AWS costs $16,438/month, of which $14,126 is egress. Moving that workload from a challenger *to* a hyperscaler would add roughly $14,000/month — $168,000/year — for an identical service to the customer. The asymmetry is the point: hyperscalers are expensive to leave and expensive to arrive at, whereas the challengers are cheap at both ends, which is why a challenger is the correct *destination* for read-amplified workloads and a hyperscaler is the incumbent you are trying to leave.

### 12.5 The exit-readiness test

The only credible proof (§9.5) is a dated, measured restore. Cymbal's test is specified as follows, using the archive's actual cost structure:

- **Scope.** A 5% sample of the archive — 25,600 GiB — rehydrated from the archive class, restored to a second provider, and checksum-verified against the source manifest.
- **Illustrative cost of one test.** Glacier Deep Archive retrieval at $0.02/GB standard ≈ **$512**, plus request charges (50,000 GETs at $0.0004 per 1,000 ≈ $0.02 — negligible), plus the restoration window during which the rehydrated copy is billed at the warmer class's rate. Call it **~$520–600 per test** (§3.2 shows the steady-state monthly retrieval on this estate is $5.12; the test is a deliberate 100× burst).
- **Frequency.** Quarterly for the first year, then annually, with a different 5% sample each time so that the whole estate is exercised over a decade.
- **Success criteria.** Checksums match; the elapsed time is measured against a stated objective; every API call, policy and lock that did not port is written down as a finding; the accountable owner for the next test is named.
- **What the test does not prove.** It does not prove the *whole-estate* exit is affordable — that is the $29,400 figure in §12.4 — and it does not exercise the distribution workload's cutover. A restore test is evidence about the data plane; the policy plane needs its own rehearsal.

At roughly $2,000/year for four tests, this is the cheapest risk control in the entire design, and it is the one most often omitted.

### 12.6 The recommendation

**Split, deliberately, along the line the gates drawn in §12.2 already created.** The eligible archive providers are the hyperscalers and MinIO; the eligible and *cost-appropriate* providers for the lake and distribution include the challengers. Since the gates have already partitioned the estate, the split is not a compromise — it is the natural reading of the constraint set.

| Placement | Provider and class | Illustrative monthly cost | Why |
| --- | --- | --- | --- |
| Regulatory archive | **AWS S3 Glacier Deep Archive** | **$512** | Lowest cost among the four gate-surviving options ($512 vs $655 vs $1,059 vs $1,429), regime-assessed Object Lock, 10-year retention feasible at ~$6,144/year, and the exit exposure (~$29,400) is a known, budgeted figure rather than a surprise |
| Analytics data lake | **Wasabi Hot Cloud Storage** | **$8,782** | No regime gate applies to derived data; APAC pricing published; no egress charge for the multi-engine read pattern; no minimum-duration exposure for churning datasets would require care, but the lake is largely append-only |
| Report and media distribution | **Wasabi Hot Cloud Storage** | **$858** | The egress-heavy workload lands on the zero-egress model; the fair-use policy is the risk to monitor (this is a *published* distribution workload, and §4.2 flags exactly this profile as the one the policy is written to bound) |
| **Two-provider total** | AWS + Wasabi | **$10,152/month — $121,824/year** | Against $474,504/year single-provider AWS: an illustrative **74% reduction**, with the archive still on a regime-assessed hyperscaler |

Three honest qualifications, because a worked example that ends in a clean recommendation is a sales document:

1. **The 74% is a function of the egress profile, not of provider quality.** It comes overwhelmingly from moving read-amplified workloads off per-GB egress. If Cymbal's distribution egress were 20% of the estate per month instead of 200%, the gap narrows to roughly a factor of two on that line and the split becomes a much closer call.
2. **A residually cheaper answer exists and is rejected on a non-cost ground.** Putting the lake and distribution on Backblaze would total $9,101/month — $1,051/month less than the recommendation — but Backblaze fails the residency gate. **A 10% saving that cannot be placed inside the residency constraint is not a saving.** If the residency requirement were to change, this is the configuration to re-run.
3. **Wasabi's fair-use policy is a real risk on the distribution workload.** The row chosen is precisely the profile §4.2 identifies as the one Wasabi's free-egress policy is written to bound, and the condition is qualitative. The mitigation is either a contractual confirmation of the peak request rate and concurrency, or a fallback to Backblaze-with-a-partner-CDN. The recommendation should be written with the fallback named in it.

### 12.7 What the split costs operationally

The saving is not free, and the operational delta must be stated in the same document as the saving (this is the anti-pattern in §13.6 in its constructive form). The costs are real but they are all of the *same* kind — the S3-compatible API keeps the data plane common — and none of them is the re-architecture that a non-S3 provider would have required.

| Operational consequence | Effect |
| --- | --- |
| Two provider relationships, two contracts, two security reviews, two sets of audit evidence | Third-party risk, procurement and audit effort roughly doubles for the storage function |
| Two invoices, two cost-allocation models, two sets of volume tiers to monitor | FinOps tooling must normalise the unit difference (binary GiB vs decimal GB, §2.1) or the internal chargeback will be wrong by ~9.6% |
| Two IAM models to administer | Wasabi documents bucket and object policies plus IAM-style roles, so the policy model does port; Backblaze's absent IAM roles do not — another reason the chosen split is Wasabi rather than Backblaze |
| Two monitoring and alerting integrations, two sets of operational limits to watch (Wasabi's policy-bound API requests; AWS's per-prefix rates) | Observability must span both; the runbooks are provider-specific even when the SDK is not |
| Staff familiarity | The single largest indirect cost in most migrations (§9.1) and the least modelled. A realistic assumption — **stated as an assumption, not a sourced figure** — is one additional platform-engineering FTE, or 10–15% of the storage run cost, to administer a dual-provider estate competently. At the illustrative numbers above, that assumption shrinks the saving without erasing it. |
| The exit test must cover both providers | Two restore paths, two sets of findings, one accountable owner |

The net position, stated plainly: the split produces a large illustrative saving, converts the exit from a $29,400 surprise into a known line, removes the single-provider concentration that §11.3 would otherwise flag — and costs an operational increment that a design document must show rather than omit. The right way to present it to a decision-maker is not "we saved 74%" but "we saved an illustrative 74% while making the exit cost knowable, and here is what the second provider costs us to run."

---

## 13. The Anti-Patterns

Eight failure modes, each observed repeatedly in this topic's literature and each with a specific structural cause. The guardrail column is the actionable part.

| # | Anti-pattern (symptom) | Cause | Guardrail |
| --- | --- | --- | --- |
| 1 | **The egress estimate that omitted the CDN.** The migration business case shows egress falling by 90% and the first invoice after cutover shows the saving was half that. | The estimate compared origin egress rates and ignored that a distribution architecture re-creates the traffic on a CDN line with its own commercial terms, and that CDN egress was never priced (§10.6). | Model all three egress states — billed at origin, waived to CDN, never billed — and price the CDN contract explicitly, or mark it ⚠. A cost model that ends at the origin is unfinished. |
| 2 | **The retrieval-fee and minimum-duration surprise in an archive tier.** A workload that "never reads its data" turns out to read 40% of it in a quarter for an audit, and the retrieval bill exceeds the annual storage saving. | The storage rate was compared and the retrieval rate and minimum duration were not. §3.2 puts the crossover at roughly 32% of the estate per month against Glacier Deep Archive and ~12.5% against Google Archive. | Compute the crossover for your own estate and put it in the design: "above X% of the estate retrieved per month, this class loses to a flat-rate provider." Also check *churn*, not just reads — a minimum duration re-arms on every rewrite (§3.3). |
| 3 | **The exit decided on the storage line alone.** The design document compares $/TB, concludes the challenger is 70% cheaper, and discovers on leaving that the egress line is 58× one month's storage (§12.4). | Storage is the visible line and egress is invisible during procurement because a customer who does not leave never pays it (§4.5). | Record the full-egress figure next to the monthly storage figure at design time (§9.4, rule 4). One number, one line, and it is the number that makes the decision reversible. |
| 4 | **The "S3-compatible" service that fails on one SDK call.** Everything works in the proof of concept; production breaks on `PutObjectAcl`, `PutObjectTagging`, or a browser `POST` to a presigned URL. | Compatibility was treated as a boolean rather than a spectrum (§7.4). Backblaze's documented gaps are exactly these: no ACLs, no IAM roles, no object tagging, no website configuration, no POST-presigned uploads. | Run the question "which of my ten most important API calls are Tier 3 on the target?" before committing (§7.4). Test the *policy* plane, not just the data plane. |
| 5 | **The durability claim quoted without the vendor-survival caveat.** An 11-nines figure is used to justify holding a regulated estate with a single specialist provider. | Durability was read as a statement about the service's survival. It is a statement about hardware-induced loss under the vendor's own design, and it excludes deletion, account compromise, non-payment, region destruction and vendor failure (§8.1, §8.4). | Quote durability and the going-concern question in the same paragraph. Mitigate contractually (data-return clause, escrow, exit window) and architecturally (a second copy in a second provider) — not analytically. |
| 6 | **The enterprise-wide provider choice made from one workload's arithmetic.** A pilot on the distribution workload shows a 20× saving and the standard is written across every workload in the organisation, including the regulated archive that the chosen provider cannot legally hold. | One workload's cost profile was generalised into a platform decision. The gates in §12.2 are per-workload and per-data-class, not per-company. | Apply the eligibility gates per data class first, then price the survivors per workload. An estate with more than one access pattern almost never has one right answer, and §12 is the worked proof. |
| 7 | **The migration whose reversal cost was never modelled.** The business case prices the migration *to* the new provider and is silent on the migration *back*, so a disappointing provider becomes a negotiated hostage rather than a reversible decision. | Reversal was treated as a failure mode rather than a cost line. The five lines of §9.3 apply in both directions. | Price the return leg. If the reversal cost is not in the business case, the decision is a one-way door and should be labelled as one and escalated accordingly. |
| 8 | **The enterprise feature assumed to port.** Object Lock rules, retention policies, bucket policies, event notifications and inventory are designed once against the incumbent and assumed to exist on the target; the compliance evidence pack turns out to be provider-specific. | These are exactly the Tier 3 capabilities of §7.4 — the ones that differ in kind, not in configuration — and the IAM model is the least portable artefact in the estate (§6.2). | Enumerate the policy and lock artefacts as migration deliverables with owners, and rehearse the whole set against the target. Data migrates with `rclone`; compliance evidence does not. |

The common thread across all eight is a single error of framing: **the comparison was run against the wrong line.** Object storage's headline number is a rate; the decisions live in the structure around it — what is charged per what unit, what the allowance is a multiple of, what the minimum duration does to churn, and what leaving costs. A comparison that starts and ends with $/TB is not a cheap comparison, it is an expensive one.

---

## 14. The Claims Audit

Every figure in this guide belongs to a high-risk class — prices, allowances, durability figures and compliance claims all move, and all are easy to state wrongly. The table below records what was verified, what is flagged, and what was rejected, with the source and the date it was checked. **All prices, allowances and structures in this guide were checked on 19 September 2026.** Nothing was carried forward from another guide in this repository, from a comparison article, or from memory.

### 14.1 Verified — read on the vendor's own page

| Claim | Source (vendor's own material) | Checked |
| --- | --- | --- |
| AWS S3 Standard $0.023/GB-mo first 50 TB / $0.022 / $0.021; Standard-IA $0.0125; One Zone-IA $0.01; Glacier IR $0.004; Glacier Flexible $0.0036; Glacier Deep Archive $0.00099; Express One Zone $0.11 | AWS Price List API (file last modified 18 Sep 2026) and the S3 pricing page | 19 Sep 2026 |
| AWS egress $0.09/GB first 10 TB, $0.085, $0.07, $0.05; first 100 GB/month free; inter-region $0.02/GB; S3→CloudFront $0.00/GB | AWS S3 pricing page (worked examples) and AWS RRS data-transfer table | 19 Sep 2026 |
| S3 request pricing $0.005/1,000 PUT-class and $0.0004/1,000 GET-class; Intelligent-Tiering monitoring $0.0025 per 1,000 objects/month | AWS S3 pricing page | 19 Sep 2026 |
| S3 durability 99.999999999% and 99.99% availability for Standard; S3 SLA 99.9% / 99.0% by class with 10/25/100% credits; credits are the "sole and exclusive remedy" | S3 User Guide (Data protection, Object Lock, PCI DSS compliance) and the S3 Service Level Agreement | 19 Sep 2026 |
| S3 per-prefix limits 3,500 PUT/s and 5,500 GET/s | S3 consistency/performance documentation | 19 Sep 2026 |
| Google Cloud Storage hourly rates → ≈$0.0200/$0.0100/$0.0040/$0.0012 per GiB-month for Standard/Nearline/Coldline/Archive; operations, retrieval and egress ($0.12→$0.11→$0.08/GiB); minimum durations 30/90/365 days | Cloud Storage pricing page | 19 Sep 2026 |
| GCS durability 11 nines "regardless of storage class and location type"; Bucket Lock claiming help with "FINRA, SEC, and CFTC"; SLA tiers 99.95%/99.9%/99.0% capped at 50% of the monthly bill | Cloud Storage documentation (durability, Bucket Lock, SLA) | 19 Sep 2026 |
| Azure Hot $0.0184/$0.0177/$0.0170; Cool $0.01; Cold $0.0036; Archive $0.002; archive read $5 per 10,000; archive rehydration "typically up to 15 hours"; egress $0.087→$0.05/GB; reserved capacity 100 TB $1,545/mo (1-yr) | Azure Blob Storage pricing page and bandwidth pricing page | 19 Sep 2026 |
| Azure LRS 11 nines / ZRS 12 nines; "Redundancy protects against hardware failure, not against data-modifying operations"; immutable storage validated by Cohasset against CFTC 1.31(c)-(d), FINRA 4511, SEC 17a-4(f), with the policy required to be *locked* | Azure Data redundancy documentation and immutable-storage documentation | 19 Sep 2026 |
| OCI Object Storage $0.0255 Standard / $0.01 IA / $0.0026 Archive per GB-month; $0.0034 per 10,000 requests; first 10 TB/month free outbound in NA/EU/UK then $0.0085/GB, APAC $0.025, MEA $0.05; request-rate capacity documented as an SLO (12,000/3,000/2,000 RPS in 3-AD regions) | OCI price list and Object Storage documentation | 19 Sep 2026 |
| Cloudflare R2 $0.015 Standard / $0.01 IA per GB-month; Class A $4.50/million; Class B $0.36/million; free tier 10 GB + 1M Class A + 10M Class B, **Standard only**; usage rounded up to the next billing unit; egress free "via the Workers API, S3 API, and r2.dev" subject to footnote 1; 30-day IA minimum and $0.01/GB IA retrieval; 1,000 buckets; 5 TiB objects; REST API 1,200 requests/5 min | R2 pricing page (last updated 7 Aug 2026), R2 limits, R2 bucket locks, R2 data security | 19 Sep 2026 |
| Backblaze B2 $6.95/TB/month; no minimum file size or storage duration fees; free egress up to 3× average monthly storage then $0.01/GB; unlimited free egress to named partner CDNs/compute; Class A/B/C free; Vaults 11 nines durability | B2 pricing page and transaction-pricing page | 19 Sep 2026 |
| Backblaze S3-compatible API does **not** support ACLs, IAM roles, object tagging, website configuration or POST-presigned uploads; `GetObjectAcl` returns the bucket ACL; `x-amz-tagging` rejected | Backblaze S3-compatible API documentation | 19 Sep 2026 |
| Wasabi $7.99/TB/month; 4 KB minimum billable object size; 90-day minimum duration by default (30-day available, applying only to objects uploaded after the change); Timed Deleted Storage invoicing; free egress and API requests "subject to Wasabi's free egress and API request policies"; encryption options limited to client-side and SSE-C | Wasabi pricing page, pricing FAQ, and policy documentation | 19 Sep 2026 |
| DigitalOcean Spaces $5/month including 250 GiB across 100 buckets and 1 TiB outbound; $0.02/GiB storage and $0.01/GiB transfer beyond; Cold $0.007/GiB with 30-day minimum; **800 operations/second per bucket**; 4 KiB / 128 KiB minimum billable sizes; "large subset of the S3 RESTful API" | Spaces pricing page and Spaces pricing/limits documentation | 19 Sep 2026 |
| MinIO AIStor tiers (Free / Enterprise Lite below 400 TiB / Enterprise), SUBNET <4 hour SLA on Enterprise, and claims of FIPS 140-3, SOC 2, ISO 27001, SEC 17a-4(f) and FINRA 4511(c)-(d) compliance | MinIO pricing page | 19 Sep 2026 |
| S3 moved to strong read-after-write consistency for all GET, PUT, LIST and metadata operations, announced in the AWS News Blog post "Amazon S3 Update – Strong Read-After-Write Consistency" | AWS News Blog post (fetched) | 19 Sep 2026 — see the flag below on the date |

### 14.2 Flagged — verified as stated but with a qualification the reader must carry

| Claim | Flag |
| --- | --- |
| The December 2020 date for S3's strong-consistency change | The blog post as fetched states the change was effective "immediately" but does not print a publication date in the extracted text. The **December 2020** date is therefore flagged here rather than asserted; the *content* of the change (strong read-after-write for all operations, all regions, existing objects, at no charge) is verified from the post itself. |
| Google Cloud Storage monthly rates | The published figures are **hourly** per-GiB rates. The monthly figures in §5.2 are **my conversion** at 730 hours, not a Google-published monthly rate. The conversion is arithmetic on Google's numbers; the monthly rate is mine. |
| Google's Archive storage rate ($0.0012/GiB-month) | Same conversion caveat; also note that Google's Archive *read operations* are charged at the Class B Archive rate of $0.05/1,000, the same as its write class, which is a structure that the storage line hides. |
| Wasabi free egress and free API requests | Both are **"subject to"** a fair-use policy whose condition is qualitative — "a reasonable rate", "does not impose an unreasonable burden" — and whose remedy is the right to "limit or suspend your service". It is verified as the vendor states it, and it is not a contractual rate. |
| Backblaze free egress | A **ratio** to stored data (3×), computed on byte-hours, not an unlimited grant. It also *shrinks* if the stored estate shrinks, which matters for an exit designed as "copy out, then delete". |
| Cloudflare R2 zero egress | Free on R2's own egress paths only; "if you connect other metered services to an R2 bucket, you may be charged by those services". The free tier excludes Infrequent Access; billing rounds up to the next whole unit. |
| Azure Archive immutability | Microsoft's own documentation requires the time-based retention policy to be **locked** to be in a compliant immutable state — the distinction between "immutability protection" and "compliant immutable state" is the vendor's, and it is easy to miss. |
| Durability figures generally | All are **design targets** ("designed to provide", "designed for at least"), not measured outcomes, and all exclude deletion, overwrite, account compromise, non-payment, region destruction and vendor failure (§8.1). |
| S3→CloudFront at $0.00/GB and other CDN waivers | Verified as stated from the vendor's data-transfer table. What is **not** verified is the CDN's own egress price, which is where the cost lands next (§15). |
| Oracle's 10 TB/month free outbound | Verified from Oracle's pricing material as an origin-geography-group allowance; the APAC and MEA rates beyond it differ, so the "free egress" shorthand is only true below the allowance and in the named origin regions. |

### 14.3 Rejected — claims encountered and not carried into this guide

| Rejected claim | Why it was rejected |
| --- | --- |
| Any single "cheapest provider" ranking, including one this guide's own arithmetic could be read to produce | The models in §3 and §12 change rank twice with the egress ratio alone (§3.9) and are gated by non-commercial constraints in §12.2. A ranking would be a false precision. This guide deliberately crowns no winner. |
| A specific durability figure for Oracle OCI, Cloudflare R2 or DigitalOcean Spaces | None was found on the pages read. Repeating a third-party's figure would have violated the source rule; §15 records the gap. |
| Any compliance certification implied rather than claimed by the vendor | R2's documentation points readers to the Cloudflare Trust Hub without asserting specific frameworks for R2; Wasabi's trust centre claims "applicable SEC standards" in prose without naming a regime or an assessor. Both are reported exactly as the vendor states them, and no certification is asserted on any vendor's behalf. |
| "S3-compatible" treated as a boolean across providers | Rejected as a framing: the vendors' own documentation shows it is a spectrum with documented, specific gaps (§7.2, §7.4). |
| CDN egress rates for any provider | Not fetched for this guide. The $0.00/GB origin-to-CDN waivers are verified; the CDN's own egress rates are not, and no figure is offered. |
| Any performance benchmark presented as fact | No independent benchmarks were run or collected. Vendor product claims (e.g. Express One Zone latency positioning, gRPC throughput for analytics) are repeated only as *the vendor's own claims*, labelled as such in §10.3. |
| Prices carried forward from other guides in this repository | Explicitly prohibited. Where this guide references another guide (S3 internals, the on-premises field, API-compatibility tiering), it cites the guide and does not import its figures. |

---

## 15. What Could Not Be Verified

Recorded honestly, because a gap that is stated can be closed later and a gap that is hidden becomes an error in a design document.

**Pricing and allowances that could not be extracted.**
- **Wasabi's monthly minimum charge.** The invoice glossary documents a "Minimum Active Storage" line and states that a difference charge is applied when Timed Active Storage falls below it, but the value of the minimum was not extracted from the pages read. Any small Wasabi estate should establish it directly.
- **Oracle's Always Free object-storage allowances.** Outbound transfer is documented as up to 10 TB/month; the Always Free object-storage capacity figure was not verified.
- **Cloudflare R2's storage-billing unit in the presence of IA.** R2 states usage rounds up to the next billing unit and describes storage as the average of peak daily usage over 30 days; the interaction between those two rules on a mixed Standard/IA bucket was not worked through.
- **Reserved-capacity and private-pricing rates beyond the published blocks.** Azure's published 100 TB and 1 PB reserved-capacity blocks were read; hyperscaler private pricing is not published and is not modelled.

**Service levels that were referenced but not read.**
- **Azure Blob's storage SLA** — published as a separate support-legal document; the commitment and credit table were not read.
- **Oracle OCI's object-storage SLA** — not read. Note that Oracle's own documentation classifies request-rate capacity as an SLO rather than an SLA, which is a distinction a design should not blur.
- **Cloudflare R2's SLA** — not read.
- **Backblaze B2's SLA** — no SLA document was located on the pages checked.
- **Wasabi's SLA** — the SLA page as fetched carried navigation and a link to a February 2024 reference PDF; the operative terms were not read.
- **DigitalOcean Spaces' Standard Storage SLA** — Cold Storage is documented at 99.5%; the Standard Storage commitment was not read.

**Durability figures that were not published on the pages read.**
- **Oracle OCI** — no numeric durability figure was extracted; the overview describes redundant storage with checksum monitoring and repair. This guide does **not** assert a figure for OCI.
- **Cloudflare R2** — no numeric durability figure on the pricing, limits, consistency, bucket-lock or data-security pages read.
- **DigitalOcean Spaces** — runs on Ceph with redundancy and fault tolerance; no numeric figure published. No figure is asserted here.

**Compliance claims deliberately not made.**
- **R2's specific certifications.** Its data-security documentation directs readers to the Cloudflare Trust Hub. No framework is asserted for R2 in this guide, on Cloudflare's behalf or otherwise.
- **Wasabi's regulatory-regime position.** Its trust centre claims "applicable SEC standards" in prose. No assessor, rule number or assessment is named, and §11.2 records this exactly as written.
- **Oracle's and DigitalOcean's regime positions.** Nothing was asserted in the pages read; the matrix shows ✖ rather than a guess.

**Data-residency specifics.**
- **R2's jurisdictional data-location controls.** Their existence was noted; their enumeration, coverage and pricing were not verified. This is why §12.2 holds R2 "pending evidence" rather than accepting or eliminating it.
- **Wasabi's specific named regions.** The matrix records named regions and an APAC price; the specific region list was not enumerated, so the §12 residency gate must be confirmed contractually rather than inferred from this guide.
- **Backblaze's region list.** Recorded as US and EU from the pages read; a full enumeration was not performed.

**Performance figures.**
- **No independent benchmarks were run or collected.** Every performance statement in §10 is either a vendor-documented limit (verifiable) or a vendor's own product claim (labelled as such). A reader needing a benchmark for a specific access pattern should generate it against their own workload.
- **CDN egress rates.** Outside the scope of what was verified. The origin-to-CDN waivers are verified; the CDN leg is not priced here.

**Method limitations.**
- **Search coverage.** The research relied primarily on direct extraction of vendor pricing, limits, SLA, durability and compliance pages, with search used to locate them. Where a search returned no useful result, that is recorded here as a limitation of the research method and is **not** treated as evidence that a feature or document does not exist. In particular, the *absence* of a durability figure or an SLA in this guide means only that one was not found on the pages read, not that the vendor does not publish one.
- **Point-in-time prices.** Every rate in this guide was checked on 19 September 2026 and every one is a moving target. The *structure* (what is charged per what unit, what the allowance is a multiple of, what minimum durations apply) is the durable part of the analysis and is more likely to survive a price revision than any rate here; the rates themselves must be re-checked before they are used in a business case.

---

## 16. Glossary, Cross-References and Closing Summary

### 16.1 Glossary

| Term | Meaning in this guide |
| --- | --- |
| **Bucket** | The top-level container for objects; the unit of naming, policy, region and, on several providers, the unit of request-rate limits. |
| **Object** | The stored entity: bytes, a key, and HTTP-visible metadata. Immutable in the sense that a change is a new version or an overwrite, never an in-place edit. |
| **Key** | The object's name within a bucket. Keys with `/` in them are *prefixes*, not directories, on every provider except those with a hierarchical namespace. |
| **Prefix** | The key-string component used for partitioning and, on AWS, the unit against which per-second request limits are documented. |
| **Storage class / tier** | A named combination of storage rate, availability, minimum duration and retrieval fee. The class, not the provider, is usually the biggest single cost lever. |
| **Durability** | A modelled annual probability of hardware-induced data loss under the provider's redundancy design. Excludes deletion, overwrite, account compromise, non-payment, region destruction and vendor failure (§8.1). |
| **Availability** | The fraction of time the service is usable, expressed as an SLA with service credits. A separate property from durability and from latency. |
| **Request class** | The billing category an operation falls into — AWS's PUT/GET split, Google's Class A/B, Azure's per-10,000 write/read/list split, R2's Class A/B. The mapping is not uniform across providers (§2.3). |
| **Egress** | Data transferred out of the provider, priced per GB on the hyperscalers and absent, allowance-bounded or policy-bounded on the challengers (§4). |
| **Retrieval fee** | A per-GB charge for reading data from a cold class. Independent of the storage rate and, at Azure Archive, priced at $5 per 10,000 reads against $0.004 on Hot (§2.5). |
| **Minimum storage duration** | A period for which an object is billed even if deleted earlier — 30/90/180 days on AWS, up to 365 on Google Archive, 90 on Wasabi by default. A penalty mechanism, not a rule (§2.5). |
| **Minimum billable object size** | A floor below which an object is billed as if larger: 4 KB at Wasabi and DigitalOcean Standard, 128 KB for several AWS classes. Turns cost into a function of object count (§3.7, §10.3). |
| **Free allowance** | A grant that reduces a bill: an absolute monthly volume (AWS/Azure 100 GB, Oracle 10 TB), a subscription allowance (DigitalOcean 1 TiB), a multiple of stored data (Backblaze 3×) or an unbounded but service-conditional grant (R2, Wasabi) (§4.3). |
| **S3-compatible API** | An implementation of the S3 REST API surface. A spectrum, not a boolean: Tier 1 functional, Tier 2 configuration-different, Tier 3 capability-absent (§7.4). |
| **Multipart upload** | Chunked upload of a single object. Universally supported, differently specified — part counts, sizes and concurrency limits differ per provider (§10.5). |
| **Object Lock / WORM** | Write-once-read-many retention: Object Lock with retain-until-date and legal hold (AWS, Backblaze, Wasabi), Bucket Lock and Object Retention Lock (Google), time-based retention and legal holds (Azure), bucket locks (R2), retention rules (Oracle) (§11.2). |
| **Legal hold** | An indefinite, non-duration-based hold placed on an object or version, independent of a retention period. |
| **Versioning** | Retention of prior object versions on overwrite or delete. A protection and a cost: versions are billable storage and they enlarge the estate to be migrated. |
| **Lifecycle rule** | Policy-driven transition between classes or deletion by age or prefix. The mechanism that makes small-object and orphaned-multipart waste visible — or invisible, if unwritten. |
| **Replication** | Copying objects to another bucket, region or provider (AWS CRR/SRR and MRAP, Google's default and turbo replication, Azure object replication, OCI replication policies, Wasabi object replication). Billed as data transfer, and a residency event (§11.1). |
| **Strong read-after-write consistency** | A GET immediately after a successful PUT returns the new data, and LIST reflects it. S3 moved to this for all operations in December 2020 (date flagged in §14.2); Google, Azure and Oracle all document strong consistency today (§6.2). |
| **Service credit** | The remedy a provider's SLA offers for unavailability: a discount on a future bill, claimed by the customer within a deadline, capped — at Google, at 50% of the monthly bill. Not compensation for an incident (§8.2). |
| **SLO** | An internal objective rather than a contractual commitment. Oracle documents request-rate capacity as an SLO, which is precisely the distinction between "we target this" and "we owe you this". |
| **Redundancy class** | Where copies live: LRS (one datacentre), ZRS (zones), GRS/RA-GRS and GZRS/RA-GZRS (regions), and the single-AZ classes such as S3 One Zone-IA (§8.3). |
| **Key custody** | Who holds the encryption keys: provider-managed (SSE-S3, Google-managed), customer-managed in the provider's KMS (SSE-KMS, CMEK, Azure CMK), customer-supplied (SSE-C), or client-side against keys the customer holds entirely. The row that narrows the challenger field (§11.4). |
| **Egress waiver** | A zero-rated transfer path: S3→CloudFront at $0.00/GB, Google's CDN waiver, Backblaze's named partner CDNs, DigitalOcean's enumerated Spaces→Droplet paths. A cheaper dependency, not an absence of one (§4.5, §10.6). |
| **Cache fill** | The origin→CDN leg of a distribution request. Free from S3 to CloudFront; Google waives Cloud Storage transfer for its CDNs but states "cache fill charges may apply" (§10.6). |
| **Going concern** | Whether the provider will still exist. Covered by no durability figure and no SLA, and the reason §8.4 and §11.3 exist. |
| **Exit cost** | The fully-loaded cost of moving the estate elsewhere: transfer, requests at both ends, rehydration and temporary duplicate storage, tooling and dual-running, and the reversal leg (§9.3). |
| **Exit test** | A dated, measured restore or re-host into a different provider with checksum reconciliation and a documented failure list. The only credible evidence of exit-readiness (§9.5). |

### 16.2 Cross-references

This guide is deliberately bounded. The following guides own their subjects and are cited rather than restated:

- **[`s3_architecture_guide.md`](s3_architecture_guide.md)** — owns S3's data model, internals, durability and consistency mechanics, its storage classes in depth, security, performance and API. This guide uses S3's *prices and structures* as comparison inputs and cross-refers for everything else.
- **[`cloud_object_storage_lakehouse_guide.md`](cloud_object_storage_lakehouse_guide.md)** — owns the lakehouse architectures on object storage, the on-premises and private S3-compatible field (MinIO, Dell ObjectScale, IBM COS, NetApp StorageGrid, Ceph RGW, Pure FlashBlade, VAST) in its §5, and the API-compatibility tiering in its §13. This guide extends the provider line with the commercial challengers and does not repeat the hyperscaler storage-class tables.
- **[`dell_objectscale_guide.md`](dell_objectscale_guide.md)**, **[`storagegrid_guide.md`](storagegrid_guide.md)** and **[`cephfs_alternatives_guide.md`](cephfs_alternatives_guide.md)** — own their respective products and the on-premises alternatives field.
- **[`cloud_providers_guide.md`](cloud_providers_guide.md)** — owns the broader multi-cloud platform comparison; this guide stays on object storage as a service.
- **[`billion_user_system_arch.md`](architecture/billion_user_system_arch.md)** — owns large-scale architecture patterns; its object-storage price line is a single-point reference, not a comparison.

The boundary in one sentence: **`s3_architecture_guide.md` owns S3; `cloud_object_storage_lakehouse_guide.md` owns lakehouses and the private cloud field; this guide owns the cost structure, the egress axis, the cross-provider feature matrix, the exit analysis and the selection framework.**

### 16.3 Closing summary

Object storage is a commodity market with a non-commodity bill. Every provider in this comparison lands within a factor of about three on the storage line, and every one of them has converged on eleven nines of durability, strong consistency and a WORM capability of some description. What has *not* converged is the cost structure: whether requests are free or charged, whether the egress allowance is an absolute volume or a multiple of the estate, whether the unit is a gibibyte or a decimal gigabyte, whether an archive class carries a retrieval fee and a minimum duration, and whether the object you store is billed at the size you wrote. Those differences decide the money, and they decide it far more than the rate per terabyte that every comparison starts from.

The egress axis is the sharpest expression of this, because it is the one cost line that scales with the action that ends the relationship and the one line that is invisible during procurement and enormous during exit. The three challengers examined here remove that line — Cloudflare R2 on its own paths, Wasabi under a fair-use policy, Backblaze as an allowance ratio with a partner-CDN channel — and each of those removals carries a condition that a design must state rather than assume. The hyperscalers, meanwhile, offer the compliance depth, key custody and regime assessments that a regulated estate needs, at the cost of an exit that can run to five years of storage charges. Neither posture is wrong; they are correct for different workloads, which is why the estate-level answer is usually a split, and why no single price list can tell you which side of the line your data belongs on.

What this guide offers instead of a winner is a method: decompose the bill into storage, requests, egress and retrieval-plus-minimum-duration; establish the structure of each component before debating the rate; gate the candidate set on residency, immutability-with-assessment, key custody and operational ceiling before applying any price; model the workload archetype rather than the average; and record the full-egress number beside the monthly number in the same design document, because that is the number that decides whether the choice you are making is reversible. Seal it with a dated restore test, because an architecture diagram showing a second provider proves nothing that a checksum has not confirmed.

The provider with the lowest price per gigabyte is the least interesting part of the decision. The decision is the cost of the exit.

---

> **Author:** Jack Liu Shurui — Solution Architect
> **Last Updated:** September 2026
> **Prices, allowances, durability figures and compliance claims checked:** 19 September 2026
> **Repository:** [github.com/jackliusr/research](https://github.com/jackliusr/research)
> **License:** MIT — free for internal reference and education

