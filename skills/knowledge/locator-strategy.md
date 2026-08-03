# LOCATOR STRATEGY

## Purpose
The single source of truth for discovering, selecting, validating, and maintaining Playwright locators. Objective: generate the most stable locator for every element — prefer long-term stability over shorter syntax.

## Used By Skills
03 Locator Generation · 08 Framework Validation · 09 Test Execution & Self-Healing (locator-drift repairs)

## General Principles
- **LS-001 — Use the Live Application.** Inspect the running DOM. Never assume attributes.
- **LS-002 — One Element, One Locator.** Avoid multiple locators for the same element unless necessary.
- **LS-003 — Prefer Stability.** Choose locators least affected by styling, layout, position, or minor UI changes.
- **LS-004 — Readability Matters.** A readable locator beats a shorter but unclear one.

## Locator Priority
1. `data-testid`
2. `data-test`
3. `id`
4. `name`
5. `getByRole()`
6. `getByLabel()`
7. `getByPlaceholder()`
8. Stable CSS selector
9. XPath (final fallback)

## By Strategy

**Data Test Attributes** (preferred whenever available): `page.getByTestId('login-button')` or `page.locator('[data-testid="login-button"]')`.

**ID Locators** — only when unique, stable, not dynamically generated. Correct: `page.locator('#username')`. Avoid: `page.locator('#input-12345')` if the ID changes per session.

**Role Locators** (preferred, semantic): `page.getByRole('button', { name: 'Login' })` — readable, stable, accessibility-friendly.

**Label Locators** (preferred for form controls): `page.getByLabel('Email')` — avoid CSS when a label is available.

**Placeholder Locators** — only when higher-priority options are unavailable: `page.getByPlaceholder('Enter username')`.

**CSS Locators** — only when semantic locators are unavailable. Prefer `input[type="email"]`; avoid `div > div > div > button` or `.container button:nth-child(4)`.

**XPath** — final option. Preferred patterns: AND (`//button[@type='submit' and @name='login']`), Contains (`//button[contains(text(),'Login')]`), Following Sibling (`//label[text()='Email']/following-sibling::input`), Ancestor (`//input[ancestor::form]`). Avoid absolute XPath like `/html/body/div/div/div/form/input`.

## Special Cases
- **Dynamic Elements** — prefer stable attributes over dynamic IDs/classes/text.
- **Duplicate Elements** — don't pick the first match blindly; find additional attributes for unique identification.
- **Tables** — locate by header, row, column, or cell content; avoid relying on row index unless required.
- **Menus** — open the menu first, wait for visibility, then interact. Never click hidden elements.
- **Dialogs** — wait until visible before interacting; verify the dialog closes after the action.
- **iFrames** — use `frameLocator()`; never access iframe elements directly from the main page.
- **Shadow DOM** — use Playwright's native Shadow DOM support; avoid JS workarounds unless necessary.

## Validation Requirements
Every locator must be: unique, stable, readable, maintainable, present in the application, and matching the intended element.

## Do Not Generate
Generic class selectors (`.btn`), position-based selectors (`div:nth-child(3)`), absolute XPath, dynamic IDs, or any fabricated locator not present in the application.

## Locator Decision Flow
```
Element Found
  → data-testid? Yes → Use it
  → No → ID? Yes → Validate stability
  → No → Role? Yes → Use role
  → No → Label? Yes → Use label
  → No → CSS? Yes → Validate
  → No → XPath (last resort)
```

## Final Checklist
✓ Correct element selected · ✓ locator is unique · ✓ follows priority order · ✓ no fabricated attributes · ✓ no unstable selectors · ✓ no absolute XPath · ✓ no unnecessary complexity · ✓ readable · ✓ maintainable
