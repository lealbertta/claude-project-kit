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

Four things have to come back from this step, and each has a home in the plan:

- **The files.** Open the ones you intend to change, their tests, and their
  configuration.
- **The call sites.** Grep for every symbol, field, or route you are about to
  change and list what reads it. Naming the file you will edit while missing the
  four places that call into it is the single most common reason a plan gets
  amended mid-implementation.
- **The precedent.** Something in this codebase probably already solves a problem
  shaped like this one. Follow it, or say why you are not — a plan that introduces
  a second way of doing an existing thing owes an argument, and that argument is
  usually an ADR (`record-decision`).
- **What you could not determine.** "I could not tell whether the cache is
  invalidated on this path" is a real planning output and belongs in the plan. An
  unknown carried silently into Implement gets resolved by guessing.

### Dispatch the reading, keep the deciding

This step reads far more than it keeps, which is exactly the shape a subagent is
for: send the reading out, take back the findings. **You stay the author of the
plan.** You hold the spec, the ADRs, and the conversation with the human, and an
`investigator` can hold none of those — it cannot ask a question, so anything it
would have asked comes back unanswered or, worse, guessed.

Name the questions **before** you dispatch, one per investigator, and dispatch them
**together**. The count is bounded by the unknowns you can name, not by how large
the ticket feels; two to four is the usual band and one is a fine answer. Every
investigator costs a fresh context that re-pays the discovery you have already
made, so a brief you could answer yourself in two greps is cheaper answered
yourself.

**A change ticket and a bug ticket have different unknowns, so they get different
briefs.**

| Ticket | What is unknown | Dispatch |
|--------|-----------------|----------|
| Change, feature, refactor | *where the code is* | **survey** briefs, split by area — one per subsystem, file cluster, or contract the change touches |
| Bug | *why it happens* | **hypothesis** briefs, one candidate cause each, every one told to kill its own |

The bug case is the one that pays. Left to itself an agent finds one plausible
explanation and stops looking, and every search after the first is biased toward
it — so the diagnosis you get is the one that occurred to you earliest, not the one
that is true. Several investigators, each dispatched against a different cause and
each trying to **falsify** it, is the countermeasure: what survives being attacked
is evidence, and what merely got confirmed is not.

On a bug, the surviving hypothesis **is** plan item `.1` — the riskiest assumption
on a bug fix is always the diagnosis, so the thinnest slice that proves it comes
first and everything else waits behind it.

It also settles the spec's **Autopsy**, which asks which test should have caught
this and why it didn't. That question has no honest answer until the cause is
known, so an Autopsy filled in at spec time, from the symptom, is the one to go back
and correct here — `write-spec` says it is written before the fix is planned, not
before the cause is found. Where the corrected answer differs from what was
originally written, both go in, per `.claude/rules/documentation.md`.

**Record the refuted ones too**, one line each, in `notes.md` → *Causes ruled out*.
They are the half of the search space nobody has to pay for again, and without a
note they get re-proposed the first afternoon the fix looks shaky.

### Do not believe a report you did not run

An investigator's finding is a **claim, and it becomes evidence when you re-run
it.** Each one comes back labelled `traced` or `ran`; re-run the `ran` ones and
confirm the command prints what the report said. This is the same rule Verify uses
when two reviewers disagree — settle it by running it — applied one stage earlier,
and for the same reason: without it you have built a machine that produces
confident diagnoses at speed.

Two reports contradicting each other is a good outcome and not a problem to
average away. It means the code is ambiguous enough that two careful readers
disagreed, which usually outlives this ticket — run the thing, and note the
ambiguity where it will be read again.

**Where subagents are unavailable, do the same investigation serially in this
session.** The four findings above are the requirement; the parallelism is only how
you get them cheaply and without anchoring. What you lose serially is independence,
so on a bug write each candidate cause down *before* testing any of them — that is
the cheap half of what the parallel dispatch buys.

## 3. Design the change

**Number the items in the order you mean to land them, and order by risk rather
than convenience.** Item `.1` is the thinnest slice that runs end to end through
every layer this change touches and proves the assumption most likely to be wrong —
not the easiest item, and not the bottom layer because it is tidy to start there.
Say what it would mean if that slice fails. A wrong assumption found on the first
afternoon is a plan amendment; the same assumption found last is a rewrite.

**Cut the slice through the layers, not along them**, and the usual objection —
*the risky item depends on a duller one* — mostly disappears, because a slice that
runs end to end carries its own dependencies. Serializing a field in one item and
reading it back in the next puts the risky half second and lands a field nothing
reads; the two are one item. Where a dependency genuinely cannot be folded in, say
which item is the risky one and why it could not come first, so the next reader
knows the order was chosen rather than inherited.

**Every item leaves the tree green.** Each is separately committable and the suite
passes after it. **An item that only works once the next one lands is not two
items** — that rule is what forces the slice above, and it is the one most often
broken by a plan that looks tidy.

**From agreement onward the numbers are frozen ids.** `<item>.<n>` — `42.1`, `42.2`
— is cited from commit subjects, so a change added later takes the next free number
even where it belongs logically in the middle, and sits where the plan says rather
than last. Never renumber, never reuse. Nothing in the numbering implies a
dependency: where item B genuinely cannot land before item A, say so **on B**.

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
  it. An untagged gated criterion gets reported as passing. Carry the spec's marking
  through: a `[Gated, blocks merge]` criterion means this ticket cannot complete
  until that check runs, which is worth knowing now rather than on the day the
  branch is finished.

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
| 42.1 | <what and why> | `path` | AC-1 | `TestName` | <what to break so it fails> |

**Call sites:** <what reads or calls the thing that changes, and which item covers
each — or "none beyond the files above", having actually looked>

**Precedent:** <the existing thing this follows, with its path, and how closely it
matches — or the argument for why this change does it differently, which is usually
an ADR>

**Could not determine:** <what investigation did not settle, and what would settle
it — or "nothing", which is a strong claim and rarely true>

**Implications:** schema / undo / performance / compatibility / lifecycle —
one line each, or "none" with the reason.

**One-way doors:** <items a revert does not undo, each with what would make it
reversible — or "none, everything here is undone by reverting">

**Pre-mortem:** <three past-tense failures, each with where it went: a plan item,
a test, a risk row, or Unverifiable>

**Unverifiable here:** <criteria needing a real device, real users, or eyes, or "none">

**If it goes wrong:** <how we would notice it in the wild, and how to undo it>
```

A one-session ticket may drop *Call sites*, *Precedent*, and *One-way doors* by
deleting the heading when they are genuinely empty — but not the table, the
implications, or the pre-mortem, which are the three that catch what a small ticket
gets wrong. **Deleting *Could not determine* is a claim**, not a saving: it says
investigation left no unknown behind, which happens and is worth the two seconds of
asking whether it is true.

On approval, write it to `plan.md` and post it verbatim to the issue before ending
the session. The implementation happens in a fresh session and will not inherit
this context.
