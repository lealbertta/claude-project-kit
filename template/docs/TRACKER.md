# Tracker and board

## What belongs here

The two systems the loop writes to — the issue tracker and the project board — and
the rule that keeps them from disagreeing with the repo. Filled in once at setup and
read rarely after that, which is why it is not in `WORKFLOW.md`.

## What does not belong here

How a ticket gets done (`WORKFLOW.md`), how one gets chosen (`pick-ticket`), or the
commands this project runs (`CLAUDE.md` → Commands). The board moves themselves are
stated in the stage that makes them; this file collects them and says who owns what.

**Without a tracker, only *One owner per fact* applies** — and in that configuration
it is what tells you the spec's `Status:` line is the status rather than a copy.

## Project configuration

<!-- FILL IN ONCE. Everything else in this file is generic. -->

| Setting | Value |
|---------|-------|
| Repository | `<owner>/<repo>` |
| Ticket label | `<task>` |
| Blocked label | `<blocked>` |
| Gated label | `<gated>` — on the issue carrying a check that shipped outstanding |
| Priority labels | `<p-high>`, `<p-medium>`, `<p-low>` — the copy; the spec's `Priority:` line holds the reason |
| Board | `<https://github.com/users/<owner>/projects/<N>>` |
| Board project ID | `<PVT_...>` |
| Status field ID | `<PVTSSF_...>` |
| Status option IDs | Todo `<id>`, In progress `<id>`, In review `<id>`, Done `<id>` |

The test and verification commands are **not** here. `CLAUDE.md` → Commands owns
them, and a second copy is the one that goes stale.

## Board state

The tracker has to show where the work actually is. Two moves per ticket, both the
assistant's to make, and each is stated in the stage that makes it:

| When | Status | Stated in |
|------|--------|-----------|
| The ticket is claimed — in the same step that assigns the issue | `In progress` | `pick-ticket`, or `WORKFLOW.md` → Plan |
| Start of **Verify** — checks passed, PR opened, diff going to both reviewers | `In review` | `WORKFLOW.md` → Verify, step 2 |
| Closing the issue | `Done`, **automatically** | nothing — the automation does it |

**Never set `Done` by hand** — closing the issue does it, and doing both is a way
to get them out of sync.

That automation is also why these steps are easy to skip: the end of a cycle
always looks right on the board while the middle shows nothing. `In review`
matters most and is easiest to lose — a ticket that closes with its review still
outstanding reads as `Done` and nowhere says otherwise.

## One owner per fact

The board is a third place state can live, alongside the issue and this repo. It
does not get to hold a copy of anything: **each fact has exactly one owner, chosen
by who edits it, and is never mirrored.**

| Fact | Owner | Why there |
|------|-------|-----------|
| Status | the board's Status field | written by the automation and the two moves above, and nowhere else |
| Priority | **one** of the board field or the label — never both | whichever you actually sort by; the *reason* stays in prose in `spec.md` |
| Blocked | the same single choice — a status option or a label | a field and a label for one fact disagree within a month |
| Requirements, criteria | `spec.md` | the standard the reviewer scores against |
| Plan, notes | `plan.md`, `notes.md` | versioned with the diff they describe |
| Review findings | the **PR**, after consolidation | anchored to the lines that caused them, and outlives the `docs/work/` folder that does not |

The kit's default is that the spec owns priority and the label is the copy. **With a
board in use, flip it:** the field owns it, because a triage session changes the
field and touches nothing else, and the label goes away unless you genuinely filter
on labels from the CLI. What survives in the spec is the sentence explaining the
priority, which is the half no field can hold.

**The spec's own `Status:` line flips owner with the configuration, and this is the
authority on which.** With a tracker installed, the tracker owns it and the spec
line is a copy — written by `sync-tickets` refreshing it from the tracker and by
nothing else, never by hand, and never gated on, because gating on a copy checks the
copy rather than the fact. **With no tracker there is no other owner**, so the spec
line *is* the status: `Agreed` at the end of Plan, `In progress` when Implement
starts, `Done` or `Abandoned` at close — the same three moves the board would have
made — and the reviewer's closing check reads it, because in that configuration it
is the only thing to read.

## Setting the status

The field is on the *project item*, not on the issue, so it takes a lookup first.

```sh
# --limit must exceed the board's TOTAL item count, Done ones included. The default
# of 100 is a silent failure on any board with history: the item is simply absent.
ITEM_ID=$(gh project item-list <N> --owner <owner> --limit 2000 --format json \
  --jq '.items[] | select(.content.number == <ISSUE>) | .id')

[ -n "$ITEM_ID" ] || { echo "issue <ISSUE> is not an item on project <N>"; exit 1; }

gh project item-edit --id "$ITEM_ID" --project-id <PROJECT_ID> \
  --field-id <STATUS_FIELD_ID> --single-select-option-id <OPTION_ID>
```

**An empty lookup is a stop, not a skip.** It means the issue was never added to
the board, and continuing quietly is how a ticket runs its whole cycle without the
board ever showing it.

## What the board can hide

Three states where the board and the work disagree and nothing reports it:

- **A draft item** has no issue behind it, so it has no `spec:` label, no repo
  folder, and no way for any of this to see it. Work triaged into drafts is
  invisible to the loop — promote it to an issue before it is picked up.
- **An archived item** leaves the board while its issue stays open. The board then
  fails to show work that is in flight, which is the exact failure this section
  exists to prevent. Archive on close, never before.
- **A second project.** An issue can sit on several boards. The configuration table
  names one, and it is the one the moves above write to; the others are somebody
  else's view and are not kept in step.

## Before the first run

Three checks, once, when the board is configured:

- **The `Done` automation is actually on.** Projects enables *item closed → Done*
  and *PR merged → Done* by default, but they can be switched off — and if they are,
  "never set `Done` by hand" leaves every ticket parked in `In review` forever.
- **The token can write project fields.** Projects is GraphQL-only and needs
  `project` scope. `GITHUB_TOKEN` in Actions does not have it and fine-grained PATs
  are patchy, so a board move that works from your machine can fail in CI with an
  error that reads like a missing project.
- **The field and option IDs are current.** Renaming a status option keeps its ID;
  deleting and recreating one mints a new ID, and the configuration table above then
  points at a value nothing will ever match. Re-run `gh project field-list` after
  any change to the field itself.
