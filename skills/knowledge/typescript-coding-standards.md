# TYPESCRIPT CODING STANDARDS

## Purpose
TypeScript coding standards for generated Playwright automation code — code quality, maintainability, consistency. Refer alongside skills/knowledge/framework-rules.md, skills/knowledge/framework-architecture.md, and skills/knowledge/playwright-best-practices.md. File/method ordering within a generated file is owned by skills/knowledge/output-structure.md, not repeated here.

## Dependency Matrix
Which Skills load this file is declared in agent/agent.md's Skill Dependency Matrix — that table is the single source of truth. Every Skill that emits TypeScript loads it.

## General Principles
- **TS-001 — Enable Strong Typing.** e.g. `const username: string = 'standard_user';`.
- **TS-002 — Never Use `any`.** Use proper interfaces, types, or generics instead.
- **TS-003 — Always Specify Return Types.** e.g. `async login(): Promise<void> {}`, `async getTitle(): Promise<string> {}`. Not `async login() {}`.
- **TS-004 — Prefer `readonly`** for properties that never change: `private readonly usernameInputLocator: Locator;`.
- **TS-004a — Never declare away a nullable return.** Several Playwright APIs return `T | null` — most commonly `textContent()` (`Promise<string | null>`) and `getAttribute()` (`Promise<string | null>`). Declaring the method `Promise<string>` and returning that value directly is a type error, and it compiles only if someone silences it. Handle the null instead:
```typescript
// Wrong — Type 'string | null' is not assignable to type 'string'
async getColumnsSelectedCount(): Promise<string> {
    return await this.columnsSelectedTextLocator.textContent();
}

// Right — innerText() returns Promise<string>, and reflects rendered text (BP-004)
async getColumnsSelectedCount(): Promise<string> {
    return await this.columnsSelectedTextLocator.innerText();
}

// Also right — when textContent() is genuinely needed
async getColumnsSelectedCount(): Promise<string> {
    return (await this.columnsSelectedTextLocator.textContent()) ?? '';
}
```
Never "fix" this with `as string`, `!`, or `any` — that hides a real null at runtime instead of handling it. Best of all, prefer an assertion (`await expect(locator).toHaveText(...)`) over a getter whenever the value is only being compared.
- **TS-005 — Use `const` by Default.** Use `let` only when reassignment is required; avoid unnecessary mutable variables.

## Imports
- **TS-101 — Import Only Required Modules.** `import { Page, Locator, expect } from '@playwright/test';` — avoid wildcard imports.
- **TS-102 — Remove Unused Imports.**
- **TS-103 — Group Imports:** external libraries → framework classes → project utilities.
```typescript
import { Page, Locator, expect } from '@playwright/test';
import { BasePage } from './BasePage';
import { WaitHelper } from '../utils/WaitHelper';
```

## Class Design
- **TS-201 — One Class Per File.**
- **TS-202 — Meaningful Class Names:** `LoginPage`, `DashboardPage`, `ProfilePage`, `WaitHelper` — not `Page`, `Helper`, `Class1`.
- **TS-203 — Constructor Responsibilities:** only initialize properties, assign locators, and lightweight setup. No business logic inside constructors.

## Methods
- **TS-301 — One Responsibility Per Method:** `fillUsername()`, `clickLoginButton()`, `verifyDashboardVisible()` — avoid combining unrelated actions.
- **TS-302 — Descriptive Method Names.** Good: `performLogin()`, `verifyCurrentUrl()`, `clickLogoutButton()`. Bad: `execute()`, `action()`, `verify()`.
- **TS-303 — Keep Methods Small** and focused.
- **TS-304 — Avoid Duplicate Code** — extract repeated logic into a reusable method.

## Variables
- **TS-401 — Meaningful Names:** `usernameInputLocator`, `dashboardTitleLocator`, `errorMessageLocator` — not `a`, `x`, `temp`, `button`.
- **TS-402 — Avoid Magic Values.** Prefer named constants: `const validUsername = 'standard_user';`.
- **TS-403 — Keep Variable Scope Small** — declare close to point of use.

## Comments
- **TS-501 — Comment Only When Necessary.** Code should be self-explanatory; avoid obvious comments (e.g. `// Click login button` above `await loginButton.click();`).
- **TS-502 — Use `// TODO:`** for missing information instead of guessing.

## Async Programming
- **TS-601 — Always Use `await`.**
- **TS-602 — Avoid Nested Async Logic** — keep async code simple and readable.
- **TS-603 — Do Not Ignore Promises** — every Promise should be awaited or intentionally handled.

## Error Handling
- **TS-701 — Fail Fast.** Allow errors to surface naturally; don't suppress exceptions.
- **TS-702 — Avoid Empty Catch Blocks** — always handle exceptions intentionally.

## Code Style
- **TS-801 — Consistent Formatting** — consistent indentation and spacing.
- **TS-802 — Remove Dead Code** — no unused variables/methods, commented-out code, empty methods.
- **TS-803 — Prefer Readability** over clever or shortened implementations.

## Generation Checklist
✓ No `any` type · ✓ explicit return types · ✓ readonly properties · ✓ proper async/await · ✓ meaningful names · ✓ small reusable methods · ✓ clean imports · ✓ no duplicate logic · ✓ no dead code · ✓ readable implementation
