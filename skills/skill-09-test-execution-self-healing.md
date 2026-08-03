# SKILL 09 — TEST EXECUTION & SELF-HEALING

## Purpose
Actually run the generated Playwright suite across a multi-browser matrix, and repair failures whose root cause is a framework defect (bad locator, broken Page Object logic, mismatched assertion, stale test data) — not a genuine application defect. This Skill is what turns "code that looks correct" into "code that is verified passing." Framework Validation (Skill 08) checks the code statically; this Skill checks it by actually executing it.

## Execution Dependencies
**Knowledge:** skills/knowledge/framework-rules.md, skills/knowledge/playwright-best-practices.md, skills/knowledge/locator-strategy.md, skills/knowledge/generation-patterns.md, skills/knowledge/output-structure.md, skills/knowledge/test-case-parsing-rules.md
**Templates:** None — produces an Execution Report (data), not generated code, except for the narrowly-scoped `playwright.config.ts` browser matrix described below.

## Inputs
Required: Complete generated framework (Page Objects, Spec files, Test Data), Validation Report (from Skill 08), Test Case Model, live application access.
Optional: existing `playwright.config.ts`, existing execution/CI configuration, a user-specified browser list.

## Expected Output — Execution Report
- Pass/fail result per Test Case ID, per browser
- Root-cause diagnosis for every failure
- Repair actions taken (which Skill was re-invoked, what changed)
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

## `playwright.config.ts` Exception
skills/knowledge/framework-architecture.md and skills/knowledge/output-structure.md normally prohibit generating `playwright.config.ts` unless explicitly requested. Running a multi-browser matrix requires browser `projects` to be configured, so this Skill is explicitly permitted to generate or update **only the `projects` array** of `playwright.config.ts` (Chromium/Firefox/WebKit definitions). Never add unrelated configuration (CI settings, custom reporters, global timeouts, etc.) unless the user asks for it.

## Workflow
Read Complete Framework → Configure/Confirm Browser Matrix → Run Full Suite on Every Browser → Collect Results → For Each Failure: Diagnose Root Cause → Route to the Correct Repair Skill (if it's a framework defect) → Re-generate Only the Affected Artifact → Re-run the Affected Test on Every Browser → Repeat Until Passing or Retry Budget Exhausted → Generate Execution Report

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
| Test data conflict | Skill 06 — Test Data Generation | Only the affected dataset in `testData.ts` |
| Flakiness | Skill 04/07 — replace hard waits with proper Playwright waiting | Only the affected method/test |

After a repair, re-run only the affected test(s) on every browser before moving on — don't re-run the entire suite unless the fix touched shared code (BasePage, a shared component, a shared dataset).

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

## Coverage & Completeness
Cross-reference the Execution Report against the Test Case Model (per Skill 01) and the Test Case Traceability Matrix (per Skill 08): every automated test case must have a final per-browser pass/fail/blocked status. No test case may be silently omitted from the Execution Report.

## Success Criteria
Full suite executed on every browser in the matrix · every failure diagnosed before any repair attempt · repairs scoped to the correct Skill and artifact only · genuine application defects documented, never masked · retry budget respected · Execution Report complete and cross-referenced against the Test Case Model.

## Failure Handling
If the suite cannot be executed at all (missing runner, browsers not installed, application unreachable), stop and report this as a Critical Failure — do not hand an unexecuted framework to Skill 10 as if it were verified. If only specific test cases are blocked after the retry budget, that is a non-critical failure — document them and continue.

## References
**Knowledge:** skills/knowledge/framework-rules.md, skills/knowledge/playwright-best-practices.md, skills/knowledge/locator-strategy.md, skills/knowledge/generation-patterns.md, skills/knowledge/output-structure.md, skills/knowledge/test-case-parsing-rules.md
**Templates:** None
**Consumed by:** Skill 10 — Framework Review
