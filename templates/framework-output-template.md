# FRAMEWORK OUTPUT TEMPLATE

## Purpose
Defines how the final Playwright automation framework should be assembled and presented after generation completes, ensuring every generated framework follows the same structure, organization, and quality regardless of AI model. Refer to all project Standards, Knowledge files, and generation-patterns.md before producing final output.

## Used By Skills
07 Framework Validation · 08 Framework Review

## Generation Objective
Generate a complete, production-ready framework that follows the defined architecture, all framework standards, naming conventions, locator strategy, coding standards, and generation patterns, and is immediately usable. No partial implementations.

## Generation Pipeline
Application Analysis → Requirement Analysis → Page Identification → UI Component Detection → Pattern Detection → Locator Discovery → Page Object Generation → Utility Generation → Verification Generation → Test Specification Generation → Framework Validation → Return Final Output. Do not skip any stage.

## Output Organization
Return generated files grouped by folder, in order: `pages/` → `utils/` → `test-data/` → `tests/`. Maintain this order for every generation.

## Page Output
One Page Object per discovered page: private locators, constructor, navigation methods, action methods, verification methods, compound business methods. Never combine unrelated pages into one file.

## Test Output
One Spec file per business feature: positive tests, negative tests, boundary tests (when applicable), edge cases (when applicable). Tests validate business behaviour only.

## Utility Output
Generate utility classes (WaitHelper, TestDataHelper, DateHelper) only when they remove duplicated logic — avoid unnecessary helper classes.

## Code Quality Requirements
Every generated file must compile, follow strict TypeScript, follow Playwright best practices, use stable locators, avoid duplicate code, and follow project naming standards and architecture.

## Validation Before Delivery
- **Application Analysis:** every major page explored, business workflow understood, navigation analyzed
- **Page Objects:** one per page, private readonly locators, public methods only
- **Test Specifications:** business scenarios covered — positive, negative, edge cases
- **Framework:** naming conventions followed, coding standards followed, no duplicated logic, no fabricated locators, production ready

## Delivery Format
For each file, include file path, file name, and complete source code, clearly separated:
```
pages/LoginPage.ts
<Complete Source Code>
------------------------------------------------
pages/DashboardPage.ts
<Complete Source Code>
------------------------------------------------
utils/WaitHelper.ts
<Complete Source Code>
------------------------------------------------
tests/login.spec.ts
<Complete Source Code>
```
Do not truncate files or omit implementations.

## Failure Handling
If sufficient information is unavailable, do not fabricate code. Instead: explain what's missing, add `// TODO:` comments, and continue generating the remaining framework — generate as much valid code as possible without assumptions.

## Final Quality Check
✓ Application completely analyzed · ✓ correct generation patterns selected · ✓ stable locators used · ✓ Page Objects complete · ✓ verification methods generated · ✓ test specifications complete · ✓ utilities reusable · ✓ output follows templates · ✓ framework follows all standards · ✓ framework is production ready
