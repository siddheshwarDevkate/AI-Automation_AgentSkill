---
name: skill-10-framework-review.md
description: Make the final production-readiness decision, weighted by actual execution results rather than static validation alone, and generate the review report and README.
---

# SKILL 10 — FRAMEWORK REVIEW

## Purpose
Perform the final architectural and quality review of the generated framework, including final sign-off on Test Case coverage AND actual execution results. Acts as the final quality gate and determines production readiness. Does NOT generate or modify framework code.

## Execution Dependencies
Load the Knowledge files and Template listed for Skill 10 in .claude/agent/agent.md's **Skill Dependency Matrix** — that table is the single source of truth and is deliberately not restated here.

## Inputs
Required: Generated Framework, Validation Report (including the Test Case Traceability Matrix), Execution Report (from Skill 09, including per-browser pass/fail/blocked status and Re-Validation Gate outcome), user requirements, project standards.
Optional: existing framework, previous review reports.

**Use the post-repair Validation Report.** If Skill 09 repaired code, its Re-Validation Gate produced a replacement Validation Report describing the framework as it now stands. Read that one. If the Execution Report shows repairs modified code but records no Re-Validation Gate outcome, the gate was skipped — treat that as missing evidence per Execution Results Review below, not as a pass.

## Expected Output
- `README.md` (generated if it doesn't already exist — see README Generation below)
- Framework Review Report
- Overall assessment
- Test Case Coverage Summary (X of Y test cases automated; list of any "Not Automated" with reason)
- Execution Summary (X of Y automated test cases Verified Passing across the full browser matrix; list of Blocked/flaky/application-defect cases)
- Risks
- Improvement recommendations
- Final readiness decision

## Responsibilities
Review the complete framework end to end: architecture, maintainability, scalability, consistency, reusability, Test Case coverage, AND real execution results. Identify improvement opportunities and determine production readiness, per the final quality checklist in .claude/skills/templates/framework-output-template.md. Generate the framework's `README.md` as part of final delivery.

## README Generation
Generate `README.md` (or update it if one already exists) summarizing: what the framework covers, how many test cases are automated, how to run the suite, and the current Test Case Coverage / Execution Summary numbers. Keep it factual and derived only from this run's Validation Report and Execution Report — never describe coverage or pass rates the framework hasn't actually achieved.

## Workflow
Read Generated Framework → Read Validation Report → Read Execution Report → Review Test Case Coverage → Review Execution Results → Review Architecture → Review Maintainability → Review Reusability → Review Completeness → Identify Risks → Generate Review Report → Generate/Update README.md → Determine Production Readiness

## Test Case Coverage Review
Using the Validation Report's Test Case Traceability Matrix, confirm:
- Every supplied test case is either automated or explicitly documented as "Not Automated" with a clear reason.
- No test case was silently dropped.
- No generated test exists outside the supplied test case set.

Report coverage as "X of Y test cases automated" and list every unautomated test case with its blocking reason.

## Execution Results Review (Critical)
This is the decisive input Skill 08's static validation cannot provide: whether the framework actually works. Using Skill 09's Execution Report, confirm:
- Every automated test case has a final per-browser status: **Verified Passing**, **Blocked — Manual Review Required**, or **Application Defect**.
- Every repair Skill 09 performed was scoped correctly (no assertion weakened, no test data altered to mask a real failure — per the Strict Repair Rule in .claude/skills/skill-09-test-execution-self-healing.md).
- No test is reported as passing without having actually been executed across the full browser matrix.
- **The reported results come from Skill 09's final full-suite confirmation run**, not carried forward from a canary, a batch, or a repair re-run. Skill 09's staged strategy makes partial runs normal *during* healing; only the final run describes the delivered code. A report whose rows were assembled from batch results across different code states is not evidence the suite passes together — treat it as missing evidence.
- **The Re-Validation Gate ran if any repair modified code**, and its findings were resolved or documented. A repaired framework reviewed only against its pre-repair Validation Report has not actually been validated in its delivered state.

A framework whose tests were generated but never executed, whose failures were "fixed" by weakening assertions, or whose repairs were never re-validated, must never be marked Production Ready — this Skill exists specifically to catch that.

## Architecture Review
Verify clear separation of responsibilities, logical folder structure, consistent organization, proper dependency flow, modular implementation.

## Maintainability Review
Assess readability, reusability, scalability, consistency, and simplicity. Flag areas that may become difficult to maintain.

## Consistency Review
Check naming, method style, file/feature organization, and coding approach — the framework should read as if written by a single engineer.

## Reusability Review
Identify opportunities for shared business methods, shared Page Object patterns, shared test data, shared verification/helper logic. Flag unnecessary duplication.

## Risk Assessment
Identify risks: unautomated test cases, tests still Blocked after the repair budget, application defects discovered during execution, incomplete implementation, tight coupling, duplicate logic, maintainability/scalability concerns. Classify by impact.

## Improvement Recommendations
Recommend better reuse, organization, simplification, readability, or reduced duplication — without changing intended behaviour.

## Final Readiness Decision
Choose one, weighted primarily by the Execution Summary:
- **Production Ready** — all automated test cases Verified Passing across the full browser matrix, no unresolved framework defects.
- **Production Ready with Minor Improvements** — all automated test cases Verified Passing, but with non-blocking maintainability/style recommendations, or a small number of "Not Automated" cases with legitimate (e.g. app doesn't support it) reasons.
- **Requires Review Before Production** — one or more test cases Blocked or flagged as an Application Defect; needs human review before shipping.
- **Not Ready for Production** — significant execution failures, broken traceability, repairs that violated the Strict Repair Rule, or code-modifying repairs that were never re-validated.
- **Generated, NOT Verified** — the user explicitly overrode a failed Skill 00 readiness gate, so the suite was never executed. State plainly that no test has been run and that coverage numbers describe generated code only. This framework can never be reported as Production Ready, regardless of how clean the static review is.

Always state the decision alongside both the Test Case coverage and Execution Summary numbers, e.g. "18 of 20 test cases automated; 17 of 18 Verified Passing on Chromium/Firefox/WebKit; 1 Blocked (TC-014, see Execution Report)."

## Success Criteria
Framework reviewed · Validation Report assessed · Execution Report assessed · Test Case coverage summarized · execution results summarized · risks identified · recommendations documented · final readiness determined.

## Failure Handling
If the review can't be completed, document the limitation, report unavailable information, and complete the review using only verified artifacts. Never approve a framework without sufficient evidence, never report full coverage without confirming it against the Test Case Traceability Matrix, and never report "Production Ready" without confirming actual passing execution results from Skill 09.

## Consumed By
Agent — final delivery decision
