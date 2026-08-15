# Architecture

## What belongs here

Module boundaries, dependency direction, data layering, and the seams that exist on
purpose. The invariants in `CLAUDE.md` are the short form of this document; this is
where they are explained.

## What does not belong here

Decisions and their alternatives (`DECISIONS/`), or per-feature design (the PRD).

---

## Strategy

<Two or three sentences: the organizing idea. What is kept dependency-free and
testable, what is allowed to touch the framework, and why the split falls there.>

## Modules

| Module | Owns | May depend on |
|--------|------|---------------|
| `<core>` | <deterministic rules, no framework> | — |
| `<domain>` | <…> | `<core>` |
| `<persistence>` | <…> | `<core>` |
| `<adapters>` | <framework integration> | `<core>`, `<domain>` |
| `<app>` | <composition root, entry point> | all |
| `<tests>` | — | all |

Dependencies must remain **acyclic**. A cycle is not a style problem; it is the
thing that makes "domain rules are testable without the framework" quietly false.

## Folder layout

```
<tree>
```

## Composition

<Where dependencies are wired, and the rule that domain code never locates its own
collaborators. Name the actual file.>

## Data layers

The same concept usually needs several types. Collapsing them because they
currently have the same fields is the change you regret three phases later.

### Authoring model

<What a human or tool writes. Optimized for editing.>

### Runtime model

<What the program uses. Optimized for lookup and correctness.>

### Persisted DTO

<What is written to disk or the wire. Optimized for stability and versioning. Never
contains framework references.>

### Live instance

<What exists at runtime with identity and mutable state.>

State explicitly which conversions exist and which direction they run. A conversion
that exists in only one direction is a design statement worth making out loud.

## Transactions

<How mutations that must be undoable are represented, applied, inverted, and
batched. One path, so undo, redo, autosave, and batch operations cannot disagree.>

An inverse must be **applicable**. A rule enforced on apply that a legitimate
inverse would violate will throw on an undo button — this is a real bug shape, not
a theoretical one.

## Boundaries with external systems

<Each external dependency, the single adapter that owns it, and the rule that
feature code never calls it directly.>

## Compatibility

<Which contracts are frozen, what versioning scheme applies, and what constitutes a
breaking change.>

## Extensibility checkpoints

Places where a future requirement is expected to attach, and the shape it should
take. Also record where you deliberately did **not** leave a seam, so its absence
does not read as an oversight.
