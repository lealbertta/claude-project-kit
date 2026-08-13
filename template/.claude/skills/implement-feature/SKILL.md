---
name: implement-feature
description: Implement an approved plan in small, individually verified increments.
---

# Implement a Feature

This is the Implement stage of `docs/WORKFLOW.md`. Start from the agreed
`plan.md`, not from memory of the planning conversation.

1. Read the approved plan and the acceptance criteria it maps to.
2. Take the plan's items in order. For each one:
   - Write the failing test first. Run it. Confirm it fails **for the stated
     reason**, not because it does not compile.
   - Implement the smallest change that makes it pass.
   - Run the targeted test. Do not defer verification to the end.
   - Confirm the change matches the plan item before moving on.
3. Stay inside the planned modules. No unrelated refactors, no opportunistic cleanup.
4. Before declaring an item done, run its mutation check: break the thing the test
   covers and confirm the test goes red. Record which mutations you ran. If a
   mutation survives, the test is the defect — fix the test in this cycle.
5. Run **full verification** before declaring the item done — not the targeted tests
   you have been running, and not "when practical". A focused run cannot see what
   you broke three modules away; `docs/WORKFLOW.md` → Verify has the argument.
6. Update ADRs, the work item's `notes.md`, fixtures, and contract documentation when a
   contract moved. A change whose contracts moved is not complete while those are stale.

## Commit as you go

Commit at each plan item, with the item id in the subject. **Review reads the
working tree against the merge-base, including uncommitted and untracked files**
(`docs/WORKFLOW.md` → Session shape), so checkpoints cost the review nothing and
they are what makes a three-day branch legible in `git log`.

**Never reset, stash, revert, or amend to make the diff look right for review.** The
reviewer sees the work in whatever state it is in; rewriting history to present
something tidy destroys work to solve a problem that does not exist.

## When reality contradicts the plan

Stop and report. Do not improvise around it. A file that is not shaped as assumed,
an item that turns out unnecessary, or a change that must touch something unplanned
are all re-entry conditions, not judgement calls to absorb quietly.

Report as: what the plan assumed, what is actually there, and the smallest
amendment that would work. Then wait.

## What to write down as you go

Keep a running list of things the next person needs and would not find:

- Behaviors you changed that nobody asked you to change, and why
- Cases you found unreachable today but reachable after some named future change
- A test you *wanted* to write and could not, and what would make it possible
- Anything you deferred, with the ticket that owns it

This list becomes the "notes carried forward" section of the post-cycle report.
An observation that lives only in your context is lost at the end of the session.
