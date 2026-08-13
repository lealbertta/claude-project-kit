# Plan — <id> <title>

Written in the Plan stage, agreed before any edit, and **posted to the issue** so
the Implement session can read it without inheriting the planning context.

**Non-goals:** <what this deliberately does not touch>

| ID | Change | File(s) | Serves | Proven by | Mutation it must catch |
|----|--------|---------|--------|-----------|------------------------|
| <042>.1 | <what and why> | `path` | AC-1 | `TestName` | <what to break so it fails> |

If you cannot name the mutation, the test is decoration and the plan is not
finished.

The IDs are `<item>.<n>` and they are **cited from commit subjects** — `042.3:
pin restore against the ground-plane fallback`. That is what keeps partial
progress legible in `git log` on a branch that lands over three days, and what
lets a reviewer map a commit to the criterion it serves without reading the diff.
The numbers are permanent: a change inserted later takes the next free number even
where it belongs logically in the middle. Never renumber, never reuse.

**Implications** — one line each, or "none" **with the reason**. "None" without a
reason is the sentence that hides the migration.

- Schema / stored data:
- Undo / redo:
- Performance:
- Compatibility:
- Lifecycle:

**Unverifiable here:** <criteria needing a real device, real users, or eyes, or "none">

**Rollback:** <how to undo this if it goes wrong>

---

## Amendments

When reality contradicts the plan, amend it here rather than improvising. Each
amendment says what was assumed, what is actually there, and the smallest change
that works.

- **<date>** — <…>
