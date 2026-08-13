---
name: sync-tickets
description: Create or update tracker issues from a spec or PRD, idempotently, and report drift.
---

# Sync Tickets

Turn a spec into tracker issues without creating duplicates, and report where the
spec and the tracker have drifted apart.

**The rule this rests on: the spec holds requirements, the tracker holds state.**
Neither writes into the other's column. Do not treat an edited issue body as a
requirement change.

### The one copy of state, and why it is allowed

A spec carries a `Status:` line anyway. It is a courtesy copy so that a spec
pasted into a chat is still self-describing, and **the tracker wins any
disagreement** — nothing reads the spec's copy to decide anything, no step is
gated on it, and a stale one is a cosmetic defect rather than a wrong answer.
That is the whole license. Anything beyond it — a due date, an assignee, a
percentage — is mirrored state that goes stale without anyone noticing, and does
not go in the spec.

Report a disagreement between the two as drift, in the same list as the rest. Do
not fix it silently, in either direction.

`Priority:` runs the other way. It is a judgement with its reason attached, so it
lives in the spec and the tracker's label is the copy — mirror it onto the issue
where you use priority labels, and report the drift when a triage session changes
one and not the other.

## The join key

Every issue created from a spec carries a label naming exactly what it covers:

```
spec:<work-id>:<requirement-id>          e.g. spec:003-stacking:FR-7
```

That label is the whole mapping. It survives renames, moves, re-parenting, and
manual edits, and it makes "which issue covers FR-7?" a one-line query. **Do not
maintain a local map file** — a hand-edited `ticket-map.json` is a third source of
truth that nobody updates when someone files an issue in the tracker's own UI, and
it conflicts on every branch.

## What maps to what

| Spec element | Tracker | Notes |
|--------------|---------|-------|
| PRD (feature) | Epic | One per feature folder |
| User story | Story | Carries the priority; independently shippable |
| Functional requirement | Task under its story | **Its acceptance criteria go in the description as the definition of done** |
| Acceptance criterion | *nothing* | An AC is how work is judged, not a unit of work |
| A requirement too big for one session | Subtasks | Only then |

Do not create a subtask per acceptance criterion. It manufactures busywork tickets
and separates verification from the thing being verified, so a task can close with
its criteria sitting in unclosed children.

## Procedure

1. Read the spec or PRD. Extract requirements in document order.
2. For each requirement, query for its key before creating anything:
   `label = "spec:<work-id>:<FR-id>"`. Found → compare. Missing → create.
3. Create with the label attached, the acceptance criteria in the description, and
   a link back to the spec file.
4. **Report, do not reconcile.** Print three lists and stop:
   - Requirements with no issue
   - Issues whose requirement no longer exists in the spec
   - Issues whose title or criteria no longer match the spec
   - Specs whose `Status:` or `Priority:` line disagrees with the tracker
5. Ask before creating or closing anything. Tracker writes are outward-facing and
   a mis-scoped run makes dozens of them.
6. Write the result to `<work-dir>/tickets.md` as a generated table, headed **"Generated
   by sync-tickets — do not edit."** Regenerate it; never hand-edit it.

## Why it only reports

Auto-reconciling a spec against a tracker overwrites deliberate human edits — a
retitled issue, a criterion tightened during triage, a task split in the UI. Those
are usually the *newer* information. Print the drift and let a human decide which
side is right.

## Before running it the first time

- Confirm the tracker connection works and you are pointed at the right project.
  A sync run against the wrong board is dozens of issues to delete by hand.
- Run it on one requirement first and look at what it produced.
