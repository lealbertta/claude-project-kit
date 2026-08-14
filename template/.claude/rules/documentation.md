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

## Names and references

### Every id has one form

| Family | Form | Example |
|--------|------|---------|
| Ticket | the issue number, **unpadded** | `42`, addressed as `#42` |
| Plan item | `<ticket>.<n>` | `42.3` |
| Acceptance criterion | `AC-<n>` | `AC-2` |
| Requirement, in a PRD | `FR-<n>` | `FR-7` |
| Risk | `R-<nn>` | `R-04` |
| ADR | `ADR-<nnnn>`, in `<nnnn>-<slug>.md` | `ADR-0004` |
| Trap | the heading `Trap <n>` — cite it by **name** | `Trap 3` |
| Spec→issue join label | `spec:<ticket>:<FR>` | `spec:42:FR-7` |

**Pad an internal sequence; never pad an id that mirrors an external one.** ADRs
and risks are ours and pad so they sort. A ticket's id *is* its issue number, so
padding it invents an `042` ↔ `#42` transformation that nothing enforces and
everyone has to remember — and a rule kept only in people's heads is the one that
produces a folder nobody can find.

**Every number here is permanent.** One added later takes the next free value even
where it belongs logically in the middle: tickets, commits, and prior comments
already cite the old ones, and renumbering breaks every reference silently.

**Nothing under `docs/work/` carries a leading zero, in either configuration.**
With a tracker the number is the issue's. Without one it is a plain counter, which
by the rule above would pad — and does not, because the day a tracker arrives the
project would otherwise hold two renderings of the same kind of id and a migration
nobody planned. One rendering means every glob, grep, and citation written today
still works then.

**Every ticket folder is one directory deep**, including the children of an epic.
An epic's folder holds its `PRD.md` and nothing else; its tickets sit beside it, and
each names it on the spec's `Part of:` line. So `docs/work/42-*/` resolves any
ticket without first knowing whether it belongs to a feature — and the directory
check below, which only scans one level, sees all of them rather than silently
skipping the nested ones.

**The slug is decoration and is never part of a key.** `docs/work/42-restore-reads-last-support/`
is identified by `42`; the rest is for humans and may be reworded at any time. A
ticket's directory resolves by number — `docs/work/42-*/` — which is why the join
label carries the number and not the slug.

### References point one way, from the disposable to the durable

A ticket dies; the trap it discovered does not. So the edge runs `spec.md` → `Trap 3`,
`notes.md` → `ADR-0004`, `spec.md` → `R-04`, and **never back**. The reference graph
stays acyclic, no pair can disagree, and there is exactly one place to update.

Two consequences worth stating, because they are what make the one-way rule
affordable:

- **The reverse lookup is a grep, so the name it greps for must be stable.**
  "Which tickets hit Trap 3?" is `grep -rl 'Trap 3' --include=spec.md docs/work`.
  Numbers are already permanent; this extends the same promise to **trap headings and
  ADR titles**, which are cited as text. Reword one and every citation to it goes
  quiet — not broken, which would at least be visible, but silently unfindable.
- **Provenance is the one thing a durable document may record, and it cites the
  issue.** An ADR's `Related work:` and a risk row's `Raised by` answer "what made us
  decide this", which is worth keeping. They name `#42` and never
  `docs/work/42-<slug>/`, because an issue is permanent and a folder is deletable.
  They are provenance, not maintained links, and nothing reads them to decide
  anything.

### Checking the references

The failure this catches is not a path that fails to resolve — it is a reference
that **should exist and doesn't**. A link checker cannot see those, because they
are absences. These can:

```sh
# ADRs nothing cites. "An ADR nothing cites constrains nobody" is the rule above;
# this is the only way to find one.
for f in docs/DECISIONS/[0-9]*.md; do
  case "$f" in *0000-*) continue;; esac        # the template is not a decision
  id="ADR-$(basename "$f" | cut -d- -f1)"
  grep -rqF "$id" --include='*.md' --exclude-dir=DECISIONS docs .claude CLAUDE.md \
    || echo "uncited: $id"
done

# Risks raised in a spec with no row in the register. \b keeps FR-7 out of it.
grep -rhoE '\bR-[0-9]+' --include=spec.md docs/work | sort -u | while read -r r; do
  grep -qE "^\| *$r " docs/RISK_REGISTER.md || echo "no register row: $r"
done

# Ticket directories whose number disagrees with the issue their spec names.
for d in docs/work/[0-9]*/; do
  [ -f "${d}spec.md" ] || continue             # an epic folder carries PRD.md
  n=${d#docs/work/}; n=${n%%-*}
  grep -qE "Issue:.*#${n}\b" "${d}spec.md" || echo "id mismatch: $d"
done
```

Run them when promoting out of `notes.md`, which is when a new ADR, trap, or risk
row is created and therefore when a citation most often fails to get written.
