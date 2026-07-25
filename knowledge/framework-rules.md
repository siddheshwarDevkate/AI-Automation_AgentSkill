# FRAMEWORK RULES

**Version:** 1.0 · **Category:** Global Standard · **Applies To:** Agent, Skills, Templates

## Purpose
The mandatory engineering rules for generating the Playwright automation framework. Every Agent, Skill, and Template MUST follow these — they have the highest priority after explicit user instructions.

## Used By Skills
All 8 skills (01–08)

## Rule Priority
1. User Instructions
2. Framework Rules (this document)
3. Project Standards
4. Playwright Best Practices
5. General Software Engineering Practices

Higher priority always overrides lower.

## Critical Rules
Violating any CRITICAL rule is a framework generation failure — stop until corrected.

1. **Never fabricate information.** Generate only from information obtained from the live application. If undeterminable, STOP and report — never guess.
2. **The running application is always the source of truth.** Use the configured browser automation tool (Playwright MCP in the current implementation). Never rely on assumptions.
3. **Understand before generating.** Framework generation begins only after successful application analysis.
4. **Generate only production-ready code.** Incomplete implementations are prohibited.
5. **Every generated file must pass validation before delivery.**

## Page Object Model Rules
- **POM-01:** Every locator must remain `private readonly`. Never expose locators outside the Page Object.
- **POM-02:** Specs interact only through public methods. Correct: `await loginPage.performLogin(username, password);` — Incorrect: `await loginPage.usernameInputLocator.fill(username);`
- **POM-03:** Never expose Page properties. Correct: `await loginPage.verifyCurrentUrl(...)` — Incorrect: `await expect(loginPage.page).toHaveURL(...)`
- **POM-04:** Every Page Object exposes action methods, verification methods, and compound methods.
- **POM-05:** Business logic belongs inside Page Objects; Spec files stay clean and readable.

## Locator Rules
- **LOC-01:** Follow priority — `data-testid` → `data-test` → `id` → `name` → `getByRole()` → `getByLabel()` → stable CSS → XPath (last resort).
- **LOC-02:** Never generate a locator that doesn't exist — add a TODO instead of fabricating.
- **LOC-03:** Never use unstable selectors (e.g. `.btn`, `div:nth-child(5)`, `.container div span`).
- **LOC-04:** Prefer semantic locators: `page.getByRole('button', { name: 'Login' })` over `page.locator("button:nth-child(4)")`.
- **LOC-05:** XPath is the final fallback, only when every higher-priority strategy fails.

## Playwright Rules
- **PW-01:** Never use `page.waitForTimeout()` — always use Playwright's waiting mechanisms.
- **PW-02:** Never use `textContent()` — always use `innerText()`.
- **PW-03:** Always use Playwright assertions: `await expect(locator).toBeVisible();`, not `expect(await locator.isVisible()).toBe(true);`.
- **PW-04:** Always use async/await; never ignore Promises.
- **PW-05:** Prefer `expect(page).toHaveURL(...)` over manual URL validation.

## TypeScript Rules
- **TS-01:** Never use `any` — use explicit typing.
- **TS-02:** Every method declares a return type, e.g. `async login(): Promise<void>`.
- **TS-03:** Avoid unused imports.
- **TS-04:** Avoid unused variables.
- **TS-05:** Prefer `readonly` whenever possible.

## Spec File Rules
- **SPEC-01:** Specs describe business behaviour, not UI implementation.
- **SPEC-02:** Never access locators directly.
- **SPEC-03:** Never duplicate actions — reuse Page Object methods.
- **SPEC-04:** Keep tests independent; one test never depends on another.
- **SPEC-05:** Use meaningful test names (e.g. "Valid login redirects user to dashboard", not "Test1").

## Assertion Rules
Always verify expected behaviour using Playwright assertions: `toBeVisible()`, `toBeHidden()`, `toHaveURL()`, `toHaveTitle()`, `toHaveText()`, `toHaveValue()`, `toBeEnabled()`, `toBeDisabled()`. Avoid manual assertions whenever a Playwright assertion exists.

## Output Rules
Do not generate `playwright.config.ts`, `package.json`, or `tsconfig.json` unless explicitly requested. Do not overwrite existing files unless instructed. Do not modify unrelated code.

## Code Quality Rules
Prefer reusable methods · keep methods small · Single Responsibility Principle · avoid duplication · meaningful names · readable code · concise Specs · maintainable Page Objects.

## Failure Conditions
Stop generation if: application cannot be analyzed, authentication fails, a locator cannot be determined, framework validation fails, or required information is unavailable. Never fabricate implementation to bypass a failure.

## Self Review Checklist
Before returning generated code, verify: no fabricated locators · no private-locator access from Specs · no `page.waitForTimeout()` · no `textContent()` · no `any` type · proper POM · meaningful method names · stable locators · Playwright best practices followed · output matches requested structure.
