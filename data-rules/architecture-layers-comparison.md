# Architecture Layers Comparison: Medallion vs Traditional 5-Layer

## Overview

This document compares two architectural approaches for dbt data modeling:

1. **Medallion Architecture** (from `dbt-datamodelling-snowflake.mdc`) - 3-layer approach
2. **Traditional 5-Layer Architecture** (from `coding_standards/rules/dbt/dbt.mdc`) - 5-layer approach

**Important:** The choice between architectures is **dependent on client configuration needs**. Do not arbitrarily choose one architecture over the other. **Talk to the client** about how they want their data organized, understand their requirements, and then select the appropriate architecture. Once a choice is made, use that architecture's layer structure and purpose-built functions consistently throughout the project. The selected architecture's patterns should be integrated into the `dbt-datamodelling-snowflake.mdc` rules file to ensure consistency.

---

## Architecture Comparison

### Medallion Architecture (3 Layers)

| Layer | Purpose | Database | Materialization | Naming Pattern |
|-------|---------|----------|-----------------|----------------|
| **Raw** | Load data as-is from source systems with minimal transformation | `raw` | Tables (ELT) or Views (file ingestion) | `raw_<party>__<source>__<table>` |
| **Integration** | Data cleansing, conforming, deduplication, business rules | `int` | Views (default) or Tables (performance) | `int_<party>__<source>__<entity>` |
| **Analytics** | Dimensional models (Kimball star schema) | `anl` | Tables (always) | `anl_<subject>__<fact\|dim>_<name>` |

**Key Characteristics:**

- Singular nouns for Raw/Integration layers
- Plural nouns for Analytics layer
- Business-friendly naming in Analytics layer
- Focus on Medallion data quality progression (bronze → silver → gold)

### Traditional 5-Layer Architecture

| Layer | Purpose | Database | Materialization | Naming Pattern |
|-------|---------|----------|-----------------|----------------|
| **DL** | Raw data from cloud storage | `DL` | View/Table | N/A (sources only) |
| **STG** | Fivetran-landed data, sources defined | `STG` | Source only | N/A (sources) |
| **ODS** | Operational Data Store — normalized, light transforms | `ODS` | Table | `ods__<source>__<object>` |
| **DW** | Data Warehouse — dimensional models (Kimball) | `DW` | Table | `dw__<subject>__dim_<object>` or `dw__<subject>__fact_<object>` |
| **BI** | Team/project-specific views and aggregations | `BI` | View/Table | `bi__<team>__<object>` |

**Key Characteristics:**

- Singular nouns for ODS layer
- Plural nouns for DW layer
- Separate staging layer for Fivetran/ELT tools
- Separate BI layer for team-specific views

**Synopsis:** The choice between Medallion and Traditional architectures is **dependent on client configuration needs**. Do not arbitrarily choose one architecture over the other. **Talk to the client** about how they want their data organized, understand their requirements, and then select the appropriate architecture. Once a choice is made, use that architecture's layer structure and purpose-built functions consistently throughout the project. The selected architecture's patterns should be integrated into the `dbt-datamodelling-snowflake.mdc` rules file to ensure consistency.

---

## Layer Mapping & Equivalency

### Conceptual Mapping

| Medallion | Traditional 5-Layer | Purpose Alignment |
|-----------|---------------------|-------------------|
| **Raw** | **DL** + **STG** | Both represent source system data with minimal transformation |
| **Integration** | **ODS** | Both perform normalization, cleansing, and business rule application |
| **Analytics** | **DW** + **BI** | Both represent dimensional models and business-facing analytics |

### Detailed Comparison

#### Raw Layer (Medallion) vs DL/STG (Traditional)

**Medallion Raw:**

- Single layer for all source data
- Handles both cloud storage ingestion and ELT tool outputs
- Materialization: Tables (ELT) or Views (file ingestion)
- Metadata columns: `__load_id`, `__load_dts`, `__source_filename`
- Models in `models/raw/` folder

**Traditional DL/STG:**

- **DL:** Separate layer for cloud storage data (Views/Tables)
- **STG:** Separate layer specifically for Fivetran-landed data (Sources only)
- Sources defined in `sources/` folder
- More granular separation of ingestion methods

**Key Difference:** Medallion combines both ingestion patterns into one layer, while Traditional separates them.

#### Integration Layer (Medallion) vs ODS (Traditional)

**Medallion Integration:**

- Data cleansing, conforming, deduplication
- Normalized entities ready for dimensional modeling
- Soft deletes: `is_deleted` boolean flag
- Materialization: Views (default) or Tables (performance)
- Models in `models/int/` folder
- Singular nouns: `int_sales__customer`

**Traditional ODS:**

- Operational Data Store — normalized, light transforms
- Materialization: Tables (always)
- Models in `models/ods/` folder
- Singular nouns: `ods__netsuite__accounts`
- Similar purpose but always materialized as tables

**Key Difference:** Medallion defaults to Views (cost optimization), Traditional always uses Tables (performance).

#### Analytics Layer (Medallion) vs DW/BI (Traditional)

**Medallion Analytics:**

- Single layer for all dimensional models
- Dimensional models (Kimball star schema)
- Facts: `anl_sales__fact_orders` (plural, verb-based)
- Dimensions: `anl_sales__dim_customers` (plural, noun-based)
- Materialization: Tables (always)
- Business-friendly naming required
- Models in `models/anl/` folder

**Traditional DW/BI:**

- **DW:** Dimensional models (Kimball)
  - Facts: `dw__finance__fact_transaction_details` (plural)
  - Dimensions: `dw__finance__dim_accounts` (plural)
  - Materialization: Tables (always)
  - Models in `models/dw/` folder
- **BI:** Team/project-specific views and aggregations
  - Materialization: Views or Tables
  - Models in `models/bi/` folder
  - Naming: `bi__<team>__<object>`

**Key Difference:** Medallion combines dimensional models and team-specific views into one Analytics layer, while Traditional separates DW (enterprise dimensional models) from BI (team-specific views).

---

## Naming Convention Comparison

### Model Naming Patterns

| Aspect | Medallion | Traditional |
|--------|-----------|------------|
| **Delimiter** | Double underscore `__` | Double underscore `__` |
| **Raw/Staging** | `raw_<party>__<source>__<table>` | `ods__<source>__<object>` (no raw prefix) |
| **Integration/ODS** | `int_<party>__<source>__<entity>` (singular) | `ods__<source>__<object>` (singular) |
| **Analytics/DW** | `anl_<subject>__<fact\|dim>_<name>` | `dw__<subject>__dim_<object>` or `dw__<subject>__fact_<object>` |
| **BI Layer** | Included in `anl_` | `bi__<team>__<object>` (separate layer) |

**Important:** Both architectures require **full words** `fact_` and `dim_` - abbreviations like `f_`, `fi_`, `d_` are **NOT allowed**. This aligns with the "Avoid Abbreviations" principle in both standards.

### Table Naming by Layer

**Medallion:**

- Raw: Singular (`customer`, `sales_order`)
- Integration: Singular (`customer`, `product`)
- Analytics: Plural (`fact_orders`, `dim_customers`)
  - **Facts:** Always use full word `fact_` prefix (e.g., `fact_orders`, `fact_sales`)
  - **Dimensions:** Always use full word `dim_` prefix (e.g., `dim_customers`, `dim_products`)
  - **Lookup Tables:** Are dimensions and use `dim_` prefix (e.g., `dim_status_codes`, `dim_regions`, `dim_product_categories`)
  - **No abbreviations:** `f_`, `fi_`, `d_` are NOT allowed

**Traditional:**

- ODS: Singular (`account`, `transaction`)
- DW: Plural (`dim_accounts`, `fact_transactions`)
  - **Facts:** Always use full word `fact_` prefix (e.g., `fact_transaction_details`)
  - **Dimensions:** Always use full word `dim_` prefix (e.g., `dim_accounts`, `dim_customers`)
  - **Lookup Tables:** Are dimensions and use `dim_` prefix (e.g., `dim_status_codes`, `dim_regions`, `dim_product_categories`)
  - **No abbreviations:** `f_`, `fi_`, `d_` are NOT allowed
- BI: Plural (team-specific)

---

## Materialization Strategy Comparison

| Layer | Medallion | Traditional |
|-------|-----------|-------------|
| **Raw/DL** | Tables (ELT) or Views (file ingestion) | Views/Tables |
| **Staging** | N/A (included in Raw) | Source only (no materialization) |
| **Integration/ODS** | Views (default) or Tables (performance) | Tables (always) |
| **Analytics/DW** | Tables (always) | Tables (always) |
| **BI** | Included in Analytics | Views or Tables |

**Key Insight:** Medallion optimizes for cost by defaulting Integration layer to Views, while Traditional prioritizes performance with Tables.

---

## Constraints & Database Enforcement

### Primary Keys, Foreign Keys, and Constraint Types

Both architectures require explicit declaration of constraints (PRIMARY KEY, FOREIGN KEY, UNIQUE) for data integrity and query optimization. However, **database platforms differ significantly in how they enforce these constraints**.

### Database Constraint Enforcement Models

#### Databases That Enforce Constraints (Traditional RDBMS)

**Examples:** PostgreSQL, MySQL, SQL Server, Oracle

- **Primary Keys:** Enforced - prevents duplicate and NULL values
- **Foreign Keys:** Enforced - prevents orphaned records, cascades deletes/updates
- **Unique Constraints:** Enforced - prevents duplicate values
- **NOT NULL:** Enforced - prevents NULL values

**Implications:**

- Data integrity is guaranteed at the database level
- Invalid data insertion will fail with errors
- Referential integrity is automatically maintained
- dbt tests serve as **validation** but constraints provide **enforcement**

#### Databases That Don't Enforce Constraints (Modern Cloud Platforms)

**Examples:** Snowflake, BigQuery, Databricks (Delta Lake)

- **Primary Keys:** Informational only - declared but not enforced (except NOT NULL)
- **Foreign Keys:** Informational only - declared but not enforced
- **Unique Constraints:** Informational only - declared but not enforced
- **NOT NULL:** Enforced - this is the only constraint type that is enforced

**Implications:**

- Constraints are **metadata** for query optimization (e.g., Snowflake's Query Optimizer uses them for join elimination)
- Data integrity must be enforced through **dbt tests**
- Invalid data can be inserted without database-level errors
- dbt tests serve as **both validation AND enforcement**

### Why Declare Constraints Even If Not Enforced?

1. **Documentation:** Constraints document the logical data model and relationships between tables

2. **Tooling:** Data catalog tools, BI tools, and lineage tools read constraint metadata to understand relationships

3. **Future-Proofing:** If migrating to a database that enforces constraints, they're already declared

**Note:** Use dbt tests for referential integrity validation.

### dbt Testing Strategy for Constraint Enforcement

Since many modern cloud databases don't enforce constraints, **dbt tests become critical for data integrity**:

#### Required Tests for Every Model

**Primary Keys:**

```yaml
- name: account_key
  tests:
    - unique
    - not_null
```

**Foreign Keys:**

```yaml
- name: customer_key
  tests:
    - relationships:
        to: ref('dim_customers')
        field: customer_key
```

**Composite Unique Keys:**

```yaml
tests:
  - dbt_utils.unique_combination_of_columns:
      combination_of_columns:
        - order_id
        - line_item_id
```

**Unique/Alternate Keys:**

```yaml
- name: email_address
  tests:
    - unique
```

#### Testing Best Practices

1. **Test at Every Layer:** Apply tests at Raw, Integration/ODS, and Analytics/DW layers
2. **Test Early:** Catch data quality issues as early as possible in the pipeline
3. **Test Relationships:** Ensure referential integrity between facts and dimensions
4. **Test Composite Keys:** Validate business rules for multi-column uniqueness
5. **Automate Testing:** Run tests as part of CI/CD pipeline, not just manually

### Constraint Declaration Syntax by Database

#### Snowflake

```sql
-- Declare constraints for documentation (not enforced)
CONSTRAINT pk_customer PRIMARY KEY (customer_key)
CONSTRAINT fk_order_customer FOREIGN KEY (customer_key) REFERENCES dim_customers(customer_key)
CONSTRAINT ak_customer_email UNIQUE (email_address)
```

**Note:** Snowflake does not enforce these constraints. Use dbt tests to validate referential integrity (e.g., test that foreign keys in fact tables exist in dimension tables).

#### PostgreSQL / Traditional RDBMS

```sql
-- Constraints are enforced automatically
CONSTRAINT pk_customer PRIMARY KEY (customer_key)
CONSTRAINT fk_order_customer FOREIGN KEY (customer_key) REFERENCES dim_customers(customer_key)
CONSTRAINT ak_customer_email UNIQUE (email_address)
```

#### BigQuery

```sql
-- Constraints are informational, similar to Snowflake
CONSTRAINT pk_customer PRIMARY KEY (customer_key) NOT ENFORCED
CONSTRAINT fk_order_customer FOREIGN KEY (customer_key) REFERENCES dim_customers(customer_key) NOT ENFORCED
CONSTRAINT ak_customer_email UNIQUE (email_address) NOT ENFORCED
```

### Summary: Constraint Strategy

| Database Type | Constraint Enforcement | dbt Tests Role | Constraint Declaration |
|---------------|----------------------|----------------|----------------------|
| **Traditional RDBMS** (PostgreSQL, MySQL, SQL Server) | Full enforcement | Validation only | Required for integrity |
| **Cloud Platforms** (Snowflake, BigQuery, Databricks) | Informational only | **Enforcement** | Required for optimization + documentation |

**Key Takeaway:** For cloud platforms that don't enforce constraints, **dbt tests are not optional - they are the primary mechanism for ensuring data integrity**. Always declare constraints for documentation and tooling, but rely on dbt tests for actual enforcement and referential integrity validation.

---

## When to Use Which Architecture

### Use Medallion Architecture When

- ✅ You want a simpler, more streamlined approach
- ✅ Cost optimization is important (Views in Integration layer)
- ✅ You're building a modern data platform
- ✅ You want to align with Databricks/Medallion terminology
- ✅ Business-friendly naming is a priority
- ✅ You want fewer layers to manage

### Use Traditional 5-Layer Architecture When

- ✅ You need clear separation between ingestion methods (DL vs STG)
- ✅ You want to separate enterprise DW models from team-specific BI views
- ✅ Performance is prioritized over cost (Tables everywhere)
- ✅ You have established ODS/DW/BI patterns
- ✅ Multiple teams need separate BI layers
- ✅ You're migrating from legacy architectures

---

## Migration Considerations

### Medallion → Traditional

- Split Raw layer into DL (cloud storage) and STG (Fivetran)
- Convert Integration Views to ODS Tables
- Split Analytics into DW (enterprise) and BI (team-specific)

### Traditional → Medallion

- Combine DL + STG into Raw layer
- Convert ODS Tables to Integration Views (if cost optimization desired)
- Combine DW + BI into Analytics layer
- Update naming conventions to Medallion patterns

---

## Recommendations

1. **For New Projects:** Consider Medallion Architecture for its simplicity and cost optimization
2. **For Existing Projects:** Evaluate migration cost vs. benefit; both architectures are valid
3. **Hybrid Approach:** Possible to use Medallion naming with Traditional layer separation
4. **Documentation:** Clearly document which architecture is in use and why

---

## References

- **Medallion Architecture:** `data-rules/dbt-datamodelling-snowflake.mdc`
- **Traditional Architecture:** `../coding_standards/rules/dbt/dbt.mdc`

---

## Origins & Historical Context

### Medallion Architecture Origins

**Origin:** The Medallion Architecture was introduced by **Databricks** as part of their Data Lakehouse paradigm, emerging around **2020-2021** as a modern approach to data architecture that combines the best aspects of data lakes and data warehouses.

**Philosophy:**

- **Data Lakehouse Concept:** Combines the flexibility and cost-effectiveness of data lakes with the ACID transactions and schema enforcement of data warehouses
- **Progressive Data Quality:** The bronze → silver → gold (or raw → integration → analytics) progression represents increasing data quality and refinement
- **Cloud-Native Design:** Built for modern cloud data platforms (Databricks, Snowflake, BigQuery) with separation of compute and storage
- **Agile & Iterative:** Designed to support rapid data ingestion and iterative refinement without heavy upfront schema design

**Key Drivers:**

1. **Cost Optimization:** Views in Integration layer reduce storage costs while maintaining flexibility
2. **Simplicity:** Fewer layers reduce complexity and maintenance overhead
3. **Modern Data Patterns:** Supports both batch and streaming data ingestion
4. **Self-Service Analytics:** Business-friendly naming in Analytics layer enables self-service BI

**Terminology:**

- The "Medallion" name refers to the progressive quality levels (bronze/silver/gold medals)
- Also known as "Multi-hop Architecture" or "Layered Architecture" in some contexts
- Aligns with modern data engineering practices and dbt's philosophy of incremental transformation

### Traditional Architecture Origins

**Origin:** The traditional 5-layer architecture (DL → STG → ODS → DW → BI) evolved from **classical data warehouse methodologies** developed in the **1980s-1990s** by pioneers like:

- **Bill Inmon** (Corporate Information Factory / CIF): Normalized, enterprise-wide data warehouse approach
- **Ralph Kimball** (Dimensional Modeling): Star schema approach for business intelligence
- **Operational Data Store (ODS) Concept:** Introduced by Inmon in the 1990s as an intermediate layer between operational systems and data warehouse

**Philosophy:**

- **Separation of Concerns:** Each layer has a distinct, well-defined purpose
- **Enterprise Architecture:** Designed for large-scale, enterprise-wide data integration
- **Structured Approach:** Heavy emphasis on schema design, normalization, and data quality gates
- **Performance-First:** Tables materialized at each layer for optimal query performance

**Key Drivers:**

1. **Enterprise Integration:** Supports multiple source systems with different ingestion patterns
2. **Clear Boundaries:** Distinct layers prevent mixing concerns (operational vs. analytical)
3. **Team Separation:** BI layer allows different teams to build their own views without affecting enterprise DW
4. **Proven Patterns:** Decades of refinement in enterprise data warehousing

**Evolution:**

- **1990s:** ODS concept emerges as intermediate layer
- **2000s:** Data Lake (DL) layer added for unstructured/big data
- **2010s:** Staging (STG) layer formalized for ELT tools like Fivetran
- **2020s:** Still widely used in enterprise environments with established data warehouse infrastructure

---

## Reasoning & Design Principles

### Medallion Architecture Reasoning

**Why 3 Layers?**

1. **Simplicity & Maintainability:**
   - Fewer layers reduce cognitive overhead and maintenance burden
   - Easier for new team members to understand and contribute
   - Reduced complexity in dependency management

2. **Cost Optimization:**
   - Integration layer defaults to Views (no storage cost)
   - Only Analytics layer materialized as Tables (where performance matters)
   - Aligns with cloud cost models (pay for compute, minimize storage)

3. **Agile Data Engineering:**
   - Supports rapid iteration and experimentation
   - Less rigid schema requirements enable faster time-to-value
   - Accommodates changing business requirements

4. **Modern Data Patterns:**
   - Designed for cloud-native platforms with separation of compute/storage
   - Supports both batch and streaming ingestion
   - Aligns with modern ELT (Extract-Load-Transform) patterns

5. **Business Alignment:**

   - Analytics layer focuses on business-friendly naming and self-service
   - Clear progression from technical (Raw) to business (Analytics)
   - Reduces need for translation layers

**Trade-offs:**

- Less granular control over ingestion patterns
- Combined DW/BI layer may create conflicts in enterprise environments
- Views in Integration layer may impact query performance for complex transformations

### Traditional Architecture Reasoning

**Why 5 Layers?**

1. **Separation of Concerns:**
   - **DL:** Handles unstructured/big data from cloud storage
   - **STG:** Manages ELT tool outputs (Fivetran, Stitch, etc.)
   - **ODS:** Provides operational reporting and normalized entities
   - **DW:** Enterprise-wide dimensional models
   - **BI:** Team-specific views without affecting enterprise models

2. **Enterprise Scale:**
   - Supports multiple ingestion patterns simultaneously
   - Allows different teams to work independently
   - Prevents conflicts between operational and analytical workloads

3. **Performance Optimization:**
   - Tables materialized at each layer ensure optimal query performance
   - Reduces compute costs for frequently accessed data
   - Supports complex transformations without view overhead

4. **Governance & Control:**
   - Clear boundaries enable better data governance
   - Easier to implement security and access controls per layer
   - Supports compliance and audit requirements

5. **Proven Methodology:**

   - Decades of refinement in enterprise environments
   - Well-understood patterns and best practices
   - Extensive tooling and support ecosystem

**Trade-offs:**

- Higher storage costs (Tables at every layer)
- More complex dependency management
- Longer time-to-value due to more layers
- Requires more upfront schema design

---

## Definitive Best Practices

### Medallion Architecture Best Practices

#### Raw Layer Best Practices

1. **Minimal Transformation:**
   - Load data as-is from source systems
   - Preserve source system structure and naming
   - Only add metadata columns (`__load_id`, `__load_dts`, `__source_filename`)

2. **Materialization Strategy:**
   - Use Tables for ELT tools (Fivetran, Stitch) - they handle materialization
   - Use Views for cloud storage ingestion (S3, GCS, Azure Blob) - transform on-read
   - Document materialization choice and rationale

3. **Naming Conventions:**
   - Use singular nouns matching source system names
   - Pattern: `raw_<party>__<source_system>__<table_name>`
   - Preserve cryptic source names but document them

4. **Data Quality:**
   - Implement basic data type validation
   - Capture data quality metrics (record counts, null percentages)
   - Log ingestion errors and failures

5. **Lineage Tracking:**
   - Always use dbt `sources` for external data
   - Document source system details in `sources.yml`
   - Track data freshness and SLAs

#### Integration Layer Best Practices

1. **Data Quality Focus:**
   - Implement data cleansing and standardization
   - Apply business rules and transformations
   - Handle deduplication and data quality issues

2. **Normalization:**
   - Create one-row-per-entity structures
   - Normalize data to 3NF where appropriate
   - Prepare data for dimensional modeling

3. **Materialization Strategy:**
   - Default to Views for cost optimization
   - Convert to Tables only when:
     - Query performance degrades significantly
     - Downstream models have complex dependencies
     - Data is accessed frequently by multiple consumers

4. **Soft Deletes:**
   - Always detect and flag deleted records
   - **Deletion Detection Methods:**
     - **Explicit Deletes:** Detect records with explicit delete flags or status indicators in source systems
     - **Missing from Incremental Loads:** If a record existed in a previous incremental load but does not appear in the current load, mark it as deleted (`is_deleted = true`)
     - **Missing from Type 2 SCD/Snapshot Queries:** If a record was previously returned by the source query supporting a Type 2 SCD or snapshot but no longer appears in the query results, mark it as deleted
   - Use `is_deleted` boolean flag
   - Preserve deleted records for historical analysis
   - **Important:** Always compare current load against previous state to detect records that have disappeared, not just those explicitly marked as deleted in source systems

5. **Surrogate Keys:**
   - Generate deterministic surrogate keys using SHA2-256
   - Always include natural keys alongside surrogates
   - Use consistent naming: `<entity>_key`

6. **Naming Conventions:**
   - Use singular nouns (one-row-per-entity)
   - Pattern: `int_<party>__<source>__<entity>` or `int_<subject_area>__<entity>`
   - Use business terminology, not source system names

#### Analytics Layer Best Practices

1. **Dimensional Modeling:**
   - Follow Kimball star schema methodology
   - Separate Facts (measures) from Dimensions (descriptors)
   - Use conformed dimensions for enterprise consistency
   - **Unknown Records:** All dimensions must include an "unknown" or "not applicable" record (typically with key = 0 or special hash value) to ensure referential integrity and enable equal joins (INNER JOIN) instead of LEFT JOIN. This allows fact tables to always join successfully even when foreign keys are NULL or don't match.
   - **Type 2 SCD Primary Keys:** Always use `dbt_utils.generate_surrogate_key(['natural_key', 'last_updated_at'])` to create surrogate keys. Use `last_updated_at` from source (not `dbt_valid_from` which isn't populated until snapshot publish, and not creation date which is immutable). Fact tables contain natural keys from source and join to dimensions using natural key + date range logic to get surrogate keys. When joining multiple Type 2 SCDs, each dimension joins independently using its natural key from the fact table.

2. **Business-Friendly Naming:**
   - Remove all source system references from names
   - Use business domain language
   - Expand abbreviations to full words
   - Standardize common terms across models

3. **Materialization:**
   - Always use Tables for performance
   - Never use Views in Analytics layer
   - Consider incremental models for large fact tables

4. **Naming Conventions:**
   - Facts: Plural, verb-based (`fact_orders`, `fact_sales`)
   - Dimensions: Plural, noun-based (`dim_customers`, `dim_products`)
   - Pattern: `anl_<subject_area>__<fact|dim>_<name>`

5. **Documentation:**
   - Every column must have business-friendly description
   - Document business glossary mappings
   - Include usage examples and sample queries
   - Document business domain owner

6. **Field Ordering:**
   - Follow dbt Labs standard: identifiers → strings → numerics → booleans → dates → timestamps → audit
   - Group related fields together
   - Put most frequently queried fields first

### Traditional Architecture Best Practices

#### Data Lake (DL) Layer Best Practices

1. **Cloud Storage Integration:**
   - Use Views for cloud storage data (S3, GCS, Azure Blob)
   - Implement schema-on-read patterns
   - Handle various file formats (Parquet, JSON, CSV)

2. **Data Ingestion:**
   - Support both batch and streaming ingestion
   - Implement data quality checks at ingestion
   - Track data lineage and provenance

3. **Naming & Organization:**
   - Organize by source system and date partitions
   - Use descriptive folder structures
   - Document data formats and schemas

#### Staging (STG) Layer Best Practices

1. **Source Definition:**
   - Define all sources in `sources.yml` files
   - Organize by source system in `sources/` folder
   - Document source freshness requirements

2. **ELT Tool Integration:**
   - Use dbt `sources` for Fivetran, Stitch, etc.
   - Never materialize staging layer (sources only)
   - Document source system schemas and changes

3. **Data Freshness:**
   - Implement freshness checks in `sources.yml`
   - Set appropriate `warn_after` and `error_after` thresholds
   - Monitor and alert on freshness violations

#### Operational Data Store (ODS) Best Practices

1. **Normalization:**
   - Normalize data to 3NF (Third Normal Form)
   - Create one-row-per-entity structures
   - Apply business rules and data quality rules

2. **Materialization:**
   - Always use Tables for performance
   - Support operational reporting needs
   - Optimize for frequent access patterns

3. **Naming Conventions:**
   - Use singular nouns (one-row-per-entity)
   - Pattern: `ods__<source>__<object>`
   - Use business terminology

4. **Data Quality:**
   - Implement comprehensive data quality checks
   - Handle soft deletes and data retention
   - Track data lineage from sources

5. **Surrogate Keys:**
   - Generate surrogate keys for all entities
   - Include natural keys for traceability
   - Use consistent naming patterns

#### Data Warehouse (DW) Layer Best Practices

1. **Dimensional Modeling:**
   - Follow Kimball star schema methodology
   - Use conformed dimensions for enterprise consistency
   - Implement slowly changing dimensions (SCD) appropriately
   - **Unknown Records:** All dimensions must include an "unknown" or "not applicable" record (typically with key = 0 or special hash value) to ensure referential integrity and enable equal joins (INNER JOIN) instead of LEFT JOIN. This allows fact tables to always join successfully even when foreign keys are NULL or don't match.

2. **Fact Tables:**
   - Use plural nouns, verb-based concepts
   - Pattern: `dw__<subject>__fact_<object>`
   - Define grain clearly (one row per what?)
   - Include degenerate dimensions appropriately

3. **Dimension Tables:**
   - Use plural nouns, noun-based entities
   - Pattern: `dw__<subject>__dim_<object>`
   - Implement SCD Type 1, 2, or 3 as needed
   - Include surrogate keys and natural keys

4. **Materialization:**
   - Always use Tables for performance
   - Consider incremental models for large facts
   - Use partitioning strategies for very large tables

5. **Field Ordering:**
   - Primary key first
   - Natural keys next
   - Foreign keys
   - Attributes
   - Measures (facts only)
   - Dates and timestamps
   - Audit columns last

6. **Documentation:**
   - Document grain for every fact table
   - Document SCD type for every dimension
   - Include business descriptions for all columns
   - Document relationships and hierarchies

#### Business Intelligence (BI) Layer Best Practices

1. **Team-Specific Views:**
   - Create team/project-specific aggregations
   - Never modify enterprise DW models
   - Use Views for simple transformations, Tables for complex aggregations

2. **Naming Conventions:**
   - Pattern: `bi__<team>__<object>` or `bi__<project>__<object>`
   - Use plural nouns
   - Document team/project ownership

3. **Materialization Strategy:**
   - Use Views for simple column selections/filters
   - Use Tables for complex aggregations or frequently accessed data
   - Consider performance vs. storage trade-offs

4. **Dependencies:**
   - Always reference DW layer models, never ODS or STG directly
   - Document business logic and calculations
   - Include usage examples

5. **Governance:**
   - Each team owns their BI layer models
   - Implement access controls per team
   - Regular review and cleanup of unused models

---

## Additional Resources

### Medallion Architecture

- **Databricks Documentation:** [Databricks Medallion Architecture](https://www.databricks.com/glossary/medallion-architecture)
- **Data Lakehouse Concept:** [What is a Data Lakehouse?](https://www.databricks.com/discover/data-lakehouse)
- **dbt Labs:** [dbt Best Practices](https://docs.getdbt.com/guides/best-practices)

### Traditional Architecture

- **Bill Inmon:** "Building the Data Warehouse" (1992) - Corporate Information Factory
- **Ralph Kimball:** "The Data Warehouse Toolkit" (1996) - Dimensional Modeling
- **Operational Data Store:** Inmon's ODS concept from 1990s
- **Data Warehouse Institute (TDWI):** Best practices and methodologies

### General Data Architecture

- **dbt Labs Style Guide:** [How we style our dbt models](https://docs.getdbt.com/best-practices/how-we-style/1-how-we-style-our-dbt-models)
- **Snowflake Best Practices:** [Snowflake Data Modeling Guide](https://docs.snowflake.com/en/user-guide/data-warehouse-overview.html)
