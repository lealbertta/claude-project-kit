# Plan — 042 Restore reads last support

Agreed 2025-03-05 and posted to #42 before implementation started, so the Implement
session read it instead of inheriting a planning context it never had.

**Non-goals:** the placement solver, multi-support objects (#58), and the drag
path — which reads the support correctly already and is the reason this bug is
only visible through restore.

| ID | Change | File(s) | Serves | Proven by | Mutation it must catch |
|----|--------|---------|--------|-----------|------------------------|
| 042.1 | Serialize the support id alongside the position. It is already stable and already unique; nothing new has to be minted. | `src/core/scene/serialize` | AC-1 | `SerializedStateCarriesSupport` | Drop the field on write — the round trip must fail, not fall back to a default |
| 042.2 | Restore seats against the recorded support instead of querying what is beneath. | `src/core/placement/restore` | AC-1, AC-2 | `RestoreReadsLastSupport` | Reinstate the beneath-query as the primary path — must fail even though every coordinate is still correct |
| 042.3 | Missing support: seat on the ground plane, emit `support-lost` once. | `src/core/placement/restore` | AC-3 | `RestoreEmitsOnceOnMissingSupport` | Emit twice; and separately, emit zero times — a test that only catches one of those is counting nothing |
| 042.4 | Migration: files written before this read as support-less and take the ground plane, which is what they already did. | `src/core/scene/migrations` | AC-1 | `LegacyFileRestoresWithoutSupport` | Bump the version without writing the migration — must fail loudly rather than reading a missing field as id `0` |

If you cannot name the mutation, the test is decoration and the plan is not
finished. 042.3 is the one that needed two: an emit-once criterion is two
independent failures and a single mutation only ever pins one of them.

**Implications**

- **Schema / stored data:** yes — the serialized object gains `support`. Forward
  migration in 042.4; old files stay readable and lose nothing they had.
- **Undo / redo:** this *is* the undo path. Redo shares it, and gets the fix free —
  which is why `RestoreReadsLastSupport` runs over both directions.
- **Performance:** one id lookup per restored object, against a map that is already
  built. Not none, but not measurable — AC-4 is the check that decides.
- **Compatibility:** files written after this land are unreadable by the previous
  release. Acceptable per ADR-0004; noted here because "none" would have been a lie.
- **Lifecycle:** an object deleted while something rests on it already leaves a
  dangling reference. That is not new, but 042.3 is the first code that has to have
  an answer for it.

**Unverifiable here:** AC-4. Needs the oldest supported device; a desktop timing
run is not evidence and does not get reported as one.

**Rollback:** revert the four changes together. Files written in between keep the
extra field, which the old reader ignores.

---

## Amendments

- **2025-03-06** — 042.3 assumed the restore path could emit directly. It cannot:
  events are drained once per frame and emitting mid-restore reorders them against
  the selection change. Smallest change that works — queue the event with the
  others and let the existing drain publish it. No criterion changes; the "exactly
  one" assertion moves to the drained queue, which is where a user would observe it
  anyway.
