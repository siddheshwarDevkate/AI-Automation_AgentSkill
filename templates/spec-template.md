# SPEC TEMPLATE (Test Specification Template)

## Purpose
The standard structure for generating Playwright test specification files, ensuring consistency, readability, and maintainability. Refer to framework-rules.md, naming-conventions.md, and generation-patterns.md alongside this file.

## Used By Skills
04 Spec Generation

## Objective
Generate clean, readable, maintainable Playwright test specs. Tests validate business behaviour and never contain UI implementation details — all application interaction happens through Page Objects.

## Generation Workflow
Analyze Business Scenario → Identify Required Page Objects → Prepare Test Data → Generate Test Setup → Generate Positive Test Cases → Generate Negative Test Cases → Generate Edge Cases → Validate Test Coverage → Validate Generated Code

## File Structure (fixed order)
Imports → Constants/Test Data → `test.describe()` → Page Object Declarations → `beforeEach()` → Positive Test Cases → Negative Test Cases → Boundary/Edge Case Tests → Cleanup (if required).

## Imports
Import only required files:
```typescript
import { test } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';
import { DashboardPage } from '../pages/DashboardPage';
```

## Test Data
Keep reusable data together, avoid repeating hardcoded values:
```typescript
const validUsername = 'standard_user';
const validPassword = 'secret_sauce';
const invalidPassword = 'invalid_password';
```

## Page Object Declaration
Instantiate only the Page Objects required by the current feature:
```typescript
let loginPage: LoginPage;
let dashboardPage: DashboardPage;
```

## Test Setup (`beforeEach()`)
Responsibilities: launch application, navigate to starting page, create Page Objects, perform common initialization. Never place assertions inside `beforeEach()`.

## Test Organization
Group related scenarios: `test.describe('Login', () => {...})`. Within a feature group, order as Positive Scenarios → Negative Scenarios → Edge Cases. Avoid mixing unrelated features.

## Positive Test Cases
Successful user flows (successful login, search, record creation, logout) — each test validates one business outcome.

## Negative Test Cases
Validation scenarios: invalid username/password, empty required fields, invalid input format, unauthorized access. Verify expected error behaviour.

## Edge Cases
Boundary scenarios where applicable: maximum/minimum length input, special characters, empty results, duplicate data. Generate only when relevant.

## Assertions
Verify business behaviour through Page Object methods, not direct locator assertions. Correct:
```typescript
await loginPage.verifyErrorMessageVisible();
```
Incorrect:
```typescript
await expect(loginPage.errorMessageLocator).toBeVisible();
```

## Test Independence
Each test must execute independently, not depend on another test, create its own required state, and clean up if necessary.

## Test Naming
Describe blocks represent the feature; test names describe business behaviour. Good: "Valid user can login successfully", "Invalid password displays error message". Avoid: "Test1", "Verify Login", "Sample Test".

## Code Quality
Generated test files should be readable, modular, independent, reuse Page Object methods, avoid duplicate logic and unnecessary assertions, and follow Arrange → Act → Assert.

## Do Not Generate
Access to private locators or protected Page properties, duplicated Page Object logic, `page.waitForTimeout()`, hardcoded unnecessary waits, commented-out code, or unused variables.

## Validation Checklist
✓ Correct imports · ✓ proper Page Object usage · ✓ business-focused test cases · ✓ positive scenarios covered · ✓ negative scenarios covered · ✓ edge cases included where applicable · ✓ no duplicate logic · ✓ no direct locator access · ✓ tests are independent · ✓ follows all project standards
