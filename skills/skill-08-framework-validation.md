# SKILL 08 — FRAMEWORK VALIDATION

## Purpose
Validate the generated framework against project standards, coding guidelines, architecture, and — now mandatory — Test Case traceability before delivery. Identifies issues — does NOT modify the generated framework.

## Execution Dependencies
Load the Knowledge files and Template listed for Skill 08 in agent/agent.md's **Skill Dependency Matrix** — that table is the single source of truth and is deliberately not restated here. This Skill's row is **all ten** Knowledge files, because it validates the framework against every standard.

## Inputs
Required: Test Case Model, Application Analysis Report (including its Page Inventory), Generated Page Objects (with verification methods from Skill 05), Generated Spec Files, `test-data/testData.ts` (from Skill 06), project standards.
Optional: existing framework, previous validation reports.

## Expected Output
- **Compilation result** — the exact command run and its output; clean, or every error with file and line
- Validation Report
- Validation summary
- Test Case Traceability Matrix (Test Case ID → Spec test → assertions → test data)
- List of issues
- Compliance status
- Quality score

## Responsibilities
Validate framework completeness, architecture, and generated artifacts; detect duplicate implementations and missing components; verify every generated artifact traces back to a supplied test case; measure standards compliance — checked against the delivery checklist in skills/templates/framework-output-template.md.

## Workflow
Read Test Case Model → Read Generated Framework → **Run the Compilation Check** → Validate Architecture → Validate Test Case Traceability → Validate Generated Files → Validate Standards Compliance → Detect Issues → Generate Validation Report

## Compilation Check (Critical — run this first)
**Actually compile the generated code. Do not infer that it compiles from reading it, and never infer it from tests passing.**

Run the project's type-check — `npx tsc --noEmit`, or whatever command the Target Project Profile records — and treat **every** reported error as a blocking validation failure.

**Why this is mandatory, and why passing tests do not prove it:** Playwright does not type-check. It transpiles TypeScript with esbuild, which strips type annotations without verifying them. Code with genuine type errors therefore runs, and passes, and produces a green Execution Report — while the project does not actually build. A suite can be 100% passing and the framework still broken for anyone who runs `tsc`, opens it in an IDE, or wires it into CI.

This makes compilation the one check that must never be skipped or assumed. It is also the cheapest: one command, objective pass/fail, no judgement required.

Errors this catches that no amount of reading reliably will:
- **Nullable returns from Playwright APIs** — e.g. `textContent()` returns `Promise<string | null>`, so a method declared `Promise<string>` that returns it directly is a type error (see skills/knowledge/typescript-coding-standards.md TS-004a).
- **Protected/private access violations** — e.g. a spec calling `somePage.page.goto(...)`, where `page` is `protected` in `BasePage`. This is simultaneously a POM-03 violation and a compile error.
- Missing or wrong imports, unresolved paths, incorrect method signatures, wrong argument types.

Report each error with its file, line, and the rule it maps to where one applies. A framework that does not compile cannot pass validation regardless of how clean everything else looks — this check runs before the others precisely so the rest is not evaluated against code that cannot build.

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

**Raw Playwright Access Check (Critical):** Scan every spec file for direct use of the Playwright `Page` object. A spec must never reach through a Page Object to the browser. Specifically, flag any of these in a `*.spec.ts`:
- `somePage.page.<anything>` — reaching into the `protected` `page` property (POM-03, and a compile error).
- `page.goto(...)`, `page.waitForLoadState(...)`, `page.click(...)`, `page.locator(...)`, or any other raw `page.*` call — navigation and interaction belong in Page Object methods (POM-02, BP-502).
- `expect(...)` on a locator or on `page` — assertions belong in `verify*()` methods (POM-02, VT-001).

Each is a rule violation whether or not the test passes. The repair is to add or call the appropriate Page Object method — e.g. replace `await selectUserGroupPage.page.goto('/select-user-group')` with `await selectUserGroupPage.navigateTo()` — never to loosen the access modifier on `page` so the spec compiles. Widening `protected` to `public` to silence this is a violation of POM-03, not a fix for it.

## Assertion Validation
Verify every test case's `expectedResult` has a corresponding assertion, duplicates avoided, business outcomes validated.

## Test Data Validation
Verify every test case's `testData` needs are satisfied or documented as a gap, shared data is reused, no duplicate datasets, sensitive data handled appropriately.

**Test Data Lifecycle Check:** Using Skill 06's Test Data Lifecycle Plan, confirm every test case classified state-creating or state-mutating either uses a uniqueness factory for its uniqueness-constrained values or has teardown wired — a case with neither will pass once and fail on every subsequent run, so flag it as a violation of skills/knowledge/framework-rules.md's TD-02/TD-03. Also confirm no teardown contains an assertion (TD-03) and no test consumes a record another test created (TD-04).

## Standards Compliance
Validate against: skills/knowledge/framework-rules.md, skills/knowledge/playwright-best-practices.md, skills/knowledge/typescript-coding-standards.md, skills/knowledge/naming-conventions.md, skills/knowledge/generation-patterns.md, skills/knowledge/test-case-parsing-rules.md. Identify every violation.

## Duplicate Detection
Detect duplicate methods, Page Objects, scenarios, assertions, or test data. Recommend reuse.

**Extraction Analysis Check (Critical):** Verify skills/knowledge/framework-architecture.md's Mandatory Extraction Analysis actually ran and was acted on. Independently re-check it — do not take the generation notes' word for it:
- Compare every spec's `beforeEach`/`afterEach` against every other spec's. Any routine appearing in two or more specs and not imported from `hooks/` or a fixture is a violation of RL-01/RL-02.
- **Flag inline login specifically.** Multiple specs each performing their own login in `beforeEach` is the most common instance of this and must be reported.
- Look for identical method bodies across two or more Page Objects that should have moved to `BasePage`.

**Empty `utils/`, `hooks/`, or `fixtures/` is not automatically a pass.** If those folders are empty *and* the checks above find duplication left inline, that is a failed extraction, not an absence of shared logic — flag it and require the extraction. If they are empty and genuinely nothing is shared, confirm the generation notes state that conclusion explicitly; silence is not a conclusion.

## Completeness Validation
Ensure every parsed test case is accounted for (automated or explicitly documented as not automated), every required file generated, no missing component or incomplete implementation.

## Re-Validation Mode (invoked by Skill 09)
Skill 09 re-invokes this Skill after any repair that modified generated code, before Skill 10 sees the results — because a repair can introduce exactly the violations this Skill exists to catch (an inline assertion added to route around a missing verification method, a new component-masquerading-as-a-page class, a naming or lifecycle regression).

When invoked in this mode:
- Re-run the full check set against the **current** state of the framework — the earlier Validation Report describes pre-repair code and is stale by definition.
- Scope depends on what the repairs touched: if they were confined to specific files, re-validate those plus anything importing them; if they touched a shared artifact (`BasePage.ts`, a shared component class, `testData.ts`, a `fixtures/`/`hooks/` file), re-validate the whole framework.
- Emit a **replacement** Validation Report, clearly marked as post-repair and noting which Skill 09 repairs prompted it. Never leave two reports where Skill 10 could read the outdated one.
- A violation found here is reported back to Skill 09, which repairs it under its existing retry budget rather than this Skill modifying anything — this Skill never modifies the framework, in either mode.

## Success Criteria
**Compilation check actually run and clean** · framework inspected · all artifacts validated · Page Object count matches the Page Inventory exactly · no raw Playwright `page` access in any spec · Test Case Traceability Matrix complete · test data lifecycle verified · compliance verified · issues documented · Validation Report generated (and replaced, if re-validation ran post-repair).

## Failure Handling
If validation can't be completed, document the limitation, continue validating remaining artifacts, and report incomplete validation. Never assume compliance or traceability without verification.

## Consumed By
Skill 09 — Test Execution & Self-Healing (both as its input, and as the re-validation gate Skill 09 re-invokes after any code-modifying repair) · Skill 10 — Framework Review
