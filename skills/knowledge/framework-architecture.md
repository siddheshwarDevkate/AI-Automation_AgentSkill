# FRAMEWORK ARCHITECTURE

## Purpose
Defines the target Playwright framework architecture: folder structure, responsibilities, dependencies, and design principles. Coding/implementation standards live in skills/knowledge/framework-rules.md — this file governs structure only.

## Used By Skills
01 Test Case Analysis · 02 Application Analysis · 08 Framework Validation · 10 Framework Review

## Objective
Generate a clean, modular, scalable, production-ready Playwright automation framework based on the Page Object Model (POM).

## Architecture Principles
1. **Single Responsibility** — each file has one responsibility.
2. **Separation of Concerns** — never mix Page Objects, Test Specifications, Utilities, Test Data, and Helpers.
3. **Reusability** — implement common logic once; avoid duplication.
4. **Maintainability** — support future enhancements with minimal changes.
5. **Readability** — prefer meaningful method names over shorter code.

## Target Folder Structure
```
project-root/
├── pages/
│   ├── BasePage.ts
│   ├── LoginPage.ts
│   ├── DashboardPage.ts
│   └── ...
├── tests/
│   ├── login.spec.ts
│   ├── dashboard.spec.ts
│   └── ...
├── utils/
│   ├── WaitHelper.ts
│   ├── TestDataHelper.ts
│   └── ...
├── test-data/
│   └── testData.ts
└── README.md
```
Generate additional folders only if explicitly requested.

## Page Object Architecture
Every application page gets its own Page Object (e.g. `LoginPage.ts`, `DashboardPage.ts`, `ProfilePage.ts`, `SettingsPage.ts`). Never combine multiple pages into one class.

## Page Identity Test (Critical)
A "page" is a distinct, navigable screen in the application — normally identified by its own URL/route, or reached as a full-screen destination via navigation. It is NOT a UI feature, widget, or pattern that lives inside a screen.

Skill 02 (Application Analysis) produces a **Page Inventory** — the definitive, numbered list of actual pages the Test Case Model touches. This inventory is the sole source of truth for how many Page Objects get generated. **The number of generated Page Objects must equal the number of entries in the Page Inventory — never more, never fewer.**

## Page vs Component — Do Not Confuse (Critical)
UI patterns and features found ON a page — a data table, a column-selection dropdown, a free-text search box, a filter panel, a modal, pagination — are NOT pages. They are behaviour belonging to the Page Object of the page they appear on, expressed as methods on that class. They must never become a separate Page Object of their own, no matter how complex or independent they look.

**Forbidden anti-pattern (this has actually happened — do not repeat it):** a single "Reports" page containing a data table, a column-selection dropdown, and a free-text search box must NOT produce `DataTablePage.ts`, `ColumnSelectionPage.ts`, and `FreeTextPage.ts`. It must produce exactly one `ReportsPage.ts` with methods like `selectColumns()`, `performFreeTextSearch()`, and `verifyTableData()`. If 20 test cases exercise 4 real pages, the output is 4 Page Objects — never one per detected feature or pattern.

**The only exception — genuinely shared components.** If the exact same widget (e.g. a shared header, sidebar, or modal) appears identically across two or more distinct pages in the Page Inventory, it may be extracted into its own reusable component class, composed into every Page Object that uses it. This class is never counted against the Page Inventory and must never be named or treated as a Page Object (see Naming Conventions — it does not get a `*Page.ts` name).

## Module vs Page — Two Different, Independent Groupings (Critical)
A Test Case's `module` field (e.g. "Login", "Reports") and a Page Inventory `page` entry are **not the same thing** and must never be assumed to line up 1:1:
- `module` drives **Spec file** boundaries (Skill 07) — one `.spec.ts` file per module.
- Page Inventory `page` drives **Page Object** boundaries (Skill 04) — one `*Page.ts` file per real page.

One module can span multiple pages (e.g. a "Reports" module might involve a Report List page and a Report Detail page — two Page Objects, one spec file). One page can serve multiple modules (e.g. a shared Search page might be exercised by both a "Search" module's test cases and a "Filter" module's). Grouping by `module` when generating Page Objects (instead of by the Page Inventory) is a common cause of the Page vs Component confusion above — always use the Page Inventory for Page Object boundaries, and `module` only for Spec file boundaries.

## BasePage
All Page Objects inherit from `BasePage`, which contains only reusable functionality: navigation, generic waits, common helper methods. No page-specific logic belongs in BasePage.

## Page Object Responsibilities
Each Page Object contains: private locators, public action methods, public verification methods, compound business actions. Example: `fillUsername()`, `fillPassword()`, `clickLoginButton()`, `performLogin()`, `verifyLoginPageVisible()`, `verifyCurrentUrl()`. Implementation rules live in skills/knowledge/framework-rules.md.

## Test Architecture
Tests validate business behaviour, not UI implementation. Business logic belongs inside Page Objects, not test files. How a test body is internally structured (Arrange → Act → Assert, independence) is owned by skills/knowledge/playwright-best-practices.md's Test Structure section.

## Test Organization
Organize by feature: `tests/login.spec.ts`, `dashboard.spec.ts`, `search.spec.ts`, `profile.spec.ts`. Avoid placing unrelated scenarios in the same file.

## Utilities
Utility classes (e.g. `WaitHelper`, `TestDataHelper`) contain only reusable helper logic — never business logic.

## Test Data
Store reusable test data separately; avoid hardcoding values throughout the framework.

## Dependency Flow
```
Tests → Page Objects → BasePage → Playwright
```
Utilities may be used by both Tests and Page Objects. Never create circular dependencies.

## Code Reuse
Before generating new methods, check whether an existing one can be reused or extended instead of duplicated.

## Scalability
The framework should support additional pages, test suites, utility classes, multiple environments, and parallel execution without architectural changes.

## Do Not Generate
Which files may or may not be generated (config files, CI/CD, the Skill 09 `playwright.config.ts` exception) is owned entirely by skills/knowledge/output-structure.md's Output Restrictions section — follow it exactly. Generate only the requested framework files.

## Execution Layer
The framework is not considered complete once code is generated — Skill 09 executes the full suite across the browser matrix and repairs framework-side failures before Skill 10 makes the final production-readiness call. See skills/knowledge/framework-rules.md's Execution & Self-Healing Rules and skills/skill-09-test-execution-self-healing.md for the full workflow.

## Validation Checklist
✓ Folder structure correct · ✓ every page has its own Page Object · ✓ Page Object count equals the Page Inventory count (no one-per-component/pattern Page Objects) · ✓ BasePage used correctly · ✓ tests contain business scenarios · ✓ utilities contain only reusable logic · ✓ no duplicate responsibilities · ✓ architecture follows POM · ✓ suite executed across the browser matrix before readiness is declared
