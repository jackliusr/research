# Data Architect Skill Gaps — Based on the Digital Profile

## A Personalized Skill-Gap Analysis: Solution Architect → Data Architect

> **For:** Jack Liu Shurui — Solution Architect (GenAI), Crédit Agricole CIB, Singapore
> **Analysis Date:** August 2026
> **Source Material (the digital profile):** github.com/jackliusr/research (~379 commissioned guides across banking/, technology/, management/, personal/)
> **Target Role:** Data Architect / Senior Data Architecture roles in banking and enterprise
> **Sibling guide:** architecture/skill_gaps_enterprise_architect_guide.md (Solution Architect → Enterprise Architect)

---

## 0. Executive Summary

Jack Liu Shurui is a Singapore-based Solution Architect at Crédit Agricole CIB's Corporate & Investment Bank, with a career spanning full-stack development, DevOps/SRE, solution architecture, and team leadership across SCADA, healthcare (FHIR), ERP, e-commerce, payments, gaming, and core banking. His research repository — the digital profile analyzed here — contains ~379 guides, of which **~55 are directly data-related** (25 in `technology/data/` plus data guides at the `technology/` root, ~12 data-model guides in `banking/`, and the feature-store / vector-DB / closed-loop-data guides in `ai_llm/`).

**Core finding: the profile is DENSE where most aspiring data architects are weak, and thin exactly where the modern data-architect job market is moving.** The repo shows deep, evidence-backed coverage of the *classical* data-architecture core — dimensional modeling, Data Vault 2.0, data mesh, data fabric, lakehouse, integration patterns with a genuine CDC deep-dive, 1,500-line global compliance coverage (GDPR + PDPA included), and a full 944-line governance guide that ends in a worked BCBS 239 example. That is a top-decile knowledge base for data modeling, architecture patterns, integration, governance, and platforms.

The genuine gaps cluster in six places, three of which are standing priorities carried forward from prior sessions:

1. **Managed cloud data warehouses (Snowflake / BigQuery / Redshift / Synapse)** — referenced ~67 times in `data/enterprise_data_platforms_guide.md` but with **no dedicated guide**; the single most likely product-level question in a 2026 data-architect interview.
2. **Business case & TCO modeling for data initiatives** (standing priority) — `finops_guide.md` covers cloud cost, but no dedicated business-case/TCO methodology exists.
3. **Zero Trust for data platforms** (standing priority) — NIST SP 800-207 appears in only 5 files as a mention; no dedicated guide.
4. **Platform Engineering for data** (standing priority) — the self-serve data-platform / data-product-platform layer that makes data mesh operational; mentions only.
5. **Streaming operations depth** — excellent conceptual coverage (`event_stream_processing_guide.md`, 1,218 lines, banking chapter included) but no dedicated Kafka-internals, Flink, or schema-registry guide, and CDC lives only as a section of `data_integration_frameworks_guide.md`.
6. **Data quality / observability / metadata-catalog / MDM operationalization** — all present as *sections* of the governance guide, but none has a dedicated guide; no dbt, no data-contracts, no DAMA-DMBOK formal alignment.

**Overall readiness estimate: ~70–75% of a senior data-architect profile.** The remaining distance is bridgeable in 9–12 months using the repo's own proven pattern — commissioning 400–900-line companion guides (5 recommended in this document) plus ~4 certification-aligned study tracks (DAMA-DMBOK, cloud DW, streaming ops, data-platform TCO).

**Standing priorities carried forward:** Zero Trust, Platform Engineering, business case/TCO — each confirmed as a real gap by this scan (no dedicated guide exists for any of the three) and each folded into the prioritized list (gaps #2, #3, #4).

---

## 1. The Data-Architect Skill Framework (Assessment Axes)

A data architect designs, governs, and evolves the enterprise's data landscape: models, pipelines, platforms, and the policies that make data trustworthy, secure, and usable. The framework below defines ten assessment axes, synthesized from DAMA-DMBOK practice areas, modern data-platform job profiles, and the 2026 market shift toward cloud-native, streaming, and AI-ready data. Each axis states **what it requires** — these requirements are the rubric used for the heatmap in §2.

### 1.1 Data Modeling

What it requires: conceptual/logical/physical modeling; relational normalization; **dimensional modeling** (Kimball star schemas, conformed dimensions, slowly changing dimensions, fact tables — additive/semi-additive/non-additive, degenerate dimensions); **Data Vault 2.0** (hubs, links, satellites, hash keys, PIT tables, bridge tables); **anchor modeling**; NoSQL modeling (document, key-value, wide-column, graph — adjacency vs. property-graph, indexing and query patterns); canonical/universal data models and industry models (BIAN, ACORD, FIBO); model versioning, migration, and forward-compatibility; model-to-physical mapping on MPP and cloud engines.

### 1.2 Data Architecture Patterns

What it requires: enterprise data architecture (DAMA context diagram, data flows); warehouse (Inmon 3NF) vs. star-schema (Kimball) vs. Data Vault reference architectures; **data lake**, **lakehouse**, **data mesh** (four principles, data products, federated computational governance), **data fabric** (active metadata, knowledge graphs); medallion/bronze-silver-gold and its alternatives; lambda vs. kappa; late-arriving data, schema evolution, duplicate-key handling; pattern selection criteria (scale, latency, regulation, org shape); reference architectures for regulatory reporting (BCBS 239, MAS 626).

### 1.3 Data Integration

What it requires: ETL vs. ELT vs. EtLT; batch orchestration and batch-window optimization; **CDC** (log-based, timestamp-based, trigger-based; Debezium, Oracle GoldenGate, AWS DMS); API-based integration (REST, GraphQL, event APIs); file-based integration (SFTP/CFT, EDI, SWIFT MT/MX, ISO 20022); messaging and queuing (Kafka, MQ, JMS); idempotency, exactly-once vs. at-least-once, backfills and reprocessing; data pipeline versioning, lineage of pipelines, failure recovery, dead-letter handling; integration platform evaluation (iPaaS vs. hand-rolled).

### 1.4 Data Platforms

What it requires: relational engines (Oracle, PostgreSQL, MySQL, SQL Server); MPP engines (Teradata, Exadata, Greenplum, Vertica); **cloud data warehouses (Snowflake, BigQuery, Redshift, Synapse)**; **lakehouse platforms (Databricks, Fabric, open table formats Delta/Iceberg/Hudi/Paimon)**; graph databases (Neo4j, Neptune); search engines (Elasticsearch/OpenSearch); vector databases (Pinecone, Milvus, Weaviate, Qdrant, pgvector); object storage (S3, GCS, ADLS) and file formats (Parquet, ORC, Avro); NoSQL engines (Cassandra, DynamoDB, MongoDB, HBase); engine-selection trade-offs (cost, concurrency, latency, governance fit); platform operations: tuning, partitioning, clustering, vacuuming/compaction, cost governance.

### 1.5 Streaming

What it requires: event-driven architecture; stream processing engines (Kafka Streams, Flink, Spark Structured Streaming, Pulsar); event time vs. processing time, watermarks, windows, late data; state management and fault tolerance (checkpointing, exactly-once semantics); schema registry and contract evolution; CDC-to-stream patterns; stream–batch unification (Kappa, unified engines); ordering and partitioning; backpressure; observability of streaming pipelines; real-time use cases in banking (fraud, AML, intraday liquidity, real-time risk, trade surveillance).

### 1.6 Governance

What it requires: governance frameworks and operating models (DAMA-DMBOK, COBIT, DGI); policies, standards, roles (CDO, data stewards, data owners); **data quality management** (dimensions, measurement, remediation, DQ SLOs); **metadata management** (technical, business, operational; active metadata); **master data management** (party, product, account, security/reference data; hub/registry/coexistence); lineage and impact analysis; data catalogs (Collibra, DataHub, OpenMetadata, Alation); data contracts; regulatory data governance (BCBS 239, MAS 626/Notice 645, GDPR, PDPA); data classification and lifecycle management.

### 1.7 Security & Privacy

What it requires: data classification and protection; encryption at rest/in transit and key management (KMS/HSM); tokenization, masking, anonymization/pseudonymization; access control for data (RBAC/ABAC, row/column-level security, dynamic masking); secure data sharing (delta sharing, clean rooms); secrets management; audit logging of data access; **Zero Trust applied to data planes** (NIST SP 800-207, micro-segmentation, identity-aware access, continuous verification); privacy engineering: PDPA (Singapore), GDPR, cross-border transfer mechanisms (SCCs, DPF), privacy-by-design, DPIAs; regulatory context (MAS TRM, MAS Notice 645 on technology risk).

### 1.8 Cloud Data

What it requires: cloud data services across AWS/GCP/Azure (S3/GCS/ADLS, Redshift/BigQuery/Synapse, Glue/Dataflow/ADF, EMR/Dataproc/HDInsight, KMS/KMS/AKV); multi-cloud and hybrid data architecture; cloud data migration (lift-and-shift vs. re-platform vs. re-architect), migration tooling and cutover; data gravity and egress cost; FinOps for data (storage tiering, compute auto-scaling, warehouse cost governance); region/residency and data sovereignty (Singapore IMDA, MAS outsourcing); cloud-native table formats and managed lakehouse; data platform TCO across cloud vs. on-prem.

### 1.9 ML Data

What it requires: feature engineering and **feature stores** (online/offline consistency, point-in-time correctness, feature validation); **vector databases and embeddings pipelines**; RAG data architecture (chunking, indexing, retrieval evaluation); training-data management (labeling, curation, versioning, data-centric AI); data quality for ML (drift detection — data drift, concept drift, model drift); closed-loop data engineering (feedback → retraining); ML data lineage and reproducibility; eval-data governance; serving data (batch vs. online inference, feature serving); MLOps platform data layer.

### 1.10 Business & Communication

What it requires: translating business problems into data architecture; **business case development and TCO modeling** (NPV/IRR/payback, option value, build-vs-buy, 3–5-year cost projections, sensitivity analysis); stakeholder communication (business, risk, compliance, engineering); data-driven storytelling for executives; consulting methodology (issue trees, MECE, hypothesis-driven analysis); vendor evaluation and negotiation; influencing without authority; architecture decision records and trade-off communication; regulatory dialogue literacy (MAS, BCBS).

### 1.11 Scoring Rubric (how each axis is assessed)

Every axis in §2 is scored with a five-level proficiency scale, so the heatmap is reproducible and the learning plan (§5) maps directly to level deltas:

| Level | Label | What it means in evidence terms |
|---|---|---|
| L5 | **Expert / DENSE** | Multiple dedicated guides (≥4) plus verified section-level depth in adjacent guides; can design, defend trade-offs, and teach the axis; banking context applied |
| L4 | **Strong / DENSE** | 2–3 dedicated guides or 1 dedicated guide + strong sections elsewhere; can design and defend standard patterns |
| L3 | **Working / thin** | Section-level coverage inside other guides (verified by TOC), or 1 dedicated comparison-style guide; can discuss competently but not operate/build at depth |
| L2 | **Familiar / thin** | Mention-level only (keyword hits across the repo, no dedicated treatment); can name concepts but not design with them |
| L1 | **Absent / gap** | No meaningful evidence in the repo; topic does not appear or appears once incidentally |

Evidence rules used throughout this document: (a) a **dedicated guide** (filename + line count) is the primary evidence unit; (b) **section-level** coverage counts only when verified by TOC grep (e.g., CDC §6 of `data_integration_frameworks_guide.md`); (c) **keyword mentions** never constitute coverage on their own — they are flagged as mentions and used only to distinguish "comparison-level" from "absent"; (d) banking-domain guides count toward the relevant axis when they treat the data aspect (e.g., `../banking/temenos_data_model_guide.md` → Data Modeling). Target-state definition: **L4 on all ten axes, with L5 on Data Modeling, Governance, and Data Architecture Patterns** — the three axes that define the senior data-architect brand in banking.

### 1.12 How the ten axes map to DAMA-DMBOK knowledge areas

The ten axes are the assessment lens; DAMA-DMBOK (gap #7) is the vocabulary a data architect is expected to speak. The mapping shows that the profile already evidences most DMBOK knowledge areas — the missing piece is the *framework itself*:

| DMBOK knowledge area | Maps to axis | Profile evidence status |
|---|---|---|
| Data Governance | 1.6 Governance | DENSE (`data_governance_guide.md`) |
| Data Architecture | 1.2 Patterns + 1.1 Modeling | DENSE |
| Data Modeling & Design | 1.1 Data Modeling | DENSE |
| Data Storage & Operations | 1.4 Data Platforms + 1.8 Cloud Data | DENSE (on-prem) / thin (managed cloud) |
| Data Security | 1.7 Security & Privacy | thin (data-plane; Zero Trust gap) |
| Data Integration & Interoperability | 1.3 Data Integration | DENSE |
| Document & Content Management | (RAG/vector coverage) | DENSE via `ai_llm/rag/` |
| Reference & Master Data | 1.6 Governance (MDM) | thin (section-level) |
| Data Warehousing & BI | 1.4 + 1.1 | DENSE |
| Metadata Management | 1.6 Governance (metadata) | thin (section-level; catalog gap) |
| Data Quality | 1.6 Governance (DQ) | thin (section-level; observability gap) |
| Big Data & Data Science | 1.9 ML Data | DENSE |
| Data Mining / Analytics | 1.9 ML Data | DENSE (CLV, analytics guides) |

The pattern is striking: **the profile has done the work DMBOK describes, without ever studying the DMBOK itself.** Commissioning `dama_dmbok_guide.md` is therefore a formalization task — map existing guides to the 11 knowledge areas and add the CDMP exam angle — rather than net-new learning, which is why its effort estimate is low (§5, gap #7).

---

## 2. Digital-Profile Heatmap

Method: every `.md` guide in the repo was listed and classified by axis. **DENSE** = multiple dedicated guides (≥4) plus section-level coverage in adjacent guides; **thin** = 1–3 dedicated guides or section-level coverage only; **gap** = no dedicated guide; topic appears only as mentions. Guide line-counts (wc -l) are shown as evidence of depth. All paths are relative to the `technology/` directory where this guide lives; `data/` = `technology/data/`, `ai_llm/` = `technology/ai_llm/`, `ai_llm/rag/` = `technology/ai_llm/rag/`, `../banking/` and `../management/` = repo siblings.

### 2.11 Raw scan counts per axis (the numbers behind the heatmap)

| Axis | Dedicated guides | Section-level support (verified) | Mention-level only | Verdict |
|---|---|---|---|---|
| Data Modeling | ~16 | `data_governance_guide.md` §8 (MDM models) | anchor modeling | **DENSE (L5)** |
| Architecture Patterns | ~14 | `databricks_guide.md` §2 (lakehouse), `data_governance_guide.md` §12 (BCBS 239 RA) | data contracts | **DENSE (L5)** |
| Data Integration | ~13 | CDC §6 of `data_integration_frameworks_guide.md`; ISO 20022 in `../banking/iso_20022_core_processes_guide.md` | dbt, Airflow, iPaaS | **DENSE (L4)** |
| Data Platforms | ~18 | `databricks_guide.md` §7 (security), §4 (three clouds) | Snowflake/BigQuery/Redshift (0 dedicated), PostgreSQL/MySQL/MongoDB | **DENSE (L4)\* — cloud-DW product layer L2** |
| Streaming | 2 dedicated | `event_stream_processing_guide.md` §9–10 (banking, CEP), §6–7 (time/state) | Kafka internals, Flink, schema registry | **thin (L3)** |
| Governance | ~13 | DQ §6 / metadata §7 / MDM §8 in `data_governance_guide.md`; compliance §3 (cross-border) | DMBOK formal, Collibra/DataHub/OpenMetadata, data contracts | **DENSE (L4/L5)** |
| Security & Privacy | 13 (few data-specific) | `databricks_guide.md` §7 (security & governance in banking); compliance frameworks (GDPR/PDPA) | Zero Trust (5 files), KMS/masking/RLS | **thin (L3)** |
| Cloud Data | ~6 (infra-heavy) | `databricks_guide.md` §4; `data/enterprise_data_platforms_guide.md` | managed DW services, migration playbooks, residency patterns | **thin (L3)** |
| ML Data | ~10 | 18 RAG guides (`ai_llm/rag/`); `ml_system_design_interview_guide.md` | training-data lifecycle, eval-data governance | **DENSE (L4)** |
| Business & Communication | ~15 | `product_thinking_guide.md`, `../management/forward_deployed_engineering_guide.md` | business case/TCO (0 dedicated), negotiation | **DENSE (L4)\* — business case/TCO L1** |

Counts are filename-verified (full `ls` of every directory, August 2026). "Mention-level only" rows name the exact topics the greps surfaced as present-but-undedicated — those are the raw material for §3.

### 2.1 Data Modeling — **DENSE** (≈16 dedicated guides)

| Evidence (dedicated guides) | Depth |
|---|---|
| `data_model_resource_book_guide.md` (758 L) — universal/industry data models, banking applications | Advanced |
| `nosql_data_modelling_guide.md` (913 L) — document, key-value, wide-column, graph modeling | Advanced |
| `data/data_vault_2_modeling.md` (589 L) — hubs/links/satellites, hash keys, vs. Kimball/Inmon | Advanced |
| `data/types_of_dimensions_data_warehousing.md`, `data/crm_data_warehouse_modelling.md`, `data/handling_duplicate_keys_data_warehousing.md` | Advanced (dimensional practice) |
| `neo4j_graph_database_guide.md` (781 L), `graph_engineering_guide.md`, `centrality_measures_guide.md` | Advanced (graph modeling) |
| `data/elasticsearch_data_modeling_schema_design.md` | Solid (search modeling) |
| `../banking/temenos_data_model_guide.md`, `../banking/fineract_database_models_guide.md`, `../banking/oracle_flexcube_data_model_guide.md`, `../banking/data_models_banking_insurance_guide.md`, `../banking/universal_banking_model_guide.md`, `../banking/banking_limits_domain_guide.md` | Advanced (banking models) |

Coverage notes: Kimball, Inmon and Data Vault 2.0 all have dedicated treatment; banking data models are unusually deep (Temenos, Flexcube, Fineract). **Thin sub-areas:** anchor modeling (mentioned only), model versioning/migration tooling (Liquibase/Flyway-class) as an operations topic, and FIBO/ACORD outside banking.

### 2.2 Data Architecture Patterns — **DENSE** (≈14 dedicated guides)

| Evidence | Depth |
|---|---|
| `data/data_mesh_architectures_guide.md` (928 L) — four principles, data products, self-serve platform, maturity model | Advanced |
| `data/data_fabric_guide.md` — active metadata, knowledge-graph fabric | Advanced |
| `data/enterprise_data_platforms_guide.md` (450 L) — platform landscape, evaluation, deployment patterns | Advanced |
| `cloud_object_storage_lakehouse_guide.md` (1,008 L), `data/on_prem_lakehouse_guide.md` (600 L) + `data/on_prem_lakehouse_analysis.md` | Advanced (lakehouse) |
| `data/delta_lake_vs_iceberg.md` (624 L), `data/paimon_iceberg_delta_comparison.md`, `databricks_guide.md` (807 L, lakehouse pattern) | Advanced (open table formats) |
| `data/mpp_databases_guide.md`, `data/alibaba_data_platform.md` | Solid (MPP/platform patterns) |
| `late_arriving_data_guide.md` (707 L), `schema_evolution_data_drift_guide.md`, `s3_architecture_guide.md` | Advanced (edge patterns) |

Coverage notes: mesh + fabric + lakehouse + medallion-adjacent patterns are all dedicated. **Thin sub-areas:** data contracts as a formal pattern (mentions in 9 data guides, no dedicated), and regulatory-reporting reference architecture beyond the BCBS 239 worked example inside `data_governance_guide.md`.

### 2.3 Data Integration — **DENSE** (≈13 dedicated guides)

| Evidence | Depth |
|---|---|
| `data_integration_frameworks_guide.md` (696 L) — patterns, CDC deep-dive (§6), data quality in integration (§7), batch-window optimization (§9), decision tree (§12) | Advanced |
| `data/data_pipeline_guide.md` (494 L) — ETL vs. ELT, pipeline lifecycle; `data/data_pipeline_versioning.md`, `data/backfill_data_engineering.md` | Advanced |
| `legacy_integration_patterns_guide.md`, `message_queue_data_loss_guide.md` (703 L), `apache_seata_guide.md` (distributed transactions), `temporal_workflow_guide.md` | Advanced |
| File-transfer/batch stack: `axway_transfer_cft_guide.md`, `axway_cft_controlm_integration.md`, `control_m_guide.md`, `control_m_external_conditions_guide.md`, `control_m_migration_options_guide.md` | Solid–Advanced (banking batch reality) |

Coverage notes: this axis reflects a CIB reality — deep on batch/file-transfer (Control-M, Axway CFT) and CDC-as-a-section. **Thin sub-areas:** transformation-layer tooling (**dbt** — mentions only), Airflow-class orchestration (mentions only), iPaaS and SaaS integration, and SWIFT/ISO 20022 integration depth (the ISO 20022 guide lives in `../banking/iso_20022_core_processes_guide.md` and is process-focused, not pipeline-focused).

### 2.4 Data Platforms — **DENSE** (≈18 dedicated guides, with one product-level thin spot)

| Evidence | Depth |
|---|---|
| `databricks_guide.md` (807 L), `oracle_database_guide.md` (726 L), `oracle_sharding_guide.md`, `oracle_instant_client_versions_guide.md`, `polardb_vs_oceanbase_guide.md` | Advanced |
| `data/mpp_databases_guide.md`, `data/enterprise_data_platforms_guide.md`, `data/on_prem_lakehouse_guide.md` | Advanced |
| `neo4j_complete_guide.md`, `neo4j_graph_database_guide.md`, `neo4j_bloom_alternatives_guide.md`, `ai_llm/rag/vector_databases_guide.md` (933 L) | Advanced (graph + vector) |
| Object storage: `s3_architecture_guide.md`, `storagegrid_guide.md`, `dell_objectscale_guide.md`, `cephfs_alternatives_guide.md` | Advanced |
| Domain engines: `jbase_universe_guide.md`, `jbase_vs_infobasic_guide.md`, `ibm_as400_guide.md` | Solid (Temenos/legacy DBs) |

Coverage notes: on-prem and open-source engines are deeply covered. **The thin spot is real and important: there is no dedicated Snowflake, BigQuery, Redshift, or Synapse guide.** Snowflake is discussed inside `databricks_guide.md` (20 mentions), `data/enterprise_data_platforms_guide.md` (67 mentions) and `data/mpp_databases_guide.md` (24 mentions) — comparison-level knowledge, not product-level architecture. No dedicated PostgreSQL/MySQL/MongoDB/Cassandra guides either (all appear as mentions across banking guides).

### 2.5 Streaming — **thin** (2 dedicated + 4 adjacent; strong concepts, thin operations)

| Evidence | Depth |
|---|---|
| `event_stream_processing_guide.md` (1,218 L) — ESP patterns, time semantics/windowing, state & fault tolerance, banking ESP (§9), CEP in banking (§10) | Advanced (conceptual) |
| `complex_event_processing_guide.md`, `message_queue_data_loss_guide.md` (703 L) | Solid |
| `kafka_alternatives_guide.md` (903 L) — Kafka ecosystem comparison | Solid (comparison, not internals) |
| Adjacent: `apache_seata_guide.md` (distributed tx), `ai_llm/rag/rag_with_data_streaming_guide.md`, `temporal_workflow_guide.md` | Solid |

Coverage notes: **the concepts are strong; the operational/product layer is thin.** There is no dedicated Kafka-internals guide (only `kafka_alternatives_guide.md`), no Flink or Kafka Streams dedicated guide (zero files match `*flink*`), no schema-registry/Confluent guide, no dedicated CDC-to-streaming operations guide (CDC exists as §6 of `data_integration_frameworks_guide.md`), and no dedicated exactly-once/checkpointing operations guide (covered as sections of `event_stream_processing_guide.md`).

### 2.6 Governance — **DENSE** (≈13 dedicated guides)

| Evidence | Depth |
|---|---|
| `data_governance_guide.md` (944 L) — frameworks, operating model, data quality (§6), metadata (§7), MDM (§8), platform governance (§9), banking/regulatory (§10), **worked BCBS 239 example (§12)** | Advanced |
| `data/data_compliance_frameworks.md` (1,530 L) — 14+ privacy frameworks incl. **GDPR (§a) and PDPA (§b)**, cross-border transfers, implementation guide | Advanced |
| `data/data_governance_framework.md`, `data/china_data_governance_frameworks.md`, `data/data_lineage_tools.md`, `data/data_profiling_guide.md`, `data/dataops_guide.md` | Advanced |
| `api_governance_guide.md`, `audit_as_code_guide.md`, `ai_llm/ai_governance_bias_redteaming_guide.md`, `ai_llm/implementing-responsible-ai.md` | Solid–Advanced |
| `../banking/financial_risk_compliance_systems_guide.md`, `../banking/risk_management_models_guide.md` | Advanced (banking context) |

Coverage notes: governance is the strongest non-AI axis in the profile. **Thin sub-areas:** DAMA-DMBOK as a formal framework (the acronym appears in ~58 files but there is no dedicated DMBOK guide or certification-aligned study path), data-catalog tooling (Collibra/DataHub/OpenMetadata — zero dedicated guides), DQ measurement metrics/SLOs operationalization (section-only), and data contracts (mentions only).

### 2.7 Security & Privacy — **thin** (13 security guides, but few data-specific; Zero Trust = gap)

| Evidence | Depth |
|---|---|
| `distributed_auth_guide.md`, `openbao_vs_vault_guide.md` (secrets management) | Solid–Advanced |
| `openscap_guide.md`, `penetration_testing_execution_standard_guide.md`, `osint_guide.md`, `sbom_c_proc_cobol.md`, `vuln_scanning_c_proc_cobol.md`, `ssh_key_disabled_user_guide.md`, `container_certificates_guide.md` | Solid (infra security) |
| `adversarial_ml_attacks_guide.md`, `ai_llm/prompt_injection_guide.md`, `llm_development_risks_security_guide.md`, `ai_llm/ai_governance_bias_redteaming_guide.md` | Advanced (AI security) |
| Privacy law: `data/data_compliance_frameworks.md` (GDPR, PDPA deep-dives) | Advanced |

Coverage notes: the profile is **strong on infrastructure security and AI security, thin on data-plane security**. No dedicated guide covers encryption-at-rest/in-transit for data platforms, KMS/HSM, tokenization/masking, row/column-level security, dynamic data masking, or secure data sharing/clean rooms (these appear as sections inside `databricks_guide.md` §7 and `data_governance_guide.md`). **Zero Trust: gap** — "zero trust" appears in only 5 files, always as a mention; there is no NIST SP 800-207 guide and no Zero-Trust-for-data-plane treatment (standing priority, confirmed).

### 2.8 Cloud Data — **thin** (infra DENSE, managed data services thin)

| Evidence | Depth |
|---|---|
| `cloud_providers_guide.md` (1,196 L), `singapore_data_centres_guide.md` (data centres + sovereignty), `aws_sap_c02_guide.md`, `finops_guide.md` | Advanced (cloud infra + cost) |
| `cloud_object_storage_lakehouse_guide.md` (1,008 L), `s3_architecture_guide.md` | Advanced (object storage) |
| `databricks_guide.md` (§4 "Databricks on the Three Clouds"), `data/alibaba_data_platform.md`, `data/enterprise_data_platforms_guide.md` | Solid (multi-cloud data) |

Coverage notes: cloud *infrastructure* and *object storage* are deeply covered; **managed cloud data services are thin** — no dedicated Redshift/BigQuery/Synapse/Glue/Dataflow/ADF guides, no cloud data migration playbook (lift-and-shift vs. re-platform vs. re-architect with cutover patterns), no data-residency/multi-region architecture guide (MAS outsourcing is covered only inside banking guides).

### 2.9 ML Data — **DENSE** (≈10 dedicated guides; training-data ops thin)

| Evidence | Depth |
|---|---|
| `feature_store_guide.md` (674 L) — online/offline stores, point-in-time correctness, validation | Advanced |
| `ai_llm/rag/vector_databases_guide.md` (933 L) + 18 RAG guides (`ai_llm/rag/*`) incl. `ai_llm/rag/rag_with_data_streaming_guide.md`, `ai_llm/rag/rag_evaluation_methodology_guide.md` | Advanced (RAG data architecture) |
| `ai_llm/closed_loop_data_engineering_guide.md` — feedback → retraining loops | Advanced |
| `schema_evolution_data_drift_guide.md`, `drift_detection_methods_guide.md` | Advanced (ML data quality) |
| `ml_platforms_comparison_guide.md`, `mlops_lifecycle_frameworks_guide.md`, `ml_system_design_interview_guide.md`, `customer_lifetime_value_prediction.md`, `advanced_analytics_solutions_guide.md` | Solid–Advanced |

Coverage notes: for an AI-era data architect this axis is ahead of the market — feature stores, vector DBs, RAG pipelines and drift are all dedicated. **Thin sub-areas:** training-data lifecycle (labeling, curation, versioning, data-centric AI), eval-data governance, and ML-specific lineage/versioning at scale (partial via `ai_llm/closed_loop_data_engineering_guide.md`).

### 2.10 Business & Communication — **DENSE** (≈15 guides; business-case/TCO = gap)

| Evidence | Depth |
|---|---|
| `storytelling_skills_guide.md` (442 L), `../management/communication_stakeholder_management_skills_guide.md`, `../management/managing_up_down_sideways_guide.md` (525 L), `ice_breaking_guide.md`, `toastmasters_guide.md` | Advanced (communication) |
| `../management/mckinsey_approach_guide.md`, `../management/management_consulting_skills_guide.md`, `../management/ss680_management_consultants_guide.md` | Advanced (consulting method) |
| `../management/vendor_management_guide.md`, `../management/mba_body_of_knowledge_guide.md`, `../management/analytics_management_guide.md`, `product_thinking_guide.md`, `../management/forward_deployed_engineering_guide.md`, `first_time_manager_guide.md`, `../management/time_energy_management_guide.md` | Solid–Advanced |

Coverage notes: soft skills and consulting method are unusually strong for a technical profile. **The gap is financial framing: business case development and TCO modeling have no dedicated guide.** `finops_guide.md` covers cloud cost governance (showback/chargeback, commitments) but not project business cases (NPV/IRR/payback, build-vs-buy, TCO models, sensitivity analysis) — standing priority, confirmed.

### Heatmap summary table

| # | Axis | Score | Dedicated guides | One-line evidence |
|---|---|---|---|---|
| 1 | Data Modeling | **DENSE** | ~16 | Kimball + Data Vault 2.0 + NoSQL + graph + 6 banking model guides |
| 2 | Data Architecture Patterns | **DENSE** | ~14 | mesh, fabric, lakehouse, Delta vs. Iceberg, MPP |
| 3 | Data Integration | **DENSE** | ~13 | patterns + CDC deep-dive + Control-M/CFT batch stack |
| 4 | Data Platforms | **DENSE*** | ~18 | Databricks, Oracle, Neo4j, vector DBs; *no Snowflake/BigQuery/Redshift dedicated |
| 5 | Streaming | **thin** | 2+4 | concepts deep (1,218 L), no Kafka/Flink/schema-registry dedicated |
| 6 | Governance | **DENSE** | ~13 | 944 L governance + 1,530 L compliance (GDPR, PDPA) + BCBS 239 worked example |
| 7 | Security & Privacy | **thin** | 13 (few data-specific) | infra/AI security strong; data-plane security section-only; **Zero Trust = gap** |
| 8 | Cloud Data | **thin** | ~6 (infra-heavy) | cloud infra DENSE; managed DW/migration/sovereignty thin |
| 9 | ML Data | **DENSE** | ~10 | feature store, vector DB, RAG ×18, drift, closed-loop |
| 10 | Business & Communication | **DENSE*** | ~15 | consulting/communication strong; **business case/TCO = gap** |


---

## 3. Gap Analysis

This section names the specific missing knowledge for every thin or gap axis. The distinction matters: "thin" means the repo has solid section-level or adjacent coverage but no dedicated depth; "gap" means the topic appears only as mentions or not at all. Each entry states what a data architect needs that the profile does not currently evidence.

### 3.1 Streaming — thin (concepts strong, operations thin)

What is covered well: ESP semantics (windowing, watermarks, time handling), CEP patterns, state and fault-tolerance concepts, Kafka ecosystem comparison, message-queue data-loss failure modes, streaming RAG.

Specific missing knowledge:
- **Kafka internals**: log segments, ISR/acks semantics, partition assignment, consumer-group rebalancing, compaction, tiered storage, KRaft vs. ZooKeeper, sizing (partitions, replicas, retention), and the operational failure modes of each. `kafka_alternatives_guide.md` compares the ecosystem but assumes Kafka knowledge rather than building it.
- **Stream-processing engines in practice**: Flink (checkpointing, savepoints, state backends, exactly-once via two-phase commit), Kafka Streams (KTable/KStream, interactive queries) — zero dedicated guides; the concepts in `event_stream_processing_guide.md` are engine-agnostic.
- **Schema registry & contract evolution**: Avro/Protobuf schemas, compatibility rules (backward/forward/full), breaking-change management in a streaming bus.
- **CDC-to-stream pipelines operationally**: Debezium connector configs, log-based CDC vs. query-based, incremental snapshots, CDC → Kafka → Flink pipeline design. CDC exists as §6 of `data_integration_frameworks_guide.md` — strong conceptually, but no operational playbook.
- **Streaming in the bank**: replay/backfill of streams, exactly-once end-to-end, latency SLOs, stream observability. `event_stream_processing_guide.md` §9 covers banking use cases conceptually.

### 3.2 Cloud Data — thin (infra dense, managed data services thin)

What is covered well: cloud provider comparison, object storage and S3 architecture, lakehouse on cloud, Databricks across three clouds, data-centre landscape in Singapore, FinOps cost governance.

Specific missing knowledge:
- **Managed cloud warehouses product-level**: Snowflake (virtual warehouses, micro-partitions, cloning/time-travel, data sharing, cost controls), BigQuery (slot model, partitioning/clustering, reservations), Redshift (RA3, distribution styles, sort keys, workload management), Synapse (dedicated vs. serverless SQL). All three are referenced as competitors in `databricks_guide.md` and `data/enterprise_data_platforms_guide.md` but none has a dedicated guide — in a 2026 data-architect assessment this is the most probable product question.
- **Cloud data migration**: lift-and-shift vs. re-platform vs. re-architect decision frameworks, dual-run and cutover strategies, migration of on-prem Teradata/Oracle/Exadata estates (a real CACIB-style problem), data validation and reconciliation during migration.
- **Data residency & sovereignty architecture**: multi-region replication, egress costs, MAS outsourcing/cloud notice constraints, China data governance (a dedicated `data/china_data_governance_frameworks.md` exists — but no architecture-level residency pattern guide).
- **Serverless data services**: Glue/Dataflow/ADF pipelines, serverless SQL, event-driven ingestion (S3 events → Lambda/Cloud Functions).

### 3.3 Security & Privacy — thin (infra/AI security strong, data-plane security thin; Zero Trust gap)

What is covered well: distributed auth, secrets management (OpenBao/Vault), AI security (adversarial ML, prompt injection, LLM risk), compliance scanning/SBOM, privacy law (GDPR + PDPA deep-dives in `data/data_compliance_frameworks.md`).

Specific missing knowledge:
- **Data-plane security engineering**: encryption at rest/in transit for warehouses and lakes, KMS/HSM key hierarchies (envelope encryption), tokenization vs. masking vs. pseudonymization and when each is used in a bank, dynamic data masking, row-level security and column-level security patterns, secure data sharing (Delta Sharing is a section in `databricks_guide.md`; needs its own depth).
- **Zero Trust for data (standing priority)**: NIST SP 800-207 pillars (identity, device, network, application, data), micro-segmentation of the data plane, identity-aware proxies for data services, continuous verification, ZTA in banking (MAS TRM alignment, MAS Notice 645). Currently: "zero trust" appears in 5 files as a mention; **no dedicated guide exists**.
- **Privacy engineering beyond law**: privacy-by-design, DPIA practice, purpose limitation in data platforms, cross-border transfer mechanisms applied to a lakehouse (SCCs, DPF) — the law is covered in `data/data_compliance_frameworks.md` §3; the *architecture* of compliance (where transfers physically occur, what controls sit where) is not.

### 3.4 Governance — DENSE, but three named gaps at the operational layer

What is covered well: governance frameworks, operating models, DQ sections, metadata sections, MDM sections, lineage tools, profiling, compliance law, BCBS 239 worked example.

Specific missing knowledge:
- **DAMA-DMBOK formal structure**: the 11 knowledge areas and the "DAMA wheel", DMBOK 1 vs 2 evolution, and the CDMP certification exam structure. The acronym appears across ~58 files but there is no dedicated DMBOK guide; a data architect is expected to speak DMBOK fluently in governance conversations and job interviews.
- **Data catalog & metadata tooling**: Collibra vs. DataHub vs. OpenMetadata vs. Alation — deployment, business glossary, automated lineage harvesting, catalog-driven mesh. `data/data_lineage_tools.md` covers lineage tools, but catalog platforms as a product category have no dedicated guide.
- **Data quality operationalization**: DQ dimensions (completeness, uniqueness, timeliness, validity, accuracy, consistency), measurable DQ metrics and SLOs, DQ rules engines, remediation workflows, data observability (Great Expectations / dbt tests / elementary), data contracts as an enforcement mechanism. All appear as sections of `data_governance_guide.md` §6 and `data/dataops_guide.md`; no dedicated guide.
- **MDM implementation patterns**: hub/registry/coexistence styles, party/customer master, product master, security/reference data management, MDM in core-banking integration (Temenos/Flexcube contexts exist in banking guides but MDM-as-a-discipline has no dedicated guide).

### 3.5 Data Platforms — DENSE with one product-level thin spot

Missing knowledge (single but high-impact): **cloud data warehouse platforms at product level** — see §3.2. Everything else on this axis is DENSE; the gap is specifically the managed-cloud-DW product layer, which is where modern data-architecture assessments spend the most time.

### 3.6 ML Data — DENSE with two thin corners

Missing knowledge:
- **Training-data lifecycle**: labeling workflows and tooling, data curation and deduplication, dataset versioning, data-centric AI methodology, eval-set governance. The profile covers the *serving* side (features, vectors, RAG, drift) excellently; the *training* side is thinner.
- **ML data lineage & reproducibility at scale**: experiment-to-data linkage, feature lineage (partially in `feature_store_guide.md`), model-card data provenance.

### 3.7 Business & Communication — DENSE, one named gap (standing priority)

Missing knowledge: **business case development and TCO modeling for data initiatives**. `finops_guide.md` handles cloud *operating* cost; nothing covers project-level financial framing: cost-benefit analysis, NPV/IRR/payback, build-vs-buy quantification, 3–5-year TCO models (Gartner/Forrester style), sensitivity analysis, and the narrative of "what is this data platform worth to the bank". This is the standing business-case/TCO priority and it is confirmed absent.

### 3.8 Data Modeling / Integration / Patterns — DENSE, minor refinements only

- Anchor modeling (mentioned, not dedicated) — low priority.
- Model migration tooling (Liquibase/Flyway-class) — low priority.
- dbt transformation layer — mention-level; worth folding into the DQ/observability gap rather than a standalone gap.
- SWIFT/ISO 20022 integration from a pipeline perspective — covered process-side in `../banking/iso_20022_core_processes_guide.md`; pipeline-side integration of MT/MX flows is a minor addition.

---

## 4. Prioritized Skill-Gap List (Top 10, Ranked by Data-Architect Impact)

Ranking logic: (a) frequency and difficulty of the topic in real data-architect assessments and in CACIB-style enterprise data work, (b) whether the gap is a standing priority, (c) leverage — one guide covering many interview/assessment situations. **Standing priorities (Zero Trust, Platform Engineering, business case/TCO) are items #2, #3, #4** — each verified as a real gap by this scan.

| # | Gap | Axis | Why it ranks here (rationale) | Status |
|---|---|---|---|---|
| 1 | **Managed cloud data warehouse platforms** (Snowflake, BigQuery, Redshift, Synapse) | Cloud Data / Data Platforms | The #1 product-level topic in 2026 data-architecture interviews and modernization assessments; profile has comparison-level mentions only (67 in `data/enterprise_data_platforms_guide.md`) and zero dedicated guides. Every bank data program is cloud-DW-or-lakehouse; being unable to argue Snowflake vs. Databricks vs. BigQuery at product depth caps interview performance. | thin → **commission guide** |
| 2 | **Business case & TCO modeling for data initiatives** | Business & Communication | Standing priority. Data architecture decisions (platform choice, migration, build-vs-buy) are funded through business cases; the profile has consulting/communication depth but no financial-framing depth. `finops_guide.md` covers operating cost only. | gap → **commission guide** |
| 3 | **Zero Trust for data platforms** | Security & Privacy | Standing priority. NIST SP 800-207 + data-plane ZTA (micro-segmentation, identity-aware data access, MAS TRM/N645 alignment). Only 5 mention-level hits in the whole repo. High regulatory salience in Singapore banking. | gap → **commission guide** |
| 4 | **Platform Engineering for data** | Data Architecture Patterns / Platforms | Standing priority. The self-serve data platform layer (IDP for data, golden paths for pipelines, data-product platform, mesh enablement) — the missing operational half of the excellent `data/data_mesh_architectures_guide.md`. ~37 mention-level hits, no dedicated guide. | gap → **commission guide** |
| 5 | **Streaming operations depth** (Kafka internals, Flink, schema registry, CDC→stream ops) | Streaming | Concepts are DENSE (1,218-line ESP guide); operations are thin. Interviewers probe Kafka/Flink specifics; the profile would currently answer conceptually, not operationally. | thin → **commission guide** |
| 6 | **Data quality & data observability operationalization** (DQ dimensions, metrics/SLOs, dbt, Great Expectations, data contracts) | Governance | Governance is DENSE but DQ is section-level only; DQ is the most-asked governance question in practice ("how do you measure data quality?") and data contracts are the 2026 hot pattern. | thin → **commission guide** (or fold into a DMBOK guide) |
| 7 | **DAMA-DMBOK formal framework & CDMP alignment** | Governance | Named in ~58 files but never studied as a framework; DMBOK fluency is expected in governance roles and is the standard certification (CDMP). Also anchors the governance story in interviews. | thin → **commission guide** |
| 8 | **Data catalog & metadata tooling** (Collibra, DataHub, OpenMetadata, Alation) | Governance | Lineage tools are covered (`data/data_lineage_tools.md`); catalog platforms as a product category are not. Catalogs are the operating system of data mesh and modern governance. | gap → **commission guide** |
| 9 | **Master data management implementation patterns** (hub/registry/coexistence; party/product/security master; core-banking MDM) | Governance / Data Integration | Covered only as §8 of `data_governance_guide.md`; MDM is a distinct discipline with its own architecture patterns and is a classic data-architect assignment in banking. | thin → **commission guide** |
| 10 | **ML training-data lifecycle** (labeling, curation, versioning, data-centric AI, eval data) | ML Data | ML Data is DENSE on serving (features, vectors, RAG, drift); the training-data side is thin. Rising importance as GenAI programs industrialize data curation. | thin → **optional guide** |

### 4.1 Gap-to-impact matrix (why these ten, in this order)

| Gap | Interview frequency (2026 assessments) | CACIB work relevance | Effort to close | Net priority |
|---|---|---|---|---|
| 1. Cloud DW platforms | Very high (product question) | High (cloud modernization) | High (4–6 wks) | **1** |
| 2. Business case/TCO | High (senior roles) | High (funding every initiative) | Medium (2–3 wks) | **2** |
| 3. Zero Trust for data | High (security question) | Very high (MAS TRM/N645) | Medium (2–3 wks) | **3** |
| 4. Platform Engineering for data | Medium-high (emerging) | High (mesh enablement) | Medium (3–4 wks) | **4** |
| 5. Streaming operations | High (Kafka/Flink probes) | High (fraud, intraday liq.) | Medium-high (4–5 wks) | **5** |
| 6. DQ & observability | Very high (governance question) | Very high (BCBS 239 DQ) | Low (2–3 wks) | **6** |
| 7. DAMA-DMBOK | Medium (framing) | Medium (governance fluency) | Low (1–4 wks) | **7** |
| 8. Catalog tooling | Medium | High (lineage at scale) | Low (2 wks) | **8** |
| 9. MDM patterns | Medium | High (party/security master) | Low (2 wks) | **9** |
| 10. ML training data | Medium (AI-era) | Medium-high (GenAI programs) | Low (2 wks) | **10** |

The three standing priorities (#2, #3, #4) sit in the top four — the prior-session signal and this scan agree.

**Deliberately not prioritized** (present in the profile at sufficient depth, or low leverage): dimensional modeling (DENSE), Data Vault (DENSE), data mesh/fabric (DENSE), lakehouse formats (DENSE), compliance law incl. GDPR/PDPA (DENSE), cloud infra/FinOps (DENSE), feature stores/vector DBs/RAG (DENSE), communication/consulting (DENSE), data lineage tooling (covered), graph databases (DENSE), NoSQL modeling (DENSE).

---

## 5. Learning Plan

Per gap: resources (books/courses/docs), effort estimate (weeks of focused study at his demonstrated pace — the repo's guides run 450–1,500 lines each, so "1 companion guide" ≈ 1–2 focused weeks), and whether a companion guide should be commissioned following the repo's own pattern. The repo's pattern: a 400–900-line, TOC-structured, banking-contextualized guide authored by "Generated Research / Jack Liu Shurui". Commissioning is recommended where the topic is broad and interview-critical; folding into existing guides is recommended where the topic is a section-sized extension.

### Gap #1 — Managed cloud data warehouses
- **Resources:** Snowflake docs (key concepts: virtual warehouses, micro-partitions, time travel, data sharing, Snowpipe), BigQuery docs (slots, partitioning/clustering, BI engine, reservations), Redshift docs (RA3, AQUA, distribution/sort), "Snowflake: The Definitive Guide" (O'Reilly), "Google BigQuery: The Definitive Guide" (Lakshmanan), each platform's "vs. Databricks" pages, cloud provider data-warehouse comparison pages; practice: spin up free tiers and run a small TPC-H-style benchmark.
- **Effort:** 4–6 weeks (3 platforms × ~2 weeks, or one deep + two comparative).
- **Companion guide:** **Yes — commission `cloud_data_warehouse_platforms_guide.md`** (~700 L): per-platform architecture, cost models, migration paths from on-prem MPP (Teradata/Exadata), decision tree, banking use cases, TCO angle (ties into gap #2).

### Gap #2 — Business case & TCO modeling (standing priority)
- **Resources:** "Business Case Analysis with NPV" (Harvard Business School-style cases), Gartner TCO methodology write-ups, "Cloud FinOps" (O'Reilly), AWS/Azure TCO calculators and their methodology notes, "The Business Case for Data Architecture" articles from TDWI, cost-benefit analysis chapter of any good project-finance text; McKinsey/BCG casebooks for framing (profile already has `../management/mckinsey_approach_guide.md` as a base).
- **Effort:** 2–3 weeks to be fluent; 1 week for the guide.
- **Companion guide:** **Yes — commission `data_business_case_tco_guide.md`** (~500 L): CBA structure, NPV/IRR/payback worked examples, build-vs-buy quantification, 3–5-year TCO models for warehouse/lakehouse/mesh, sensitivity analysis, exec narrative. Folds directly into gap #1's TCO angle.

### Gap #3 — Zero Trust for data (standing priority)
- **Resources:** NIST SP 800-207 (Zero Trust Architecture), CISA ZTMM, "Zero Trust Architecture" vendor papers (BeyondCorp/BeyondProd originals), MAS TRM and MAS Notice 645 (Singapore context), NIST 800-53 data-security controls (AC, SC families).
- **Effort:** 2–3 weeks.
- **Companion guide:** **Yes — commission `zero_trust_data_platform_guide.md`** (~550 L): the 7 tenets applied to data planes, identity-aware access to warehouses/lakes, micro-segmentation, encryption/KMS, continuous verification, banking + MAS alignment, worked reference architecture.

### Gap #4 — Platform Engineering for data (standing priority)
- **Resources:** "Team Topologies" (Skelton & Pais), "Platform Engineering" (O'Reilly, Camille Fournier ed.), Backstage/IDP docs, "Data Platform in Action" (Fruhlinger et al.) and its referenced architecture, the platform chapter of the profile's own `data/data_mesh_architectures_guide.md`, CNCF platform engineering whitepaper.
- **Effort:** 3–4 weeks.
- **Companion guide:** **Yes — commission `data_platform_engineering_guide.md`** (~600 L): IDP-for-data, golden paths for pipelines, self-serve ingestion/transformation, data product templates, platform maturity model, mesh enablement, banking DX context. This is the natural sequel to `data/data_mesh_architectures_guide.md`.

### Gap #5 — Streaming operations
- **Resources:** "Kafka: The Definitive Guide" (Shapira et al.), Confluent docs (schema registry, compatibility), "Stream Processing with Apache Flink" (Hienz & Katsifodimos), `event_stream_processing_guide.md` (already in profile, use as the conceptual spine), Debezium docs for CDC ops.
- **Effort:** 4–5 weeks (Kafka 2–3, Flink 1–2).
- **Companion guide:** **Yes — commission `streaming_platform_operations_guide.md`** (~650 L): Kafka internals and ops, Flink/Kafka Streams, schema registry, CDC→stream pipelines (Debezium), exactly-once end-to-end, streaming SLOs/observability, banking real-time use cases. Complements the existing conceptual guide rather than replacing it.

### Gap #6 — Data quality & observability operationalization
- **Resources:** DAMA-DMBOK DQ chapter, "Data Quality: Dimensions, Metrics" (Sebastian-Coleman's "Metrics for Data Quality"), Great Expectations / dbt / elementary docs, data contract pattern references (data-contract specification), "Data Observability" literature (Monte Carlo/Metaplane).
- **Effort:** 2–3 weeks.
- **Companion guide:** **Yes (fold option):** either a dedicated `data_quality_observability_guide.md` (~500 L) or a strong new chapter folded into a future DMBOK guide (gap #7). Recommend the dedicated guide — DQ is the most quizzed governance topic.

### Gap #7 — DAMA-DMBOK formal
- **Resources:** DAMA-DMBOK 2nd Edition (the book), the DAMA wheel/11 knowledge areas, CDMP exam blueprint and sample questions, TDWI DMBOK summaries.
- **Effort:** 3–4 weeks part-time; 1 week if focused on the exam outline only.
- **Companion guide:** **Yes — commission `dama_dmbok_guide.md`** (~600 L): 11 knowledge areas mapped to the profile's existing coverage (governance, modeling, integration, warehousing already DENSE), BCBS 239/MAS alignment, CDMP exam map. High synergy: it will *reference* existing guides rather than duplicate them.

### Gap #8 — Data catalog & metadata tooling
- **Resources:** OpenMetadata docs, DataHub docs, Collibra/Alation marketing-to-architecture deep dives, `data/data_lineage_tools.md` (already in profile — extend), catalog-driven mesh case studies.
- **Effort:** 2 weeks.
- **Companion guide:** **Optional — commission `data_catalog_platforms_guide.md`** (~450 L) or fold into gap #7's DMBOK guide as a tooling chapter. Recommend folding to keep guide count focused.

### Gap #9 — MDM implementation patterns
- **Resources:** "Master Data Management" (DAMA/DGI white papers), hub/registry/coexistence patterns, party/master in Temenos/Flexcube (profile has the model guides to build on), MDM in KYC/AML context (MAS 626 alignment), Informatica/Reltio product architecture docs.
- **Effort:** 2 weeks.
- **Companion guide:** **Optional — commission `master_data_management_guide.md`** (~500 L) — high banking relevance (party/security master), or fold as an MDM chapter into the DMBOK guide. Recommend dedicated only if the bank work demands it.

### Gap #10 — ML training-data lifecycle
- **Resources:** "Data-Centric AI" (MIT/Andrew Ng materials), Snorkel/Label Studio docs, dataset versioning (DVC/HuggingFace datasets), `ai_llm/closed_loop_data_engineering_guide.md` (already in profile — extend), eval-data governance write-ups.
- **Effort:** 2 weeks.
- **Companion guide:** **Optional — commission `ml_training_data_lifecycle_guide.md`** (~450 L) or fold into the closed-loop guide. Lower urgency than #1–#7.

### Sequencing recommendation (9–12 months, parallelizable with day job)

| Window | Focus | Deliverable |
|---|---|---|
| Months 1–2 | Cloud DW platforms (gap #1) + business case/TCO (gap #2) | Two commissioned guides; TCO feeds the DW guide |
| Months 3–4 | Zero Trust for data (gap #3) + Platform Engineering for data (gap #4) | Two commissioned guides (both standing priorities) |
| Months 5–6 | Streaming operations (gap #5) + DQ/observability (gap #6) | Two commissioned guides |
| Months 7–8 | DAMA-DMBOK (gap #7) + catalog/MDM (gaps #8–9, folded or dedicated) | One guide + CDMP exam attempt if desired |
| Months 9–10 | ML training-data (gap #10) + interview rehearsal with §6 scenario bank | Optional guide + mock assessments |
| Ongoing | Apply: volunteer for CACIB data-platform/data-governance initiatives to convert knowledge into evidence | Work artifacts that mirror the guides |


---

## 6. Worked Example — Data-Architecture Assessment End-to-End

To make the framework and gap analysis concrete, here is a full data-architect interview/assessment scenario worked end-to-end. It is calibrated to Jack's world: a CIB in Singapore. The scenario is generic enough to be reused as a rehearsal template, and each section shows where the profile's existing guides supply the answer and where the gaps in §4 would surface.

### 6.1 The scenario

> **Prompt (from a hiring panel, ~45 minutes):** "Our regional bank runs regulatory and management reporting on a 20-year-old on-prem Teradata estate fed by overnight batch loads from core banking, payments, and risk systems. The board wants a modern, cloud-ready data platform: near-real-time ingestion for fraud and intraday-liquidity use cases, a governed lakehouse for analytics and AI, and full BCBS 239 / MAS 626 compliance. We've been told to also evaluate Snowflake. Design the target data architecture and tell us how you would sequence the migration."

### 6.2 Step 1 — Clarify before designing (the 5-minute opening that separates seniors)

Strong candidates ask, in order:
1. **Scope and drivers:** which reporting domains are in scope (credit risk, market risk, liquidity, finance)? Is this a greenfield replacement or incremental (BCBS 239 remediation-style)? What is the board's trigger — cost, latency, regulation, AI ambitions?
2. **Sources and volumes:** which source systems (Temenos/Flexcube core, Murex, payments hub, trade finance), row/volume estimates, peak batch windows, whether source APIs or only DB-level access (CDC feasibility) exist.
3. **Latency and SLAs:** which use cases truly need near-real-time (fraud is minutes; intraday liquidity can be 15-minute intervals) vs. which are happily batch — the honest answer is usually "only 2–3 use cases need streaming."
4. **Governance reality:** who owns data domains today, what the DQ state is, whether a CDO function exists, current lineage tooling.
5. **Constraints:** cloud residency (MAS outsourcing, data in Singapore), budget envelope and 3-year TCO expectations, team skills (can they run Kafka/Flink?), vendor preferences.

*Repo mapping:* the clarification playbook is exactly the consulting-method material in `../management/mckinsey_approach_guide.md` and `../management/communication_stakeholder_management_skills_guide.md`; the domain framing comes from `../banking/financial_risk_compliance_systems_guide.md`, `../banking/core_banking_systems_guide.md`, and `event_stream_processing_guide.md` §9.

### 6.3 Step 2 — Target architecture (the 25-minute core)

**A. Ingestion & integration (near-real-time + batch coexistence)**
- Keep the existing Control-M/Axway CFT batch stack for legacy file feeds during migration (profile: `control_m_guide.md`, `axway_transfer_cft_guide.md`, `legacy_integration_patterns_guide.md`).
- Add log-based CDC on core-banking DBs (Debezium or GoldenGate-class) for the 3 streaming use cases: fraud, intraday liquidity, and risk-position delta feeds (profile: §6 CDC deep-dive in `data_integration_frameworks_guide.md`; operational gap from §3.1).
- Publish CDC events to Kafka with a schema registry (Avro) so consumers can evolve independently — this is where gap #5 (schema registry ops) would be probed.
- Two pipeline tiers: stream (Kafka → Flink for enrichment/aggregation) and batch (ELT into the lake for everything else). State explicitly: **this is a hybrid lambda-style architecture, deliberately** — the bank has no use case that justifies full kappa (profile: `data/data_pipeline_guide.md`, `event_stream_processing_guide.md`).

**B. Storage & processing (the lakehouse core)**
- Landing zone on object storage (S3-class, Singapore region for residency; profile: `cloud_object_storage_lakehouse_guide.md`, `s3_architecture_guide.md`, `singapore_data_centres_guide.md`).
- Bronze (raw, immutable, CDC-log-shaped) → silver (conformed, cleaned, Data-Vault-style hubs/links/satellites for auditability — profile: `data/data_vault_2_modeling.md`) → gold (dimensional marts for reporting; profile: `data/types_of_dimensions_data_warehousing.md`, `data/crm_data_warehouse_modelling.md`). Use Delta/Iceberg open formats to avoid lock-in (profile: `data/delta_lake_vs_iceberg.md`, `data/paimon_iceberg_delta_comparison.md`).
- Processing: Databricks-class lakehouse engine for ELT; explain **why Databricks over Snowflake here** (open formats, first-class Spark/Flink integration, Delta sharing, in-region deployment) **and concede the counter-argument** (Snowflake's operational simplicity, separation of compute/storage, data sharing, lower operational burden) — this is the exact conversation gap #1 exists for; a candidate with only comparison-level Snowflake knowledge will stall here.
- Late-arriving data and duplicate handling are first-class design points (profile: `late_arriving_data_guide.md`, `data/handling_duplicate_keys_data_warehousing.md`).

**C. Serving**
- Gold marts served to BI (existing reporting tools) via the warehouse/SQL engine; risk aggregation layer rebuilt against the Data-Vault-style silver for BCBS 239 lineage (profile: `data_governance_guide.md` §12 worked example).
- Feature store for fraud ML (online/offline consistency, point-in-time correctness — profile: `feature_store_guide.md`); vector/embeddings path for document and trade-surveillance AI (profile: `ai_llm/rag/vector_databases_guide.md`, RAG series).

**D. Governance & security (the differentiator section)**
- Data domains owned by business + data stewards; federated computational governance from the mesh playbook (profile: `data/data_mesh_architectures_guide.md`).
- Catalog + automated lineage at the top of the program — every BCBS 239 data lineage request is answered from the catalog, not by hand (profile: `data/data_lineage_tools.md`; tooling gap #8).
- Data quality SLOs with measured dimensions per critical data element (gap #6 — expect the follow-up "how exactly would you measure completeness and timeliness?").
- Security: KMS envelope encryption, row/column-level security on the lake, dynamic masking for non-production, **Zero Trust data-plane design** (identity-aware access to the lake, micro-segmentation, continuous verification — gap #3, standing priority), PDPA/GDPR controls from `data/data_compliance_frameworks.md`.
- MDM for party/security master feeding KYC and limits (gap #9; profile base: `../banking/banking_limits_domain_guide.md`).

**E. Cloud & TCO (the closing argument)**
- Single-region Singapore-first deployment for residency; egress-cost discipline; warehouse compute auto-scaling to control spend (profile: `finops_guide.md`).
- A 3-year TCO comparison: on-prem Teradata run-rate vs. lakehouse-on-cloud vs. Snowflake-managed, including migration cost, run cost, and the option value of AI use cases (gap #2 — the standing business-case priority; without it the recommendation lacks a funding story).

### 6.4 Step 3 — Migration sequencing (the pragmatic close)

1. **Phase 0 (0–3 mo):** catalog + lineage + DQ baselining on the existing estate; freeze new report builds. (Governance before plumbing — BCBS 239 wisdom; profile: `data_governance_guide.md` §12.)
2. **Phase 1 (3–9 mo):** stand up lakehouse landing + bronze/silver for 2–3 highest-value risk domains in parallel with Teradata; dual-run reporting with reconciliation (gap #2 TCO and gap #1 platform selection get finalized here).
3. **Phase 2 (9–15 mo):** CDC → Kafka → Flink for fraud/intraday-liquidity; cut over batch reporting domain-by-domain (Control-M schedules migrate gradually — profile: `control_m_migration_options_guide.md`).
4. **Phase 3 (15–24 mo):** decommission Teradata, enable self-serve data products and AI workloads; platform engineering layer (gap #4) makes the platform self-serve so the team doesn't become a bottleneck.

### 6.5 Evaluation rubric (how a panel scores this answer against the 10 axes)

| Axis | What a strong answer evidences | Score if answered from profile as-is |
|---|---|---|
| Data Modeling | Data-Vault silver + dimensional gold, SCD handling, late-arriving data | **9/10** — DENSE |
| Architecture Patterns | Mesh governance + lakehouse + explicit lambda rationale | **9/10** — DENSE |
| Data Integration | CDC vs. batch split, dual-run, Control-M migration | **8/10** — DENSE |
| Data Platforms | Databricks vs. Snowflake product-level argument | **5/10** — gap #1 caps this |
| Streaming | Kafka/Flink/schema-registry operational specifics | **5/10** — gap #5 caps this |
| Governance | BCBS 239 lineage, DQ SLOs, catalog-first | **7/10** — DQ/catalog specifics thin (gaps #6, #8) |
| Security & Privacy | PDPA/GDPR framing + data-plane ZTA | **5/10** — ZTA is gap #3 |
| Cloud Data | Residency, region, egress, migration sequencing | **6/10** — managed-DW migration thin (gap #1) |
| ML Data | Feature store + vector path for fraud/surveillance | **8/10** — DENSE |
| Business & Communication | Clear structure, trade-offs named, exec-friendly close | **8/10** — but the TCO close is the weak beat (gap #2) |

**Panel verdict pattern:** "Technically deep on modeling and governance; strong banking context; **needs product-level cloud-DW depth, streaming operations, and a sharper financial/TCO argument**" — which is precisely the prioritized list in §4. After the learning plan, the expected profile is 8–9/10 on all axes except a deliberate 7/10 on streaming ops (an honest, defensible position for an architect who leads rather than operates).

### 6.6 Common mistakes this scenario exposes (and the profile already protects against)

- Designing a full kappa architecture because "streaming is modern" — the profile's `data/data_pipeline_guide.md` and `event_stream_processing_guide.md` teach the hybrid discipline.
- Ignoring regulatory lineage until asked — the profile's BCBS 239 worked example pre-empts this.
- Recommending a platform without a funding story — the exact gap #2.
- No residency/sovereignty answer for Singapore — `singapore_data_centres_guide.md` and `data/data_compliance_frameworks.md` supply it.
- Hand-waving DQ measurement — gap #6; a panel will push on "define completeness for a risk data element."

### 6.7 Second worked scenario — the pure data-modeling assessment

> **Prompt:** "Model the lending domain for a digital bank: products, facilities, loans, limits, collateral, and exposures. The model must serve both a regulatory capital/limits engine and a BI layer. Walk us through your modeling choices."

**Expected senior answer structure:**

1. **Clarify the two consumers** — an operational limits/regulatory engine (needs transactional fidelity, auditability, performance at write) and a BI layer (needs query ergonomics, conformed dimensions). This immediately justifies a **two-model architecture** rather than one compromise model.
2. **Conceptual → logical:** identify core entities — Party (borrower/guarantor), Product (loan program), Facility (committed line), Loan/Contract (drawn amount), Collateral, Limit (type: regulatory, credit, concentration), Exposure (computed position). Draw the many-to-many Party↔Facility and Facility↔Limit relationships; flag Party as master data (gap #9 — the MDM conversation).
3. **Logical → physical for the operational side:** choose **Data Vault 2.0** — hubs (PARTY, PRODUCT, FACILITY, LOAN, COLLATERAL), links (FACILITY_LOAN, PARTY_FACILITY, LIMIT_FACILITY), satellites for attributes with history; hash keys for parallel loads from core-banking CDC; insert-only for audit. Justify vs. 3NF: source volatility (products change), audit requirements, multi-source integration (profile: `data/data_vault_2_modeling.md`).
4. **Logical → physical for the BI side:** star schema — FACT_EXPOSURE (additive measures: drawn, undrawn, exposure-at-default proxies; semi-additive: peak usage; non-additive: utilization ratio), dims: DIM_PARTY (SCD2 for credit-migration history), DIM_PRODUCT, DIM_FACILITY, DIM_DATE, DIM_COLLATERAL. Conformed DIM_PARTY shared with the risk mart (profile: `data/types_of_dimensions_data_warehousing.md`, `data/crm_data_warehouse_modelling.md`).
5. **Edge cases named unprompted** — late-arriving repayment data (`late_arriving_data_guide.md`), duplicate loan keys across core and payments systems (`data/handling_duplicate_keys_data_warehousing.md`), schema evolution of product attributes (`schema_evolution_data_drift_guide.md`), SCD handling for credit-grade changes.
6. **Governance tie-in** — every measure traceable to source via the vault satellite lineage; limits data quality SLOs (completeness of collateral valuation, timeliness of exposure) — the BCBS 239 angle (profile: `data_governance_guide.md` §12).

**Why this scenario matters for Jack:** it is the one assessment format where the profile is already at L5 — every element above maps to an existing dedicated guide. The panel follow-ups that would expose gaps are the same ones as §6.5: "how does this model perform on Snowflake vs. Teradata?" (gap #1), "what does the MDM hub look like?" (gap #9), "how do you measure the quality of exposure data?" (gap #6). **Strategy: use the modeling scenario as the confidence anchor, and prepare the three follow-up bridges explicitly.**

### 6.8 Follow-up question bank (with answer maps)

Ten questions a data-architect panel can ask after either scenario, with where the answer lives in the profile (or which gap it exposes):

| # | Follow-up question | Answer map (repo evidence → gap) |
|---|---|---|
| 1 | "How do you handle late-arriving facts in the exposure mart?" | `late_arriving_data_guide.md` (DENSE) |
| 2 | "What are the three DQ dimensions you would put SLAs on for risk data, and how do you measure them?" | `data_governance_guide.md` §6 + `data/data_profiling_guide.md` → **gap #6** for metric specifics |
| 3 | "Snowflake vs. Databricks — how do you decide?" | `databricks_guide.md`, `data/enterprise_data_platforms_guide.md` → **gap #1** for product depth |
| 4 | "How would CDC from Temenos/Flexcube look end-to-end?" | `data_integration_frameworks_guide.md` §6 (CDC deep-dive) + `../banking/temenos_data_model_guide.md` → **gap #5** for ops |
| 5 | "How does this architecture satisfy BCBS 239 principle 2 (data architecture and IT infrastructure)?" | `data_governance_guide.md` §12 (worked example) — DENSE |
| 6 | "Where does data mesh apply here, and where is it a distraction?" | `data/data_mesh_architectures_guide.md` — DENSE (argue the counter-case) |
| 7 | "What does the data catalog contribute and which one would you pick?" | `data/data_lineage_tools.md` → **gap #8** (catalog platforms) |
| 8 | "How do you protect personal data in the lake per PDPA?" | `data/data_compliance_frameworks.md` §b (PDPA) → **gap #3** for the enforcement layer (masking/RLS/ZTA) |
| 9 | "What is the 3-year TCO story for this platform, and how would you defend it to the CFO?" | `finops_guide.md` → **gap #2** (business case/TCO) |
| 10 | "How do you sequence a Teradata-to-cloud migration without a big-bang?" | `control_m_migration_options_guide.md`, `legacy_integration_patterns_guide.md`, `data/data_pipeline_guide.md` — DENSE (dual-run, domain-by-domain) |

**Rehearsal protocol:** for each question, produce a 60–90-second structured answer (claim → evidence → trade-off → next step) in the consulting style of `../management/mckinsey_approach_guide.md`. Questions 2, 3, 8, 9 are the ones that currently cost interview points — they map to gaps #6, #1, #3, #2 respectively, i.e., exactly the commissioned guides in §9.3.

---

## 7. One-Page Summary

**Goal:** Solution Architect (GenAI) → credible senior Data Architect, using the research repo as the digital profile (~379 guides; ~55 data-related).

**Readiness:** ~70–75% of a senior data-architect profile today.

**Strengths (DENSE — evidence-backed, do not spend study time here):**
- Data modeling: Kimball + Data Vault 2.0 + NoSQL + graph + 6 banking model guides
- Architecture patterns: mesh, fabric, lakehouse, Delta vs. Iceberg, MPP
- Integration: patterns + CDC deep-dive + real batch/file-transfer stack (Control-M, Axway CFT)
- Platforms: Databricks, Oracle, Neo4j, vector DBs, object storage
- Governance: 944-line governance guide incl. BCBS 239 worked example; 1,530-line compliance survey (GDPR, PDPA)
- ML data: feature stores, vector DBs, 18 RAG guides, drift, closed-loop data engineering
- Business/communication: consulting method + storytelling + stakeholder management

**Gaps (ranked):**
1. Managed cloud DW platforms (Snowflake/BigQuery/Redshift) — comparison-level only, no dedicated guide
2. Business case & TCO modeling — **standing priority**, absent
3. Zero Trust for data — **standing priority**, 5 mentions only
4. Platform Engineering for data — **standing priority**, absent
5. Streaming operations — concepts deep, Kafka/Flink/schema-registry ops absent
6. Data quality & observability — section-level only; no dbt/data contracts
7. DAMA-DMBOK formal — named 58×, never studied
8. Data catalog tooling — lineage covered, catalog platforms not
9. MDM implementation — section-level only
10. ML training-data lifecycle — serving side DENSE, training side thin

**Plan (9–12 months):** commission 7–8 companion guides following the repo's own pattern (cloud DW + TCO first, then Zero Trust + Platform Engineering — the standing priorities — then streaming ops + DQ, then DMBOK + catalog/MDM, then ML training data), optionally sit the CDMP exam, and convert knowledge into work evidence through CACIB data initiatives. Rehearse with the §6 scenario bank.

**Standing priorities carried forward and verified:** Zero Trust ✅ gap (no dedicated guide), Platform Engineering ✅ gap (no dedicated guide), business case/TCO ✅ gap (no dedicated guide).

---

## 8. Glossary

1. **Data architect** — The role responsible for the enterprise's data landscape: models, integration, platforms, governance, security, and the roadmap that ties data capabilities to business outcomes. Distinct from data engineer (builds pipelines) and data scientist (builds models): the architect sets the structure, standards, and target state that both operate within.
2. **Data modeling** — The discipline of representing data structures at conceptual (business concepts), logical (entities/attributes/relationships, technology-free), and physical (tables, indexes, partitions, engine-specific) levels; the foundation artifact of all data architecture.
3. **Dimensional modeling** — Kimball's approach for analytics: fact tables (numeric measures, foreign keys to dimensions) surrounded by dimension tables (descriptive attributes), with conformed dimensions shared across marts and slowly changing dimensions (SCD types 1–3) tracking history. Optimized for query performance and business readability.
4. **Data vault** — Dan Linstedt's modeling methodology (Data Vault 2.0, 2013) built on hubs (business keys), links (relationships), and satellites (attributes with history), using hash keys for parallel loadability, insert-only auditability, and resilience to source change. Bridges Kimball and Inmon.
5. **Data integration** — Moving and combining data across source systems into a target (warehouse, lake, operational store): batch, streaming, CDC, API, file-based; includes transformation, deduplication, reconciliation, and error handling.
6. **ETL** — Extract, Transform, Load: data is transformed in a staging area before loading into the target. The classic approach, well-suited when the target engine is not powerful enough to transform in place.
7. **ELT** — Extract, Load, Transform: raw data is loaded first and transformed inside the target (Snowflake, BigQuery, Databricks). Exploits powerful cloud engines; enables re-transformation without re-extraction. The modern default.
8. **Data warehouse** — A centralized, integrated, subject-oriented, time-variant, non-volatile store for analytics and reporting; classically Inmon 3NF or Kimball star schemas, historically on-prem (Teradata, Exadata), now cloud (Snowflake, BigQuery).
9. **Data lake** — A low-cost store for raw data in native formats (Parquet/ORC/JSON), schema-on-read, holding everything from landing zones to analytics. Strengths: flexibility, cheap storage. Weaknesses: governance and quality gaps without disciplined structure.
10. **Lakehouse** — The pattern that combines lake storage economics with warehouse governance: open table formats (Delta, Iceberg, Hudi) over object storage, ACID transactions, schema enforcement, and a unified engine for BI and ML. Databricks' model; now standard practice.
11. **Data mesh** — Zhamak Dehghani's organizational pattern: domain-owned data products, data as a product, self-serve data platform, federated computational governance. Shifts accountability from a central team to business domains.
12. **Data governance** — The framework of roles, policies, standards, and processes ensuring data is available, trustworthy, secure, and compliant; spans ownership/stewardship, quality, metadata, MDM, lineage, and regulatory obligations.
13. **DAMA** — The Data Management Association (DAMA International): the professional body for data management; publishes the DMBOK and administers the CDMP certification.
14. **DMBOK** — The Data Management Body of Knowledge (DAMA-DMBOK, 2nd edition): the standard reference organizing data management into 11 knowledge areas (data governance, architecture, modeling, storage, integration, quality, metadata, master data, warehousing/BI, security, big data/AI) and the data lifecycle.
15. **Data quality** — The degree to which data meets requirements, measured across dimensions: completeness, uniqueness, timeliness, validity, accuracy, consistency. Managed through rules, metrics/SLOs, profiling, and remediation workflows.
16. **Metadata** — Data about data: technical (schemas, columns, lineage), business (glossaries, definitions, ownership), and operational (run logs, freshness). Active metadata (machine-readable, continuously updated) powers data fabric and catalogs.
17. **Master data** — The core shared entities of an enterprise: customer/party, product, account, employee, location, and reference/security data; the single source of truth that operational and analytical systems consume via MDM.
18. **Streaming** — Continuous processing of events as they occur (vs. batch windows): event-driven pipelines, stream processing engines, time semantics, windows, state, exactly-once delivery; enables fraud detection, intraday liquidity, real-time risk.
19. **CDC** — Change Data Capture: capturing changes (inserts/updates/deletes) at the source — log-based (Debezium, GoldenGate) or query-based — and propagating them to targets in near-real-time; the standard bridge from OLTP systems to streaming/lakehouse.
20. **Cloud data platform** — The managed cloud services composing a modern data estate: object storage (S3/GCS/ADLS), warehouses (Snowflake/BigQuery/Redshift/Synapse), lakehouse (Databricks), processing (Glue/Dataflow/ADF), catalogs, and security (KMS); selected and integrated by the data architect.
21. **Vector database** — A store optimized for embedding vectors and similarity search (Pinecone, Milvus, Weaviate, Qdrant, pgvector); the retrieval backbone of RAG and semantic search.
22. **RAG** — Retrieval-Augmented Generation: grounding LLM output in retrieved documents (chunking → embedding → vector search → context assembly); the dominant pattern for making GenAI accurate and auditable on enterprise knowledge.
23. **Feature store** — The ML-data layer that centralizes feature definitions and values with online/offline consistency, point-in-time correctness, validation, and reuse across training and serving (Feast, Tecton, Databricks Feature Store).
24. **Data security** — The controls protecting data: encryption at rest/in transit, key management, access control (RBAC/ABAC, row/column-level), masking/tokenization, audit logging, secure sharing; increasingly framed under Zero Trust.
25. **Privacy** — The discipline of lawful, minimal, purpose-bound handling of personal data: consent, purpose limitation, data minimization, retention, subject rights, DPIA, privacy-by-design — distinct from (and enforced through) data security.
26. **PDPA** — Singapore's Personal Data Protection Act (2012, amended 2020/2024): the consent/notification-based regime governing personal data in Singapore; enforcement includes financial penalties (up to 10% of turnover under 2024 amendments) and applies extraterritorially to data of Singapore individuals.
27. **GDPR** — The EU General Data Protection Regulation (2018): the global benchmark privacy law — lawful bases, rights (access, erasure, portability), DPIA, breach notification, cross-border transfer rules (SCCs, DPF), fines up to 4% of global turnover / €20M.
28. **TCO** — Total Cost of Ownership: the full 3–5-year cost of a system including acquisition, migration, operations, staff, facilities/cloud, and decommissioning; the standard financial frame for platform selection and build-vs-buy.
29. **Business case** — The structured financial and strategic argument for an initiative: problem, options, costs, benefits, NPV/IRR/payback, risks, sensitivity analysis, and a recommendation; the artifact that gets data programs funded.

### 8.1 Glossary-to-repo mapping (where each term is already evidenced)

| Term | Primary repo evidence (relative to this file) | Coverage |
|---|---|---|
| Data architect | This guide; `architecture/skill_gaps_enterprise_architect_guide.md` | — |
| Data modeling | `data_model_resource_book_guide.md`, `nosql_data_modelling_guide.md` | DENSE |
| Dimensional modeling | `data/types_of_dimensions_data_warehousing.md`, `data/crm_data_warehouse_modelling.md` | DENSE |
| Data vault | `data/data_vault_2_modeling.md`, `data/handling_duplicate_keys_data_warehousing.md` | DENSE |
| Data integration | `data_integration_frameworks_guide.md`, `data/data_pipeline_guide.md`, `legacy_integration_patterns_guide.md` | DENSE |
| ETL / ELT | `data/data_pipeline_guide.md` §1 (explicit ETL vs. ELT), `databricks_guide.md` | DENSE |
| Data warehouse | `data/mpp_databases_guide.md`, `oracle_database_guide.md`, `data/enterprise_data_platforms_guide.md` | DENSE |
| Data lake | `cloud_object_storage_lakehouse_guide.md`, `s3_architecture_guide.md` | DENSE |
| Lakehouse | `databricks_guide.md`, `data/delta_lake_vs_iceberg.md`, `data/on_prem_lakehouse_guide.md` | DENSE |
| Data mesh | `data/data_mesh_architectures_guide.md` | DENSE |
| Data governance | `data_governance_guide.md`, `data/data_governance_framework.md` | DENSE |
| DAMA / DMBOK | Named across ~58 files; **no dedicated guide** (gap #7) | thin |
| Data quality | `data_governance_guide.md` §6, `data/data_profiling_guide.md`, `data/dataops_guide.md` | thin (section-level) |
| Metadata | `data_governance_guide.md` §7, `data/data_lineage_tools.md` | thin (section-level) |
| Master data | `data_governance_guide.md` §8, `../banking/data_models_banking_insurance_guide.md` | thin (section-level) |
| Streaming | `event_stream_processing_guide.md`, `complex_event_processing_guide.md`, `message_queue_data_loss_guide.md` | thin (concept DENSE, ops gap #5) |
| CDC | `data_integration_frameworks_guide.md` §6 | thin (section-level) |
| Cloud data platform | `cloud_providers_guide.md`, `data/enterprise_data_platforms_guide.md`, `databricks_guide.md` | thin (no managed-DW product depth, gap #1) |
| Vector database | `ai_llm/rag/vector_databases_guide.md` | DENSE |
| RAG | `ai_llm/rag/` (18 guides) | DENSE |
| Feature store | `feature_store_guide.md` | DENSE |
| Data security | `databricks_guide.md` §7, `distributed_auth_guide.md`, `openbao_vs_vault_guide.md` | thin (data-plane gap #3) |
| Privacy | `data/data_compliance_frameworks.md` (GDPR §a, PDPA §b) | DENSE (law) / thin (privacy engineering) |
| PDPA | `data/data_compliance_frameworks.md` §b | DENSE |
| GDPR | `data/data_compliance_frameworks.md` §a | DENSE |
| TCO | `finops_guide.md` (cloud cost only); **no TCO methodology guide** (gap #2) | gap |
| Business case | No dedicated guide; consulting base in `../management/mckinsey_approach_guide.md` (gap #2) | gap |

---

## 9. Verification Notes

### 9.1 What was scanned

- **Full directory listings** of `banking/` (74 guides), `management/` (22), `technology/` root (175 entries incl. subdirs), `technology/ai_llm/` (46), `technology/ai_llm/rag/` (18), `technology/data/` (25), `technology/architecture/` (8), `personal/` (4) — every `.md` filename classified against the 10 axes.
- **Line counts** (`wc -l`) for 23 core data guides as depth evidence (e.g., `data_governance_guide.md` 944, `data/data_compliance_frameworks.md` 1,530, `event_stream_processing_guide.md` 1,218, `data/data_mesh_architectures_guide.md` 928).
- **Table-of-contents greps** for 7 key guides to verify section-level coverage (CDC §6 in `data_integration_frameworks_guide.md`; DQ §6 / metadata §7 / MDM §8 / BCBS 239 §12 in `data_governance_guide.md`; GDPR §a / PDPA §b in `data/data_compliance_frameworks.md`).
- **Targeted keyword greps** (23 topics × whole repo): DAMA/DMBOK, master data, CDC, Snowflake/BigQuery/Redshift, PDPA/GDPR, Zero Trust, Platform Engineering, TCO/business case, data quality, metadata/catalog, Kafka, Flink/Spark, mesh, fabric, lakehouse, vector DB, feature store, RAG, Kimball/Inmon, data vault, lineage, observability.
- **Filename-pattern scans** for dedicated guides that should exist but don't (`*snowflake*`, `*bigquery*`, `*redshift*`, `*kafka*`, `*flink*`, `*mdm*`, `*quality*`, `*catalog*`, `*zero*trust*`, `*platform*engineering*`, `*business*case*`, `*dmbok*` → all zero hits).
- **Profile context files** `../personal/jack_liu_profile.md` and `../personal/jackliusr_digital_footprint.md` for role/bio grounding; sibling `architecture/skill_gaps_enterprise_architect_guide.md` for format and prior standing priorities.

### 9.2 What was flagged (honesty notes)

1. **Mentions ≠ coverage.** Whole-repo keyword counts overcount (e.g., "rag" matches inside "storage"; "lineage"/"catalog" match many incidental contexts). Every DENSE/thin/gap verdict above rests on *dedicated guides* (filenames) plus verified section-level coverage (TOC greps), not on raw grep counts. Raw counts are cited only directionally (e.g., "67 Snowflake mentions in `data/enterprise_data_platforms_guide.md`" as evidence of comparison-level — not product-level — coverage).
2. **Snowflake/BigQuery/Redshift absence is the single most consequential flag:** three guides discuss them in comparisons, zero dedicate a guide; the heatmap marks Data Platforms DENSE* and Cloud Data thin precisely to record this nuance.
3. **Zero Trust, Platform Engineering, business case/TCO are confirmed absent** as dedicated guides — the standing priorities from prior sessions are verified, not assumed.
4. **Streaming verdict is deliberately two-sided:** conceptual coverage is Advanced (1,218-line ESP guide with a banking chapter); the axis is scored thin because the operational/product layer (Kafka internals, Flink, schema registry, CDC-to-stream ops) has no dedicated guide.
5. **Governance is DENSE but operational tooling is section-level:** DQ, metadata, MDM, and lineage all exist as strong sections inside `data_governance_guide.md`; the gap analysis distinguishes "covered as sections" (governance) from "mentioned only" (catalog platforms, dbt, data contracts).
6. **Line counts reflect file length, not quality** — they are used as a depth proxy consistent with the repo's own convention of 400–1,500-line guides.
7. **Readiness estimate (70–75%) is judgment**, triangulated from axis scores in §2/§6.5; it is a planning input, not a measurement.
8. **Banking-domain data depth** (`../banking/temenos_data_model_guide.md`, `../banking/fineract_database_models_guide.md`, `../banking/oracle_flexcube_data_model_guide.md`, `../banking/data_models_banking_insurance_guide.md`, `../banking/universal_banking_model_guide.md`) is a genuine differentiator this scan surfaced — most aspiring data architects cannot model a bank; the profile can.
9. **Proposed companion guides do not exist yet.** Every filename in the §9.3 commission list (e.g., `cloud_data_warehouse_platforms_guide.md`, `zero_trust_data_platform_guide.md`) is a plan, not part of the profile; a link-integrity check of all 128 referenced paths confirms exactly those 10 are the only unresolved references, by design.

### 9.3 Companion-guide commission list (the repo's own pattern)

| Guide (proposed filename) | Gap | Est. size |
|---|---|---|
| `cloud_data_warehouse_platforms_guide.md` | #1 | ~700 L |
| `data_business_case_tco_guide.md` | #2 (standing) | ~500 L |
| `zero_trust_data_platform_guide.md` | #3 (standing) | ~550 L |
| `data_platform_engineering_guide.md` | #4 (standing) | ~600 L |
| `streaming_platform_operations_guide.md` | #5 | ~650 L |
| `data_quality_observability_guide.md` | #6 | ~500 L |
| `dama_dmbok_guide.md` | #7 | ~600 L |
| `data_catalog_platforms_guide.md` (or fold into #7) | #8 | ~450 L |
| `master_data_management_guide.md` (or fold into #7) | #9 | ~500 L |
| `ml_training_data_lifecycle_guide.md` (or fold into closed-loop guide) | #10 | ~450 L |

---

## Appendix A — Running This Assessment Again (repeatable protocol)

The digital profile is a living asset: every commissioned guide changes the heatmap. Re-run this assessment quarterly (or after each research sprint) in six steps:

1. **Re-scan:** `ls` every directory (banking/, technology/ + data/ + architecture/ + ai_llm/ + ai_llm/rag/, management/) and diff filenames against the previous scan.
2. **Re-verify section-level claims:** grep TOCs (`grep -nE '^#{1,2} '`) of the guides named as section-level evidence; a section may have grown into a guide (promote the axis) or a guide may have been superseded.
3. **Re-run the gap probes:** the filename-pattern checks in §9.1 (`*snowflake*`, `*zero*trust*`, `*platform*engineering*`, `*business*case*`, `*dmbok*`, `*flink*`, `*mdm*`, `*quality*`, `*catalog*`) — a hit means a gap closed and the ranking shifts.
4. **Re-score the ten axes** with the §1.11 rubric (L1–L5) and update the §2.11 table; the target is L4 everywhere with L5 on Modeling, Patterns, Governance.
5. **Re-prioritize §4** — standing priorities may change; new ones should be added only with repo evidence (mentions ≠ coverage).
6. **Re-plan §5** — commission new guides in the §9.3 format; update the readiness estimate in §0.

**Stopping rule:** when the heatmap shows no axis below L4, and the §6.8 question bank can be answered at 60–90 seconds each with evidence, the data-architect trajectory is interview-ready; further effort should shift to *work evidence* (initiatives led) rather than knowledge acquisition — the classic senior-architect inflection.

---

*This guide was generated from a live scan of github.com/jackliusr/research (August 2026). Where the repo is silent, this guide says so. Companion guides follow the repo's established format: author header, TOC, 400–900 lines, banking context, references.*
