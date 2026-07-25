# SKILL 05 — ASSERTION GENERATION

## Purpose
Identify required validation points across the application and business workflows. Decides WHAT should be verified — verification-template.md governs the implementation of assertion methods.

## Execution Dependencies
**Knowledge:** framework-rules.md, playwright-best-practices.md, generation-patterns.md, output-structure.md
**Templates:** verification-template.md

## Inputs
Required: Application Analysis Report, Generated Page Objects, Generated Spec Plan.
Optional: business requirements, existing assertion methods.

## Expected Output
- Assertion Plan
- Verification mapping
- Business validation points
- Assertion priority

Passed to verification-template.md for implementation.

## Responsibilities
Identify validation points, map assertions to business workflows, eliminate duplicate assertions, select the appropriate verification strategy (per verification-template.md's discovery steps), ensure business coverage.

## Workflow
Read Application Analysis → Read Spec Plan → Identify Validation Points → Classify Assertions → Remove Duplicates → Validate Business Coverage → Generate Assertion Plan → Apply verification-template.md

## Validation Point Identification
Every important business action needs an associated validation, e.g.:
- Login → Dashboard visible
- Save Record → success message, record persisted
- Delete Record → confirmation, record removed
- Search → expected results displayed

Focus on business outcome, not implementation.

## Assertion Classification
Visibility, navigation, content, state change, business rule, data validation, error validation, permission validation, success/negative validation. Generate only what the application requires.

## Assertion Priority
- **High:** authentication, data creation/modification/deletion, payments, approval flow
- **Medium:** search, filters, navigation, sorting
- **Low:** cosmetic UI, optional messages, minor layout changes

Critical business validations must never be omitted.

## Assertion Mapping
Map each assertion to its workflow (e.g. Login → Dashboard Visible → Correct URL → Correct User Session). Provide sufficient verification without becoming excessive.

## Duplicate Detection
Reuse existing verification methods (per verification-template.md's reusability guidance) instead of creating multiple assertions for the same business outcome.

## Assertion Placement
Page validation → Page Object. Component validation → component method. Business validation → verification method. Do not place assertion logic directly inside Spec files.

## Coverage Analysis
Verify every business workflow and critical operation has validation, negative scenarios include appropriate assertions, success scenarios validate expected outcomes, and no unnecessary assertions were generated. Quality over quantity.

## Success Criteria
Validation points identified · Assertion Plan generated · duplicates removed · business coverage verified · assertion mapping complete.

## Failure Handling
If planning can't continue for a workflow, document the unsupported validation, continue with the rest, and report missing business information. Never invent business rules that don't exist.

## References
**Knowledge:** framework-rules.md, generation-patterns.md, playwright-best-practices.md
**Templates:** verification-template.md
**Consumed by:** verification-template.md (implementation), Skill 07 — Framework Validation
