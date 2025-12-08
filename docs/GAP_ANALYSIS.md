# dbt Cursor Rules - Gap Analysis & Best Practices Review

## Executive Summary

This document identifies critical gaps and inconsistencies between the current cursor rules and industry best practices, with a focus on **naming convention enforcement**. Key conflicts exist between:

- Snowflake modeling rules (singular nouns for raw/int, plural for analytics)
- Organization's style guide (plural facts/dimensions, layer prefixes)
- **dbt Labs official style guide** ([source](https://docs.getdbt.com/best-practices/how-we-style/1-how-we-style-our-dbt-models)) which recommends **plural models** and **`_id` primary keys** (not `_key`)

**NEW FINDINGS:** Comparison with dbt Labs official style guide reveals significant conflicts in primary key naming (`_id` vs `_key`), timestamp naming (`_at` vs `_datetime`), and model pluralization approach.

---

## 🔴 CRITICAL NAMING CONVENTION CONFLICTS

### 1. Table/Model Naming: Singular vs Plural ⚠️ **CONFLICT WITH dbt LABS**

**Current State:**

- **Snowflake rules (line 17-26):** Layer-specific:
  - Raw/Int: **singular nouns** (`customer`, `sales_order`)
  - Analytics: **plural nouns** (`fact_orders`, `dim_customers`)
- **Organization style guide:**
  - ODS objects: **singular** (`salesforce__account`)
  - DW/BI objects: **plural** (`fact_orders`, `dim_customers`)
- **dbt Labs style guide:** **ALL models should be pluralized** (`customers`, `orders`, `products`)

**Gap:** **CONFLICT** - dbt Labs recommends plural for ALL models, but our rules use singular for raw/int layers.

**dbt Labs Guidance:**

- Models should be pluralized: `customers`, `orders`, `products` (not `customer`, `order`, `product`)
- This applies to ALL models regardless of layer

**Current Implementation:**

- ✅ Raw/Int: Singular (preserves source system naming, aligns with relational model theory)
- ✅ Analytics: Plural (aligns with dbt Labs and dimensional modeling)

**Decision Required:**

- **Option A:** Keep layer-specific approach (singular raw/int, plural analytics) - **Current approach**
- **Option B:** Switch all layers to plural (aligns with dbt Labs completely)
- **Option C:** Document the deviation from dbt Labs with rationale (dimensional modeling best practice)

**Decision (Nov 2025):** Keep the layer-specific approach permanently. Raw + Integration layers stay singular to emphasize single-entity rows, while Analytics remains plural (`fact_*/dim_*`) for aggregated/business-friendly datasets. Every rule file must document this deliberate divergence from dbt Labs so future contributors do not “correct” it back to all-plural naming.

---

### 2. Primary Key Naming: `_id` vs `_key` ⚠️ **CRITICAL CONFLICT WITH dbt LABS**

**Current State:**

- **Snowflake rules (line 35-37):** `<singular_table_name>_key` with SHA2-256 hash
- **Organization style guide:** `<object>_key` with SHA2-256 hash ✅
- **dbt Labs style guide:** `<object>_id` (e.g., `account_id`, `customer_id`) - **NO mention of SHA2-256 hashing**

**Gap:** **MAJOR CONFLICT** - dbt Labs official style guide uses `_id` suffix, not `_key`. Our rules use `_key` with SHA2-256 hashing for surrogate keys, which is a dimensional modeling best practice but conflicts with dbt Labs convention.

**dbt Labs Guidance:**

- Primary key should be named `<object>_id` (e.g., `account_id`, `customer_id`)
- Keys should be **string data types**
- Makes it easier to know what `id` is being referenced in downstream joined models

**Decision Required:**

- **Option A:** Keep `_key` with SHA2-256 (dimensional modeling best practice, better for data warehousing)
- **Option B:** Switch to `_id` (aligns with dbt Labs, more common in application databases)
- **Option C:** Layer-specific - use `_id` for raw/int layers, `_key` for analytics layer (hybrid approach)

#### Decision (Nov 2025)

Standardize on `_key` SHA2-256 surrogate keys in **every layer** (raw, int, anl). When a natural/source column already ends with `_key`, name the surrogate `<entity>_pk` to prevent duplicate column names; foreign keys must mirror whichever pattern the parent table uses. Document this in all rules so analysts understand the Kimball-aligned rationale for deviating from dbt Labs

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
- Decision: map Medallion terminology to the legacy stack for clarity — `raw` = landing, `int` = curated/ODS/integration, `anl` = marts/BI. Every rule file must describe this translation so both vocabularies stay in sync.

**Action Required:** Add comprehensive dbt model naming section to Snowflake rules.

---

### 4. Foreign Key Naming Inconsistency

**Current State:**

- **Snowflake rules (line 22):** Use exact name of referenced PK (e.g., `customer_key`)
- **Style guide (line 138):** Use consistent naming (e.g., `customer_id` not `user_id`)

**Gap:** Style guide example uses `customer_id` but rules specify `customer_key`. Need clarity on FK naming when PK is `_key`.

**Action Required:** Clarify FK naming: should be `<referenced_table>_key` to match PK pattern. Decision: foreign keys always mirror the parent’s surrogate (`<entity>_key`), except when the parent had to fall back to `<entity>_pk` because the source already owned `<entity>_key`.

---

## 🟡 MISSING NAMING CONVENTIONS

### 5. Column Naming Patterns Not Enforced ⚠️ **CONFLICTS WITH dbt LABS**

**Current State (legacy vs updated):**

- **Legacy rules (pre-Nov 2025):** Timestamps used `_created_datetime` / `_updated_datetime`, dates used `_created_on` / `_updated_on`
- **Updated rules:** Now use `<event>_at` and `<event>_date` per dbt Labs guidance
- **dbt Labs style guide:**
  - Timestamps: `<event>_at` (e.g., `created_at`, `updated_at`) - **UTC by default**
  - Dates: `<event>_date` (e.g., `created_date`, `updated_date`)
  - Events should be **past tense** (`created`, `updated`, `deleted`)

**Gap:** **MAJOR CONFLICT** - Our timestamp naming (`_datetime`) conflicts with dbt Labs (`_at`). Our date naming (`_on`) conflicts with dbt Labs (`_date`).

**dbt Labs Guidance:**

- Timestamp columns: `<event>_at` (e.g., `created_at`, `updated_at`) in UTC
- Date columns: `<event>_date` (e.g., `created_date`, `updated_date`)
- If different timezone, indicate with suffix: `created_at_pt`
- Events should be past tense: `created`, `updated`, `deleted`
- Decision: adopt the dbt Labs timestamp/date guidance wholesale and propagate it across README + rule files.

**Other Patterns (Aligned):**

- ✅ **Booleans:** `is_` or `has_` prefix (both agree)
- ✅ **Currency:** Decimal format (`19.99`), `_in_cents` suffix if non-decimal (both agree)
- ✅ **Lookup tables:** Pattern with `_key`, `_code`/`_name`, `_description` (our rules)
- ⚠️ **Field ordering:**
  - **Our rules:** Identifiers → Attributes → Activity dates → Audit fields
  - **dbt Labs:** ids → strings → numerics → booleans → dates → timestamps (more granular)
 ANSWER: lets follow dbt labs standards,

**Action Required:**

1. **Decide on timestamp/date naming:** `_at`/`_date` (dbt Labs) vs `_datetime`/`_on` (current)
2. **Update field ordering** to match dbt Labs pattern or document deviation
3. Add comprehensive column naming standards section

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

## 🔵 MISSING BEST PRACTICES FROM dbt LABS

### 8. Model Versioning Pattern ⚠️ **NEW FROM dbt LABS**

**Missing:**

- **dbt Labs guidance:** Model versions should use `_v1`, `_v2`, etc. suffix for consistency
  - Examples: `customers_v1`, `customers_v2`
  - Used when creating new versions of existing models

**Current State:** No versioning pattern defined in our rules.

**Action Required:** Add model versioning standards section.

---

### 9. Model Materialization Strategy

**Missing:**

- When to use `view` vs `table` vs `incremental`
- Incremental model patterns (unique_key, strategy)
- Table materialization for marts (style guide line 78)
- Performance considerations for materialization choice

**Action Required:** Add materialization guidance section.

---

### 10. dbt Model Configuration Patterns

**Missing:**

- In-model `config()` block formatting (style guide line 67-77)
- Directory-level configs in `dbt_project.yml`
- Tags for build grouping (style guide line 80-82)
- Schema/database overrides

**Action Required:** Add dbt configuration standards section.

---

### 11. Snapshot Patterns

**Missing:**

- Snapshot naming (`_HIST` suffix? - style guide line 85)
- Snapshot folder structure mirroring models (style guide line 86)
- Snapshot strategy documentation in YAML (style guide line 88)
- End-dating deleted records (style guide line 88)

**Action Required:** Add snapshot standards section.

---

### 12. Soft Delete Patterns

**Missing:**

- Soft delete detection in STG/ODS (style guide line 91)
- Boolean flag naming (`is_deleted`, `is_active`)
- Logical deletion vs physical deletion handling

**Action Required:** Add soft delete handling section.

---

### 13. Source Definition Patterns

**Missing:**

- Source naming conventions (`party__system` pattern)
- Source freshness checks
- Source metadata columns (`__load_id`, `__load_dts`, `__source_filename` - mentioned but not enforced)

**Action Required:** Add source definition standards section.

---

### 14. Testing Patterns Beyond Basics

**Missing:**

- Test organization by layer
- Custom test patterns
- Test naming conventions
- Test coverage expectations by layer
- Relationship test patterns for FKs

**Action Required:** Expand testing section beyond YAML basics.

---

### 15. Macro Usage Patterns ✅ **COMPLETED**

**Status:** ✅ **RESOLVED** - Comprehensive macro best practices document created.

**New Rules File:** `rules/dbt-macro-best-practices.mdc`

**Coverage:**

- ✅ When to use macros vs CTEs vs models (decision tree)
- ✅ Macro naming conventions (`macro_<verb>_<noun>` pattern)
- ✅ Macro organization and file structure
- ✅ Documentation standards with templates
- ✅ Parameter conventions and validation
- ✅ Common macro patterns (surrogate keys, soft deletes, type casting, date utilities)
- ✅ Error handling and input validation
- ✅ Testing macros
- ✅ dbt-utils package usage guidance
- ✅ Performance considerations
- ✅ Versioning and migration patterns
- ✅ Macro review checklist

---

### 16. Documentation Standards

**Missing:**

- Model description requirements (purpose, change log, developer - style guide line 54-61)
- Column description standards
- Business glossary integration
- Lineage documentation expectations

**Action Required:** Expand documentation section beyond YAML formatting.

---

## 📋 QUESTIONS TO RESOLVE

### High Priority

1. **Table naming:** Singular for raw/int, plural for analytics? Or plural everywhere (per dbt Labs)?
2. **Primary key naming:** Use `_id` (dbt Labs) or `_key` with SHA2-256 (dimensional modeling)? Layer-specific?
3. **Timestamp naming:** Use `_at` (dbt Labs) or `_datetime` (current)? Date: `_date` or `_on`?
4. **Model prefixes:** Enforce `raw_`, `int_`, `anl_` prefixes? With `__` delimiter?
5. **Fact/Dimension prefixes:** Require `fact_` and `dim_` prefixes for analytics layer?
6. **FK naming:** Always match PK name (`customer_key` → `customer_key`)?
7. **Field ordering:** Use dbt Labs pattern (ids → strings → numerics → booleans → dates → timestamps) or current (identifiers → attributes → audit)?
8. **Repository-wide alignment:** Should every rules file (README, architecture, QA, DevOps) reference the same authoritative naming conventions, or can documents intentionally diverge? Document the decision to prevent future drift.

### Medium Priority

1. **Model versioning:** Add `_v1`, `_v2` suffix pattern for model versions?
2. **Natural keys:** Always include alongside surrogate keys?
3. **Materialization:** Default strategies by layer?
4. **Tags:** Enforce tagging strategy for model grouping?
5. **Keys data type:** Enforce string data types for keys (per dbt Labs)?

### Low Priority

1. **Abbreviations:** Whitelist of acceptable abbreviations?
2. **Reserved words:** Maintain a list of Snowflake reserved words to avoid?
3. **Macro patterns:** Standard macros for common transformations?
4. **Seed files:** Naming and structure conventions?

---

## 🎯 RECOMMENDED ACTIONS

### Immediate (Critical Conflicts)

1. ✅ **Resolve singular/plural conflict** - Add layer-specific naming rules
2. ✅ **Add dbt model naming patterns** - Enforce prefixes and delimiters
3. ✅ **Clarify FK naming** - Match PK naming pattern
4. ✅ **Map Medallion to custom layers** - Resolve architecture conflict

### Short Term (Missing Standards)

1. **Resolve timestamp/date naming conflict** - Choose `_at`/`_date` (dbt Labs) or `_datetime`/`_on` (current)
2. **Resolve primary key naming** - Document decision on `_id` vs `_key` with rationale
3. **Add column naming patterns** - Timestamps, booleans, currency, lookups (aligned with chosen standard)
4. **Add model versioning pattern** - `_v1`, `_v2` suffix for model versions
5. **Add materialization guidance** - When to use view/table/incremental
6. **Add snapshot patterns** - Naming, structure, strategies
7. **Add soft delete patterns** - Detection and handling

### Medium Term (Enhancements)

1. **Expand testing patterns** - Beyond basic unique/not_null
2. ✅ **Add macro standards** - When and how to use macros **COMPLETED**
3. **Enhance documentation** - Model descriptions, change logs
4. **Add source patterns** - Naming, freshness, metadata

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
- [x] Macro naming ✅ **COMPLETED**
- [ ] Test naming
- [ ] Tag naming conventions

---

## 🔗 REFERENCES

- **dbt Labs Official Style Guide:** [How we style our dbt models](https://docs.getdbt.com/best-practices/how-we-style/1-how-we-style-our-dbt-models)
- Organization Style Guide: `dbt_style_guide (2).md`
- Current Rules:
  - `rules/dbt-datamodelling-snowflake.mdc`
  - `rules/dbt-sql-style.mdc`
  - `rules/dbt-yaml-style.mdc`

---

## 📝 SUMMARY OF dbt LABS CONFLICTS

### Critical Conflicts Requiring Decision

1. **Primary Key Naming:**
   - **dbt Labs:** `<object>_id` (e.g., `account_id`)
   - **Our Rules:** `<singular_table_name>_key` with SHA2-256 hash
   - **Impact:** High - affects all models
   - **Status:** Resolved — `_key` enforced across every layer (Nov 2025) with `_pk` fallback only when the source already has `<entity>_key`.
2. **Model Pluralization:**
   - **dbt Labs:** ALL models plural (`customers`, `orders`)
   - **Our Rules:** Singular for raw/int, plural for analytics
   - **Impact:** Medium - affects naming consistency
   - **Status:** Resolved — layer-specific approach retained intentionally; rationale documented.

3. **Timestamp Naming:**
   - **dbt Labs:** `<event>_at` (e.g., `created_at`)
   - **Our Rules:** `<object>_created_datetime`
   - **Impact:** High - affects all timestamp columns
   - **Status:** Resolved — `<event>_at` adopted across README + rule files.

4. **Date Naming:**
   - **dbt Labs:** `<event>_date` (e.g., `created_date`)
   - **Our Rules:** (legacy) `<object>_created_on`
   - **Impact:** Medium - affects all date columns
   - **Status:** Resolved — `<event>_date` adopted.

5. **Field Ordering:**
   - **dbt Labs:** ids → strings → numerics → booleans → dates → timestamps
   - **Our Rules:** Identifiers → Attributes → Activity dates → Audit fields
   - **Impact:** Low - affects readability but not functionality
   - **Status:** Resolved — dbt Labs ordering is now the rulebook standard.

### Aligned Patterns (No Changes Needed)

- ✅ Snake_case for all names
- ✅ Underscores, not dots in model names
- ✅ No abbreviations
- ✅ Boolean prefixes (`is_`, `has_`)
- ✅ Currency decimal format
- ✅ Business terminology over source terminology
- ✅ Avoid reserved words
