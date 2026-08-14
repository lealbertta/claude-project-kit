# Notes — 042 Restore reads last support

Written during Implement and Verify, not reconstructed at the end. This file dies
with the ticket; everything below that outlives it was promoted before #42
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

## Review findings and their disposition

Both reviewers ran at once on the same tree, neither having seen the other's
report. Consolidated verdict: **READY FOR HUMAN REVIEW** — no blockers survived
the merge.

| Finding | From | Disposition |
|---------|------|-------------|
| The beneath-query is implemented three times — restore, drag, and the layout preview. This branch deleted the restore copy; two remain. | architect, `duplication`, in-diff: no | **New ticket**, filed in `PRODUCT.md` → Next. Not absorbed here: the reviewer is enforcing scope on the same diff and wins inside the branch. Merging the other two is a good idea and a different ticket. |
| The branch establishes a rule nothing has written down — placement state is restored from a recorded reference, never re-derived — and it binds work that does not exist yet. | architect, `adr-gap`, in-diff: yes | **ADR proposal**, drafted for Dana. `rule: none`, so it did not promote to a blocker: the architect found a rule that *should* exist, which is not the same as a rule that does. |
| `LegacyFileRestoresWithoutSupport` sits in a file matched by no suite config and never runs. | architect, `dead-code`, in-diff: yes | **Dismissed** — and see below. It does run. |
| AC-4 cannot be settled here. | reviewer, `question` | Reported outstanding, not as a pass and not as a failure. See Gated. |
| `RestoreEmitsOnceOnMissingSupport` builds its fixture through a helper that hides which support is missing. | reviewer, `should` | Fixed in 042.5. Tests read top to bottom or they are not evidence anyone checks. |

**The two reviewers disagreed about a fact**, which is the interesting row. The
reviewer recorded the suite green with `LegacyFileRestoresWithoutSupport` among the
passes; the architect read the project config and concluded the file could not be
matched. Settled by running it rather than by trusting the more confident of the
two: it runs. The architect had read `<config/test-globs>`, which the runner stopped
consulting two releases ago and which nobody deleted. **The disagreement was the
finding** — a stale config that reads as authoritative outlives this ticket, so its
deletion is the second new ticket this review produced.

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

- Trap → `TESTING_TRAPS.md`: *the assertion pins a consequence, not the rule* — a
  shape the file did not have.
- Gotcha → `CLAUDE.md`: placement tests assert on the support reference, never on
  coordinates.
- Risk → `RISK_REGISTER.md`: R-04, dangling support references, mitigated only on
  the restore path. Named in `spec.md` → Risks when it was raised, not discovered at
  the end — the register row is the promotion, the spec line is the cross-reference.
- Decision → none *made here*. The serialized-format change was covered by ADR-0004
  and nothing amended it. The architect's `adr-gap` finding is a **proposal** for
  Dana to accept or reject, not a decision this ticket took: an agent that writes
  the ADR it recommended has approved its own recommendation.
- New tickets → two, both from the architect and both in `PRODUCT.md` → Next: merge
  the two remaining beneath-query copies, and delete `<config/test-globs>`.
