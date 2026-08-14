---
paths:
  - "docs/**/*"
  - "CLAUDE.md"
  - ".claude/**/*.md"
---

# Documentation Rules

- Keep canonical decisions in `docs/DECISIONS/` using the ADR template. A decision
  argued in a commit message or an issue comment is not recorded.
- **An ADR ships with its rule.** The reasoning lives in the ADR; the rule it implies
  goes in `.claude/rules/` (one subsystem) or `CLAUDE.md` → Non-negotiables
  (everywhere), in the same change. Nothing loads `docs/DECISIONS/` by default, so an
  ADR nothing cites constrains nobody. See the `record-decision` skill.
- Update `docs/PRODUCT.md` → Next when work is agreed, dropped, or parked. Status
  belongs in the tracker, never mirrored here.
- Update `docs/RISK_REGISTER.md` when a material risk is discovered or mitigated.
- Do not duplicate detailed requirements across several files. **A rule appears in
  exactly one place — where it is stated is where it is argued with**; elsewhere,
  link to it, because two copies drift into two different rules. When two documents
  disagree, the one named in `CLAUDE.md` → Where things are written down wins, and
  the other is corrected in the same change.
- Keep `CLAUDE.md` concise and broadly applicable. Move subsystem details to
  path-scoped rules or skills.
- **Write down what was rejected, with the reason it lost.** A spec or PRD that
  records only the chosen approach cannot stop the rejected one being re-proposed,
  and cannot tell a later reader whether the choice still holds. Local reasons stay
  in the ticket; a reason that constrains later work becomes an ADR.
- Mark assumptions, unresolved decisions, and measured results clearly. A number
  that was computed and a number that was guessed must not read the same.
- When a document describes a command, keep the command runnable or label it as a
  template.
- **Record divergence rather than restating it.** When what shipped differs from
  what a requirement said, write down both and name the open point. Silently
  editing the requirement to match the code destroys the only evidence that a
  decision was ever made.
- A ticket's `notes.md` dies with the ticket. Before it closes, promote anything
  that outlives it: a decision to an ADR, a vacuous-test shape to
  `docs/TESTING_TRAPS.md`, a risk to the register, a repeated correction to
  `CLAUDE.md` → Gotchas. Everything not promoted is understood to be disposable.
