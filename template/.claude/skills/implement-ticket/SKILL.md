---
name: implement-ticket
description: Implement a ticket's approved plan in small, individually verified increments.
---

# Implement a Ticket

This is the Implement stage of `docs/WORKFLOW.md`. Start from the agreed
`plan.md`, not from memory of the planning conversation.

1. Read the approved plan and the acceptance criteria it maps to.
2. Take the plan's items in numeric order, which is the order the plan chose to
   land them in, unless an item states a dependency that says otherwise. **Item
   `.1` is first because it is the most likely to be wrong** — do not reorder it
   behind something easier, and do not read a failure there as a crisis. Finding
   out early is what it is for. For each item:
   - Write the failing test first. Run it. Confirm it fails **for the stated
     reason**, not because it does not compile.
   - Implement the smallest change that makes it pass.
   - Run the targeted test. Do not defer verification to the end.
   - Confirm the change matches the plan item before moving on.
3. Stay inside the planned modules. No unrelated refactors, no opportunistic cleanup.
   **When you reach one of the plan's *Could not determine* items, answer it and
   write the answer down** — in `notes.md` if the code turned out as the plan hoped,
   and as a `plan.md` → Amendment if it did not, which is a re-entry condition like
   any other contradiction. The plan named those unknowns so that resolving one is a
   visible act; resolving it in your head is the guess the section exists to prevent.
4. Before declaring an item done, run its mutation check: break the thing the test
   covers and confirm the test goes red. Record which mutations you ran. If a
   mutation survives, the test is the defect — fix the test in this cycle.
5. Run **full verification** before declaring the item done — not the targeted tests
   you have been running, and not "when practical". A focused run cannot see what
   you broke three modules away; `docs/WORKFLOW.md` → Verify has the argument.
6. Update ADRs, the ticket's `notes.md`, fixtures, and contract documentation when a
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
amendment that would work. Then wait. Amendments are recorded under `plan.md` →
Amendments, so the next reader sees what was assumed as well as what was done.

**A one-way door the plan did not name is always a stop**, never a judgement call.
If landing an item means writing data in a new format, minting an id, publishing a
message, or dropping something an older build reads — and the plan's *One-way
doors* section did not say so — the plan was wrong about the thing that is hardest
to take back. Report it before you write the code, not after.

## What to write down as you go

Keep a running list of things the next person needs and would not find:

- Behaviors you changed that nobody asked you to change, and why
- Cases you found unreachable today but reachable after some named future change
- A test you *wanted* to write and could not, and what would make it possible
- Anything you deferred, with the ticket that owns it

This list becomes the "notes carried forward" section of the post-cycle report.
An observation that lives only in your context is lost at the end of the session.
