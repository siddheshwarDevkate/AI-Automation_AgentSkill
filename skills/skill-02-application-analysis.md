# SKILL 02 — APPLICATION ANALYSIS

## Purpose
Analyze the target application, guided by the Test Case Model produced by Skill 01, to build the understanding required to automate exactly the scenarios the test cases define. Focuses on business/user understanding only — does NOT generate locators, Page Objects, or test scripts.

## Execution Dependencies
**Knowledge:** skills/knowledge/framework-architecture.md, skills/knowledge/framework-rules.md, skills/knowledge/generation-patterns.md, skills/knowledge/output-structure.md
**Templates:** None

## Inputs
Required: Test Case Model (from Skill 01), Application URL, login credentials, browser automation session.
Optional: existing framework.

## Expected Output — Application Analysis Report
- Application overview
- Authentication flow
- Navigation structure
- **Page Inventory** — the definitive, numbered list of distinct pages the Test Case Model touches (see below); this is the sole source of truth for how many Page Objects Skill 04 generates
- Business workflows (mapped to their source Test Case IDs)
- Reusable UI components (mapped to the page they belong to, never listed as pages themselves)
- Test Case Discrepancies (test case steps/expectations that don't match actual application behaviour)
- Important observations

No implementation code is generated at this stage.

## Responsibilities
- Launch and authenticate into the application
- Explore only the pages and workflows referenced by the Test Case Model
- Execute each test case's steps against the live application to confirm they are actually possible
- Identify reusable UI components encountered along the way
- Understand page relationships
- Document any mismatch between a test case and real application behaviour
- Produce the Application Analysis Report

## Workflow
Read Test Case Model → Launch Application → Authenticate → For Each Module/Workflow Referenced by a Test Case: Navigate → Execute the Test Case's Steps → Compare Actual Behaviour Against `expectedResult` → Identify UI Components Encountered (attaching each to its owning page) → Build the Page Inventory → Document Findings → Generate Application Analysis Report

## Test-Case-Guided Scope
Exploration is scoped to what the Test Case Model actually references — do not free-crawl the entire application. Only visit a page or workflow outside that scope if a test case's steps require passing through it (e.g. a login step on the way to a dashboard scenario).

## Application Exploration
Use the configured browser automation tool (Playwright MCP in the current implementation) to interact with the live application: open menus, expand navigation drawers, visit modules, navigate between pages, open dialogs/forms, view tables, open detail pages, perform non-destructive interactions.

Avoid destructive actions (permanent delete, reset database, production config changes, data removal) unless explicitly required by a test case.

## Page Identification
For every page a test case touches, document: name, purpose, entry point, exit points, navigation path, primary functionality, connected pages.

## Page Inventory (Critical)
Build a definitive, numbered list of the distinct pages the Test Case Model actually touches — this is the single most important artifact this Skill produces, because Skill 04 must generate exactly one Page Object per entry, no more and no fewer.

A page qualifies for the inventory only if it is a distinct navigable screen — normally its own URL/route, or a full-screen destination reached via navigation. Apply the Page Identity Test from skills/knowledge/framework-architecture.md.

**Do not add an entry for a UI feature, widget, or pattern found on a page.** A data table, a column-selection dropdown, a free-text search box, a filter panel, or a modal is not a page — it belongs to whichever page it appears on, and must be recorded under that page's UI components, never as its own inventory entry. If 20 test cases touch 4 real pages, the Page Inventory has exactly 4 entries, however many components/features exist across them.

For each entry, record: page name, URL/route (if applicable), and the Test Case IDs whose steps touch it.

## UI Components Belong to a Page, Not the Inventory
When a component (table, dropdown, free-text search, filter panel, etc.) is identified during exploration, attach it to the Page Inventory entry for the page it appears on. Never let a component acquire enough documentation weight that it gets mistaken for a page in its own right by a downstream Skill.

## Test Case Verification
While executing each test case's steps, confirm:
- Every step is actually performable in the current application.
- The page/element the step refers to exists and behaves as described.
- The actual outcome matches the test case's `expectedResult`.

If a step or expected result does not match reality, do not silently proceed or "fix" the test case. Record it as a **Test Case Discrepancy** in the Application Analysis Report, and mark the affected test case as blocked if the mismatch prevents automation — this is surfaced to the user via the Agent, not resolved by assumption.

## Business Workflow Analysis
Understand how users perform the business operations defined by the test cases end to end (e.g. Login → Dashboard → Search → View Details → Create → Update → Logout). Focus on the workflow itself, not automation implementation.

## UI Component Identification
Identify reusable UI components (forms, tables, search panels, filters, dropdowns, tabs, navigation menus, sidebars, dialogs, toast messages, file upload, pagination) encountered while executing test case steps, and document where each appears. Locator discovery belongs to Skill 03 — do not inspect locators here.

## Preparing for Downstream Skills
Ensure the analysis captures enough detail to understand application hierarchy, page relationships, user navigation, and the workflows defined by the Test Case Model. Do not attempt Application Modeling — the downstream Skills consume this report directly.

## Success Criteria
Test Case Model read · every referenced workflow/page visited and confirmed reachable · authentication understood · business workflows documented and mapped to their Test Case IDs · Page Inventory built with every page correctly distinguished from the components/features that live on it · reusable UI components identified and attached to their owning page · Test Case Discrepancies documented · Application Analysis Report generated.

## Failure Handling
Stop analysis if the application cannot be accessed, login fails, a required page cannot be reached, or browser automation fails. If a test case references a workflow or page that does not exist in the application, document it as a Test Case Discrepancy and mark that test case as blocked rather than fabricating a substitute. Report the issue to the Agent — never continue with incomplete analysis.

## References
**Knowledge:** skills/knowledge/framework-architecture.md, skills/knowledge/framework-rules.md, skills/knowledge/generation-patterns.md, skills/knowledge/output-structure.md
**Templates:** None
**Consumed by:** Skill 03 — Locator Generation
