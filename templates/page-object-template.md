# PAGE OBJECT TEMPLATE

## Purpose
The standard structure for generating Playwright Page Objects, ensuring consistency, readability, and maintainability. Refer to framework-rules.md, naming-conventions.md, typescript-coding-standards.md, and playwright-best-practices.md alongside this file.

## Used By Skills
03 Page Object Generation

## Objective
Generate one Page Object per application page. Each Page Object contains only logic related to that page — never combine multiple pages into a single class.

## Generation Workflow
Analyze Page → Identify Elements → Generate Locators → Generate Constructor → Generate Navigation Methods → Generate Action Methods → Generate Verification Methods → Generate Compound Business Methods → Validate Output

## File Structure (fixed order)
Imports → Class Declaration → Private Locator Variables → Constructor → Navigation Methods → Input Methods → Click Methods → Selection Methods → Verification Methods → Compound Business Methods → Private Helper Methods (optional). Never change this order.

## Imports
Generate only required imports, e.g.:
```typescript
import { Page, Locator, expect } from '@playwright/test';
import { BasePage } from './BasePage';
```

## Class Declaration
Every Page Object extends `BasePage`:
```typescript
export class LoginPage extends BasePage {
}
```

## Locators
Generate all locators before the constructor — `private`, `readonly`, meaningful names, following the stable locator strategy:
```typescript
private readonly usernameInputLocator: Locator;
private readonly passwordInputLocator: Locator;
private readonly loginButtonLocator: Locator;
```
Never expose locators publicly.

## Constructor
Responsibilities: call `super(page)`, initialize locators — nothing else. No business logic in the constructor.

## Navigation Methods
Generate when applicable, e.g. `navigateTo()`, `refreshPage()`.

## Action Methods
One method per action, each with a single responsibility — e.g. `fillUsername()`, `fillPassword()`, `clickLoginButton()`, `clickSearchButton()`.

## Verification Methods
Reusable methods, e.g. `verifyLoginPageVisible()`, `verifyDashboardVisible()`, `verifyCurrentUrl()`, `verifyPageTitle()`, `verifyErrorMessageVisible()`. Avoid placing assertions inside action methods.

## Compound Methods
Reusable business workflows, e.g. `performLogin()`, `performLogout()`, `performSearch()`, `performCheckout()`. Reuse action methods instead of duplicating logic.

## Private Helpers
Generate only when they improve readability or eliminate duplication — don't create unnecessary helper methods.

## Code Quality
Every generated Page Object must follow framework rules, locator strategy, naming conventions, and Playwright best practices; use explicit return types; use async/await correctly; avoid duplicate logic; be production ready.

## Validation Checklist
✓ One class per file · ✓ correct imports · ✓ private readonly locators · ✓ constructor initializes every locator · ✓ public action methods · ✓ public verification methods · ✓ compound business methods · ✓ no unused code · ✓ no fabricated locators · ✓ follows all project standards
