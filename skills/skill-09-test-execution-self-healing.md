# SKILL 09 — TEST EXECUTION & SELF-HEALING

## Purpose
Actually run the generated Playwright suite across a multi-browser matrix, and repair failures whose root cause is a framework defect (bad locator, broken Page Object logic, mismatched assertion, stale test data) — not a genuine application defect. This Skill is what turns "code that looks correct" into "code that is verified passing." Framework Validation (Skill 08) checks the code statically; this Skill checks it by actually executing it.

## Execution Dependencies
Load the Knowledge files listed for Skill 09 in agent/agent.md's **Skill Dependency Matrix** — that table is the single source of truth and is deliberately not restated here. This Skill uses no Template; it produces an Execution Report (data), not generated code, except for the narrowly-scoped `playwright.config.ts` browser matrix and worker cap described below.

## Inputs
Required: Complete generated framework (Page Objects, Spec files, Test Data), Validation Report (from Skill 08), Test Case Model, live application access.
Optional: existing `playwright.config.ts`, existing execution/CI configuration, a user-specified browser list.

## Expected Output — Execution Report
- Pass/fail result per Test Case ID, per browser
- Root-cause diagnosis for every failure
- Repair actions taken (which Skill was re-invoked, what changed)
- **Re-Validation Gate outcome** — whether it ran, what it found, what was fixed (or an explicit note that no repair modified code)
- Test cases still failing after repair, with reason
- Application defects discovered (failures that are NOT framework bugs)
- Final per-browser and overall pass rate

## Browser Matrix
Resolve the matrix in this order, and **use the first one that applies**:
1. **A browser set the user explicitly specified** — always wins.
2. **Browsers already configured/installed in the target project** (e.g. an existing `playwright.config.ts` with MS Edge, or only Chromium installed) — use what's actually there rather than forcing installs.
3. **Default: Chromium, Firefox, WebKit** — when neither of the above applies.

**Minimum two browser engines.** If only one is available and cannot be supplemented, run it, and record the reduced matrix prominently in the Execution Report — never report single-browser results as if they were full-matrix coverage.

**A test case is "Verified Passing" only once it passes on every browser in the resolved matrix.** A pass on one browser and a failure on another is still a failure, reported per-browser.

## Execution Concurrency (Critical)
The Browser Matrix (how many browser *engines* to cover) and execution concurrency (how many browser *instances* run at the same time) are separate controls — resolving a 3-browser matrix does not mean 3+ browser windows should be running simultaneously, and it must never balloon into 6. **Cap parallel execution at 2 workers — 2 browser instances running concurrently at any moment — during both the initial run and every self-healing re-run**, regardless of how many browsers are in the resolved matrix or how many test files/cases exist. Set this by configuring `workers: 2` in `playwright.config.ts` (or passing `--workers=2` to the test runner) unless a user-specified value or an existing project configuration says otherwise — same precedence as the Browser Matrix above (user-specified → project-configured → default of 2). Never let a run or a repair fan out to more concurrent browser instances than this cap, even to "go faster."

## `playwright.config.ts` Exception
skills/knowledge/framework-architecture.md and skills/knowledge/output-structure.md normally prohibit generating `playwright.config.ts` unless explicitly requested. Running a multi-browser matrix requires browser `projects` to be configured, so this Skill is explicitly permitted to generate or update **only the `projects` array and the top-level `workers` value** of `playwright.config.ts` (Chromium/Firefox/WebKit `projects` definitions, and `workers` per the Execution Concurrency rule above). Never add unrelated configuration (CI settings, custom reporters, global timeouts, etc.) unless the user asks for it.

## Workflow
Read Complete Framework → Configure/Confirm Browser Matrix → Run Full Suite on Every Browser → Collect Results → For Each Failure: Diagnose Root Cause → Route to the Correct Repair Skill (if it's a framework defect) → Re-generate Only the Affected Artifact → Re-run the Affected Test on Every Browser → Repeat Until Passing or Retry Budget Exhausted → **Re-Validation Gate (if any repair modified code)** → Generate Execution Report

## Root-Cause Diagnosis
For every failing test, classify the failure before touching anything:
1. **Locator drift** — element no longer matches the selector used.
2. **Page Object logic defect** — wrong step order, wrong method, timing issue introduced during generation.
3. **Assertion mismatch (framework-side)** — the verification method checks the wrong element/value, but the application actually behaves as the test case's `expectedResult` describes.
4. **Test data conflict** — stale/duplicate/invalid data causes a false failure (e.g. "record already exists").
5. **Flakiness** — inconsistent timing/race condition, not a real defect; fix with proper Playwright waiting (per skills/knowledge/playwright-best-practices.md), never with `waitForTimeout()`.
6. **Genuine application defect** — the application's actual behaviour contradicts the test case's `expectedResult`, confirmed by direct observation against the live app.

Only categories 1–5 are repaired by this Skill. Category 6 is never repaired — see the Strict Repair Rule below.

## Repair Routing
| Root Cause | Re-invoke | Scope of Change |
|---|---|---|
| Locator drift | Skill 03 — Locator Generation | Only the broken locator(s) |
| Page Object logic defect | Skill 04 — Page Object Generation | Only the affected method(s) |
| Assertion mismatch (framework-side) | Skill 05 — Assertion Generation | Only the affected verification method |
| Test data conflict | Skill 06 — Test Data Generation (+ Skill 07 if teardown wiring is missing) | The affected dataset in `testData.ts`, and the spec's teardown if that's the gap |
| Flakiness | Skill 04/07 — replace hard waits with proper Playwright waiting | Only the affected method/test |

After a repair, re-run only the affected test(s) on every browser before moving on — don't re-run the entire suite unless the fix touched shared code (BasePage, a shared component, a shared dataset).

**Data collisions are a lifecycle gap, not a value problem.** When a failure is diagnosed as a test data conflict on a state-creating/state-mutating test case ("record already exists", "duplicate key", a record left behind by an earlier run), the repair is to add the missing uniqueness factory or teardown per skills/knowledge/test-data-lifecycle.md — routed back to Skill 06 for the factory, Skill 07 for the teardown wiring. Hand-editing a literal in `testData.ts` so this one run passes leaves the identical failure waiting for the next run and is forbidden under TD-05.

## Strict Repair Rule (Critical)
**Never repair a test by weakening it.** Specifically, never:
- Loosen, remove, or comment out an assertion just to make a test pass.
- Alter test data or a test case's expected values to match a wrong actual result.
- Add a `try/catch` that swallows the failure.
- Retry-until-green without understanding why it failed first.

If the actual application behaviour contradicts the test case's `expectedResult` (Root Cause 6), this is a **genuine application defect or a stale test case** — not something this Skill fixes. Leave the test failing, document it as an Application Defect / Test Case Discrepancy in the Execution Report, and surface it to the Agent for the user to resolve (fix the application, or confirm the test case needs updating). Silently forcing a green result here is a fabrication and is forbidden under AP-004 and AP-010 in agent/agent.md.

## Retry Budget
Cap repair attempts at **3 per test case per browser**. If a test case still fails after 3 repair attempts, stop repairing it, mark it **"Blocked — Manual Review Required"** in the Execution Report with the full diagnosis history, and continue with the remaining test cases. Never loop indefinitely on a single failure.

## Flakiness Handling
A test that fails intermittently (passes on retry with no code change) is not automatically "healed" by retrying until green. Diagnose why it's flaky (usually a missing/incorrect wait) and fix the underlying timing issue per skills/knowledge/playwright-best-practices.md. Document tests that could not be stabilized after the retry budget as flaky, not as passing.

## Re-Validation Gate (Critical)
**If any repair in this Skill modified generated code, re-invoke Skill 08 before producing the Execution Report.** Skill 08's original Validation Report describes the framework as it existed *before* those repairs — handing it to Skill 10 unchanged means the final production-readiness decision is made against code that no longer exists.

This matters because repairs can reintroduce exactly what Skill 08 exists to catch: an inline assertion written into a spec to route around a missing verification method, a new class created for a component instead of folding it into its page's Page Object, a naming-convention regression, a lifecycle gap. A repair that fixes a test failure while breaking a framework rule is not a successful repair.

**How the gate runs:**
1. After the repair loop finishes (all tests passing or retry budget exhausted), check whether any repair modified a file. If nothing was modified, skip the gate and proceed to the Execution Report.
2. Re-invoke Skill 08 in its Re-Validation Mode, scoped per that Skill: the modified files plus their importers, or the whole framework if a shared artifact (`BasePage.ts`, a shared component class, `testData.ts`, a `fixtures/`/`hooks/` file) was touched.
3. If re-validation reports violations, repair them under this Skill's existing retry budget, then run the gate again. A test whose violation cannot be resolved within budget is marked **"Blocked — Manual Review Required"** like any other, with the violation as its reason.
4. Record the outcome in the Execution Report: whether the gate ran, what it found, and what was fixed. If the gate did not run because nothing was modified, say so explicitly.

**Never pass an un-re-validated, repaired framework to Skill 10.** Skill 10 must receive the post-repair Validation Report, not the pre-repair one.

## Coverage & Completeness
Cross-reference the Execution Report against the Test Case Model (per Skill 01) and the Test Case Traceability Matrix (per Skill 08): every automated test case must have a final per-browser pass/fail/blocked status. No test case may be silently omitted from the Execution Report.

## Success Criteria
Full suite executed on every browser in the matrix · execution capped at 2 concurrent workers · every failure diagnosed before any repair attempt · repairs scoped to the correct Skill and artifact only · data collisions repaired as lifecycle gaps, not literal edits · genuine application defects documented, never masked · retry budget respected · **Re-Validation Gate run whenever a repair modified code, with its outcome recorded** · Execution Report complete and cross-referenced against the Test Case Model.

## Failure Handling
If the suite cannot be executed at all (missing runner, browsers not installed, application unreachable), stop and report this as a Critical Failure — do not hand an unexecuted framework to Skill 10 as if it were verified. If only specific test cases are blocked after the retry budget, that is a non-critical failure — document them and continue.

## Consumed By
Skill 10 — Framework Review
