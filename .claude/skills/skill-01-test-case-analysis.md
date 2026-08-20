---
name: skill-01-test-case-analysis
description: Parse the supplied CSV/Excel Test Case file into the canonical Test Case Model - the mandatory source of scenario truth that drives every downstream Skill.
---

# SKILL 01 — TEST CASE ANALYSIS

## Purpose
Read the company-supplied Test Case file (CSV or Excel) and convert it into a canonical Test Case Model that drives every subsequent Skill. This is the mandatory entry point of the pipeline — the framework is no longer generated from free-form exploration alone; every generated artifact must trace back to a supplied test case.

## Execution Dependencies
Load the Knowledge files listed for Skill 01 in .claude/agent/agent.md's **Skill Dependency Matrix** — that table is the single source of truth and is deliberately not restated here. This Skill uses no Template; it produces a Test Case Model (data), not generated code.

## Inputs
Required: Test Case file (`.csv` or `.xlsx`), supplied by the user, typically under `.claude/input/`.
Optional: existing framework, prior Test Case Model (for incremental regeneration).

## Expected Output — Test Case Model
- Parsed test case records: `id`, `title`, `module`, `preconditions`, `steps`, `testData`, `expectedResult`, `priority`, `type`
- Skipped Test Cases (with reason)
- Unmapped Columns (if any)
- Module/feature grouping derived from the test cases
- Parsing assumptions

No implementation code, locators, or application exploration happens at this stage.

## Responsibilities
- Locate and read the Test Case file
- Detect its format (CSV vs Excel) and parse accordingly
- Normalize headers to canonical fields (per .claude/skills/knowledge/test-case-parsing-rules.md)
- Validate that every row has the minimum required fields
- Group test cases by module/feature
- Produce the Test Case Model

## Workflow
Locate Test Case File → Detect Format → Parse Rows → Normalize Headers to Canonical Fields → Validate Required Fields → Split Steps into Ordered Actions → Group by Module → Document Skipped Rows / Unmapped Columns → Generate Test Case Model

## Required-File Gate
This Skill cannot produce a usable Test Case Model from a missing or empty Test Case file. If the file is missing, unreadable, or contains zero valid rows after parsing, stop and request a populated file from the user. This is a Critical Failure for the Agent — never continue generation with zero test cases.

## Grouping Strategy
Group parsed test cases by their `module` field. If `module` is absent, infer a reasonable grouping from `id`/`title` prefixes and document the inference — this grouping becomes the primary input for Spec file boundaries in Skill 07. **Note:** `module` is independent from the Page Inventory that Skill 02 builds — one module can span multiple real pages and one page can serve multiple modules; never assume they're the same grouping (see .claude/skills/knowledge/framework-architecture.md's Module vs Page distinction).

## Incremental Mode — Diffing Against a Prior Test Case Model
If the user supplies a prior Test Case Model, a prior Test Case file, or a prior run's `execution-state.md`/`traceability-report.md` alongside an existing framework, compare this run's parsed records against it by `id`, and label each as **New**, **Changed** (any field differs), **Removed** (existed before, absent now), or **Unchanged**. Include this classification in the Test Case Model's output so downstream Skills can scope their work to New/Changed records only, per .claude/agent/agent.md's Incremental Generation Mode. If no prior model/report is supplied, skip this step entirely — treat every record as New.

## Preparing for Downstream Skills
The Test Case Model is consumed directly by:
- **Skill 02 (Application Analysis)** — scopes exploration to only the pages/workflows referenced by the test cases, instead of open-ended free exploration.
- **Skill 05 (Assertion Generation)** — derives verification methods primarily from each test case's `expectedResult`.
- **Skill 06 (Test Data Generation)** — derives test data primarily from each test case's `testData`.
- **Skill 07 (Spec Generation)** — generates exactly one spec per test case, tagged with its `id` for traceability.

## Success Criteria
Test Case file located and parsed · headers normalized · every row validated · steps ordered correctly · module grouping produced · Test Case Model generated · skipped rows and unmapped columns documented.

## Failure Handling
Stop if the Test Case file cannot be found, cannot be parsed, or contains no valid rows after parsing. Report the issue to the Agent — never continue generation from an empty or fabricated Test Case Model.

## Consumed By
Skill 02 — Application Analysis · Skill 05 — Assertion Generation · Skill 06 — Test Data Generation · Skill 07 — Spec Generation
