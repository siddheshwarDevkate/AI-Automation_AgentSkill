# SKILL 03 — LOCATOR GENERATION

## Purpose
Analyze the application's DOM and generate stable, maintainable, reusable locators. Responsible only for locator generation — does NOT generate Page Objects, test scripts, or assertions.

## Execution Dependencies
**Knowledge:** skills/knowledge/framework-rules.md, skills/knowledge/locator-strategy.md, skills/knowledge/playwright-best-practices.md, skills/knowledge/naming-conventions.md, skills/knowledge/generation-patterns.md
**Templates:** None

## Inputs
Required: Application Analysis Report, Live Application, Browser Automation Session (Playwright MCP).
Optional: existing Page Objects, existing framework.

## Expected Output
- Locator Map (organized by page)
- Page-wise element list
- Reusable component locator list
- Locator notes

Does NOT generate Page Objects, test scripts, assertions, or test data.

## Responsibilities
Open every discovered page, inspect the DOM, identify interactive elements, select the most stable locator, validate uniqueness, map elements to pages, document decisions.

## Workflow
Read Application Analysis → Open Target Page → Inspect DOM → Identify Interactive Elements → Select Best Locator → Validate Locator → Generate Locator Map → Proceed to Next Page (repeat until all pages analyzed)

## Element Discovery
Identify elements that participate in user interaction: buttons, textboxes, password fields, text areas, dropdowns, checkboxes, radio buttons, links, tables, search boxes, filters, date pickers, tabs, sidebars, menus, dialogs, toast messages, upload/download controls, pagination, cards, interactive icons. Ignore decorative elements.

## Locator Selection Strategy (per skills/knowledge/locator-strategy.md)
Priority order: `data-testid` → `data-test` → `id` → `name` → ARIA role → label → specific CSS selector → XPath (last resort). Never generate unstable locators.

## Locator Validation
Accept only locators that are unique, stable, readable, maintainable, and survive minor UI changes. Reject dynamic IDs, positional selectors, random CSS classes, and fragile XPath.

## Shared Component Analysis
Identify reusable components (header, sidebar, navigation menu, search panel, common dialog, common table, pagination, footer) and generate their locators once for reuse across multiple Page Objects.

## Locator Map Format
Organize generated locators by page, using Skill 02's Page Inventory as the exhaustive list of top-level groups — never add a group for a component/feature/pattern. Elements belonging to a table, dropdown, filter panel, or search box nest under the page that contains them, e.g.:
```
Reports Page  (one Page Inventory entry)
  Column Selection Dropdown, Free Text Search Box, Data Table (rows/columns), Filter Panel
Dashboard
  Search Box, Profile Menu, Logout Button
```
`Reports Page` is one group because it is one Page Inventory entry — its column selector and free-text search are elements nested inside it, never separate top-level groups.

## Special Handling
Handle dynamic applications carefully: lazy-loaded elements, infinite scroll, dynamic menus, nested dialogs, frames, Shadow DOM, loading indicators. Inspect the DOM only after the element becomes available — never rely on hardcoded waits.

## Success Criteria
Every page inspected · interactive elements identified · stable locators selected · Locator Map generated · shared components identified · locator quality validated.

## Failure Handling
Stop if the DOM cannot be inspected, a required page is inaccessible, browser automation fails, or a stable locator cannot be identified. If an element cannot be located, do not fabricate a locator — document the issue, add a TODO, and continue with remaining elements.

## References
**Knowledge:** skills/knowledge/framework-rules.md, skills/knowledge/locator-strategy.md, skills/knowledge/playwright-best-practices.md, skills/knowledge/generation-patterns.md
**Templates:** None
**Consumed by:** Skill 04 — Page Object Generation
