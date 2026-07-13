# Data Profiling: A Comprehensive Guide
## Table of Contents

1. [What is Data Profiling?](#1-what-is-data-profiling)
2. [Three Types of Data Profiling](#2-three-types-of-data-profiling) — 2.1 Structure Discovery · 2.2 Content Discovery · 2.3 Metadata
3. [Data Profiling Dimensions](#3-data-profiling-dimensions-what-to-measure)
4. [Data Profiling Process](#4-data-profiling-process)
5. [Data Profiling Tools](#5-data-profiling-tools) — 5.1 Open Source · 5.2 Commercial
6. [Sampling Strategies](#6-sampling-strategies-for-large-datasets)
7. [Profiling for Specific Use Cases](#7-profiling-for-specific-use-cases)
8. [Integration with Data Pipelines](#8-integration-with-data-pipelines)
9. [Tool Comparison Table](#9-comparison-table)
10. [Best Practices](#10-best-practices)

---

## 1. What is Data Profiling?

### Definition

**Data profiling** is the systematic process of examining source data to understand its structure, content, quality, and relationships.

Data profiling answers foundational questions about a dataset:

- **What is actually in this data?** (content), **how is it organized?** (structure), **how do elements relate?** (relationships)

### Key Distinctions

- **Data profiling ≠ Data quality monitoring.** Profiling is exploratory discovery; monitoring is ongoing operational measurement against thresholds. Profiling asks "what does this look like?"; monitoring asks "is it meeting our standards today?"
- **Data profiling ≠ Data cleansing.** Profiling identifies issues (nulls, duplicates, format violations). Cleansing corrects or removes them. Typical workflow: profile → identify → cleanse → re-profile.

### When Does Data Profiling Happen?

- **Before data warehouse design:** Source data granularity, key structures, and data types inform dimensional modeling.
- **Before data migration:** Profiling legacy systems reveals quality issues that must be addressed before moving to a new platform, and identifies transformation and mapping requirements.
- **During data integration projects:** When combining data from multiple sources, profiling each independently and then the merged dataset uncovers format inconsistencies, domain mismatches, and reference data conflicts.
- **As part of data governance programs:** Profiling establishes baseline quality metrics, enables data catalog enrichment, and informs data stewardship priorities.
- **Before machine learning model development:** Feature understanding, missing value patterns, and distribution analysis feed directly into preprocessing pipeline design.
- **During system onboarding to a data lake:** Profiling detects schema drift, handles semi-structured data (JSON, Avro, Parquet), and guides schema-on-read strategies.

---
## 2. Three Types of Data Profiling

Each examines a different facet of the data, and together they provide a complete picture.

### 2.1 Structure Discovery (Column Profiling)

**What it is:** Examines individual columns (attributes) to understand their intrinsic characteristics. This is the most common form of profiling and provides a foundational statistical summary of every column.

**Key metrics computed for each column:**

| Metric | Description | Example Output |
|--------|-------------|----------------|
| **Data type** | Inferred vs. declared type | `string`, `integer`, `date`, `boolean` |
| **Length** | Min, max, mean field length | `min=2, max=50, mean=18.3` |
| **Null count / null %** | Missing values and their proportion | `nulls=1,234 (4.7%)` |
| **Distinct values** | Number of unique values | `distinct=892` |
| **Cardinality** | Distinct count relative to total rows | `cardinality=0.089` |
| **Min / Max** | Range boundaries | `min=0.0, max=999,999.99` |
| **Mean / Median / StdDev** | Central tendency and spread | `mean=245.6, median=120.0, std=1,023.4` |
| **Frequency distribution** | Value counts and percentages | `{'Active': 45%, 'Inactive': 30%, 'Pending': 25%}` |
| **Top-K values** | Most common distinct values | `top_5: ['Active', 'Inactive', 'Pending', 'Suspended', 'Closed']` |

**Pattern discovery:** Structure discovery identifies common patterns in string columns using regex matching. A column with values like `john@example.com` and `jane@corp.org` is classified as email. Phone numbers, SSNs, dates, ZIP codes, and credit cards are similarly detected through pattern frequency analysis, which reports the dominant pattern and the percentage of values that deviate from it.

**Output:** A **column profile report** — a structured document showing per-column statistics, value distributions (histograms), pattern summaries, and quality flags ("high null rate", "suspected PII", "unexpected data type").

### 2.2 Content Discovery (Relationship Profiling)

**What it is:** Examines relationships between columns — within a single table and across tables. Answers questions like "Can column A in Table 1 serve as a foreign key to column B in Table 2?" and "Which columns are functionally dependent on which others?"

**Key analyses and their significance:**

| Analysis | What It Finds | Why It Matters |
|----------|---------------|----------------|
| **Primary key candidates** | Columns (or combinations) with high uniqueness | Identifying natural keys vs. surrogate keys; understanding table grain |
| **Foreign key candidates** | Cross-table value overlap and inclusion ratios | Discovering implicit relationships not enforced by constraints |
| **Functional dependencies** | Column A's value determines column B's value | Normalization analysis, deduplication logic, SCD type decisions |
| **Inclusion/exclusion ratios** | % of values in column A appearing in column B | Referential integrity assessment; detecting orphaned or dangling records |
| **Overlap analysis** | Exact/cross-value matching between columns | Identifying redundant or duplicate columns across tables |

**Foreign key discovery workflow:** The profiling system scans all column pairs across tables, computing an inclusion ratio: of the distinct values in column A, what fraction appear in column B? A ratio of 1.0 (or very close) makes A a strong foreign-key candidate for B. Business rules and domain knowledge validate or reject these candidates — e.g., a `customer_id` column in an orders table may have 100% inclusion with `customer_id` in the customers table (valid FK), while a `status_code` column in two unrelated tables may also show high overlap by coincidence.

**Output:** A **relationship map** — a visual or structured representation of:
- Candidate primary keys with uniqueness scores
- Foreign key candidates with inclusion ratios
- Functional dependency chains
- Referential integrity gaps (orphaned records, missing parent references)
- Data lineage showing how columns relate across source systems

### 2.3 Metadata Profiling

**What it is:** Examines the database or schema structure itself, independent of actual data values. It inventories what exists at the schema, table, and column level — a prerequisite for any data management initiative.

**Key metrics collected:**

| Metric | Description | Why It Matters |
|--------|-------------|----------------|
| **Table row counts** | Number of records per table | Understanding dataset volume and growth |
| **Column counts** | Number of columns per table | Schema complexity assessment |
| **Data types** | Declared SQL types (VARCHAR, INT, DATE, DECIMAL) | Type compatibility analysis for integration |
| **Nullable flags** | Whether columns allow NULLs | Identifying mandatory vs. optional fields |
| **Primary keys** | Defined PK columns | Understanding table identity |
| **Foreign keys** | Defined FK constraints | Documented vs. actual relationships |
| **Indexes** | Existing indexes (clustered, non-clustered, unique) | Performance and query optimization context |
| **Default values** | Default expressions on columns | Understanding fallback values |
| **Constraints** | CHECK, UNIQUE, NOT NULL constraints | Enforced vs. unenforced business rules |

**Output:** A **schema inventory** (data dictionary / metadata catalog) containing:
- Complete list of tables, columns, and their declared metadata
- Constraint definitions (PK, FK, unique, check, default)
- Index inventory
- Data type summary with cross-system mapping recommendations
- Physical storage details (row counts, table sizes, partition information)
- Data lineage documentation

---

## 3. Data Profiling Dimensions (What to Measure)

While the three profiling types define *what* you examine, these quality dimensions define *what aspects of quality* you measure. A thorough profiling exercise evaluates data across multiple dimensions.

| Dimension | Key Questions | Typical Metrics |
|-----------|---------------|-----------------|
| **Completeness** | Is data missing? How much? Which columns are affected? | Null count, null %, fill rate (non-null / total rows), column completeness score |
| **Uniqueness** | Are there duplicates? How many per column? | Duplicate count, duplicate %, uniqueness ratio (distinct / total), exact vs. fuzzy duplicate breakdown |
| **Consistency** | Are values in expected format? Do codes match reference values? | Format adherence %, pattern violation count, cross-column consistency score, standardization level |
| **Validity** | Do values conform to business rules and domain constraints? | Domain violation count, rule pass rate, out-of-range values, invalid code detection |
| **Accuracy** | Is data correct when compared to a trusted reference? | Error rate (requires reference data), misclassification count, deviation from known-true values |
| **Timeliness** | Is data current enough for its intended purpose? | Age distribution (days since last update), staleness indicators, latency from source system, data currency score |
| **Cardinality** | How many distinct values exist? Is the column a key or an attribute? | Distinct count, cardinality ratio (distinct / total), value frequency distribution, entropy |

**Practical note:** Not every dimension applies to every column. A phone number column is profiled for completeness, consistency (format adherence to `+1 XXX-XXX-XXXX`), and validity (valid area codes), but not for uniqueness since many rows may share a phone number. A customer ID column is profiled for uniqueness and completeness, with cardinality near 1.0 expected. Always tailor dimensions to column semantics and business context.

---

## 4. Data Profiling Process

A structured, repeatable process ensures profiling efforts are consistent, thorough, and actionable. The following seven-step process is widely adopted in the industry.

### Step 1: Scope

Define the boundaries of the profiling exercise:
- Identify data sources, databases, schemas, tables, and columns to profile
- Prioritize by business criticality — core entities first (customer, product, transaction, employee)
- Determine profiling depth: full scan vs. sampling, per-column detail vs. summary view
- Document inclusion/exclusion criteria for tables and columns

### Step 2: Plan

Design the profiling approach before executing:
- Define profiling rules and thresholds (e.g., "warn when null rate > 5%", "flag uniqueness < 0.99 for key columns")
- Select sampling strategy for large tables (random, stratified, cluster, systematic)
- Choose profiling dimensions per data type (numeric vs. string vs. date)
- Identify expected patterns and domains (email, phone, date formats, currency codes)
- Gather reference datasets needed for accuracy and validity checks
- Document business rules for validity assessment

### Step 3: Extract

Acquire the data to be profiled:
- For small tables (< 1M rows): extract 100% of data
- For large tables: apply the chosen sampling strategy
- Extract metadata from system catalogs using INFORMATION_SCHEMA or equivalent
- For distributed systems: extract from each partition, shard, or node
- Run extraction during off-peak hours or on read replicas to avoid production impact
- Validate extraction: verify row counts and a few key column stats as a sanity check

### Step 4: Profile

Execute the profiling algorithms:
- Run column statistics: nulls, distinct values, min/max/mean, frequency distributions, quantiles
- Run pattern and domain discovery on string columns using regex
- Run key analysis: primary key candidates, foreign key candidates, functional dependencies
- Run cross-table relationship discovery across all relevant tables
- Compute quality dimension scores per column and per table
- Generate profiling report with visualizations (histograms, bar charts, heat maps)

### Step 5: Analyze

Interpret and contextualize the profiling results:
- Review outliers, anomalies, and unexpected patterns
- Identify data quality issues with business impact assessment
- Prioritize findings as critical, major, or minor
- Cross-reference with business knowledge — some "issues" may be legitimate (e.g., 90% nulls in an optional field is fine)
- Classify findings: schema issues, data quality issues, relationship issues, metadata gaps
- Document findings with examples and relevant context for downstream consumers

### Step 6: Act

Take action on findings:
- Remediate critical data quality issues (cleanse, standardize, enrich)
- Update data models and schemas based on profiling insights
- Work with source system owners to fix root causes in production systems
- Implement automated data quality rules and validation in pipelines
- Document transformation rules and mapping specifications for migration or integration projects
- For issues that cannot be fixed immediately, document workarounds and risk acceptance

### Step 7: Monitor

Establish ongoing visibility:
- Set up automated data quality checks for critical dimensions using Great Expectations, Deequ, or equivalent
- Create dashboards to track quality metrics over time (Apache Superset, Metabase, or your observability platform)
- Define alert thresholds for quality degradation
- Schedule periodic re-profiling (monthly, quarterly) to detect drift in data characteristics
- Feed monitoring results back into the data governance program for continuous improvement

---

## 5. Data Profiling Tools

### 5.1 Open Source Tools

#### Great Expectations

- **What it is:** Python-based, the most popular modern data quality and profiling framework. Uses a "declarative assertions" model called Expectations — statements about what data should look like: `expect_column_values_to_not_be_null("email")`, `expect_column_mean_to_be_between("age", 18, 65)`.
- **Profiling features:**
  - Automatic generation of Expectations from data (a profiler step infers column statistics and creates baseline Expectations)
  - Column statistics: min, max, mean, null count, distinct values, quantiles, distribution charts
  - Auto-generated Data Docs — rich HTML documentation with profiling results that can be versioned and shared
  - Support for expectations across single columns, column pairs, and multi-table (cross-table) scenarios
- **Integrations:** dbt, Airflow, Prefect, Spark, SQL databases (PostgreSQL, Snowflake, BigQuery, Redshift, Databricks)
- **CI/CD:** Run Expectations in CI/CD pipelines as data quality gates — fail the build if critical expectations are not met
- **Best for:** Teams wanting code-first, CI/CD-integrated data quality with rich, shareable documentation
- **Website:** greatexpectations.io

#### Apache Griffin

- **What it is:** Open-source data quality platform originally developed by eBay and LinkedIn. Spark-based backend for distributed profiling and quality measurement.
- **Features:** Accuracy measurement (source-to-target comparison), completeness analysis, timeliness measures, data profiling visualization UI, metric storage and trending
- **Best for:** Large-scale Spark/Hadoop environments needing batch data quality measurement at enterprise scale
- **Considerations:** Less actively maintained than Great Expectations; steeper setup curve

#### OpenRefine

- **What it is:** Desktop GUI tool (formerly Google Refine) for data cleaning and exploratory profiling.
- **Key features:**
  - Column statistics and distribution via faceting (text facet, numeric facet, timeline facet, scatterplot facet)
  - Clustering for fuzzy duplicate detection using key collision and nearest neighbor methods
  - GREL (General Refine Expression Language) for data transformations and cleaning operations
  - Undo/redo history makes data preparation reproducible as a JSON script
- **Not for:** Automated pipeline integration — this is a GUI tool, not a library
- **Best for:** Ad-hoc data exploration, data preparation for migration projects, one-time data cleanup exercises
- **Website:** openrefine.org

#### Pandas Profiling / YData Profiling

- **What it is:** Python library that automatically generates comprehensive profiling reports from pandas DataFrames. Now maintained as `ydata-profiling`.
- **One-liner usage:** `ProfileReport(df).to_file("report.html")` generates a complete HTML report
- **Report sections:** Overview (dataset summary), Alerts (warnings about data quality), Variables (per-column statistics, histograms, quantile statistics), Correlations (Pearson, Spearman, Kendall, Cramér's V), Missing values (matrix, bar chart, heatmap, dendrogram), Sample data (first/last rows)
- **Large dataset support:** Minimal mode disables expensive computations; sampling for rapid exploration
- **Best for:** Quick data exploration in Jupyter notebooks, ad-hoc analysis of CSV files or small datasets, initial data assessment before deeper profiling
- **Installation:** `pip install ydata-profiling`

#### Deequ (by AWS)

- **What it is:** A library for "unit testing" data on Apache Spark. Treats data quality verification like software testing with a pass/fail model.
- **Key concepts:**
  - Constraint verification: `verify(data, constraint)` returns ConstraintStatus (Success / Failure / Warning)
  - AnalysisRunner: computes column statistics in a Spark-native, distributed manner
  - ConstraintSuggestionRunner: recommends constraints from data automatically
- **Profiling example (see Section 8 for full code):** AnalysisRunner computes Size(), Completeness(), Uniqueness(), Distinctness(), Mean(), StdDev() on the Spark DataFrame
- **Best for:** Teams already using Spark who want declarative quality testing integrated into their Spark ETL pipelines
- **GitHub:** github.com/awslabs/deequ

### 5.2 Commercial Tools

#### Informatica Data Quality / Data Profiling

- **Scope:** Enterprise-grade, comprehensive profiling as part of the Informatica Intelligent Data Management Cloud (IDMC).
- **Capabilities:**
  - Column profiling: statistics, pattern discovery, and domain inference (identifies column semantics — email, SSN, phone, etc.)
  - Relationship discovery: foreign key analysis, functional dependency detection, cross-table profiling
  - Data domain discovery with automated identification of data domains (Person Name, Location, Identifier, etc.)
  - Pattern finding: regex extraction and pattern frequency reporting
  - Data quality scorecards and executive dashboards with trend tracking
- **Pricing:** Enterprise tier, typically $100K+/year
- **Best for:** Large enterprises with existing Informatica investments needing end-to-end data governance and profiling at scale

#### IBM InfoSphere Information Analyzer

- **Scope:** Enterprise data profiling and quality analysis within the IBM Information Server platform.
- **Capabilities:** Column analysis (statistics, patterns, domains), primary key analysis with uniqueness determination, functional dependency discovery, cross-table referential integrity analysis, data quality exception management with workflow-driven remediation
- **Pricing:** Enterprise (expensive)
- **Best for:** Organizations with existing IBM ecosystem investments (InfoSphere, DB2, DataStage)

#### Talend Data Quality

- **Scope:** Open-source core with commercial extensions for profiling and quality.
- **Capabilities:** Column statistics and frequency analysis, pattern and domain discovery (email, phone, SSN identification), indicator framework for tracking quality metrics over time, integration with Talend data integration pipelines
- **Pricing:** Open source (free community edition) with enterprise subscriptions for advanced features
- **Best for:** Talend integration pipeline users wanting tight coupling between profiling and data processing

#### Ataccama / Profisee

- **Scope:** Master Data Management (MDM)-centric profiling tools.
- **Capabilities:** Data quality measurement and monitoring, profiling designed for golden record creation, matching and survivorship analysis, data stewardship workflows with issue assignment and resolution tracking
- **Pricing:** Mid-market to enterprise
- **Best for:** MDM projects needing integrated profiling as part of master data management programs

#### SAS DataFlux / Data Management

- **Scope:** Statistical profiling from the SAS ecosystem.
- **Capabilities:** Advanced statistical profiling (natural fit for SAS users), data quality rules engine, address validation and standardization, data monitoring and alerting with configurable thresholds
- **Pricing:** Enterprise
- **Best for:** Organizations with existing SAS infrastructure seeking to add data quality capabilities

---

## 6. Sampling Strategies for Large Datasets

When datasets exceed millions of rows, profiling 100% of data becomes computationally expensive and time-consuming. Sampling provides a representative subset for efficient analysis.

| Strategy | How It Works | Best For | Considerations |
|----------|-------------|----------|----------------|
| **Random Sampling** | Select N% of rows with uniform random probability | General profiling with unbiased estimates | Simple and effective for most cases. Rare values may be missed at low sampling rates. |
| **Stratified Sampling** | Divide data into subgroups (strata), then sample within each group | Data with natural partitions (region, date, category, customer segment) | Ensures representation of small subgroups. Requires a meaningful stratification column. |
| **Cluster Sampling** | Randomly select entire blocks or partitions of rows | Large clustered data (HDFS partitions, database shards, daily files) | Highly efficient when data is already partitioned. Can introduce bias if clusters are internally heterogeneous. |
| **Systematic Sampling** | Select every Nth row (e.g., every 1000th row) | Sequential data (log files, time-series, sensor readings) | Very simple to implement. Risk of periodicity bias if the data has a pattern matching the sampling interval. |
| **Full Scan** | Profile 100% of rows | Small tables (< 1M rows); critical quality/regulatory checks; compliance scenarios | Most accurate but expensive. Essential when precise counts of rare issues matter for regulatory reporting. |

**Practical recommendations:**

- For initial discovery profiling on tables > 10M rows, a 10% random sample is often sufficient
- Increase sampling rate (25-50%) when profiling specifically targets rare issues or edge cases
- For compliance profiling (PII detection, regulatory reporting), prefer full scans or very high rates
- Always validate sample representativeness — compare a few quick metrics (row count, null rate on a known column) between sample and full data
- Combine approaches: stratified sampling for core dimensions + full scan for critical quality checks

---

## 7. Profiling for Specific Use Cases

Different data projects have different profiling priorities. Here is how profiling should be tailored to four common scenarios.

### Data Warehouse Design

**Goal:** Profile source systems before designing the dimensional model.

**What to profile:**
- **Grain determination:** What does one row represent? Are there multiple grains in the same table?
- **Natural key candidates:** Which columns (or combinations) are truly unique business identifiers?
- **Relationship discovery:** How do tables relate? What are parent-child hierarchies? What is the join path?
- **Data type alignment:** Are source data types compatible with warehouse targets? Will there be truncation?
- **SCD analysis:** Do dimensional attributes change over time? How frequently? What is the change pattern?

**Key output:** Candidate natural keys with uniqueness validation, relationship map across source tables, data type mapping and transformation recommendations, SCD type recommendations, grain documentation.
### Data Migration

**Goal:** Understand source data quality, mapping needs, and transformation rules before moving to a new system.

**What to profile:**
- **Completeness:** Which columns have significant missing data? Are NULLs meaningful vs. accidental?
- **Domain values:** What distinct values exist in code/reference columns? Do they map 1:1 to target domains?
- **Format inconsistencies:** Are dates in a single format? Are phone numbers consistently formatted?
- **Length constraints:** Do source data lengths exceed target column definitions? Will data be truncated?
- **Duplicate analysis:** Are there suspected duplicate records? What is the deduplication strategy?

**Key output:** Completeness report with null-pattern analysis, domain value mapping table (source values → target values), format inconsistency catalog, data truncation warnings, duplicate detection and resolution strategy.
### Data Quality Program

**Goal:** Profile to establish a baseline quality assessment, prioritize improvements, and set measurable targets.

**What to profile:**
- **Quality baseline:** Scores across all dimensions (completeness, uniqueness, consistency, validity, accuracy, timeliness)
- **Issue catalog:** Documented issues with examples, frequencies, and root causes
- **Business impact assessment:** Which quality issues have the greatest impact on downstream consumers?

**Key output:** Quality scorecard (per-dimension, per-table, per-column), prioritized issue catalog with severity ratings, root cause analysis (source issues, integration bugs, user entry problems), trend data (if re-profiling), remediation roadmap.
### Data Lake Ingestion

**Goal:** Profile data before landing it in the data lake — understand schema, handle drift, manage semi-structured data.

**What to profile:**
- **Schema-on-read analysis:** What schema would best represent the data? How does schema evolve over time?
- **Data type inference:** Should a column be stored as string, numeric, or timestamp? What precision is needed?
- **Nested/complex types:** For JSON, Avro, or Parquet data, what is the nesting structure? How deep does it go?
- **Schema drift tracking:** Do new columns appear over time? Do existing columns change type? Are fields added/removed?
- **Partitioning recommendations:** Which columns are good candidates for Hive-style partition pruning?

**Key output:** Schema recommendations and data type mapping, schema drift report (new fields, type changes, field removal), anomaly detection (unexpected null spikes, value range shifts), partition column recommendations, compression and file format guidance.

---

## 8. Integration with Data Pipelines

Modern data pipelines integrate profiling as an automated step — not an afterthought. Here are two common integration patterns.

### dbt + Great Expectations

dbt natively supports data tests, and Great Expectations expectations can be used via the `dbt-expectations` package. This gives you the dbt workflow (version-controlled, documented, CI/CD-friendly) with Great Expectations' assertion vocabulary.

```yaml
# schema.yml
version: 2

models:
  - name: customers
    description: "Customer master data"
    columns:
      - name: customer_id
        description: "Primary key"
        tests:
          - unique
          - not_null
      - name: email
        tests:
          - unique
          - not_null
          - accepted_values:
              values: ['*.com', '*.net', '*.org']
      - name: signup_date
        tests:
          - not_null
      - name: status
        tests:
          - accepted_values:
              values: ['active', 'inactive', 'churned']
```

Run profiling tests:
```bash
# Test a specific model
dbt test --select model:customers

# Test an entire source
dbt test --select source:customers+
```

### Spark + Deequ

For Spark-native profiling, Deequ integrates directly into ETL jobs for both profiling and verification in a single pipeline.

```python
from deequ import AnalysisRunner, VerificationSuite, Check
from deequ.checks import CheckLevel
from deequ.constraints import ConstraintStatus

# Profiling phase — compute column statistics
analysis_result = AnalysisRunner(spark) \
    .onData(df) \
    .addAnalyzer(Size()) \
    .addAnalyzer(Completeness("email")) \
    .addAnalyzer(Uniqueness("customer_id")) \
    .addAnalyzer(ApproxCountDistinct("segment")) \
    .addAnalyzer(Mean("transaction_amount")) \
    .addAnalyzer(StandardDeviation("transaction_amount")) \
    .addAnalyzer(MaxLength("address_line1")) \
    .run()

# Quality verification phase — assert constraints
verification_result = VerificationSuite(spark) \
    .onData(df) \
    .addCheck(
        Check(CheckLevel.WARNING, "Core quality checks")
        .isComplete("customer_id")
        .isUnique("customer_id")
        .isComplete("email")
        .hasMinLength("zip_code", 5)
        .isContainedIn("status", ["active", "inactive"])
    ) \
    .run()

# React to failures
if verification_result.status == ConstraintStatus.Failure:
    # Log, alert, or fail the pipeline
    print("Quality checks failed — investigating...")
```

### CI/CD Integration

```yaml
# .github/workflows/data-quality.yml — CI pipeline step
data-quality:
  stage: test
  script:
    - spark-submit run_profile_checks.py
    - great_expectations checkpoint run production_checks
  artifacts:
    reports:
      data_quality: reports/quality_report.html
```

---

## 9. Comparison Table

This table provides a quick-reference comparison across all tools discussed in Section 5.

| Tool | Open Source | Profiling Scope | Automation | Key Integration | Ease of Use | Best For |
|------|-------------|-----------------|------------|-----------------|-------------|----------|
| **Great Expectations** | ✅ Full | Column + Table + Cross-table | ✅ CI/CD | dbt, Airflow, Spark, SQL DBs | Medium | Code-first data quality with rich docs |
| **Pandas / YData Profiling** | ✅ Full | Column (single dataset) | ❌ One-off | Python, Jupyter | Very Easy | Ad-hoc notebooks exploration |
| **Deequ** | ✅ Full | Column + Table | ✅ CI/CD | Spark only | Medium | Spark-native quality testing |
| **OpenRefine** | ✅ Full | Column (manual/exploratory) | ❌ GUI only | Desktop standalone | Easy | Data cleaning and prep |
| **Apache Griffin** | ✅ Full | Column + Table | ✅ Batch | Spark, Hadoop | Complex | Large-scale Spark environments |
| **Informatica** | ❌ Commercial | Enterprise-wide | ✅ Automated | Broad integration | Medium | Enterprise data governance |
| **IBM InfoSphere** | ❌ Commercial | Enterprise-wide | ✅ Automated | IBM ecosystem | Medium | IBM shops with existing infra |
| **Talend** | ⚠️ Core free | Column + Table | ✅ Pipeline | Talend ecosystem | Medium | Talend pipeline users |
| **SAS DataFlux** | ❌ Commercial | Enterprise-wide | ✅ Automated | SAS infrastructure | Medium | Existing SAS customers |
| **Ataccama** | ❌ Commercial | Enterprise-wide | ✅ Automated | Broad integration | Medium | MDM projects |

---

## 10. Best Practices

- **Profile early, profile often.** Don't wait for data quality issues to surface in production. Profile during ingestion, before transformation, and after transformation. Early profiling catches problems when they are cheapest to fix.

- **Use profiling to inform data model design.** Design dimensional models (star schema, data vault, lakehouse) only after profiling reveals true granularity, key structures, and data relationships. A model designed without profiling is based on assumptions, not evidence.

- **Establish thresholds.** Define acceptable ranges for every data quality dimension per column. Examples: null rate < 5%, uniqueness > 0.99 for key columns, format adherence > 95%, domain pass rate > 98%. Automatically flag or block data that breaches thresholds.

- **Automate profiling in CI/CD.** Data quality checks should gate deployments the same way unit tests gate code changes. Great Expectations checkpoints, dbt tests, and Deequ verification suites can run as CI pipeline stages, failing the build when critical expectations are violated.

- **Combine automated profiling with manual review.** Automated tools excel at computing statistics and flagging anomalies, but they cannot interpret business context. A column with 90% nulls may be perfectly acceptable if data collection is optional. Conversely, a column with 100% completeness may still contain garbage-in, garbage-out values. Always review profiling results with domain experts.

- **Document findings and share with data producers.** Data profiling often reveals issues in source systems — missing values, inconsistent formats, violated business rules. Share findings with application teams and data entry departments so they can fix root causes rather than having downstream pipelines compensate downstream.

- **Monitor trends over time.** A single profiling exercise is a snapshot. Track quality metrics across multiple profiling cycles. Is data quality improving or degrading? Are new columns or sources introducing issues? Are remediation efforts working? Build dashboards to answer these questions.

- **Start small, iterate.** Profile highest-priority tables first (core business entities: customer, product, transaction). Add more tables as the practice matures and delivers value. Roma was not built in a day, and neither is a data profiling program.

- **Re-profile after major changes.** Schema changes, new data sources, system migrations, and application upgrades all warrant fresh profiling.

- **Profile source AND target.** In migration or integration scenarios, profile both the source and target systems. Understanding target system capabilities, constraints, and data type limitations is as important as understanding the source data.

- **Consider data privacy.** When profiling datasets, flag PII columns (email, SSN, phone, credit card) for special handling. Mask or redact sensitive data in profiling reports that are shared across teams. GDPR, CCPA, and other privacy regulations apply to profiling activities.

- **Build a data quality culture.** Profiling is most effective when data producers, data engineers, data analysts, and data stewards all participate in reviewing and acting on findings. Profiling is a team sport — the output is only valuable if it drives action.

---

## Data Sources & Further Reading

- **Great Expectations Documentation** — greatexpectations.io
- **YData Profiling Documentation** — docs.profiling.ydata.ai
- **Deequ GitHub Repository** — github.com/awslabs/deequ
- **OpenRefine Documentation** — openrefine.org/docs
- **Informatica Data Quality Documentation** — docs.informatica.com
- **Talend Data Quality Documentation** — help.talend.com
- **IBM InfoSphere Information Analyzer** — ibm.com/docs/en/ibmiis
- **DAMA-DMBOK** (Data Management Body of Knowledge) — Data Quality chapter. DAMA International.
- **"Data Quality: Concepts, Methodologies and Techniques"** by Carlo Batini, Monica Scannapieco — Springer, 2006/2016.
- **"The Data Warehouse Toolkit"** by Ralph Kimball — Dimensional modeling and data profiling context.
- **"Fundamentals of Data Engineering"** by Joe Reis and Matt Housley — O'Reilly, 2022. Chapter on data quality.
- **Industry Blogs:** Monte Carlo Data, Sifflet, Bigeye, dbt Developer Blog — modern data quality and profiling practices.

---

*This guide was compiled from open-source tool documentation, industry best practices, and the Data Management Body of Knowledge (DAMA-DMBOK). It is intended as a standalone reference for data engineers, data architects, data stewards, and anyone involved in data quality and data management programs.*
