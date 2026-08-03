# FRAMEWORK OUTPUT TEMPLATE

## Purpose
Defines how the final Playwright automation framework should be assembled and presented after generation completes, ensuring every generated framework follows the same structure, organization, and quality regardless of AI model. Refer to all project Standards, Knowledge files, and skills/knowledge/generation-patterns.md before producing final output.

## Used By Skills
08 Framework Validation · 10 Framework Review

## Generation Objective
Generate a complete, production-ready framework that follows the defined architecture, all framework standards, naming conventions, locator strategy, coding standards, and generation patterns, and is immediately usable — with every generated test traceable to a supplied Test Case. No partial implementations.

## Generation Pipeline
Test Case Analysis → Application Analysis → Page Identification → UI Component Detection → Pattern Detection → Locator Discovery → Page Object Generation (incl. Utility Generation) → Verification Generation (appends to Page Objects) → Test Data Generation (`testData.ts`) → Test Specification Generation (one per Test Case, wiring together what already exists) → Framework Validation (incl. Test Case Traceability Matrix) → Test Execution & Self-Healing (run across the browser matrix, repair framework-side failures) → Framework Review (final readiness, weighted by execution results) → Return Final Output. Do not skip any stage, and do not reorder Verification/Test Data ahead of Test Specification Generation — specs must only ever reference methods and data that already exist.

## Output Organization
Return generated files grouped by folder, in order: `pages/` (BasePage.ts first, then one file per Page Inventory entry) → `utils/` → `test-data/` → `tests/` → root-level reports (`traceability-report.md`, `execution-report.md`, `execution-state.md`, `README.md`). Maintain this order for every generation.

## Page Output
`BasePage.ts` once, then exactly one Page Object per Page Inventory entry (never one per UI pattern/component — see skills/knowledge/framework-architecture.md's Page vs Component distinction): private locators, constructor, navigation methods, action methods, verification methods, compound business methods. Never combine unrelated pages into one file, and never split one page's patterns into several files.

## Test Output
One Spec file per module (a Test Case Model grouping, independent from Page Inventory pages — see skills/knowledge/framework-architecture.md's Module vs Page distinction): positive tests, negative tests, boundary tests (when applicable), edge cases (when applicable). Tests validate business behaviour only, importing Page Object methods and `testData.ts` constants — never inline literals or direct locator assertions.

## Utility Output
Generate utility classes (WaitHelper, TestDataHelper, DateHelper) only when they remove duplicated logic — avoid unnecessary helper classes.

## Code Quality Requirements
Every generated file must compile, follow strict TypeScript, follow Playwright best practices, use stable locators, avoid duplicate code, and follow project naming standards and architecture.

## Validation Before Delivery
- **Test Case Analysis:** every row parsed, required fields validated, skipped rows documented
- **Application Analysis:** every test-case-referenced page explored, business workflow understood, navigation analyzed
- **Page Objects:** one per page, private readonly locators, public methods only
- **Test Specifications:** one test per Test Case, every test tagged with its Test Case ID, no scenario outside the Test Case Model
- **Framework:** naming conventions followed, coding standards followed, no duplicated logic, no fabricated locators, Test Case Traceability Matrix complete, production ready
- **Execution:** suite run across the full browser matrix, every failure diagnosed before repair, repairs scoped correctly and within the retry budget, no assertion/data weakened to force a pass, Execution Report complete

## Delivery Format
For each file, include file path, file name, and complete source code (or complete report content for the `.md` reports), clearly separated:
```
pages/BasePage.ts
<Complete Source Code>
------------------------------------------------
pages/LoginPage.ts
<Complete Source Code>
------------------------------------------------
pages/DashboardPage.ts
<Complete Source Code>
------------------------------------------------
utils/WaitHelper.ts
<Complete Source Code>
------------------------------------------------
test-data/testData.ts
<Complete Source Code>
------------------------------------------------
tests/login.spec.ts
<Complete Source Code>
------------------------------------------------
traceability-report.md
<Complete Report Content>
------------------------------------------------
execution-report.md
<Complete Report Content>
------------------------------------------------
README.md
<Complete Report Content>
```
Do not truncate files or omit implementations.

## Failure Handling
If sufficient information is unavailable, do not fabricate code. Instead: explain what's missing, add `// TODO:` comments, and continue generating the remaining framework — generate as much valid code as possible without assumptions.

## Final Quality Check
✓ Test Case file parsed completely · ✓ application analyzed within Test Case scope · ✓ correct generation patterns selected · ✓ stable locators used · ✓ `BasePage.ts` present · ✓ Page Object count matches the Page Inventory exactly · ✓ verification methods generated · ✓ `testData.ts` generated · ✓ one test specification per Test Case, tagged with its ID · ✓ utilities reusable · ✓ output follows templates · ✓ framework follows all standards · ✓ suite executed and verified passing across the browser matrix · ✓ all reports (traceability, execution, README) present · ✓ framework is production ready
