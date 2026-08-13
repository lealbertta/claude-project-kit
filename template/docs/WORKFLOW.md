# Workflow

Plan → Implement → Verify, once per work item.

A **work item** is one unit of work: one issue, one branch, one plan, one review,
one merge. It has acceptance criteria you can check, fits in one to three
sessions, and is independently mergeable. "Add stacking" is too big. "Rename this
variable" is too small — just do it.

Work items are **not sequenced globally.** Pick whatever is most useful from
`PRODUCT.md` → Next. Where one genuinely depends on another, its `spec.md` says so
under *Depends on*; nothing else implies an order.

Each item owns a directory:

```
docs/work/042-restore-reads-last-support/
    spec.md     what and why, acceptance criteria     (before Plan)
    plan.md     the agreed plan                       (end of Plan)
    notes.md    findings, deferrals, mutations        (during Implement/Verify)
```

Copy `docs/work/TEMPLATE/` to start one. The number matches the issue.
`docs/work/EXAMPLE-042-restore-reads-last-support/` is the same three files filled
in, and is the faster way to see what is expected of each.

### When the work is bigger than one item

A feature that will take several branches gets a parent folder with a `PRD.md` —
user stories, numbered requirements, acceptance criteria — and its work items as
subfolders beside it:

```
docs/work/003-nested-stacking/
    PRD.md              the feature: stories, requirements, criteria
    042-restore/        one work item
        spec.md plan.md notes.md
    043-subtree-move/
```

Nothing else changes: the loop below still runs per work item. Use `write-spec` to
draft either size, and `sync-tickets` to turn requirements into issues — epic per
feature, story per user story, task per requirement with its criteria as the
definition of done. Acceptance criteria never become tickets of their own.

## Project configuration

<!-- FILL IN ONCE. Everything else in this file is generic. -->

| Setting | Value |
|---------|-------|
| Repository | `<owner>/<repo>` |
| Work-item label | `<task>` |
| Blocked label | `<blocked>` |
| Priority labels | `<p-high>`, `<p-medium>`, `<p-low>` — the copy; the spec's `Priority:` line holds the reason |
| Board | `<https://github.com/users/<owner>/projects/<N>>` |
| Board project ID | `<PVT_...>` |
| Status field ID | `<PVTSSF_...>` |
| Status option IDs | Todo `<id>`, In progress `<id>`, In review `<id>`, Done `<id>` |
| Test command | `<./scripts/test.sh>` |
| Full verification | `<./scripts/verify.sh>` |

## Picking up work

```sh
gh issue list --state open --label <task> --limit 100 \
  --json number,title,labels,assignees \
  --jq 'map(select(([.labels[].name] | index("<blocked>") | not) and (.assignees | length) == 0))'
```

That returns the eligible set, not a queue — choose from it. Confirm the choice
with the human before starting, then assign yourself.

Each item's `spec.md` carries a **Priority** with the reason on the same line.
Read the reasons of the high ones before choosing; they are claims, and a claim
written three weeks ago is often no longer true. Priority narrows the set — it
does not order it, and it never overrides a *Depends on*.

```sh
grep -rl 'Priority:\*\* High' docs/work/*/spec.md
```

That works with no tracker at all, which is the point: it is the fallback for the
scaled-down setup where this file is not installed.

## Board state

The tracker has to show where the work actually is. Two moves per item, both the
assistant's to make:

| When | Status |
|------|--------|
| Start of **Plan**, in the same step that assigns the issue | `In progress` |
| Start of **Verify** — tests running, diff going to the reviewer | `In review` |
| Closing the issue | `Done`, **automatically** |

```sh
gh project item-list <N> --owner <owner> --limit 100 --format json \
  --jq '.items[] | select(.content.number == <ISSUE>) | .id'

gh project item-edit --id <ITEM_ID> --project-id <PROJECT_ID> \
  --field-id <STATUS_FIELD_ID> --single-select-option-id <OPTION_ID>
```

**Never set `Done` by hand** — closing the issue does it, and doing both is a way
to get them out of sync.

That automation is also why these steps are easy to skip: the end of a cycle
always looks right on the board while the middle shows nothing. `In review`
matters most and is easiest to lose — an item that closes with its review still
outstanding reads as `Done` and nowhere says otherwise.

## Session shape

**Each stage gets a clean context.** Plan in one session; a fresh session for
Implement; review in a third. Carrying exploration context into implementation is
how scope creeps, and a reviewer holding the implementer's assumptions is not an
independent reviewer.

Because context does not survive that boundary, **the agreed plan is written to
`plan.md` and posted as an issue comment.** That is the handoff artifact; the
Implement session reads it rather than inheriting it. A plan that exists only in a
session transcript does not exist.

Not every item needs a full loop. When the change fits in one sentence, **skip
Plan**: minimal fix plus a regression test, then Verify normally. Implement and
Verify are never skipped.

A bug fix skipping Plan still owes its **Autopsy** — which test should have caught
this, and why it didn't (`write-spec`, and `TESTING_TRAPS.md` → The autopsy). That
is the one part of the loop a one-line fix must not skip, because a defect that
escaped is the only free evidence about the suite anyone gets.

Commit only after reviewing the diff yourself. `git push` stays a human step.
Commit subjects cite the plan item they land — `042.3: pin restore against the
ground-plane fallback` — which is what makes partial progress on a branch that
takes three days legible in `git log` without opening a single diff.

## Plan

Run the `plan-feature` skill, which is the authority on plan content. This stage
adds:

- Read `spec.md` and every acceptance criterion in full
- List each change as a numbered item with a rationale
- Map each to the criterion it serves, and name the test that will prove it
- Name the mutation each test must catch
- Mark any criterion this environment cannot settle
- Make no code changes
- On agreement, write `plan.md` and post it as an issue comment

## Implement

Run the `implement-feature` skill. Read the agreed plan from `plan.md`. Execute it
item by item, test-first — the failing test comes before the implementation, not
after. Keep `notes.md` open as you go.

If reality contradicts the plan, stop and report. See Re-entry.

An item that ends **Blocked** or **Failed** updates its `Sized:` line to what is
left rather than what it was. That line is the only place a half-landed item says
so; the tracker shows it open and looking untouched.

## Verify

Run the narrowest suite that covers the criteria first, then broaden when
practical. Then hand the diff to a **fresh** reviewer — the `reviewer` agent, or
`review-change` in a clean context. Self-review does not satisfy this step.

The `reviewer` agent returns a mechanical verdict: `NEEDS WORK` if it raised any
`blocker`, otherwise `READY FOR HUMAN REVIEW`. A `NEEDS WORK` verdict re-enters at
Implement (see Re-entry) and the item comes back for another pass. Its `should` and
`question` findings never force that loop on their own — `question` is where a
Gated criterion is reported, since only the human can close one.

Resolve **every** criterion to one of three outcomes before reporting. Do not stop
at the first failure: the test run is already paid for, so extract every
conclusion it supports. Stopping happens before *fixing*, not before finishing the
assessment.

- **Verified** — a command result or reviewer finding proves it
- **Failed** — see Failure protocol
- **Gated** — only a real device, real users, real load, or human eyes can settle
  it. Never record one as verified and never as a failure. Report it outstanding
  and name the check that would close it.

Before closing, promote anything from `notes.md` that outlives this item: a
decision to an ADR, a vacuous-test shape to `TESTING_TRAPS.md`, a risk to the
register, a repeated correction to `CLAUDE.md` → Gotchas. If this item changed what
comes next, update `PRODUCT.md` → Next.

## Outcomes

Every item ends in exactly one of these, and every one gets a report:

| Outcome | Condition | Issue | Branch |
|---------|-----------|-------|--------|
| **Complete** | All criteria Verified | Closed | Merged to `main` |
| **Blocked** | No failures; at least one Gated | Open, stays assigned | Stays on its branch |
| **Failed** | At least one criterion Failed | Open, stays assigned | Stays on its branch |

A **Complete** item is merged as part of closing out. A closed issue whose work is
still on a branch looks done everywhere except where it counts.

Re-running the suite after a fast-forward is unnecessary when the merged tree is
identical to the verified commit's tree; quote the two hashes instead of spending
a run.

## Re-entry

Each stop has one resume point. Nothing restarts from picking up work.

| Stopped at | On approval, resume at |
|------------|------------------------|
| Plan output | Implement, with the agreed `plan.md` |
| Implement, plan contradicted | Plan, amending only the affected items; record it under `plan.md` → Amendments |
| Verify, criterion Failed | Implement, test-first on the fix — the regression test comes before the correction |
| Verify, Gated only | Nothing to resume; that check is a human step |

After an approved fix, re-run the suite covering the failed criterion plus
anything the fix could plausibly have disturbed — not the full set by reflex, and
not the single test in isolation. Re-resolve all criteria before reporting again.

## Failure protocol

1. Finish resolving the remaining criteria — do not abandon the run
2. Report which criterion failed and why
3. Propose a fix as a plan item
4. Wait for approval
5. On approval, re-enter at Implement

## Report

Post to the issue at the end of every item, including blocked and failed ones — an
item that ended badly is the one most worth a record.

```markdown
## <Complete | Blocked | Failed>

**Changed:** <brief description>

**Files:** `path` — what changed there

**Commands:**
| Command | Result |
|---------|--------|
| `<./scripts/test.sh>` | <pass/fail counts, or where results were written> |

**Criteria:**
| AC | Outcome | Evidence |
|----|---------|----------|
| AC-1 | Verified | <test name or reviewer finding> |

**Mutations checked:**
| Mutation | Caught by |
|----------|-----------|
| <what was broken> | <which tests failed> |

**Gated:** <criterion + the check that would close it, or "none">

**Not closed:** <deferrals, each with an owner, or "none">
```

The Commands, Criteria, and Mutations tables are governed by `CLAUDE.md`: no claim
that a build, test, or performance target passed without evidence from the run.
