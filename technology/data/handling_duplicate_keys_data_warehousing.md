# Handling Duplicate Candidate Keys & Minor Attribute Differences in Data Warehousing

*A comprehensive guide to detecting, resolving, and preventing duplicate records across heterogeneous source systems in analytical data platforms.*

---

## 1. The Problem Statement

### 1.1 What Are Duplicate Candidate Keys?

A **candidate key** is a minimal set of attributes that uniquely identifies a business entity — a customer ID, email, product SKU, or composite of name + birth date. **Duplicate candidate keys** occur when the same real-world entity appears multiple times in a data warehouse under different surrogate or physical representations, while logically representing the same object.

### 1.2 "Minor Differences" in Attributes

Duplicate records rarely look identical. The same customer carries slight variations across source systems:

| Attribute | System A (CRM) | System B (ERP) | System C (Legacy) |
|-----------|----------------|----------------|-------------------|
| Name | John Smith | Jon Smith | John A. Smith |
| Phone | +1-555-0100 | 555-0100 | (null) |
| Address | 100 Main St, Springfield | 100 Main Street, Springfield, IL | 100 Main St |
| Email | jsmith@acme.com | john.smith@acme.com | (null) |

None is "wrong" — but the warehouse has no reliable single column to prove they are the same person.

### 1.3 Why This Happens

- **Multiple source systems**: CRM, ERP, billing, marketing, and legacy systems maintain independent identity schemes.
- **Data entry inconsistencies**: Free-text fields, typos ("Jon" vs "John"), inconsistent abbreviations ("St." vs "Street").
- **Lack of data governance**: No enforced standard for entity naming, formatting, or identification.
- **System migrations**: Data moved with transformation errors, padding changes, or truncation.
- **M&A data merging**: Post-merger integration of databases that never shared a common identifier.

### 1.4 Impact on the Data Warehouse

- **Inflated counts**: `COUNT(DISTINCT customer_id)` returns 12,000 when only 8,500 real customers exist.
- **Broken referential integrity**: Fact rows reference multiple SKs for the same entity; join paths fan out.
- **Incorrect aggregates**: Revenue-per-customer, churn rate, LTV — all distorted.
- **Lost insights**: Segmentation and ML models trained on duplicated data produce unreliable outputs.
- **Wasted storage**: Duplicate rows consume disk, memory, and compute for no marginal value.

### 1.5 Real-World Example: "John Smith"

A retail company runs three source systems: **Salesforce CRM** ("John Smith", jsmith@acme.com, +1-555-0100), **SAP ERP** ("Jon Smith", john.smith@acme.com, different account #), and **Legacy billing** ("John A. Smith", different ID, +1-555-0100). Without dedup, the warehouse creates three customer rows, three keys for John's purchases, and a `COUNT(DISTINCT customer)` three times too high. Marketing sends three copies of the same campaign. Revenue-per-customer reads $4,500 when the true figure is $13,500.

---

## 2. Data Quality Dimensions (Framework)

A systematic approach rests on six data quality dimensions:

| Dimension | Definition | Duplicate-Related Question |
|-----------|------------|---------------------------|
| **Uniqueness** | No duplicates for the same entity | Are there records that refer to the same entity? |
| **Completeness** | Required attributes are populated | Which source has the most complete address? |
| **Consistency** | Same values have same representation | Is "St." vs "Street" normalised? |
| **Accuracy** | Values reflect true real-world state | Which source has the correct phone? |
| **Timeliness** | Data is current and up-to-date | Which system has the latest address? |
| **Validity** | Values conform to allowed domains | Is the email format valid? |

**Dimensional scoring** — each source record gets a score per dimension. These feed survivorship rules when merging duplicates (Section 5).

---

## 3. Strategic Approaches

### 3.1 Master Data Management (MDM)

A dedicated MDM hub ingests from all sources, performs identity resolution, and publishes a single **golden record** per entity.

```
Source A ──┐
Source B ──┤   Match ──► Merge ──► Survivorship ──► Distribute
Source C ──┘
```

1. **Match**: Run deterministic + probabilistic matching across all records.
2. **Merge**: Combine matched records into candidate groups.
3. **Survivorship**: Apply business rules to select the best attribute values.
4. **Distribute**: Publish the golden record to consuming systems.

**MDM Tools:** Informatica MDM, IBM MDM, Profisee, Semarchy xDM, Ataccama ONE.

**MDM in the DWH:** The MDM system feeds conformed dimension tables. ETL loads `dim_customer` from the MDM hub rather than directly from raw sources.

### 3.2 Data Warehouse Consolidation (Match & Merge in ETL/ELT)

Three patterns when MDM is not in place:

**a) Identity resolution in staging:** Match and merge before loading dimensions. Staging acts as a match workspace; only surviving records enter dimensions. *Pros: Clean dimensions. Cons: Audit trail lost.*

**b) Scratchpad tables:** Dedicated consolidation tables collect candidates. A scheduled merge process scores, matches, and consolidates. *Pros: Re-runnable. Cons: Extra tables.*

**c) Data Vault style:** Load every raw record as-is into Data Vault. Hubs hold business keys, satellites hold attributes. A Business Vault layer applies survivorship rules for the presentation layer. *Pros: Full audit trail. Cons: More tables.*

### 3.3 Source System Fix (Long-term)

Prevent duplicates at origin — most effective, least feasible.

- **Data governance program**: Standards, ownership, stewardship for each domain.
- **Unique constraints**: Add `UNIQUE` on candidate keys (email, tax ID) in source databases.
- **Input validation**: Dropdowns, address validation APIs, phone formatting libraries.
- **Single identity service**: All ops systems call one service to look up or register entities.

**Reality check:** Source fixes are multi-quarter. The pragmatic path is MDM or pipeline-level dedup now, plus source improvements over time.

---

## 4. Matching Techniques

### 4.1 Exact Matching

Fast and precise — but brittle. Works only when source systems share a common identifier.

```sql
SELECT a.customer_id, b.customer_id
FROM crm_customer a JOIN erp_customer b
  ON LOWER(TRIM(a.email)) = LOWER(TRIM(b.email));
```

**When it works:** Shared global identifier; uniform key population. **When it fails:** No shared ID; differing case, whitespace, punctuation.

### 4.2 Fuzzy Matching

Approximate string similarity for records that are close but not identical.

| Algorithm | Strengths | Weaknesses |
|-----------|-----------|------------|
| **Levenshtein** | Simple edit-distance; widely available | Slow on long strings |
| **Jaro-Winkler** | Excellent for names; boosts common prefixes | Not ideal for long text |
| **Soundex/Metaphone** | Phonetic ("Smith"/"Smythe") | Low precision |
| **N-gram (Trigrams)** | Robust to token reordering | Threshold tuning needed |
| **Token Sort Ratio** | Normalises word order | Ignores meaning |

**PostgreSQL (pg_trgm):** `SELECT similarity(a.name, b.name) AS score FROM crm_customer a CROSS JOIN erp_customer b WHERE similarity(a.name, b.name) > 0.7;`

**Python (thefuzz/rapidfuzz):** `process.extract(row['name'], known_names, scorer=fuzz.token_sort_ratio)` with score threshold ≥ 85.

**Apache Spark:** `df_a.join(df_b, soundex("name")).filter(levenshtein("a.name", "b.name") < 3)`

**SQL Server:** `SELECT DIFFERENCE(a.Name, b.Name) AS match_score FROM CRM_Customer a JOIN ERP_Customer b ON DIFFERENCE(a.Name, b.Name) >= 3;`

### 4.3 Deterministic vs Probabilistic Matching

**Deterministic:** Rule-based — if A matches B on exact rules, same entity. Transparent, fast, debuggable. Misses subtle partial matches.

```
IF LOWER(email_a) = LOWER(email_b) THEN match
ELSE IF SOUNDEX(name_a) = SOUNDEX(name_b) AND zip_a = zip_b THEN match
```

**Probabilistic (Fellegi-Sunter):** Computes match probability by scoring agreement/disagreement across fields. Each field has m-probability (agreement given true match) and u-probability (agreement given non-match). The ratio m/u provides field weights. Used by IBM InfoSphere QualityStage, SAS Dataflux.

- **Pros:** Captures partial evidence across multiple weak signals; adapts to data.
- **Cons:** Needs training data or EM estimation; less transparent.

**Hybrid (recommended for production):**
1. **Deterministic blocking** — cheap rules to narrow candidate pairs.
2. **Probabilistic scoring** — score each pair across all fields.
3. **Threshold decision** — above high threshold → auto-match. Below low threshold → auto-non-match. Between → manual review.

### 4.4 Blocking / Indexing

A cross-join of two 1M-row tables = 1 trillion pairs. **Blocking** groups records into buckets and compares only within each bucket.

**Common blocking keys:** ZIP (first 3–5 chars), first 3 chars of surname, Soundex(last_name), birth year, first 4 chars of company name, email domain.

**Multi-pass blocking** — run 3–5 passes with different keys, union results:

```sql
WITH candidates AS (
  SELECT a.id, b.id FROM a JOIN b ON a.zip = b.zip
  UNION
  SELECT a.id, b.id FROM a JOIN b ON SOUNDEX(a.last_name) = SOUNDEX(b.last_name)
  UNION
  SELECT a.id, b.id FROM a JOIN b ON a.birth_year = b.birth_year
)
SELECT DISTINCT a_id, b_id FROM candidates;
```

---

## 5. Survivorship (Record Consolidation) Rules

After identifying duplicates, decide which attribute values survive into the **golden record**.

### Common Survivorship Strategies

| Strategy | Description | Best For |
|----------|-------------|----------|
| **Most Recent** | Value from latest update timestamp | Address, phone, status |
| **Most Complete** | Record with most populated fields | Master record creation |
| **Source Priority** | Ranked sources (ERP > CRM > Legacy) | Corporate governance |
| **Most Frequent** | Value appearing most often | Name, description |
| **Longest/Shortest** | Pick longest description | Descriptions, dates |
| **Truncate/Append** | Combine: "John Smith (aka Jon)" | Aliases, alternate names |
| **FIFO/LIFO** | First/last from creation timestamp | Creation date audits |
| **Custom Rule** | Domain-specific logic | Industry attributes |

### SQL Survivorship Pattern

```sql
WITH ranked AS (
  SELECT *,
    ROW_NUMBER() OVER (
      PARTITION BY golden_record_group
      ORDER BY source_priority ASC,
               last_modified DESC,
               completeness_score DESC
    ) AS rn
  FROM unified_candidates
)
SELECT * FROM ranked WHERE rn = 1;
```

### Per-Attribute Survivorship

Often no single record has the best value for every attribute. Per-attribute survivorship selects the best value per column independently:

```sql
SELECT group_id,
  (SELECT name FROM unified u2 WHERE u2.group_id = u.group_id AND name IS NOT NULL
   ORDER BY CASE source_system WHEN 'ERP' THEN 1 WHEN 'CRM' THEN 2 ELSE 3 END,
            LENGTH(name) DESC LIMIT 1) AS name,
  (SELECT phone FROM unified u3 WHERE u3.group_id = u.group_id AND phone IS NOT NULL
   ORDER BY last_modified DESC LIMIT 1) AS phone
FROM (SELECT DISTINCT group_id FROM unified) u;
```

---

## 6. Handling in the Data Warehouse (by Architecture)

### 6.1 Kimball Dimensional Model

**Option A: Clean in staging.** Identity resolution before records reach dimension tables. *Pros: Clean dimensions, simple ETL. Cons: Audit trail lost.*

**Option B: SCD Type 2 + bridge.** Load every source record tagged with `source_system_id`. A bridge table links duplicate SKs to the single logical entity.

```sql
CREATE TABLE bridge_customer_entity (
  golden_sk  INT NOT NULL,
  member_sk  INT NOT NULL,
  is_master  CHAR(1) DEFAULT 'N',
  PRIMARY KEY (golden_sk, member_sk)
);
```

**Option C: Golden record dimension + source overlap tracking.** Best-version dimension with a separate bridge for lineage.

```sql
CREATE TABLE dim_customer_golden (
  golden_sk INT PRIMARY KEY, customer_id VARCHAR(50),
  name VARCHAR(200), address VARCHAR(500), phone VARCHAR(50), email VARCHAR(200)
);
CREATE TABLE bridge_customer_source (
  golden_sk INT REFERENCES dim_customer_golden(golden_sk),
  source_system VARCHAR(20), source_id VARCHAR(50), source_priority INT,
  PRIMARY KEY (golden_sk, source_system, source_id)
);
```

### 6.2 Data Vault Model

Data Vault 2.0 handles duplicate business keys natively.

```
RAW VAULT:
  Hubs — business keys from each source (duplicates allowed)
  Satellites — all source attribute sets (one per source per hub)
  Links — relationships use hub hash keys
  Same-As Links (SAL) — equivalence between hub records

BUSINESS VAULT:
  Best-record satellites — survivorship rules applied
  Point-in-Time tables — temporal bridge to correct satellite version
```

**Same-As Link example:** `CREATE TABLE sal_customer (cust_hash_a CHAR(32), cust_hash_b CHAR(32), match_reason VARCHAR(200), match_score DECIMAL(5,2), load_date TIMESTAMP, PRIMARY KEY (cust_hash_a, cust_hash_b));`

**Pros:** Full audit trail, source data never modified, re-runnable, incremental. **Cons:** More tables, higher storage, needs Data Vault discipline.

### 6.3 One Big Table (OBT) / Flat Model

Dedup applied at materialisation via window functions. Least preferred.

```sql
SELECT * FROM (
  SELECT *, ROW_NUMBER() OVER (
    PARTITION BY customer_group ORDER BY source_priority, last_updated DESC
  ) AS rn FROM customer_obt_raw
) WHERE rn = 1;
```

**Pros:** Simple. **Cons:** Dedup logic repeated per query/refresh; inconsistent across schedules.

---

## 7. Operational Patterns & SQL Examples

### 7.1 Finding Duplicates

```sql
-- Identify duplicates by email
SELECT email, COUNT(*) AS cnt,
       ARRAY_AGG(customer_id ORDER BY last_updated) AS ids,
       ARRAY_AGG(source_system ORDER BY last_updated) AS sources
FROM staging_customer WHERE email IS NOT NULL
GROUP BY email HAVING COUNT(*) > 1 ORDER BY cnt DESC;

-- Duplicate pairs with field-level differences
SELECT a.customer_id, b.customer_id,
       a.name, b.name, a.phone, b.phone, a.address, b.address,
       CASE WHEN a.name != b.name THEN 'NAME ' ELSE '' END ||
       CASE WHEN a.phone != b.phone THEN 'PHONE ' ELSE '' END ||
       CASE WHEN a.address != b.address THEN 'ADDRESS ' ELSE '' END AS diffs
FROM staging_customer a
JOIN staging_customer b ON COALESCE(a.email,'') = COALESCE(b.email,'')
  AND a.customer_id < b.customer_id
WHERE a.name != b.name OR a.phone != b.phone OR a.address != b.address;
```

### 7.2 Fuzzy Matching Pipeline (Python)

```python
import pandas as pd
from thefuzz import fuzz, process

crm = pd.read_csv("crm_customers.csv")
erp = pd.read_csv("erp_customers.csv")
erp_names = erp["name"].str.lower().str.strip().tolist()
erp_ids = erp["customer_id"].tolist()

matches = []
for _, row in crm.iterrows():
    candidates = process.extract(row["name"].lower().strip(), erp_names,
                                 scorer=fuzz.token_sort_ratio, limit=2)
    for match_name, score, idx in candidates:
        if score >= 80:
            matches.append({"crm_id": row["customer_id"], "erp_id": erp_ids[idx],
                            "score": score, "name_crm": row["name"]})

match_df = pd.DataFrame(matches).drop_duplicates(["crm_id", "erp_id"])
```

### 7.3 Survivorship Merge via ROW_NUMBER

```sql
WITH unified AS (
  SELECT customer_id, name, email, phone, address, last_updated,
         'CRM' AS src, 10 AS src_weight,
         (CASE WHEN name IS NOT NULL THEN 1 ELSE 0 END +
          CASE WHEN email IS NOT NULL THEN 1 ELSE 0 END +
          CASE WHEN phone IS NOT NULL THEN 1 ELSE 0 END +
          CASE WHEN address IS NOT NULL THEN 1 ELSE 0 END) AS completeness
  FROM crm_customer UNION ALL
  SELECT customer_id, name, email, phone, address, last_updated,
         'ERP', 8,
         (CASE WHEN name IS NOT NULL THEN 1 ELSE 0 END +
          CASE WHEN email IS NOT NULL THEN 1 ELSE 0 END +
          CASE WHEN phone IS NOT NULL THEN 1 ELSE 0 END +
          CASE WHEN address IS NOT NULL THEN 1 ELSE 0 END)
  FROM erp_customer UNION ALL
  SELECT customer_id, name, email, phone, address, last_updated,
         'Legacy', 5,
         (CASE WHEN name IS NOT NULL THEN 1 ELSE 0 END +
          CASE WHEN email IS NOT NULL THEN 1 ELSE 0 END +
          CASE WHEN phone IS NOT NULL THEN 1 ELSE 0 END +
          CASE WHEN address IS NOT NULL THEN 1 ELSE 0 END)
  FROM legacy_customer
),
ranked AS (
  SELECT *, ROW_NUMBER() OVER (
    PARTITION BY COALESCE(email, customer_id)
    ORDER BY src_weight DESC, completeness DESC, last_updated DESC
  ) AS rn FROM unified
)
INSERT INTO dim_customer (customer_sk, name, email, phone, address, source_system)
SELECT ROW_NUMBER() OVER (ORDER BY COALESCE(email, customer_id)) +
         COALESCE((SELECT MAX(customer_sk) FROM dim_customer), 0),
       name, email, phone, address, src
FROM ranked WHERE rn = 1;
```

### 7.4 dbt Dedup & Quality Tests

```sql
-- models/dim_customer_dedup.sql
{{ config(materialized='table') }}
WITH source AS (SELECT * FROM {{ ref('stg_customer_all_sources') }}),
deduped AS (
  SELECT *, ROW_NUMBER() OVER (
    PARTITION BY match_key ORDER BY source_priority, loaded_at DESC
  ) AS rn FROM source WHERE match_key IS NOT NULL
)
SELECT {{ dbt_utils.generate_surrogate_key(['match_key']) }} AS customer_sk,
       name, email, phone, address, source_system
FROM deduped WHERE rn = 1
```

```yaml
# tests/generic/test_no_duplicate_candidate_keys.yml
version: 2
models:
  - name: stg_customer_all_sources
    tests:
      - dbt_utils.expression_is_true:
          expression: "ROW_NUMBER() OVER (PARTITION BY email ORDER BY loaded_at) = 1"
          condition: "email IS NOT NULL"
          name: unique_email_in_staging
          severity: warn
```

---

## 8. Special Cases

### 8.1 Merge/Purge (List Cleaning)

Marketing list dedup: exact match on email → fuzzy match on name+address → cluster transitive matches → manual review.

**Tools:** OpenRefine (free, GUI), DataCleaner (free, GUI), Melissa Data (commercial).

### 8.2 Cross-System Identity Resolution

**Enterprise ID / Global ID:** A canonical identifier from a central identity service. Every system maps its local ID to the global ID.

**Identity Graph:** Maps every known identifier (email, phone, device ID, account number, loyalty card) to a single entity node. Used by Salesforce CDP, ActionIQ.

**Customer 360:** Consolidating CRM + support + billing + marketing + web behaviour into a unified profile across 20+ source systems.

**Implementation path:** Collect all identifiers → run matching → assign global entity ID per cluster → maintain `entity_id_map(source_system, local_id, global_id)` → all downstream analytics reference `global_id`.

### 8.3 M&A Data Integration

| Phase | Activity | Duration |
|-------|----------|----------|
| Phase 1 | Load both into staging with system tags | Weeks |
| Phase 2 | Coarse matching (tax ID, email) | Weeks |
| Phase 3 | Fine matching (name + address, fuzzy) | Weeks–Months |
| Phase 4 | Manual review via stewardship UI | Ongoing |
| Phase 5 | Survivorship + golden record creation | Months |
| Phase 6 | Backfill fact tables to reference golden keys | Months |

**Key challenges:** Different ID systems (solve with identity map), different attribute definitions (document mapping rules), time pressure (start thin, enrich over time).

---

## 9. Tools & Technologies

| Tool | Type | Best For |
|------|------|----------|
| **OpenRefine** | Free, open source | Ad-hoc cleaning, fuzzy matching, clustering |
| **Python (pandas + thefuzz)** | Free, programmable | Custom dedup pipelines, ETL integration |
| **Apache Spark (spark-validator)** | Free, scalable | Large-scale dedup in data lakes |
| **PostgreSQL (pg_trgm)** | Free, embedded DB | Fuzzy matching in relational DB |
| **dbt (data build tool)** | Free, open-source ELT | Dedup as transform step + quality tests |
| **Zingg** | Free, ML-based | ML entity resolution, trains dedup models |
| **Ataccama ONE** | Commercial, AI-assisted | Automated matching + survivorship |
| **Profisee** | Commercial, cloud-native | Enterprise MDM + stewardship |
| **Informatica MDM** | Commercial, enterprise | Full-scale MDM with governance |
| **IBM MDM / QualityStage** | Commercial, enterprise | Probabilistic matching, virtual/physical hub |
| **Tamr** | Commercial, ML-powered | ML-based entity resolution at scale |
| **Data Ladder / Melissa Data** | Commercial | Address verification + dedup + survivorship |
| **SAS Dataflux** | Commercial | Data quality + probabilistic matching |

---

## 10. Prevention & Governance

- **Data governance**: Define data standards, assign owners, implement stewardship for duplicate review.
- **Unique constraints in source**: Prevent duplicates at creation where feasible.
- **Input validation**: Dropdowns, address validation APIs, phone formatters.
- **Dedup at registration**: Search existing records before creating new ones.
- **Monitoring dashboards**: Track duplicate rates per domain per source. Alert on spikes.
- **CI/CD for data quality**: Automated dedup tests in every pipeline deployment. Use `dbt test` to enforce uniqueness on candidate keys. Block deployments that introduce new duplicates.
- **Data lineage**: Every golden record attribute carries a pointer to source system, source record, and timestamp. Use dbt docs, Apache Atlas, or OpenLineage.

---

## 11. Comparison: Approaches for Different Scales

| Scale | Dataset | Approach | Tools |
|-------|---------|----------|-------|
| **Small** | < 100K rows | Interactive dedup | OpenRefine, Excel, Python |
| **Medium** | 100K–10M rows | ETL/ELT + ROW_NUMBER + fuzzy matching | dbt, PostgreSQL, Python |
| **Large** | 10M–1B rows | Spark-based dedup, Data Vault hubs + survivorship | Spark, Zingg, Tamr |
| **Enterprise** | Multi-source, multi-domain | MDM hub + governance + stewardship | Informatica MDM, Profisee, IBM MDM |

---

## Data Sources & Further Reading

- Kimball Group — *The Data Warehouse Toolkit, 3rd Edition* (ETL and data quality chapters)
- Dan Linstedt & Michael Olschimke — *Building a Scalable Data Warehouse with Data Vault 2.0*
- David Loshin — *Master Data Management* and *The Practitioner's Guide to Data Quality Improvement*
- DAMA International — *DAMA-DMBOK: Data Management Body of Knowledge*
- Fellegi & Sunter — "A Theory for Record Linkage" (1969), *Journal of the American Statistical Association*
- PostgreSQL docs — `pg_trgm` extension
- dbt Labs — Data quality testing and `dbt-utils` documentation
- Apache Spark — Data quality best practices
- Microsoft — SQL Server DQS documentation
- Python — `thefuzz` / `rapidfuzz` library docs
- Zingg — Open-source ML entity resolution (zingg.ai)
- Tamr — ML-powered data mastering (tamr.com)
- Profisee — MDM survivorship (profisee.com/blog/mdm-survivorship/)
- Data Ladder — Guide to golden record survivorship (dataladder.com)
- ScaleFree — Data Vault Same-As-Link patterns (scalefree.com)

---

*This guide covers the full lifecycle of handling duplicate candidate keys in data warehousing — from problem definition and matching techniques through survivorship, architecture-specific patterns, tools, governance, and scale-based recommendations. The right approach depends on data volume, source system heterogeneity, and organisational data governance maturity.*
