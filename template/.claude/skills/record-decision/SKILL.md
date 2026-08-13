---
name: record-decision
description: Write an ADR for a durable decision, or amend an existing one.
---

# Record a Decision

Write an ADR when a choice will still constrain work in six months: an invariant,
a contract, a technology commitment, a rule that says "never do X".

Do **not** write one for a choice that the code already explains, or one that the
next person can reverse in an afternoon without consequence.

## Procedure

1. Read `docs/DECISIONS/` first. Most "new" decisions amend an existing one.
2. Copy `docs/DECISIONS/0000-adr-template.md` to the next number.
3. State the decision precisely enough to be violated. "Use a layered
   architecture" cannot be violated; "domain code must not reference the
   presentation module" can.
4. Record the alternatives *with the reason each lost*. An alternatives list with
   no reasoning is why the same debate happens again next quarter.
5. Fill in **Revisit trigger** with a measurable condition. A decision with no
   revisit trigger becomes folklore.
6. Fill in **Compatibility and migration**, even when the answer is "none" — with
   the reason.
7. If this amends an earlier ADR, say so **in both files** and be specific about
   which clause. "Amends ADR-0004 decision 2" is useful; "supersedes ADR-0004" is
   usually false and throws away the parts still in force.
8. Update the invariant in `CLAUDE.md` in the same change if the decision changed one.

## Amend, don't rewrite

When reality diverges from a recorded decision, the record gains a line; it does
not get edited to match. The pair — what was decided, what actually happened — is
the thing with value. Editing the first to match the second leaves a document that
looks like it was right all along and teaches nobody anything.
