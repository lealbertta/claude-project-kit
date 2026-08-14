# PRD — <id> <feature name>

<!--
Only for work too big for one ticket — a feature that will become several. This is
an **epic**, and `docs/WORKFLOW.md` will not run its loop on one: break it into the
child tickets below and run the loop on each. Small work does not get a PRD; it
gets a `spec.md` and nothing else.

This file is the parent. Its child tickets live in subfolders beside it:

  docs/work/003-nested-stacking/
      PRD.md              <- this file (the epic)
      042-restore/        <- one ticket
          spec.md plan.md notes.md
      043-subtree-move/
          spec.md ...
-->

- **Epic:** <#41>
- **Status:** <Proposed | Agreed | In progress | Done | Abandoned>. The tracker is
  the truth and wins any disagreement; this field exists so a PRD pasted into a
  chat is still self-describing.
- **Source:** <where this came from — a person and a date, a review, a run of
  support tickets. Prose citations are fine and do not need to resolve as links.>
- **Sized:** <rough number of tickets, and how many have landed.>
- **Priority:** <High | Medium | Low>, then the reason. Story-level priorities
  rank work *within* the feature; this one ranks the feature against everything
  else.

## Overview and problem statement

**What exists today.** <Concrete and current. This is what makes the problem
falsifiable.>

**The problem.** <What the current state prevents, in terms a user would recognize.>

**Why now.** <What changed to make this worth doing at this point and not later.
The alternatives it beat go in *Alternatives considered* below.>

## User stories

Each story is independently shippable and independently testable, and carries a
priority — the same three words a spec uses, with the reason on the same line. A
bare rank is a queue position wearing a label. Order within a priority is not
implied.

### US-1 — <short name>

> As a <user>, I want <action> so that <outcome>.

**Priority:** <High | Medium | Low> — <the reason, which is what makes it arguable
when it goes stale.>

#### FR-1 — <title>

<The rule or behavior. Not the implementation — a PRD that names classes has
stopped being a PRD.>

**Acceptance criteria**

- **AC-1.1** — <observable condition, checkable by someone who did not write the code>
- **AC-1.2** `[Gated]` — <only a real device / real users / real load / human eyes
  can settle this; name the check that would close it>

#### FR-2 — <title>

…

### US-2 — <short name>

…

### Development stories

Observability, fixtures, and escape hatches the team needs are real stories and get
lost when only end-user stories are written.

> As the <developer/owner>, I want <…> so that <…>.

<!--
Numbering is permanent. A requirement added later takes the next free number even
if it belongs logically in the middle — tickets, ADRs, and every prior comment
already reference the old ones. Never renumber, never reuse.
-->

## Non-functional requirements

- **NFR-1** — <measurable. "Fast" is not one. "The answer is a function of position
  only, never of how many times it was asked" is.>

## Alternatives considered

The shapes this feature could have taken, and why each lost. Not optional: at PRD
scale the rejected shape is the one that gets re-proposed by whoever joins next,
and re-arguing it costs more than writing it down once did. Number them; they get
cited from child tickets and from ADRs.

1. **<A different shape for the whole feature>** — <why it lost>
2. **<Buy / adopt / extend something existing instead of building>** — <why it lost>
3. **<Do nothing>** — <what happens if this is never built. Where that answer is
   "not much", say so and reconsider the feature.>

A rejection whose reason constrains work beyond this feature belongs in
`docs/DECISIONS/` as well; cite the ADR from the line here.

## Out of scope

**Deferred** — plausible later.

- <…>

**Excluded** — a different product. Saying no once, here, is cheaper than four times.

- <…>

## Open questions

| # | Question | Blocks | Owner | Resolution |
|---|----------|--------|-------|------------|
| 1 | <…> | FR-2 | <who> | <date + answer, or "open"> |

Write the answer here when it lands. The resolution is worth more than the
question and must not live only in a chat log.

## Tickets

Generated as the feature is broken down. One row per child ticket. `Status` is the
same courtesy copy as the header field above — the tracker wins any disagreement,
and nothing reads this column to decide anything.

| Item | Covers | Status |
|------|--------|--------|
| [042-restore](042-restore/) | FR-1, FR-2 | In progress |
