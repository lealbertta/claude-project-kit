```
                 /\_____/\
                /  o   o  \
               ( ==  ^  == )
                )         (
               (           )
              ( (  )   (  ) )
             (__(__)___(__)__)
```

# Claude Project Kit

A starting set of agent files for a new project: a short always-loaded instruction
file, path-scoped rules, on-demand skills, review subagents, a permission posture,
and a documentation layer built around independent tickets.

Extracted from a real multi-phase project, then stripped of everything specific to
its stack. Nothing here assumes a language, framework, or platform.

## What it is for

Three problems, one system:

1. **Context budget.** Everything the agent needs, none of it loaded until it is
   relevant. The instruction file stays short; subsystem rules load on path match;
   procedures load as skills when invoked.
2. **Evidence.** Work is planned before it is written, verified by two reviewers who
   did not write it and did not read each other, and reported with the commands that
   prove it — and a test does not count until it has been shown to fail on a mutation.
3. **No false sequencing.** Tickets are independent by default, so nothing has
   to be reordered or renumbered when priorities change.

## Install

Needs Claude Code — the rules, skills, and subagents are its features. Everything
under `docs/` is plain Markdown and reads fine anywhere. `gh` is needed only for the
tracker and pull request steps; the loop runs without it, and the no-tracker and
no-PR fallbacks are stated where they apply rather than left to be improvised.

```bash
cp -R template/. /path/to/new-project/
```

Then work through `ADAPTING.md`. Nothing here works until the `<ANGLE_BRACKET>`
placeholders are filled in — that is deliberate; a template that runs unedited is a
template nobody reads. The fill-in pass is a checklist, in order, and takes under an
hour.

Add to the new project's `.gitignore`:

```
CLAUDE.local.md
.claude/settings.local.json
```

Then read, in this order: `ADAPTING.md` for what to fill in,
`template/docs/WORKFLOW.md` for the loop those files serve, and
`template/docs/work/EXAMPLE-42-restore-reads-last-support/` for what a finished
ticket's three documents actually look like. Everything else is reference, read when
its turn comes.

## Using it

Each stage is a skill you invoke by name. Nothing fires on its own, and nothing here
is a background process — the sequence is yours to drive.

| To | Invoke | Which produces |
|----|--------|----------------|
| say what a change is for | `write-spec` | an issue, then `docs/work/<id>-<slug>/spec.md` — or a PRD, when the work spans several tickets |
| turn a PRD's requirements into issues | `sync-tickets` | one issue per requirement, idempotently, plus a drift report |
| choose the next thing | `pick-ticket` | one ticket, claimed — or a refusal, where what you handed it is an epic |
| plan it | `plan-ticket-implementation` | `plan.md`, risk-first and pre-mortemed, posted to the issue |
| build it | `implement-ticket` | a commit per plan item, each verified before the next |
| review it | the `reviewer` and `architect` agents together — `review-change` is the reviewer's brief where subagents are unavailable | a draft PR carrying two independent reviews, consolidated into one verdict |
| record why | `record-decision` | an ADR, and the rule it implies written where rules load |

Plan, Implement, and Verify each run in their own session — see *Each stage gets a
clean context* below for why, and `docs/WORKFLOW.md` for what each stage owes the
next.

## File map

```
template/
  CLAUDE.md                      Short, always loaded: role, stack, commands, structure, non-negotiables, gotchas.
  AGENTS.md                      Symlink to CLAUDE.md, so non-Claude agents read the same file.
  CLAUDE.local.md.example        Machine-local notes. Copy, gitignore.

  .claude/
    settings.json                Allow verification, ask on config/history, deny secrets & irreversible git.
    settings.local.json.example  Machine-local env. Copy, gitignore.
    rules/                       Auto-load on path match via `paths:` frontmatter.
      documentation.md             Where each kind of fact belongs; id forms, one-way references, and the checks.
      testing.md                   Test-first, mutation-checked, wiring pinned, gated criteria named.
      data-and-migrations.md       Schema versioning, atomic writes, forward-only migrations, preserve-unknown.
      code.md                      Language-agnostic skeleton — replace the specifics, keep the structure.
    skills/                      Load on demand, by name.
      pick-ticket/                 Everything upstream of the loop: the eligible set, the epic guard, claiming it.
      write-spec/                  Draft a ticket spec, or a PRD when the work spans several.
      plan-ticket-implementation/  Plan stage. Dispatches the reading, keeps the deciding. Risk-first, pre-mortemed.
      implement-ticket/            Implement stage. Test-first, increment-verified, stops when reality diverges.
      review-change/               Independent review — of the tests as much as the code.
      record-decision/             ADR or ticket, how to amend rather than rewrite, and shipping the rule with it.
      sync-tickets/                Spec → tracker issues, idempotent, reports drift instead of resolving it.
    agents/                      Read-only roles a session dispatches. Each one reports; none of them decides.
      investigator.md              Plan, several at once. Locates the code, or tries to kill one hypothesis about a bug.
      architect.md                 Verify. Whole-tree: convention drift, ADR conformance, duplication. Blocks at its discretion.
      reviewer.md                  Verify. Ticket-scoped: runs the checks, attacks the tests, verdicts the branch.

  docs/
    PRODUCT.md                   Vision and pillars (frozen) + Now / Next / Someday (weekly).
    WORKFLOW.md                  Plan → Implement → Verify, on one chosen ticket. Outcomes, re-entry, the report.
    TRACKER.md                   Tracker and board config, the board moves, and one owner per fact. Set up once.
    ARCHITECTURE.md              Modules, dependency direction, data layers, seams.
    TEST_STRATEGY.md             Which layer covers what; fixtures; what only reality can settle.
    TESTING_TRAPS.md             Eleven ways a passing test proves nothing, and the autopsy that finds the next one.
    RISK_REGISTER.md             Material risks, with mitigations and owners.
    DECISIONS/                   ADR template.
    work/TEMPLATE/               Copy per ticket.
      spec.md                      Source, priority, what and why, acceptance criteria, alternatives considered.  (before Plan)
      plan.md                      The agreed plan: mutations, call sites, precedent, unknowns, one-way doors, pre-mortem.  (end of Plan)
      notes.md                     Causes ruled out, review dispositions, deferrals, mutations run.  (Plan onward)
      PRD.md                       Only when the work spans several tickets — and then it is not a ticket.
                                   `sync-tickets` generates a fourth file, `tickets.md`, where a tracker is in use.
    work/EXAMPLE-42-…/          A filled-in ticket. Read it before writing your first. Delete it after.
```

## The ideas worth keeping if you keep nothing else

The rest of this file is the reasoning, one idea per paragraph, each opening with the
claim in bold. Skim the bold; read the paragraph where you disagree.

**The agent has a role, and the goal has a tension in it.** `CLAUDE.md` opens by
naming who the agent is, who it works with, what that person knows that it does not,
and the goal they share — stated so it can be traded against. Then it names the
tension in that goal, because there always is one: ship the smallest thing that
helps, except where a wrong answer costs someone something real, and there
*plausible* is not shippable. Without that paragraph the non-negotiables read as
arbitrary ceremony instead of as the price of a specific risk.

**The loop runs on one ticket, and an epic is not a ticket.** `WORKFLOW.md` starts
where a ticket has already been chosen and takes it to closed. A feature that will
take several branches has no single diff to review and no single merge to make, so
it gets broken down *before* the loop rather than planned around inside it — and
the same guard fires again at Plan, which is where a ticket most often turns out to
be two.

**References point one way, from the disposable thing to the durable one.** A
ticket dies; the trap it discovered does not — so the edge runs spec → trap, notes →
ADR, spec → risk, and never back. The graph stays acyclic, no pair can disagree, and
there is one place to update. What makes that affordable is that the reverse lookup
is a grep for the id, which is why every id is permanent and every prose tail beside
one — a folder slug, a trap heading, an ADR title — is decoration you may reword
freely. Renumbering is what makes a citation go quiet rather than visibly break, and
nothing renumbers. The single exception is
*provenance* — an ADR's "related work", a risk's "raised by" — which cites an issue
number, never a folder path, because issues are permanent and folders get deleted.

**The reference check looks for absences, not broken links.** A link checker tells
you a path failed to resolve. The failure that actually costs you is a reference
that *should exist and doesn't*: an ADR nobody cites, a risk raised in a spec with
no row in the register. Three greps, run when promoting out of `notes.md`, which is
the one step that creates a document and its citation at the same time.

**Tickets are independent.** Each owns a directory, an issue, a branch, and its
own status, so nothing has to be reordered, renumbered, or reopened when priorities
change. `PRODUCT.md` → Next is an unordered set, and dependencies are stated on the
ticket that has them rather than implied by position.

**Each stage gets a clean context, and the plan is written down.** Plan, implement,
and review in separate sessions. Carrying exploration context into implementation is
how scope creeps, and a reviewer holding the implementer's assumptions is not an
independent reviewer. Because context does not survive the boundary, the agreed plan
is written to `plan.md` and posted to the issue — a plan that exists only in a
session transcript does not exist. What the implementer loses at that boundary is
not the planner's judgement but the unwritten nine-tenths of what it read, so the
plan names files, call sites, precedent, and the things investigation could not
settle. A plan right in outline and vague in detail is the characteristic failure of
this shape, and it fails silently: it reads perfectly well until someone builds it.

**Subagents go inside a stage, never between two.** Between stages the handoff is a
written artifact. Inside one, delegation is free money wherever a step reads far
more than it keeps — so the planner sends out `investigator` briefs and keeps the
deciding, because planning is the stage that argues with the human and a subagent
has no way to ask. On a change ticket the briefs are *surveys*: where the code is,
what calls it, what precedent it should follow. On a **bug** they are *hypotheses*,
one candidate cause each, and every investigator is told to kill its own. Left alone
an agent finds one plausible explanation and stops looking, and every search after
the first bends toward it — which on a bug ticket corrupts item `.1` itself, since
there the riskiest assumption *is* the diagnosis. What survives being attacked is
evidence; what merely got confirmed is not, and the two read identically in a
report. Then the planner re-runs the finding before building on it, because a claim
nobody reproduced is a suspicion — the same rule Verify uses to settle a
disagreement between its two reviewers, one stage earlier.

**Plan the riskiest item first, then try to talk yourself out of the plan.** The
plan's item `.1` is the thinnest slice that proves the assumption most likely to be
wrong, not the easiest one — a wrong assumption found on the first afternoon is an
amendment and the same one found last is a rewrite. Then two passes that cost
minutes: a **pre-mortem** written in the past tense, because *what did go wrong* and
*what could go wrong* do not return the same list; and a sweep for **one-way doors**,
the items a `git revert` does not undo, each of which is either converted into a
reversible one or agreed with the human before it is built.

**Verify sends the branch to two reviewers at once, and consolidates.** The
`reviewer` scores this ticket's diff against its criteria and its tests; the
`architect` asks whether the tree the branch leaves behind is still coherent.
**Either may block, on its own judgement** — neither is advisory. Neither reads the
other's review, which now has to be said in both directions, because both post to
the same place.

**Both post to a PR, opened before either one runs**, each as its own review under
its own name. A later reader can see which pass caught what, and a finding keeps the
voice of the agent that made it. Consolidation is then the load-bearing step and it
happens on the PR: merge duplicates and say both raised it, settle a factual
disagreement by *running it* — a blocker refuted on fact does not gate, whoever
marked it — and give every finding exactly one disposition, dismissals included. Not
blocking does not mean optional; a finding nobody dispositioned is a finding that was
ignored with extra steps.

The PR is where the review is *recorded* and never what the reviewers read, since
`gh pr diff` sees only what was pushed while the review surface is the working tree.
It is also the half that lasts: `docs/work/` is disposable and `notes.md` is promoted
and dies with the ticket, while a merged PR keeps each finding attached to the line
that caused it.

**An approval covers the tail you described when you asked.** A bare "approved",
answering a summary of what happens next, authorises that whole tail — commit, merge,
close, report — and is not to be re-asked a step at a time; a narrower word
authorises only what it says. That makes the summary the contract, and puts the
burden where it belongs: on saying what is about to happen *before* asking.

**An ADR that briefs nobody has not landed.** Nothing loads `docs/DECISIONS/` by
default, so a decision recorded only there gets violated by the next session, which
had no reason to open the directory. The reasoning stays in the ADR; the rule it
implies is written into `.claude/rules/` or the non-negotiables in the same change,
and appears in exactly one place — where a rule is stated is where it is argued with.

**A test is done when it fails on a mutation, not when it passes.** Break the thing
it covers and confirm it goes red. `TESTING_TRAPS.md` catalogues eleven ways a green
suite proved nothing — the most common being that the rule is thoroughly tested and
the production wiring is tested by nothing. There is no mutation score and no
coverage threshold: the output is a named surviving mutation on the lines this change
touched, because a ratio invites a threshold and a threshold invites tests written to
move it.

**A bug fix owes an autopsy.** Which test should have caught this, and why didn't
it? The regression test stops that defect coming back; the autopsy is what stops the
next one, and it is the only measurement of a suite that arrives already paid for. A
new failure shape becomes a trap, a repeat becomes a `CLAUDE.md` gotcha. Skip it and
you accumulate one test per bug and never learn the shape you keep falling for.

**Three outcomes, not two.** Verified, Failed, and *Gated* — for what only real
hardware, real users, real load, or human eyes can settle. A gated criterion is
never a pass and never a failure; it is outstanding, and it names the check that
would close it. Two-outcome reporting is what turns "we didn't check" into "green".
Shipping with one outstanding takes a human accepting it on the record, against a
report posted to the issue first — and the criteria that must never ship unproven
say so in the spec, where they are written, rather than at the end with a finished
branch on the table.

**A rejected alternative is worth more than the chosen one.** Every spec numbers
the approaches that lost and why, including the honest *do nothing* baseline. The
chosen approach is visible in the code forever; the rejected one is visible
nowhere, so it gets re-proposed six weeks later by someone who only sees the
outcome, and the argument runs again from the start. Reasons local to the item
stay in its spec; a reason that constrains later work becomes an ADR.

**Every item says where it came from and what it is worth.** A `Source` line — a
person and a date, a review, a bug hit in anger — is who to ask when a criterion
turns out to be ambiguous, and what lets a stale item be killed honestly rather
than inherited. A `Priority` carries its reason on the same line, because a
priority you cannot argue with is a queue position wearing a label.

**Record divergence instead of restating it.** When what shipped differs from what
the spec said, both go in the document. Editing the spec to match the code leaves
something that looks like it was right all along and teaches nobody anything.

## Scaling down

The full set suits a project with a tracker and several things in flight. A stale
document is worse than a missing one, because it is still read — so on something
smaller, take a subset rather than filling in templates you will not maintain. Four
files carry most of the value:

```
CLAUDE.md                    stack, commands, non-negotiables, gotchas
docs/TESTING_TRAPS.md        keep verbatim
docs/DECISIONS/              the ADR template, and your first decision
.claude/settings.json        the permission posture
```

"Scaling down" in `ADAPTING.md` has the rest: which piece to add, and the specific
thing that has to go wrong before it earns its place.
