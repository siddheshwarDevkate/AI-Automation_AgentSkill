# SKILL 01 — APPLICATION ANALYSIS

## Purpose
Analyze the target application and build a complete understanding of its functionality before any automation artifacts are generated. Focuses on business/user understanding only — does NOT generate locators, Page Objects, or test scripts.

## Execution Dependencies
**Knowledge:** framework-architecture.md, framework-rules.md, generation-patterns.md, output-structure.md
**Templates:** None

## Inputs
Application URL, login credentials, existing browser session, user requirements, business scenarios, existing framework (optional).

## Expected Output — Application Analysis Report
- Application overview
- Authentication flow
- Navigation structure
- Application modules & pages
- Business workflows
- Reusable UI components
- Important observations

No implementation code is generated at this stage.

## Responsibilities
- Launch and authenticate into the application
- Explore accessible pages systematically
- Understand user workflows and navigation patterns
- Identify reusable UI components
- Understand page relationships
- Produce the Application Analysis Report

## Workflow
Launch Application → Authenticate → Explore Navigation → Visit Every Accessible Page → Understand Business Workflow → Identify UI Components → Document Findings → Generate Application Analysis Report

## Application Exploration
Use the configured browser automation tool (Playwright MCP in the current implementation) to interact with the live application: open menus, expand navigation drawers, visit modules, navigate between pages, open dialogs/forms, view tables, open detail pages, perform non-destructive interactions.

Avoid destructive actions (permanent delete, reset database, production config changes, data removal) unless explicitly required.

## Page Identification
For every discovered page, document: name, purpose, entry point, exit points, navigation path, primary functionality, connected pages.

## Business Workflow Analysis
Understand how users perform business operations end to end (e.g. Login → Dashboard → Search → View Details → Create → Update → Logout). Focus on the workflow itself, not automation implementation.

## UI Component Identification
Identify reusable UI components (forms, tables, search panels, filters, dropdowns, tabs, navigation menus, sidebars, dialogs, toast messages, file upload, pagination) and document where each appears. Locator discovery belongs to Skill 02 — do not inspect locators here.

## Preparing for Downstream Skills
Ensure the analysis captures enough detail to understand application hierarchy, page relationships, user navigation, major workflows, and feature organization. Do not attempt Application Modeling — the downstream Skills consume this report directly.

## Success Criteria
Application explored · authentication understood · pages identified · navigation understood · business workflows documented · reusable UI components identified · Application Analysis Report generated.

## Failure Handling
Stop analysis if the application cannot be accessed, login fails, required pages cannot be reached, or browser automation fails. Report the issue to the Agent — never continue with incomplete analysis.

## References
**Knowledge:** framework-architecture.md, framework-rules.md, generation-patterns.md, output-structure.md
**Templates:** None
**Consumed by:** Skill 02 — Locator Generation
