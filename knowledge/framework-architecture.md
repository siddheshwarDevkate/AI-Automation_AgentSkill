# FRAMEWORK ARCHITECTURE

## Purpose
Defines the target Playwright framework architecture: folder structure, responsibilities, dependencies, and design principles. Coding/implementation standards live in framework-rules.md — this file governs structure only.

## Used By Skills
01 Application Analysis · 07 Framework Validation · 08 Framework Review

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

## BasePage
All Page Objects inherit from `BasePage`, which contains only reusable functionality: navigation, generic waits, common helper methods. No page-specific logic belongs in BasePage.

## Page Object Responsibilities
Each Page Object contains: private locators, public action methods, public verification methods, compound business actions. Example: `fillUsername()`, `fillPassword()`, `clickLoginButton()`, `performLogin()`, `verifyLoginPageVisible()`, `verifyCurrentUrl()`. Implementation rules live in framework-rules.md.

## Test Architecture
Tests validate business behaviour, not UI implementation. Keep tests small and readable, following Arrange → Act → Assert. Business logic belongs inside Page Objects, not test files.

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
Unless explicitly requested: `playwright.config.ts`, `package.json`, `tsconfig.json`, CI/CD configuration, Docker files. Generate only the requested framework files.

## Validation Checklist
✓ Folder structure correct · ✓ every page has its own Page Object · ✓ BasePage used correctly · ✓ tests contain business scenarios · ✓ utilities contain only reusable logic · ✓ no duplicate responsibilities · ✓ architecture follows POM
