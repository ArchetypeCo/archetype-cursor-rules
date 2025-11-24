# Repository To-Do

Running checklist of follow-up items across all rule sets. Update this file whenever new work is identified or completed.

## Open Tasks

- [ ] Decide on authoritative naming conventions (singular vs plural, `_key` vs `_id`, `_datetime` vs `_at`) and propagate the decision across every rules document.
- [ ] Bring `README.md` and any future onboarding docs in sync whenever directories or rule files move (e.g., `data-rules/`, `ai-ml-rules/`, `devops-rules/.cursor/rules`).
- [ ] Add front matter + structured guidance to `qa-test-rules/.cursor/rules/playwright-cursor-rules.mdc` so Cursor reliably applies those standards.
- [ ] Expand the QA/testing coverage beyond Playwright to include data pipeline validation and cloud infrastructure testing; update the README section currently linking only to Playwright rules.
- [ ] Flesh out guidance for `cpm-rules` (currently empty) or clearly document its scope.
- [ ] Identify owners for each rule set (data, AI/ML, DevOps, QA) and record them in the docs for accountability.

## Completed Tasks

- [x] Create this running To-Do list inside `docs/`.
- [x] Document a repeatable validation/testing approach (see README “Validation Flow” section for dbt, Terraform, CI/CD, Python scripts, and Playwright suites).

