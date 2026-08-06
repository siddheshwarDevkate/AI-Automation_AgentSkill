# SKILL 04 — PAGE OBJECT GENERATION

## Purpose
Generate production-ready Playwright Page Objects using the Application Analysis Report and Locator Map. Exposes business operations through public methods while encapsulating implementation details. Does NOT generate Spec files, test data, or assertions in test files.

## Execution Dependencies
Load the Knowledge files and Template listed for Skill 04 in agent/agent.md's **Skill Dependency Matrix** — that table is the single source of truth and is deliberately not restated here.

## Inputs
Required: Application Analysis Report (including its Page Inventory), Locator Map.
Optional: existing framework, existing BasePage, existing utility classes.

## Expected Output
- `BasePage.ts` (generated if it doesn't already exist — see BasePage Generation below)
- Exactly one Page Object per Page Inventory entry (from Skill 02) — private locators, public action methods, baseline pattern-driven verification methods, compound business methods, covering every pattern/component/feature that appears on that page as methods on the same class
- Utility classes, only if genuinely needed to remove duplication (see Utility Class Generation below)

Does NOT generate: Spec files, test data, or in-test assertions. Does NOT generate the full, final set of verification methods — Skill 05 (Assertion Generation) adds whatever this Skill's baseline pattern coverage doesn't already satisfy for each test case's `expectedResult`.

## Responsibilities
Create Page Object classes — one per Page Inventory entry, no more, no fewer — initialize locators, implement business actions, implement reusable methods, encapsulate Playwright interactions, maintain clean architecture — following skills/templates/page-object-template.md exactly.

## BasePage Generation
Before generating any page-specific Page Object, check whether `BasePage.ts` already exists in the target framework.
- If it exists, reuse it as-is — do not regenerate or overwrite it.
- If it doesn't exist, generate it first, per skills/knowledge/framework-architecture.md's BasePage definition: reusable navigation, generic waits, and common helper methods only — never page-specific logic. Every Page Object generated this run extends it.

## Utility Class Generation
While generating Page Objects, watch for logic that would otherwise be duplicated across multiple classes (e.g. a repeated wait pattern, a repeated date-formatting routine). When found, extract it into a utility class (e.g. `WaitHelper.ts`, `DateHelper.ts`) in `utils/`, per skills/knowledge/framework-architecture.md's Utilities section, instead of duplicating it inline. Do not create a utility class speculatively — only when actual duplication is found.

**Checking is mandatory, even though creating is conditional.** After generating the full set of Page Objects, compare their method bodies against each other, per skills/knowledge/framework-architecture.md's Mandatory Extraction Analysis: identical bodies across two or more classes move to `BasePage`; repeated non-page-specific helper logic moves to `utils/`. Record the outcome — what was extracted, or an explicit "no logic duplicated across Page Objects." An empty `utils/` is a valid result only as that stated conclusion, never because the comparison was skipped.

## Workflow
Check/Generate BasePage → Read Application Analysis → Read Page Inventory → Read Locator Map → For Each Page Inventory Entry, Generate ONE Page Object (per skills/templates/page-object-template.md structure), Folding In Every Pattern/Component Found On That Page As Methods → Generate Action Methods → Generate Baseline Verification Methods → Extract Duplicated Logic Into Utility Classes When Found → Validate Generated Class Against the Page Object Count Rule Below → Proceed to Next Page Inventory Entry

## Page Object Count Rule (Critical)
**The number of generated Page Objects must equal the number of entries in Skill 02's Page Inventory — never one per detected pattern, component, or feature.**

Before generating anything for a page, enumerate every pattern/component identified on it (e.g. a data table, a column-selection dropdown, a free-text search box, a filter panel) and fold all of them into methods on that ONE page's Page Object. Never spin up a separate class for any of them.

**Decision test for every class you're about to create:**
1. Does this correspond exactly to one entry in the Page Inventory? → Yes: generate its Page Object.
2. Is this a pattern/component/feature that lives inside a page already covered by a Page Object? → Yes: add it as method(s) on that existing Page Object. Do NOT create a new class.
3. Is this the exact same widget appearing identically on two or more different Page Inventory entries? → Only then may it become a separate reusable component class (not a Page Object, not counted against the Page Inventory, not named `*Page.ts`) — composed into each Page Object that uses it.

**Forbidden example (has actually happened — do not repeat):** a "Reports" page (one Page Inventory entry) containing a data table, a column-selection dropdown, and a free-text search box must produce exactly one `ReportsPage.ts` with methods such as `selectColumns()`, `performFreeTextSearch()`, and `verifyTableData()`. It must NOT produce `DataTablePage.ts`, `ColumnSelectionPage.ts`, and `FreeTextPage.ts`. If 20 test cases exercise 4 real pages, generate exactly 4 Page Objects.

Before moving to Spec Generation, count the generated Page Objects and confirm the number matches the Page Inventory exactly. If it doesn't, find the extra component-masquerading-as-a-page class(es), merge their methods into the correct page's Page Object, and delete the incorrect class.

## Pattern Detection
Patterns and components detected on a page (per skills/knowledge/generation-patterns.md's Pattern Library) become METHODS on that page's single Page Object — never a Page Object of their own. Before generating, identify reusable UI/business patterns across pages (authentication, dashboard, CRUD, search, listing, detail, wizard/multi-step, settings, report, profile, modal/dialog, table management, file upload, approval workflow). Reuse method structure, naming, navigation, verification, and helper methods across pages sharing a pattern rather than generating each independently — but the pattern still lives inside its page's existing class.

Common reusable business operations: login, logout, search, filter, create, edit, delete, save, cancel, upload, download, approve, reject — implement consistently across the framework. Move genuinely shared logic into BasePage, shared component classes, or utility classes rather than duplicating it, per the Page Object Count Rule's exception above.

## Page Complexity Analysis
- **Simple** (Login, Forgot Password, Profile): lightweight Page Object
- **Medium** (Search, Dashboard, Listing): reusable helper methods, grouped business actions
- **Complex** (multi-step forms, admin modules, wizards, multi-table pages): break into multiple focused methods; avoid long single methods

## Page Object Structure
Follow skills/templates/page-object-template.md order: Imports → Class Declaration → Constructor → Private Readonly Locators → Navigation Methods → Action Methods → Verification Methods → Compound Business Methods.

## Locator Implementation
Use only locators from the Locator Map. Never guess, create, or modify validated locators. If a locator is unavailable, add a TODO instead of fabricating one.

## Method Generation
Generate methods expressing clear business intent (e.g. `fillUsername()`, `clickLoginButton()`, `selectCountry()`, `uploadDocument()`, `searchCustomer()`, `deleteRecord()`).

## Compound Methods
Combine common business actions into reusable methods (e.g. `performLogin()`, `createCustomer()`, `updateProfile()`, `submitOrder()`). Avoid duplicating interaction logic across methods.

## Verification Methods
Generate the baseline reusable verification methods implied by the recognized pattern(s) (e.g. `verifyDashboardVisible()`, `verifyErrorMessageVisible()`, `verifyCurrentUrl()`, `verifyPageTitle()`). This is a starting set, not the final one — Skill 05 (Assertion Generation) will add whatever specific `expectedResult` values from the test cases still need a method after this baseline. Assertions stay inside the Page Object — never expose locators to Spec files.

## Code Quality
Every generated Page Object must compile, follow strict TypeScript, follow naming conventions, follow Playwright best practices, use private readonly locators, expose only public business methods, and avoid duplicate logic.

## Validation
Before completing each Page Object verify: correct class structure, complete constructor, all required locators initialized, public methods generated, verification methods available, no direct locator access from outside, no framework rule violations. Before completing the full set, verify the Page Object Count Rule: generated class count equals the Page Inventory count exactly.

## Success Criteria
`BasePage.ts` present (generated or reused) · exactly one Page Object per Page Inventory entry · every pattern/component found on a page folded into that page's own class · no component/pattern generated its own Page Object · genuinely duplicated logic extracted into utility classes, not left inline · all locators implemented · action/baseline-verification/business methods generated · framework standards followed.

## Failure Handling
If generation fails, document the issue, preserve generated code, add TODOs where required, and continue with remaining Page Objects. Never fabricate missing implementation.

## Consumed By
Skill 05 — Assertion Generation (immediately next; extends these same files) · later read by Skill 07 — Spec Generation
