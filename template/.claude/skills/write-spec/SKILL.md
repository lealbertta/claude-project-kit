---
name: write-spec
description: Write the spec for a ticket, or the PRD for a feature that spans several.
---

# Write a Spec

Two sizes. Pick the smaller one that fits.

- **One ticket** — fits in one to three sessions, independently mergeable.
  Copy `docs/work/TEMPLATE/spec.md` to `docs/work/<id>-<slug>/spec.md`.
- **A feature spanning several tickets** — copy `docs/work/TEMPLATE/PRD.md` to
  `docs/work/<id>-<slug>/PRD.md`, then create child ticket folders beside it.
  That parent is an **epic**, and the `docs/WORKFLOW.md` loop does not run on one —
  it runs on each child.

Most work is the first. Reach for a PRD only when you already know the work needs
more than one branch.

`docs/work/EXAMPLE-42-restore-reads-last-support/spec.md` is a filled-in one.
Read it before the first spec on a project: it shows the level of specificity a
criterion needs, and what an alternative and a divergence look like written
honestly.

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
   implicit — an untagged gated criterion gets reported as passing. Then decide
   **here, and not later**, whether it may ship unproven: one that must not is
   `[Gated, blocks merge]`, and that ticket cannot complete until the check runs.
   Everything else becomes an acceptance question at Verify — asked when the branch
   is finished, reviewed, and everyone wants it in, which is why the call belongs
   in the spec, where the rest of the standard of done already lives.
7. Write **Alternatives considered** before the spec is called done — see below.
8. List non-goals, and mark each *deferred* or *excluded*.
9. Fill the header block. **Source** is who asked and when; **Sized** is a rough
   number of sessions; **Priority** is High/Medium/Low *with the reason on the
   same line*. Ask for what you do not know rather than inventing it — an invented
   priority is worse than a missing one, because it gets acted on.
10. Record open questions with an owner. Ask the developer for the answers you
    need now; leave the rest open rather than guessing and calling it a decision.

## Alternatives considered

The section people skip, and the one that pays. A spec that records only the
chosen approach cannot stop the rejected one being proposed again in six weeks,
and it cannot tell a later reader whether the choice is still right — because the
reason is the only part that transfers.

- Write **two or more**, numbered, each with the reason it lost. A list of one is
  a decision defending itself.
- Include the honest baseline — *do nothing* — wherever it is a real option. An
  item whose baseline was never stated is an item nobody checked was worth doing.
- A rejection is not "we preferred X". Name what it costs: the case it handles
  worse, the thing it makes unreachable, the work it doubles.

**Where it goes.** Local reasons stay in `spec.md`. A rejection whose reason
constrains work *beyond this item* is a decision — write the ADR
(`record-decision`) and cite it from the list, so the spec stays short and the
constraint is somewhere the next item will actually look.

## Priority

One of High, Medium, Low, and then the reason, on the same line. Both halves are
load-bearing: the word is what makes the set greppable, and the reason is what
makes it arguable instead of a queue position someone has to accept.

This does **not** reintroduce ordering. `PRODUCT.md` → Next stays an unordered
set, and priority is a claim about *this* item that stands on its own — a real
sequencing constraint is still a *Depends on* line and nothing else.

## When the ticket is a bug fix

One extra section, written **before** the fix is planned, under the heading
**Autopsy**. It is two lines and it is the highest-value thing in the spec:

- **Should have been caught by:** <the test that existed and passed anyway, or
  "nothing — no test covered this path">
- **Why it wasn't:** <it asserted the wrong thing / it never ran / its fixture made
  the branch unreachable / it moved with the constant it was pinning>

Then act on the second line. A failure shape not already in `docs/TESTING_TRAPS.md`
goes in it; one that *is* already there names the trap, because a trap with a second
victim means the rule is not holding where it is written — which is a
`CLAUDE.md` → Gotchas line.

Writing the regression test without answering these produces a suite that grows one
test per bug and learns nothing.

## Questions worth asking before the spec is done

- What is true today that makes this necessary now?
- What does this make *reachable* that was previously impossible? That is usually
  where the next defect comes from, and the answer goes in **Risks** — either as a
  new row in `docs/RISK_REGISTER.md` raised by this item, or as a reference to the
  row that already covers it. "Nothing new" is a fine answer once it has been asked.
- Which existing rule was written when the world was simpler than it is now?
- What is the smallest version that proves the risky part?
- How would we know this was wrong?

## Numbering

Requirement and criterion numbers are permanent. A requirement added later takes
the next free number even when it belongs logically in the middle. Tickets, ADRs,
and prior comments already reference the old numbers; renumbering silently breaks
every one of those references.
