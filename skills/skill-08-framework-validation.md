# SKILL 08 — FRAMEWORK VALIDATION

## Purpose
Validate the generated framework against project standards, coding guidelines, architecture, and — now mandatory — Test Case traceability before delivery. Identifies issues — does NOT modify the generated framework.

## Execution Dependencies
**Knowledge:** skills/knowledge/framework-architecture.md, skills/knowledge/framework-rules.md, skills/knowledge/locator-strategy.md, skills/knowledge/naming-conventions.md, skills/knowledge/generation-patterns.md, skills/knowledge/output-structure.md, skills/knowledge/playwright-best-practices.md, skills/knowledge/typescript-coding-standards.md, skills/knowledge/test-case-parsing-rules.md
**Templates:** skills/templates/framework-output-template.md

## Inputs
Required: Test Case Model, Application Analysis Report (including its Page Inventory), Generated Page Objects (with verification methods from Skill 05), Generated Spec Files, `test-data/testData.ts` (from Skill 06), project standards.
Optional: existing framework, previous validation reports.

## Expected Output
- Validation Report
- Validation summary
- Test Case Traceability Matrix (Test Case ID → Spec test → assertions → test data)
- List of issues
- Compliance status
- Quality score

## Responsibilities
Validate framework completeness, architecture, and generated artifacts; detect duplicate implementations and missing components; verify every generated artifact traces back to a supplied test case; measure standards compliance — checked against the delivery checklist in skills/templates/framework-output-template.md.

## Workflow
Read Test Case Model → Read Generated Framework → Validate Architecture → Validate Test Case Traceability → Validate Generated Files → Validate Standards Compliance → Detect Issues → Generate Validation Report

## Architecture Validation
Verify correct folder structure (per skills/knowledge/framework-architecture.md), expected files generated, proper separation of responsibilities, correct dependency flow.

## Test Case Traceability Validation
- Every parsed test case is either mapped to exactly one generated spec test (tagged with its `id`), or explicitly recorded as "Not Automated" with a reason.
- No generated spec test exists that isn't traceable back to a supplied test case `id` (no fabricated scenarios).
- Every test's assertions trace back to its source test case's `expectedResult`.
- Every test's data trace back to its source test case's `testData` (or a documented gap).

Build the Test Case Traceability Matrix from these checks — this is the primary completeness signal for this framework, replacing free-form feature coverage.

## Page Object Validation
Verify expected Page Objects exist, no duplicates, required business/verification methods present, consistent implementation.

**Page Object Count Check (Critical):** Count the generated Page Objects and compare against Skill 02's Page Inventory. They must match exactly. If there are more Page Objects than Page Inventory entries, at least one is a UI pattern/component/feature masquerading as a page (e.g. `DataTablePage.ts`, `ColumnSelectionPage.ts`, `FreeTextPage.ts` where only one real page exists) — this is a violation of skills/knowledge/framework-rules.md's Critical Rule 9 / POM-05, not a stylistic nitpick. Flag it, identify which real page each extra class actually belongs to, and require its methods be merged into that page's correct Page Object before the framework can pass validation.

## Spec Validation
Verify required Spec files generated, one test per test case (per the Strict Traceability Rule in Skill 07), no duplicate scenarios beyond what duplicate test cases explicitly require, Page Objects referenced correctly, no inline assertions or hardcoded data literals (everything imported from Page Object methods and `testData.ts`).

**Inline Reusable Logic Check:** Scan each spec file for a reusable/common function defined inline (a data generator, custom wait condition, retry wrapper, or setup/teardown routine used by more than one test or spec) — this violates skills/knowledge/framework-rules.md's RL-01. Flag it and require it be moved to `utils/`, `fixtures/`, or `hooks/` per skills/knowledge/framework-architecture.md's Reusable Logic Placement section before the framework can pass validation.

## Assertion Validation
Verify every test case's `expectedResult` has a corresponding assertion, duplicates avoided, business outcomes validated.

## Test Data Validation
Verify every test case's `testData` needs are satisfied or documented as a gap, shared data is reused, no duplicate datasets, sensitive data handled appropriately.

## Standards Compliance
Validate against: skills/knowledge/framework-rules.md, skills/knowledge/playwright-best-practices.md, skills/knowledge/typescript-coding-standards.md, skills/knowledge/naming-conventions.md, skills/knowledge/generation-patterns.md, skills/knowledge/test-case-parsing-rules.md. Identify every violation.

## Duplicate Detection
Detect duplicate methods, Page Objects, scenarios, assertions, or test data. Recommend reuse.

## Completeness Validation
Ensure every parsed test case is accounted for (automated or explicitly documented as not automated), every required file generated, no missing component or incomplete implementation.

## Success Criteria
Framework inspected · all artifacts validated · Page Object count matches the Page Inventory exactly · Test Case Traceability Matrix complete · compliance verified · issues documented · Validation Report generated.

## Failure Handling
If validation can't be completed, document the limitation, continue validating remaining artifacts, and report incomplete validation. Never assume compliance or traceability without verification.

## References
**Knowledge:** skills/knowledge/framework-rules.md, skills/knowledge/playwright-best-practices.md, skills/knowledge/typescript-coding-standards.md, skills/knowledge/naming-conventions.md, skills/knowledge/generation-patterns.md, skills/knowledge/test-case-parsing-rules.md
**Templates:** skills/templates/framework-output-template.md
**Consumed by:** Skill 09 — Test Execution & Self-Healing
