# Notes — 42 Restore reads last support

Written from Plan onward, not reconstructed at the end. This file dies with the
ticket; everything below that outlives it was promoted before #42 closed, and the
promotion is named on the line.

## Causes ruled out

Three `investigator` hypotheses went out together at Plan, each told to kill its
own. One survived and became 42.1; these are the other two, plus the one the
survivor displaced.

| Candidate cause | Refuted by |
|-----------------|------------|
| The beneath-query picks the wrong object at the contact point, because the float tolerance is too loose. | `ran` — the query returns the correct support on a fully built scene at every tolerance in the range. The query is right; **calling it at restore time is what is wrong**, because the scene it queries is half-rebuilt and "beneath" is a different answer than it was at save. |
| Undo restores objects before their supports, so the support is not there yet to be found. | `ran` — the restore order is dependency-first and has been since the ordering rewrite. It would have produced this exact symptom, which is why it is written down here rather than left as something someone re-checks in six months. |
| The drag path records the wrong support in the first place. | `traced` — drag writes the support it actually seated against, verified at the write. This is why the bug is visible only through restore, and it is on the plan's Non-goals for the same reason. |

**The first row is the one that pays for this table.** It is the explanation that
occurs to everyone first, it survives a casual look, and a session that started
there would have spent itself tuning a tolerance — arriving at a fix that made the
symptom rarer and left the cause in place. It was killed in nine minutes by an
investigator whose only job was to kill it.

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
| `RestoreEmitsOnceOnMissingSupport` builds its fixture through a helper that hides which support is missing. | reviewer, `should` | Fixed in 42.4. Tests read top to bottom or they are not evidence anyone checks. |

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

| Criterion | Why not here | What would close it | Batched with | Decision |
|-----------|--------------|---------------------|--------------|----------|
| AC-4 | No oldest-supported device in this environment. The desktop run finished in a quarter of the budget, which says nothing: the device is where the margin disappears. | Restore a 200-object selection on the oldest supported device and read the frame time from the on-device profiler. | #47, which needs the same device session for its scroll measurement — one booking, two criteria. | Accepted outstanding by Dana, 2025-03-07 → #63, labelled `gated`. |

**If it fails:** restoring a large selection drops frames on the oldest supported
device — a stutter on undo, visible to the user, and not data loss. Undone by
reverting 42.1 to the beneath-query, which is the bug this item fixed, so the real
answer would be to cache the support lookup rather than to roll back.

That trade is what Dana accepted, and it went up on #42 **before** the ask, not
after — the acceptance answers a report, not a sentence in a chat window. AC-4 was
not tagged `[Gated, blocks merge]` in the spec, which is why it was an acceptance
question at all: a frame budget that misses is a follow-up, not a wrong feature.

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
- Gated check → #63, labelled `gated`, carrying AC-4 and the device session that
  would close it. It is an issue and not a ticket — there is no branch to open and
  nothing to merge — and it exists because this file dies with the ticket, which
  closed while the check was still outstanding.
- New tickets → two, both from the architect and both in `PRODUCT.md` → Next: merge
  the two remaining beneath-query copies, and delete `<config/test-globs>`.
