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

042.2 is first in risk order rather than first in dependency order — 042.1 has to
land before it compiles, but 042.2 is the item that proves the whole idea works,
and it is where the plan would have died if the recorded id had turned out not to
be stable.

**Call sites**

Grepped rather than assumed, and it changed the plan once. `restore` has three
callers, not the two the spec implies: undo, redo, and **paste**, which seats
pasted objects through the same path. Paste gets 042.2 for free and AC-1's test
runs over it as well, which is cheaper than finding out later that a third caller
had its own copy of the beneath-query.

The serialized shape from 042.1 is read by the autosave writer and the document
loader (`src/app/documents`). Both pass the object through whole and needed no
change — checked, not assumed.

**Implications**

- **Schema / stored data:** yes — the serialized object gains `support`. Forward
  migration in 042.4; old files stay readable and lose nothing they had.
- **Undo / redo:** this *is* the undo path. Redo shares it, and gets the fix free —
  which is why `RestoreReadsLastSupport` runs over both directions.
- **Performance:** one id lookup per restored object, against a map that is already
  built. Not none, but not measurable — AC-4 is the check that decides.
- **Compatibility:** 042.4 bumps the schema version, so files written after this
  lands are refused by the previous release. Acceptable per ADR-0004; noted here
  because "none" would have been a lie.
- **Lifecycle:** an object deleted while something rests on it already leaves a
  dangling reference. That is not new, but 042.3 is the first code that has to have
  an answer for it.

**One-way doors**

- **042.1 with 042.4.** Reverting the code does not unwrite files already on disk.
  A user who saves once on this build has a file carrying the bumped version, and
  the reverted build refuses it. Made as reversible as it can be — 042.4's reader
  tolerates a missing `support`, so a revert still opens everything that opened
  before — and *not* made fully reversible, which was a decision and not an
  oversight: writing the old version back would mean maintaining two writers to
  protect a window measured in hours.

Nothing else here survives a revert. 042.2 and 042.3 are code paths only.

**Pre-mortem**

Written before this was presented, in the past tense.

1. **Old files stopped opening.** The version bump landed without the migration and
   every pre-existing file read its missing `support` as id `0`, seating everything
   on whichever object happened to be first. → 042.4, and its mutation is exactly
   this failure.
2. **Undo got slower on large selections.** One id lookup per restored object is
   nothing on a desktop and not obviously nothing on the oldest device at 200
   objects. → AC-4, which is why it is Gated rather than measured here.
3. **Nothing was actually fixed**, because the path we changed was not the one undo
   calls. → settled by the call-site grep above; it also turned up paste.

The third is the one that paid. The first two were already covered and the
pre-mortem only confirmed it, which is the normal outcome and not a reason to skip
the pass.

**Unverifiable here:** AC-4. Needs the oldest supported device; a desktop timing
run is not evidence and does not get reported as one.

**If it goes wrong**

**Nothing in the product would tell us.** A wrongly seated object looks correct
until the support next moves — that is the entire reason this bug lived a year —
and the `support-lost` event from 042.3 fires for the *missing*-support case, not
the wrong-support one. The signal is a user report, as it was the first time.

Undo is: revert the four changes together, accepting the one-way door above.

---

## Amendments

- **2025-03-07** — 042.5 added after the review pass: unpick the fixture helper in
  `RestoreEmitsOnceOnMissingSupport` so the missing support is visible in the test
  body, and add the offset comparison that the surviving mutation exposed
  (`notes.md`). It is `.5` and not `.3a` even though it belongs logically beside
  042.3 — the numbers are ids, the next free one is the one you get, and a commit
  already cites 042.3.
- **2025-03-06** — 042.3 assumed the restore path could emit directly. It cannot:
  events are drained once per frame and emitting mid-restore reorders them against
  the selection change. Smallest change that works — queue the event with the
  others and let the existing drain publish it. No criterion changes; the "exactly
  one" assertion moves to the drained queue, which is where a user would observe it
  anyway.
