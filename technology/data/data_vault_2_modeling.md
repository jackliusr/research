# Data Vault 2.0 Data Modeling Methodology — A Comprehensive Guide

> **Author:** Generated Research | **Topic:** Data Vault 2.0 Enterprise Data Warehousing | **Audience:** Data engineers, architects, decision-makers

---

## 1. What is Data Vault 2.0?

### Origins
Data Vault was created by **Dan Linstedt** in the 1990s as a response to the rigidity of Kimball and Inmon approaches when facing rapidly changing source systems and audit requirements. The original methodology was formalized around 2000, and **Data Vault 2.0** was released in 2013, codified in Linstedt & Olschimke's *"Building a Scalable Data Warehouse with Data Vault 2.0"* (2015).

### Core Philosophy

1. **Agility** — Model adapts to source system changes without breaking existing structures. Changes are additive, not destructive.
2. **Auditability** — Every data point traces to its source system and load timestamp. Insert-only pattern preserves complete history.
3. **Scalability** — Hash keys enable parallel loading across all hubs, naturally suiting MPP databases and cloud warehouses.
4. **Integration** — Designed for multi-source enterprises where the same business concept appears across systems with different identifiers.

### Key Differences from Other Methodologies

| Dimension | Data Vault 2.0 | Kimball (Star Schema) | Inmon (3NF) |
|-----------|---------------|----------------------|-------------|
| **Primary concern** | Audit trail & integration | Query performance & usability | Enterprise consistency |
| **Loading** | Parallel, insert-only | Sequential dim loads | Complex ELT |
| **Source changes** | Additive (new satellites) | Dimension rebuilds | Full remodelling |
| **Business user accessibility** | Low (needs PIT/Bridge) | High (intuitive star schemas) | Low |
| **Storage** | 2-3× larger (full history) | Smaller (partial history) | Moderate |

**vs Kimball:** Kimball prioritizes query performance via denormalized facts/dimensions. Star schemas are intuitive for business users, but source changes often require painful dimension rebuilds. DV 2.0 sacrifices query performance (requiring PIT/Bridge tables as a presentation layer) in exchange for fully additive source changes.

**vs Inmon:** Inmon's CIF normalizes into 3NF for consistency but can be rigid. DV 2.0 separates business keys (hubs), relationships (links), and context (satellites) into independent structures — source changes typically only add a new satellite rather than changing the schema across the entire model.

### The Three Core Components
- **Hubs** — Store unique business keys (the *who* and *what*)
- **Links** — Model relationships between hubs (the *how* entities relate)
- **Satellites** — Store descriptive context and history (the *when*, *where*, *why*)

### What DV 2.0 Added Over Original Data Vault
- **Hash keys** (replaced integer surrogate keys for parallel loading)
- **Point-in-Time (PIT) tables** (snapshots for efficient time-based queries)
- **Bridge tables** (flatten many-to-many relationships for BI tools)
- **Business Vault** (formalized layer for computed/best-record data)
- **Parallel loading pattern** (standardized methodology)
- **Hub-and-spoke architecture** (integrated into enterprise architecture)

---

## 2. Core Components

### 2.1 Hubs

**Purpose:** Store unique business keys (natural keys) from source systems. Represent core entities like Customer, Product, Order, Employee, Location.

**Structure:**

| Column | Type | Description |
|--------|------|-------------|
| `{entity}_hk` | CHAR(32) / BINARY(16) | Hash key (MD5 of business key) — PK |
| `{business_key}` | VARCHAR | Natural key from source |
| `record_source` | VARCHAR(50) | Source system identifier |
| `load_dts` | TIMESTAMP | When first loaded |

**Rules:** Never update. Business keys never change (if a key changes, it's a *new* entity). Each unique business key across all sources maps to one hub row.

**Example SQL:**

```sql
CREATE TABLE hub_customer (
    customer_hk   CHAR(32) PRIMARY KEY,    -- MD5 hash of business key
    customer_id   VARCHAR(100) NOT NULL,    -- business key from source
    record_source VARCHAR(50)  NOT NULL,
    load_dts      TIMESTAMP    NOT NULL,
    UNIQUE (customer_id, record_source)
);
```

**Composite business key:**

```sql
CREATE TABLE hub_product (
    product_hk     CHAR(32) PRIMARY KEY,
    product_sku    VARCHAR(50) NOT NULL,
    product_region VARCHAR(20) NOT NULL,
    record_source  VARCHAR(50) NOT NULL,
    load_dts       TIMESTAMP   NOT NULL,
    UNIQUE (product_sku, product_region, record_source)
);
```

**Naming:** `hub_<entity>` or `H_<ENTITY>` (e.g., `hub_customer`, `H_CUSTOMER`).

**Best practices:**
- Use **hash keys** (MD5) over sequence/surrogate keys — enables parallel loading
- Always **trim and uppercase** before hashing for case-insensitive consolidation: `MD5(UPPER(TRIM(business_key)))`
- Avoid surrogate sequence keys (create sequential dependencies)
- Same business key from different sources should produce the same hash for cross-source consolidation

### 2.2 Links

**Purpose:** Model relationships between hubs — many-to-many, many-to-one, one-to-one, and n-ary (3+ hubs).

**Structure:**

| Column | Type | Description |
|--------|------|-------------|
| `{link_name}_hk` | CHAR(32) | Hash of entire relationship — PK |
| `{hub1}_hk` | CHAR(32) | FK to first hub |
| `{hub2}_hk` | CHAR(32) | FK to second hub |
| ... | ... | Additional hub FKs |
| `record_source` | VARCHAR(50) | Source system |
| `load_dts` | TIMESTAMP | When relationship first observed |

**Rules:** Never update. Captures the moment a relationship was first known. If relationships change over time, add a satellite on the link rather than modifying the link. Links contain no descriptive attributes — those go in link satellites.

**Example SQL:**

```sql
CREATE TABLE link_customer_order (
    link_customer_order_hk CHAR(32) PRIMARY KEY,
    customer_hk            CHAR(32) NOT NULL REFERENCES hub_customer(customer_hk),
    order_hk               CHAR(32) NOT NULL REFERENCES hub_order(order_hk),
    record_source          VARCHAR(50) NOT NULL,
    load_dts               TIMESTAMP NOT NULL
);
```

**Naming:** `link_<entity1>_<entity2>` or `L_<ENTITY1>_<ENTITY2>`.

**Best practices:**
- Represent **business relationships**, not every database foreign key
- **Self-referencing links** model hierarchies (e.g., `link_employee_manager`)
- **N-ary links** (3+ hubs) are valid (e.g., customer+product+store for a purchase)
- **Non-historical links** (without satellites) are fine when the relationship has no changing descriptive attributes

### 2.3 Satellites

**Purpose:** Store descriptive attributes (context) that change over time. Attached to hubs (hub satellite) or links (link satellite).

**Structure:**

| Column | Type | Description |
|--------|------|-------------|
| `{parent}_hk` | CHAR(32) | FK to parent hub/link |
| `load_dts` | TIMESTAMP | When this version was loaded |
| `record_source` | VARCHAR(50) | Source system |
| `hash_diff` | CHAR(32) | Optional: MD5 of all descriptive columns for change detection |
| `{attr1..N}` | Various | Descriptive attributes |
| `load_end_dts` | TIMESTAMP | Optional: for DELETE tracking |

**Types:**
- **Hub Satellite** — Describes a hub's attributes (customer name, email, segment)
- **Link Satellite** — Describes a relationship's attributes (order status, quantity, discount)
- **Multi-active Satellite** — Handles multi-valued attributes with a sequence/date in the PK

**Example SQL (hub satellite):**

```sql
CREATE TABLE sat_customer_detail (
    customer_hk   CHAR(32) NOT NULL REFERENCES hub_customer(customer_hk),
    load_dts      TIMESTAMP NOT NULL,
    record_source VARCHAR(50) NOT NULL,
    name          VARCHAR(100),
    email         VARCHAR(200),
    segment       VARCHAR(50),
    PRIMARY KEY (customer_hk, load_dts)
);
```

**SCD handling:** Each attribute change = new row with new `load_dts`. Querying a point in time: `SELECT ... WHERE load_dts <= target_date ORDER BY load_dts DESC LIMIT 1`.

**Naming:** `sat_<hub/link>_<descriptive_name>` or `S_<HUB/LINK>_<SUFFIX>`.

**Best practices for satellite splitting:**

| Practice | Recommendation |
|----------|---------------|
| **By change rate** | Fast-changers (status, segment) separate from slow-changers (name, DOB) — reduces scan costs |
| **By subject area** | Group attributes by business domain (contact info, financial, preferences) |
| **By source system** | Different sources with different attribute sets → separate satellites |
| **Avoid one-column sats** | Group 3-7 related attributes per satellite; too many creates join issues |
| **Hash diffs** | Use `hash_diff` (MD5 of all descriptive columns) for efficient change detection |

---

## 3. Data Vault 2.0 Innovations Over Original Data Vault

| Feature | Original DV (1.0) | Data Vault 2.0 |
|---------|-------------------|----------------|
| **Primary Keys** | Integer surrogate keys (sequences) | Hash keys (MD5, SHA-2) from business keys |
| **Loading** | Sequential (hubs one at a time) | Parallel (all hubs concurrently) |
| **Business Vault** | Not formalized | Computed rules, best-record logic |
| **PIT & Bridge** | Not standard | Point-in-Time and Bridge tables |
| **Architecture** | Standalone DWH | Hub-and-spoke enterprise architecture |
| **Hashing** | Collision risk accepted | MD5 with SHA-2 option, very low risk |
| **Multi-source** | Manual key mapping | Hash-key-driven automatic consolidation |
| **CDC support** | Full snapshots only | Delta/incremental loads natively supported |

---

## 4. Hash Keys in DV 2.0

### Why Hash Keys?

1. **Parallel Loading** — Hash computed from data, not assigned by a sequence. Multiple processes independently compute the same hash for the same business key without coordination.
2. **Environment Consistency** — Same business key → same hash in dev, test, prod. No key mapping needed.
3. **MPP Distribution Key** — Hash keys distribute naturally across nodes, enabling colocated joins.
4. **Cross-Source Deduplication** — Same business key from different sources produces the same hash.

### Hash Composition

| Component | Formula | Notes |
|-----------|---------|-------|
| **Hub hash** | `MD5(UPPER(TRIM(business_key)))` | Case-insensitive, trimmed |
| **Composite hub hash** | `MD5(UPPER(TRIM(k1))\|'\|'\|UPPER(TRIM(k2)))` | Delimiter separates composite parts |
| **Link hash** | `MD5(SORT(hk1 \|\| hk2 \|\| ...))` | Sorted to avoid ordering issues |
| **Hash diff (satellite)** | `MD5(col1\|'\|'\|col2\|...\|colN)` | For change detection without column comparison |

### Collision Risk
Probability of MD5 collision across N rows ≈ N² / (2 × 2^128). For 10^10 rows: ~2^-64 — negligible. For extreme requirements, use SHA-256 (slower, longer hash).

### NULL Handling
```sql
MD5(UPPER(COALESCE(TRIM(business_key), '')))   -- NULL → hash of empty string
```

### Distribution
Hash keys serve as natural distribution keys for MPP databases (DISTKEY in Redshift, hash partitioning in Snowflake). Ensures even distribution and colocated hub-satellite joins.

### Performance
- MD5 hex: CHAR(32) — human-readable
- MD5 binary: BINARY(16) — half storage, faster comparison
- SHA-256: BINARY(32) — strongest but slowest

---

## 5. Advanced DV 2.0 Concepts

### 5.1 Point-in-Time (PIT) Tables

**Purpose:** Raw DV stores data by load time (`load_dts`), not effective time. Querying "what was the state on date X" requires scanning all satellite rows with window functions. PIT tables pre-compute this.

**Structure:** Contains hub hash key + effective_date + one column per satellite pointing to the applicable `load_dts` (navigational PIT) or the actual attribute values (materialized PIT).

```sql
CREATE TABLE pit_customer (
    customer_hk                   CHAR(32)   NOT NULL,
    effective_dts                 TIMESTAMP  NOT NULL,
    sat_customer_detail_load_dts  TIMESTAMP,   -- FK-like pointer
    sat_customer_credit_load_dts  TIMESTAMP,
    PRIMARY KEY (customer_hk, effective_dts)
);
```

**PIT types:**
- **Navigational** — Pointers to satellite rows; joins to resolve values
- **Materialized (denormalized)** — Contains actual attribute values alongside effective_dts
- **Per-satellite PIT** — One PIT per satellite when they have different cadences

**Refresh:** Typically rebuilt periodically (daily) via drop-and-rebuild or incremental refresh. Some environments generate PIT on-demand via SQL views.

**Key benefit:** Without PIT, BI tools need complex window functions to determine active satellite rows. PIT reduces this to a simple equality join.

### 5.2 Bridge Tables

**Purpose:** Resolve many-to-many relationships for BI tools that expect star-schema-style foreign keys. Without bridges, BI tools double-count measures in many-to-many relationships.

**Use cases:** Customer↔Account (many-to-many), Employee↔Project with allocation percentages, Product↔Category with multiple category membership.

**Types:**
- **Equal bridge** — All grains equally weighted; grain_count helps BI tools avoid double-counting
- **Weighted bridge** — Proportional allocation (employee 60% Project A, 40% Project B)
- **Time-varying bridge** — Effective/expiry dates for changing membership

**Example:**

```sql
CREATE TABLE bridge_customer_account (
    bridge_hk         CHAR(32)   PRIMARY KEY,
    link_cust_acct_hk CHAR(32)   NOT NULL,
    customer_hk       CHAR(32)   NOT NULL,
    account_hk        CHAR(32)   NOT NULL,
    weight_factor     DECIMAL(5,2),
    grain_count       INTEGER,
    effective_dts     TIMESTAMP  NOT NULL,
    expiry_dts        TIMESTAMP
);
```

### 5.3 Business Vault

**Purpose:** Layer between Raw Vault and Presentation Layer containing computed, consolidated, and business-rule-enriched data derived from the Raw Vault.

**Common components:**

| Component | Description | Example |
|-----------|-------------|---------|
| **Best-record satellite** | Merged golden record from multiple sources | Email from CRM if available, else from ERP |
| **Computed satellite** | Calculated metrics | Customer lifetime value, days since last order |
| **MDM satellite** | Master Data Management output | Deduplicated customer after identity resolution |
| **Reference tables** | Small lookup dimensions | Currency codes, country codes, holiday calendars |
| **Aggregated satellite** | Pre-computed aggregates | Monthly customer order totals |

**Timing:** Business Vault loads **after** Raw Vault — it depends on raw data being available.

```sql
-- Example: Best-record customer satellite
INSERT INTO bv_sat_customer_best_record (customer_hk, load_dts, record_source, name, email)
SELECT
    COALESCE(crm.customer_hk, erp.customer_hk),
    CURRENT_TIMESTAMP, 'BEST_RECORD',
    COALESCE(crm.name, erp.name),           -- CRM name preferred
    COALESCE(erp.email, crm.email)          -- ERP email preferred
FROM sat_customer_detail_crm crm
FULL OUTER JOIN sat_customer_detail_erp erp
    ON COALESCE(crm.customer_hk, erp.customer_hk) = COALESCE(erp.customer_hk, crm.customer_hk);
```

---

## 6. Loading Patterns

### 6.1 Standard DV 2.0 Parallel Load Pattern

```
Source → Stage (full snapshot or CDC delta)
  → Load ALL Hubs (parallel — no inter-hub dependencies)
    → Load ALL Links (parallel — depend on hub keys, not on each other)
      → Load ALL Satellites (parallel — depend on hub/link keys)
        → Build Business Vault (sequential, depends on Raw Vault)
          → Generate PIT / Bridge tables
            → Load Presentation Layer (data marts)
```

**Key insight:** Hash keys eliminate sequential key lookups. All hubs load simultaneously, then all links, then all satellites.

### 6.2 Hub Load Pattern

```sql
INSERT INTO hub_customer (customer_hk, customer_id, record_source, load_dts)
SELECT DISTINCT
    MD5(UPPER(TRIM(customer_id))) AS customer_hk,
    customer_id,
    'CRM_SYSTEM' AS record_source,
    CURRENT_TIMESTAMP AS load_dts
FROM stage_customer AS src
WHERE NOT EXISTS (
    SELECT 1 FROM hub_customer h
    WHERE h.customer_hk = MD5(UPPER(TRIM(src.customer_id)))
);
```

### 6.3 Link Load Pattern

```sql
INSERT INTO link_customer_order (link_customer_order_hk, customer_hk, order_hk, record_source, load_dts)
SELECT DISTINCT
    MD5(hc.customer_hk || ho.order_hk) AS link_customer_order_hk,
    hc.customer_hk, ho.order_hk,
    'CRM_SYSTEM', CURRENT_TIMESTAMP
FROM stage_order src
JOIN hub_customer hc ON hc.customer_id = src.customer_id
JOIN hub_order ho    ON ho.order_id    = src.order_id
WHERE NOT EXISTS (
    SELECT 1 FROM link_customer_order l
    WHERE l.link_customer_order_hk = MD5(hc.customer_hk || ho.order_hk)
);
```

### 6.4 Satellite Load Pattern (Delta Detection)

**Method 1 — Column comparison (simpler, slower for wide tables):**

```sql
INSERT INTO sat_customer_detail (customer_hk, load_dts, record_source, name, email, segment)
SELECT h.customer_hk, CURRENT_TIMESTAMP, 'CRM_SYSTEM',
       src.name, src.email, src.segment
FROM stage_customer src
JOIN hub_customer h ON h.customer_id = src.customer_id
WHERE NOT EXISTS (
    SELECT 1 FROM sat_customer_detail s
    WHERE s.customer_hk = h.customer_hk
      AND s.name = src.name AND s.email = src.email AND s.segment = src.segment
      AND s.load_dts = (SELECT MAX(load_dts) FROM sat_customer_detail
                         WHERE customer_hk = h.customer_hk)
);
```

**Method 2 — Hash diff (preferred for wide tables):**

Pre-compute `hash_diff = MD5(COALESCE(name,'')||'|'||COALESCE(email,'')||'|'||COALESCE(segment,''))`. Compare only `hash_diff` between incoming data and the most recent satellite row. Insert only when different.

**Method 3 — DELETE end-dating:**

For tracking source-side deletions, use `load_end_dts`. When a source record disappears, set `load_end_dts = CURRENT_TIMESTAMP` on the most recent satellite row (soft delete). Then regular delta detection handles new inserts.

---

## 7. When to Use Data Vault vs Kimball vs Inmon

| Factor | Data Vault 2.0 | Kimball (Star Schema) | Inmon (3NF) |
|--------|---------------|----------------------|-------------|
| **Audit Trail** | ✅ Excellent (full history) | ⚠️ Partial (SCD Type 2) | ✅ Good |
| **Query Performance** | ⚠️ Requires PIT/Bridge | ✅ Excellent (star join) | ❌ Slower |
| **Loading Speed** | ✅ Parallel, fast | ⚠️ Sequential dim loads | ❌ Complex |
| **Source Changes** | ✅ Handles gracefully | ⚠️ Requires dim rebuild | ❌ Full remodelling |
| **Business User Understanding** | ❌ Complex | ✅ Intuitive | ❌ Complex |
| **Implementation Time** | Longer initially | Faster initially | Moderate |
| **Best For** | Multi-source enterprise, regulated industries, agile DW | Departmental marts, BI self-service, mid-market | Enterprise 3NF, existing Inmon shops |

### Hybrid Approach
Many organizations use **DV 2.0 for enterprise integration** and **Kimball star schemas for the presentation layer** — getting audit trail + agility + user-friendly querying:

```
Source → Stage → Raw Vault → Business Vault → PIT/Bridge → Star Schema Marts
```

---

## 8. Common Pitfalls

### Over-modelling
**Problem:** Creating a link for every foreign key rather than every meaningful business relationship.
**Fix:** Ask "Does this relationship have its own descriptive attributes that change over time?" If not, consider placing it as an attribute in a satellite.

### Hash Collisions
**Problem:** Assuming MD5 collisions can never happen.
**Fix:** Monitor periodically: `SELECT customer_hk FROM hub_customer GROUP BY customer_hk HAVING COUNT(*) > 1`. For petabyte-scale datasets, consider SHA-256.

### Satellite Proliferation
**Problem:** One satellite per column ("mono-attribute satellites") creates join hell.
**Fix:** Group 3-7 related attributes. Split by rate of change (fast vs slow) and subject area.

### Missing PIT Tables
**Problem:** Exposing raw DV to BI tools. Complex window functions make queries near-impossible for business users.
**Fix:** Generate PIT and Bridge tables early. Treat them as non-negotiable components.

### Ignoring Business Vault
**Problem:** Raw Vault alone doesn't handle data quality, deduplication, or survivorship.
**Fix:** Plan Business Vault from day one — survivorship rules, golden records, and computed measures.

### Hash Key Distribution Skew
**Problem:** NULL-heavy or low-cardinality business keys cause uneven distribution in MPP.
**Fix:** COALESCE for NULLs. Consider composite business keys or alternate distribution strategies for low-cardinality keys.

### Underestimating Storage
**Problem:** Planning based on Kimball/Inmon experience. DV stores 2-3× more data (all history, all relationships).
**Fix:** Plan for 2-3× the storage of a star schema. Implement data lifecycle management if costs are constrained.

### No Source-to-Target Documentation
**Problem:** Without documentation, mappings become tribal knowledge.
**Fix:** Maintain source-to-target mapping in dbt YAML, data catalog, or metadata repository.

### Hash Key Platform Mismatch
**Problem:** Different platforms (Oracle MD5 vs Python hashlib) produce different hashes for the same input.
**Fix:** Standardize `UPPER(TRIM(COALESCE(..., '')))` before hashing. Verify hash alignment across environments with test data.

### Neglecting Reference Data
**Problem:** Small lookup tables modeled as hubs with satellites create excessive join overhead.
**Fix:** Model static reference data as reference tables in the Business Vault, not as hubs.

---

## 9. Tooling

| Tool | Type | Purpose |
|------|------|---------|
| **dbt + dbtvault** | Open-source | ELT-based DV modelling with hub/link/sat macros, hash diff detection, PIT/Bridge generation |
| **Snowflake** | Cloud DW | Native hash functions, zero-copy cloning for PIT rebuilds, automatic clustering on hash keys |
| **BigQuery** | Cloud DW | Serverless, excellent for large satellite scans |
| **Redshift** | Cloud DW | DISTKEY on hash keys for collocated joins |
| **Databricks (Delta Lake)** | Lakehouse | Z-ordering on hash keys, Delta time travel complements DV audit trail |
| **WhereScape RED** | Commercial | Automated DV generation, code-first, multi-DB support |
| **Data Vault Automation (DVA)** | Commercial | Full lifecycle DV automation from Datavault Architecture |
| **RapidAccelerator** | Commercial | DV2.0 automation from Genesee Academy |
| **VaultSpeed** | Commercial | Automated DV2.0 modelling, metadata-driven, cloud + on-prem |
| **Python/Pandas** | Custom | Full control for smaller datasets or proofs-of-concept |

**Example (Python hash key generation):**
```python
import hashlib
def compute_hub_key(business_key: str) -> str:
    normalized = str(business_key).strip().upper() if business_key else ''
    return hashlib.md5(normalized.encode('utf-8')).hexdigest()
```

---

## 10. Architecture and Layering

```
┌─────────────────────────────────────────────────┐
│              Source Systems                      │
│  (CRM, ERP, Legacy, APIs, External Feeds)        │
└────────────────────┬────────────────────────────┘
                     │ (CDC or full snapshot)
                     ▼
┌─────────────────────────────────────────────────┐
│              Staging Area                        │
│  (temporary, no history, minimal transformations) │
└────────────────────┬────────────────────────────┘
                     │ (parallel load)
                     ▼
┌─────────────────────────────────────────────────┐
│           Raw Data Vault                         │
│   ┌────────┐  ┌────────┐  ┌────────────┐       │
│   │ Hubs   │  │ Links  │  │ Satellites │       │
│   │(insert │  │(insert │  │ (insert    │       │
│   │ only)  │  │ only)  │  │  only)     │       │
│   └────────┘  └────────┘  └────────────┘       │
│   Full audit trail: every row = record_source   │
│   + load_dts. No updates, no deletes.          │
└────────────────────┬────────────────────────────┘
                     │ (computation)
                     ▼
┌─────────────────────────────────────────────────┐
│           Business Vault                         │
│  Best-record sats, computed sats, MDM,          │
│  reference tables, aggregated sats              │
│  (depends on Raw Vault being available)         │
└────────────────────┬────────────────────────────┘
                     │ (materialization)
                     ▼
┌─────────────────────────────────────────────────┐
│         PIT / Bridge Layer                       │
│  PIT tables (time-travel queries)               │
│  Bridge tables (many-to-many resolution)        │
│  Generated periodically (daily) or on-demand    │
└────────────────────┬────────────────────────────┘
                     │ (mapping)
                     ▼
┌─────────────────────────────────────────────────┐
│         Presentation Layer (Data Marts)          │
│  Star schemas, aggregated views, BI tool access  │
└─────────────────────────────────────────────────┘
```

---

## 11. Practical Modeling Process

### Step 1: Identify Business Concepts → Hubs
Walk through source systems and identify core business nouns (Customer, Product, Order, Employee, Store, Supplier, Invoice). Each becomes a hub with its natural/business key(s). Same concept across sources → same hub.

### Step 2: Identify Business Relationships → Links
For each pair/group of hubs, ask: "Do these entities have a business relationship that could change over time?" E.g., Customer places Order → `link_customer_order`. Product belongs to Category → `link_product_category`.

### Step 3: Identify Descriptive Attributes → Satellites
Collect all descriptive attributes for each hub and link. Group by:
- **Rate of change:** Fast (status, segment) separate from slow (name, SKU)
- **Source system:** Different sources with different attribute sets
- **Subject area:** Contact info, financial details, compliance flags

### Step 4: Design Business Vault
Define best-record rules (which source is authoritative for which attribute), computed measures, and MDM rules.

### Step 5: Design PIT and Bridge Tables
Identify which hubs need PIT (typically all major entities) and which links need bridges (typically many-to-many relationships BI tools query).

### Step 6: Implement Loading Pipelines
Build hub loaders (parallel) → link loaders (parallel, after hubs) → satellite loaders (parallel, hash-diff-based) → Business Vault → PIT/Bridge materialization → data marts.

---

## 12. References

### Books
- **Daniel Linstedt & Michael Olschimke** — *"Building a Scalable Data Warehouse with Data Vault 2.0"* (2015, Morgan Kaufmann) — The definitive reference
- **Dan Linstedt** — *"Data Vault 2.0: Getting Started"* (2019, free eBook from Data Vault Alliance)

### Online Resources
- **Data Vault Alliance** — datavaultalliance.com — Official standards and certification
- **Data Vault Community Forum** — forum.data-community.org — Active DV2.0 discussion
- **dbtvault Documentation** — dbtvault.readthedocs.io — dbt Data Vault automation package
- **Scalefree Knowledge Base** — scalefree.com/knowledge — Tutorials and webinars
- **Genesee Academy** — geneseeacademy.com — Training and certification

### Key Standards
- Data Vault 2.0 Standards (Data Vault Alliance)
- DV 2.0 Hash Key Standards (MD5 implementation specification)
- DV 2.0 Modeling Conventions (Naming and structural standards)

### Community
- **Reddit:** r/dataengineering — Frequent DV2.0 discussions
- **LinkedIn:** Data Vault 2.0 group — Professional community and case studies
- **Medium:** Implementation tutorials for dbt, Snowflake, and Databricks

---

> **Note:** Data Vault 2.0 is a mature, standards-driven methodology. For production implementations, consult official DV2.0 standards from Data Vault Alliance. The methodology's strengths — auditability, agility, scalability — come with a complexity cost requiring disciplined implementation and strong SQL skills.
