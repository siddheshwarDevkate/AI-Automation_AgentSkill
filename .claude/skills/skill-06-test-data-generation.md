---
name: skill-06-test-data-generation.md
description: Generate test-data/testData.ts plus the Test Data Lifecycle Plan, emitting uniqueness factories and teardown requirements for every state-creating or state-mutating test case.
---

# SKILL 06 — TEST DATA GENERATION

## Purpose
Determine the test data required to execute the generated scenarios, sourced primarily from each test case's `testData` field, and produce the actual `test-data/testData.ts` file. This Skill runs after Assertion Generation and before Spec Generation, so that by the time Spec Generation runs, every data constant a test will import already exists.

## Execution Dependencies
Load the Knowledge files listed for Skill 06 in .claude/agent/agent.md's **Skill Dependency Matrix** — that table is the single source of truth and is deliberately not restated here. This Skill uses no Template; output format is governed directly by .claude/skills/knowledge/output-structure.md's Test Data Output section.

## Inputs
Required: Test Case Model (from Skill 01).
Optional: existing test data, configuration files, environment information, Application Analysis Report (for environment context).

## Expected Output
- `test-data/testData.ts` — the real, generated file, including any uniqueness factory functions
- Test data categories
- Test data mapping (per Test Case ID → constant name)
- **Test Data Lifecycle Plan** — per Test Case ID: state effect (read-only / state-creating / state-mutating), which values are uniquified and how, which cases require teardown, and any case where uniqueness could not be guaranteed
- Shared test data strategy

This Skill does not depend on Spec Generation or Assertion Generation's outputs — every value it needs comes directly from the Test Case Model's own `testData` field, parsed back in Skill 01. Producing the file now (before Spec Generation) is what lets Skill 07 import real constants instead of embedding literals.

## Primary Source of Truth
Each test case's `testData` field is the primary source for the values a scenario needs. Use it as supplied — do not alter values it already provides. Only infer additional data when `testData` is empty or incomplete for a step that clearly requires input, and clearly flag any inferred (as opposed to supplied) value in the output.

**The one sanctioned modification** is the per-run uniqueness suffix described in .claude/skills/knowledge/test-data-lifecycle.md's Unique Data Generation section, applied only to uniqueness-constrained values on state-creating test cases. The supplied value remains the base; nothing else about it changes. Every other value is written exactly as supplied.

## Test Data Lifecycle (Critical)
Values alone are not enough — data that works on the first run must still work on the tenth, and under parallel workers. Apply .claude/skills/knowledge/test-data-lifecycle.md in full:
1. **Classify every test case by state effect** — read-only, state-creating, or state-mutating.
2. **For state-creating cases, emit a factory function** (e.g. `buildUniqueCustomerName()`) for each uniqueness-constrained value instead of a fixed literal — never for credentials, values an `expectedResult` asserts exactly, or negative cases that depend on the supplied value.
3. **Record which cases require teardown**, so Skill 07 can wire it into the spec's `afterEach()`/fixture.

This is designed in here, at generation time. A collision discovered later by Skill 09 is a lifecycle gap that routes back to this Skill — not something to patch by hand-editing a literal.

## Responsibilities
- Parse `testData` from every test case
- Categorize it, map it to its source Test Case ID, eliminate duplicate data
- Promote reusable datasets shared across multiple test cases into named constants
- Flag sensitive information
- Document any data gaps left after parsing `testData`
- Generate `test-data/testData.ts` per .claude/skills/knowledge/output-structure.md's Test Data Output format

## Workflow
Read Test Case Model → Extract `testData` per Test Case → Classify Test Data → Classify Each Test Case by State Effect → Identify Uniqueness-Constrained Values → Identify Reusable Datasets → Remove Duplicate Data → Map Data to Test Case IDs → Document Gaps → Generate `test-data/testData.ts` (constants + uniqueness factories) → Produce the Test Data Lifecycle Plan

## Test Data Identification
For each test case, extract the values referenced by its `testData` field and cross-reference them against its `steps` to confirm every input the steps require has a corresponding value. If a step needs an input value with none supplied in `testData`, document it as a gap rather than fabricating one.

## Test Data Classification
Valid, invalid, boundary, empty, duplicate, special-character, business-rule, environment-specific — classify based on how the test case's own `type`/`expectedResult` characterizes the scenario (e.g. a test case whose `expectedResult` describes an error is feeding invalid/boundary data).

## Data Reusability
Identify test data shared verbatim across multiple test cases (e.g. the same valid login credentials reused by several scenarios) and promote it to a single shared constant instead of duplicating it per test case.

## Data Mapping
Map each dataset to its source Test Case ID, e.g.:
- TC-001 (Valid login) → `validUsername`, `validPassword`
- TC-002 (Invalid password) → `validUsername`, `invalidPassword`

This mapping tells Skill 07 (Spec Generation) exactly which constant to import for each test case.

## Sensitive Data Handling
Flag values that should never be hardcoded: passwords, API keys, tokens, secrets, environment URLs — even when they come directly from the supplied `testData` column. Recommend external configuration or secure storage rather than writing them verbatim into `testData.ts`.

## Data Quality
Verify every test case's data needs are satisfied by its `testData` (or a documented gap exists), no duplicate or unnecessary datasets exist, sensitive values are identified, and reused data stays consistent across test cases.

## Success Criteria
`test-data/testData.ts` generated · every constant mapped to its source Test Case ID(s) · every test case classified by state effect · uniqueness-constrained values on state-creating cases emitted as factory functions, never fixed literals · teardown requirements recorded for Skill 07 · Test Data Lifecycle Plan produced · data gaps documented, not fabricated · duplicates removed · shared datasets identified · sensitive data documented.

## Failure Handling
If a test case's `testData` is missing or insufficient for its steps, document the gap, continue with remaining test cases, and report it for the final coverage report. Never invent business data that wasn't supplied and can't be unambiguously inferred from the application.

## Consumed By
Skill 07 — Spec Generation · Skill 08 — Framework Validation · Skill 09 — Test Execution & Self-Healing (data-lifecycle repairs)
