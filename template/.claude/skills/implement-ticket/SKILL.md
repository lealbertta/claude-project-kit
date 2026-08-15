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
   out early is what it is for. Each item runs the cycle below, and no item skips it.
3. Stay inside the planned modules. No unrelated refactors, no opportunistic cleanup.
   **When you reach one of the plan's *Could not determine* items, answer it and
   write the answer down** — in `notes.md` if the code turned out as the plan hoped,
   and as a `plan.md` → Amendment if it did not, which is a re-entry condition like
   any other contradiction. The plan named those unknowns so that resolving one is a
   visible act; resolving it in your head is the guess the section exists to prevent.
4. Before declaring an item done, run its mutation check: break the thing the test
   covers and confirm the test goes red. Record which mutations you ran. If a
   mutation survives, the test is the defect — fix the test in this cycle.
5. Run **full verification before declaring a plan item done** — every item, not
   once at the end, and not "when practical". Not the targeted tests you have been
   running either: a focused run cannot see what you broke three modules away, and
   the plan's own rule is that every item leaves the tree green. Verify runs it once
   more on the finished branch, as the gate; this one is how you find out at item
   `.3` rather than after `.7`. `docs/WORKFLOW.md` → Verify has the argument.
6. Update ADRs, the ticket's `notes.md`, fixtures, and contract documentation when a
   contract moved. A change whose contracts moved is not complete while those are stale.

## Every item is test-driven: red, green, refactor

**This is where the kit's test-first non-negotiable is carried out**, and it is the
whole shape of an item. Steps 4 and 5 above are the last two beats of the same
cycle — mutation, then full verification — and they run per item, not per branch.

**Red.** Write the test the plan named in its *Proven by* column, and write it
before the code it tests exists. Run it. Confirm it fails **for the reason the plan
stated** — the assertion you meant, not a compile error, a missing import, or a
typo'd fixture name. A test that fails because nothing builds yet has told you
nothing about the behavior, and it is the shape that quietly becomes a test asserting
whatever the code does. Get it failing on the assertion, then copy that failure line
into `notes.md` → **Red steps**.

That record is the point of the beat and it costs nothing, because you have already
run it. It is also the one thing mutation testing cannot give you: by the time a
mutation runs, everything compiles and the test's *reason* for failing is no longer
in question. The red line is the evidence that this test could ever fail without its
implementation — and, unrecorded, it is a claim rather than evidence.

**Green.** The smallest change that makes that test pass. Not the general version,
not the version that also handles the case item `.4` is for. If the smallest change
is embarrassing, that is the cycle working — the next red is what generalises it.

**Refactor.** With the test green, tidy what you just wrote: names, duplication you
introduced, a shape that only became obvious once the code existed. Re-run the test.
This beat is the one that gets dropped, and dropping it is how a branch of
individually-smallest changes lands as something nobody would have designed.

**It is bounded by the same scope rule as everything else.** Code this item wrote is
yours to clean; code it merely sits next to is not — `CLAUDE.md` → non-negotiable 4,
and there is no exception here. *"This pattern is now duplicated in three other
files"* is a real finding and a different ticket: write it in `notes.md`, leave it
alone, and let the `architect` raise it at Verify.

Then confirm the change matches the plan item, and go to step 4.

### When the test genuinely cannot come first

Two cases, and neither is a licence to write the test afterwards and call it
test-driven:

- **The behavior is gated** — feel, visuals, real hardware, real load, real users.
  The plan marked it (`plan-ticket-implementation` → Gated criteria) and the honest
  move is to say so, not to write a test that asserts something adjacent and
  cheaper. It goes to `notes.md` → Gated and is never recorded as verified.
- **You do not yet know the shape well enough to name the assertion.** Spike it —
  then **throw the spike away and start at red.** A spike kept as the green step is
  the implementation writing its own test, and every assertion in it will be true by
  construction. Anything the spike taught you that the plan did not know is a *Could
  not determine* answered, per step 3.

A third case that is not one of these: the test is awkward to write because the code
is hard to test. That is the design telling you something, and the answer is in the
design, not in skipping the beat.

## Commit as you go

Commit at each plan item, with the item id in the subject. **Review reads the
working tree against the merge-base, including uncommitted and untracked files**
(`docs/WORKFLOW.md` → Session shape), so checkpoints cost the review nothing and
they are what makes a three-day branch legible in `git log`.

Rearranging the history — reset, stash, revert, amend — is yours to do as you like.
The reviewer reads the working tree, so it changes nothing about what gets reviewed.
The one thing to watch is doing it **while a review is running**: the surface is the
tree as it stands, so work stashed mid-review is work the reviewer scores as absent.

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

Write the things the next person needs and would not find into `notes.md` as you
go — it already has a section for each:

| What you found | Where it goes |
|----------------|---------------|
| A behavior you changed that nobody asked you to change, and why | `notes.md` → **Changed without being asked** |
| A case unreachable today but reachable after some named future change | `notes.md` → **Unreachable today**, recorded as *inert* and not as covered |
| A test you *wanted* to write and could not | `notes.md` → **Not closed**, with what would make it possible as the unblocker |
| Anything you deferred | `notes.md` → **Not closed**, naming the ticket or person that owns it |

**Write them when you find them, not at the end.** An observation that lives only in
your context is lost when the session is, and Verify runs in a different one.

`notes.md` is the working record and it dies with the ticket. Of this list, only the
deferrals surface on the issue — `docs/WORKFLOW.md` → Report → **Not closed** — and
anything that outlives the ticket is promoted before it closes, which is Verify's
last step and not yours.
