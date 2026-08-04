# OUTPUT STRUCTURE

## Purpose
Defines the expected structure, organization, and quality of all generated Playwright framework artifacts, so every generated framework follows the same layout and formatting regardless of the AI model. Refer alongside skills/knowledge/framework-rules.md, skills/knowledge/framework-architecture.md, and skills/knowledge/naming-conventions.md.

## Dependency Matrix
Which Skills load this file is declared in agent/agent.md's Skill Dependency Matrix — that table is the single source of truth.

## Objective
Generate a clean, modular, production-ready framework that's immediately usable with minimal or no manual modification. Generate only the requested artifacts.

## Output Directory Structure
```
project-root/
├── pages/       (BasePage.ts, LoginPage.ts, DashboardPage.ts, ...)
├── tests/       (login.spec.ts, dashboard.spec.ts, ...)
├── utils/       (WaitHelper.ts, TestDataHelper.ts, ...)
├── fixtures/    (test.extend() setup/teardown shared by 2+ specs — generated only when needed)
├── hooks/       (shared beforeEach()/afterEach() routines reused by 2+ specs — generated only when needed)
├── test-data/   (testData.ts)
├── traceability-report.md  (Test Case ID → Spec test coverage summary)
├── execution-report.md     (per-browser pass/fail/blocked status, repairs made)
├── execution-state.md      (running pipeline progress ledger, see agent/agent.md)
└── README.md
```
Only add folders when explicitly required. **Exception:** `fixtures/` and `hooks/` are added automatically, without a separate explicit request, the first time reusable setup/teardown logic needs extracting out of a spec file — see skills/knowledge/framework-architecture.md's Reusable Logic Placement section. Never scaffold either one empty.

## File Generation Order
BasePage → Page Objects → Utilities → Test Data → Test Specifications. Never generate test files before their required Page Objects exist. `fixtures/`/`hooks/` files are produced alongside Test Specifications, only at the point Spec Generation (Skill 07) identifies logic that two or more spec files need to share — never generated speculatively ahead of that need.

## Page Object Output Order
Imports → Class Declaration → Private Locators → Constructor → Navigation Methods → Action Methods → Verification Methods → Compound Business Methods → Helper Methods (if required). Maintain this order consistently across every file.

## Test Specification Output Order
Imports → Test Data → `test.describe()` → `beforeEach()` → Positive Test Cases → Negative Test Cases → Edge Cases (if applicable). Business logic stays inside Page Objects.

## Test Case Traceability Tag
Every generated test must include its source Test Case ID tag — the exact format is owned by skills/knowledge/naming-conventions.md (NC-303).

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
Do not generate `playwright.config.ts`, `package.json`, `tsconfig.json`, `node_modules`, `package-lock.json`, or CI/CD configuration unless explicitly requested. **Exception:** Skill 09 (Test Execution & Self-Healing) may generate/update only the `projects` array of `playwright.config.ts` to define the browser matrix.

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
