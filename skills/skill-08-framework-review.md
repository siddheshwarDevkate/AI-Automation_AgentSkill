# SKILL 08 — FRAMEWORK REVIEW

## Purpose
Perform the final architectural and quality review of the generated framework. Acts as the final quality gate and determines production readiness. Does NOT generate or modify framework code.

## Execution Dependencies
**Knowledge:** framework-architecture.md, framework-rules.md, generation-patterns.md, output-structure.md, playwright-best-practices.md
**Templates:** framework-output-template.md

## Inputs
Required: Generated Framework, Validation Report, user requirements, project standards.
Optional: existing framework, previous review reports.

## Expected Output
- Framework Review Report
- Overall assessment
- Risks
- Improvement recommendations
- Final readiness decision

## Responsibilities
Review the complete framework end to end: architecture, maintainability, scalability, consistency, reusability. Identify improvement opportunities and determine production readiness, per the final quality checklist in framework-output-template.md.

## Workflow
Read Generated Framework → Read Validation Report → Review Architecture → Review Maintainability → Review Reusability → Review Completeness → Identify Risks → Generate Review Report → Determine Production Readiness

## Architecture Review
Verify clear separation of responsibilities, logical folder structure, consistent organization, proper dependency flow, modular implementation.

## Maintainability Review
Assess readability, reusability, scalability, consistency, and simplicity. Flag areas that may become difficult to maintain.

## Consistency Review
Check naming, method style, file/feature organization, and coding approach — the framework should read as if written by a single engineer.

## Reusability Review
Identify opportunities for shared business methods, shared Page Object patterns, shared test data, shared verification/helper logic. Flag unnecessary duplication.

## Risk Assessment
Identify risks: missing business coverage, incomplete implementation, tight coupling, duplicate logic, maintainability/scalability concerns. Classify by impact.

## Improvement Recommendations
Recommend better reuse, organization, simplification, readability, or reduced duplication — without changing intended behaviour.

## Final Readiness Decision
Choose one: **Production Ready** · **Production Ready with Minor Improvements** · **Requires Review Before Production** · **Not Ready for Production**, with a concise explanation.

## Success Criteria
Framework reviewed · Validation Report assessed · risks identified · recommendations documented · final readiness determined.

## Failure Handling
If the review can't be completed, document the limitation, report unavailable information, and complete the review using only verified artifacts. Never approve a framework without sufficient evidence.

## References
**Knowledge:** framework-rules.md, framework-architecture.md, generation-patterns.md, naming-conventions.md
**Templates:** framework-output-template.md
**Consumed by:** Agent — final delivery decision
