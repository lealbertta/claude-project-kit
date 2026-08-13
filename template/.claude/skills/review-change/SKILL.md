---
name: review-change
description: Independent correctness, test-strength, and scalability review of a change.
---

# Review a Change

Use a fresh subagent or a clean session. Self-review in the session that wrote the
code does not satisfy this step — you will re-derive the same assumptions.

Review the diff against the accepted plan, the acceptance criteria, and the
relevant ADRs.

## Check the code

- Incorrect behavior, or an acceptance criterion nothing actually satisfies
- Data incompatibility or a missing migration
- Broken transaction, undo, or batch boundaries
- Nondeterminism: iteration order, timing, hash ordering, floating-point drift
- Hidden global state, ambient lookups, circular module dependencies
- Allocations, synchronous IO, or unbounded loads in hot paths
- Resource leaks and duplicated shared dependencies
- Lifecycle, interruption, and resume regressions
- Unauthorized scope expansion or dependency/configuration changes

## Check the tests — this is the half that gets skipped

For every test in the diff, ask what implementation mutation it would survive.
Run the mutations you can. Consult `docs/TESTING_TRAPS.md` first; the specific
shapes to look for are:

- The rule is tested directly but the **production wiring** is tested by nothing
- The assertion is expressed in terms of the constant it is supposed to pin
- The fixture makes the branch under test unreachable, so it never runs
- Both sides of a comparison move together, so the assertion cannot fail
- A sequence is compared after collapsing it, hiding a whole class of wrong behavior
- A counting assertion has only one half ("X is zero") and passes for a component
  that does nothing at all
- A branch inert on current data is claimed as covered rather than recorded as inert

A passing suite that survives a deleted feature is a finding, and usually a larger
one than any bug in the diff.

## Report

Findings with file and line references, severity, evidence, and a concrete
correction. Separate blockers from optional improvements. Do not report personal
style preferences as blockers. If you found nothing material, say so plainly
rather than manufacturing findings.
