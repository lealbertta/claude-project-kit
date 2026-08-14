---
name: plan-ticket-implementation
description: Plan how to implement one ticket before any code is written. No code changes.
---

# Plan a Ticket's Implementation

This is the Plan stage of `docs/WORKFLOW.md`. It runs on **one ticket**, already
chosen, and it produces a plan and nothing else.

The plan is the handoff artifact, not a note to yourself. Implement runs in a fresh
session that will not inherit this context, so **anything you worked out and did not
write down did not happen.**

## 1. Orient

1. Read `CLAUDE.md`, and only the project documents and path-scoped rules this
   change actually touches.
2. Read the ticket's `spec.md` in full — every acceptance criterion, and also
   **Non-goals**, **Risks**, **Depends on**, and **Open questions**. An open
   question that blocks a criterion gets answered before that criterion is planned.
3. Read the ADRs the change sits on top of. Only **Accepted** ones constrain, and
   one **Amended by** a later ADR constrains as amended.
4. Restate the requested behavior, the explicit non-goals, the affected systems,
   and your assumptions. If the restatement is not obviously the same thing the
   spec said, the spec is ambiguous — settle that with the human now, while it is
   still the cheapest thing in the cycle to change.

**Check the ticket is still one ticket.** A ticket is one branch, one plan, one
review, one merge, and roughly one to three sessions. If planning shows it is
bigger than that — two independently mergeable changes wearing one id, or a
criterion that cannot be met until some other work lands — **stop and say so
instead of planning around it.** Splitting is cheap before a plan exists and
expensive afterwards, and a plan that quietly grows to cover an epic is how a
branch becomes unreviewable. `docs/WORKFLOW.md` → Entry has the guard.

## 2. Investigate before you design

**Do not design against what you assume the code looks like.** A plan built on the
probable shape of a file is a plan whose first amendment is already written.

- Open the files you intend to change, their tests, and their configuration.
- **Find the call sites.** Grep for every symbol, field, or route you are about to
  change and list what reads it. Naming the file you will edit while missing the
  four places that call into it is the single most common reason a plan gets
  amended mid-implementation.
- **Find the precedent.** Something in this codebase probably already solves a
  problem shaped like this one. Follow it, or say why you are not — a plan that
  introduces a second way of doing an existing thing owes an argument, and that
  argument is usually an ADR (`record-decision`).
- **Name what you could not determine.** "I could not tell whether the cache is
  invalidated on this path" is a real planning output and belongs in the plan. An
  unknown carried silently into Implement gets resolved by guessing.

## 3. Design the change

**Order the plan by risk, not by convenience.** Item `.1` is the thinnest slice
that runs end to end through every layer this change touches and proves the
assumption most likely to be wrong — not the easiest item, and not the bottom
layer because it is tidy to start there. Say what it would mean if that slice
fails. A wrong assumption found on the first afternoon is a plan amendment; the
same assumption found last is a rewrite.

**Every item leaves the tree green.** Each is separately committable and the suite
passes after it. An item that only works once the next one lands is not two items.

**The numbers are ids, not an order.** `<item>.<n>` — `042.1`, `042.2` — is cited
from commit subjects, so the numbers are permanent: a change added later takes the
next free one even where it belongs logically in the middle. Never renumber, never
reuse. Where item B genuinely cannot land before item A, say so **on B**; nothing
else in the numbering implies a dependency.

**Break contracts in parallel, never in one step.** Where the change alters
something already written down elsewhere — a persisted shape, a serialized field, a
published interface, anything an older build or another process reads — plan
*expand → migrate → contract* as separate items: add the new form beside the old,
move the readers over, then remove the old one. Each step reverts on its own, which
is the entire point. The single item that swaps one for the other is a one-way door
wearing an ordinary item's clothes.

Then, for the plan as a whole:

- **Implications** — data model, persisted schema, undo/redo, performance,
  compatibility, lifecycle. One line each, or "none" **with the reason**. "None"
  without a reason is the sentence that hides the migration.
- **Tests.** Name the test that proves each criterion, and for each test name the
  mutation it must catch. If you cannot name one, the test is decoration and the
  plan is not finished.
- **Gated criteria.** Mark every criterion this environment cannot settle — real
  hardware, real users, real load, human eyes — and name the check that would close
  it. An untagged gated criterion gets reported as passing.

## 4. Stress the plan before you present it

Two passes over the finished plan. Both are cheap, and both are usually where the
plan actually changes.

**The pre-mortem — write it in the past tense.** *It is a month from now. This
shipped and it went wrong.* Write the three most likely reasons as things that
already happened, not things that might. The tense is the technique rather than a
flourish: people are markedly better at explaining an outcome than at predicting
one, and "what could go wrong" and "what did go wrong" do not return the same list.
Each answer then goes somewhere — a new plan item, a test with a named mutation, a
row in `docs/RISK_REGISTER.md`, or a line under *Unverifiable here*. A pre-mortem
answer with nowhere to go was not an answer.

**The one-way doors.** Walk the items and ask which ones a `git revert` does not
undo: data already written in the new format, an id minted, a message published, a
column dropped, a file the user has already opened and saved. For each, either
convert it into a two-way door — the parallel-change split above, a flag, keeping
the old reader alive one release longer — or name it irreversible and get the
human's explicit agreement **before** it is implemented, not after. **A revert is
not a rollback once something else has read the output.** "None — everything here
is undone by reverting" is the common answer and is worth writing down.

Then the questions this kit keeps asking, because they keep paying: which item here
is load-bearing and which is speculative; what does this change make *reachable*
that was previously unreachable; and which existing rule was written when the world
was simpler than it is now.

## 5. Write it out

Do not edit implementation files. Output the plan and wait for approval.

```markdown
## Plan — <id> <title>

**Non-goals:** <what this deliberately does not touch>

| ID | Change | File(s) | Serves | Proven by | Mutation it must catch |
|----|--------|---------|--------|-----------|------------------------|
| 042.1 | <what and why> | `path` | AC-1 | `TestName` | <what to break so it fails> |

**Call sites:** <what reads or calls the thing that changes, and which item covers
each — or "none beyond the files above", having actually looked>

**Implications:** schema / undo / performance / compatibility / lifecycle —
one line each, or "none" with the reason.

**One-way doors:** <items a revert does not undo, each with what would make it
reversible — or "none, everything here is undone by reverting">

**Pre-mortem:** <three past-tense failures, each with where it went: a plan item,
a test, a risk row, or Unverifiable>

**Unverifiable here:** <criteria needing a real device, real users, or eyes, or "none">

**If it goes wrong:** <how we would notice it in the wild, and how to undo it>
```

A one-session ticket may drop *Call sites* and *One-way doors* by deleting the
heading when they are genuinely empty — but not the table, the implications, or the
pre-mortem, which are the three that catch what a small ticket gets wrong.

On approval, write it to `plan.md` and post it verbatim to the issue before ending
the session. The implementation happens in a fresh session and will not inherit
this context.
