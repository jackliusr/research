# Types of Dimensions in Data Warehousing

A comprehensive reference on dimension types in dimensional modelling, covering definitions, implementation patterns, use cases, and trade-offs for each type.

---

## 1. Introduction

### What Is a Dimension in Dimensional Modelling?

In the Kimball methodology, a **dimension** provides the *context* surrounding a business process — the who, what, where, when, and why of a business event. Dimensions contain descriptive attributes that serve as the entry points for slicing, filtering, and grouping measures in analytical queries. A dimension table typically has a surrogate key (system-generated integer primary key), one or more natural/business keys, and descriptive attributes (text fields describing the dimension member).

### Dimensions vs Facts

| Aspect | Dimension | Fact |
|--------|-----------|------|
| **Purpose** | Context (who, what, where, when, why) | Measures the business event |
| **Content** | Descriptive text, codes, categories | Numeric measures |
| **Granularity** | One row per dimension member | One row per business event |
| **Update pattern** | Slowly changing (SCD) | Append-only |

### Role in Star and Snowflake Schemas

- **Star schema**: Fact table at the centre, surrounded by denormalised dimensions. Simple and query-performant.
- **Snowflake schema**: Normalised dimensions with sub-dimensions branching off. Kimball recommends against snowflaking — it adds join complexity without meaningful savings in modern columnar warehouses.

### Dimension Attributes

Attributes are descriptive text fields characterising a dimension member. For `Dim_Customer`: `customer_name`, `segment`, `region`, `channel`, `credit_tier`. Analysts filter on these (`WHERE region = 'APAC'`) and group by them (`GROUP BY segment`).

---

## 2. Conformed Dimensions

### Definition

A **conformed dimension** is identical (or compatible) across multiple fact tables, data marts, or business process areas. Conformed dimensions are the backbone of the Kimball **bus architecture**, enabling enterprise-wide integration.

### Key Characteristics

- Same dimension table used by two or more fact tables
- Same surrogate key values point to the same business entity
- Same attribute names and meanings across all consumers
- Same granularity in the conformed version

### Examples

| Dimension | Shared Across |
|-----------|--------------|
| `Dim_Date` | Sales fact, Inventory fact, HR fact, Support fact |
| `Dim_Customer` | Sales fact, Support ticket fact, Marketing fact |
| `Dim_Product` | Sales fact, Inventory fact, Procurement fact |

### Why They Matter

Conformed dimensions enable **drill-across** reporting — comparing metrics from different business processes on the same dimensional axis. For example: *"Show me total sales and total support tickets for the same customers across the same time period."* Without conformed dimensions, each fact table would have incompatible dimensional references, breaking this query.

### Implementation Approaches

- **Identical (shared physical table)**: One physical dimension table used by all facts. The most common and recommended approach.
- **Shrunken (subset)**: A dimension containing a subset of rows (different grain) or attributes from the base dimension. Used when a fact table operates at a higher grain.

### Conformed vs Non-Conformed

A non-conformed dimension may share the name `Dim_Customer` across data marts but with different surrogate keys, attributes, or incompatible values — breaking drill-across.

---

## 3. Slowly Changing Dimensions (SCD)

SCD strategies address how dimension attributes change over time. Kimball defined Types 0–7.

### SCD Type 0 — Retain Original

Never modify the dimension row. Original values preserved forever. **Use for**: Truly immutable attributes — original creation date, first-ever customer ID. *Example: `original_signup_date` should never be overwritten.*

### SCD Type 1 — Overwrite

Update the attribute value in place. No history kept. **Use for**: Corrections and attributes where historical accuracy is irrelevant. *Example: fixing a typo in a customer's email address.* Simple but breaks point-in-time reporting — a report on last year's data shows this year's value.

### SCD Type 2 — Add New Row (Full History)

Insert a new row with a new surrogate key on attribute change. The old row is retained. **Use for**: Attributes where historical reporting matters — customer segment, employee department, product category.

**Additional columns**: `effective_date`, `expiration_date` (nullable for current), `current_flag` (Y/N), `version_number`.

```sql
UPDATE dim_customer SET expiration_date = '2025-06-01', current_flag = 'N'
WHERE customer_nk = 'CUST001' AND current_flag = 'Y';

INSERT INTO dim_customer (customer_sk, customer_nk, name, segment, effective_date, current_flag)
VALUES (45291, 'CUST001', 'Acme Corp', 'Gold', '2025-06-01', 'Y');
```

**Pros**: Full audit trail, accurate point-in-time reporting (`WHERE effective_date <= dt AND (expiration_date IS NULL OR expiration_date > dt)`). **Cons**: Table grows over time; join queries must filter by date.

### SCD Type 3 — Add Previous Value Column

Store the previous value alongside the current value in the same row. **Use for**: Simple before-and-after comparisons. *Example: `Dim_Customer` with `current_segment` and `previous_segment` columns.*

```sql
UPDATE dim_customer
SET previous_segment = current_segment, current_segment = 'Gold'
WHERE customer_sk = 12345;
```

**Pros**: No new rows or tables; very easy to compare current vs previous. **Cons**: Only stores one version of change; not suitable for regulatory/historical reporting.

### SCD Type 4 — History Table

Current values in the main dimension; full history in a separate table. **Use for**: High-velocity attributes (credit score, risk rating).

### SCD Type 5 — Mini-Dimension + Type 1/2 Hybrid

Rapidly changing attributes extracted into a separate mini-dimension with its own tracking. The fact table stores two foreign keys — one to the base dimension, one to the mini-dimension.

### SCD Type 6 — Hybrid of Types 1, 2, and 3

Combines Type 2 (historical rows), Type 1 (current value overwritten in all rows), and Type 3 (previous value column). Every row carries the current value, historical effective dates, and a previous value column.

### SCD Type 7 — Dual Current/History Access

Hybrid of Types 1 and 2. Current value (Type 1 overwrite) and historical rows (Type 2) coexist; a flag or dual-key design lets queries choose current-only or point-in-time access.

---

## 4. Role-Playing Dimensions

### Definition

A single physical dimension table used in multiple roles within the same fact table. The same table is joined to the fact via different foreign key columns, each representing a different contextual role.

### Examples

| Physical Dimension | Role 1 | Role 2 | Role 3 |
|-------------------|--------|--------|--------|
| `Dim_Date` | Order Date | Ship Date | Delivery Date |
| `Dim_Employee` | Sales Rep | Manager | Support Agent |
| `Dim_Location` | Shipping Address | Billing Address | Warehouse |

### Implementation

Create database **views** or **aliases** that reference the same physical table, avoiding data duplication while providing semantic clarity.

```sql
CREATE VIEW vw_order_date AS SELECT * FROM dim_date;
CREATE VIEW vw_ship_date  AS SELECT * FROM dim_date;

CREATE TABLE fact_sales (
  order_sk INT REFERENCES vw_order_date(date_sk),
  ship_sk  INT REFERENCES vw_ship_date(date_sk)
);
```

In the star schema, the fact table has multiple foreign keys referencing the same dimension:

```
fact_sales
├── order_date_sk    ──→ dim_date (as "Order Date")
├── ship_date_sk     ──→ dim_date (as "Ship Date")
└── delivery_date_sk ──→ dim_date (as "Delivery Date")
```

### When to Use

Use when the same entity type plays multiple distinct contextual roles and attributes are identical across roles. If different roles need different attributes, create separate dimensions.

---

## 5. Junk Dimensions

### Definition

A **junk dimension** combines multiple low-cardinality flags, indicators, and miscellaneous attributes into a single table. Rather than creating many tiny dimensions or bloating the fact table with individual flag columns, these attributes are grouped together.

### When to Use

When you have 3–20 small flag/indicator columns that are low cardinality (2–50 values each), not descriptive enough for their own dimension, and don't naturally belong in a conformed dimension.

### Examples of Junk Attributes

| Attribute | Possible Values |
|-----------|----------------|
| `is_expedited` | Y, N |
| `is_gift` | Y, N |
| `payment_type` | Credit Card, PayPal, Wire Transfer |
| `channel` | Web, Mobile, In-Store |
| `shipping_tier` | Standard, Express, Overnight |

### How to Create

1. Collect all flag/indicator columns from the source system
2. Find the distinct combinations of all values
3. Assign a surrogate key to each unique combination
4. Store the junk dimension surrogate key in the fact table

```sql
CREATE TABLE dim_junk_order (
  junk_sk       INT PRIMARY KEY,
  is_expedited  CHAR(1),
  is_gift       CHAR(1),
  payment_type  VARCHAR(20),
  channel       VARCHAR(20)
);

INSERT INTO dim_junk_order VALUES
(1, 'N', 'N', 'Credit Card', 'Web'),
(2, 'Y', 'N', 'Credit Card', 'Web');
```

### Benefits

- **Eliminates snowflaking tiny tables**: Consolidates flags into one table instead of several
- **Reduces fact table width**: One `junk_sk` column instead of 10 flag columns
- **Improves query performance**: Fewer joins, narrower fact table
- **Simplifies ETL**: One dimension to manage instead of many

### Potential Drawback

The junk dimension can grow large if the product of cardinalities of all included attributes is high. Keep it to low-cardinality attributes only.

---

## 6. Degenerate Dimensions

### Definition

A **degenerate dimension** is a dimension key stored directly in the fact table with no corresponding dimension table. It is a business identifier (order number, invoice number, ticket number) that has no additional descriptive attributes because those have been absorbed into other dimensions or the fact.

### When to Use

Use for transaction/document header numbers where:
- Header-level attributes (customer, date, shipping address) have moved to proper dimensions
- Line-item attributes (product, quantity, price) are in the fact table
- Only the header number itself remains — no other descriptive fields need their own dimension

### Examples

| Degenerate Dimension | In the Fact Table |
|---------------------|-------------------|
| `order_number` | `fact_order_lineitems` |
| `invoice_number` | `fact_invoice_lines` |
| `ticket_id` | `fact_ticket_activities` |
| `transaction_ref` | `fact_transaction_ledger` |

### Implementation

```sql
CREATE TABLE fact_order_lineitems (
  order_sk      INT,          -- FK to dim_order if header has descriptive attributes
  order_number  VARCHAR(20),  -- Degenerate dimension — no dim_order_number table
  product_sk    INT,          -- FK to dim_product
  customer_sk   INT,          -- FK to dim_customer
  date_sk       INT,          -- FK to dim_date
  quantity      INT,
  unit_price    DECIMAL(10,2),
  line_total    DECIMAL(10,2)
);
```

### Not to Be Confused With

- **Surrogate keys**: System-generated integers that join to dimension tables. Degenerate dimensions are business-visible identifiers stored directly in the fact.
- **Fact attributes**: Numeric measures. Degenerate dimensions are identifiers, not measures.

### Why "Degenerate"?

The dimension has "degenerated" — it lost its attribute table because all descriptive content moved to other dimensions. What remains is just the identifier.

---

## 7. Outrigger Dimensions

### Definition

An **outrigger dimension** branches off another dimension table, creating a secondary join path from the dimension rather than from the fact. This is the characteristic pattern of a snowflake schema.

### Example

`dim_product_category` is an outrigger of `dim_product`: `dim_product.category_sk` → `dim_product_category.category_sk`. The category dimension holds attributes like `category_name` and `category_manager` that change independently of the product attributes.

### When to Use

- When a dimension attribute has its own complex sub-attributes that change independently
- When the outrigger is a conformed dimension consumed by multiple dimension tables
- When storage normalisation is a genuine concern (rare in modern MPP warehouses)

### The Kimball Debate

Kimball generally **discourages** outriggers and snowflake schemas: (1) every outrigger adds a join hop, slowing queries; (2) BI tools must navigate multi-hop paths; (3) modern columnar databases compress denormalised dimensions efficiently, making storage normalisation savings negligible.

### Recommended Alternative

Flatten outrigger attributes into the main dimension. Instead of a separate `dim_product_category`, include `category_name` and `category_manager` in `dim_product`. When the outrigger is a conformed dimension used across the enterprise (e.g., `dim_date`), keep it separate — but join it from the fact, not from another dimension.

---

## 8. Shrunken Dimensions

### Definition

A **shrunken dimension** is a subset of a base dimension — fewer rows (different grain) or fewer attributes. Used when a fact table operates at a higher level of granularity than the base dimension.

### Example

| Dimension | Grain | Rows |
|-----------|-------|------|
| `Dim_Product` | Individual product SKU | 50,000 |
| `Dim_Product_Category` (shrunken) | Category | 200 |

A fact table at category grain (`fact_category_sales`) uses the shrunken `Dim_Product_Category`, not the full `Dim_Product`.

### Relationship to Base Dimension

- Shrunken dimensions share the same surrogate key values as the base dimension for matching rows
- Queries that need to drill from category-level facts to product-level facts can join through the shrunken dimension to the base dimension
- A shrunken dimension is considered **conformed** — compatible with the base dimension for drill-across

### When to Use

- When you have fact tables at different grains (e.g., daily product sales vs monthly category sales)
- When you need conformed dimensions but the base dimension is too detailed for a higher-grain fact table
- When you want to reduce the size of dimensions joined to aggregate fact tables

---

## 9. Static Dimensions

### Definition

A **static dimension** contains pre-defined, unchanging attributes loaded once and never modified. Generated from external reference data or calculated attributes that don't depend on business operations.

### Examples

| Dimension | Content |
|-----------|---------|
| `Dim_Date` | Every date from 2000 to 2030 with pre-calculated attributes |
| `Dim_Time` | Every second of the day with time-of-day attributes |
| `Dim_Holiday_Calendar` | Holiday dates by country |
| `Dim_Currency` | Currency codes, names, symbols |

### Date Dimension Benefits

Pre-populating `dim_date` gives analysts rich time-based attributes: `calendar_date`, `year`, `month`, `month_name`, `day_of_week`, `is_weekend`, `is_holiday`, `quarter`, `fiscal_year`, `fiscal_quarter`, `fiscal_period`, `day_of_year`, `week_of_year`.

```sql
CREATE TABLE dim_date (
  date_sk          INT PRIMARY KEY,
  calendar_date    DATE UNIQUE,
  year             SMALLINT,
  month            SMALLINT,
  month_name       VARCHAR(20),
  day_of_week      VARCHAR(10),
  is_weekend       CHAR(1),
  is_holiday       CHAR(1),
  quarter          CHAR(2),
  fiscal_year      SMALLINT,
  fiscal_quarter   CHAR(2)
);
```

Populated once (or annually for new dates) and never updated in place.

---

## 10. Monster Dimensions

### Definition

A **monster dimension** is a very large dimension table with millions to billions of rows. These present unique performance and management challenges.

### Examples

| Scenario | Estimated Rows |
|----------|---------------|
| `Dim_Customer` for a large bank | 100M+ |
| `Dim_Product` for Amazon-scale retailer | 500M+ |
| `Dim_Household` for national census | 100M+ |

### Strategies for Handling

| Strategy | Description |
|----------|-------------|
| **Partitioning by natural grouping** | Partition by industry sector, geographic region |
| **Active vs historical partitioning** | Frequently queried rows in an "active" subset; rarely accessed rows in "historical" |
| **Columnstore indexes** | Columnar storage for efficient compression and scan performance |
| **Bitmap indexes** | On low-cardinality columns for fast filtering |
| **Aggregate fact tables** | Pre-aggregate facts at higher grains to reduce joins to monster dimensions |
| **Vertical splitting** | Separate frequently used columns from rarely used ones, linked by surrogate key |
| **Subtype dimensions** | Split into logical subtypes (e.g., `Dim_Individual_Customer` + `Dim_Business_Customer`) |

### Key Consideration

Monster dimensions are often unavoidable (e.g., a bank has 100M customers). The goal is to design access patterns and storage strategies that minimise their query impact. Conformed monster dimensions — once built and optimised — serve the entire enterprise.

---

## 11. Mini-Dimensions (Rapidly Changing Dimensions)

### Definition

A **mini-dimension** is a separate dimension extracted from a base dimension to handle attributes that change frequently. The base dimension retains stable attributes; the mini-dimension captures rapidly changing ones.

### When to Use

When a dimension has attributes that change daily or weekly, using SCD Type 2 on the base dimension would cause it to grow excessively. For example: `credit_score` changing monthly for millions of customers. Adding 12 rows/year per customer would inflate a 100M-row dimension to 1.2B rows.

### Example

```
Dim_Customer (stable)              MiniDim_Credit_Profile (rapidly changing)
├── customer_sk (PK)               ├── credit_profile_sk (PK)
├── name                           ├── credit_score
├── date_of_birth                  ├── risk_rating
└── gender                         ├── LTV_tier
                                   ├── effective_date
                                   ├── expiration_date
                                   └── current_flag

fact_loan_application
├── customer_sk (FK → Dim_Customer)
├── credit_profile_sk (FK → MiniDim_Credit_Profile)
├── loan_amount
└── ...
```

### How It Works

The fact table stores both keys. At load time, the system looks up which mini-dimension row was active for the transaction date and stores the appropriate key. This enables point-in-time analysis without exploding the base dimension.

### Pros vs Cons

| Pros | Cons |
|------|------|
| Prevents base dimension explosion from SCD Type 2 | Requires join to mini-dimension for rapidly changing attributes |
| Accurate point-in-time analysis | More complex ETL — fact load must resolve the correct mini-dimension key |

—

## 12. Parent/Child Dimensions (Hierarchy Dimensions)

### Definition

Dimensions with recursive relationships — employees with managers, account hierarchies, organisational structures, bill-of-materials. Rows relate to other rows in the same table.

### Implementation Approaches

#### Self-Referencing Foreign Key

The dimension includes a column referencing its own primary key. Simplest approach.

```sql
CREATE TABLE dim_employee (
  employee_sk   INT PRIMARY KEY,
  employee_name VARCHAR(100),
  supervisor_sk INT REFERENCES dim_employee(employee_sk)
);
```

**Use when**: Fixed-depth hierarchy, every node has one parent, simple traversal.

#### Bridge Table (Kimball Pattern)

Expands the recursive hierarchy into a flat, pre-joined structure.

```sql
CREATE TABLE bridge_employee_hierarchy (
  ancestor_sk    INT REFERENCES dim_employee(employee_sk),
  descendant_sk  INT REFERENCES dim_employee(employee_sk),
  depth          INT,    -- 0=self, 1=direct report, etc.
  is_top         CHAR(1) -- Y for root ancestor
);
```

**Use when**: Ragged hierarchies (varying depth per branch), multi-parent hierarchies, or "all descendants of this manager" queries.

#### Materialized Path

Store the full path from root to node as a delimited string: `"1.4.12.8"`. Fast subtree queries (`LIKE '1.4.%'`), good for variable-depth hierarchies with low write volume.

#### Nested Sets

Each node has left/right values for its tree position. Fast descendant queries (`WHERE lft BETWEEN parent.lft AND parent.rgt`) when the hierarchy is relatively static.

### Kimball Bridge Table for Multi-Parent Hierarchies

For multi-parent hierarchies (employee reports to two managers; account rolls up to multiple regions), use a bridge table with one row per ancestor/descendant pair, plus `depth`, `is_top`, and optional `weight` for proportional allocation across parents.

```
bridge_org_hierarchy
├── ancestor_sk (FK → dim_org_unit)
├── descendant_sk (FK → dim_org_unit)
├── depth (INT)
├── is_top (CHAR(1))
└── weight (DECIMAL(3,2))
```

---

## 13. Comparison: When to Use Which Dimension Type

| Dimension Type | Use When | Don't Use When |
|---------------|----------|----------------|
| **Conformed** | Same attribute needed across multiple fact tables | Different grains or incompatible attribute definitions |
| **SCD Type 1** | Error correction, history-irrelevant attributes | Regulatory/historical reporting; audit trails |
| **SCD Type 2** | Full audit trail, "as-was" reporting | High-velocity attributes (>100 changes/day) |
| **SCD Type 3** | Simple before-and-after comparison | More than one change to track; complete history needed |
| **Role-Playing** | Same entity plays multiple roles in one fact | Each role has completely different attributes |
| **Junk** | 3–20 low-cardinality flags/indicators | High-cardinality codes (use deg or regular dim) |
| **Degenerate** | Transaction number with no descriptive attributes | Transaction number with many descriptive fields (needs a dim) |
| **Shrunken** | Fact at higher grain needs conformed dimension | Fact at same grain (use the full dimension) |
| **Static** | Date, time, currency, calendar | Mutable attributes (use SCD Type 1/2) |
| **Mini-Dimension** | Rapidly changing attributes in large dimension | Slowly changing or stable attributes |
| **Outrigger** | Dimension attribute has many sub-attributes | Simple 1-attribute grouping (flatten instead) |
| **Monster** | Inevitably large dimensions — use partitioning/indices | Dimensions that can be split into subtypes |
| **Parent/Child** | Recursive relationships (org chart, account hierarchy) | Fixed-depth hierarchies (model as regular attributes) |

---

## 14. Data Sources and Further Reading

### Foundational Books

- **Kimball, Ralph; Ross, Margy** — *The Data Warehouse Toolkit* (3rd Ed.). Wiley, 2013. The canonical reference for dimensional modelling.
- **Kimball, Ralph; Caserta, Joe** — *The Data Warehouse ETL Toolkit*. Wiley, 2004. ETL patterns for SCD loading and dimension management.
- **Linstedt, Dan; Olschimke, Michael** — *Building a Scalable Data Warehouse with Data Vault 2.0*. Alternative modelling with hub-link-satellite patterns.

### Online Resources

- **Kimball Group** (https://www.kimballgroup.com) — Articles and columns on dimensional modelling, SCD strategies, junk/deg dimensions, and star schema design.
- **Microsoft** — Azure Synapse Analytics / SSIS dimension documentation, including SCD wizard and best practices.
- **Oracle** — Oracle Data Warehousing Guide: dimensional design patterns and star query optimisation.
- **Snowflake** — "Designing a Star Schema": clustering keys on monster dimensions, query optimisation.
- **Amazon Redshift** — "Designing tables": distribution keys, sort keys, and compression for dimension tables.
- **Google BigQuery** — "Star schema and dimensional modelling": clustering, partitioning, and denormalisation.

### Industry Patterns

- **Stitch Fix** — "Multidimensional Data Modeling" series: real-world dimension design at scale.
- **Airbnb Engineering** — "Data Modeling at Airbnb": marketplace dimension design with rapidly changing attributes.
- **Netflix Tech Blog** — Dimensional modelling at Netflix: monster dimensions and mini-dimension patterns.

---

*This guide is a standalone reference on dimension types in data warehousing, following the Kimball dimensional modelling methodology.*
