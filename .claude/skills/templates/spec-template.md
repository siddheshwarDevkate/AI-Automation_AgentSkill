# SPEC TEMPLATE (Test Specification Template)

## Purpose
The standard structure for generating Playwright test specification files, ensuring consistency, readability, and maintainability. Refer to .claude/skills/knowledge/framework-rules.md, .claude/skills/knowledge/naming-conventions.md, and .claude/skills/knowledge/generation-patterns.md alongside this file.

## Dependency Matrix
Template assignment is declared in .claude/agent/agent.md's Skill Dependency Matrix — that table is the single source of truth.

## Objective
Generate clean, readable, maintainable Playwright test specs — one per supplied Test Case, tagged with its Test Case ID. Tests validate business behaviour and never contain UI implementation details — all application interaction happens through Page Objects, and all data comes from `test-data/testData.ts`. This Skill wires together methods and constants that Skills 04–06 already created; it never defines new ones.

## Generation Workflow
Read Test Case → Identify Required Page Objects → Import Required Test Data Constants (from `testData.ts`) → Generate Test Setup → Generate the Test, Tagged with its Test Case ID → Validate Against `expectedResult` → Validate Generated Code

## File Structure (fixed order)
Imports → `test.describe()` → Page Object Declarations → `beforeEach()` → `afterEach()` (when the spec creates or mutates state) → Positive Test Cases → Negative Test Cases → Boundary/Edge Case Tests.

## Imports
Import only required files — Page Objects, and the specific test data constants this spec file needs from `test-data/testData.ts`:
```typescript
import { test } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';
import { DashboardPage } from '../pages/DashboardPage';
import { validUsername, validPassword, invalidPassword } from '../test-data/testData';
```

## Test Data
Never declare test data inline in the spec file. Import the constants Skill 06 (Test Data Generation) already produced in `test-data/testData.ts` — that file, not this one, is the source of truth for values:
```typescript
import { validUsername, validPassword, invalidPassword } from '../test-data/testData';
```
If a value a test case's steps need has no matching constant in `testData.ts`, that is a Skill 06 gap — document it in the coverage report. Do not hardcode a literal here to route around it.

## Page Object Declaration
Instantiate only the Page Objects required by the current feature:
```typescript
let loginPage: LoginPage;
let dashboardPage: DashboardPage;
```

## Test Setup (`beforeEach()`)
Responsibilities: construct Page Objects and navigate to the starting page. Never place assertions inside `beforeEach()`, and never place a login here.
```typescript
test.beforeEach(async ({ page }) => {
    loginPage = new LoginPage(page);
    dashboardPage = new DashboardPage(page);
    await loginPage.navigateTo();
});
```

**Authentication never lives here — the project already provides it.** The user's framework has a hand-written authentication fixture; Skill 00 recorded its name in the **Reuse Inventory**. Consume it (RS-05). There is nothing to "extract later," because it was never yours to duplicate — and a second login path drifts from the one the user maintains the first time they change it.

Wrong — an inline login, whether copied across specs or written into just one:
```typescript
// login.spec.ts, reports.spec.ts, settings.spec.ts ... all repeating this
test.beforeEach(async ({ page }) => {
    const loginPage = new LoginPage(page);
    await loginPage.navigateTo();
    await loginPage.performLogin(validUsername, validPassword);
});
```

Right — consume the project's existing fixture. Its real name and import path come from the Reuse Inventory; the listing below shows the shape of such a fixture so the consuming spec makes sense, **not a file to generate**:
```typescript
// fixtures/auth.fixture.ts — USER-OWNED, shown for reference
import { test as base } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';
import { validUsername, validPassword } from '../test-data/testData';

export const test = base.extend<{ authenticatedPage: void }>({
    authenticatedPage: [async ({ page }, use) => {
        const loginPage = new LoginPage(page);
        await loginPage.navigateTo();
        await loginPage.performLogin(validUsername, validPassword);
        await use();
    }, { auto: true }],
});

export { expect } from '@playwright/test';
```
```typescript
// reports.spec.ts — imports test from the fixture, not from @playwright/test
import { test } from '../fixtures/auth.fixture';
import { ReportsPage } from '../pages/ReportsPage';
```
For shared setup that isn't naturally a fixture (a common cleanup step, a shared navigation call), export a function from `hooks/` and call it from each spec's `beforeEach()` instead.

## Test Organization
Group related scenarios by module: `test.describe('Login', () => {...})`. Within a feature group, order as Positive Scenarios → Negative Scenarios → Edge Cases. Avoid mixing unrelated features in one file — one Spec file per module (module and Page Inventory page are independent groupings, see .claude/skills/knowledge/framework-architecture.md's Module vs Page distinction).

## Test Case Driven Generation
Do not discover scenarios freely. Each test case's `type` field (or, if absent, an inference documented alongside the generated tests) determines whether it's a positive, negative, or edge-case test — generate exactly what the test case describes, nothing more.

## Positive Test Cases
Successful user flows (successful login, search, record creation, logout) — each test validates one business outcome, as described by its test case's `expectedResult`.

## Negative Test Cases
Validation scenarios: invalid username/password, empty required fields, invalid input format, unauthorized access. Verify the error behaviour described by the test case's `expectedResult`.

## Edge Cases
Boundary scenarios: maximum/minimum length input, special characters, empty results, duplicate data. Generate only for test cases whose `type`/`expectedResult` actually describe a boundary condition.

## Assertions
Verify business behaviour through Page Object methods (finalized by Skill 05), not direct locator assertions. Correct:
```typescript
await loginPage.verifyErrorMessageVisible('Invalid username or password');
```
Incorrect:
```typescript
await expect(loginPage.errorMessageLocator).toBeVisible();
```
If the verification method you need doesn't exist yet, that is a Skill 05 gap — document it, do not write an inline assertion here to route around it.

**Per-Step Assertions:** When a test case defines an expected result for more than one of its steps (per .claude/skills/knowledge/test-case-parsing-rules.md's Expected Result Parsing), call the matching verification method right after the step it validates — do not perform every action first and assert everything at the end. A single test case still yields a single `test()`; that method simply contains one action-then-verify pair per checkpoint the test case defines, e.g.:
```typescript
test('[TC-021] Registration form validates each field as it is completed', async () => {
    await registerPage.fillEmail(validEmail);
    await registerPage.verifyEmailAccepted();

    await registerPage.fillPassword(weakPassword);
    await registerPage.verifyPasswordStrengthWarning();

    await registerPage.clickSubmit();
    await registerPage.verifyRegistrationBlocked();
});
```

## Test Independence
Each test must execute independently, not depend on another test, create its own required state, and clean up if necessary.

## Teardown (`afterEach()`)
Any test that creates or mutates application state must undo it, per .claude/skills/knowledge/test-data-lifecycle.md. Teardown contains cleanup calls only — never assertions, and never a swallowed failure:
```typescript
test.afterEach(async () => {
    await customerPage.deleteCustomerIfPresent(createdCustomerName);
});
```
Use the uniqueness factory rather than a shared constant, so repeated runs and parallel workers never collide:
```typescript
test('[TC-031] New customer is created successfully', async () => {
    createdCustomerName = buildUniqueCustomerName();

    await customerPage.createCustomer(createdCustomerName);
    await customerPage.verifyRecordCreated(createdCustomerName);
});
```
Only data this suite created may be cleaned up — never delete pre-existing application data to force a clean starting state. When two or more specs need the same teardown, extract it to `hooks/` instead of repeating it.

## Test Naming
Describe blocks represent the feature/module; test names describe business behaviour. Good: "Valid user can login successfully", "Invalid password displays error message". Avoid: "Test1", "Verify Login", "Sample Test".

## Test Case Traceability Tag
Every generated test must be prefixed with its source Test Case ID, per .claude/skills/knowledge/naming-conventions.md's NC-303:
```typescript
test('[TC-014] Valid login redirects user to dashboard', async () => {
    await loginPage.performLogin(validUsername, validPassword);
    await dashboardPage.verifyDashboardVisible();
});
```
Never generate a test without this tag, and never invent an ID that doesn't exist in the parsed Test Case Model — per .claude/skills/knowledge/test-case-parsing-rules.md.

## Complete Worked Example
Everything above assembled into one file, in the fixed File Structure order:

```typescript
import { test } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';
import { DashboardPage } from '../pages/DashboardPage';
import { validUsername, validPassword, invalidPassword } from '../test-data/testData';

test.describe('Login', () => {
    let loginPage: LoginPage;
    let dashboardPage: DashboardPage;

    test.beforeEach(async ({ page }) => {
        loginPage = new LoginPage(page);
        dashboardPage = new DashboardPage(page);
        await loginPage.navigateTo();
    });

    test('[TC-014] Valid login redirects user to dashboard', async () => {
        await loginPage.performLogin(validUsername, validPassword);
        await dashboardPage.verifyDashboardVisible();
    });

    test('[TC-015] Invalid password displays error message', async () => {
        await loginPage.performLogin(validUsername, invalidPassword);
        await loginPage.verifyErrorMessageVisible('Invalid username or password');
    });
});
```

## Code Quality
Generated test files should be readable, modular, independent, reuse Page Object methods, avoid duplicate logic and unnecessary assertions, and follow Arrange → Act → Assert.

## Do Not Generate
Access to private locators or the protected `page` property (`somePage.page.goto(...)`, `somePage.page.waitForLoadState(...)` — both compile errors and POM-03 violations; call a Page Object method instead), any raw `page.*` call, `expect()` on a locator, duplicated Page Object logic, inline test data literals, `page.waitForTimeout()`, hardcoded unnecessary waits, commented-out code, unused variables, or a reusable/common function defined inline in the spec file (data generators, custom wait conditions, retry wrappers, shared setup/teardown logic used by more than one test or spec) — extract those to `utils/`, `fixtures/`, or `hooks/` per .claude/skills/knowledge/framework-architecture.md's Reusable Logic Placement section instead. A one-off helper used by only this spec, with no expectation of reuse, may still stay local.

## Validation Checklist
✓ Correct imports, including test data imported from `testData.ts` (never inline) · ✓ proper Page Object usage · ✓ one test per Test Case · ✓ every test tagged with its Test Case ID · ✓ no scenario generated outside the Test Case Model · ✓ no duplicate logic · ✓ no reusable/common function defined inline in the spec file · ✓ no direct locator access · ✓ no inline assertions bypassing a missing verification method · ✓ every step-level expected result is asserted right after its step, not deferred to the end · ✓ state-creating tests use a uniqueness factory, not a fixed literal · ✓ state-creating/mutating tests have teardown, containing no assertions · ✓ setup shared with another spec (especially login) lives in `fixtures/`/`hooks/`, not copied into each `beforeEach` · ✓ tests are independent · ✓ follows all project standards
