# AWS Solutions Architect Professional (SAP-C02) Certification Guide

**Author:** Jack Liu Shurui | **Certification:** AWS Certified Solutions Architect – Professional (SAP-C02) | **Updated:** July 2026

*Part of the Enterprise Architect skill gap analysis — AWS/Azure/GCP architect certifications identified as a medium-priority gap.*

---

## 1. Exam Overview

| Item | Detail |
|------|--------|
| **Exam Code** | SAP-C02 |
| **Duration** | 180 minutes |
| **Questions** | 75 (65 scored + 10 unscored pretest) |
| **Format** | Multiple choice, multiple response |
| **Passing Score** | ~750/1000 (scaled) |
| **Fee** | $300 USD |
| **Validity** | 3 years (recertification required) |
| **Prerequisites** | 2+ years hands-on AWS experience + Associate-level certification recommended |
| **Delivery** | Pearson VUE or PSI — online proctored or testing center |

### 1.1 Candidate Profile
Two+ years designing/distributed systems on AWS. Proficient in: HA/cost-efficient/scalable architecture, multi-account environments, complex migrations, AWS Well-Architected Framework. Questions include case studies (scenario + 2–5 questions).

### 1.2 Certification Path
`Cloud Practitioner → Solutions Architect Associate → Solutions Architect Professional → Specialty Exams`

---

## 2. Exam Domains and Weighting

| Domain | Title | Weight |
|--------|-------|--------|
| **1** | Design for Organizational Complexity | **26%** |
| **2** | Design for New Solutions | **28%** |
| **3** | Migration Planning | **15%** |
| **4** | Cost Control | **10%** |
| **5** | Continuous Improvement for Existing Solutions | **21%** |

> **Tip:** Domain 2 is heaviest but Domains 1+5 = 47% of the exam. Study all five.

---

## 3. Domain 1: Design for Organizational Complexity (26%)

### 3.1 AWS Organizations
Foundation of multi-account architecture. **Key concepts:** Management account (creator/payer), member accounts, OUs (organizational units), SCPs (Service Control Policies), consolidated billing.
- **SCPs:** Define **maximum permissions** — do NOT grant, only restrict. Inherited downward. Evaluation: Deny always wins, then intersection of SCPs + IAM determines effective permissions.
- **SCP strategies:** Allow list (deny all, allow specific) vs. Deny list (allow all, deny specific). Common SCPs: deny leaving org, deny disabling CloudTrail, restrict EC2 instance types, enforce tags, restrict regions.
- **Consolidated billing:** Single payer, aggregated volume discounts, no cost.
- **Feature sets:** Consolidated Billing only vs. All Features (includes SCPs, account creation APIs).

### 3.2 AWS Control Tower
Managed landing zone service. Automates multi-account baseline.
- **Landing zone:** Creates Security OU (Audit + Log Archive accounts), Sandbox OU, core accounts.
- **Guardrails:** Preventive (SCPs) and detective (Config rules). Do NOT manually modify CT-managed SCPs.
- **Account Factory:** Automated account provisioning via Service Catalog. Customizations via CfCT.
- **Control Tower vs. Organizations:** CT = opinionated, automated, costs extra. Organizations = flexible, manual, free base.

### 3.3 IAM Identity Center (formerly AWS SSO)
Centralized SSO across AWS accounts and business apps.
- **Permission sets:** Collections of IAM policies assigned to users/groups per account.
- **Federation:** SAML 2.0 (Azure AD, Okta) and OIDC. **SCIM** for automatic user/group provisioning.
- **AD integration:** AWS Managed Microsoft AD (full AD in cloud), AD Connector (proxy to on-premises), Simple AD (Samba-based, <500 users).
- **ABAC:** Attribute-based access control using user attributes (cost center, job code).
- Replaces per-account IAM users — manage centrally, assign via permission sets.

### 3.4 IAM Best Practices (Professional Level)
- **Cross-account access:** IAM roles (most common), resource-based policies (S3, SQS, SNS, Lambda, KMS), permission boundaries (max permissions for delegated admins).
- **IAM evaluation logic:** Explicit Deny wins. SCP denies if not allowed. Permission boundary caps max. Identity OR resource-based policy must allow (unless NotPrincipal creates explicit deny).
- **Best practices:** Roles not users, rotate keys, use `aws:SourceArn`/`aws:SourceAccount` for confused deputy protection, `aws:RequestedRegion` for regional restrictions, IAM Access Analyzer.

### 3.5 AWS Config
Resource inventory, configuration history, compliance evaluation.
- **Rules:** Managed (pre-built) or custom (Lambda). **Conformance packs:** YAML bundles of rules + remediation.
- **Aggregators:** Multi-account/region compliance view. **Auto-remediation:** Config rule → EventBridge → SSM Automation.
- **Recording:** On resource change or scheduled. Config = **resource state**. CloudTrail = **API calls**.

### 3.6 AWS CloudTrail
Records API activity. Enabled by default for management events only.
- **Management events:** Control plane (CreateVpc, RunInstances). Free, 90-day retention.
- **Data events:** Data plane (S3 GetObject, Lambda Invoke). Paid, high volume.
- **Organization trail:** Single trail logs all accounts in org. **Log aggregation:** Central S3 bucket.
- **Insights:** Detects anomalous API activity. **CloudTrail Lake:** SQL-queryable log data store.
- **CloudTrail vs. Config:** API calls vs. resource states. Complementary.

### 3.7 AWS Service Catalog
Approved IT service catalog with governance.
- **Products:** CloudFormation templates. **Portfolios:** Product collections with IAM permissions.
- **Constraints:** Limit what users can configure. **Launch constraints:** Define IAM roles for provisioning.
- Use case: Let devs launch approved EC2 without giving EC2 full access. Integrates with Control Tower Account Factory.

### 3.8 AWS License Manager
Manage software licenses (Microsoft, Oracle, SAP) across AWS and on-premises.
- License configurations, discovery, constraints, BYOL support, Dedicated Host management.

### 3.9 Tag Strategies
Standard tags: `CostCenter`, `Environment`, `Owner`, `Project`, `DataClassification`, `Compliance`.
- Enforcement: SCPs (`aws:RequestTag`), Config rules, Budgets, Cost Categories.

### 3.10 Multi-Account Structure
**Recommended structure:**
- **Management account:** Billing, Organizations, IAM Identity Center. Minimize resources.
- **Security OU:** Log Archive account (immutable S3 for logs) + Security Tooling account (GuardDuty, Security Hub, Config aggregator).
- **Infrastructure OU:** Shared Services account (Directory Service, VPC endpoints) + Network Account (TGW hub, Direct Connect, VPN).
- **Workload OU:** Prod/Non-prod sub-OUs per workload.
- **Sandbox OU:** Individual accounts for experimentation.

### 3.11 Network Architecture
- **Transit Gateway:** Hub-and-spoke. Inter-region peering, multicast support, network segmentation via separate route tables.
- **Centralized egress:** All VPC traffic → TGW → egress VPC (NAT, IGW, Network Firewall) → internet. No IGW in workload VPCs.
- **Centralized ingress:** CloudFront+WAF → ALB in ingress VPC → TGW → workload VPC.

### 3.12 Best Practices
- **CloudFormation StackSets:** Deploy across accounts/regions. Service-managed for org-wide.
- **Delegated administrators:** Designate member accounts as service admins (GuardDuty, Config, CloudTrail, Macie, SSM).
- **Break-glass access:** Emergency process for management account, MFA, audit trail.
- **Root account:** Lock away with MFA. Never use for daily operations.

---

## 4. Domain 2: Design for New Solutions (28%)

### 4.1 AWS Global Infrastructure
AWS global infrastructure consists of **Regions** (geographically isolated areas, min 3 AZs each), **Availability Zones** (isolated data centers within a region connected by low-latency fiber), **Local Zones** (single AZs placed close to metro populations for latency-sensitive workloads like gaming and live video), **Wavelength Zones** (AWS infrastructure embedded at 5G telecom edge locations for ultra-low-latency mobile apps), **Outposts** (AWS hardware and services deployed on-premises for hybrid workloads requiring data residency or low-latency on-premises access), and **Points of Presence** (600+ edge locations for CloudFront and Global Accelerator).

**Key exam distinction:** Local Zones = metro proximity (1 AZ, limited services). Wavelength = 5G edge (telco infrastructure). Outposts = on-premises AWS (same APIs, control plane in AWS Region).

### 4.2 Multi-Region Architectures

Four primary disaster recovery patterns, ordered by increasing resilience and cost:

| Pattern | RTO | RPO | Cost | Description |
|---------|-----|-----|------|-------------|
| **Backup & Restore** | Hours | Hours | Lowest | Data backed up to secondary region; full rebuild on failover |
| **Pilot Light** | Minutes | Hours | Low | Core services (DB, config) running; scale up application on failover |
| **Warm Standby** | Minutes | Minutes | Medium | Scaled-down full stack running in standby region; scale up at failover |
| **Multi-Site (Active-Active)** | Real-time | Seconds | High | Both regions serve live traffic simultaneously |

**Active-Passive** uses Route 53 failover routing with health checks against the primary region. When the primary fails, DNS records update to point to the standby region. Useful for workloads that can tolerate a few minutes of downtime.

**Active-Active** requires DynamoDB Global Tables or Aurora Global Database for data replication, Route 53 latency-based or geolocation routing to distribute traffic, CI/CD pipelines deploying to both regions, and Global Accelerator for fast regional failover.

### 4.3 Amazon Route 53

Amazon Route 53 is AWS's DNS service. **Routing policies** determine how traffic is directed:

| Policy | Behavior | Best For |
|--------|----------|----------|
| **Simple** | Returns a single record (random if multiple values) | Single-server, simple mappings |
| **Weighted** | Distributes traffic by assigned weight (0–255) | Canary deployments, A/B testing |
| **Latency** | Routes to region with lowest latency for requester | Global performance optimization |
| **Geolocation** | Routes based on requester's geographic location | Localization, content restrictions |
| **Geoproximity** | Routes based on location with bias drift | Regional traffic steering |
| **Failover** | Primary/secondary with health check association | Active-passive disaster recovery |
| **Multi-Value** | Returns up to 8 healthy records randomly | Client-side load balancing at DNS level |

**Health checks** can monitor endpoints directly (HTTP/HTTPS/TCP), track CloudWatch alarm states, or be calculated checks that combine multiple health checks (OR/AND logic). Health checks associate with DNS failover records and SNS notifications.

**Private hosted zones** provide DNS resolution within VPCs, supporting split-view DNS (same name resolves differently inside vs. outside VPC). **Route 53 Resolver** provides hybrid DNS with inbound endpoints (on-premises queries forwarded to AWS), outbound endpoints (AWS queries forwarded to on-premises DNS), resolver rules (conditional forwarding), and DNS Firewall (block/allow domain lists).

### 4.4 Amazon CloudFront

CloudFront is AWS's content delivery network (CDN) with 600+ edge locations.

**Core concepts:**
- **Origins:** S3 buckets, ALBs, EC2 instances, custom HTTP servers, Lambda function URLs, MediaPackage
- **Behaviors:** Path-pattern rules controlling cache behavior, origin selection, TTL, protocol policies, and allowed HTTP methods. Multiple behaviors per distribution enable different handling for /api/* vs. /static/*
- **Origin Shield:** An additional centralized caching layer that reduces origin load by collapsing requests from multiple edge locations. Reduces origin requests and lowers cost.
- **Lambda@Edge:** JavaScript or Python functions executed at edge locations across four triggers — viewer request, origin request, origin response, viewer response. Max 5 seconds for viewer triggers, 30 seconds for origin triggers. Node.js and Python runtimes only.
- **CloudFront Functions:** Lightweight JavaScript functions for viewer request/response only. Sub-millisecond execution, millions of requests per second, significantly lower cost than Lambda@Edge. Ideal for URL rewrites, header manipulation, and simple A/B testing.
- **Real-time logs:** Stream to Kinesis Data Streams for near-real-time log analysis.
- **WAF integration:** Associate a WAF web ACL directly with the distribution for edge-based web application protection.
- **Signed URLs vs. Signed Cookies:** Signed URLs grant per-file access. Signed cookies grant access to groups of files (for streaming or batch access).
- **Field-level encryption:** Encrypt specific data fields (e.g., credit card numbers, PII) at the edge before forwarding to origin. Origin never sees raw sensitive data.

**CloudFront vs. Global Accelerator (common exam trap):**
| Feature | CloudFront | Global Accelerator |
|---------|------------|-------------------|
| Protocol | HTTP/HTTPS only | TCP/UDP (any port) |
| Caching | Yes (static + dynamic content) | No (network path optimization only) |
| Lambda@Edge | Yes | No |
| WAF integration | Native | Via ALB integration |
| Use case | Content delivery, API acceleration | Gaming, VoIP, non-HTTP protocols |

### 4.5 AWS WAF

AWS WAF protects web applications from common web exploits. **Key features:**
- **Web ACLs:** Regional (for ALB, API Gateway, AppSync) or CloudFront (global)
- **Rate-based rules:** Block IP addresses after exceeding a request rate threshold (e.g., 2000 requests in 5 minutes)
- **Rule groups:** Reusable collections. AWS Managed Rule Groups include the core rule set (OWASP top 10), SQLi, XSS, known bad inputs, Linux/Microsoft-specific threats
- **Bot Control:** Managed rule group to categorize bots as verified (Googlebot, Bingbot), monitoring (suspected), or untrusted (known bad actors)
- **CAPTCHA:** Challenge-based rules that present a CAPTCHA to suspicious clients
- **IP sets / Regex pattern sets:** Reusable matching conditions for IP allow/deny lists and custom pattern matching

**Exam tip:** WAF is a regional service. For CloudFront, you associate one regional WAF ACL. For ALB/API Gateway/AppSync in multiple regions, you must deploy WAF in each region.

### 4.6 AWS Shield Advanced

Shield Advanced provides enhanced DDoS protection for EC2, ELB, CloudFront, Global Accelerator, and Route 53. **Capabilities:**
- Layer 3/4/7 DDoS protection with automated mitigations
- **Cost protection:** AWS waives DDoS-related scaling charges (EC2 scaling, ELB traffic spikes)
- **Advanced reporting:** Real-time metrics, attack vector diagnostics, top contributors
- **DDoS Response Team (DRT):** 24/7 access to AWS's DDoS experts
- **Proactive engagement:** DRT contacts you during events and configures mitigations on your behalf
- **Health-based detection:** Uses Route 53 health checks to distinguish real user traffic from DDoS traffic

### 4.7 AWS Firewall Manager

Firewall Manager centralizes firewall rule management across accounts in AWS Organizations. Manages:
- **WAF rules** across all accounts (common rule sets)
- **Shield Advanced** protections for all accounts
- **Network Firewall** policies for centralized VPC inspection
- **Security Group** baselines — defines common rules and remediates drift in member accounts
- Requires a **delegated administrator** account. Automatically applies policies to new accounts as they join the organization.

### 4.8 Network Protection (Layered Model)

| Layer | Service | Characteristics |
|-------|---------|----------------|
| **Edge** | CloudFront + WAF + Shield | DDoS mitigation + web app filtering at edge |
| **VPC perimeter** | Network ACLs (stateless) | Subnet-level IP/port allow/deny, ordered rule evaluation |
| **Instance level** | Security Groups (stateful) | Instance-level allow-only rules, automatically allow return traffic |
| **Deep inspection** | AWS Network Firewall | Managed stateful inspection, IPS/IDS, domain filtering |
| **Traffic analysis** | Traffic Mirroring, Reachability Analyzer | Copy traffic to inspection appliances, test network paths |

**Security Groups vs. NACLs (frequent exam topic):**
- SGs are **stateful** — return traffic is automatically allowed regardless of rules.
- NACLs are **stateless** — return traffic must be explicitly allowed by rules.
- SGs support allow rules only. NACLs support both allow and deny rules.
- NACL rules are evaluated **in order by rule number** (lowest first). SGs evaluate all rules.
- SGs attach to ENIs/instances. NACLs attach to subnets.

**AWS Network Firewall** is a managed firewall service for VPC. Supports stateful inspection, intrusion prevention, domain filtering, and threat intelligence feeds. Typically deployed in a central ingress/egress VPC with Transit Gateway connectivity for multi-VPC inspection.

### 4.9 High Availability Design

**Auto Scaling strategies:**
- **Target tracking scaling:** Keep a metric at target value (e.g., CPU at 50%)
- **Step scaling:** Respond to metric severity (e.g., add 2 instances if CPU > 80%)
- **Scheduled scaling:** Predictable traffic patterns (e.g., scale up at 8 AM)
- **Predictive scaling:** ML-based forecasting of future traffic, proactively scales
- **Health checks:** ELB health checks, EC2 status checks, custom health checks

**Multi-AZ design:** Deploy across at least 2 AZs for production. ALB + ASG spanning multiple AZs provides automatic traffic distribution. RDS Multi-AZ provides synchronous standby in another AZ. ElastiCache Multi-AZ with automatic failover. DynamoDB natively replicates across 3 AZs.

**Graceful degradation patterns:** SQS queue as buffer when processing slows down, circuit breaker pattern (fail fast, retry later via exponential backoff), degrade features (switch to read-only mode or serve stale cache) when backend unavailable.

**Circuit breaker:** Implemented at application layer (Resilience4j). States: Closed (normal) → Open (fail fast) → Half-Open (test) → Closed (recovered). API Gateway can serve 503 responses when circuit is open.

### 4.10 Hybrid Architectures

**AWS Direct Connect:**
- Dedicated private network connection from on-premises to AWS. Available at 50 Mbps to 100 Gbps.
- **Virtual interfaces (VIFs):** Private VIF (connect to VPCs, single or via TGW), Public VIF (connect to public services like S3, DynamoDB), Transit VIF (connect TGW via Direct Connect Gateway)
- **LAG (Link Aggregation):** Combine multiple connections for higher throughput and redundancy
- **MACsec:** IEEE 802.1AE encryption on dedicated connections for encryption-in-transit
- **SiteLink:** Send data between Direct Connect locations using the AWS global backbone (no public internet traversal)
- **Direct Connect Gateway:** Single interconnect with VPCs in any AWS region globally

**VPN options:**
- **Site-to-Site VPN:** IPSec tunnels between on-premises router and AWS VPN endpoints. Supports dynamic routing (BGP) or static routes. Can terminate on Virtual Private Gateway or Transit Gateway. **Accelerated VPN** uses Global Accelerator for better performance on long-distance connections.
- **Client VPN:** OpenVPN-based solution for remote users. Supports certificate-based, SAML, Active Directory, and mutual authentication.

**Transit Gateway deep dive:**
- Hub-and-spoke architecture connecting VPCs, VPN, and Direct Connect
- **Inter-region peering:** Connect TGWs across regions (data transfer charges apply)
- **Multicast:** Native multicast support via TGW multicast domains (financial services, media streaming)
- **Network segmentation:** Separate route tables for different VPC attachments (isolate prod from dev)
- **Route propagation:** Dynamic route learning via BGP from VPN and Direct Connect
- **Bandwidth:** Up to 50 Gbps per VPC attachment

### 4.11 Edge Services

**AWS Global Accelerator:**
- Provides two Anycast IP addresses — traffic enters the nearest edge location, traverses the AWS global network, and reaches the optimal regional endpoint
- Supports TCP and UDP on any port
- **No caching** — only network path optimization
- **Traffic dial:** Control percentage of traffic to each endpoint (0–100%)
- **Endpoint weights:** Weighted distribution across multiple endpoints
- **Fleet management:** Automatically routes away from unhealthy endpoints

**Lambda@Edge patterns (exam scenarios):**
1. **URL rewrite:** Clean URLs to S3 key paths (viewer request)
2. **A/B testing:** Randomly assign cookies to treatment groups (viewer request)
3. **Auth at edge:** Validate JWT tokens before allowing origin access (viewer request)
4. **Header injection:** Add security headers to responses (origin response)
5. **Device redirect:** Redirect mobile users to mobile-optimized site (viewer request)

### 4.12 Container Architectures

**Amazon ECS (Elastic Container Service):**

| Feature | Fargate (Serverless) | EC2 (Managed) |
|---------|---------------------|---------------|
| Infrastructure | AWS manages hosts | You manage EC2 instances |
| Pricing | Per vCPU + memory consumed | Per EC2 instance + resources |
| Isolation | Each task gets its own ENI | Tasks share host network |
| Storage | EFS, ephemeral 20–200 GB | EBS, EFS, instance store, ephemeral |
| Best for | Batch, microservices, periodic | GPU, long-running, persistent storage |

- **Task definitions:** JSON describing containers (image, CPU, memory, ports, env vars, volumes, health checks)
- **Service discovery:** AWS Cloud Map or Service Connect
- **Task placement (EC2):** Spread (distribute across AZs/hosts), Binpack (pack densely), Random

**Amazon EKS (Elastic Kubernetes Service):**
- **Managed control plane:** Multi-AZ, auto-scaling, patched by AWS
- **Node groups:** Managed (AWS handles patching/scaling), Self-managed (full control), Fargate profiles (serverless pods)
- **IRSA (IAM Roles for Service Accounts):** Associate IAM roles with K8s service accounts via OIDC provider. Avoids long-lived credentials in pods.
- **Karpenter:** Open-source auto-scaler that provisions right-sized EC2 instances instantly (faster and more efficient than Cluster Autoscaler)
- **EKS Anywhere:** Run EKS on-premises for consistent hybrid Kubernetes
- **Add-ons:** VPC CNI (networking), CoreDNS (DNS), kube-proxy (service routing)

**Amazon ECR (Elastic Container Registry):**
- **Image scanning:** Vulnerability scanning with Inspector (basic or enhanced)
- **Replication:** Cross-region and cross-account replication for latency and DR
- **Lifecycle policies:** Automatically expire old images by age or count
- **Immutable tags:** Prevent tag overwrites for supply chain security
- **Pull-through cache:** Cache public images (Docker Hub, Quay, etc.) in your private registry

### 4.13 Serverless Architectures

**AWS Lambda:**
- **Function design:** Single-purpose, stateless, short-running. Max 15 min execution timeout, 10 GB memory, 10 GB container image (via ECR) or 250 MB unzipped zip.
- **Event sources:** S3 (object create), DynamoDB Streams, Kinesis, SQS, SNS, EventBridge, API Gateway, Cognito, S3 Object Lambda.
- **Destinations:** Route async invocation results (success/failure) to SQS, SNS, EventBridge, or another Lambda function.
- **Reserved concurrency:** Guarantee a specific number of concurrent executions for a function (prevents other functions from consuming all concurrency).
- **Provisioned concurrency:** Pre-warm execution environments to eliminate cold starts. Pay for pre-warmed instances.
- **VPC access:** Functions in a VPC can access RDS, ElastiCache, etc. However, they lose internet access unless using VPC endpoints (for DynamoDB, S3, etc.) or a NAT Gateway.
- **Lambda layers:** Share common code, libraries, or custom runtimes across functions (e.g., SDK layers, utility libraries).
- **SnapStart:** Reduces Java cold start latency by taking a snapshot of the initialized execution environment.

**Lambda resource limits (exam-relevant):**
| Resource | Limit |
|----------|-------|
| Memory | 128 MB – 10,240 MB |
| Ephemeral storage (/tmp) | 512 MB – 10,240 MB |
| Execution timeout | 900 seconds (15 min) |
| Deployment package | 50 MB (zip) / 250 MB (unzipped) / 10 GB (container) |
| Concurrent executions | 1,000 (soft limit, can be increased) |

**Amazon API Gateway:**

| Type | Use Case | Auth Options | Features |
|------|----------|-------------|----------|
| **REST** | Full-featured APIs | IAM, Lambda, Cognito, custom | Usage plans, caching, model validation, SDK generation, API keys |
| **HTTP** | Lower-latency, lower-cost | JWT, Lambda, IAM OIDC | CORS, auto-deploy, no usage plans |
| **WebSocket** | Real-time bidirectional | Lambda, IAM, Cognito | Connection management, route selection, callbacks |

- **Usage plans & API keys:** Throttle and quota API access per client (REST only)
- **Throttling:** 10,000 rps default per region (can be increased). Burst limit of 5,000
- **Caching:** Response caching (0.5 GB to 237 GB), configurable TTL
- **VPC links:** Connect HTTP and HTTP APIs to private ALBs in a VPC via NLB (traffic: API Gateway → NLB → ALB → app)

**AWS Step Functions:**
- **Workflows:** State machines with sequential, parallel, branching, retry, and catch patterns
- **Standard vs. Express:**

| Feature | Standard | Express |
|---------|----------|---------|
| Max execution duration | 1 year | 5 minutes |
| Execution rate | 2,000 / second | 100,000 / second |
| Pricing | Per state transition | Per execution + duration |
| Exactly-once | Yes | At-least-once |
| Use case | Long-running, audit-critical | High-volume, short-lived |

- **Error handling:** Retry with exponential backoff (maxAttempts, intervalSeconds, backoffRate), Catch (fallback state), TimeoutSeconds, HeartbeatSeconds
- **Callback pattern:** Pause execution and wait for an external task token — critical for human approval workflows
- **Direct integrations:** Over 200 AWS services — don't always need a Lambda function between Step Functions and the target service

**Amazon EventBridge:**
- **Event buses:** Default (AWS services), Custom (your applications), Partner (SaaS — Zendesk, Datadog, PagerDuty)
- **Rules:** Content-based pattern matching (e.g., `{"source": ["aws.ec2"], "detail-type": ["EC2 Instance State-change Notification"]}`)
- **Targets:** Lambda, SQS, SNS, Step Functions, Kinesis, ECS tasks, Systems Manager — over 20 targets
- **Schema registry:** Discover, store, and manage event schemas. Generate code bindings for Java, Python, TypeScript.
- **EventBridge Pipes:** Point-to-point integration between sources and targets. Includes filtering, enrichment (with Lambda, Step Functions, API dest), and transformation. Lower cost than chaining EventBridge rules.
- **EventBridge Scheduler:** Schedule tasks on cron expressions or one-time schedules. Replaces many Lambda + CloudWatch Events cron patterns.

**Service comparison (frequent exam topic):**
| Feature | EventBridge | SNS | SQS |
|---------|-------------|-----|-----|
| Pattern | Event routing with filtering | Pub/sub with attribute filtering | Queue |
| Content filtering | Advanced (content-based patterns) | Message attribute-based | None (consumer filters) |
| Consumer type | Multiple target types | Many targets + fan-out | Single consumer group |
| Ordering | Best-effort | Standard (best-effort) / FIFO (ordered) | Standard (best-effort) / FIFO (ordered) |
| Retention | 24 hours | 0–14 days | 1 min – 14 days |

### 4.14 Event-Driven Architectures

**Amazon SQS:**
| Feature | Standard | FIFO |
|---------|----------|------|
| Throughput | Unlimited | 3,000 msg/s (with batching) or 300/s (without) |
| Ordering | Best-effort | Exactly-once, FIFO |
| Delivery | At-least-once | Exactly-once |
| Use case | Any decoupling | Transaction ordering, deduplication needed |

- **Dead-letter queues (DLQ):** Messages exceeding `maxReceiveCount` move to the DLQ for analysis
- **Delay queues:** Messages hidden for up to 15 minutes on arrival
- **Long polling:** Reduces empty responses and cost (`ReceiveMessageWaitTimeSeconds` up to 20s)
- **Visibility timeout:** Message hidden after receive; re-appears if not processed in time (default 30s, max 12h)

**Amazon SNS:**
- **Topics:** Standard (unlimited throughput) or FIFO (ordering + deduplication, only SQS FIFO subscribers)
- **Subscriptions:** HTTP/S, Email (JSON/text), SMS, Lambda, SQS, mobile push (Apple, Google, Amazon, Baidu), platform endpoints
- **Message filtering:** Subscriptions filter by policy — subscriber receives matching messages only
- **Raw message delivery:** Deliver message body directly (not wrapped in SNS envelope)

**Amazon Kinesis:**
| Service | Purpose | Latency |
|---------|---------|---------|
| **Data Streams** | Real-time streaming ingestion (shard-based, 1 MB/s write/shard, 2 MB/s read/shard) | ~200 ms |
| **Data Firehose** | Load streams to S3, Redshift, OpenSearch, Splice Machine, MongoDB, HTTP endpoints | ~60s (near real-time) |
| **Data Analytics** | Real-time SQL or Apache Flink processing on streams | ~1s |
| **Video Streams** | Ingest video from cameras for ML/playback | Real-time |

- **Shard management:** Split (increase shards) or merge (decrease shards) up to daily limit of 24 operations
- **Partition key:** Determines shard assignment (critical for even distribution)
- **Enhanced fan-out:** 2 MB/s per consumer per shard for multiple concurrent consumers

**Amazon MQ:** Managed ActiveMQ or RabbitMQ. For migrating existing message brokers that use JMS, AMQP, MQTT, or STOMP. Less scalable than SQS/SNS but necessary for lift-and-shift of legacy messaging. Available in active/standby (HA) or single-instance.

### 4.15 Database Design

**Amazon RDS:**
- **Multi-AZ:** Synchronous standby replica in another AZ. Automatic DNS failover (~1–2 min). No application changes needed.
- **Read replicas:** Up to 15 asynchronous replicas. Cross-region supported. Reduces primary read load.
- **RDS Proxy:** Connection pooling for Lambda/serverless apps. Queues DB connections, reduces failover time by preserving connections.
- **Performance Insights:** Visualize DB load, find top SQL queries and wait events. Free for 7 days.
- **IAM DB authentication:** Use IAM tokens (15-minute expiry) instead of passwords. Supported for MySQL and PostgreSQL.

**Amazon Aurora:**
- MySQL 5x and PostgreSQL 3x throughput performance compared to standard RDS
- **Global Database:** One primary region writing, up to 5 secondary read-only regions with ~1-second replication latency. Writes to secondary regions are forwarded to primary.
- **Serverless v2:** Auto-scales from 0.5 to 256 ACUs in sub-second increments. Pay per second. Multi-AZ.
- **Parallel Query:** Push analytic queries (large aggregations, scans) down to the storage layer for faster performance.
- **Backtrack:** "Rewind" the cluster to a point in time (from seconds to days back) without restoring from backup. Useful for rapid recovery from user errors.
- **Zero-ETL integration:** No-code integration with Redshift and OpenSearch for analytics queries without ETL pipelines.
- **Storage:** Auto-scaling, up to 128 TiB, six copies across 3 AZs, self-healing storage.

**Amazon DynamoDB:**
- **DAX (DynamoDB Accelerator):** In-memory cache providing sub-millisecond read latency. Write-through caching. Does not support write-heavy use cases as a cache.
- **TTL:** Automatically expire items based on a timestamp attribute. Free, no write cost for expiration. Use for session data, event expiration.
- **Streams:** Change Data Capture (CDC) with ordered, at-least-once delivery. 24-hour retention. Triggers Lambda for downstream processing.
- **Global Tables:** Multi-region, multi-active replication based on DynamoDB Streams. Conflict resolution: last-writer-wins.
- **On-Demand vs. Provisioned:** On-Demand = pay-per-request, unlimited throughput. Provisioned = set RCU/WCU, lower cost for predictable workloads.
- **Adaptive Capacity:** Dynamically splits hot partitions and redistributes throughput. Reduces throttling from uneven access patterns.
- **Transactions:** ACID across multiple items (up to 25 items or 4 MB). 2x cost.
- **Design patterns (exam-critical):** Single-table design (multiple entities in one table with prefix keys), composite sort keys (`status#timestamp` for time-series queries), GSI overloading (reuse GSIs for multiple access patterns), sparse indexes (GSI on optional attribute), write sharding (distribute partition key to avoid hot keys).

**Amazon ElastiCache:**

| Feature | Redis | Memcached |
|---------|-------|-----------|
| Data structures | Rich (strings, hashes, lists, sets, sorted sets, streams, bitmaps) | Simple key-value only |
| Persistence | AOF + RDB snapshots | Ephemeral (in-memory only) |
| Multi-AZ | Redis Cluster with auto-failover | No |
| Replication | Multi-AZ, Global Datastore | No |
| Backup/Restore | Scheduled + on-demand snapshots | No |
| Cluster mode | Sharded with online resharding | Auto-discovery via client library |

- **Global Datastore for Redis:** Fully managed cross-region replication with <1 second latency for disaster recovery.
- **Use cases:** Session stores, cache-aside pattern (lazy loading), rate limiting, leaderboards (sorted sets), message queues (Redis Streams).

**Other managed databases (know their purpose for the exam):**
| Service | Engine | Use Case |
|---------|--------|----------|
| **DocumentDB** | MongoDB-compatible | Lift-and-shift MongoDB workloads |
| **Neptune** | Graph (Gremlin, SPARQL) | Social graphs, recommendation engines, fraud detection, knowledge graphs |
| **Keyspaces** | Apache Cassandra-compatible | Large-scale time-series, IoT device data, high-throughput writes |
| **Timestream** | Custom time-series engine | IoT sensor data, DevOps monitoring, industrial telemetry |
| **QLDB** | Ledger (immutable journal) | Financial transactions, supply chain, audit trails, reconciliation |

### 4.16 Storage Design

**Amazon S3:**
- **Storage classes (ordered by decreasing cost/access, increasing durability):** S3 Standard → S3 Standard-IA → S3 One Zone-IA → S3 Glacier Instant Retrieval → S3 Glacier Flexible Retrieval → S3 Glacier Deep Archive → S3 Intelligent-Tiering
- **Lifecycle policies:** Automate transitions between classes (e.g., Standard → Standard-IA after 30 days → Glacier after 90 days → delete after 365 days)
- **Intelligent-Tiering:** Auto-optimizes cost by moving objects between access tiers (frequent, infrequent, archive instant, archive access, deep archive). Per-object monitoring fee.
- **Object Lock:** WORM storage. Governance mode (admins can override) vs. Compliance mode (no one can delete, including root). Also supports legal holds.
- **Replication:** Same-Region (SRR) and Cross-Region (CRR). Same or different account. Replication Time Control (RTC) guarantees 15-minute replication for compliance.
- **Batch Operations:** Perform actions on millions of objects (copy, encrypt, tag, restore from Glacier).
- **Access Points:** Named network endpoints with dedicated IAM policies. Simplifies per-application or per-team access management.
- **Multi-Region Access Points:** Single global endpoint routing requests to active-active replicated bucket in multiple regions.
- **S3 Object Lambda:** Transform data on retrieval (redact PII, resize images, convert formats, compress/decompress). No need to store multiple variants of the same object.
- **S3 Storage Lens:** Organization-wide storage metrics, usage trends, and optimization recommendations. Free tier includes 14 days of metrics; paid tier provides 15 months, anomaly detection, and bucket-level details.
- **S3 Select / Glacier Select:** Retrieve subset of data using SQL queries (reduces data transfer and cost).
- **Transfer Acceleration:** Fast uploads via CloudFront edge locations. Adds cost.

**Amazon EBS (Elastic Block Store):**
| Volume Type | Max IOPS | Max Throughput | Use Case |
|-------------|----------|----------------|----------|
| **gp3** | 16,000 (base 3,000) | 1,000 MB/s | General purpose, most workloads |
| **io2 Block Express** | 256,000 | 4,000 MB/s | Mission-critical, large databases |
| **io1** | 64,000 | 1,000 MB/s | Legacy IOPS-intensive |
| **st1** (HDD) | 500 | 500 MB/s | Big data, logs, data warehouses |
| **sc1** (HDD) | 250 | 250 MB/s | Cold, infrequent access |

- **Snapshots:** Incremental, stored in S3. Only changed blocks are saved.
- **Fast Snapshot Restore (FSR):** Create volumes from snapshots instantly without pre-warming blocks. Configure FSR on specific AZs.
- **Data Lifecycle Manager (DLM):** Automate EBS snapshot and AMI creation with retention policies. Supports cross-account copy and pre-script/post-script.
- **Multi-Attach (io2):** Attach a single io2 volume to up to 16 Nitro-based EC2 instances in the same AZ for clustered databases (e.g., Teradata, SAP HANA).

**Amazon EFS:**
- **Performance modes:** General Purpose (default, <7000 IOPS) vs. Max I/O (higher throughput/latency, for HPC/big data)
- **Throughput modes:** Bursting (proportional to storage — 50 MB/s per TB) vs. Provisioned (fixed, independent of size)
- **Lifecycle management:** Transition files to EFS IA after N days (cost savings for infrequently accessed files)
- **Replication:** Cross-region file system replication for DR
- **Access points:** Simplify POSIX permissions for application-specific access (enforce user/group/permissions per access point)
- **Classes:** Standard (Multi-AZ) vs. One Zone (single AZ, lower cost)

**Amazon FSx:**
| Service | Engine | Use Case |
|---------|--------|----------|
| **FSx for Lustre** | Lustre | HPC, ML training, video processing, genome analysis |
| **FSx for NetApp ONTAP** | NetApp ONTAP | Lift-and-shift NetApp, NFS/CIFS/SMB, snapshots, cloning, deduplication |
| **FSx for OpenZFS** | OpenZFS | ZFS workloads, snapshots, cloning, NFS, compression |
| **FSx for Windows File Server** | Windows Server SMB | Windows file shares, AD integration, DFS namespaces, shadow copies |

### 4.17 Caching Strategies

| Cache Layer | Service | Use Case |
|-------------|---------|----------|
| **Edge CDN** | CloudFront | Static content, API response caching, global distribution |
| **Application in-memory** | ElastiCache Redis | Session store, database query results, rate limiting |
| **Database-specific** | DynamoDB DAX | Sub-millisecond DynamoDB reads |
| **Connection pool** | RDS Proxy | Reuse DB connections, reduce failover time |

**Cache-aside (lazy loading):** Application checks cache → miss → reads from database → populates cache → returns. Simple but first read is slow.

**Write-through:** Application writes to cache → cache synchronously writes to database. Consistent but higher write latency.

**Common exam scenario:** Use ElastiCache to cache RDS query results for read-heavy workloads. Use DAX to reduce DynamoDB read latency. Use CloudFront for global static/dynamic content delivery.

### 4.18 Loosely Coupled Architectures

Key design patterns for the exam:

| Requirement | Architecture |
|-------------|--------------|
| Order processing, decouple front-end from back-end | API Gateway → SQS → Lambda → DynamoDB |
| Fan-out to multiple microservices | SNS → SQS queues (one per service) |
| Real-time stream processing | Kinesis Data Streams → Data Analytics → S3 |
| Scheduled batch processing | EventBridge Scheduler → Lambda or Batch |
| Human approval workflow | Step Functions callback → SNS → URL → task token resume |
| Image processing pipeline | S3 event → SQS → Lambda thumbnail → S3 output |

**Core principles:** Idempotency (safe to retry), retry with exponential backoff + jitter (avoid thundering herd), async processing (producer doesn't wait for consumer), dead-letter queues (handle failures gracefully).

---

## 5. Domain 3: Migration Planning (15%)

### 5.1 The 7 R's
| Strategy | What | Speed | Optimization |
|----------|------|-------|-------------|
| **Refactor** | Re-architect (serverless/containers) | Slowest | Highest |
| **Replatform** | Managed services (RDS, ECS Fargate) | Medium | High |
| **Repurchase** | SaaS (e.g., Salesforce) | Fast | Variable |
| **Rehost** | Lift-and-shift (MGN) | Fastest | Lowest |
| **Relocate** | VMware Cloud on AWS (HCX) | Fast | Low |
| **Retain** | Keep on-premises | N/A | N/A |
| **Retire** | Decommission | N/A | Savings |

### 5.2 AWS Cloud Adoption Framework (CAF)
Six perspectives: Business (ROI, risk), People (culture, training), Governance (portfolio, data), Platform (architecture, workloads), Security (identity, encryption), Operations (monitoring, incident management).

### 5.3 Migration Phases
1. **Assess:** Inventory, dependency mapping, TCO. Tools: Migration Evaluator, Migration Hub.
2. **Mobilize:** Landing zone, governance, security, skills. Tools: Control Tower, Organizations.
3. **Migrate:** Waves, cutover, validation. Tools: MGN, DMS, DataSync, Snow Family.

### 5.4 Data Migration Services
| Service | Use Case | Speed |
|---------|----------|-------|
| **DataSync** | NFS/SMB to S3/EFS/FSx | Up to 10 Gbps |
| **Transfer Family** | SFTP/FTPS/FTP to S3 | Transfer speed |
| **Snowcone** | Small data (8/14 TB) | Ship (days) |
| **Snowball Edge** | Medium (80 TB) | Ship (days) |
| **Snowmobile** | Massive (100 PB/truck) | Ship (weeks) |
| **Storage Gateway** | Hybrid (File/Volume/Tape) | Online |
| **MGN** | Server volume replication | Near real-time |

**Decision:** Online (DataSync, MGN) when bandwidth sufficient, <50 TB. Offline (Snow) when limited bandwidth, remote, petabyte-scale.

### 5.5 Server Migration
- **MGN (CloudEndure):** Block-level continuous replication, supports Windows/Linux/x86. Non-disruptive testing, automated cutover.
- **VM Import/Export:** Import VMs as AMIs, export EC2 back on-premises.

### 5.6 Database Migration
- **DMS:** Homogeneous (MySQL→Aurora MySQL) and heterogeneous (Oracle→Aurora PostgreSQL, requires SCT). CDC for ongoing replication. Validation compares source/target.
- **SCT:** Converts schema, stored procedures, views, triggers. Reports complexity. **Not needed** for homogeneous migrations.

### 5.7 Landing Zones
- **Control Tower:** Fast setup, managed guardrails, Account Factory, opinionated.
- **Custom:** Full flexibility, complex, integrates existing policies.
- **Hybrid:** Control Tower + CfCT for extensions.

### 5.8 Migration Wave Planning
Wave grouping by dependencies, prioritization (easy first), cutover planning with rollback procedures, post-migration validation.
- **Greenfield:** New environment, full design freedom. **Brownfield:** Integrate with existing accounts, networking, governance.
- **AWS Prescriptive Guidance:** Migration patterns, strategies, playbooks, architecture guidance.

---

## 6. Domain 4: Cost Control (10%)

### 6.1 AWS Pricing Model
- **Compute:** On-Demand, Reserved (Standard up to 72%, Convertible up to 54%), Savings Plans (Compute up to 66%, EC2 Instance up to 72%), Spot (up to 90%).
- **Storage:** Per GB-month + request pricing. **Data transfer:** Inbound free, outbound charged ($0.12/GB first 10TB tier). Inter-region charged, inter-AZ charged (~$0.01/GB), same-AZ free.
- **CloudFront/Direct Connect:** Lower egress costs.

### 6.2 Purchasing Options Detail
- **Reserved Instances:** Standard vs. Convertible, regional vs. zonal, 1/3 year terms. RI Marketplace, modification (AZ, scope, size, not region/family for Standard).
- **Savings Plans:** Compute SP (any EC2/Fargate/Lambda, any region) vs. EC2 Instance SP (specific family in region). Hourly $/hr commitment. All/Partial/No Upfront.
- **Spot:** Capacity pools (instance type + AZ). Allocation strategies: lowestPrice, capacityOptimized, capacityOptimizedPrioritized. Spot Fleet (mix of spot + on-demand). Diversification reduces interruptions. Use for batch, CI/CD, ML, stateless, fault-tolerant.

### 6.3 Cost Management Tools
| Tool | Purpose |
|------|---------|
| Cost Explorer | Visualization, forecasting, tagging |
| Cost & Usage Report (CUR) | Detailed CSV/Parquet, Athena+QuickSight |
| Budgets | Cost/usage/RI/SP budgets + alerts |
| Cost Anomaly Detection | ML-based anomaly detection |
| Reserved Instance Reporting | Utilization + coverage |
| Savings Plans Recommendations | ML-driven purchase recommendations |
| Compute Optimizer | Right-sizing (EC2, ASG, Lambda, EBS) |
| Trusted Advisor | Cost checks (idle, underutilized, RI) |

### 6.4 Cost Optimization Strategies
1. **Right-sizing:** Compute Optimizer to downsize over-provisioned.
2. **Purchasing:** RIs/SPs for baseline, Spot for variable/fault-tolerant, On-Demand for unpredictable.
3. **Storage:** Lifecycle policies, Intelligent-Tiering, delete orphaned EBS/old snapshots, Storage Lens.
4. **Data transfer:** CloudFront, Direct Connect, compression, CDN, Transfer Acceleration.
5. **Architecture:** Serverless (zero-cost idle), containers (better utilization), Graviton (40% savings), Auto Scaling.

### 6.5 Cost Governance
- **Budgets:** Cost, usage, RI/SP coverage/utilization. SNS alerts at thresholds.
- **SCPs:** Deny expensive resources in non-prod (GPU instances, large instance types).
- **Tag enforcement:** Config rules + SCPs for required cost tags.
- **Cost Categories:** Hierarchical allocation (BU → Dept → Team).
- **CUR** for chargeback/showback. **Athena + QuickSight** for dashboards.
- **Pricing Calculator:** Estimate costs pre-deployment.

### 6.6 Cost Allocation
Standardized tags (`CostCenter`, `Environment`, `Project`), Cost Categories (tag-based grouping), cross-account cost allocation via CUR, percentage split for shared costs (TGW, DX, NAT).

---

## 7. Domain 5: Continuous Improvement for Existing Solutions (21%)

### 7.1 Monitoring & Observability
- **CloudWatch:** Metrics (standard + custom via PutMetricData), Logs (subscription/metric filters, export to S3/Firehose/Lambda), Alarms (OK/ALARM/INSUFFICIENT_DATA), Dashboards, Contributor Insights (top contributors), Container/Lambda Insights, ServiceLens (traces + metrics + logs), Synthetics (canaries with Puppeteer).
- **X-Ray:** Service maps, traces (annotations, sampling rules: 1/sec + 5% additional), segments/subsegments. SDK for instrumentation.
- **Managed Grafana/Prometheus:** AMP (Prometheus metrics) + AMG (Grafana dashboards) for enterprise observability.

### 7.2 Reliability Improvements
- **DR patterns:** Active-Active (RTO<1min, RPO seconds), Warm Standby (15min, RPO minutes), Pilot Light (30min, RPO hours), Backup & Restore (hours, hours).
- **AWS Backup:** Centralized. Backup policies (schedule, retention, cross-region copy), Vault Lock (WORM).
- **Incident response:** Runbooks (SSM Documents), Systems Manager Automation, Incident Manager (on-call, response plans, post-incident analysis), Personal Health Dashboard → EventBridge.

### 7.3 Security Improvement
| Service | Function |
|---------|----------|
| **Security Hub** | Findings aggregation, CIS/PCI/SOC2 standards. Enable org-wide via delegated admin. |
| **GuardDuty** | Threat detection (CloudTrail, VPC Flow, DNS, EKS, RDS, S3). Automate response. |
| **Inspector** | Vulnerability scanning (EC2, ECR, Lambda, network reachability). |
| **Macie** | Sensitive data discovery in S3 (PII, credentials, financial). |
| **Detective** | Root cause analysis of security findings. |
| **Config auto-remediation** | Non-compliant → EventBridge → SSM Automation → fix. |

### 7.4 Performance Optimization
- **Compute Optimizer:** Right-size EC2/ASG/Lambda/EBS.
- **Auto Scaling:** Target tracking, step, scheduled, predictive.
- **Caching:** CloudFront, ElastiCache, DAX.
- **Database:** Performance Insights, query tuning, indexing, partitioning.
- **Lambda Power Tuning:** Tune memory for cost/speed tradeoff.

### 7.5 Operational Excellence
- **Systems Manager:** Automation (runbooks), Run Command (execute across instances), Patch Manager (OS patching), Session Manager (SSH without bastion), Parameter Store (hierarchical config), State Manager (OS config).
- **CloudFormation:** Drift detection, change sets, StackSets, resource import.
- **CDK:** Infrastructure as code in TypeScript/Python/Java/C#/Go.
- **Terraform:** Multi-cloud IaC.

### 7.6 Well-Architected Framework
**Six pillars:** Operational Excellence (run as code, anticipate failure, learn from incidents), Security (strong identity, traceability, encrypt all layers), Reliability (auto-recover, test recovery, scale horizontally), Performance Efficiency (serverless, go global, experiment), Cost Optimization (consumption model, measure efficiency), Sustainability (minimize energy, maximize utilization).
- **Well-Architected Tool:** Pillar-specific questions → improvement plan.
- **Lenses:** Serverless, HPC, IoT, Streaming, Data Analytics, SAP, Financial Services.

### 7.7 Event-Driven Automation
| Trigger | Action |
|---------|--------|
| Config NON_COMPLIANT | EventBridge → SSM remediate |
| GuardDuty finding | EventBridge → Step Function → isolate instance |
| CloudWatch ALARM | EventBridge → Lambda scale/restart |
| Cost anomaly | SNS → Lambda notify + pause |
| Budget threshold | SNS → Lambda stop dev instances |
| Schedule | EventBridge Scheduler → Lambda optimize |

**Common remediations:** Encrypt unencrypted EBS, apply S3 Block Public Access, revoke open SG ports, stop cost-exceeding instances.

---

## 8. Exam Preparation Tips

### 8.1 Study Resources
| Category | Resources |
|----------|-----------|
| **AWS Docs** | Service FAQs, User Guides, Whitepapers (Well-Architected, Organizations, DR, Migration) |
| **Skill Builder** | Official courses, Exam Readiness, Builder Labs |
| **re:Invent** | YouTube breakout sessions for SAP-C02 |
| **Practice Exams** | Tutorials Dojo (recommended), WhizLabs, AWS Official ($20) |
| **Flash Cards** | Memrise, Anki (community SAP-C02 decks) |
| **Hands-On** | Free Tier, AWS Workshops (workshops.aws), personal projects |

### 8.2 Study Plan (3–6 Months)
- **Weeks 1–4:** Read all 5 domain whitepapers, baseline practice test.
- **Weeks 5–10:** Deep dive weak domains, weekly hands-on labs, detailed notes.
- **Weeks 11–14:** Cross-domain scenarios, full practice exams.
- **Weeks 15–18:** Timed practice exams (180 min), flash cards for service limits.
- **Weekly:** 1–2 hours daily + 3–4 hours weekend labs.

### 8.3 Question Strategy
1. Read ALL answers before choosing.
2. Eliminate obviously wrong options first.
3. Identify keywords: "lowest cost", "minimize overhead", "HA", "fault tolerance".
4. Watch for constraints: budget, compliance, team skill, existing infra.
5. For multi-response: select no more than you're confident about.
6. Know WHY correct and why others wrong — this is the real exam skill.

### 8.4 Common Traps
| Trap | Truth |
|------|-------|
| CloudFront vs. Global Accelerator | CF = HTTP + cache. GA = TCP/UDP + network optimization. |
| Config vs. CloudTrail | Config = resource state. CloudTrail = API calls. |
| SG vs. NACL | SG = stateful, allow only. NACL = stateless, allow+deny, ordered. |
| Regional vs. Global | WAF regional for ALB, global for CloudFront. Config per region. |
| Cost vs. HA | Read carefully: "cheapest" vs. "most available" give different answers. |
| DMS + SCT | SCT only for heterogeneous migrations. DMS alone for homogeneous. |

### 8.5 Hot Services by Priority
- **Must Know:** Organizations, Control Tower, SCPs, TGW, Direct Connect, Route 53, CloudFront, WAF, Shield, Lambda, API Gateway, Step Functions, EventBridge, SQS, SNS, Kinesis, DynamoDB, Aurora, S3, EC2 ASG, RDS.
- **Very Important:** DMS, MGN, Systems Manager, CloudWatch, Config, CloudTrail, GuardDuty, Security Hub, IAM, KMS, CloudFormation.
- **Important:** Elastic Beanstalk, OpsWorks, Service Catalog, Macie, Inspector, ElastiCache, ECS/EKS, VPC (NACL, SG, TGW, VPN).

### 8.6 Key Whitepapers
1. AWS Well-Architected Framework
2. AWS Organizations User Guide
3. Architecting for the Cloud: AWS Best Practices
4. Disaster Recovery of Workloads on AWS
5. AWS Security Best Practices
6. AWS Migration Best Practices (Prescriptive Guidance)
7. AWS Cloud Adoption Framework
8. AWS Cost Management User Guide

---

## 9. Domain-by-Domain Study Checklist

### 9.1 Domain 1: Organizational Complexity
- [ ] Design multi-account strategy (management, security, log archive, shared services, network, workload)
- [ ] Design SCP hierarchy (Root → OUs → Accounts), understand deny vs. allow lists
- [ ] Set up Control Tower landing zone (guardrails, Account Factory, CfCT)
- [ ] Understand cross-account access (IAM roles, resource-based policies, permission boundaries)
- [ ] Configure IAM Identity Center (permission sets, SAML/OIDC/SCIM, AD integration)
- [ ] Multi-account Config (aggregators, conformance packs, auto-remediation)
- [ ] CloudTrail organization trail (log aggregation, data vs. management events)
- [ ] TGW network architecture (segmentation, centralized egress)
- [ ] Tag strategies and enforcement
- [ ] Service Catalog (portfolios, constraints, launch roles)
- [ ] CloudFormation StackSets for multi-account deployments

### 9.2 Domain 2: New Solutions
- [ ] Multi-region active-passive (Route 53 failover, Aurora Global, DynamoDB Global Tables)
- [ ] Multi-region active-active (Global Accelerator, DynamoDB Global Tables)
- [ ] Route 53 routing policies, health checks, private hosted zones, Resolver
- [ ] CloudFront (behaviors, origins, Lambda@Edge vs. CF Functions, signed URLs)
- [ ] WAF (web ACLs, rate-based, managed rules, bot control, CAPTCHA)
- [ ] Shield Advanced (DRT, proactive engagement, cost protection)
- [ ] Hybrid DX + VPN (private/transit VIF, SiteLink, MACsec, LAG)
- [ ] TGW (inter-region peering, multicast, segmentation)
- [ ] Containers (ECS Fargate vs. EC2, EKS Fargate profiles, IRSA, ECR replication)
- [ ] Serverless (Lambda, API Gateway, Step Functions, EventBridge)
- [ ] Event-driven (SQS, SNS, Kinesis, EventBridge integration)
- [ ] Database selection (RDS Multi-AZ, Aurora Global, DynamoDB DAX, ElastiCache)
- [ ] S3 storage classes, lifecycle, replication, Object Lock, access points
- [ ] Caching strategies (CloudFront, ElastiCache, DAX)
- [ ] Loosely coupled architectures (queues, pub/sub, async, idempotency)

### 9.3 Domain 3: Migration Planning
- [ ] Plan migration waves (prioritization, dependency analysis, cutover, rollback)
- [ ] Select data transfer service (DataSync, Snow, MGN, Storage Gateway)
- [ ] Design landing zone (Control Tower or custom, multi-account baseline, security)
- [ ] Understand CAF (6 perspectives) and 7 R's
- [ ] Database migration (DMS + SCT, homogeneous vs. heterogeneous, CDC)
- [ ] MGN (continuous replication, test instances, cutover)
- [ ] Assess readiness (Migration Evaluator, Migration Hub, dependency mapping)
- [ ] Greenfield vs. brownfield considerations
- [ ] AWS Prescriptive Guidance for patterns and strategies

### 9.4 Domain 4: Cost Control
- [ ] Choose optimal purchasing (RIs, Savings Plans, Spot) for workload patterns
- [ ] Set up cost governance (Budgets, SCPs, tagging, anomaly detection)
- [ ] Identify cost optimization (right-sizing, lifecycle policies, spot adoption)
- [ ] Tag strategy for cost allocation (tags, Cost Categories, percentage split)
- [ ] Design cost-efficient architectures (serverless, containers, Graviton, ASG)
- [ ] Optimize data transfer (CloudFront, Direct Connect, compression)
- [ ] Use Cost Explorer, CUR, Budgets for monitoring
- [ ] Compute Optimizer + Trusted Advisor cost checks
- [ ] Savings Plans (Compute vs. EC2 Instance, payment options)
- [ ] Spot (interruptions, allocation strategies, diversification)
- [ ] AWS Pricing Calculator estimates

### 9.5 Domain 5: Continuous Improvement
- [ ] Comprehensive monitoring (CloudWatch metrics/logs/alarms/dashboards, Contributor Insights)
- [ ] X-Ray tracing (service maps, annotations, sampling)
- [ ] DR design (RPO/RTO, AWS Backup, cross-region backup, vault lock)
- [ ] Incident response (runbooks, SSM Automation, Incident Manager)
- [ ] Security improvement (Security Hub, GuardDuty, Inspector, Macie, Detective)
- [ ] Config rules for compliance (managed, custom, auto-remediation)
- [ ] Performance optimization (right-sizing, ASG, caching, DB tuning)
- [ ] CloudWatch Synthetics (canaries, visual monitoring, API monitoring)
- [ ] Event-driven remediation (EventBridge → Lambda)
- [ ] Operational Excellence (SSM Automation, Run Command, Patch Manager, Session Manager, Parameter Store)
- [ ] Infrastructure as Code (CloudFormation, CDK, Terraform, StackSets, drift detection)
- [ ] Well-Architected Framework (6 pillars, tool, lenses)

---

## 10. Exam Day Tips
- **180 min / 75 questions** ≈ 2.4 min per question
- **Case studies:** 5–8 questions per scenario — read carefully
- **Flag and return** to uncertain questions
- **Elimination:** If you can remove 2 of 4, you have a 50% chance
- **Sleep well, eat before, arrive early**
- **Hands-on experience is mandatory** — theory alone won't pass

---

> **About this guide:** A living study reference for Jack Liu Shurui's AWS SAP-C02 preparation, part of the Enterprise Architect skill gap analysis. All information reflects the current SAP-C02 exam guide as of July 2026. AWS services evolve — always cross-reference with current AWS documentation and the official SAP-C02 exam guide.
