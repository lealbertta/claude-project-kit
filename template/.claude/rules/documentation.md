---
paths:
  - "docs/**/*"
  - "CLAUDE.md"
  - ".claude/**/*.md"
---

# Documentation Rules

- Keep canonical decisions in `docs/DECISIONS/` using the ADR template. A decision
  argued in a commit message or an issue comment is not recorded.
- Update `docs/PRODUCT.md` → Next when work is agreed, dropped, or parked. Status
  belongs in the tracker, never mirrored here.
- Update `docs/RISK_REGISTER.md` when a material risk is discovered or mitigated.
- Do not duplicate detailed requirements across several files. Link to the source
  of truth. When two documents disagree, the one named in `CLAUDE.md` §Sources of
  truth wins, and the other is corrected in the same change.
- Keep `CLAUDE.md` concise and broadly applicable. Move subsystem details to
  path-scoped rules or skills.
- Mark assumptions, unresolved decisions, and measured results clearly. A number
  that was computed and a number that was guessed must not read the same.
- When a document describes a command, keep the command runnable or label it as a
  template.
- **Record divergence rather than restating it.** When what shipped differs from
  what a requirement said, write down both and name the open point. Silently
  editing the requirement to match the code destroys the only evidence that a
  decision was ever made.
- A work item's `notes.md` dies with the item. Before it closes, promote anything
  that outlives it: a decision to an ADR, a vacuous-test shape to
  `docs/TESTING_TRAPS.md`, a risk to the register, a repeated correction to
  `CLAUDE.md` → Gotchas. Everything not promoted is understood to be disposable.
