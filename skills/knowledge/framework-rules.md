# FRAMEWORK RULES

**Version:** 1.0 · **Category:** Global Standard · **Applies To:** Agent, Skills, Templates

## Purpose
The mandatory engineering rules for generating the Playwright automation framework. Every Agent, Skill, and Template MUST follow these — they have the highest priority after explicit user instructions.

## Used By Skills
All 10 skills (01–10)

## Rule Priority
1. User Instructions
2. Framework Rules (this document)
3. Project Standards
4. Playwright Best Practices
5. General Software Engineering Practices

Higher priority always overrides lower.

## Critical Rules — THE NON-NEGOTIABLES
**Violating any rule in this section is a framework generation failure. Stop and correct it before continuing — never work around it, never proceed hoping a later Skill will catch it.**

1. **Never fabricate information.** Generate only from information obtained from the supplied Test Cases and the live application. If undeterminable, STOP and report — never guess.
2. **The Test Case file decides WHAT gets automated; the running application decides HOW.** Use the configured browser automation tool (Playwright MCP in the current implementation) to confirm application behaviour. Never rely on assumptions for either.
3. **Understand before generating.** Framework generation begins only after the Test Case file has been parsed and application analysis has succeeded.
4. **Every generated Spec test must trace back to exactly one supplied Test Case ID.** Never generate a scenario that isn't traceable to a parsed test case.
5. **Generate only production-ready code.** Incomplete implementations are prohibited.
6. **Every generated file must pass validation before delivery.**
7. **Code is not correct until it has actually been executed and passed** across the configured browser matrix (per skills/skill-09-test-execution-self-healing.md). Never call a framework production ready on static validation alone.
8. **Never repair a failing test by weakening it.** No loosened assertions, no altered test data to hide a failure, no suppressed exceptions. A genuine application defect is reported, never masked.
9. **One Page Object per real application page — never one per UI pattern, component, or feature.** A data table, dropdown, filter panel, or free-text search box found on a page is a method on that page's Page Object, not a class of its own. Page Object count must equal Skill 02's Page Inventory count exactly. See POM-05.

## Page Object Model Rules
These are access-enforcement rules (what code is and isn't allowed to touch). What a Page Object structurally contains lives in skills/knowledge/framework-architecture.md's Page Object Responsibilities.
- **POM-01:** Every locator must remain `private readonly`. Never expose locators outside the Page Object.
- **POM-02:** Specs interact only through public methods. Correct: `await loginPage.performLogin(username, password);` — Incorrect: `await loginPage.usernameInputLocator.fill(username);`
- **POM-03:** Never expose Page properties. Correct: `await loginPage.verifyCurrentUrl(...)` — Incorrect: `await expect(loginPage.page).toHaveURL(...)`
- **POM-04:** Business logic belongs inside Page Objects; Spec files stay clean and readable.
- **POM-05 (Critical):** The number of generated Page Objects must equal the number of entries in Skill 02's Page Inventory — never one per detected UI pattern, component, or feature (e.g. a data table, column-selection dropdown, or free-text search box). Full rule, decision test, and forbidden example live in skills/knowledge/framework-architecture.md's Page vs Component distinction and skills/skill-04-page-object-generation.md's Page Object Count Rule.

## Locator Rules
Locator selection, priority order, validation, and special-case handling are owned entirely by skills/knowledge/locator-strategy.md — follow it exactly. Do not choose or restate a different priority elsewhere in the framework.

## Playwright API Rules
Waiting strategy, assertion style, action/verification method shape, and test structure are owned entirely by skills/knowledge/playwright-best-practices.md — follow it exactly.

## TypeScript Rules
Typing, imports, class/method design, variable naming, comments, and async patterns are owned entirely by skills/knowledge/typescript-coding-standards.md — follow it exactly.

## Spec File Rules
Spec files must comply with **POM-02** (interact only through public Page Object methods — never access a locator or assert directly on one), skills/knowledge/playwright-best-practices.md's Test Structure section (Arrange → Act → Assert, test independence, grouping), skills/knowledge/naming-conventions.md's Test Naming rules (NC-301–NC-303, including the mandatory Test Case ID tag), and the Reusable Logic Rules below. Business logic and assertions never live inside a Spec file.

## Reusable Logic Rules
Where common/reusable logic belongs is owned by skills/knowledge/framework-architecture.md's Reusable Logic Placement section — follow it exactly. This document's enforcement:
- **RL-01:** A Spec file never defines a reusable/common function inline — no data generators, custom wait conditions, retry wrappers, or shared setup/teardown routines written directly in a `*.spec.ts` file.
- **RL-02:** Generic, test-lifecycle-independent helpers (formatting, random data, string/number utilities) go in `utils/`. Playwright `test.extend()` setup/teardown shared by two or more spec files goes in `fixtures/`. Shared `beforeEach()`/`afterEach()` routines reused by two or more spec files go in `hooks/`.
- **RL-03:** `fixtures/` and `hooks/` are generated only once there is actual reusable logic to place in them — never scaffolded empty.
- **RL-04:** A helper used by exactly one spec file, with no expectation of reuse, may stay local to that file — this rule targets duplicated/shared logic, not every local function.

## Execution & Self-Healing Rules
- **EX-01:** Run the full generated suite across the resolved browser matrix (user-specified → project-configured → Chromium/Firefox/WebKit default; see skills/skill-09-test-execution-self-healing.md's Browser Matrix) before any readiness decision is made.
- **EX-02:** Diagnose the root cause of every failure before attempting a repair — never patch blindly.
- **EX-03:** Repair only framework-side defects (locator drift, Page Object logic, framework-side assertion mismatch, test data conflict, flakiness) by re-invoking the correct upstream Skill for the specific affected artifact.
- **EX-04:** Never repair by weakening an assertion, altering test data to hide a failure, or suppressing an exception — see Critical Rule 8.
- **EX-05:** Cap repair attempts at 3 per test case per browser. Beyond that, mark the case "Blocked — Manual Review Required" and continue.
- **EX-06:** A genuine application defect (actual behaviour contradicts the source test case's `expectedResult`) is documented, never silently forced to pass.
- **EX-07:** Cap execution concurrency at 2 workers (2 browser instances running at once), independent of how many browsers are in the resolved matrix — see skills/skill-09-test-execution-self-healing.md's Execution Concurrency rule. Never fan a run or a repair out to more concurrent browser instances than this to go faster.

## Output Rules
Which files may or may not be generated (including the `playwright.config.ts` restriction and Skill 09's narrow exception) is owned entirely by skills/knowledge/output-structure.md's Output Restrictions section — follow it exactly. Do not overwrite existing files unless instructed. Do not modify unrelated code.

## Code Quality Rules
Method size, duplication, and reuse standards are owned by skills/knowledge/typescript-coding-standards.md (Methods section) and skills/knowledge/framework-architecture.md's Code Reuse principle. This document's only addition: Single Responsibility Principle applies at every level — files, classes, and methods alike.

## Failure Conditions
Stop generation if: the Test Case file is missing/empty/unparsable, application cannot be analyzed, authentication fails, a locator cannot be determined, framework validation fails, the generated suite cannot be executed, or required information is unavailable. Never fabricate implementation, or a scenario outside the Test Case file, to bypass a failure.

## Self Review Checklist
Before returning generated code, confirm every applicable per-file Final Checklist has passed: skills/knowledge/locator-strategy.md, skills/knowledge/playwright-best-practices.md, skills/knowledge/typescript-coding-standards.md, skills/knowledge/naming-conventions.md, skills/knowledge/output-structure.md, skills/knowledge/generation-patterns.md. In addition, specific to this document: no fabricated information anywhere in the framework · every Spec test traces to exactly one supplied Test Case ID · suite actually executed across the browser matrix · no repair weakened correctness to force a pass.
