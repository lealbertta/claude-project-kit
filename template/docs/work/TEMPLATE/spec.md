# <id> — <title>

- Issue: <#42>
- Status: <Proposed | Agreed | In progress | Done | Abandoned>

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

- <Work item that must land first, or "nothing">

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
