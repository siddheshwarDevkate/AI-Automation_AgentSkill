# OUTPUT STRUCTURE

## Purpose
Defines the expected structure, organization, and quality of all generated Playwright framework artifacts, so every generated framework follows the same layout and formatting regardless of the AI model. Refer alongside .claude/skills/knowledge/framework-rules.md, .claude/skills/knowledge/framework-architecture.md, and .claude/skills/knowledge/naming-conventions.md.

## Dependency Matrix
Which Skills load this file is declared in .claude/agent/agent.md's Skill Dependency Matrix — that table is the single source of truth.

## Objective
Generate a clean, modular, production-ready framework that's immediately usable with minimal or no manual modification. Generate only the requested artifacts.

## Output Directory Structure
```
project-root/
├── .claude/     (or .opencode/ — the resource root: agent, skills, knowledge,
│                 templates, input/. INPUT ONLY — never a generation target)
├── pages/       (BasePage.ts, LoginPage.ts, DashboardPage.ts, ...)
├── tests/       (login.spec.ts, dashboard.spec.ts, ...)
├── utils/       (WaitHelper.ts, TestDataHelper.ts, ...)
├── fixtures/    (browser + authentication fixture from Skill 00; further test.extend() setup/teardown shared by 2+ specs)
├── hooks/       (shared beforeEach()/afterEach() routines reused by 2+ specs)
├── test-data/   (testData.ts)
├── traceability-report.md  (Test Case ID → Spec test coverage summary)
├── execution-report.md     (per-browser pass/fail/blocked status, repairs made)
├── execution-state.md      (running pipeline progress ledger, see .claude/agent/agent.md)
└── README.md
```
Only add folders when explicitly required.

**Never generate framework output into `.claude/` or `.opencode/`.** Those directories hold the Agent, Skills, Knowledge, Templates, and the user's Test Case input — they are read, never written to by generation. Every generated file above lands in the project root (or the Target Project Profile's equivalent).

## Who Creates What, and When
| Path | Created by | When |
|---|---|---|
| `pages/BasePage.ts` | Skill 00 | Scaffold, before Skill 01 |
| `fixtures/` + auth fixture | Skill 00 | Scaffold, from the Login Recipe |
| `hooks/`, `utils/` | Skill 00 (then extended by 04/07) | Scaffold |
| `pages/` (empty), `tests/` (empty), `test-data/` (empty) | Skill 00 | Scaffold |
| `pages/*Page.ts` | Skill 04 (+05 appends) | After the Page Inventory exists |
| `test-data/testData.ts` | Skill 06 | Before specs |
| `tests/*.spec.ts` | Skill 07 | Last generation step |

**Skill 00 creating `fixtures/` and `hooks/` with content in them is not "scaffolding empty folders."** RL-03 forbids empty speculative folders; the scaffold creates them because it has the authentication fixture and shared hooks to put in them right then. `pages/`, `tests/`, and `test-data/` *are* created empty on purpose — they are declared destinations for Skills 04, 07, and 06, and their emptiness after Skill 00 is the correct state.

Later Skills add to `utils/`, `fixtures/`, and `hooks/` when the application genuinely requires something new — under the Reuse-First Rules (RS-01–RS-05) in .claude/skills/skill-00-framework-scaffold.md, which require searching the scaffold and calling what is already there before adding anything.

## Target Layout Alignment (Critical)
**The structure above is the default, not an override.** When generating into an existing project, the Preflight Gate's Target Project Profile (see .claude/agent/agent.md) records the layout that project actually uses — and the generated files go *there*.

Concretely, before generating anything:
- Use the `testDir` from the project's existing `playwright.config.ts` as the spec destination. If it says `./src/tests`, specs go in `src/tests/` — not `tests/`.
- Respect `rootDir` and any `paths` aliases in `tsconfig.json`. If the project resolves imports through an alias (`@pages/*`), generated imports use it rather than fragile relative paths.
- Mirror the project's existing source convention (`src/`-based vs. root-based) for `pages/`, `utils/`, `fixtures/`, `hooks/`, and `test-data/`.
- Honour any config import the existing setup expects (e.g. a `config/env.config` module the project's `playwright.config.ts` imports). If that module is missing, report it at Preflight as a blocking setup gap — never generate around it and never leave the project in a state where its own config can't resolve.

**Generating the default layout into a project whose config points elsewhere produces a framework that cannot run** — the specs exist, but the runner never finds them, or the config fails to resolve and nothing compiles. That failure surfaces at Skill 09 as "the suite cannot be executed," which is the wrong diagnosis: the environment is fine, the files are simply in the wrong place. Skill 09 treats it as Root Cause 7 (build/compile defect) and realigns the paths — but aligning here, up front, avoids it entirely.

**Pre-created empty folders are a layout instruction.** A user who has already created `tests/`, `pages/`, `utils/`, `testData/`, `hooks/`, and `fixtures/` has told you where things go. Generate into exactly those directories.

**Match their spelling, don't correct it.** If the project has `testData/`, generate into `testData/` — do not create a second `test-data/` alongside it because that is what the default structure says, and do not rename their folder. The naming rules in .claude/skills/knowledge/naming-conventions.md govern names this framework *chooses*; an existing directory in the target project is not a name being chosen. Two folders serving the same purpose is a worse outcome than a folder whose name differs from the default. Record the mapping in the Target Project Profile so every Skill imports from the same path.

When no existing project layout is detectable, use the default structure above.

## File Generation Order
Scaffold (BasePage + auth fixture + hooks/utils, Skill 00) → Page Objects → Utilities → Test Data → Test Specifications. Never generate test files before their required Page Objects exist. Additional `fixtures/`/`hooks/` files beyond the scaffold are produced alongside Test Specifications, only at the point Spec Generation (Skill 07) identifies further logic that two or more spec files need to share — never generated speculatively ahead of that need.

## Page Object Output Order
Imports → Class Declaration → Private Locators → Constructor → Navigation Methods → Action Methods → Verification Methods → Compound Business Methods → Helper Methods (if required). Maintain this order consistently across every file.

## Test Specification Output Order
Imports → Test Data → `test.describe()` → `beforeEach()` → Positive Test Cases → Negative Test Cases → Edge Cases (if applicable). Business logic stays inside Page Objects.

## Test Case Traceability Tag
Every generated test must include its source Test Case ID tag — the exact format is owned by .claude/skills/knowledge/naming-conventions.md (NC-303).

## Utility Output
Utility classes (WaitHelper, TestDataHelper, DateHelper) contain only reusable functionality — no business logic.

## Test Data Output
Store reusable test data separately, e.g.:
```typescript
export const validUsername = "standard_user";
export const validPassword = "secret_sauce";
export const invalidPassword = "invalid_password";
```
Avoid scattering hardcoded values across multiple files.

## Code Organization
Maintain consistent formatting: separate imports, properties, constructor, and methods with blank lines. Avoid overly compact code.

## Import Order
External Packages → Framework Classes → Utilities → Project Files.
```typescript
import { Page, Locator, expect } from '@playwright/test';
import { BasePage } from './BasePage';
import { WaitHelper } from '../utils/WaitHelper';
```

## Method Organization (inside a Page Object)
Navigation → Input → Click → Selection → Verification → Compound Business → Private Helper methods. Keep this order consistent across every generated Page Object.

## Output Quality
Generated code should be readable, reusable, modular, maintainable, strongly typed, properly formatted, and easy to extend. Avoid unnecessary complexity.

## Output Restrictions
Do not generate `playwright.config.ts`, `package.json`, `tsconfig.json`, `node_modules`, `package-lock.json`, or CI/CD configuration unless explicitly requested. **Exception:** Skill 09 (Test Execution & Self-Healing) may generate/update only the `projects`, `workers`, `reporter`, and failure-artifact `use` keys of `playwright.config.ts`, per its own `playwright.config.ts` Exception section. Skill 00 does **not** share that exception — Preflight already proved the runner works and the project compiles, which means those files exist; the scaffold leaves them alone.

## Multiple Page Handling
One Page Object + one Spec file per logical page or feature. Avoid combining unrelated pages into a single file.

## Code Completeness
Every generated file should be complete — no incomplete methods, no placeholder implementations unless information is genuinely unavailable, in which case use `// TODO:` rather than fabricating.

## Validation Before Delivery
✓ Folder structure correct · ✓ required files generated · ✓ naming conventions followed · ✓ Page Objects complete · ✓ Specs complete · ✓ utilities reusable · ✓ no reusable/common function left inline inside a spec file · ✓ no duplicate code · ✓ framework rules followed · ✓ suite executed across the browser matrix at a 2-worker concurrency cap · ✓ output is production ready

## Delivery Format
Return files in logical order, each with file path, file name, and complete source code, clearly separated:
```
pages/LoginPage.ts
<Code>
------------------------------------------------
tests/login.spec.ts
<Code>
------------------------------------------------
utils/TestDataHelper.ts
<Code>
```
Maintain this structure for every generated response.
