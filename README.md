# Claude Project Kit

A starting set of agent files for a new project: a short always-loaded instruction
file, path-scoped rules, on-demand skills, review subagents, a permission posture,
and a documentation layer built around independent work items.

Extracted from a real multi-phase project, then stripped of everything specific to
its stack. Nothing here assumes a language, framework, or platform.

## What it is for

Three problems, one system:

1. **Context budget.** Everything the agent needs, none of it loaded until it is
   relevant. The instruction file stays short; subsystem rules load on path match;
   procedures load as skills when invoked.
2. **Evidence.** Work is planned before it is written, verified by someone who did
   not write it, and reported with the commands that prove it — and a test does not
   count until it has been shown to fail on a mutation.
3. **No false sequencing.** Work items are independent by default, so nothing has
   to be reordered or renumbered when priorities change.

## Install

```bash
cp -R template/. /path/to/new-project/
```

Then work through `ADAPTING.md`. Nothing here works until the `<ANGLE_BRACKET>`
placeholders are filled in — that is deliberate; a template that runs unedited is a
template nobody reads.

Add to the new project's `.gitignore`:

```
CLAUDE.local.md
.claude/settings.local.json
```

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
      documentation.md             Where each kind of fact belongs; record divergence, don't restate it.
      testing.md                   Test-first, mutation-checked, wiring pinned, gated criteria named.
      data-and-migrations.md       Schema versioning, atomic writes, forward-only migrations, preserve-unknown.
      code.md                      Language-agnostic skeleton — replace the specifics, keep the structure.
    skills/                      Load on demand, by name.
      write-spec/                  Draft a work item spec, or a PRD when it spans several.
      plan-feature/                Plan stage. Produces a plan and nothing else.
      implement-feature/           Implement stage. Test-first, increment-verified, stops when reality diverges.
      review-change/               Independent review — of the tests as much as the code.
      record-decision/             ADR or work item, how to amend rather than rewrite, and shipping the rule with it.
      sync-tickets/                Spec → tracker issues, idempotent, reports drift instead of resolving it.
    agents/
      architect.md                 Advisory, whole-tree. Convention drift, ADR conformance, duplication.
      reviewer.md                  Work-item-scoped. Runs the checks, attacks the tests, verdicts the branch.

  docs/
    PRODUCT.md                   Vision and pillars (frozen) + Now / Next / Someday (weekly).
    WORKFLOW.md                  Plan → Implement → Verify, per work item. Board state, outcomes, re-entry.
    ARCHITECTURE.md              Modules, dependency direction, data layers, seams.
    TEST_STRATEGY.md             Which layer covers what; fixtures; what only reality can settle.
    TESTING_TRAPS.md             Eleven ways a passing test proves nothing, and the autopsy that finds the next one.
    RISK_REGISTER.md             Material risks, with mitigations and owners.
    DECISIONS/                   ADR template.
    work/TEMPLATE/               Copy per work item.
      spec.md                      Source, priority, what and why, acceptance criteria, alternatives considered.  (before Plan)
      plan.md                      The agreed plan, with the mutation each test must catch.  (end of Plan)
      notes.md                     Findings, deferrals, mutations run.       (during Implement/Verify)
      PRD.md                       Only when the work spans several items.
                                   `sync-tickets` generates a fourth file, `tickets.md`, where a tracker is in use.
    work/EXAMPLE-042-…/          A filled-in work item. Read it before writing your first. Delete it after.
```

## The ideas worth keeping if you keep nothing else

**The agent has a role, and the goal has a tension in it.** `CLAUDE.md` opens by
naming who the agent is, who it works with, what that person knows that it does not,
and the goal they share — stated so it can be traded against. Then it names the
tension in that goal, because there always is one: ship the smallest thing that
helps, except where a wrong answer costs someone something real, and there
*plausible* is not shippable. Without that paragraph the non-negotiables read as
arbitrary ceremony instead of as the price of a specific risk.

**Work items are independent.** Each owns a directory, an issue, a branch, and its
own status, so nothing has to be reordered, renumbered, or reopened when priorities
change. `PRODUCT.md` → Next is an unordered set, and dependencies are stated on the
item that has them rather than implied by position.

**Each stage gets a clean context, and the plan is written down.** Plan, implement,
and review in three separate sessions. Carrying exploration context into
implementation is how scope creeps, and a reviewer holding the implementer's
assumptions is not an independent reviewer. Because context does not survive the
boundary, the agreed plan is written to `plan.md` and posted to the issue — a plan
that exists only in a session transcript does not exist.

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

The full set suits a project with a tracker and several things in flight. For
something smaller, see "Scaling down" in `ADAPTING.md` — the minimum useful subset
is four files.
