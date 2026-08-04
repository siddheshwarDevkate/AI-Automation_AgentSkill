# FRAMEWORK RULES

**Version:** 1.0 · **Category:** Global Standard · **Applies To:** Agent, Skills, Templates

## Purpose
The mandatory engineering rules for generating the Playwright automation framework. Every Agent, Skill, and Template MUST follow these — they have the highest priority after explicit user instructions.

## Dependency Matrix
Which Skills load this file is declared in agent/agent.md's Skill Dependency Matrix — that table is the single source of truth. In practice this file appears in every Skill's row.

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

## Zero-Tolerance Test Case Compliance
These five prohibitions carry the same severity as the Critical Rules above and govern how the supplied Test Case file specifically is honored throughout generation:
- **Traceability:** Never generate a test that lacks a corresponding Test Case ID from the input file. See Critical Rule 4, skills/knowledge/naming-conventions.md's NC-303, and skills/skill-07-spec-generation.md's Strict Traceability Rule.
- **No UI Discovery:** Never derive or add a test scenario by analyzing the live application UI. Skill 02's application analysis exists only to confirm HOW an already-supplied test case behaves (Critical Rule 2) — never to discover WHAT to test. See skills/knowledge/generation-patterns.md's Pattern Detection Strategy Step 1 and skills/skill-07-spec-generation.md's Strict Traceability Rule.
- **No Hallucination:** Never invent a test step, test case, or expected result not explicitly present in the supplied Test Case file. See Critical Rule 1.
- **No Scope Creep:** Never add an extra scenario — exploratory testing, edge cases, "extra coverage" — that the Test Case file didn't specify, even if application analysis surfaces interesting behaviour along the way. See skills/skill-07-spec-generation.md's Strict Traceability Rule and Coverage Analysis.
- **No Omissions:** Never skip a supplied test case regardless of its perceived complexity or importance. A test case may only be excluded from the Spec files when it is genuinely unautomatable (missing page, element, method, or data) and is explicitly documented as "Not Automated" with a reason — skipping without that documentation, or skipping because a case merely looks hard, is forbidden. See skills/skill-07-spec-generation.md's Unautomatable Test Cases section.
- **Input Integrity:** Generate scripts only for the test cases present in the supplied input file. Never supplement the suite with scenarios or ideas from outside that file. See AP-009 in agent/agent.md.

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

**Test Granularity:** Each generated `test()` executes the complete ordered sequence of its source test case's steps as one method — never split a single test case's steps into separate `test()`s per atomic step. See skills/skill-07-spec-generation.md's Strict Traceability Rule.

## Reusable Logic Rules
Where common/reusable logic belongs is owned by skills/knowledge/framework-architecture.md's Reusable Logic Placement section — follow it exactly. This document's enforcement:
- **RL-01:** A Spec file never defines a reusable/common function inline — no data generators, custom wait conditions, retry wrappers, or shared setup/teardown routines written directly in a `*.spec.ts` file.
- **RL-02:** Generic, test-lifecycle-independent helpers (formatting, random data, string/number utilities) go in `utils/`. Playwright `test.extend()` setup/teardown shared by two or more spec files goes in `fixtures/`. Shared `beforeEach()`/`afterEach()` routines reused by two or more spec files go in `hooks/`.
- **RL-03:** `fixtures/` and `hooks/` are generated only once there is actual reusable logic to place in them — never scaffolded empty.
- **RL-04:** A helper used by exactly one spec file, with no expectation of reuse, may stay local to that file — this rule targets duplicated/shared logic, not every local function.

## Test Data Lifecycle Rules
How generated data survives repeated runs and parallel workers is owned by skills/knowledge/test-data-lifecycle.md — follow it exactly. This document's enforcement:
- **TD-01:** Every test case is classified by state effect (read-only / state-creating / state-mutating) during Test Data Generation.
- **TD-02:** A uniqueness-constrained value on a state-creating test case is emitted as a factory function, never a fixed literal. The supplied value stays the base — the per-run suffix is the only permitted modification, and it never applies to credentials, values asserted exactly by an `expectedResult`, or negative cases that depend on the supplied value.
- **TD-03:** Every state-creating/state-mutating test has teardown. Teardown contains no assertions, swallows no failures, and never deletes pre-existing application data.
- **TD-04:** No test consumes a record another test created, and no test depends on another test's ordering or cleanup — parallel workers make both unsafe.
- **TD-05:** A data collision found at execution time is repaired by adding the missing factory or teardown (routing back to Skills 06/07), never by hand-editing a literal so one run goes green. Editing a value to mask a failure remains forbidden under Critical Rule 8 / EX-04.

## Execution & Self-Healing Rules
- **EX-01:** Run the full generated suite across the resolved browser matrix (user-specified → project-configured → Chromium/Firefox/WebKit default; see skills/skill-09-test-execution-self-healing.md's Browser Matrix) before any readiness decision is made.
- **EX-02:** Diagnose the root cause of every failure before attempting a repair — never patch blindly.
- **EX-03:** Repair only framework-side defects (locator drift, Page Object logic, framework-side assertion mismatch, test data conflict, flakiness) by re-invoking the correct upstream Skill for the specific affected artifact.
- **EX-04:** Never repair by weakening an assertion, altering test data to hide a failure, or suppressing an exception — see Critical Rule 8.
- **EX-05:** Cap repair attempts at 3 per test case per browser. Beyond that, mark the case "Blocked — Manual Review Required" and continue.
- **EX-06:** A genuine application defect (actual behaviour contradicts the source test case's `expectedResult`) is documented, never silently forced to pass.
- **EX-07:** Cap execution concurrency at 2 workers (2 browser instances running at once), independent of how many browsers are in the resolved matrix — see skills/skill-09-test-execution-self-healing.md's Execution Concurrency rule. Never fan a run or a repair out to more concurrent browser instances than this to go faster.
- **EX-08:** If any repair modified generated code, re-invoke Skill 08 before the Execution Report is produced — see skills/skill-09-test-execution-self-healing.md's Re-Validation Gate. A pre-repair Validation Report describes code that no longer exists and must never reach Skill 10. A repair that fixes a test failure while breaking a framework rule is not a successful repair.

## Output Rules
Which files may or may not be generated (including the `playwright.config.ts` restriction and Skill 09's narrow exception) is owned entirely by skills/knowledge/output-structure.md's Output Restrictions section — follow it exactly. Do not overwrite existing files unless instructed. Do not modify unrelated code.

## Code Quality Rules
Method size, duplication, and reuse standards are owned by skills/knowledge/typescript-coding-standards.md (Methods section) and skills/knowledge/framework-architecture.md's Code Reuse principle. This document's only addition: Single Responsibility Principle applies at every level — files, classes, and methods alike.

## Failure Conditions
Stop generation if: the Test Case file is missing/empty/unparsable, application cannot be analyzed, authentication fails, a locator cannot be determined, framework validation fails, the generated suite cannot be executed, or required information is unavailable. Never fabricate implementation, or a scenario outside the Test Case file, to bypass a failure.

## Self Review Checklist
Before returning generated code, confirm every applicable per-file Final Checklist has passed: skills/knowledge/locator-strategy.md, skills/knowledge/playwright-best-practices.md, skills/knowledge/typescript-coding-standards.md, skills/knowledge/naming-conventions.md, skills/knowledge/output-structure.md, skills/knowledge/generation-patterns.md. In addition, specific to this document: no fabricated information anywhere in the framework · every Spec test traces to exactly one supplied Test Case ID · suite actually executed across the browser matrix · no repair weakened correctness to force a pass.
