---
name: write-spec
description: Write the spec for a work item, or the PRD for a feature that spans several.
---

# Write a Spec

Two sizes. Pick the smaller one that fits.

- **One work item** — fits in one to three sessions, independently mergeable.
  Copy `docs/work/TEMPLATE/spec.md` to `docs/work/<id>-<slug>/spec.md`.
- **A feature spanning several items** — copy `docs/work/TEMPLATE/PRD.md` to
  `docs/work/<id>-<slug>/PRD.md`, then create child item folders beside it.

Most work is the first. Reach for a PRD only when you already know the work needs
more than one branch.

## Procedure

This is written **with** the developer, not for them. Ask before assuming.

1. Read `docs/PRODUCT.md` — vision, pillars, and what is deliberately not built.
   A request that serves no pillar is a "no" by default; say so rather than
   speccing it.
2. Read the relevant ADRs. A spec that contradicts a recorded decision needs the
   decision amended first, not quietly ignored.
3. Draft the problem statement first and confirm it before writing requirements.
   Most bad specs are correct answers to a problem nobody had.
4. Write requirements as rules, not implementations. If a line names a class, a
   function, or a file, it belongs in `plan.md` instead.
5. Write acceptance criteria that someone who did not write the code could check.
   "Works correctly" is not one.
6. Tag `[Gated]` every criterion only a real device, real users, real load, or
   human eyes can settle, and name the check that would close it. Do not leave it
   implicit — an untagged gated criterion gets reported as passing.
7. List non-goals, and mark each *deferred* or *excluded*.
8. Record open questions with an owner. Ask the developer for the answers you
   need now; leave the rest open rather than guessing and calling it a decision.

## Questions worth asking before the spec is done

- What is true today that makes this necessary now?
- What does this make *reachable* that was previously impossible? That is usually
  where the next defect comes from.
- Which existing rule was written when the world was simpler than it is now?
- What is the smallest version that proves the risky part?
- How would we know this was wrong?

## Numbering

Requirement and criterion numbers are permanent. A requirement added later takes
the next free number even when it belongs logically in the middle. Tickets, ADRs,
and prior comments already reference the old numbers; renumbering silently breaks
every one of those references.
