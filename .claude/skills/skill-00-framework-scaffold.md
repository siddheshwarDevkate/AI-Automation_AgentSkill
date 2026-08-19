# SKILL 00 — FRAMEWORK SCAFFOLD

## Purpose
Build the framework's skeleton **once, up front**, before any test-case-specific
generation begins: `BasePage.ts`, the browser/authentication fixture, the shared
hooks, the utility layer, and the empty directories that Skills 04–07 fill in.

Every later Skill then *composes* against this skeleton instead of inventing its
own foundation. That is the whole point of running it first. When each Page
Object brings its own navigation helper and each spec writes its own login, the
result is a framework whose shared logic is scattered across twenty files and
whose failures are twenty separate bugs. Establishing one home for each concern
before generation starts makes reuse the path of least resistance rather than a
cleanup pass nobody runs.

## Execution Dependencies
Load the Knowledge files and Template listed for Skill 00 in
.claude/agent/agent.md's **Skill Dependency Matrix** — that table is the single
source of truth and is deliberately not restated here.

## Inputs
Required: Target Project Profile (from the Preflight Gate), Intake results
(application URL, credentials, **Login Recipe**).
Optional: an existing framework in the target project.

## Position in the Pipeline — and why it runs before Skill 01
This Skill runs immediately after the Preflight Gate, before Test Case Analysis.

It does not read the Test Case file and produces nothing derived from it: no
Page Objects, no specs, no test data, no locators, no assertions. It therefore
cannot violate traceability — there is nothing here to trace. The "no Test Case
file, no generation" rule is satisfied by Preflight check 1, which already proved
the file exists and parses. Running the scaffold first means Skills 04–07 have
somewhere to put things from their very first file, instead of retrofitting a
`BasePage` around Page Objects that were written without one.

## Expected Output — the Scaffold

Generated into the **Target Project Profile's layout and spelling**, never into
the default names when the project already uses others (see
.claude/skills/knowledge/output-structure.md's Target Layout Alignment).

| Path | Contents at scaffold time |
|---|---|
| `pages/BasePage.ts` | The base class — shared navigation, waiting, and interaction helpers. **Complete and working.** |
| `pages/` | Otherwise empty. Skill 04 fills it, one class per Page Inventory entry. |
| `tests/` | **Empty.** Skill 07 fills it, one spec file per module. |
| `test-data/` | **Empty.** Skill 06 generates `testData.ts` here. |
| `utils/` | Helpers the scaffold itself needs, and the declared home for generic helper logic. |
| `fixtures/` | The browser + authentication fixture (`test.extend()`), built from the Login Recipe. |
| `hooks/` | Shared `beforeEach`/`afterEach` routines the fixture doesn't naturally cover. |

`pages/`, `tests/`, and `test-data/` are created empty **on purpose** — they are
the declared destinations, and their emptiness at this stage is the correct
state, not an incomplete one.

## BasePage — what belongs in it
`BasePage` holds only logic that is genuinely common to every page:
- navigation (`navigateTo(path)`, `getCurrentUrl()`, waiting for a page to settle)
- generic element interaction wrappers every page needs
- generic waiting helpers built on Playwright's auto-waiting and web-first
  assertions — **never** `waitForTimeout()`
- the `protected page: Page` property, per POM-03 — `protected`, and it stays
  that way for the life of the framework

**No page-specific logic, no locators for a particular screen, no business
methods.** If it only makes sense for one page, it belongs on that page's class
(Skill 04), not here.

Keep it lean. A `BasePage` stuffed with speculative helpers "in case a page needs
them" is worse than a small one: every unused method is a method a later Skill
has to read past to decide whether something already exists. Generate what the
architecture requires; let Skills 04 and 09 promote genuinely shared logic up
into it when a second page proves the need.

## The Authentication Fixture — the Login Recipe's only home
The Intake Gate captured an ordered Login Recipe: the URL, the credentials, and
every dropdown, checkbox, radio button, intermediate screen, and MFA step
required to reach the authenticated landing page.

**Implement it exactly once, here, as a `fixtures/` fixture composed via
`test.extend()`.** Every spec that needs a logged-in session consumes that
fixture. No spec ever performs its own login, and no Page Object re-implements
one.

This is the single most valuable thing this Skill produces.
.claude/skills/knowledge/framework-architecture.md's Mandatory Extraction
Analysis calls authentication "the near-universal case" and Skill 08 flags inline
login as a violation — but both of those are *after-the-fact* checks that only
work if someone runs them. Building the fixture before any spec exists removes
the opportunity: there is nothing to extract later because it was never
duplicated in the first place.

Requirements:
- Follows the Login Recipe step by step, in the order the user gave. If the
  Recipe selects a user group from a dropdown before the password screen, the
  fixture does exactly that.
- Ends by confirming the success signal the user named (landing URL or visible
  element), so a broken login fails loudly at setup rather than as a confusing
  assertion failure inside an unrelated test.
- Reads credentials the way the target project already handles secrets —
  environment variables, an `.env`, or an existing config module. **Never
  hardcode a password into any generated file** (see the Intake Gate's
  credential-handling rules).
- Exposes a plain, well-named fixture (e.g. `authenticatedPage`) that a spec can
  consume without knowing any of the above.

If the Login Recipe was incomplete and Preflight check 4 could not authenticate,
this Skill does not guess at the missing steps — that failure was already a hard
stop at Preflight and is resolved with the user.

## RS — Reuse-First Rules (Critical, binding on every later Skill)
This Skill declares the scaffold; these rules are what make the rest of the
pipeline honour it. They apply to Skills 03–09, not just here.

- **RS-01 — Search before writing.** Before generating any method, helper,
  fixture, or hook, search the existing scaffold — `BasePage`, `utils/`,
  `fixtures/`, `hooks/`, and the Page Objects generated so far. If something
  already does the job, **call it**. Do not write a second one.
- **RS-02 — Extend, don't fork.** If an existing function *nearly* fits, extend
  it — a parameter, an overload, a widened type — rather than copying it into a
  near-identical twin. Two functions that differ by one line are one function
  with an argument.
- **RS-03 — New shared logic goes into the scaffold, not beside it.** When
  something genuinely new and reusable is needed, add it to the correct scaffold
  home (`utils/` / `fixtures/` / `hooks/` / `BasePage`) per
  .claude/skills/knowledge/framework-architecture.md's Reusable Logic Placement
  — so the *next* Skill finds it under RS-01 instead of writing a third copy.
  Never leave it inline in a spec (RL-01) and never create a parallel folder for
  it.
- **RS-04 — The scaffold is a floor, not a ceiling.** It is expected to grow.
  Adding a `DateHelper` to `utils/` because three test cases need date
  formatting, or a `seededRecord` fixture because a workflow requires one, is
  correct behaviour — the scaffold was never meant to anticipate every
  application. What is forbidden is growing it by duplication.
- **RS-05 — Modify the scaffold when the application requires it.** If the real
  application needs something the scaffold's shape doesn't accommodate — an iframe
  wrapper in `BasePage`, a tenant-switch step in the auth fixture — change the
  scaffold file. Do not work around it in a Page Object. A scaffold that gets
  bypassed is worse than no scaffold, because later Skills still trust it.

Skill 08 validates RS-01–RS-03 as duplicate detection; Skill 09 must obey them
during repair, when the temptation to paste a quick local helper is highest.

## Idempotency — never clobber what the project already has
The Target Project Profile records what already exists. Respect it:
- **A file that already exists is not regenerated.** An existing `BasePage.ts`,
  fixture, or helper is the project's, and this Skill reads and reuses it rather
  than overwriting it. Report what was reused.
- **A pre-created empty folder is a layout instruction**, including its exact
  spelling (`testData/` stays `testData/`). Generate into it; never create a
  same-purpose sibling under the default name.
- **Only fill genuine gaps.** If the project already has a working
  authentication fixture, this Skill adds nothing — it records that the scaffold
  requirement was already satisfied.

## Output Restrictions
This Skill does **not** generate `playwright.config.ts`, `package.json`,
`tsconfig.json`, or CI configuration —
.claude/skills/knowledge/output-structure.md's Output Restrictions apply here in
full, and Skill 09's narrow `playwright.config.ts` exception is Skill 09's alone.
Preflight already proved the runner is installed and the project compiles, which
means those files exist and work. Leave them alone.

It also generates no test, no Page Object beyond `BasePage`, and no test data.
Producing a scaffold and a `LoginPage.ts` in the same breath is out of scope —
Page Objects require Skill 02's Page Inventory and Skill 03's locators, neither
of which exists yet.

## Workflow
Read Target Project Profile → Read Intake Results (URL, credentials, Login
Recipe) → Inventory What Already Exists → Create Missing Directories (in the
project's own spelling) → Generate `BasePage.ts` → Generate the Browser +
Authentication Fixture from the Login Recipe → Generate `hooks/` and `utils/`
entries the scaffold itself requires → Verify It Compiles → Record the Scaffold
Manifest

## Compile Before Handing Over
Run the project's type-check (`npx tsc --noEmit`, or whatever the Target Project
Profile records) against the scaffold before this Skill reports success.

The scaffold is the foundation every later file imports. A type error in
`BasePage` or the auth fixture is not one broken file — it is every Page Object
and every spec failing at Skill 09 for a reason that has nothing to do with them.
Catching it here costs one command; catching it at Skill 09 costs a full suite
run and a misdiagnosis. This is the same reasoning that puts the Preflight Gate
before generation, applied to the first thing generation produces.

## Scaffold Manifest — this Skill's record
Record in `execution-state.md`:
- Every file created, and every existing file **reused instead of created**
- The resolved directory names actually used (default vs. project's own spelling)
- The authentication fixture's name and the Login Recipe steps it implements
- Compile check result
- Anything the Login Recipe required that could not be implemented, and why

Skills 03–09 read this manifest as their RS-01 starting point — it is the
"what already exists" list, so a missing entry here is how a duplicate gets
written later.

## Success Criteria
Scaffold directories exist in the project's own layout and spelling · `BasePage`
generated with shared logic only and `page` left `protected` · authentication
fixture implements the Login Recipe exactly, with no hardcoded credentials ·
`pages/`, `tests/`, and `test-data/` left empty for their owning Skills · nothing
existing was overwritten · scaffold type-checks clean · Scaffold Manifest
recorded.

## Failure Handling
**Stop and report** if: the Target Project Profile is missing (Preflight did not
run), the Intake results are missing or the Login Recipe is incomplete, the
scaffold cannot be written to the target layout, or the scaffold does not
compile after generation.

A scaffold that does not compile is never handed to Skill 01 — every subsequent
Skill would build on top of it, and the resulting failures would surface at Skill
09 pointing at the wrong files entirely.

## Consumed By
Skill 04 — Page Object Generation (`BasePage` is the parent class) · Skill 06 —
Test Data Generation (`test-data/` destination) · Skill 07 — Spec Generation
(fixtures, hooks, `tests/` destination) · Skill 08 — Framework Validation
(validates RS-01–RS-03 as duplicate detection) · Skill 09 — Test Execution &
Self-Healing (the scaffold is the shared-artifact set whose failures are
systemic)
