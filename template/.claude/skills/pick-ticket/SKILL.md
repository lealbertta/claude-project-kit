---
name: pick-ticket
description: Choose the next ticket to work on from the eligible set, confirm it, and claim it.
---

# Pick a Ticket

Everything upstream of the loop. `docs/WORKFLOW.md` begins with a ticket already
chosen; this is how one gets chosen, and how you find out that what you were
handed is not a ticket at all.

## 1. Get the eligible set

```sh
gh issue list --state open --label <task> --limit 100 \
  --json number,title,labels,assignees \
  --jq 'map(select(([.labels[].name] | index("<blocked>") | not) and (.assignees | length) == 0))'
```

That returns **a set, not a queue.** Nothing about the order it prints in means
anything — `PRODUCT.md` → Next is unordered on purpose, and a ticket is eligible
or it is not.

`--limit` truncates silently. Past a hundred open unassigned tickets you are
choosing from a page rather than from the set, and nothing says so.

**With no tracker**, the spec header block is the whole system and this is the
query:

```sh
# Priority carries the sort; Status carries what is still eligible. In this
# configuration the spec's Status: line IS the status, so a finished ticket is
# excluded here or not at all.
grep -rl 'Priority:\*\* High' --include=spec.md docs/work | while read -r s; do
  grep -qE '^- \*\*Status:\*\* (Done|Abandoned)' "$s" || echo "$s"
done
```

## 2. Read the priorities as claims, not as ranks

Each `spec.md` carries a **Priority** with the reason on the same line. Read the
reasons of the high ones before choosing. They are claims, and a claim written
three weeks ago is often no longer true — which is the entire point of writing the
reason next to the word.

Priority **narrows** the set. It does not order it, and it never overrides a
*Depends on*: a dependency is a fact about the work, a priority is a judgement
about its value, and only one of those can make a ticket impossible to start.

## 3. Check that it is actually a ticket

**An epic is not a ticket and the loop will not run on one.** A feature that will
take several branches has no single diff to review and no single merge to make, so
a plan drawn over it is a plan nobody can check and a branch nobody can land.

If what you picked is one, stop here. It gets broken down first — `write-spec`
drafts the PRD and the child tickets, `sync-tickets` files them — and then you come
back and pick **one** of the children.

The same guard fires again at Plan, and that is normal:
`plan-ticket-implementation` stops rather than planning around a ticket that turns
out to be two, because splitting is cheap before a plan exists and expensive after.

## 4. Confirm, then claim it

**Confirm the choice with the human before starting.** Picking work is not a
neutral act — the set is eligible, not equivalent, and the reason to prefer one
over another is usually something the tracker does not hold.

Then claim it, in one step:

- Assign the issue to yourself
- Move the board to `In progress` — `docs/TRACKER.md` → Setting the status has the
  lookup, and an empty item lookup is a stop rather than a skip

A ticket that is being worked on and shows as unassigned in `Todo` is how two
people start the same thing.

## 5. Hand off to the loop

From here `docs/WORKFLOW.md` takes over: **Plan → Implement → Verify**, once, on
this one ticket.
