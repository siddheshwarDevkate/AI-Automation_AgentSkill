# SKILL 05 — ASSERTION GENERATION

## Purpose
Identify required validation points from each test case's `expectedResult` and write the corresponding verification methods directly into the Page Objects Skill 04 just generated. This Skill runs immediately after Page Object Generation and before Test Data Generation / Spec Generation, so that by the time Spec Generation runs, every verification method a test will ever call already exists.

## Execution Dependencies
**Knowledge:** skills/knowledge/framework-rules.md, skills/knowledge/playwright-best-practices.md, skills/knowledge/generation-patterns.md, skills/knowledge/output-structure.md, skills/knowledge/test-case-parsing-rules.md
**Templates:** skills/templates/verification-template.md

## Inputs
Required: Test Case Model (from Skill 01), Application Analysis Report, Generated Page Objects (from Skill 04).
Optional: existing assertion/verification methods.

## Expected Output
- Verification methods added to the appropriate Page Object files (real code, not just a plan)
- Assertion Plan / verification mapping (per Test Case ID → verification method)
- Business validation points
- Assertion priority

## You Are Extending Skill 04's Files, Not Just Planning (Critical)
This Skill writes real `verify*()` methods into the same Page Object `.ts` files Skill 04 created earlier in this generation run. This is expected and required — it is not a violation of "don't modify existing files" (that rule is about files outside the current generation task, not the artifacts this run is actively building; see AP-007 in agent/agent.md). Before adding a method, check whether Skill 04 already generated an equivalent one (e.g. a generic `verifyPageTitle()` from a recognized pattern) — reuse it instead of adding a duplicate.

## Primary Source of Truth
Every test case's `expectedResult` field is the primary — and normally the only — source for what must be asserted. Do not invent additional business rules beyond what the test case describes. Supplement with an implicit UI-state assertion (e.g. confirming a page finished navigating) only when it's necessary to make the test meaningful, and never when it would contradict or go beyond the test case's `expectedResult`.

## Responsibilities
- Parse each test case's `expectedResult` into one or more concrete validation points
- For each validation point, reuse an existing verification method on the owning Page Object, or add a new one if none matches
- Eliminate duplicate assertions across test cases
- Prioritize assertions per the test case's `priority` field when present
- Ensure every test case has corresponding assertions ready before Spec Generation runs

## Workflow
Read Test Case Model → Read Application Analysis Report → Read Generated Page Objects → For Each Test Case: Parse `expectedResult` → Identify Element(s)/Outcome(s) It Describes → Identify the Owning Page Object (per the Page Inventory) → Reuse or Add a Verification Method on That Page Object → Classify and Prioritize → Remove Duplicates → Generate Assertion Plan → Apply skills/templates/verification-template.md

## Validation Point Identification
Derive validation points directly from `expectedResult` text, e.g.:
- `expectedResult`: "User is redirected to the dashboard" → Dashboard visible, correct URL
- `expectedResult`: "Error message 'Invalid credentials' is displayed" → error message visible with expected text
- `expectedResult`: "Record appears in the table" → row exists with expected values

If an `expectedResult` is ambiguous or too vague to derive a concrete assertion, document the gap rather than guessing at business intent.

## Assertion Classification
Visibility, navigation, content, state change, business rule, data validation, error validation, permission validation, success/negative validation — classify strictly based on what the `expectedResult` actually states.

## Assertion Priority
Use each test case's own `priority` field when present. When absent, infer from the nature of the workflow:
- **High:** authentication, data creation/modification/deletion, payments, approval flow
- **Medium:** search, filters, navigation, sorting
- **Low:** cosmetic UI, optional messages, minor layout changes

Critical business validations (as flagged by test case priority) must never be omitted.

## Assertion Mapping
Map each assertion back to its source Test Case ID (e.g. TC-014 → `verifyDashboardVisible()`, `verifyCurrentUrl()`). This mapping feeds the final traceability/coverage report, and tells Skill 07 (Spec Generation) exactly which method to call for each test case.

## Duplicate Detection
Reuse existing verification methods (per skills/templates/verification-template.md's reusability guidance) instead of creating multiple assertions for the same business outcome, even across different test cases.

## Assertion Placement
Page validation → Page Object. Component validation → the method on the Page Object that owns that component (per the Page Object Count Rule in skills/skill-04-page-object-generation.md — a component never gets its own class). Business validation → verification method. Do not place assertion logic directly inside Spec files — Spec Generation (Skill 07) only calls these methods, never asserts directly.

## Coverage Analysis
Verify every test case's `expectedResult` has at least one corresponding assertion, negative scenarios include appropriate error assertions, and no assertion was generated that isn't traceable to a test case's `expectedResult`.

## Success Criteria
Every test case's `expectedResult` parsed into validation points · every needed verification method exists on the correct Page Object (reused or newly added) · Assertion Plan generated per Test Case ID · duplicates removed · priorities applied · coverage verified.

## Failure Handling
If an `expectedResult` can't be mapped to a concrete assertion, document the gap, continue with the rest, and report it for the final coverage report. Never invent a validation that isn't traceable to a test case.

## References
**Knowledge:** skills/knowledge/framework-rules.md, skills/knowledge/generation-patterns.md, skills/knowledge/playwright-best-practices.md, skills/knowledge/test-case-parsing-rules.md
**Templates:** skills/templates/verification-template.md
**Consumed by:** Skill 06 — Test Data Generation, Skill 07 — Spec Generation, Skill 08 — Framework Validation
