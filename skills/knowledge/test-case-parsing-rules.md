# TEST CASE PARSING RULES

## Purpose
Defines how to ingest a company-supplied Test Case file (CSV or Excel) and normalize it into a canonical Test Case Model that every downstream Skill can consume, regardless of the exact column headers, sheet layout, or tool used to produce the file. Refer alongside skills/knowledge/framework-rules.md and skills/knowledge/output-structure.md.

## Used By Skills
01 Test Case Analysis · 05 Assertion Generation · 06 Test Data Generation · 07 Spec Generation · 08 Framework Validation · 09 Test Execution & Self-Healing

## Objective
Convert raw spreadsheet/CSV rows into a structured, canonical Test Case Model — one record per test case — without losing or fabricating information. The Test Case file is the mandatory source of scenario truth for the entire framework; nothing downstream may invent a scenario that isn't traceable to a parsed row.

## Input Location
Read the Test Case file from `skills/Input/` (e.g. `TestCases.xlsx`, `TestCases.csv`). If multiple candidate files exist there, ask the user which one to use rather than guessing. If the file exists but contains zero data rows, treat this as a Critical Failure — request a populated file before continuing. Never generate a framework from an empty or fabricated Test Case Model.

## Supported Formats
- **CSV** — comma or semicolon delimited; detect the delimiter from the header row.
- **Excel (`.xlsx` / `.xls`)** — read the first worksheet by default. If the workbook contains multiple worksheets that each hold tabular test case data, treat every worksheet as a separate module/feature group and document this assumption in the Test Case Analysis Report rather than silently merging or ignoring extra sheets.

## Canonical Field Schema
Every parsed test case must resolve to these canonical fields:

| Canonical Field | Purpose |
|---|---|
| `id` | Unique Test Case identifier — the traceability key threaded through every downstream artifact |
| `title` | Scenario name / short description |
| `module` | Feature/module grouping — drives Spec file boundaries |
| `preconditions` | State required before executing the steps |
| `steps` | Ordered list of user actions |
| `testData` | Input values referenced by the steps |
| `expectedResult` | Observable outcome that verification methods must assert |
| `priority` | Optional — High/Medium/Low, informs Assertion Generation priority |
| `type` | Optional — positive/negative/boundary, informs Spec classification |

## Header Alias Matching
Company templates vary — match headers case-insensitively and tolerate wording differences instead of requiring exact column names:
- `id` ← "TC ID", "Test Case ID", "ID", "Case No", "TC#"
- `title` ← "Test Case", "Scenario", "Title", "Description", "Test Scenario"
- `module` ← "Module", "Feature", "Component", "Application Area"
- `preconditions` ← "Precondition", "Preconditions", "Pre-requisite", "Setup"
- `steps` ← "Steps", "Test Steps", "Action", "Test Procedure"
- `testData` ← "Test Data", "Data", "Input Data", "Input"
- `expectedResult` ← "Expected Result", "Expected", "Expected Output", "Expected Behaviour"
- `priority` ← "Priority", "Severity"
- `type` ← "Type", "Category", "Test Type"

If a header cannot be matched to a canonical field with reasonable confidence, do not silently drop the column. Surface it in the Test Case Analysis Report as an "Unmapped Column" and ask whether it should be treated as one of the canonical fields.

## Step Parsing
Steps are often stored in a single cell as a numbered or line-separated list (e.g. "1. Open login page\n2. Enter username\n3. Click Login"). Split into an ordered array and preserve sequence exactly — never reorder or merge steps.

## Missing Required Data
`id` and `steps` are required for a row to become an automatable test case.
- If either is missing, do not fabricate a value. Exclude the row from automation and record it under "Skipped Test Cases" in the Test Case Analysis Report, with the reason.
`title`, `expectedResult`, `testData`, `module`, `priority`, `type` are recommended but non-blocking — document any gap. Only infer a value when the live application makes the answer unambiguous during Application Analysis; otherwise leave it flagged as a limitation rather than guessing.

## Duplicate / Conflicting IDs
If two rows share the same `id`, stop and report the conflict — never silently pick one, merge them, or invent a suffix.

## Traceability ID Format
Preserve the `id` value exactly as supplied (e.g. `TC-014`, `LOGIN_003`). This value is threaded through every downstream artifact: Application Analysis scope, Spec file test titles/tags, Assertion mapping, Test Data mapping, and the final coverage report.

## Output — Test Case Model
A structured list of parsed test case records (canonical fields above), plus:
- Skipped Test Cases (with reason)
- Unmapped Columns (if any)
- Module/feature groupings derived from the data
- Assumptions made during parsing (e.g. multi-sheet handling)

## Final Checklist
✓ File format detected · ✓ headers mapped to canonical fields (or flagged as unmapped) · ✓ every row parsed into a record · ✓ steps preserved in order · ✓ rows missing required fields skipped and documented · ✓ duplicate IDs reported, not merged · ✓ no fabricated field values · ✓ Test Case Model ready for Skill 02
