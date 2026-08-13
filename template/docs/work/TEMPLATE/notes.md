# Notes — <id> <title>

What this work item found. Written as you go, not reconstructed at the end — an
observation that lives only in a session's context is lost when the session ends.

This file dies with the work item. Anything that will still matter in six months
gets **promoted** before this item closes:

- A decision → `docs/DECISIONS/`
- A way a test went vacuous → `docs/TESTING_TRAPS.md`
- A risk → `docs/RISK_REGISTER.md`
- A rule Claude keeps getting wrong → the Gotchas list in `CLAUDE.md`

## Mutations checked

| Mutation | Caught by |
|----------|-----------|
| <what was broken> | <which tests failed, and how many> |

A mutation that survived is a finding. Record it even after you fix the test.

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

| Criterion | What would close it | Batched with |
|-----------|---------------------|--------------|
| AC-<n> | <the specific check> | <other items waiting on the same session> |
