# Archetype Cursor Rules

Comprehensive Cursor AI rules for dbt data modeling with Snowflake, enforcing Medallion architecture and dimensional modeling best practices.

## Overview

This repository contains Cursor AI rules that enforce consistent standards across data modeling, SQL/YAML authoring, macros, AI/ML workflows, DevOps/Cloud operations, and QA automation. Use these rules as Cursor prompt files to keep every discipline aligned with the same conventions.

## Architecture

- **Logical (dbt models):** Simplified Medallion layers `raw`, `int`, `anl` defined in `data-rules/dbt-datamodelling-snowflake.mdc`.
- **Physical (Snowflake account objects):** Databases/Schemas/Warehouses follow `RAW`, `PREPARE`, `ANALYZE` conventions with RBAC and security controls described in `devops-rules/.cursor/rules/snowflake-architecture.mdc`.
- **Cloud foundations:** Cross-cloud guardrails (networking, identity, tagging) live in `devops-rules/.cursor/rules/cloud-architecture.mdc`.

## Rules Catalog

### Data Rules (`data-rules/`)

**Owner:** Data Engineering Lead

- **`dbt-datamodelling-snowflake.mdc`** — Naming conventions (layer-specific), Medallion patterns, dimensional modeling, constraints, SCD/fact guidance, optimization, and governance.
- **`dbt-sql-style.mdc`** — SQL formatting, CTE structure, query organization, join discipline, and dbt-specific dependencies (`raw_` → `int_` → `anl_`).
- **`dbt-yaml-style.mdc`** — YAML documentation/testing standards, file layout mirrors, description requirements, unit-testing examples for dbt 1.8+.
- **`dbt-macro-best-practices.mdc`** — Macro decision tree, naming, structure, documentation template, common patterns (surrogate keys, soft deletes), error handling, testing, dbt-utils usage.

### AI & ML Rules (`ai-ml-rules/`)

**Owner:** AI/ML Architect

- **`snowflake-ml-notebooks.mdc`** — Snowflake notebook setup, runtime selection, Snowpark ML workflows, model registry/deployment, MLOps, and naming.

### DevOps & Cloud Rules (`devops-rules/.cursor/rules/`)

**Owner:** Cloud Platform Lead

- **`snowflake-architecture.mdc`** — Account object management, warehouse sizing, RBAC, authentication/network policies, Terraform automation, and checklists.
- **`cloud-architecture.mdc`** — Azure/AWS architecture patterns (networking, identity, storage), cross-cloud security baseline, tagging, service decision framework.
- **`terraform-cloud-infrastructure-as-code-best-practices.mdc`** — Terraform structure, naming, remote state, tagging, security, and workflow checklists.
- **`ci-cd-pipelines.mdc`** — Azure DevOps & GitHub Actions pipeline standards, template usage, caching, security, Python/Terraform CI specifics.
- **`python-scripting.mdc`** — Guidance for automation scripts (argparse, logging, error handling, SDK auth, type hints, dependencies).

### QA & Testing Rules (`qa-test-rules/.cursor/rules/`)

**Owner:** QA Lead

- **`playwright-cursor-rules.mdc`** — Playwright best practices for TypeScript/JavaScript end-to-end tests, fixtures, locator strategy, config usage, and stability tips.

### Project Management (`cpm-rules/`)

**Owner:** PMO Lead

- **`cpm-rules.mdc`** — (Placeholder) Critical Path Method and project management standards.

### General Authoring Prompts (`general-rules/`)

- **`code-style-consistent.mdc`** — Persona enforcing code style analysis before generating new code.
- **`how-to-documentation.mdc`** — Persona for converting technical content into user-friendly "How To" guides.

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

- Timestamps: `<event>_at` (UTC). Add suffix (e.g., `_at_pt`) only when a non-UTC timezone is mandatory.
- Dates: `<event>_date`
- Booleans: `is_` or `has_` prefix
- Field ordering: identifiers/keys → strings → numerics → booleans → dates → timestamps → audit/meta fields

## Usage

1. Copy the relevant rule directories (e.g., `data-rules/`, `devops-rules/.cursor/rules/`, `ai-ml-rules/`) into your project root.
2. Cursor AI automatically loads `.mdc` files and enforces their instructions on matching glob patterns.
3. Follow the checklists/tests described in each rule file (dbt compile/test, Terraform fmt/plan, Playwright runs) to validate compliance.

## Validation Flow

Use these repeatable workflows to confirm each rule set is being honored in real projects. Adjust the commands to match your repository structure.

### Data Modeling & dbt Rules

- Install dependencies: `dbt deps`
- Compile everything to ensure configs and naming patterns resolve: `dbt compile --select path:models/raw path:models/int path:models/anl`
- Run tests (includes YAML + unit tests): `dbt test --select path:models/raw path:models/int path:models/anl`
- Exercise macros directly when updated: `dbt run-operation macro_name --args '{...}'`

### Terraform & Cloud Architecture Rules

- Format consistently: `terraform fmt -recursive`
- Static validation: `terraform validate`
- Preview infrastructure changes (per environment): `terraform plan -var-file="envs/dev.tfvars"`
- Enforce remote state/locking before `terraform apply`

### CI/CD Pipelines

- Azure DevOps templates: `az pipelines run --name <pipeline>` (or use pipeline validation command) after YAML edits
- GitHub Actions: `act --job <job-name>` (optional) or push to a test branch with required status checks

### Python Scripting Standards

- Lint/format: `ruff check .` and `black --check .`
- Execute script entry points with `--help` to verify argparse + logging wiring

### QA & Testing Rules

- Playwright suites: `npx playwright test --reporter=list`
- Optional targeted runs per project: `npx playwright test tests/<suite>.spec.ts`

## Documentation

- `docs/GAP_ANALYSIS.md` — Living record of naming conflicts, architecture decisions, and industry best-practice gaps.
- `docs/To-Do.md` — Running repository-level task list for outstanding work.

## Contributing

1. Keep guidance consistent across rule files (update GAP_ANALYSIS + README when conventions change).
2. Log new work or open questions in `docs/To-Do.md`.
3. Test changes in representative projects (dbt compile/test, Terraform plan/apply, CI pipeline dry runs, Playwright suites).

## License

[Add your license here]

## References

- [Data Modeling with Snowflake, Second Edition](https://www.snowflake.com/)
- [dbt Style Guide](https://github.com/dbt-labs/corp/blob/main/dbt_style_guide.md)
- [Kimball Dimensional Modeling](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/)
