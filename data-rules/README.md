# Data Rules Documentation

This directory contains comprehensive data modeling rules and style guides for dbt projects. These rules ensure consistency, maintainability, and best practices across all data transformation work.

## Overview

The data rules are organized into architecture-specific modeling guides and cross-cutting style guides that apply regardless of which architecture you choose.

## Architecture Selection

**Important:** Before starting any dbt project, you must choose an architecture based on client needs:

1. **Medallion Architecture** (`dbt-datamodelling-medallion.mdc`) - 3-layer approach (Raw → Integration → Analytics)
2. **Traditional 5-Layer Architecture** (`dbt-datamodelling-traditional.mdc`) - 5-layer approach (DL → STG → ODS → DW → BI)

**Do not arbitrarily choose one architecture.** Talk to the client about their requirements, understand their data organization needs, and select the appropriate architecture. Once chosen, use that architecture's patterns consistently throughout the project.

## File Structure

### Architecture Comparison

* **`architecture-layers-comparison.md`** - Comprehensive comparison document
  * **Start here first** - This document drives the decision between Medallion and Traditional approaches
  * Compares Medallion vs Traditional 5-Layer architectures
  * Includes: Layer purposes, naming conventions, materialization strategies, origins, reasoning, best practices, migration considerations
  * Use this document to understand architectural differences and make informed decisions
  * Review this before selecting an architecture

### Architecture-Specific Modeling Guides

* **`dbt-datamodelling-medallion.mdc`** - Complete dbt data modeling rules for Medallion Architecture
  * Includes: Naming standards, architectural layers (Raw, Integration, Analytics), SQL style, CTE structure, field ordering, surrogate keys, joins, booleans, model configuration, testing, model headers, macros, sources, Jinja style, packages, Git workflow, and anti-patterns
  * Database-agnostic with Snowflake-specific optimizations where applicable
  * Set `alwaysApply: false` - activate only when using Medallion Architecture

* **`dbt-datamodelling-traditional.mdc`** - Complete dbt data modeling rules for Traditional 5-Layer Architecture
  * Includes: Naming standards, architectural layers (DL, STG, ODS, DW, BI), SQL style, CTE structure, field ordering, surrogate keys, joins, booleans, model configuration, testing, model headers, macros, sources, Jinja style, packages, Git workflow, and anti-patterns
  * Database-agnostic with Snowflake-specific optimizations where applicable
  * Set `alwaysApply: false` - activate only when using Traditional Architecture

### Cross-Cutting Style Guides

These style guides apply regardless of which architecture you choose:

* **`dbt-sql-style.mdc`** - SQL formatting and query organization rules
  * General formatting (keywords, trailing commas, indentation, line length)
  * CTE structure and organization
  * Query organization and join discipline
  * dbt-specific notes (layer dependencies, model naming, snapshots)
  * Set `alwaysApply: true` - always active

* **`dbt-yaml-style.mdc`** - YAML formatting and documentation rules
  * Formatting basics (indentation, line length, quoting)
  * File organization (schema.yml, sources.yml)
  * Documentation and testing requirements
  * Unit testing guidance (dbt Core 1.8+)
  * Example templates
  * Set `alwaysApply: true` - always active

* **`dbt-macro-best-practices.mdc`** - Macro creation and usage guidelines
  * When to use macros vs CTEs vs models
  * Macro naming conventions
  * Macro organization and structure
  * Documentation requirements
  * Testing strategies
  * Performance considerations
  * Set `alwaysApply: true` - always active

## How to Use These Rules

### Step 1: Choose Your Architecture

1. Review `architecture-layers-comparison.md` to understand both architectures
2. Discuss with the client their data organization needs
3. Select either Medallion or Traditional architecture
4. Document the decision and reasoning

### Step 2: Activate Architecture-Specific Rules

In your Cursor rules configuration, activate the appropriate architecture file:

**For Medallion Architecture:**

* Activate `dbt-datamodelling-medallion.mdc`
* Deactivate `dbt-datamodelling-traditional.mdc`

**For Traditional Architecture:**

* Activate `dbt-datamodelling-traditional.mdc`
* Deactivate `dbt-datamodelling-medallion.mdc`

### Step 3: Style Guides Are Always Active

The following files should always be active (they have `alwaysApply: true`):

* `dbt-sql-style.mdc` - SQL formatting rules
* `dbt-yaml-style.mdc` - YAML documentation rules
* `dbt-macro-best-practices.mdc` - Macro guidelines

### Step 4: Reference During Development

Use these rules as your guide when:

* Creating new dbt models
* Writing SQL transformations
* Defining YAML schemas and tests
* Creating macros
* Making architectural decisions
* Reviewing code

## Key Concepts

### Type 2 Slowly Changing Dimensions (SCD)

Both architecture guides include comprehensive guidance on Type 2 SCDs:

* **Primary Key Generation:** Always use `dbt_utils.generate_surrogate_key(['natural_key', 'last_updated_at'])` for surrogate keys
* **Fact Table Joins:** Join using natural key + date range logic to find the correct surrogate key version
* **Multi-Dimension Joins:** Each Type 2 SCD joins independently using its natural key from the fact table

See the architecture-specific files for detailed examples and patterns.

### Database-Specific Optimizations

Both architecture guides include Snowflake-specific optimizations:

* **Clustering:** Only apply `CLUSTER BY` for multi-terabyte tables
* **Micro-partitions:** Rely on natural ingestion order for initial pruning
* **Virtual Columns:** Use for simple transformation logic without storage overhead
* **Snowflake Streams:** For Type 2 SCD change detection (up to 40% faster)
* **Dynamic Tables:** Declarative alternative for Type 2 dimensions

### Constraints and Testing

* **Constraints:** Declared for documentation and tooling (not enforced in cloud platforms)
* **dbt Tests:** Required for data integrity enforcement (primary keys, foreign keys, unique constraints)
* **Unit Tests:** Use dbt Unit Tests (dbt Core 1.8+) for complex SQL logic validation

## Best Practices Summary

1. **Choose Architecture First:** Don't start coding until architecture is selected
2. **Consistency:** Use the selected architecture's patterns throughout the project
3. **Documentation:** Document all models, columns, and business logic in YAML
4. **Testing:** Write tests for primary keys, foreign keys, and complex logic
5. **Naming:** Follow the architecture-specific naming conventions consistently
6. **Layer Dependencies:** Never skip layers (e.g., Analytics cannot reference Raw directly)
7. **Macros:** Use macros for reusable logic (3+ models), CTEs for model-specific logic
8. **SQL Style:** Follow the SQL style guide for formatting and organization

## Additional Resources

* **Architecture Comparison:** See `architecture-layers-comparison.md` for detailed architectural guidance
* **SQL Style:** See `dbt-sql-style.mdc` for query formatting and organization
* **YAML Style:** See `dbt-yaml-style.mdc` for documentation and testing patterns
* **Macro Best Practices:** See `dbt-macro-best-practices.mdc` for macro creation guidelines

## Questions?

If you're unsure which architecture to use or how to apply these rules:

1. Review `architecture-layers-comparison.md` for architectural guidance
2. Consult with the Data Engineering Lead
3. Discuss with the client about their specific requirements
4. Reference the architecture-specific modeling guides for detailed patterns
