# NAMING CONVENTIONS

## Purpose
Consistent naming conventions for all generated Playwright framework artifacts, to improve readability, maintainability, and predictability.

## Dependency Matrix
Which Skills load this file is declared in agent/agent.md's Skill Dependency Matrix — that table is the single source of truth.

## General Principles
- **NC-001 — PascalCase for Classes:** `LoginPage`, `DashboardPage`, `ProfilePage`, `BasePage`, `WaitHelper`, `TestDataHelper`. Not `loginPage`, `login_page`, `LOGINPAGE`.
- **NC-002 — camelCase for Variables/Methods:** `performLogin()`, `fillUsername()`, `verifyDashboardVisible()`, `loginButtonLocator`, `validUsername`. Not `PerformLogin()`, `FillUsername()`, `LOGINBUTTON`.
- **NC-003 — UPPER_CASE for Constants (optional):** only for shared global constants, e.g. `DEFAULT_TIMEOUT`, `BASE_URL`. For local test data prefer `const validUsername = 'standard_user';`.

## File Naming
- **NC-101 Page Object Files:** `<PageName>Page.ts` — e.g. `LoginPage.ts`, `DashboardPage.ts`, `SearchPage.ts`, `ProfilePage.ts`. `<PageName>` must come from a Page Inventory entry (skills/knowledge/framework-architecture.md's Page Identity Test) — never from a UI pattern or feature name. `DataTablePage.ts`, `ColumnSelectionPage.ts`, and `FreeTextPage.ts` are not valid names; those belong on the Page Object of whatever real page they appear on.
- **NC-102 Test Files:** `<feature>.spec.ts` — e.g. `login.spec.ts`, `dashboard.spec.ts`, `search.spec.ts`, `profile.spec.ts`.
- **NC-103 Utility Files:** `<Feature>Helper.ts` — e.g. `WaitHelper.ts`, `TestDataHelper.ts`, `DateHelper.ts`.
- **NC-104 Fixture Files:** `<name>.fixture.ts` — e.g. `auth.fixture.ts`, `apiClient.fixture.ts`. Export shared setup/teardown via `test.extend()`; see skills/knowledge/framework-architecture.md's Reusable Logic Placement section.
- **NC-105 Hook Files:** `<name>Hooks.ts` — e.g. `loginHooks.ts`, `cleanupHooks.ts`. Export plain functions called from a spec's `beforeEach()`/`afterEach()`.

## Page Object Naming
- **NC-201 Locator Variables:** `<elementName>Locator` — e.g. `usernameInputLocator`, `passwordInputLocator`, `loginButtonLocator`, `errorMessageLocator`.
- **NC-202 Input Methods:** `fill<Element>()` — e.g. `fillUsername()`, `fillPassword()`, `fillEmail()`.
- **NC-203 Click Methods:** `click<Element>()` — e.g. `clickLoginButton()`, `clickSaveButton()`, `clickLogoutButton()`.
- **NC-204 Select Methods:** `select<Element>()` — e.g. `selectCountry()`, `selectRole()`, `selectCategory()`.
- **NC-205 Verification Methods:** `verify<ExpectedBehavior>()` — e.g. `verifyDashboardVisible()`, `verifyLoginPageVisible()`, `verifyErrorMessageVisible()`, `verifyCurrentUrl()`, `verifyPageTitle()`.
- **NC-206 Getter Methods:** `get<Element>()` — e.g. `getUsername()`, `getPageTitle()`, `getErrorMessage()`.
- **NC-207 Compound Business Methods:** `perform<Action>()` — e.g. `performLogin()`, `performLogout()`, `performSearch()`, `performCheckout()`.

## Test Naming
- **NC-301 Describe Blocks:** represent the feature/module from the Test Case Model — `test.describe('Login', () => {...})`.
- **NC-302 Test Names:** describe business behavior. Good: "Valid login redirects user to dashboard", "Invalid password displays error message". Avoid: "Test1", "Login Test", "Verify Login".
- **NC-303 Test Case Traceability Tag:** prefix every test name with its source Test Case ID in square brackets — e.g. `"[TC-014] Valid login redirects user to dashboard"`. Never omit it and never invent an ID not present in the parsed Test Case Model.

## Variable Naming
- **NC-401 Test Data:** `validUsername`, `validPassword`, `invalidPassword`, `expectedMessage`.
- **NC-402 Page Object Instances:** `loginPage`, `dashboardPage`, `profilePage`.
- **NC-403 Booleans:** meaningful prefixes — `isLoggedIn`, `isVisible`, `hasPermission`, `canEdit`.

## Folder Naming
Lowercase — `pages`, `tests`, `utils`, `fixtures`, `hooks`, `test-data`. Not `Pages`, `TestFiles`, `Utilities`.

## Consistency Rule
Use the same naming pattern throughout the framework. Correct: `LoginPage`, `DashboardPage`, `ProfilePage`. Incorrect (mixed styles): `LoginPage`, `dashboard_page`, `PROFILEPAGE`.

## Final Checklist
✓ Classes use PascalCase · ✓ methods/variables use camelCase · ✓ locators end with `Locator` · ✓ Page files end with `Page.ts` · ✓ test files end with `.spec.ts` · ✓ helper files end with `Helper.ts` · ✓ test names describe business behavior · ✓ every test name carries its Test Case ID tag · ✓ folder names lowercase · ✓ naming consistent throughout
