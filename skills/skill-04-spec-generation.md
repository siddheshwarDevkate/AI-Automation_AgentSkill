# SKILL 04 — SPEC GENERATION

## Purpose
Convert application workflows into executable business-focused test scenarios. Determines WHAT tests should be generated — spec-template.md governs HOW the file is structured.

## Execution Dependencies
**Knowledge:** framework-rules.md, generation-patterns.md, naming-conventions.md, output-structure.md
**Templates:** spec-template.md

## Inputs
Required: Application Analysis Report, Generated Page Objects.
Optional: existing Spec files, business requirements, user stories.

## Expected Output
- Feature-wise test scenarios
- Business workflow mapping
- Spec Generation Plan

Handed to spec-template.md for final code generation into Spec files.

## Responsibilities
Identify application features, discover business scenarios, organize test coverage, eliminate duplicate scenarios, map scenarios to Spec files, prepare input for spec-template.md.

## Workflow
Read Application Analysis → Identify Features → Identify Business Workflows → Discover Test Scenarios → Group Related Scenarios → Determine Spec File Boundaries → Validate Coverage → Generate Spec Plan → Apply spec-template.md

## Feature Identification
Identify independent application features (Authentication, Dashboard, User Management, Orders, Reports, Settings, Profile, Administration). Generate Spec files around business features, not individual pages.

## Business Scenario Discovery
Identify every workflow that should be automated (login, logout, create/search/update/delete record, upload/download file, approve/reject request) — only for functionality that actually exists in the application. Never invent business behaviour.

## Scenario Classification
Classify as: happy path, negative, boundary, validation, smoke, regression, end-to-end. Generate only categories relevant to the application.

## Feature Boundary Detection
Small feature → single Spec file. Large feature → multiple Spec files. Keep each Spec focused on one business feature; never combine unrelated functionality.

## Scenario Dependency Analysis
Independent scenarios → separate test cases. Dependent business workflows → single end-to-end scenario. Minimize unnecessary dependencies between tests (per spec-template.md's test-independence rule).

## Duplicate Detection
Before generating a scenario, compare against previously identified ones. If an existing scenario already validates the same business objective, extend it rather than duplicate it.

## Coverage Analysis
Before completing, verify every feature, critical workflow, and important business rule has coverage, and no major functionality is missing. Prioritize business value over test quantity.

## Success Criteria
Features identified · business workflows mapped · scenarios discovered and de-duplicated · Spec Generation Plan prepared · coverage validated.

## Failure Handling
If generation can't continue for a given feature, document the limitation, skip unsupported functionality, and continue with the rest. Never fabricate scenarios for features that don't exist.

## References
**Knowledge:** framework-rules.md, generation-patterns.md, naming-conventions.md
**Templates:** spec-template.md
**Consumed by:** spec-template.md (code generation), Skill 05 — Assertion Generation
