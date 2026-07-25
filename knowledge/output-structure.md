# OUTPUT STRUCTURE

## Purpose
Defines the expected structure, organization, and quality of all generated Playwright framework artifacts, so every generated framework follows the same layout and formatting regardless of the AI model. Refer alongside framework-rules.md, framework-architecture.md, and naming-conventions.md.

## Used By Skills
01 Application Analysis · 03 Page Object Generation · 04 Spec Generation · 05 Assertion Generation · 06 Test Data Generation · 07 Framework Validation · 08 Framework Review

## Objective
Generate a clean, modular, production-ready framework that's immediately usable with minimal or no manual modification. Generate only the requested artifacts.

## Output Directory Structure
```
project-root/
├── pages/       (BasePage.ts, LoginPage.ts, DashboardPage.ts, ...)
├── tests/       (login.spec.ts, dashboard.spec.ts, ...)
├── utils/       (WaitHelper.ts, TestDataHelper.ts, ...)
├── test-data/   (testData.ts)
└── README.md
```
Only add folders when explicitly required.

## File Generation Order
BasePage → Page Objects → Utilities → Test Data → Test Specifications. Never generate test files before their required Page Objects exist.

## Page Object Output Order
Imports → Class Declaration → Private Locators → Constructor → Navigation Methods → Action Methods → Verification Methods → Compound Business Methods → Helper Methods (if required). Maintain this order consistently across every file.

## Test Specification Output Order
Imports → Test Data → `test.describe()` → `beforeEach()` → Positive Test Cases → Negative Test Cases → Edge Cases (if applicable). Business logic stays inside Page Objects.

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
Do not generate `playwright.config.ts`, `package.json`, `tsconfig.json`, `node_modules`, `package-lock.json`, or CI/CD configuration unless explicitly requested.

## Multiple Page Handling
One Page Object + one Spec file per logical page or feature. Avoid combining unrelated pages into a single file.

## Code Completeness
Every generated file should be complete — no incomplete methods, no placeholder implementations unless information is genuinely unavailable, in which case use `// TODO:` rather than fabricating.

## Validation Before Delivery
✓ Folder structure correct · ✓ required files generated · ✓ naming conventions followed · ✓ Page Objects complete · ✓ Specs complete · ✓ utilities reusable · ✓ no duplicate code · ✓ framework rules followed · ✓ output is production ready

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
