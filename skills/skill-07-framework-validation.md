# SKILL 07 — FRAMEWORK VALIDATION

## Purpose
Validate the generated framework against project standards, coding guidelines, and architecture before delivery. Identifies issues — does NOT modify the generated framework.

## Execution Dependencies
**Knowledge:** framework-architecture.md, framework-rules.md, locator-strategy.md, naming-conventions.md, generation-patterns.md, output-structure.md, playwright-best-practices.md, typescript-coding-standards.md
**Templates:** framework-output-template.md

## Inputs
Required: Generated Page Objects, Generated Spec Files, Assertion Plan, Test Data Plan, project standards.
Optional: existing framework, previous validation reports.

## Expected Output
- Validation Report
- Validation summary
- List of issues
- Compliance status
- Quality score

## Responsibilities
Validate framework completeness, architecture, and generated artifacts; detect duplicate implementations and missing components; measure standards compliance — checked against the delivery checklist in framework-output-template.md.

## Workflow
Read Generated Framework → Validate Architecture → Validate Generated Files → Validate Standards Compliance → Detect Issues → Generate Validation Report

## Architecture Validation
Verify correct folder structure (per framework-architecture.md), expected files generated, proper separation of responsibilities, correct dependency flow.

## Page Object Validation
Verify expected Page Objects exist, no duplicates, required business/verification methods present, consistent implementation.

## Spec Validation
Verify required Spec files generated, features correctly mapped, no duplicate scenarios, Page Objects referenced correctly, business workflows covered.

## Assertion Validation
Verify critical workflows contain assertions, duplicates avoided, verification coverage sufficient, business outcomes validated.

## Test Data Validation
Verify every scenario has test data, shared data is reused, no duplicate datasets, sensitive data handled appropriately.

## Standards Compliance
Validate against: framework-rules.md, playwright-best-practices.md, typescript-coding-standards.md, naming-conventions.md, generation-patterns.md. Identify every violation.

## Duplicate Detection
Detect duplicate methods, Page Objects, scenarios, assertions, or test data. Recommend reuse.

## Completeness Validation
Ensure every discovered feature is covered, every required file generated, no missing component or incomplete implementation.

## Success Criteria
Framework inspected · all artifacts validated · compliance verified · issues documented · Validation Report generated.

## Failure Handling
If validation can't be completed, document the limitation, continue validating remaining artifacts, and report incomplete validation. Never assume compliance without verification.

## References
**Knowledge:** framework-rules.md, playwright-best-practices.md, typescript-coding-standards.md, naming-conventions.md, generation-patterns.md
**Templates:** framework-output-template.md
**Consumed by:** Skill 08 — Framework Review
