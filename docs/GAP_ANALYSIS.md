# dbt Cursor Rules - Gap Analysis & Best Practices Review

## Executive Summary

This document identifies critical gaps and inconsistencies between the current cursor rules and industry best practices, with a focus on **naming convention enforcement**. Key conflicts exist between the Snowflake modeling rules (singular nouns) and the organization's style guide (plural facts/dimensions, layer prefixes).

---

## 🔴 CRITICAL NAMING CONVENTION CONFLICTS

### 1. Table/Model Naming: Singular vs Plural

**Current State:**
- **Snowflake rules (line 17-19):** Enforces **singular nouns** (`customer`, `sales_order`)
- **Style guide (line 114-116):** 
  - ODS objects: **singular** (`salesforce__account`)
  - DW/BI objects: **plural** (`fact_orders`, `dim_customers`)

**Gap:** These are **directly contradictory**. The Snowflake rules will enforce singular for ALL tables, but the style guide requires plural for facts/dimensions.

**Recommendation:** 
- **ODS/Staging:** Singular (`customer`, `sales_order`)
- **DW Facts:** Plural (`fact_orders`, `fact_sales`)
- **DW Dimensions:** Plural (`dim_customers`, `dim_products`)
- **BI Marts:** Plural (business terminology)

**Action Required:** Update Snowflake rules to specify layer-specific naming.

---

### 2. Primary Key Naming: `_id` vs `_key`

**Current State:**
- **Snowflake rules (line 20-21):** `<singular_table_name>_key` with SHA2-256 hash ✅ (recently updated)
- **Style guide (line 122-123):** `<object>_key` with SHA2-256 hash ✅

**Status:** ✅ **ALIGNED** - Both now specify `_key` with SHA2-256 hash.

---

### 3. Model Prefix Patterns Missing

**Current State:**
- **Snowflake rules:** No mention of dbt model naming patterns
- **Style guide (line 109-116):** 
  - Layer prefixes: `stg_`, `ods_`, `dw_` (with `__` delimiter)
  - Subject area prefixes: `sales__dim_account_manager`
  - Party/system prefixes: `party_1__source_system_1__table_1`

**Gap:** Snowflake rules don't enforce dbt model naming conventions at all.

**Missing Patterns:**
- `stg_<party>__<source>__<table>` for staging
- `ods_<party>__<source>__<table>` for ODS
- `<subject_area>__<fact|dim>_<name>` for DW
- `bi_<team>__<project>__<model>` for BI

**Action Required:** Add comprehensive dbt model naming section to Snowflake rules.

---

### 4. Foreign Key Naming Inconsistency

**Current State:**
- **Snowflake rules (line 22):** Use exact name of referenced PK (e.g., `customer_key`)
- **Style guide (line 138):** Use consistent naming (e.g., `customer_id` not `user_id`)

**Gap:** Style guide example uses `customer_id` but rules specify `customer_key`. Need clarity on FK naming when PK is `_key`.

**Action Required:** Clarify FK naming: should be `<referenced_table>_key` to match PK pattern.

---

## 🟡 MISSING NAMING CONVENTIONS

### 5. Column Naming Patterns Not Enforced

**Missing from Rules:**
- **Timestamp naming:** `_created_datetime` / `_updated_datetime` vs `_created_at` / `_updated_at` (style guide allows both)
- **Date vs Timestamp:** `_on` for dates, `_at` for timestamps (style guide line 127-129)
- **Boolean prefixes:** `is_` or `has_` (style guide line 132)
- **Currency fields:** Decimal vs `_in_cents` suffix (style guide line 135)
- **Lookup table pattern:** `<object>_KEY`, `<object>_CODE`, `<object>_DESCRIPTION` (style guide line 140-143)
- **Field ordering:** Identifiers → Attributes → Activity dates → Audit fields (style guide line 125)

**Action Required:** Add comprehensive column naming standards section.

---

### 6. Layer-Specific Naming Rules Missing

**Current State:**
- **Snowflake rules:** Generic "singular nouns" for all tables
- **Style guide:** Layer-specific rules:
  - **DL/STG:** Follow source naming (may need views for cryptic names like `F0911`)
  - **ODS:** Singular, business terminology
  - **DW:** Plural, star schema (`fact_*`, `dim_*`)
  - **BI:** Plural, business terminology

**Action Required:** Add layer-specific naming rules to Snowflake rules.

---

## 🟠 ARCHITECTURE MISMATCH

### 7. Medallion vs Custom Layer Architecture

**Current State:**
- **Snowflake rules (line 30-44):** Medallion Architecture (source/raw → integration → analytics/reporting)
- **Style guide (line 3-49):** Custom layers (DL → STG → ODS → DW → BI)

**Gap:** Two different architectural paradigms. Need to reconcile or specify when to use which.

**Questions to Resolve:**
- Does Medallion map to custom layers? (source=DL, integration=STG/ODS, analytics=DW/BI?)
- Which architecture takes precedence?
- Should rules support both?

**Action Required:** Map Medallion layers to organization's layer structure or choose one.

---

## 🔵 MISSING BEST PRACTICES

### 8. Model Materialization Strategy

**Missing:**
- When to use `view` vs `table` vs `incremental`
- Incremental model patterns (unique_key, strategy)
- Table materialization for marts (style guide line 78)
- Performance considerations for materialization choice

**Action Required:** Add materialization guidance section.

---

### 9. dbt Model Configuration Patterns

**Missing:**
- In-model `config()` block formatting (style guide line 67-77)
- Directory-level configs in `dbt_project.yml`
- Tags for build grouping (style guide line 80-82)
- Schema/database overrides

**Action Required:** Add dbt configuration standards section.

---

### 10. Snapshot Patterns

**Missing:**
- Snapshot naming (`_HIST` suffix? - style guide line 85)
- Snapshot folder structure mirroring models (style guide line 86)
- Snapshot strategy documentation in YAML (style guide line 88)
- End-dating deleted records (style guide line 88)

**Action Required:** Add snapshot standards section.

---

### 11. Soft Delete Patterns

**Missing:**
- Soft delete detection in STG/ODS (style guide line 91)
- Boolean flag naming (`is_deleted`, `is_active`)
- Logical deletion vs physical deletion handling

**Action Required:** Add soft delete handling section.

---

### 12. Source Definition Patterns

**Missing:**
- Source naming conventions (`party__system` pattern)
- Source freshness checks
- Source metadata columns (`__load_id`, `__load_dts`, `__source_filename` - mentioned but not enforced)

**Action Required:** Add source definition standards section.

---

### 13. Testing Patterns Beyond Basics

**Missing:**
- Test organization by layer
- Custom test patterns
- Test naming conventions
- Test coverage expectations by layer
- Relationship test patterns for FKs

**Action Required:** Expand testing section beyond YAML basics.

---

### 14. Macro Usage Patterns

**Missing:**
- When to create macros vs CTEs
- Macro naming conventions
- Reusable transformation patterns
- dbt-utils package usage

**Action Required:** Add macro standards section.

---

### 15. Documentation Standards

**Missing:**
- Model description requirements (purpose, change log, developer - style guide line 54-61)
- Column description standards
- Business glossary integration
- Lineage documentation expectations

**Action Required:** Expand documentation section beyond YAML formatting.

---

## 📋 QUESTIONS TO RESOLVE

### High Priority

1. **Table naming:** Singular for ODS, plural for DW/BI? Or singular everywhere?
2. **Model prefixes:** Enforce `stg_`, `ods_`, `dw_`, `bi_` prefixes? With `__` delimiter?
3. **Fact/Dimension prefixes:** Require `fact_` and `dim_` prefixes for DW layer?
4. **Architecture:** Medallion vs custom layers - which takes precedence?
5. **FK naming:** Always match PK name (`customer_key` → `customer_key`)?

### Medium Priority

6. **Timestamp naming:** Standardize on `_datetime` vs `_at` vs `_on`?
7. **Field ordering:** Enforce identifier → attribute → audit ordering?
8. **Natural keys:** Always include alongside surrogate keys?
9. **Materialization:** Default strategies by layer?
10. **Tags:** Enforce tagging strategy for model grouping?

### Low Priority

11. **Abbreviations:** Whitelist of acceptable abbreviations?
12. **Reserved words:** Maintain a list of Snowflake reserved words to avoid?
13. **Macro patterns:** Standard macros for common transformations?
14. **Seed files:** Naming and structure conventions?

---

## 🎯 RECOMMENDED ACTIONS

### Immediate (Critical Conflicts)

1. ✅ **Resolve singular/plural conflict** - Add layer-specific naming rules
2. ✅ **Add dbt model naming patterns** - Enforce prefixes and delimiters
3. ✅ **Clarify FK naming** - Match PK naming pattern
4. ✅ **Map Medallion to custom layers** - Resolve architecture conflict

### Short Term (Missing Standards)

5. **Add column naming patterns** - Timestamps, booleans, currency, lookups
6. **Add materialization guidance** - When to use view/table/incremental
7. **Add snapshot patterns** - Naming, structure, strategies
8. **Add soft delete patterns** - Detection and handling

### Medium Term (Enhancements)

9. **Expand testing patterns** - Beyond basic unique/not_null
10. **Add macro standards** - When and how to use macros
11. **Enhance documentation** - Model descriptions, change logs
12. **Add source patterns** - Naming, freshness, metadata

---

## 📊 COMPLIANCE CHECKLIST

Use this to verify rules cover all naming conventions:

- [ ] Table naming (singular vs plural by layer)
- [ ] Model naming (prefixes, delimiters, patterns)
- [ ] Primary key naming (`_key` with SHA2-256)
- [ ] Foreign key naming (match PK pattern)
- [ ] Column naming (timestamps, booleans, currency)
- [ ] Field ordering (identifiers → attributes → audit)
- [ ] Constraint naming (PK, FK, unique)
- [ ] Source naming (`party__system`)
- [ ] Snapshot naming (`_HIST` suffix?)
- [ ] Macro naming
- [ ] Test naming
- [ ] Tag naming conventions

---

## 🔗 REFERENCES

- Organization Style Guide: `dbt_style_guide (2).md`
- Current Rules:
  - `rules/dbt-datamodelling-snowflake.mdc`
  - `rules/dbt-sql-style.mdc`
  - `rules/dbt-yaml-style.mdc`

