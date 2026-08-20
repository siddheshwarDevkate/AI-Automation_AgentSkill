---
name: skill-04-page-object-generation.md
description: Generate exactly one Page Object per Page Inventory entry, extending the project's own base class and calling its existing methods rather than reimplementing them.
---

# SKILL 04 — PAGE OBJECT GENERATION

## Purpose
Generate production-ready Playwright Page Objects using the Application Analysis Report and Locator Map. Exposes business operations through public methods while encapsulating implementation details. Does NOT generate Spec files, test data, or assertions in test files.

## Execution Dependencies
Load the Knowledge files and Template listed for Skill 04 in .claude/agent/agent.md's **Skill Dependency Matrix** — that table is the single source of truth and is deliberately not restated here.

## Inputs
Required: Application Analysis Report (including its Page Inventory), Locator Map, **Reuse Inventory (from Skill 00 — the user's base class, `utils/`, `fixtures/`, `hooks/`, with exact signatures and import paths)**.

## Expected Output
- Exactly one Page Object per Page Inventory entry (from Skill 02) — private locators, public action methods, baseline pattern-driven verification methods, compound business methods, covering every pattern/component/feature that appears on that page as methods on the same class
- A new `utils/` helper or base-class method **only** where the Reuse Inventory has no equivalent and two or more pages need it (see Utility Class Generation below)

Does NOT generate the base class — it is the user's and already exists.

Does NOT generate: Spec files, test data, or in-test assertions. Does NOT generate the full, final set of verification methods — Skill 05 (Assertion Generation) adds whatever this Skill's baseline pattern coverage doesn't already satisfy for each test case's `expectedResult`.

## Responsibilities
Create Page Object classes — one per Page Inventory entry, no more, no fewer — initialize locators, implement business actions, implement reusable methods, encapsulate Playwright interactions, maintain clean architecture — following .claude/skills/templates/page-object-template.md exactly.

## The Base Class Is the User's — Extend It, Never Write It
**This Skill does not generate a base class.** The project's base page class is hand-written and maintained by the user; Skill 00 (.claude/skills/skill-00-framework-inventory.md) already read it and recorded every method in the **Reuse Inventory** in `execution-state.md`.

Before generating anything:
- **Read the Reuse Inventory first.** It gives the base class's real name (it may be `BasePage.ts`, `BaseClass.ts`, `CommonPage.ts`, or anything else), its methods with exact signatures, and its import path.
- **Every generated Page Object extends that class** (RS-04) and **calls** its methods. If the base class already provides `navigateTo()`, `click()`, or a wait helper, use it — writing a page-local version of behaviour that already exists is an RS-01 violation, not a stylistic choice.
- **Never rewrite, rename, or restructure an existing base-class method to make a generated Page Object fit.** Other tests already call it. If a generated class doesn't fit, the generated class is what changes. A genuinely missing capability that two or more pages need is added as a **new** base-class method, in the user's style and recorded in the Reuse Inventory, per Skill 00's Gap Handling.

If the Reuse Inventory records no base class at all, stop and report it — Skill 00's Phase B check 6 should already have caught it. Do not invent one and proceed.

## Utility Class Generation
**Check the Reuse Inventory first — the helper usually already exists.** The user's `utils/` is where their reusable logic lives, and a repeated wait pattern or date-formatting routine is exactly the kind of thing already sitting there. Call it (RS-01).

When logic would be duplicated across multiple generated classes and the Inventory has no equivalent, add a helper to the user's `utils/` — in their file organization, naming, and typing style — and record it in the Inventory. Never create one speculatively, and never in a parallel folder.

**Checking is mandatory, even though adding is conditional.** After generating the full set of Page Objects, compare their method bodies against each other, per .claude/skills/knowledge/framework-architecture.md's Mandatory Extraction Analysis: identical bodies across two or more classes belong on the base class (as a **new** method — never by rewriting an existing one); repeated non-page-specific helper logic belongs in `utils/`. Record the outcome — what was added, or an explicit "no logic duplicated across Page Objects, everything reused from the existing framework." Adding nothing is the expected result and a valid one, but only as a stated conclusion, never because the comparison was skipped.

## Workflow
Read the Reuse Inventory (base class, utils, fixtures, hooks) → Read Application Analysis → Read Page Inventory → Read Locator Map → For Each Page Inventory Entry, Generate ONE Page Object (per .claude/skills/templates/page-object-template.md structure), Folding In Every Pattern/Component Found On That Page As Methods → Generate Action Methods → Generate Baseline Verification Methods → Reuse Existing Helpers, Adding to `utils/`/the Base Class Only Where Nothing Equivalent Exists → Validate Generated Class Against the Page Object Count Rule Below → Proceed to Next Page Inventory Entry

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
Patterns and components detected on a page (per .claude/skills/knowledge/generation-patterns.md's Pattern Library) become METHODS on that page's single Page Object — never a Page Object of their own. Before generating, identify reusable UI/business patterns across pages (authentication, dashboard, CRUD, search, listing, detail, wizard/multi-step, settings, report, profile, modal/dialog, table management, file upload, approval workflow). Reuse method structure, naming, navigation, verification, and helper methods across pages sharing a pattern rather than generating each independently — but the pattern still lives inside its page's existing class.

Common reusable business operations: login, logout, search, filter, create, edit, delete, save, cancel, upload, download, approve, reject — implement consistently across the framework. **Several of these are usually already in the user's base class or `utils/`; call those rather than writing new ones.** Genuinely shared logic with no existing equivalent goes on the base class, a shared component class, or `utils/` — never duplicated across Page Objects, per the Page Object Count Rule's exception above.

## Page Complexity Analysis
- **Simple** (Login, Forgot Password, Profile): lightweight Page Object
- **Medium** (Search, Dashboard, Listing): reusable helper methods, grouped business actions
- **Complex** (multi-step forms, admin modules, wizards, multi-table pages): break into multiple focused methods; avoid long single methods

## Page Object Structure
Follow .claude/skills/templates/page-object-template.md order: Imports → Class Declaration → Constructor → Private Readonly Locators → Navigation Methods → Action Methods → Verification Methods → Compound Business Methods.

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
Reuse Inventory read and the project's base class extended by every generated class · no method written that the Inventory already provides · exactly one Page Object per Page Inventory entry · every pattern/component found on a page folded into that page's own class · no component/pattern generated its own Page Object · genuinely duplicated logic extracted into utility classes, not left inline · all locators implemented · action/baseline-verification/business methods generated · framework standards followed.

## Failure Handling
If generation fails, document the issue, preserve generated code, add TODOs where required, and continue with remaining Page Objects. Never fabricate missing implementation.

## Consumed By
Skill 05 — Assertion Generation (immediately next; extends these same files) · later read by Skill 07 — Spec Generation
