# <id> — <title>

- **Issue:** <#42>
- **Status:** <Proposed | Agreed | In progress | Done | Abandoned>. The tracker is
  the truth and wins any disagreement; this field exists so a spec pasted into a
  chat is still self-describing. Nothing reads it to decide anything.
- **Source:** <where this came from — a person and a date, a support ticket, a
  review finding, a bug hit in anger. Prose citations are fine and do not need to
  resolve as links.>
- **Sized:** <a rough estimate in sessions, and what is left if part has landed.
  Past three sessions it is not a work item; split it.>
- **Priority:** <High | Medium | Low>, then the reason. The word carries the grep
  (`grep -rl 'Priority:\*\* High' docs/work/*/spec.md`), so keep it first and keep
  it one of the three; the reason is what makes it arguable rather than a queue
  position.

## Problem

<What is true today, and what that prevents. Concrete enough to be wrong about.>

## What this changes

<One paragraph. State the rule or behavior, not the implementation. A spec that
names classes has stopped being a spec.>

## Acceptance criteria

Each one checkable by someone who did not write the code. "Works correctly" is not
a criterion.

- **AC-1** — <observable condition>
- **AC-2** — <observable condition>
- **AC-3** `[Gated]` — <only a real device / real users / real load / human eyes
  can settle this. Name the check that would close it.>

## Alternatives considered

Not optional, and not a formality. This is what stops a settled dead end being
reopened in six weeks by someone who only sees the outcome. Number them; they get
cited.

1. **<The approach not taken>** — <why it lost. A rejection with no reason is
   worth nothing later, because the reason is the only part that transfers.>
2. **<Do nothing / leave it as is>** — <include the honest baseline wherever it is
   a real option, and say what makes it lose. An item whose baseline was never
   stated is an item nobody checked was worth doing.>

Alternatives that lost for reasons local to this item stay here. One whose reason
constrains work beyond this item is a decision, not a note — promote it to
`docs/DECISIONS/` (`record-decision`) and cite the ADR from this list.

## Autopsy

Bug fixes only. Written before the fix is planned; delete this section otherwise.

- **Should have been caught by:** <the test that existed and passed anyway, or
  "nothing — no test covered this path">
- **Why it wasn't:** <asserted the wrong thing / never ran / fixture made the branch
  unreachable / moved with the constant it was pinning>
- **Written down as:** <new trap in `docs/TESTING_TRAPS.md`, or the existing trap
  this is a second instance of — which makes it a `CLAUDE.md` → Gotchas line>

## Non-goals

- <What this deliberately does not touch, and who owns it instead.>

## Depends on

- <Work item that must land first, and why. Delete this section where there is no
  dependency — a page of "nothing" trains people to stop reading the heading.>

## Open questions

| # | Question | Blocks | Resolution |
|---|----------|--------|------------|
| 1 | <…> | AC-2 | <date + answer, or "open"> |

Write the answer here when it is resolved. The resolution is worth more than the
question and must not live only in a chat log.

## Divergences

Filled in during implementation, when what shipped differs from what this said.
Record both. Editing the criterion to match the code destroys the evidence that a
decision was ever made.

| Criterion | Says | Shipped | Status |
|-----------|------|---------|--------|
| AC-<n> | <wording> | <what was built> | Open point for <owner> |

---

## Which of these a small item may drop

Every heading above is cheap to keep and expensive to notice missing, but a
one-session item that files eight empty sections teaches the next reader to skim
past all of them. Keep the first five; drop the rest by deleting the heading, not
by writing "n/a" under it.

**Always:** the header block, Problem, What this changes, Acceptance criteria,
Alternatives considered.

**Where it applies:** Autopsy (bug fixes — never dropped for one, however small),
Non-goals (whenever someone might reasonably assume otherwise), Depends on,
Open questions, Divergences (starts empty; filled during Implement).

Delete everything from the horizontal rule down to here when you file.
`docs/work/EXAMPLE-042-restore-reads-last-support/` is a filled-in
reference: a real-shaped bug fix with an autopsy, four rejected alternatives, a
gated criterion, and a divergence recorded rather than edited away.
