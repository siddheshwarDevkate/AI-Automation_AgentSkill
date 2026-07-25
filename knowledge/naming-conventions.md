# NAMING CONVENTIONS

## Purpose
Consistent naming conventions for all generated Playwright framework artifacts, to improve readability, maintainability, and predictability.

## Used By Skills
02 Locator Generation · 03 Page Object Generation · 04 Spec Generation · 07 Framework Validation

## General Principles
- **NC-001 — PascalCase for Classes:** `LoginPage`, `DashboardPage`, `ProfilePage`, `BasePage`, `WaitHelper`, `TestDataHelper`. Not `loginPage`, `login_page`, `LOGINPAGE`.
- **NC-002 — camelCase for Variables/Methods:** `performLogin()`, `fillUsername()`, `verifyDashboardVisible()`, `loginButtonLocator`, `validUsername`. Not `PerformLogin()`, `FillUsername()`, `LOGINBUTTON`.
- **NC-003 — UPPER_CASE for Constants (optional):** only for shared global constants, e.g. `DEFAULT_TIMEOUT`, `BASE_URL`. For local test data prefer `const validUsername = 'standard_user';`.

## File Naming
- **NC-101 Page Object Files:** `<PageName>Page.ts` — e.g. `LoginPage.ts`, `DashboardPage.ts`, `SearchPage.ts`, `ProfilePage.ts`.
- **NC-102 Test Files:** `<feature>.spec.ts` — e.g. `login.spec.ts`, `dashboard.spec.ts`, `search.spec.ts`, `profile.spec.ts`.
- **NC-103 Utility Files:** `<Feature>Helper.ts` — e.g. `WaitHelper.ts`, `TestDataHelper.ts`, `DateHelper.ts`.

## Page Object Naming
- **NC-201 Locator Variables:** `<elementName>Locator` — e.g. `usernameInputLocator`, `passwordInputLocator`, `loginButtonLocator`, `errorMessageLocator`.
- **NC-202 Input Methods:** `fill<Element>()` — e.g. `fillUsername()`, `fillPassword()`, `fillEmail()`.
- **NC-203 Click Methods:** `click<Element>()` — e.g. `clickLoginButton()`, `clickSaveButton()`, `clickLogoutButton()`.
- **NC-204 Select Methods:** `select<Element>()` — e.g. `selectCountry()`, `selectRole()`, `selectCategory()`.
- **NC-205 Verification Methods:** `verify<ExpectedBehavior>()` — e.g. `verifyDashboardVisible()`, `verifyLoginPageVisible()`, `verifyErrorMessageVisible()`, `verifyCurrentUrl()`, `verifyPageTitle()`.
- **NC-206 Getter Methods:** `get<Element>()` — e.g. `getUsername()`, `getPageTitle()`, `getErrorMessage()`.
- **NC-207 Compound Business Methods:** `perform<Action>()` — e.g. `performLogin()`, `performLogout()`, `performSearch()`, `performCheckout()`.

## Test Naming
- **NC-301 Describe Blocks:** represent the feature — `test.describe('Login', () => {...})`.
- **NC-302 Test Names:** describe business behavior. Good: "Valid login redirects user to dashboard", "Invalid password displays error message". Avoid: "Test1", "Login Test", "Verify Login".

## Variable Naming
- **NC-401 Test Data:** `validUsername`, `validPassword`, `invalidPassword`, `expectedMessage`.
- **NC-402 Page Object Instances:** `loginPage`, `dashboardPage`, `profilePage`.
- **NC-403 Booleans:** meaningful prefixes — `isLoggedIn`, `isVisible`, `hasPermission`, `canEdit`.

## Folder Naming
Lowercase — `pages`, `tests`, `utils`, `test-data`. Not `Pages`, `TestFiles`, `Utilities`.

## Consistency Rule
Use the same naming pattern throughout the framework. Correct: `LoginPage`, `DashboardPage`, `ProfilePage`. Incorrect (mixed styles): `LoginPage`, `dashboard_page`, `PROFILEPAGE`.

## Final Checklist
✓ Classes use PascalCase · ✓ methods/variables use camelCase · ✓ locators end with `Locator` · ✓ Page files end with `Page.ts` · ✓ test files end with `.spec.ts` · ✓ helper files end with `Helper.ts` · ✓ test names describe business behavior · ✓ folder names lowercase · ✓ naming consistent throughout
