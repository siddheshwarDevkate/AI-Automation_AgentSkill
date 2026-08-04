# VERIFICATION TEMPLATE

## Purpose
The standard approach for generating verification methods inside Playwright Page Objects. Verification methods validate application behavior, page state, UI elements, and business outcomes. Specs invoke these methods rather than writing Playwright assertions directly. Refer to skills/knowledge/framework-rules.md, skills/knowledge/playwright-best-practices.md, and skills/knowledge/naming-conventions.md alongside this file.

## Dependency Matrix
Template assignment is declared in agent/agent.md's Skill Dependency Matrix — that table is the single source of truth.

## Objective
Generate reusable verification methods that encapsulate Playwright assertions — hiding implementation, improving test readability, promoting reuse, and following POM principles.

## You Are Appending to Skill 04's Files, Not Creating New Ones (Critical)
Every verification method this template describes gets added to the **existing** Page Object file Skill 04 already generated for that page (e.g. new methods on `ReportsPage.ts`, not a new `ReportsVerification.ts` or `VerificationHelper.ts`). Before adding a method, check whether Skill 04's baseline pattern coverage already produced an equivalent one — reuse it instead of duplicating. A verification method never gets its own class, regardless of how many of them a page needs (see skills/knowledge/framework-architecture.md's Page vs Component distinction — the same "don't split into a new file" rule that applies to tables and dropdowns applies here too).

## Generation Workflow
Identify Expected Behaviour (from the test case's `expectedResult`) → Identify Element(s) → Identify the Owning Page Object → Reuse an Existing Method or Select the Appropriate Assertion → Generate/Append the Verification Method → Validate Method Reusability → Return Production-Ready Code

## General Principles
- **VT-001 — Keep Assertions Inside Page Objects.** Correct: `await loginPage.verifyLoginSuccessful();`. Incorrect: `await expect(page.locator(...)).toBeVisible();` inside the Spec file.
- **VT-002 — Verify Business Behaviour** (user redirected, error/success message displayed, record created/deleted) over implementation detail.
- **VT-003 — One Verification Per Method.** e.g. `verifyDashboardVisible()`, `verifyErrorMessageVisible()` — don't combine unrelated assertions.

## Method Structure
Method Declaration → Playwright Assertion → Return.
```typescript
async verifyDashboardVisible(): Promise<void> {
    await expect(this.dashboardTitleLocator).toBeVisible();
}
```

Parameterized example — reusable across rows/values instead of one hardcoded method per case:
```typescript
async verifyCellValue(rowIndex: number, columnName: string, expectedValue: string): Promise<void> {
    const cellLocator = this.tableRowLocator.nth(rowIndex).getByRole('cell', { name: columnName });
    await expect(cellLocator).toHaveText(expectedValue);
}
```

## Verification Discovery Strategy
Don't generate a verification method just because a UI element exists — generate one only when it validates meaningful business behaviour or state.

1. **Identify the User Action** — login, logout, search, create/update/delete record, submit form, upload/download file, navigate.
2. **Identify Observable Outcomes** — page navigation, URL/title change, success/error message, record appears/updates/removed, dialog opens/closes, loading indicator disappears, button enabled/disabled, table data changes, upload completes, session changes.
3. **Select the Appropriate Verification** — match outcome to method, e.g. page navigation → `verifyCurrentUrl()`, `verifyPageTitle()`, `verifyDashboardVisible()`; success → `verifySuccessMessageVisible()`, `verifySuccessToast()`; errors → `verifyErrorMessageVisible()`, `verifyValidationMessage()`; data update → `verifyRecordCreated()`, `verifyRecordUpdated()`, `verifyRecordDeleted()`, `verifyCellValue()`; dialogs → `verifyDialogVisible()`, `verifyDialogClosed()`; uploads → `verifyUploadCompleted()`, `verifyUploadedFileVisible()`.
4. **Avoid Redundant Verifications** — don't generate multiple methods validating the same behaviour (e.g. `verifyDashboardVisible()`, `verifyHomePageVisible()`, `verifyLandingPageVisible()` together); pick the one that best represents the outcome.
5. **Prefer Generic Reusable Methods** when logic can be shared across pages — e.g. `verifyCurrentUrl(expectedUrl: string)` instead of `verifyDashboardUrl()`.
6. **Validate Completeness** — every major user action has a corresponding verification, every important business outcome is validated, no duplicates exist, and methods clearly express expected behaviour.

## Common Verification Methods by Category
- **Page Visibility:** `verifyLoginPageVisible()`, `verifyDashboardVisible()`, `verifyProfilePageVisible()`
- **URL:** `verifyCurrentUrl(expectedUrl)` → `await expect(this.page).toHaveURL(expectedUrl);`
- **Page Title:** `verifyPageTitle(expectedTitle)` → `await expect(this.page).toHaveTitle(expectedTitle);`
- **Element Visibility/Hidden:** `verifySearchBoxVisible()`, `verifyLogoutButtonVisible()`, `verifyErrorMessageHidden()`, `verifyLoadingSpinnerHidden()`
- **Text:** `verifySuccessMessage(expected)` → `await expect(this.successMessageLocator).toHaveText(expected);`
- **Input Value:** `verifyUsernameValue()`, `verifyEmailValue()` → `toHaveValue(expectedValue)`
- **Checkbox / Radio / Dropdown:** `verifyCheckboxChecked()`, `verifyOptionSelected()`, `verifySelectedOption()`
- **Table:** `verifyTableVisible()`, `verifyRowExists()`, `verifyCellValue()`, `verifyRecordPresent()`
- **Toast:** `verifySuccessToast()`, `verifyWarningToast()`, `verifyErrorToast()`
- **Dialog:** `verifyDialogVisible()`, `verifyDialogClosed()`
- **File Upload:** `verifyFileUploaded()`, `verifyUploadCompleted()`

## Assertion Selection Guide
Visible → `toBeVisible()` · Hidden → `toBeHidden()` · Text → `toHaveText()` · Partial text → `toContainText()` · Input value → `toHaveValue()` · Checked → `toBeChecked()` · URL → `toHaveURL()` · Title → `toHaveTitle()` · Enabled → `toBeEnabled()` · Disabled → `toBeDisabled()` · Focused → `toBeFocused()` · Count → `toHaveCount()`. Choose whichever most accurately validates the expected behaviour.

## Method Naming
Always prefix with `verify` — e.g. `verifyDashboardVisible()`, `verifyLoginSuccessful()`, `verifyCurrentUrl()`. Avoid `check()`, `validate()`, `assert()`, `test()`.

## Reusability
Prefer generic parameterized methods (`verifyCurrentUrl(expectedUrl: string)`) over page-specific duplicates (`verifyDashboardUrl()`) whenever the logic can be reused.

## Do Not Generate
Exposed Locator variables or returned Locator objects, user-action logic mixed with verification, `page.waitForTimeout()`, suppressed assertion failures, or duplicate verification methods.

## Validation Checklist
✓ Method names start with `verify` · ✓ added to the existing Page Object file, never a new class · ✓ one behaviour verified per method · ✓ uses Playwright `expect()` · ✓ assertions are reusable · ✓ explicit return types · ✓ no duplicate methods · ✓ no action logic included · ✓ follows naming conventions · ✓ follows framework standards · ✓ production-ready implementation
