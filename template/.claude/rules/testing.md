---
paths:
  - "<tests-dir>/**/*"
  - "<ci-scripts-dir>/**/*"
---

# Testing Rules

- Prefer fast, isolated tests for deterministic rules; reserve integration and
  end-to-end tests for wiring, lifecycle, and boundaries that unit tests cannot reach.
- Tests must be deterministic and independent of execution order.
- Do not use arbitrary time delays when a state, event, or completion signal can
  be awaited explicitly.
- Every bug fix includes a failing regression test written *before* the fix, when
  the behavior is testable.
- **A test is not done when it passes. It is done when it fails on a mutation.**
  Remove the behavior, invert the comparison, or delete the constant, and confirm
  the test goes red. Report which mutations were checked. Read
  `docs/TESTING_TRAPS.md` before writing tests for a new rule.
- **Pin the production wiring, not only the rule.** A rule proven by direct unit
  tests can be deleted from the code path that calls it with the whole suite still
  green. At least one test must exercise the composed path.
- Use fixtures for persisted schemas, external payloads, and representative data
  shapes. Never regenerate a fixture to make a test pass — a fixture failure is a
  migration to write.
- Maintain seeded generators so performance comparisons use identical inputs.
- Do not weaken assertions, skip tests, or swallow exceptions to make a test pass.
- Report the exact test command, result location, and failure summary.
- Some acceptance cannot be bought in the test suite — real hardware, real
  network, real users, visual judgement. Mark those criteria explicitly and name
  the check that would close them. Never record one as passing.
