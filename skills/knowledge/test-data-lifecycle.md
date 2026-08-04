# TEST DATA LIFECYCLE

## Purpose
Defines how test data is kept valid across repeated runs and parallel workers. `test-data/testData.ts` (owned by skills/knowledge/output-structure.md's Test Data Output section) defines WHAT the values are; this file defines how those values survive being used more than once. Its scope is state collision — the same suite run twice, or two workers running at once, must not fail because of data left behind by an earlier execution.

## Dependency Matrix
Which Skills load this file is declared in agent/agent.md's Skill Dependency Matrix — that table is the single source of truth.

## The Problem This Solves
A test case that creates a record ("Create customer 'Acme Corp'") passes on the first run and fails on the second with "record already exists." Nothing in the framework is broken — the application is simply holding state from run one. Left unaddressed this surfaces at the very end of the pipeline as a Skill 09 execution failure, where it is expensive to diagnose and tempting to "fix" by editing the dataset. It is designed away here instead.

## Classifying Test Cases by State Effect
During Test Data Generation (Skill 06), classify every test case by what it does to application state:
- **Read-only** — navigates, searches, filters, views. No lifecycle handling needed.
- **State-creating** — creates a record, uploads a file, registers a user. Needs unique data (below) and teardown.
- **State-mutating** — edits or deletes an existing record. Needs a known starting state, and must not depend on another test having created it.

Record this classification alongside the Test Data Mapping so Skill 07 knows which specs need teardown wiring and Skill 09 knows which failures are legitimately data-related.

## Unique Data Generation (Critical)
For a **state-creating** test case, any supplied value that the application enforces as unique (name, email, username, reference number, code) must be made unique per execution rather than written as a fixed literal.

Express this as a factory function in `test-data/testData.ts`, not as a mutated constant:
```typescript
export const customerNamePrefix = 'Acme Corp';

export const buildUniqueCustomerName = (): string =>
    `${customerNamePrefix}-${Date.now()}`;
```

**The uniqueness suffix is the only thing generation may add.** The supplied value stays recognizably intact as the prefix/base — this is a collision-avoidance mechanism, not permission to invent business data, and it does not weaken skills/knowledge/framework-rules.md's Critical Rule 1 or Skill 06's Primary Source of Truth rule.

**Never uniquify:**
- Credentials, or any value used to authenticate.
- A value the test case's `expectedResult` asserts on exactly (uniquifying it would silently change what's being verified).
- Data for a **negative** test case whose whole point is the supplied value (e.g. a duplicate-record test that *must* reuse an existing name to trigger the error).

When a value cannot be uniquified for one of these reasons but its test case still creates state, teardown below is mandatory rather than optional.

## Teardown
A state-creating or state-mutating test must leave the application as it found it. Prefer, in this order:
1. **Application-level cleanup** — call the Page Object's own delete/cancel method in an `afterEach()`, so cleanup exercises supported behaviour.
2. **Shared teardown hook** — when two or more specs need the same cleanup, extract it to `hooks/` per skills/knowledge/framework-architecture.md's Reusable Logic Placement section.
3. **Fixture-scoped cleanup** — when setup and teardown belong together (a fixture provisions a record and removes it after), express both in one `fixtures/` fixture.

Teardown must never contain assertions, and a teardown failure must never be swallowed to make a test appear green — that would violate skills/knowledge/framework-rules.md's Critical Rule 8. If cleanup cannot be performed, report it.

Cleanup applies only to data the suite itself created. **Never delete pre-existing application data** to force a clean starting state.

## Isolation Under Parallel Execution
Skill 09 runs the suite with concurrent workers (see its Execution Concurrency section), so two tests may touch the application simultaneously. Consequently:
- **Never share a mutable record between tests.** Each state-creating test creates its own data via a factory function; it must not consume a record another test created.
- **Never assume execution order.** BP-602 already requires test independence; under parallelism this extends to data — a test may not depend on another test's cleanup having run first.
- **Read-only tests may share fixtures freely** (e.g. an authenticated session), since they don't mutate state.
- If two test cases genuinely cannot run concurrently against the same account or record, mark that group serial rather than adding waits or retries to paper over the race.

## Relationship to Skill 09's Repair Loop
skills/skill-09-test-execution-self-healing.md lists "test data conflict" as a repair-routing root cause. That route exists for datasets that were wrong when generated — not as the standard answer to collisions. When Skill 09 diagnoses a failure as a data collision on a state-creating test case, the correct repair is to introduce the missing factory function or teardown described here, routed back to Skill 06 (data) and/or Skill 07 (teardown wiring) — not to hand-edit a literal in `testData.ts` so this one run goes green. A repair that only changes a literal leaves the same failure waiting for the next run.

## Reporting
Skill 06 documents, alongside the Test Data Mapping: which test cases are state-creating/mutating, which values were uniquified (and how), which need teardown, and any case where uniqueness could not be guaranteed. Skill 08 validates that every state-creating test case has either unique data or teardown, and Skill 10 reports any that has neither as a maintainability risk.

## Final Checklist
✓ Every test case classified by state effect · ✓ uniqueness-constrained values on state-creating cases generated by a factory, not a fixed literal · ✓ credentials, asserted values, and negative-case data never uniquified · ✓ state-creating/mutating tests have teardown · ✓ teardown contains no assertions and swallows no failures · ✓ no pre-existing application data deleted · ✓ no mutable record shared between tests · ✓ no test depends on another's execution order or cleanup · ✓ lifecycle decisions documented for the coverage report
