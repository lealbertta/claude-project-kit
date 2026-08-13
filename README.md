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
  CLAUDE.md                      Short, always loaded: stack, commands, structure, non-negotiables, gotchas.
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
      record-decision/             When to write an ADR, and how to amend rather than rewrite.
      sync-tickets/                Spec → tracker issues, idempotent, reports drift instead of resolving it.
    agents/
      architect.md                 Advisory, whole-tree. Convention drift, ADR conformance, duplication.
      architecture-reviewer.md     Read-only. Boundaries, data ownership, compatibility.
      verification-reviewer.md     Runs commands. Attacks the tests, reports per criterion.

  docs/
    PRODUCT.md                   Vision and pillars (frozen) + Now / Next / Someday (weekly).
    WORKFLOW.md                  Plan → Implement → Verify, per work item. Board state, outcomes, re-entry.
    ARCHITECTURE.md              Modules, dependency direction, data layers, seams.
    TEST_STRATEGY.md             Which layer covers what; fixtures; what only reality can settle.
    TESTING_TRAPS.md             Eleven ways a passing test proves nothing. Read before writing tests.
    RISK_REGISTER.md             Material risks, with mitigations and owners.
    DECISIONS/                   ADR template.
    work/TEMPLATE/               Copy per work item.
      spec.md                      What and why, acceptance criteria.        (before Plan)
      plan.md                      The agreed plan, with the mutation each test must catch.  (end of Plan)
      notes.md                     Findings, deferrals, mutations run.       (during Implement/Verify)
      PRD.md                       Only when the work spans several items.
```

## The ideas worth keeping if you keep nothing else

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

**A test is done when it fails on a mutation, not when it passes.** Break the thing
it covers and confirm it goes red. `TESTING_TRAPS.md` catalogues eleven ways a green
suite proved nothing — the most common being that the rule is thoroughly tested and
the production wiring is tested by nothing.

**Three outcomes, not two.** Verified, Failed, and *Gated* — for what only real
hardware, real users, real load, or human eyes can settle. A gated criterion is
never a pass and never a failure; it is outstanding, and it names the check that
would close it. Two-outcome reporting is what turns "we didn't check" into "green".

**Record divergence instead of restating it.** When what shipped differs from what
the spec said, both go in the document. Editing the spec to match the code leaves
something that looks like it was right all along and teaches nobody anything.

## Scaling down

The full set suits a project with a tracker and several things in flight. For
something smaller, see "Scaling down" in `ADAPTING.md` — the minimum useful subset
is four files.
