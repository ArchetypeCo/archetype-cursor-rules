# Archetype Cursor Rules

Comprehensive Cursor AI rules for dbt data modeling with Snowflake, enforcing Medallion architecture and dimensional modeling best practices.

## Overview

This repository contains Cursor AI rules that enforce consistent data modeling standards, SQL style guidelines, and YAML documentation patterns for dbt projects targeting Snowflake.

## Architecture

We follow a simplified **Medallion Architecture** with three layers:

- **`raw`**: Source data as-is with minimal transformation
- **`int`**: Integration layer for cleansing, conforming, and normalization
- **`anl`**: Analytics layer with dimensional models (facts & dimensions)

## Rules Structure

### `rules/dbt-datamodelling-snowflake.mdc`
Comprehensive Snowflake data modeling rules covering:
- Naming conventions (layer-specific: singular for raw/int, plural for anl)
- Medallion architecture patterns
- Dimensional modeling (Kimball methodology)
- Physical modeling & constraints
- Advanced patterns (SCD Type 1/2, fact tables)
- Snowflake optimization strategies
- Governance & documentation standards

### `rules/dbt-sql-style.mdc`
SQL formatting and style rules:
- General formatting (keywords, indentation, line length)
- CTE structure and organization
- Query organization patterns
- Join discipline
- dbt-specific conventions

### `rules/dbt-yaml-style.mdc`
YAML documentation and testing standards:
- Formatting basics
- File organization
- Documentation & testing patterns
- Example templates

### `rules/dbt-macro-best-practices.mdc`
Macro development and usage standards:
- When to use macros vs CTEs vs models
- Macro naming conventions and organization
- Documentation standards and templates
- Common macro patterns (surrogate keys, soft deletes, utilities)
- Error handling and validation
- dbt-utils package usage
- Performance considerations

## Key Naming Conventions

### dbt Model Naming
- **Raw Layer**: `raw_<party>__<source_system>__<table_name>`
- **Integration Layer**: `int_<party>__<source_system>__<entity>` or `int_<subject_area>__<entity>`
- **Analytics Layer**: `anl_<subject_area>__<fact|dim>_<name>`

### Table Naming
- **Raw/Int**: Singular nouns (`customer`, `sales_order`)
- **Analytics**: Plural nouns (`fact_orders`, `dim_customers`)

### Primary Keys
- Format: `<singular_table_name>_key`
- Type: SHA2-256 hash of natural keys stored as `VARCHAR`
- Always include natural key columns alongside surrogate key

### Column Naming
- Timestamps: `_created_datetime`, `_updated_datetime` (UTC)
- Dates: `_created_on`, `_updated_on`
- Booleans: `is_` or `has_` prefix
- Field ordering: Identifiers → Attributes → Activity dates → Audit fields

## Usage

1. Copy the `rules/` directory to your dbt project root
2. Cursor AI will automatically apply these rules to `.sql`, `.yml`, and `.yaml` files
3. Rules enforce naming conventions, SQL style, and documentation standards

## Documentation

See `docs/GAP_ANALYSIS.md` for a detailed analysis of naming convention gaps and best practices alignment.

## Contributing

When updating rules:
1. Ensure consistency across all rule files
2. Update this README if adding new conventions
3. Test rules with actual dbt models to verify enforcement

## License

[Add your license here]

## References

- [Data Modeling with Snowflake, Second Edition](https://www.snowflake.com/)
- [dbt Style Guide](https://github.com/dbt-labs/corp/blob/main/dbt_style_guide.md)
- [Kimball Dimensional Modeling](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/)

