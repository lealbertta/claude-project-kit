# Plan — <id> <title>

Written in the Plan stage, agreed before any edit, and **posted to the issue** so
the Implement session can read it without inheriting the planning context.
`plan-ticket-implementation` is the authority on how each section below is filled.

**Non-goals:** <what this deliberately does not touch>

| ID | Change | File(s) | Serves | Proven by | Mutation it must catch |
|----|--------|---------|--------|-----------|------------------------|
| <42>.1 | <what and why> | `path` | AC-1 | `TestName` | <what to break so it fails> |

If you cannot name the mutation, the test is decoration and the plan is not
finished.

**Item `.1` is the riskiest thing here, not the easiest** — the thinnest slice that
runs end to end and proves the assumption most likely to be wrong. A wrong
assumption found on the first afternoon is an amendment; found last, it is a
rewrite.

The IDs are `<item>.<n>` and they are **cited from commit subjects** — `42.3:
pin restore against the ground-plane fallback`. That is what keeps partial
progress legible in `git log` on a branch that lands over three days, and what
lets a reviewer map a commit to the criterion it serves without reading the diff.
The numbers are permanent: a change inserted later takes the next free number even
where it belongs logically in the middle. Never renumber, never reuse. They are
**ids and not an order**: where an item genuinely cannot land before another, say
so on the later one.

**Call sites:** <what reads or calls the thing that changes, and which item covers
each. "None beyond the files above" is a fine answer once you have actually
grepped for it; it is the assumption that gets a plan amended mid-implementation.>

**Implications** — one line each, or "none" **with the reason**. "None" without a
reason is the sentence that hides the migration.

- Schema / stored data:
- Undo / redo:
- Performance:
- Compatibility:
- Lifecycle:

**One-way doors** — items a `git revert` does not undo: data already written in the
new format, an id minted, a message published, a column dropped. Name each with
what would make it reversible instead — usually splitting it *expand → migrate →
contract* so each half reverts on its own. "None, everything here is undone by
reverting" is the common answer and is worth writing down.

- <item — why a revert does not undo it — what would make it a two-way door>

**Pre-mortem** — written in the past tense, because *what did go wrong* and *what
could go wrong* do not return the same list. It is a month from now; this shipped
and it went wrong. Each answer names where it went: a plan item, a test, a risk
row, or *Unverifiable here*.

1. <it went wrong because …> → <where that went>

**Unverifiable here:** <criteria needing a real device, real users, or eyes, or "none">

**If it goes wrong:** <how we would notice it in the wild — or why nothing would,
which is itself worth knowing — and how to undo it>

---

## Amendments

When reality contradicts the plan, amend it here rather than improvising. Each
amendment says what was assumed, what is actually there, and the smallest change
that works.

- **<date>** — <…>
