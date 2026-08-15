---
name: review-change
description: Independent correctness, test-strength, and scalability review of a change.
---

# Review a Change

Use a fresh subagent or a clean session. Self-review in the session that wrote the
code does not satisfy this step — you will re-derive the same assumptions.

Review the diff against the accepted plan, the acceptance criteria, and the
relevant ADRs.

## Where this sits

`docs/WORKFLOW.md` → Verify sends a branch to **two** reviewers: the `reviewer`
agent, whose brief is this file, and the `architect`, whose brief is
`.claude/agents/architect.md`. This skill is the fallback where subagents are not
available — run it twice, in two clean sessions, once against each brief. Running
both briefs in one session produces one review with two headings, and the second
half will be written by someone who has already made up their mind.

Outside the loop — an ad-hoc diff, a branch nobody filed a ticket for — this skill
stands on its own and the reviewer brief below is the whole job.

Either way, the review **reports and does not fix**. A reviewer that fixes things
stops reporting them, and the finding disappears into a diff nobody reads.

**Inside the loop, post your findings to the PR as your own review** — line-anchored
where a finding names a line — and never read what the other review has already put
there. Same rule at both ends: two reviews that have read each other are one review
and a confirmation of it. Outside the loop there may be no PR, and the report goes
wherever the change is being discussed.

## Check the code

- Incorrect behavior, or an acceptance criterion nothing actually satisfies
- Data incompatibility or a missing migration
- Broken transaction, undo, or batch boundaries
- Nondeterminism: iteration order, timing, hash ordering, floating-point drift
- Hidden global state, ambient lookups, circular module dependencies
- Allocations, synchronous IO, or unbounded loads in hot paths
- Resource leaks and duplicated shared dependencies
- Lifecycle, interruption, and resume regressions
- Unauthorized scope expansion or dependency/configuration changes

## Check the tests — this is the half that gets skipped

For every test in the diff, ask what implementation mutation it would survive.
Run the mutations you can. Consult `docs/TESTING_TRAPS.md` first; the specific
shapes to look for are:

- The rule is tested directly but the **production wiring** is tested by nothing
- The assertion is expressed in terms of the constant it is supposed to pin
- The fixture makes the branch under test unreachable, so it never runs
- Both sides of a comparison move together, so the assertion cannot fail
- A sequence is compared after collapsing it, hiding a whole class of wrong behavior
- A counting assertion has only one half ("X is zero") and passes for a component
  that does nothing at all
- A branch inert on current data is claimed as covered rather than recorded as inert

A passing suite that survives a deleted feature is a finding, and usually a larger
one than any bug in the diff.

Mutate logic, not narration — a mutation surviving in logging, metrics, or the
wording of an error message is not a finding, and reporting it trains people to
discount the findings that are. There is no mutation-score target; report the named
survivor.

If the change fixes a bug, check `spec.md` → Autopsy: the test that should have
caught it, why it didn't, and where that was written down. A regression test whose
lesson went nowhere leaves a suite that grows one test per bug and learns nothing.

## Report

Resolve **every** acceptance criterion to **Verified**, **Failed**, or **Gated**
before reporting — never two outcomes, never silence. A `Gated` criterion is never
a pass and never a failure: report it outstanding, name the check that would close
it, and say what it costs if that check later fails — a human is about to be asked
whether it may ship outstanding, and cannot answer that from a criterion alone.

Every finding carries a file and line, evidence, a concrete correction, and exactly
one severity — the same three the `reviewer` agent uses, because `docs/WORKFLOW.md`
→ Verify treats the two as interchangeable:

- **blocker** — must be fixed before a human spends time on this
- **should** — worth fixing, does not gate human review
- **question** — a domain or intent call only the human can settle; a `Gated`
  criterion is reported here, never as a blocker

**Your verdict is mechanical, so the loop terminates:** `NEEDS WORK` if and only if
there is at least one **blocker**, otherwise `READY FOR HUMAN REVIEW`. It covers
your own findings; the consolidated verdict also accounts for whatever the other
review marked blocking, so a branch you pass can still come back. Decide
severity per finding as you write it; weighing them into an overall impression at
the end is how the same branch reads as *nearly there* on one pass and *not quite*
on the next with nothing having changed.

Do not report personal style preferences as blockers. If you found nothing
material, say so plainly rather than manufacturing findings.
