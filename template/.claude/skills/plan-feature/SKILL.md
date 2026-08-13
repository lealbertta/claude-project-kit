---
name: plan-feature
description: Explore and plan a nontrivial feature before implementation. No code changes.
---

# Plan a Feature

This is the Plan stage of `docs/WORKFLOW.md`. It produces a plan and nothing else.

1. Read `CLAUDE.md` and only the project documents and path-scoped rules relevant
   to this change.
2. Read the work item's `spec.md` and every acceptance criterion in full.
3. Restate the requested behavior, explicit non-goals, affected systems, and
   assumptions.
4. Inspect the current code, tests, and configuration before proposing a design.
   Do not design against what you assume the code looks like.
5. Identify implications: data model, persisted schema, undo/redo, performance,
   compatibility, and lifecycle. One line each, or "none" **with the reason** —
   "none" without a reason is the sentence that hides the migration.
6. Propose the smallest vertical slice that proves the risky behavior.
7. List the files and modules likely to change, interfaces to add or reuse, and
   tests to write.
8. For each test, name the mutation it must catch. If you cannot name one, the
   test is decoration and the plan is not finished.
9. Mark any criterion that this environment cannot settle (real hardware, real
   users, visual judgement) and name the check that would close it.
10. Identify migration and compatibility concerns, and the rollback plan.
11. Do not edit implementation files. Output the plan and wait for approval.

Ask hard questions of your own plan before presenting it: which item here is
load-bearing and which is speculative; what does this change make *reachable*
that was previously unreachable; and what existing rule was written when the
world was simpler than it is now.

## Output format

```markdown
## Plan — #<ticket> <title>

**Non-goals:** <what this deliberately does not touch>

| # | Change | File(s) | Serves | Proven by | Mutation it must catch |
|---|--------|---------|--------|-----------|------------------------|
| 1 | <what and why> | `path` | AC-0 | `TestName` | <what to break so it fails> |

**Implications:** schema / undo / performance / compatibility / lifecycle —
one line each, or "none" with the reason.

**Unverifiable here:** <criteria needing a real device, real users, or eyes, or "none">

**Rollback:** <how to undo this if it goes wrong>
```

On approval, post the plan verbatim to the ticket before ending the session. The
implementation happens in a fresh session and will not inherit this context.
