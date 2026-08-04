# PLAYWRIGHT BEST PRACTICES

## Purpose
Playwright-specific best practices for generated automation code. Focuses only on Playwright APIs and recommended usage — refer alongside skills/knowledge/framework-rules.md, skills/knowledge/framework-architecture.md, and skills/knowledge/locator-strategy.md.

## Dependency Matrix
Which Skills load this file is declared in agent/agent.md's Skill Dependency Matrix — that table is the single source of truth.

## General Principles
- **BP-001 — Prefer Playwright Native APIs:** `expect()`, `locator()`, `getByRole()`, `getByLabel()`, `waitFor()`, `toHaveURL()`, `toHaveTitle()`.
- **BP-002 — Use Auto Waiting.** Playwright auto-waits for actionable elements — don't add unnecessary waits. Correct: `await loginButton.click();`. Incorrect: `await page.waitForTimeout(3000); await loginButton.click();`.
- **BP-003 — Keep Tests Deterministic.** Avoid depending on timing, execution order, existing app state, or random UI behavior.
- **BP-004 — Prefer `innerText()` over `textContent()`.** `textContent()` includes hidden text and whitespace noise; `innerText()` reflects rendered, visible text.

## Locators
- **BP-101 — Create Once, in the Constructor:**
```typescript
private readonly loginButtonLocator: Locator;
constructor(page: Page) {
    super(page);
    this.loginButtonLocator = page.getByRole('button', { name: 'Login' });
}
```
Don't recreate locators inside every method.
- **BP-102 — Reuse Existing Locators.** One element, one locator — avoid duplicate definitions.
- **BP-103 — Locator Selection.** Which locator strategy to use for a given element (priority order, validation, special cases) is owned by skills/knowledge/locator-strategy.md — follow it exactly rather than choosing an ad hoc order here.

## Actions
- **BP-201 — One Action Per Method:** `fillUsername()`, `fillPassword()`, `clickLoginButton()`. Don't combine unrelated actions.
- **BP-202 — Create Compound Methods** for frequently used business flows: `performLogin()`, `performLogout()`, `performSearch()`.
- **BP-203 — Use Meaningful Method Names.** Good: `clickLoginButton()`, `verifyDashboardVisible()`, `fillEmail()`. Bad: `click()`, `verify()`, `fill()`.

## Assertions
- **BP-301 — Use Playwright Assertions.** Correct: `await expect(locator).toBeVisible();`. Incorrect: `expect(await locator.isVisible()).toBe(true);`.
- **BP-302 — Verify Business Outcomes** (dashboard displayed, success message appears, user redirected, table updated) rather than unnecessary implementation details.
- **BP-303 — Keep Assertions Independent.** Each assertion validates one behavior.

## Waiting Strategy
- **BP-401 — Never Use Hard Waits** (`page.waitForTimeout()`).
- **BP-402 — Wait for Expected State:** `await expect(locator).toBeVisible();`, `toHaveText()`, `toHaveURL()`.
- **BP-403 — Wait Only When Needed.** Playwright already auto-waits; don't add waits after every action.
- **BP-404 — Fix Flakiness at the Root.** When Skill 09 (Test Execution & Self-Healing) diagnoses a flaky failure, fix the missing/incorrect wait per BP-402 — never "fix" flakiness by adding `page.waitForTimeout()` or by retrying blindly until green.

## Navigation
- **BP-501 — Verify Navigation** after important transitions: URL, page title, visible element.
- **BP-502 — Encapsulate Navigation** inside Page Objects, e.g. `await dashboardPage.navigateTo();`.

## Test Structure
- **BP-601 — Arrange → Act → Assert.**
- **BP-602 — Keep Tests Independent** — every test should pass on its own.
- **BP-603 — Group Related Tests:** `test.describe('Login', () => {...}); test.describe('Search', () => {...});`.

## Error Handling
- **BP-701 — Fail Fast.** Don't hide errors — let Playwright report failures naturally.
- **BP-702 — Don't Suppress Exceptions.** Avoid empty `try {} catch {}` blocks unless there's a valid business reason.

## Performance
- **BP-801 — Avoid Duplicate Operations** (repeated element lookups, unnecessary refreshes, redundant logins when reuse is possible).
- **BP-802 — Keep Tests Efficient** — avoid unnecessary clicks, navigation, assertions, waiting.

## Code Readability
- **BP-901 — Keep Methods Small**, one responsibility each.
- **BP-902 — Use Descriptive Names.** Good: `verifyErrorMessageVisible()`. Bad: `verify()`.
- **BP-903 — Remove Dead Code** — no unused variables/imports, commented code, or empty methods.

## Final Checklist
✓ Playwright APIs used correctly · ✓ no hard waits · ✓ proper assertions · ✓ stable waiting strategy · ✓ clean Page Object methods · ✓ reusable business methods · ✓ meaningful naming · ✓ efficient implementation · ✓ readable code · ✓ no unnecessary complexity · ✓ verified passing across the configured browser matrix
