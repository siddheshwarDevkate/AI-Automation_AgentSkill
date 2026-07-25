# SKILL 03 — PAGE OBJECT GENERATION

## Purpose
Generate production-ready Playwright Page Objects using the Application Analysis Report and Locator Map. Exposes business operations through public methods while encapsulating implementation details. Does NOT generate Spec files, test data, or assertions in test files.

## Execution Dependencies
**Knowledge:** framework-rules.md, playwright-best-practices.md, typescript-coding-standards.md, naming-conventions.md, generation-patterns.md, output-structure.md
**Templates:** page-object-template.md

## Inputs
Required: Application Analysis Report, Locator Map.
Optional: existing framework, existing BasePage, existing utility classes.

## Expected Output
- BasePage integration
- One Page Object per page: private locators, public action methods, public verification methods, compound business methods

Does NOT generate: Spec files, test data, or in-test assertions.

## Responsibilities
Create Page Object classes, initialize locators, implement business actions, implement reusable methods, encapsulate Playwright interactions, maintain clean architecture — following page-object-template.md exactly.

## Workflow
Read Application Analysis → Read Locator Map → Identify Required Pages → Generate Page Object (per page-object-template.md structure) → Generate Action Methods → Generate Verification Methods → Validate Generated Class → Proceed to Next Page

## Pattern Detection
Before generating, identify reusable UI/business patterns across pages (authentication, dashboard, CRUD, search, listing, detail, wizard/multi-step, settings, report, profile, modal/dialog, table management, file upload, approval workflow). Reuse method structure, naming, navigation, verification, and helper methods across pages sharing a pattern rather than generating each independently.

Common reusable business operations: login, logout, search, filter, create, edit, delete, save, cancel, upload, download, approve, reject — implement consistently across the framework. Move genuinely shared logic into BasePage, shared component classes, or utility classes rather than duplicating it.

## Page Complexity Analysis
- **Simple** (Login, Forgot Password, Profile): lightweight Page Object
- **Medium** (Search, Dashboard, Listing): reusable helper methods, grouped business actions
- **Complex** (multi-step forms, admin modules, wizards, multi-table pages): break into multiple focused methods; avoid long single methods

## Page Object Structure
Follow page-object-template.md order: Imports → Class Declaration → Constructor → Private Readonly Locators → Navigation Methods → Action Methods → Verification Methods → Compound Business Methods.

## Locator Implementation
Use only locators from the Locator Map. Never guess, create, or modify validated locators. If a locator is unavailable, add a TODO instead of fabricating one.

## Method Generation
Generate methods expressing clear business intent (e.g. `fillUsername()`, `clickLoginButton()`, `selectCountry()`, `uploadDocument()`, `searchCustomer()`, `deleteRecord()`).

## Compound Methods
Combine common business actions into reusable methods (e.g. `performLogin()`, `createCustomer()`, `updateProfile()`, `submitOrder()`). Avoid duplicating interaction logic across methods.

## Verification Methods
Generate reusable verification methods (e.g. `verifyDashboardVisible()`, `verifyErrorMessageVisible()`, `verifyCurrentUrl()`, `verifyPageTitle()`). Assertions stay inside the Page Object — never expose locators to Spec files.

## Code Quality
Every generated Page Object must compile, follow strict TypeScript, follow naming conventions, follow Playwright best practices, use private readonly locators, expose only public business methods, and avoid duplicate logic.

## Validation
Before completing each Page Object verify: correct class structure, complete constructor, all required locators initialized, public methods generated, verification methods available, no direct locator access from outside, no framework rule violations.

## Success Criteria
Every required Page Object generated · all locators implemented · action/verification/business methods generated · framework standards followed.

## Failure Handling
If generation fails, document the issue, preserve generated code, add TODOs where required, and continue with remaining Page Objects. Never fabricate missing implementation.

## References
**Knowledge:** framework-rules.md, playwright-best-practices.md, typescript-coding-standards.md, naming-conventions.md, generation-patterns.md
**Templates:** page-object-template.md
**Consumed by:** Skill 04 — Spec Generation, Skill 05 — Assertion Generation
