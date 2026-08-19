# SKILL 07 — SPEC GENERATION

## Purpose
Convert the Test Case Model into executable, business-focused Playwright specs with strict 1:1 traceability — every supplied test case becomes exactly one generated spec, and no scenario is added that wasn't in the Test Case file. By the time this Skill runs, Page Objects (Skill 04) already have every action AND verification method they need (Skill 05), and `test-data/testData.ts` already exists (Skill 06) — this Skill only wires them together into tests, it never invents a method or a data value. .claude/skills/templates/spec-template.md governs HOW the file is structured.

## Execution Dependencies
Load the Knowledge files and Template listed for Skill 07 in .claude/agent/agent.md's **Skill Dependency Matrix** — that table is the single source of truth and is deliberately not restated here.

## Inputs
Required: Test Case Model (from Skill 01), Application Analysis Report, **Reuse Inventory (from Skill 00 - the project's authentication fixture, hooks, and helpers)**, Generated Page Objects (from Skill 04, with verification methods already added by Skill 05), `test-data/testData.ts` (from Skill 06).
Optional: existing Spec files.

## Expected Output
- Spec files, one test per Test Case
- Test Case ID → Spec mapping (traceability table)

## Strict Traceability Rule
Every parsed test case (per Skill 01's Test Case Model) produces exactly one spec test. Do not discover or add extra scenarios beyond the supplied test cases, even if the exploration in Skill 02 surfaced additional interesting behaviour — log those as out-of-scope observations instead. Do not merge two test cases into one test, and do not split one test case into several tests, unless a single test case explicitly requires multiple assertions to satisfy its own `expectedResult`.

**Granularity:** A generated `test()` executes the full ordered sequence of its test case's steps as one method — never generate a separate `test()` per individual atomic step. This does not mean deferring all verification to the end: if the test case's `steps` carry a per-step expected result (not just a single final `expectedResult`), call the matching verification method inline right after that step, for every step that has one — not just after the last one. One test case still yields one `test()` method; it can, and should, contain multiple assertions when the test case itself defines multiple checkpoints.

## Responsibilities
- Map every test case to a spec file based on its `module`
- Generate exactly one test per test case, tagged with its `id`
- Reuse Page Object action/verification methods (Skills 04–05) and data constants (Skill 06) — never define new ones here
- Flag any test case that cannot be automated (missing Page Object method, verification method, or data constant) instead of fabricating a workaround
- **Check the Reuse Inventory before writing any helper — the user's `utils/`/`fixtures/`/`hooks/` usually already has it (RS-01).** Never define a reusable/common function inline inside a spec file; call the existing one, or add a new sibling to `utils/`, `fixtures/`, or `hooks/` in the user's style when nothing equivalent exists, per .claude/skills/knowledge/framework-architecture.md's Reusable Logic Placement section (see also .claude/skills/knowledge/framework-rules.md's RL-01–RL-04)
- Wire teardown for every test case Skill 06's Test Data Lifecycle Plan marked state-creating or state-mutating, per .claude/skills/knowledge/test-data-lifecycle.md's Teardown section
- Generate the actual Spec files per .claude/skills/templates/spec-template.md

## Extraction Pass (run before finalizing the specs)
Generate the specs first, then — before declaring this Skill complete — run .claude/skills/knowledge/framework-architecture.md's **Mandatory Extraction Analysis** across everything just written, and refactor what it finds. This is a required step with a recorded outcome, not an opportunistic one.

Concretely, compare every spec's `beforeEach`/`afterEach` and setup code against every other spec's:
- Same routine in two or more specs → import the existing `hooks/` routine if the Inventory has one; otherwise add a new one to `hooks/` and import it.
- Shared precondition (logged-in user, seeded record, selected context) → consume the existing fixture; the authenticated session in particular is nearly always already provided.
- Repeated helper logic with no Playwright lifecycle dependency → call the existing `utils/` helper, or add a new one there.

**Inline login in every spec is the failure this catches — and the user's framework already solved it.** The authentication fixture is hand-written, already working, and recorded by name in the Reuse Inventory. Every spec needing a logged-in session **consumes that fixture** (RS-05). Writing a login into a spec's `beforeEach` is a violation of RL-05 whether or not another spec duplicates it, and it is never necessary — the fixture existed before this pipeline ran, and a second login path will drift from the one the user maintains.

State the outcome in the Spec Generation notes: what was reused from the existing framework, what (if anything) was added to it and where, or an explicit "everything shared was already provided by the base framework." Adding nothing is the expected result — but only as that stated conclusion, never as the result of not checking.

## Test Data Lifecycle Wiring
Skill 06 produced a Test Data Lifecycle Plan classifying each test case as read-only, state-creating, or state-mutating. Honour it here:
- **Call the uniqueness factory, never the raw constant**, for a state-creating test case's uniqueness-constrained values — `await customerPage.createCustomer(buildUniqueCustomerName())`, not a fixed literal or a shared constant that every run reuses.
- **Wire teardown** for every state-creating/state-mutating case, per .claude/skills/knowledge/test-data-lifecycle.md's preference order: the page's own cleanup method in `afterEach()`, a shared routine in `hooks/` when two or more specs need it, or a `fixtures/` fixture when setup and teardown belong together.
- **Never assert inside teardown**, and never let a test consume a record another test created — under Skill 09's parallel workers there is no guaranteed ordering between them.

If the Lifecycle Plan flags a case where uniqueness could not be guaranteed and no cleanup path exists, document it as a lifecycle gap for the coverage report rather than generating a test that will collide on its second run.

## Workflow
Read Test Case Model → Read Application Analysis Report → Read Generated Page Objects and testData.ts → Group Test Cases by Module → Determine Spec File Boundaries per Module → For Each Test Case: Map Steps to Page Object Action Methods → Map `expectedResult` to the Verification Method Skill 05 Already Added → Import the Data Constant Skill 06 Already Created → Generate One Tagged Test → Validate 1:1 Coverage → Generate Spec Files per .claude/skills/templates/spec-template.md

## Feature/Module Boundaries
Use the Test Case Model's `module` grouping to determine Spec file boundaries (one Spec file per module, per .claude/skills/knowledge/framework-architecture.md). Never combine test cases from different modules into the same Spec file. **`module` and Page Inventory `page` are independent groupings** — one module can span multiple pages and one page can serve multiple modules; never assume a 1:1 mapping between them (see .claude/skills/knowledge/framework-architecture.md's Module vs Page distinction). This is a deliberate, confirmed decision: an external requirements document may describe Spec boundaries as "one file per Page Class" instead — that phrasing is not followed here, precisely because of the real prior incident documented in the Module vs Page distinction. Do not switch Spec file boundaries to page-based grouping without this decision being explicitly revisited.

## Test Case → Test Mapping
For each test case:
1. Read its ordered `steps` and map each step to an existing Page Object action method (from Skill 04). If no matching method exists, document the gap — do not invent Page Object behaviour here.
2. Read its expected result(s) — Skill 05 maps these to verification methods per-step when the source test case provides a step-level expected result for a given step, and as one overall check otherwise (see .claude/skills/skill-05-assertion-generation.md's Validation Point Identification). Call each verification method immediately after the step it validates — do not collect every action first and every assertion afterward when the test case defines multiple checkpoints. If a needed verification method doesn't exist, that is a Skill 05 gap — document it, do not write an inline assertion in the spec to route around it.
3. Import the data constant(s) Skill 06 already mapped to this test case from `test-data/testData.ts`. If none exists for a value the steps need, that is a Skill 06 gap — document it, do not hardcode a literal in the spec to route around it.
4. Generate one Playwright test whose title is derived from the test case's `title`/`id`, and whose body interleaves action and verification calls in step order — only Page Object methods, no inline assertions.
5. Tag the test with its source `id` for traceability — e.g. include it in the test title (`"[TC-014] Valid login redirects user to dashboard"`) or as a Playwright annotation/tag, per .claude/skills/templates/spec-template.md.

## Scenario Classification
Classify each generated test using the test case's own `type` field when present (positive, negative, boundary, etc.); if `type` is absent, infer the most accurate classification from the test case's steps and `expectedResult` and document the inference.

## Unautomatable Test Cases
If a test case cannot be automated — a referenced page/element doesn't exist (per Skill 02's Test Case Discrepancies), or a required Page Object action/verification method or data constant is missing — do not generate a placeholder or fabricated test. Exclude it from the Spec files and record it as "Not Automated" with the reason, for the final coverage report.

## Duplicate Detection
Test cases are the source of truth, so duplicates should already be resolved in Skill 01. If two test cases are functionally identical, still generate both tests (traceability requires one test per supplied `id`) but note the duplication in the Spec Generation notes.

## Coverage Analysis
Before completing, verify every parsed test case maps to exactly one generated test or is explicitly recorded as "Not Automated." Coverage is measured against the Test Case Model, not against discovered application functionality.

## Success Criteria
Every test case mapped to a Spec file · one test generated per test case · every generated test tagged with its source `id` · every test calls only pre-existing Page Object methods and pre-existing data constants (no inline assertions, no hardcoded literals) · every spec consumes the project's authentication fixture rather than logging in inline (RS-05) · no reusable/common function defined inline inside a spec file, and none rewritten that the Reuse Inventory already provides (RS-01) · every state-creating/state-mutating test case uses its uniqueness factory and has teardown wired · Extraction Pass run and its outcome recorded (shared setup moved to `hooks/`/`fixtures/`, or an explicit "nothing shared") · unautomatable test cases documented, not fabricated · Spec files generated.

## Failure Handling
If a test case can't be automated, document it as "Not Automated" with the reason and continue with the rest. Never invent a scenario, step, Page Object behaviour, verification method, or data value to force a test case through.

## Consumed By
Skill 08 — Framework Validation
