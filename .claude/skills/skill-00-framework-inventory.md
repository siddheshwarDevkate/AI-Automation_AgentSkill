# SKILL 00 — TARGET READINESS & FRAMEWORK INVENTORY

## Purpose
Understand the target completely, before anything is generated. Two jobs, one
Skill, because both are the same act — reading the target project:

1. **Readiness (Phase A + B).** Prove the suite can actually be run and the
   project can actually be generated into: Test Case file present, runner
   installed, a browser launchable, the application reachable and the Login
   Recipe genuinely authenticating, the project compiling, and a base framework
   present. **Any failure hard-stops the run here, at near-zero cost.**
2. **Inventory (Phase B).** **Read the base framework the user already wrote**
   and produce a precise catalogue of every reusable asset in it — base-class
   methods, `utils/` helpers, `fixtures/`, `hooks/` — so that every later Skill
   *calls* that code instead of writing its own version of it.

**This Skill generates almost nothing.** The base framework — the base class, the
reusable methods, the helper functions, the browser and authentication setup — is
hand-written and owned by the user. It is an **input to this pipeline, not an
output of it.** Skills 03–07 generate Page Objects, test data, and specs *on top
of* it.

## Why readiness and inventory are one Skill
Skill 09 is the only Skill whose preconditions no earlier Skill produces — it
needs a runner, a browser, a reachable application, and a project that compiles.
Nothing in Skills 01–08 establishes any of those. Without a gate up front, a
target that cannot run tests is discovered only at Skill 09, after all the work
is done — at which point stopping looks like the correct action and the entire
run is wasted.

Checking readiness separately from the inventory meant reading the target project
twice — once to ask "does it compile, is there a framework," and again to ask
"what is in that framework." Two reads of the same files is two chances to
disagree with each other. They are one read, here.

**Order matters within it, though: cheap checks first.** Reading an entire
framework in full is expensive. Discovering afterwards that the test runner isn't
installed wastes all of it. So Phase A runs the seconds-long environment checks
and stops on failure; Phase B only starts once the environment is known good.

## Why the inventory matters
An AI that has not read the existing framework does not reuse it — it cannot,
because it does not know what is there. It writes its own `waitForElement()`
alongside the one already in `BasePage`, its own login inside a `beforeEach`
alongside the fixture that already exists, its own date formatter next to
`utils/DateHelper.ts`. Every one of those is a duplicate of working code the user
already wrote and already trusts.

The failure is not a lack of discipline; it is a lack of information. So this
Skill's job is to supply the information, once, up front, in a form later Skills
can check against mechanically: **the Reuse Inventory.** Every method, with its
exact signature and what it does. RS-01 ("search before writing") is only
enforceable because this inventory exists.

## Execution Dependencies
Load the Knowledge files listed for Skill 00 in .claude/agent/agent.md's **Skill
Dependency Matrix** — that table is the single source of truth and is
deliberately not restated here. This Skill uses no Template: it produces an
inventory (data), not generated code.

## Inputs
Required: Intake results from .claude/agent/agent.md's Intake Gate (application
URL, credentials, **Login Recipe**, Test Case file location), **the user's
existing base framework in the target project**.

## Position in the Pipeline
Runs immediately after the Intake Gate, before Test Case Analysis — the first
Skill in the pipeline.

It reads no test cases and generates nothing derived from them, so it does not
breach Skill 01's gate. It runs first because everything depends on it: Skill 01
needs the Test Case file to exist, Skills 03–07 need the inventory from their
very first generated file, and Skill 09 needs an environment that works. A reuse
map produced after the Page Objects were written is a code review, not a reuse
map.

---

# PHASE A — ENVIRONMENT GATE (seconds; stop on any failure)

Run these before reading a single framework file. Each is cheap and objective.

1. **Test Case file** — present in `<resource-root>/input/`, readable, ≥1 valid
   row. (Skill 01 re-validates in depth; this is the cheap existence check.)
   **This check is what lets this Skill run before Skill 01** — the "no Test Case
   file, no generation" rule is satisfied by proof the file exists, not by Skill
   01 having finished parsing it.
2. **Test runner** — a Playwright test runner is installed and executable in the
   target project (e.g. `npx playwright --version` succeeds).
3. **Browsers** — at least **one** browser can actually be launched. Two or more
   is preferred, not required: restricted corporate environments frequently
   permit only one (e.g. a manually installed MS Edge referenced by
   `executablePath`). One browser passes; **zero** fails. If only one is
   available, record the reduced matrix here so Skill 09 doesn't rediscover it
   and the Execution Report reports it honestly.
   **Never attempt to install a browser** — no `npx playwright install`, no
   download. Browser provisioning is the user's, and may be blocked by policy.
4. **Application + Login Recipe** — the URL from Intake is reachable, and the
   Intake credentials authenticate **by following the Login Recipe exactly**,
   including every dropdown, checkbox, and intermediate screen the user
   described. This is the cheapest possible place to discover the Recipe is
   incomplete — fix it here, with the user, before anything is built on it.

---

# PHASE B — PROJECT READ (only once Phase A passes)

## B1 — Readiness checks that require reading the project

5. **Project compiles** — the target project builds/type-checks in its current
   state, *before* anything is generated into it.
6. **Base framework present** — the project contains the user's hand-written base
   framework: a base page class, and the `utils/`/`fixtures/`/`hooks/` layers it
   uses. **This pipeline generates tests and Page Objects on top of a framework;
   it does not supply one.** If there is no base class to extend and no
   authentication mechanism to consume, stop and say so — do not generate Page
   Objects that extend nothing.

## B2 — Target Project Profile
Record, and carry into `execution-state.md`:
- Test directory and source layout the project actually uses (e.g. `src/`-based
  vs. root-based), read from `playwright.config.ts` (`testDir`) and
  `tsconfig.json` (`rootDir`/`paths`).
- **Existing directories and their exact spelling** — if the project has
  `testData/`, `hooks/`, `fixtures/`, those are the destinations, spelled that
  way. Never create a same-purpose sibling under the default name.
- Any path alias or config import the existing setup expects.
- Installed browsers, and any already-configured `projects`/`workers` —
  including `executablePath`/`channel` values, which are read and preserved
  exactly, never rewritten (see skill-09's Never Clobber rule).
- Runner version and how the suite is invoked.

**Skills 04, 06, and 07 generate into this profile's layout**, per
.claude/skills/knowledge/output-structure.md's Target Layout Alignment section.

## B3 — What This Skill Reads for the Inventory
Read every file in the base framework, in full. Not a directory listing, not
filenames — **the actual code**, including method bodies. A signature alone does
not tell a later Skill whether `navigateTo()` waits for network idle, whether
`clickElement()` scrolls first, or whether the auth fixture already selects a
user group.

Typical locations, resolved through the Target Project Profile's actual layout
and spelling:

| Location | What to catalogue |
|---|---|
| `pages/BasePage.ts` (or the project's base class, whatever it is named) | Every method: signature, parameters, return type, what it does, and whether it is `public`/`protected`/`private` |
| `utils/` | Every exported helper/class and its purpose |
| `fixtures/` | Every fixture, its name as consumed in a spec, what it provides, and its setup/teardown behaviour |
| `hooks/` | Every exported `beforeEach`/`afterEach` routine and what it does |
| `pages/` (any existing Page Objects) | Existing page classes and their public methods |
| `test-data/` | Existing data constants and factories |
| `playwright.config.ts`, `tsconfig.json` | Path aliases and import conventions generated code must follow |

**Find the base class by reading, not by assuming the name.** It may be
`BasePage.ts`, `BaseClass.ts`, `CommonPage.ts`, or something else entirely.
Whatever the project calls it, that is the class generated Page Objects extend.

## Expected Output — the Reuse Inventory
A catalogue precise enough that a later Skill can decide "does this already
exist?" **without re-reading the framework**, and call it correctly on the first
attempt. For every reusable asset record:

- **Where it lives** — file path, and the import path/alias generated code must use
- **Its exact signature** — name, parameters with types, return type, `async` or not
- **What it does** — one line, behavioural, including anything non-obvious
  (already waits, already scrolls, already handles the iframe, throws vs. returns null)
- **How it is meant to be called** — especially fixtures, which are consumed by
  name in the test signature rather than imported

A vague inventory is worse than none: "BasePage has navigation helpers" gives a
later Skill no way to decide whether to call one, so it writes its own. Record
`async navigateTo(path: string): Promise<void>` and what it waits for.

Also record, explicitly:
- **The authentication mechanism** — the fixture or hook that logs in, its name,
  and how a spec consumes it. Every generated spec needing a session uses this.
- **Gaps, and anything added to fill them** — see Gap Handling below.
- **Anything unusual** — a non-standard base class name, an unexpected inheritance
  chain, a fixture with surprising teardown, a helper that looks reusable but is
  page-specific.

## Verify the Login Recipe against the existing auth setup
The Intake Gate captured the Login Recipe: the ordered login steps including
every dropdown, checkbox, radio button, intermediate screen, and MFA handling.

The user's framework probably already implements login. **Compare the two**, and
record the result:
- **They match** → record the fixture as the authentication mechanism. Generated
  specs consume it. Nothing else to do.
- **They differ** → report the discrepancy to the Agent and stop. Do not silently
  rewrite the user's login code to match the Recipe, and do not silently generate
  a second login path that bypasses it. Either the Recipe is stale or the fixture
  is — the user decides which.
- **No authentication exists in the framework** → that is a Gap. Handle it per
  Gap Handling below.

## Gap Handling — adding what the application needs
The framework has, say, ten utility methods. The application turns out to need an
eleventh. **Add it, in the same folder, in the same style, and use it.** That is
normal, expected work — not an exception requiring ceremony. The same applies to
a new hook, a new fixture, or a new method on the base class.

The line that matters is not *new vs. none*. It is:

> **Adding something new is routine. Changing, renaming, restructuring, or
> overwriting something that already exists is not.**

The user's existing methods are working code that other tests already depend on.
Adding an eleventh method beside them breaks nothing. Rewriting the third one
because the generated code would prefer a different signature breaks whatever was
already calling it.

**Search before you conclude anything is missing.** Most apparent gaps are the
inventory being incomplete — a helper under a name that was not expected, a
fixture in a file that was not read. Re-read first. "I did not find it" and "it is
not there" are different claims, and a duplicate written on a false gap is the
exact failure RS-01 exists to prevent.

When something genuinely isn't there:

1. **Add it where its siblings live** — a helper in `utils/` next to the other
   helpers, a hook in `hooks/`, a fixture in `fixtures/`, a shared page behaviour
   as a method on the base class. Never a parallel folder, never a private copy
   inside a Page Object or spec.
2. **Match the surrounding style** — their naming, their typing conventions,
   their import style, their file organization. The addition should be
   indistinguishable from what the user would have written.
3. **Add the minimum the application requires.** A gap is not licence to
   refactor, restructure, or "improve" the code around it.
4. **Record it in the Reuse Inventory**, marked as added by this run, so later
   Skills find it under RS-01 and the user can see exactly what was added.

**Never modify existing working code to accommodate generated code.** If a
generated Page Object does not fit the base class, the Page Object is wrong —
extend the base class with something new, or adapt the Page Object; do not
rewrite the method that already exists. The one exception is a genuine defect in
the base framework surfaced by execution — handled at Skill 09, and always
reported explicitly, because that is a change to code the user owns.

## RS — Reuse-First Rules (Critical, binding on every later Skill)
These are what make the inventory worth producing. They apply to Skills 03–09.

- **RS-01 — Search the inventory before writing anything.** Before generating any
  method, helper, fixture, or hook, check the Reuse Inventory. If something
  already does the job, **call it.** Do not write a second one. This is not a
  preference — the existing code is the user's, it is already working, and a
  duplicate silently competes with it.
- **RS-02 — Use it as it is; do not fork it.** If an existing function nearly
  fits, call it and handle the difference at the call site. Copying it into a
  near-identical private variant is forbidden. If it genuinely cannot be adapted,
  add a *new* sibling method rather than editing the existing one.
- **RS-03 — New shared logic goes into the user's existing structure.** When
  something is genuinely needed, add it to their `utils/`/`fixtures/`/`hooks/`/
  base class, in their style, and record it in the inventory. Never inline in a
  spec (RL-01), never in a new parallel folder. **Adding is fine; rewriting what
  is already there is not.**
- **RS-04 — Every generated Page Object extends the project's base class**, by
  whatever name it has, and uses its methods rather than reimplementing
  navigation, waiting, or interaction.
- **RS-05 — Every generated spec consumes the project's authentication
  mechanism.** No spec performs its own login. If the framework provides an
  authenticated fixture, that fixture is how specs get a session.

Skill 08 validates RS-01–RS-05 as duplicate detection; Skill 09 must obey them
during repair, when the temptation to paste a quick local helper is highest.

## What This Skill Does NOT Do
- **Does not generate** `BasePage`, utility helpers, fixtures, or hooks. Those are
  the user's, already written. The only exception is a confirmed, reported gap.
- **Does not refactor, reorganize, rename, or "clean up"** existing framework code.
- **Does not overwrite any existing file.**
- **Does not generate** `playwright.config.ts`, `package.json`, or `tsconfig.json`
  — .claude/skills/knowledge/output-structure.md's Output Restrictions apply in
  full.
- **Does not create empty directories speculatively.** If `pages/`, `tests/`, or
  `test-data/` do not exist, they are created by the Skill that first writes into
  them (04, 07, 06) — not here.
- **Does not generate Page Objects, specs, or test data.** Those need Skill 02's
  Page Inventory and Skill 03's locators, neither of which exists yet.

## Workflow
**Phase A** — Read Intake Results → Check Test Case File → Check Runner → Check
Browsers → Authenticate Following the Login Recipe → ✋ stop on any failure

**Phase B** — Compile Check → Confirm a Base Framework Exists → Build the Target
Project Profile → **Read Every Framework File in Full, Including Method Bodies**
→ Identify the Base Class (by reading, not by name) → Catalogue the Base Class /
`utils/` / `fixtures/` / `hooks/` / Existing Page Objects / `test-data/` → Record
Import Paths and Aliases → Identify the Authentication Mechanism → Reconcile It
Against the Login Recipe → Add Anything Genuinely Missing, in the User's Style →
Record the Target Project Profile and Reuse Inventory

## Verify After Any Addition
If — and only if — this Skill added something to the framework, re-run the
type-check before reporting success. If it added nothing, check 5 already proved
the project compiles and there is nothing new to verify.

## Recording the Output
Write both artifacts into `execution-state.md`: the **Target Project Profile**
(from B2) and the **Reuse Inventory** (from B3). Skills 03–09 read the Inventory
as their RS-01 starting point — a missing entry is exactly how a duplicate gets
written later, so an incomplete inventory is a defect in this Skill, not a
harmless omission. Skills 04, 06, and 07 read the Profile to decide where
generated files go.

## Success Criteria
**Phase A passed** (Test Case file, runner, ≥1 browser, application reachable and
the Login Recipe authenticating) · **Phase B passed** (project compiles, base
framework present) · Target Project Profile recorded with the project's actual
layout and directory spellings · every file in the base framework read in full · base class correctly identified
by reading · every reusable asset catalogued with exact signature, behaviour, and
import path · authentication mechanism identified and reconciled against the
Login Recipe · anything genuinely missing added beside its siblings in the user's
own style and recorded · **nothing existing rewritten, overwritten, refactored,
renamed, or restructured** · Reuse Inventory recorded in `execution-state.md`.

## Failure Handling
**Stop and report — this Skill is a hard gate.** Any of these ends the run here,
before a single artifact is generated:
- Intake results missing or the Login Recipe incomplete
- **Phase A:** no Test Case file, no runner, zero launchable browsers, the
  application unreachable, or the Login Recipe failing to authenticate
- **Phase B:** the project does not compile, **no base framework exists** (this
  pipeline generates tests on top of a framework — it does not supply one), or
  the existing authentication mechanism contradicts the Login Recipe

Report exactly which check failed and the command or decision that fixes it.
**A missing runner, missing browsers, or a non-compiling project is a setup
problem the user resolves** — but it must surface here, at near-zero cost, not
after ten Skills of work.

If the user explicitly asks to generate anyway, knowing the suite cannot be
executed, that is permitted — but the delivery must be labelled **"Generated,
NOT Verified"**, and Skill 10 may never mark it Production Ready.

## Consumed By
Skill 01 — Test Case Analysis (Phase A proved the file exists) · Skill 02 —
Application Analysis (the verified Login Recipe) · Skill 04 — Page Object Generation (extends the project's base class, calls its
methods) · Skill 06 — Test Data Generation (reuses existing constants/factories)
· Skill 07 — Spec Generation (consumes the project's auth fixture and hooks) ·
Skill 08 — Framework Validation (validates RS-01–RS-05 as duplicate detection) ·
Skill 09 — Test Execution & Self-Healing (base-framework failures are systemic;
repairs to user-owned code are reported, never silent)
