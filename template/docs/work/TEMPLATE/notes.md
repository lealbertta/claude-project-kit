# Notes — <id> <title>

What this ticket found. Written as you go, not reconstructed at the end — an
observation that lives only in a session's context is lost when the session ends.

This file dies with the ticket. Anything that will still matter in six months
gets **promoted** before the ticket closes:

- A decision → `docs/DECISIONS/`
- A way a test went vacuous → `docs/TESTING_TRAPS.md`
- A risk → `docs/RISK_REGISTER.md`
- A rule Claude keeps getting wrong → the Gotchas list in `CLAUDE.md`

## Causes ruled out

Bug fixes only; written at Plan, when the `investigator` hypotheses come back
(`plan-ticket-implementation` → Investigate). Delete the heading otherwise.

| Candidate cause | Refuted by |
|-----------------|------------|
| <the claim that was tested> | <the command or line that killed it> |

A cause nobody wrote down as dead gets re-proposed the first afternoon the fix
looks shaky, and re-investigated by whoever is around. This table is cheap because
the work is already done — only the sentence is new.

## Mutations checked

| Mutation | Caught by |
|----------|-----------|
| <what was broken> | <which tests failed, and how many> |

A mutation that survived is a finding. Record it even after you fix the test.

## Review findings and their disposition

Verify sends the branch to two reviewers — `reviewer` and `architect` — each of
which posts its own review to the PR and may mark findings blocking at its own
discretion (`docs/WORKFLOW.md` → Verify). **Every finding gets exactly one
disposition, including the dismissed ones.** Not blocking does not mean optional: a
finding nobody dispositioned is a finding that was ignored with extra steps.

This table is where the dispositions are **worked out**, versioned with the diff
they describe. They go back to the PR as a reply on each thread plus one summary
comment carrying this table and the verdict. Same relationship as `plan.md` and its
issue comment: the file is the artifact, the post is the publication, and the PR is
the copy that outlives this folder.

**PR:** <link>



| Finding | From | Disposition |
|---------|------|-------------|
| <one line> | reviewer / architect / both | <fixed in <42>.4 / new ticket in `PRODUCT.md` → Next / ADR proposal / risk row R-<n> / watch, until <…> / dismissed, because <…>> |

Where the two reviewers disagreed about a **fact** — one recorded the suite green,
the other found the new test file matched by no project config — record the
disagreement and how it was settled by running it. Two careful readers getting
different answers is itself a finding about the project, and it outlives this ticket.

## Changed without being asked

Behaviors this touched that nobody requested, and why it was necessary.

- <…>

## Not closed

Deferrals and hazards found here. **Every entry names who owns it** — an unowned
note is how debt disappears.

| Note | Owner |
|------|-------|
| <one sentence> | <#issue / nobody yet> |

## Unreachable today

Cases that cannot happen on current data or config, but become reachable after
some named future change. Record as *inert*, not as covered.

- <case> — reachable once <…>

## Gated

Report these to the issue **before** the acceptance is asked for, not after
(`docs/WORKFLOW.md` → Verify). The Decision column is filled in from the answer.

| Criterion | Why not here | What would close it | Batched with | Decision |
|-----------|--------------|---------------------|--------------|----------|
| AC-<n> | <what this environment lacks> | <the specific check> | <other items waiting on the same session> | <accepted by <who>, <date> → #<issue> / declined → Blocked / `blocks merge` → Blocked> |

**If it fails:** <what ships wrong, who sees it, and how it is undone. This is the
half that makes acceptance a decision rather than a nod, so it goes in the report
too — not only here.>
