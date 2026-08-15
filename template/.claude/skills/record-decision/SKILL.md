---
name: record-decision
description: Write an ADR for a durable decision, or amend an existing one.
---

# Record a Decision

Write an ADR when a choice will still constrain work in six months: an invariant,
a contract, a technology commitment, a rule that says "never do X".

Do **not** write one for a choice that the code already explains, or one that the
next person can reverse in an afternoon without consequence.

## ADR or ticket?

Four places hold plan-level state, and **the boundary between them is what stops any
of them rotting.** Put a thing in the wrong one and it is either edited when it
should have been amended, or repeated in five places until the copies disagree.

| Where | Holds | Changes |
|-------|-------|---------|
| `docs/PRODUCT.md` | what the product *is* — pillars, baseline, what is not built | rarely, and it is the human's call |
| `docs/DECISIONS/` | decisions that constrain work not yet filed | append-only — amend, never rewrite |
| `docs/work/<id>-<slug>/` | one unit of work: its spec, plan, notes | constantly; the tracker holds its status |
| `.claude/rules/`, `CLAUDE.md` | the *rule* a decision implies, where the agent will meet it | in the same change as its ADR |

**The test for ticket versus ADR:** *would this reasoning need repeating in a ticket
that does not exist yet?* If yes, it is an ADR. A spec's **Alternatives
considered** explains why *this* work was done this way and is complete on its own;
an ADR explains a rule that binds work nobody has scoped. "We rejected a shared
dialog for this one tool" belongs in the spec. "Every destructive action gets its own
confirmation" is an ADR.

**When in doubt, write the ticket.** An ADR that only ever constrained one ticket is
permanent clutter in an append-only record; a ticket whose reasoning turns out to
bind everything can be promoted to an ADR later, in five minutes. The asymmetry decides it.

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
8. **Write the rule where the agent will meet it, in the same change.** The ADR holds
   the reasoning; the rule it implies goes in `.claude/rules/` if it governs one
   subsystem, or `CLAUDE.md` → Non-negotiables if it binds everywhere. Nothing loads
   `docs/DECISIONS/` by default, so **an ADR that decides something and briefs nobody
   has not landed** — it will be violated by the next session, which had no reason to
   open the directory. State the rule there and link the ADR for the reasoning; do
   not restate the argument in both places.

## Amend, don't rewrite

When reality diverges from a recorded decision, the record gains a line; it does
not get edited to match. The pair — what was decided, what actually happened — is
the thing with value. Editing the first to match the second leaves a document that
looks like it was right all along and teaches nobody anything.
