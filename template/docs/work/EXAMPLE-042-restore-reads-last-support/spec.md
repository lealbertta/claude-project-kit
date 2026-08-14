# 042 — Restore reads last support

<!--
A filled-in reference. The `EXAMPLE-` prefix keeps it out of the numbering; delete
the folder once your own tickets are better examples than this one.

The domain is a scene editor where objects rest on supports and can be stacked. It
is illustrative — commands and paths are the same placeholders the rest of the kit
uses. What is worth copying is the shape: what a criterion looks like when someone
else can check it, what a rejected alternative looks like when the reason survives,
and what a divergence looks like recorded rather than edited away.
-->

- **Issue:** #42
- **Status:** Done. The tracker is the truth and wins any disagreement.
- **Source:** Dana, 2025-03-04, hit live in a demo — dragged a lamp off a shelf,
  pressed undo, and the lamp came back on the floor.
- **Sized:** 1 session. Landed whole.
- **Priority:** High — this is the data-loss shape. Undo is the move people make
  when they are already unsure, and here it is the move that silently discards the
  arrangement they built.

## Problem

Every object records the support it rests on, and that reference is what makes a
stack behave like a stack: move the shelf and everything on it goes with it.
Restore rebuilds an object from its saved state but reads only the position, so a
restored object is re-seated against whatever is beneath it — usually the ground
plane.

The position is correct at the instant of restore, which is why this survived a
year. Nothing looks wrong until the next thing moves: then the object that should
have ridden the shelf upward stays where it was, or falls through it.

## What this changes

A restored object is re-seated against the support it recorded, not against the
nearest surface below it. Where the recorded support no longer exists, the object
falls to the ground plane **and says so**, rather than arriving there quietly and
looking like that is where it always lived.

## Acceptance criteria

- **AC-1** — restoring an object that rested on another object leaves it recorded
  as resting on that same object. Checked by reading the restored support
  reference, not by comparing coordinates.
- **AC-2** — moving the support after a restore moves the restored object with it,
  by the same offset.
- **AC-3** — restoring against a support that no longer exists seats the object on
  the ground plane and emits exactly one `support-lost` event naming the object and
  the missing support.
- **AC-4** `[Gated]` — restoring a 200-object selection holds the 16 ms frame
  budget on the oldest supported device. Closed by one session on that hardware;
  a desktop run proves nothing here and must not be reported as if it did.

## Alternatives considered

1. **Infer the support on restore from whatever is directly beneath the object.**
   Lost: it is the bug's own logic moved up a layer. Two objects at the same height
   are indistinguishable, and it makes the result depend on the order objects are
   restored in — so the failure comes back as an intermittent one, which is worse
   than the bug we have.
2. **Re-run the placement solver over the scene after every restore.** Lost:
   correct, and it reseats objects nobody touched. Undo would stop being a local
   operation, which is a bigger promise to break than the one being fixed.
3. **Store the support as an index into the scene list instead of a stable id.**
   Lost: cheaper to write and wrong the first time anything is deleted. We would
   be back here in a month with the same bug wearing a different symptom.
4. **Do nothing, and document that undo re-seats.** The honest baseline, included
   because the fix reaches into the serialized format and that is never free. It
   loses on what the documentation would have to say: *the one action that exists
   to undo a mistake is the one that discards your arrangement.*

## Autopsy

- **Should have been caught by:** `RestoreRoundTrip`. It existed, it covered this
  exact path, and it passed.
- **Why it wasn't:** it asserted on position. Restore produces the right position
  and the wrong support, so a position-shaped assertion is green in precisely the
  case that is broken.
- **Written down as:** a shape not already in `TESTING_TRAPS.md`, added to it as
  *the assertion pins a consequence, not the rule*. It then took a second victim
  inside this same item — see `notes.md` → the survivor — and a trap that does not
  hold where it is written is also a `CLAUDE.md` → Gotchas line:
  *placement tests assert on the support reference, never on coordinates.*

## Non-goals

- **Multi-support** — an object bridging two shelves. Deferred, filed as #58.
- **The placement solver** — untouched. Owned by the layout module; this item only
  changes what restore hands it.

## Risks

- **Raises** — persisting a support id makes a *dangling reference* reachable for the
  first time: a file can now name a support that no longer exists. Restore handles
  it (AC-3), but restore is the only path that does. Filed as R-04, raised by #42,
  mitigated on the restore path only.

## Open questions

| # | Question | Blocks | Resolution |
|---|----------|--------|------------|
| 1 | Does a restored object keep its offset on the support, or re-centre? | AC-2 | 2025-03-05, Dana: keeps the offset. Re-centring is a second bug wearing this one's clothes. |

## Divergences

| Criterion | Says | Shipped | Status |
|-----------|------|---------|--------|
| AC-3 | emits one `support-lost` event | emits the event **and** clears the stored support reference | Open point for Dana. Clearing was needed to stop the next restore re-emitting for the same object, but it was not in the criterion and it means a later undo cannot recover the reference if the support comes back. |
