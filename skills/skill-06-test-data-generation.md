# SKILL 06 — TEST DATA GENERATION

## Purpose
Analyze generated test scenarios and determine the required test data. Decides WHAT test data is required — not how it's implemented.

## Execution Dependencies
**Knowledge:** framework-rules.md, generation-patterns.md, output-structure.md
**Templates:** None — this Skill produces a Test Data Plan (data only), not generated code, so no output template is required.

## Inputs
Required: Application Analysis Report, Generated Spec Plan, Assertion Plan.
Optional: existing test data, configuration files, environment information.

## Expected Output
- Test Data Plan
- Test data categories
- Test data mapping
- Shared test data strategy

Passed directly to the framework for implementation.

## Responsibilities
Identify required test data, categorize it, map it to scenarios, eliminate duplicate data, promote reusable datasets, flag sensitive information.

## Workflow
Read Application Analysis → Read Spec Plan → Identify Required Test Data → Classify Test Data → Map Data to Scenarios → Remove Duplicate Data → Generate Test Data Plan

## Test Data Identification
Identify the minimum data required per workflow: user credentials, customer/product information, search keywords, form input, upload files, dates, configuration values, reference IDs. Generate only what the application actually requires.

## Test Data Classification
Valid, invalid, boundary, empty, duplicate, special-character, business-rule, environment-specific — only categories applicable to the application.

## Data Reusability
Identify shareable data (login credentials, default customer, common search values, frequently used records) to avoid duplicate datasets for identical scenarios.

## Data Mapping
Map each dataset to its scenario, e.g.:
- Login → Valid User, Invalid User, Locked User
- Customer Creation → Valid Customer, Missing Required Fields, Duplicate Customer

## Sensitive Data Handling
Flag values that should never be hardcoded: passwords, API keys, tokens, secrets, environment URLs. Recommend external configuration or secure storage.

## Data Quality
Verify every scenario has required data, no duplicate or unnecessary datasets exist, sensitive values are identified, and business rules are supported. Keep data realistic and maintainable.

## Success Criteria
Test Data Plan generated · data mapped to scenarios · duplicates removed · shared datasets identified · sensitive data documented.

## Failure Handling
If complete test data can't be determined, document the gap, continue with remaining scenarios, and report assumptions to the Agent. Never invent business data that can't be inferred from the application.

## References
**Knowledge:** framework-rules.md, generation-patterns.md, naming-conventions.md
**Templates:** None
**Consumed by:** Framework generation, Skill 07 — Framework Validation
