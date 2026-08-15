---
name: sync-tickets
description: Create or update tracker issues from a spec or PRD, idempotently, and report drift.
---

# Sync Tickets

Turn a spec into tracker issues without creating duplicates, and report where the
spec and the tracker have drifted apart.

**What this creates, and what it does not.** A single ticket's issue is filed by
`write-spec` before its folder exists, because the folder is named after it — so
there is nothing here to create for one, and running this against a lone `spec.md`
is a drift report and nothing else. What it creates is a **PRD's** children: one
issue per requirement, under the epic issue that already exists for the same reason.

**The rule this rests on: the spec holds requirements, the tracker holds state.**
Neither writes into the other's column. Do not treat an edited issue body as a
requirement change.

### The one copy of state, and why it is allowed

A spec carries a `Status:` line anyway. With a tracker installed it is a courtesy
copy so that a spec pasted into a chat is still self-describing, and **the tracker
wins any disagreement** — no step is gated on it, and a stale one is a cosmetic
defect rather than a wrong answer. That is the whole license. Anything beyond it —
a due date, an assignee, a percentage — is mirrored state that goes stale without
anyone noticing, and does not go in the spec.

**This skill is the copy's only writer.** Refresh it from the tracker and report
that you did, in the same list as the rest. Repairing it is safe in a way repairing
the rest is not: the tracker already wins, so the copy holds nothing that can be
lost. One writer is also what keeps it from being a mirror in the sense
`docs/WORKFLOW.md` → One owner per fact forbids — the fact has one owner, and this
is the one thing that propagates it.

`Priority:` is the opposite and is **reported, never repaired**: the spec holds the
reason as well as the word, so overwriting it destroys the half no field can hold.

**With no tracker there is nothing to sync and none of this applies** — the spec's
line is the status itself, moved by the stages. `docs/WORKFLOW.md` → One owner per
fact is the authority on both configurations.

`Priority:` runs the other way. It is a judgement with its reason attached, so it
lives in the spec and the tracker's label is the copy — mirror it onto the issue
where you use priority labels, and report the drift when a triage session changes
one and not the other.

**Where a project board is in use, priority flips again**, and this skill is not
the authority on it: `docs/WORKFLOW.md` → One owner per fact says the board's
Priority field owns the value and the label goes away. What stays in the spec is
the sentence explaining the priority, which no field can hold. Read that section
before mirroring a priority anywhere.

## The join key

Every issue created from a spec carries a label naming exactly what it covers:

```
spec:<ticket>:<requirement-id>          e.g. spec:42:FR-7
```

`<ticket>` is the issue number of the spec or PRD the requirement came from —
unpadded, and **never the folder slug**. `.claude/rules/documentation.md` → Names
and references is the authority on that; the short version is that a key carrying a
slug breaks the moment someone rewords a folder name, which is exactly the failure
this label is supposed to be immune to.

That label is the whole mapping. It survives issue renames, moves, re-parenting,
folder renames, and manual edits, and it makes "which issue covers FR-7?" a one-line
query. **Do not maintain a local map file** — a hand-edited `ticket-map.json` is a
third source of truth that nobody updates when someone files an issue in the
tracker's own UI, and it conflicts on every branch.

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
   `label = "spec:<ticket>:<FR-id>"`. Found → compare. Missing → create.
3. Create with the label attached, the acceptance criteria in the description, and
   a link back to the spec file.
4. **Report, do not reconcile — with one exception.** Print these five lists and
   stop:
   - Requirements with no issue
   - Issues whose requirement no longer exists in the spec
   - Issues whose title or criteria no longer match the spec
   - Specs whose `Priority:` line disagrees with the tracker — reported, never
     repaired, because the spec holds the reason as well as the word. Specs whose
     `Status:` line disagrees are **repaired from the tracker and listed as
     repaired**: that copy has no owner but this skill, and the tracker already
     wins, so there is nothing in it to lose
   - **Open issues that are not items on the configured board**, and board items
     whose Status contradicts the issue's own state — a closed issue sitting in
     `In review`, an open one marked `Done`

   That fifth list needs a separate call. **Project fields are invisible to
   `gh issue view`** — they live on the project item, not the issue, so a report
   built only from issue data will confidently tell you everything agrees while the
   board says something else entirely.

   ```sh
   gh project item-list <N> --owner <owner> --limit 2000 --format json \
     --jq '.items[] | {number: .content.number, status: .status, title: .title}'
   ```

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
