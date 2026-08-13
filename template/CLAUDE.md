# <PROJECT>

<!--
FILL IN every <ANGLE_BRACKET>, then delete this comment.
This file loads in full, every session. Frontier models reliably follow ~150-200
instructions and the harness already uses ~50, so treat ~100 lines as the ceiling.
For each line ask: would removing it cause a mistake? If not, cut it.
Subsystem detail -> `.claude/rules/`. Procedures -> `.claude/skills/`.
Decisions -> `docs/DECISIONS/`. Anything that changes weekly -> not here at all.
-->

<One sentence: what this is and who it is for.>

## Stack

- <Language + version>
- <Framework + version>
- <Datastore / key libraries + versions>
- <Target platform and minimum supported version>

## Commands

- Install: `<...>`
- Run: `<...>`
- Test (one file): `<...>` — prefer this; the full suite is slow
- Test (all): `<...>`
- Lint / format: `<...>`
- Typecheck: `<...>`
- Build: `<...>`

## Structure

- `<src/core/>` — <deterministic rules, no framework dependency>
- `<src/app/>` — <composition root and entry point>
- `<tests/>` — <…>
- `docs/` — decisions, work items, architecture

## Non-negotiables

1. Write the failing test before the implementation, for rules, data migrations,
   and bug fixes. Where an automated test cannot express the behavior (feel,
   visuals, real-hardware performance), say so rather than writing a vacuous one.
2. **IMPORTANT: a test is done when it fails on a mutation, not when it passes.**
   Break the behavior it covers, confirm it goes red, and report which mutations
   you ran. See `docs/TESTING_TRAPS.md`.
3. Never claim a build, test, or performance target passed without output from the
   run itself.
4. Keep the change scoped to what was asked. No unrelated cleanup.
5. New work starts from a spec (`write-spec`), and a change spanning several files
   is planned before it is edited (`plan-feature`). No edits until the plan is agreed.
6. Hand nontrivial diffs to a fresh reviewer — the `review-change` skill or the
   `verification-reviewer` agent. Never self-review.
7. <Never <X> without an approved ADR (ADR-000N).>

<!-- Add two or three invariants of your own at 7, 8, 9 — the rules whose violation
     costs a rewrite. Everything else belongs in `docs/ARCHITECTURE.md`. -->

## Gotchas

<The non-obvious things that get got wrong. Add a line here every time you
correct Claude twice on the same thing — that is the signal a rule is missing.>

- <…>

## Repo etiquette

- Branch: `<convention>`
- Commit: `<convention>`
- `git push` stays a human step.
- Ask before editing <dependency manifests / build config / CI>.

## Where things are written down

- Product intent and what is next: `docs/PRODUCT.md`
- How work gets done: `docs/WORKFLOW.md`
- Architecture: `docs/ARCHITECTURE.md`
- Decisions: `docs/DECISIONS/`
- Tests: `docs/TEST_STRATEGY.md`; how they go vacuous: `docs/TESTING_TRAPS.md`
- Work items: `docs/work/<id>-<slug>/`
- Risks: `docs/RISK_REGISTER.md`

Read only what the task needs.
