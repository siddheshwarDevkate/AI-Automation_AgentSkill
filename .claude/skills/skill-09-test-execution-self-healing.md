# SKILL 09 — TEST EXECUTION & SELF-HEALING

## Purpose
Actually run the generated Playwright suite across a multi-browser matrix, and repair failures whose root cause is a framework defect (bad locator, broken Page Object logic, mismatched assertion, stale test data) — not a genuine application defect. This Skill is what turns "code that looks correct" into "code that is verified passing." Framework Validation (Skill 08) checks the code statically; this Skill checks it by actually executing it.

## Execution Dependencies
Load the Knowledge files listed for Skill 09 in .claude/agent/agent.md's **Skill Dependency Matrix** — that table is the single source of truth and is deliberately not restated here. This Skill uses no Template; it produces an Execution Report (data), not generated code, except for the narrowly-scoped `playwright.config.ts` browser matrix and worker cap described below.

## Inputs
Required: The user's base framework plus the generated Page Objects, Spec files, and Test Data, Validation Report (from Skill 08), Test Case Model, **Reuse Inventory (from Skill 00 — the list of shared artifacts whose failures are systemic, and the catalogue repairs must reuse rather than duplicate)**, live application access.
Optional: existing `playwright.config.ts`, existing execution/CI configuration, a user-specified browser list.

## Expected Output — Execution Report
- Pass/fail result per Test Case ID, per browser — **taken from the final
  full-suite confirmation run** (Execution Strategy, Stage 5)
- Execution strategy record: canary result, systemic defects fixed before full
  execution, batches run, repair re-run scope
- Root-cause diagnosis for every failure
- Repair actions taken (which Skill was re-invoked, what changed)
- **Re-Validation Gate outcome** — whether it ran, what it found, what was fixed (or an explicit note that no repair modified code)
- Test cases still failing after repair, with reason
- Application defects discovered (failures that are NOT framework bugs)
- Final per-browser and overall pass rate

## Browser Matrix
Resolve the matrix in this order, and **use the first one that applies**:
1. **A browser set the user explicitly specified** — always wins.
2. **Browsers already configured/installed in the target project** (e.g. an existing `playwright.config.ts` with MS Edge, or only Chromium installed) — use what's actually there rather than forcing installs. See Never Clobber an Existing Browser Configuration above: this config is read, never rewritten.
3. **Default: Chromium, Firefox, WebKit** — only when neither of the above applies **and** those browsers are actually installed.

**Two browser engines preferred, one accepted.** If only one browser is available and cannot be supplemented, that is a legitimate matrix — run it, and record the reduced matrix prominently in the Execution Report so single-browser results are never presented as full-matrix coverage. A restricted environment that permits exactly one approved browser is a normal, supported case, not a failure.

**Never install a browser.** Do not run `npx playwright install` or otherwise download a browser to widen the matrix — installation may be blocked by policy, and provisioning browsers is the user's decision. Resolve the matrix from what is already present.

**A test case is "Verified Passing" only once it passes on every browser in the resolved matrix.** A pass on one browser and a failure on another is still a failure, reported per-browser.

## Execution Concurrency (Critical)
The Browser Matrix (how many browser *engines* to cover) and execution concurrency (how many browser *instances* run at the same time) are separate controls — resolving a 3-browser matrix does not mean 3+ browser windows should be running simultaneously, and it must never balloon into 6. **Cap parallel execution at 2 workers — 2 browser instances running concurrently at any moment — during both the initial run and every self-healing re-run**, regardless of how many browsers are in the resolved matrix or how many test files/cases exist. Set this by configuring `workers: 2` in `playwright.config.ts` (or passing `--workers=2` to the test runner) unless a user-specified value or an existing project configuration says otherwise — same precedence as the Browser Matrix above (user-specified → project-configured → default of 2). Never let a run or a repair fan out to more concurrent browser instances than this cap, even to "go faster." The correct way to spend less wall-clock time is the Execution Strategy below — running fewer *redundant* tests — never running more of them at once.

## `playwright.config.ts` Exception
.claude/skills/knowledge/framework-architecture.md and .claude/skills/knowledge/output-structure.md normally prohibit generating `playwright.config.ts` unless explicitly requested. Running the suite requires a small amount of configuration, so this Skill is permitted to generate or update **only these four keys** — and nothing else:
- `projects` — the resolved browser matrix (subject to Never Clobber above)
- `workers` — the concurrency cap (Execution Concurrency above)
- `reporter` — a single HTML reporter (Reporting Output below)
- `use.screenshot` / `use.video` / `use.trace` — failure-only artifacts (below)

Never add anything else — CI settings, global timeouts, custom `outputDir` paths, extra reporters — unless the user asks for it.

## Reporting Output — One Report, No Clutter
**Generate exactly one report: the built-in HTML reporter.** Never add `json`, `junit`, `list`+`html` combinations, or a custom reporter unless the user explicitly asks.

```typescript
reporter: [['html', { open: 'never' }]],
use: {
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
    trace: 'on-first-retry',
},
```

**Understand what the two folders actually are before trying to remove one:**
- `playwright-report/` — the HTML report. This is the report a human reads.
- `test-results/` — **not a report.** It is Playwright's artifact directory (`outputDir`) holding screenshots, videos, and traces for individual tests.

**Do not try to merge them.** Playwright raises a configuration error when the HTML report folder and `outputDir` overlap or nest, so pointing `outputDir` inside `playwright-report/` breaks the run. The correct way to end up with only `playwright-report/` in practice is the `use` block above: with failure-only artifacts, a fully passing run produces no artifacts, so `test-results/` stays empty or is never created. When tests *do* fail, its contents are exactly what's needed to diagnose them — which is why deleting it outright is the wrong instinct.

Both directories are build output. Ensure they are git-ignored, adding them to an existing `.gitignore` if one is present.

## Never Clobber an Existing Browser Configuration (Critical)
**An existing `projects` entry is configuration the user created deliberately. Read it; do not replace it.**

This matters most in restricted environments, where a browser is often wired up by hand and cannot be reinstalled if the config is lost — for example a corporate-approved MS Edge binary referenced by an explicit `executablePath`, or a `channel` pinned to a specific build. Overwriting that with a generated `chromium`/`firefox`/`webkit` block leaves a config pointing at browsers that are not installed and cannot be installed, breaking a suite that was working.

Therefore:
- If `projects` already defines one or more browsers, **that is the resolved matrix** (Browser Matrix rule 2). Use it as-is.
- **Never remove, rename, or rewrite an existing project entry**, and never strip an `executablePath`, `channel`, `launchOptions`, or any other field on it.
- Only **append** a project when the resolved matrix genuinely requires a browser that isn't configured yet *and* that browser is already installed. If it isn't installed, do not add it and do not try to install it — record the reduced matrix instead.
- `workers` may be set per the Execution Concurrency rule whether or not `projects` already exists, since it doesn't affect which browsers run.

If the existing configuration is the only browser available, run the suite on it and report the reduced matrix prominently, per the Browser Matrix rule below. A single-browser run reported honestly is correct; a broken config is not.

## Workflow
Read Complete Framework → Configure/Confirm Browser Matrix → **Execute in Stages (see Execution Strategy below)** → For Each Failure: Diagnose Root Cause → Route to the Correct Repair Skill (if it's a framework defect) → Re-generate Only the Affected Artifact → Re-run **only the failing tests** → Repeat Until Passing or Retry Budget Exhausted → **One Final Full-Suite Confirmation Run** → **Re-Validation Gate (if any repair modified code)** → Generate Execution Report

---

# EXECUTION STRATEGY — STAGED, NOT BRUTE FORCE (Critical)

**The naive strategy — run all N tests on all browsers, fix everything, run all N again, repeat — is why this Skill dominates the run.** It is slow for one specific, avoidable reason: it keeps paying to re-learn things it already knows.

Two failure patterns account for almost all of the waste:
1. **One shared defect, N failures.** A broken `BasePage` method, a failing authentication fixture, or a type error makes *every* test fail. Running 50 tests to collect 50 copies of the same error costs 50 test-durations and yields exactly one bit of information.
2. **Re-running passing tests.** A suite of 10 with 6 failures gets fully re-run after each repair — 4 tests that were already green are executed again, every round, proving nothing.

The strategy below fixes both. Its goal is not to run fewer tests overall; it is to **never spend a test run on information already in hand.** Correctness is unchanged — the final full-suite confirmation run still proves everything passes together, and no repair may weaken a test to get there.

## Stage 1 — Canary: one test, one browser

**Run a single test first.** Pick the test whose source test case exercises the most shared infrastructure — normally a straightforward authenticated case that logs in via the fixture, navigates through `BasePage`, and asserts something simple. One browser, one test.

Its result splits the entire run in two:
- **Canary passes** → the base framework, the auth fixture, the base class, the config, and compilation are all sound. Individual failures from here are genuinely individual. Proceed to Stage 3.
- **Canary fails** → something shared is broken. **Do not run the rest of the suite.** Go to Stage 2.

This costs one test-duration and is the highest-value single action in this Skill. A suite of 50 tests that all fail on a broken login fixture takes one canary run to diagnose instead of a full matrix sweep.

## Stage 2 — Systemic-first repair: fix the shared cause before executing anything else

**A failure is systemic when its root cause lives in an artifact more than one test depends on.** The systemic artifacts are exactly the shared ones:
- compilation / import resolution failures (Root Cause 7) — nothing runs at all
- `pages/BasePage.ts` — every Page Object inherits it
- the `fixtures/` authentication fixture or any shared fixture — every authenticated test consumes it
- a `hooks/` routine used by two or more specs
- `test-data/testData.ts` module-level defects
- `playwright.config.ts` — wrong `testDir`, unusable browser entry
- the login flow itself, or the application being unreachable (which is environmental — see Failure Handling)

**When a failure is systemic, repair it before executing another test.** Route it per the Repair Routing table, re-run *only the canary* to confirm the fix, and then continue. Repeat until the canary passes.

**Never execute the remaining suite to "confirm" a systemic failure.** If the canary failed because `BasePage.navigateTo()` throws, running 49 more tests to watch it throw 49 more times adds nothing — it was already proven by the first failure. This is the single largest source of wasted execution time in this Skill, and it is forbidden. The count of affected tests is a fact you derive by reading which tests depend on the artifact, not one you buy with a suite run.

**Systemic detection also applies mid-run, not only at the canary.** If a batch in Stage 3 comes back with several failures sharing one root cause, treat it as systemic immediately: stop the batching, fix the shared cause, re-run just those failures, and resume. The trigger is a shared root cause, not a threshold count — two tests failing on the same broken `BasePage` method is the same problem as fifty.

## Stage 3 — Batched execution

With the canary green, execute the remaining tests **in batches** rather than as one monolithic run:

| Suite size | Batch size |
|---|---|
| ≤ 10 tests | run as a single batch |
| 11–30 tests | 5 |
| > 30 tests | 10 |

For each batch: run it → triage every failure → apply Stage 2's systemic check → repair the isolated failures → re-run **only the failed tests of that batch** → move to the next batch.

Batching exists so a systemic defect that the canary happened to miss is caught after 5 or 10 tests instead of after 50, and so repairs happen against a small, comprehensible failure set. Batches run at the same 2-worker cap as everything else — batching controls *how many results you look at before thinking*, not concurrency.

**Do not re-run earlier batches when starting a new one.** Their results stand until the final confirmation run.

## Stage 4 — Repair re-runs target the failing set only

**Re-run only the tests that failed. Never re-run a passing test during repair.**

With 10 tests where 4 pass and 6 fail, every repair round executes those 6 — not all 10. Playwright targets a subset directly, and each generated test carries its Test Case ID tag (NC-303), so the failing set is addressable by tag:

```
npx playwright test --grep "\[TC-003\]|\[TC-007\]|\[TC-011\]" --workers=2
```

A specific spec file, or `--grep` on the describe block, works equally well. The rule is what matters: the command names the failing tests, never the whole suite.

**The one exception is a shared-code repair.** If the fix touched `BasePage`, a fixture, a hook, or `testData.ts`, it can regress tests that were already passing — so re-run every test that depends on the changed artifact, not just the failures. This is the same regression-safety reasoning as .claude/agent/agent.md's Incremental Generation Mode, and it is not optional: a repair that fixes 6 tests and silently breaks 2 is a net loss that a failing-set-only re-run would never reveal.

## Stage 5 — One final full-suite confirmation run

**After all repairs are complete, run the entire suite once, across the full browser matrix, at the 2-worker cap.**

This run is not a formality and it is not optional. Everything before it was partial by design: batches ran in isolation, repairs were verified against subsets, and shared-code changes may have interacted in ways no subset exercised. This is the only run in which every test executes against the final state of the code.

**`execution-report.md` is populated from this run and no other.** A result carried forward from an earlier batch is not a result for the delivered framework — the code changed after it. If the final run surfaces new failures, they are diagnosed and repaired under the normal retry budget, and then the final run happens **again**. The report always reflects a single, complete, post-repair execution.

## What this strategy does not change
- **The retry budget** — still 3 repair attempts per test case per browser.
- **The concurrency cap** — still 2 workers, in every stage.
- **The browser matrix** — a test case is Verified Passing only once it passes on every browser in the resolved matrix, proven by the final run.
- **The Strict Repair Rule** — speed is never a reason to weaken a test. If the choice is between a faster green result and an honest failure, take the honest failure.
- **The Re-Validation Gate** — if any repair modified code, Skill 08 re-runs before the Execution Report is produced.

**Skipping tests is not a speed optimisation.** This strategy reduces *redundant* execution, never coverage. Every test case still ends the run with a per-browser result, and a test case quietly dropped to save time is a Delivery Gate violation, not an efficiency win.

## Record the strategy in the Execution Report
Report what was actually run, so the numbers are auditable:
- Canary test used, and its result
- Systemic defects found and repaired before full execution, with the number of tests each would have failed
- Batches run, and each batch's pass/fail counts
- Repair re-runs: which tests were re-run each round (and, for shared-code repairs, which dependents were pulled in)
- The final full-suite run: browsers, duration, and per-test-case results

---

## Root-Cause Diagnosis
For every failing test, classify the failure before touching anything. **Classify along two axes: the root cause below, and whether it is systemic or isolated** (Execution Strategy, Stage 2) — the first decides which Skill repairs it, the second decides whether anything else runs before that repair happens.
1. **Locator drift** — element no longer matches the selector used.
2. **Page Object logic defect** — wrong step order, wrong method, timing issue introduced during generation.
3. **Assertion mismatch (framework-side)** — the verification method checks the wrong element/value, but the application actually behaves as the test case's `expectedResult` describes.
4. **Test data conflict** — stale/duplicate/invalid data causes a false failure (e.g. "record already exists").
5. **Flakiness** — inconsistent timing/race condition, not a real defect; fix with proper Playwright waiting (per .claude/skills/knowledge/playwright-best-practices.md), never with `waitForTimeout()`.
6. **Genuine application defect** — the application's actual behaviour contradicts the test case's `expectedResult`, confirmed by direct observation against the live app.
7. **Build/compile defect** — the suite doesn't run because the generated code doesn't compile or resolve: a broken import, a missing module, a type error, a path that doesn't match the project's actual layout. **This is a framework defect, not an environment failure**, and it is repaired here like any other.

Only categories 1–5 and 7 are repaired by this Skill. Category 6 is never repaired — see the Strict Repair Rule below.

**Category 7 is the one most often misclassified as "cannot execute."** Skill 00's readiness gate already proved the runner, browsers, and application work and that the project compiled *before* generation. So a compile or import failure now means this run introduced it — most commonly by generating into a folder layout the project's own `playwright.config.ts`/`tsconfig.json` doesn't point at. Fix the generated code or its paths to match the Target Project Profile; do not report it as a Critical Failure and stop.

## Repair Routing
| Root Cause | Re-invoke | Scope of Change |
|---|---|---|
| Locator drift | Skill 03 — Locator Generation | Only the broken locator(s) |
| Page Object logic defect | Skill 04 — Page Object Generation | Only the affected method(s) |
| Assertion mismatch (framework-side) | Skill 05 — Assertion Generation | Only the affected verification method |
| Test data conflict | Skill 06 — Test Data Generation (+ Skill 07 if teardown wiring is missing) | The affected dataset in `testData.ts`, and the spec's teardown if that's the gap |
| Flakiness | Skill 04/07 — replace hard waits with proper Playwright waiting | Only the affected method/test |
| Build/compile defect | Skill 04/06/07 — whichever generated the offending file | Only the broken import/path/type, realigned to the Target Project Profile |
| **Base framework defect** (base class, auth fixture, shared hook — **user-owned code**) | **Skill 00 — Framework Inventory** | Only the broken method/fixture. **Always systemic, always fixed before further execution, and always reported explicitly** — see Repairing User-Owned Code below |

**Repairs obey the Reuse-First Rules** (RS-01–RS-05, .claude/skills/skill-00-framework-inventory.md). A repair may not fix a failure by pasting a local copy of logic that already exists in the base class, `utils/`, `fixtures/`, or `hooks/` — that is an RS-01 violation, and this is where the temptation is highest. Call the existing one; if something genuinely new is needed, add it beside its siblings in the user's structure (RS-03). Skill 08's Duplicate Detection catches this at the Re-Validation Gate, so a duplicate-based repair costs a round rather than saving one.

## Repairing User-Owned Code (Critical)
The base class, `utils/`, `fixtures/`, and `hooks/` are hand-written by the user. Generated Page Objects, specs, and test data are this pipeline's. **Repairs are not equally free across that line.**

- **Generated code** — repair it freely under the routing above. This run produced it; this run fixes it.
- **User-owned code** — first ask whether the defect is really there. The overwhelmingly likely explanation for a failure in working, pre-existing code is that the generated code is calling it wrongly: wrong argument, wrong order, a precondition the generated Page Object skipped. **Fix the call site, not the callee.**
- If the defect genuinely is in the base framework, fix the minimum and **report it prominently in the Execution Report** — what was changed, why, and what else calls it. A silent edit to a method other tests depend on is how a repair that fixed one test breaks five that were never in this run's scope.
- **Never rewrite, rename, or restructure user-owned code to make a generated test pass.** That is the same failure as weakening an assertion, in a different place. Adding a new sibling method is fine; changing an existing one is a reported change.

After a repair, re-run only the affected test(s) on every browser before moving on — per Execution Strategy, Stage 4. **Never re-run the whole suite mid-repair**; the only re-run that widens beyond the failing set is a shared-code repair, which pulls in that artifact's dependents. The full suite runs exactly once, at Stage 5.

**Data collisions are a lifecycle gap, not a value problem.** When a failure is diagnosed as a test data conflict on a state-creating/state-mutating test case ("record already exists", "duplicate key", a record left behind by an earlier run), the repair is to add the missing uniqueness factory or teardown per .claude/skills/knowledge/test-data-lifecycle.md — routed back to Skill 06 for the factory, Skill 07 for the teardown wiring. Hand-editing a literal in `testData.ts` so this one run passes leaves the identical failure waiting for the next run and is forbidden under TD-05.

## Strict Repair Rule (Critical)
**Never repair a test by weakening it.** Specifically, never:
- Loosen, remove, or comment out an assertion just to make a test pass.
- Alter test data or a test case's expected values to match a wrong actual result.
- Add a `try/catch` that swallows the failure.
- Retry-until-green without understanding why it failed first.

If the actual application behaviour contradicts the test case's `expectedResult` (Root Cause 6), this is a **genuine application defect or a stale test case** — not something this Skill fixes. Leave the test failing, document it as an Application Defect / Test Case Discrepancy in the Execution Report, and surface it to the Agent for the user to resolve (fix the application, or confirm the test case needs updating). Silently forcing a green result here is a fabrication and is forbidden under AP-004 and AP-010 in .claude/agent/agent.md.

## Retry Budget
Cap repair attempts at **3 per test case per browser**. If a test case still fails after 3 repair attempts, stop repairing it, mark it **"Blocked — Manual Review Required"** in the Execution Report with the full diagnosis history, and continue with the remaining test cases. Never loop indefinitely on a single failure.

**A systemic repair consumes one attempt, not one per affected test.** Fixing a broken `BasePage` method that was failing 30 tests is a single repair attempt against a single artifact — it does not exhaust 30 budgets, and it does not exhaust the budget of any individual test case. Per-test-case budget is spent only on repairs scoped to that test case. Otherwise a single shared defect would burn the entire suite's budget in one round and mark 30 healthy tests "Blocked."

## Flakiness Handling
A test that fails intermittently (passes on retry with no code change) is not automatically "healed" by retrying until green. Diagnose why it's flaky (usually a missing/incorrect wait) and fix the underlying timing issue per .claude/skills/knowledge/playwright-best-practices.md. Document tests that could not be stabilized after the retry budget as flaky, not as passing.

## Passing Tests Do Not Mean Compiling Code (Critical)
**Playwright does not type-check.** It transpiles TypeScript with esbuild, which strips type annotations without verifying them. Code containing real type errors therefore runs normally and passes — producing a fully green Execution Report for a framework that does not build.

So a green suite is **not** evidence the code compiles, and this Skill must never present it as such. Two failure modes this creates, both seen in practice:
- A method declared `Promise<string>` returning `textContent()` (`Promise<string | null>`) — runs fine, fails `tsc`.
- A spec calling `somePage.page.goto(...)` on a `protected` property — runs fine, fails `tsc`, and violates POM-03.

Compilation is proven only by Skill 08's Compilation Check. That is why the Re-Validation Gate below is not optional whenever code changed: **"all tests passed" and "the framework is valid" are separate claims, and this Skill can only establish the first one.**

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
Canary run first and systemic defects repaired before further execution · remaining tests executed in batches · repair re-runs scoped to the failing set (widened only for shared-code repairs) · **full suite executed once on every browser in the matrix as the final confirmation run, and the Execution Report populated from it** · execution capped at 2 concurrent workers in every stage · every failure diagnosed before any repair attempt · repairs scoped to the correct Skill and artifact only, with no duplicated logic introduced (RS-01) · data collisions repaired as lifecycle gaps, not literal edits · genuine application defects documented, never masked · retry budget respected · **Re-Validation Gate run whenever a repair modified code, with its outcome recorded** · Execution Report complete and cross-referenced against the Test Case Model.

## Failure Handling

**"Cannot be executed" means the environment is gone, not that the code is broken.** This distinction is the difference between stopping correctly and quitting on work you were supposed to do.

**Critical Failure — stop and report.** Only when the environment itself is unavailable: the runner has disappeared, no browser can launch, or the application is unreachable. Skill 00's Phase A verified all three before generation began, so this means something changed underneath the run. Do not hand an unexecuted framework to Skill 10 as if it were verified.

**Not a Critical Failure — repair it.** Anything wrong with the code this run produced: it doesn't compile, an import doesn't resolve, a config points at a path the generated files don't occupy, a type error. That is Root Cause 7, and it is this Skill's job. Skill 00 proved the project compiled before generation; if it doesn't compile now, this run broke it and this run fixes it. **Stopping here and reporting "the suite cannot be executed" is the single most common way this Skill gets skipped — it is not a valid exit.**

If only specific test cases are blocked after the retry budget, that is a non-critical failure — document them and continue.

## This Skill Is Not Optional
Generating the files is not the deliverable. A framework that has never been executed is unverified regardless of how correct it looks, and .claude/agent/agent.md's Delivery Gate mechanically blocks handover until `execution-report.md` contains a per-browser result for every Test Case ID. If this Skill is reached and does not run the suite, the pipeline has failed — there is no state in which producing Skills 00–08's output and stopping is a complete run.

## Consumed By
Skill 10 — Framework Review
