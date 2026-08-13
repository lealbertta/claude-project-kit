# Notes — 042 Restore reads last support

Written during Implement and Verify, not reconstructed at the end. This file dies
with the work item; everything below that outlives it was promoted before #42
closed, and the promotion is named on the line.

## Mutations checked

| Mutation | Caught by |
|----------|-----------|
| Dropped `support` on serialize | `SerializedStateCarriesSupport` — 1 test, and nothing else. The round-trip suite stayed green, which is the whole finding below. |
| Restored via the beneath-query instead of the recorded id | `RestoreReadsLastSupport` — 3 tests |
| Emitted `support-lost` twice | `RestoreEmitsOnceOnMissingSupport` — 1 test |
| Suppressed `support-lost` entirely | `RestoreEmitsOnceOnMissingSupport` — 1 test |
| Read a missing field as id `0` instead of migrating | `LegacyFileRestoresWithoutSupport` — 2 tests |
| Reversed the offset sign when re-seating | **survived** — see below |

A mutation that survived is a finding. Record it even after you fix the test.

**The survivor.** Nothing compared the offset after a support move, so flipping its
sign was invisible: every existing assertion checked *which* support an object was
on, and AC-2 was being read as satisfied by "it moved with the shelf". Added the
offset comparison to `RestoreReadsLastSupport` and confirmed the flipped sign now
fails. It is the same shape as the bug this item exists to fix — an assertion on a
consequence rather than the rule — found twice in one item, which is what turned
the autopsy line into a `CLAUDE.md` → Gotchas entry rather than just a trap.

## Changed without being asked

- **`support-lost` now clears the stored reference** (`src/core/placement/restore`).
  Not in the plan and not in AC-3. Without it, every subsequent restore of the same
  object re-emits for a support that is already known to be gone, and the
  emit-once criterion is only true for the first restore. Recorded as a divergence
  on `spec.md` rather than by editing AC-3 to match what was built.

## Not closed

| Note | Owner |
|------|-------|
| Clearing the reference means a support that comes back cannot be re-adopted by objects that lost it. Nobody has asked for that; it is a decision, not an oversight. | Dana, open point on #42 |
| Deleting a support still leaves dangling references everywhere except this path. | #61 |
| `RestoreRoundTrip` still asserts on coordinates elsewhere in the file. Left alone deliberately — out of scope for this item, and worth its own pass. | #62 |

## Unreachable today

- A support id that is present but belongs to a different scene. Not reachable
  while scenes load whole and ids are minted per scene; becomes reachable the day
  cross-scene paste lands (#49). Recorded as **inert**, not as covered — the branch
  exists and no test drives it.

## Gated

| Criterion | What would close it | Batched with |
|-----------|---------------------|--------------|
| AC-4 | Restore a 200-object selection on the oldest supported device and read the frame time from the on-device profiler. | #47, which needs the same device session for its scroll measurement — one booking, two criteria. |

## Promoted before closing

- Trap → `TESTING_TRAPS.md`: *the assertion pins a consequence, not the rule*
  (second instance).
- Gotcha → `CLAUDE.md`: placement tests assert on the support reference, never on
  coordinates.
- Risk → `RISK_REGISTER.md`: dangling support references, mitigated only on the
  restore path.
- Decision → none. The serialized-format change was covered by ADR-0004; nothing
  here amended it.
