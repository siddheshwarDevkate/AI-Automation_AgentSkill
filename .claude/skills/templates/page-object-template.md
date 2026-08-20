# PAGE OBJECT TEMPLATE

## Purpose
The standard structure for generating Playwright Page Objects, ensuring consistency, readability, and maintainability. Refer to .claude/skills/knowledge/framework-rules.md, .claude/skills/knowledge/framework-architecture.md, .claude/skills/knowledge/naming-conventions.md, .claude/skills/knowledge/typescript-coding-standards.md, and .claude/skills/knowledge/playwright-best-practices.md alongside this file.

## Dependency Matrix
Template assignment is declared in .claude/agent/agent.md's Skill Dependency Matrix — that table is the single source of truth. Skill 04 creates these files; Skill 05 appends verification methods to the same files (see Verification Methods below).

## Objective
Generate one Page Object per Page Inventory entry (.claude/skills/knowledge/framework-architecture.md's Page Identity Test) — never one per UI pattern, component, or feature. Each Page Object contains only logic related to that one real page — never combine multiple pages into a single class, and never split a single page's table/dropdown/search/filter patterns into classes of their own.

## Generation Workflow
Read the Reuse Inventory (the project's base class and helpers) → Analyze Page → Identify Elements (including every pattern/component found on it) → Generate Locators → Generate Constructor → Generate Navigation Methods → Generate Action Methods → Generate Baseline Verification Methods → Generate Compound Business Methods → Validate Output (incl. Page Object Count Rule)

## File Structure (fixed order)
Imports → Class Declaration → Private Locator Variables → Constructor → Navigation Methods → Input Methods → Click Methods → Selection Methods → Verification Methods → Compound Business Methods → Private Helper Methods (optional). Never change this order.

## The Project's Base Class — REFERENCE ONLY, DO NOT GENERATE
**The base class is hand-written and owned by the user.** Skill 00 read it and recorded its real name, methods, and import path in the **Reuse Inventory** (.claude/skills/skill-00-framework-inventory.md) — read that, and extend whatever is actually there. It may be `BasePage.ts`, `BaseClass.ts`, `CommonPage.ts`, or anything else.

The listing below is **an illustration of the shape such a class normally has**, so the rules that follow it (`protected page`, no generic `waitForPageLoad()`) have something concrete to refer to. It is not a file to generate, and it is not a specification the user's class has to match:

```typescript
import { Page } from '@playwright/test';

export class BasePage {
    protected readonly page: Page;

    constructor(page: Page) {
        this.page = page;
    }

    protected async goto(path: string): Promise<void> {
        await this.page.goto(path);
    }

    async getCurrentUrl(): Promise<string> {
        return this.page.url();
    }

    async getPageTitle(): Promise<string> {
        return this.page.title();
    }
}
```

**`page` stays `protected`.** It is reachable by subclasses and by nothing else. A spec that needs to navigate calls `await somePage.navigateTo()`; a spec that writes `somePage.page.goto(...)` is both a compile error and a POM-03 violation, and the fix is a Page Object method — never widening `page` to `public`.

**No generic `waitForPageLoad()`.** A shared `waitForLoadState('networkidle')` helper is deliberately absent: Playwright discourages `networkidle` for testing, and a page is "loaded" when the element that proves it is visible, not when the network goes quiet. Each Page Object waits for its own readiness signal instead (BP-503):
```typescript
async navigateTo(): Promise<void> {
    await this.goto('/select-user-group');
    await expect(this.userGroupHeadingLocator).toBeVisible();
}
```

**Never generate, regenerate, or overwrite the base class.** Call what the Reuse Inventory says exists (RS-01), and extend the project's class by its real name (RS-04). If the application needs a shared behaviour the class genuinely lacks, add it as a **new** method in the user's own style and record it in the Inventory — never by rewriting an existing method, which other tests already call.

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
Generate all locators before the constructor — `private`, `readonly`, meaningful names, following the stable locator strategy (.claude/skills/knowledge/locator-strategy.md's priority order):
```typescript
private readonly usernameInputLocator: Locator;
private readonly passwordInputLocator: Locator;
private readonly loginButtonLocator: Locator;
private readonly errorMessageLocator: Locator;
```
Never expose locators publicly.

## Constructor
Responsibilities: call `super(page)`, initialize every locator declared above — nothing else. No business logic in the constructor.
```typescript
constructor(page: Page) {
    super(page);
    this.usernameInputLocator = page.getByLabel('Username');
    this.passwordInputLocator = page.getByLabel('Password');
    this.loginButtonLocator = page.getByRole('button', { name: 'Login' });
    this.errorMessageLocator = page.getByTestId('login-error-message');
}
```

## Navigation Methods
Each Page Object owns its own route and its own readiness check:
```typescript
async navigateTo(): Promise<void> {
    await this.goto('/login');
    await expect(this.loginButtonLocator).toBeVisible();
}

async refreshPage(): Promise<void> {
    await this.page.reload();
}
```
`this.goto()` is BasePage's `protected` helper — legal here because this class extends BasePage. Never expose a navigation path to the spec; the spec calls `navigateTo()` and nothing else.

## Action Methods
One method per action, each with a single responsibility, explicit parameter and return types:
```typescript
async fillUsername(username: string): Promise<void> {
    await this.usernameInputLocator.fill(username);
}

async fillPassword(password: string): Promise<void> {
    await this.passwordInputLocator.fill(password);
}

async clickLoginButton(): Promise<void> {
    await this.loginButtonLocator.click();
}
```

## Verification Methods
Skill 04 generates a **baseline** set of pattern-driven verification methods (page visibility, URL, title). Skill 05 (Assertion Generation) then **appends** whatever additional methods each test case's `expectedResult` requires, directly into this same class — reusing a baseline method where one already fits instead of duplicating it. See .claude/skills/templates/verification-template.md for the full method-generation guidance.
```typescript
async verifyLoginPageVisible(): Promise<void> {
    await expect(this.loginButtonLocator).toBeVisible();
}

async verifyErrorMessageVisible(expectedMessage: string): Promise<void> {
    await expect(this.errorMessageLocator).toBeVisible();
    await expect(this.errorMessageLocator).toHaveText(expectedMessage);
}
```
Assertions stay inside the Page Object — never expose locators to Spec files.

## Compound Methods
Reusable business workflows that combine action methods — never duplicate the underlying interaction logic:
```typescript
async performLogin(username: string, password: string): Promise<void> {
    await this.fillUsername(username);
    await this.fillPassword(password);
    await this.clickLoginButton();
}
```

## Private Helpers
Generate only when they improve readability or eliminate duplication — don't create unnecessary helper methods.

## Multi-Pattern Pages — Fold Everything Into ONE Class (Critical)
When a single real page contains several UI patterns (a data table, a column-selection dropdown, a free-text search box), all of them become methods on that page's ONE Page Object — never separate classes. This is the Page Object Count Rule from .claude/skills/skill-04-page-object-generation.md, illustrated in code:

```typescript
// Correct — one page, one class, every pattern folded in as methods
export class ReportsPage extends BasePage {
    private readonly columnSelectorLocator: Locator;
    private readonly freeTextSearchLocator: Locator;
    private readonly tableRowLocator: Locator;

    constructor(page: Page) {
        super(page);
        this.columnSelectorLocator = page.getByRole('combobox', { name: 'Columns' });
        this.freeTextSearchLocator = page.getByPlaceholder('Search reports');
        this.tableRowLocator = page.getByRole('row');
    }

    // Dropdown pattern → a method here, never a ColumnSelectionPage.ts
    async selectColumns(columns: string[]): Promise<void> {
        await this.columnSelectorLocator.click();
        for (const column of columns) {
            await this.page.getByRole('option', { name: column }).click();
        }
    }

    // Free-text search pattern → a method here, never a FreeTextPage.ts
    async performFreeTextSearch(query: string): Promise<void> {
        await this.freeTextSearchLocator.fill(query);
        await this.freeTextSearchLocator.press('Enter');
    }

    // Data table pattern → a method here, never a DataTablePage.ts
    async verifyTableData(expectedRowCount: number): Promise<void> {
        await expect(this.tableRowLocator).toHaveCount(expectedRowCount);
    }
}
```

## Complete Worked Example
Everything above assembled into one file, in the fixed File Structure order:

```typescript
import { Page, Locator, expect } from '@playwright/test';
import { BasePage } from './BasePage';

export class LoginPage extends BasePage {
    private readonly usernameInputLocator: Locator;
    private readonly passwordInputLocator: Locator;
    private readonly loginButtonLocator: Locator;
    private readonly errorMessageLocator: Locator;

    constructor(page: Page) {
        super(page);
        this.usernameInputLocator = page.getByLabel('Username');
        this.passwordInputLocator = page.getByLabel('Password');
        this.loginButtonLocator = page.getByRole('button', { name: 'Login' });
        this.errorMessageLocator = page.getByTestId('login-error-message');
    }

    async navigateTo(): Promise<void> {
        await this.goto('/login');
        await expect(this.loginButtonLocator).toBeVisible();
    }

    async fillUsername(username: string): Promise<void> {
        await this.usernameInputLocator.fill(username);
    }

    async fillPassword(password: string): Promise<void> {
        await this.passwordInputLocator.fill(password);
    }

    async clickLoginButton(): Promise<void> {
        await this.loginButtonLocator.click();
    }

    async verifyLoginPageVisible(): Promise<void> {
        await expect(this.loginButtonLocator).toBeVisible();
    }

    async verifyErrorMessageVisible(expectedMessage: string): Promise<void> {
        await expect(this.errorMessageLocator).toBeVisible();
        await expect(this.errorMessageLocator).toHaveText(expectedMessage);
    }

    async performLogin(username: string, password: string): Promise<void> {
        await this.fillUsername(username);
        await this.fillPassword(password);
        await this.clickLoginButton();
    }
}
```

## Code Quality
Every generated Page Object must follow framework rules, locator strategy, naming conventions, and Playwright best practices; use explicit return types; use async/await correctly; avoid duplicate logic; be production ready.

## Validation Checklist
✓ every class extends the project's own base class, by its real name · ✓ no base class generated, and no existing base-class method rewritten · ✓ no method written that the Reuse Inventory already provides · ✓ one class per file, one file per Page Inventory entry · ✓ every detected pattern/component folded into its page's own class, none given its own file · ✓ correct imports · ✓ private readonly locators · ✓ constructor initializes every locator · ✓ public action methods · ✓ public verification methods (baseline + Skill 05 additions) · ✓ compound business methods · ✓ no unused code · ✓ no fabricated locators · ✓ follows all project standards
