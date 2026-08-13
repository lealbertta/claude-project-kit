# PRD — <id> <feature name>

<!--
Only for work too big for one item — a feature that will become several. Small
work does not get a PRD; it gets a `spec.md` and nothing else.

This file is the parent. Its child work items live in subfolders beside it:

  docs/work/003-nested-stacking/
      PRD.md              <- this file (the epic)
      042-restore/        <- one work item
          spec.md plan.md notes.md
      043-subtree-move/
          spec.md ...
-->

- Epic: <#41>
- Status: <Proposed | Agreed | In progress | Done | Abandoned>

## Overview and problem statement

**What exists today.** <Concrete and current. This is what makes the problem
falsifiable.>

**The problem.** <What the current state prevents, in terms a user would recognize.>

**Why now, and why not <the obvious alternative>.** <The alternative someone will
raise in review, and why it loses. Writing it once here is cheaper than defending
it four times.>

## User stories

Each story is independently shippable and independently testable, and carries a
priority. Order within a priority is not implied.

### US-1 `[P1]` — <short name>

> As a <user>, I want <action> so that <outcome>.

#### FR-1 — <title>

<The rule or behavior. Not the implementation — a PRD that names classes has
stopped being a PRD.>

**Acceptance criteria**

- **AC-1.1** — <observable condition, checkable by someone who did not write the code>
- **AC-1.2** `[Gated]` — <only a real device / real users / real load / human eyes
  can settle this; name the check that would close it>

#### FR-2 — <title>

…

### US-2 `[P2]` — <short name>

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

## Work items

Generated as the feature is broken down. One row per child item.

| Item | Covers | Status |
|------|--------|--------|
| [042-restore](042-restore/) | FR-1, FR-2 | In progress |
