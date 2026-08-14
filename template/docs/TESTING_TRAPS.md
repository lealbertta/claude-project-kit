# Testing Traps

## What belongs here

The specific ways tests in *this* project have passed while proving nothing. Add a
trap when you find one; each entry costs a paragraph and saves the next person a
release. Do not add general testing advice — that is what books are for.

The reliable way to find one is **the autopsy** below. Every trap here started as
something that got through.

**A trap's heading is its id.** Specs cite traps by name — `spec.md` → Autopsy says
which one a defect was a second instance of — and the reference runs one way, from
the ticket to the trap, because the ticket dies and the trap does not
(`.claude/rules/documentation.md` → Names and references). That makes the reverse
lookup a grep:

```sh
grep -rl 'Trap 3' --include=spec.md docs/work     # which defects hit this one
```

Which only works while the heading holds still. **Rewording a trap is renaming an
id**: every spec citing the old wording goes quiet rather than broken, and the
victim count — the thing that decides whether a trap has earned a `CLAUDE.md` →
Gotchas line — silently resets to zero. Add traps freely; rename them almost never,
and when you must, fix the citations in the same change.

## What does not belong here

Which tests to write (`TEST_STRATEGY.md`), or how the loop runs (`WORKFLOW.md`).

---

## The rule

**A test is not done when it passes. It is done when it fails on a mutation.**

Break the thing the test covers — delete the branch, invert the comparison, remove
the constant, revert the wiring — and confirm the test goes red. Record which
mutations you ran and what caught them.

The number that matters is not how many tests exist. It is how many distinct ways
you can break the code and still be caught.

**Do not mutate arid lines.** Logging, metrics, telemetry, and the wording of error
messages yield mutations that survive for reasons nobody should act on — and a
finding nobody should act on teaches people to dismiss the findings they should.
Mutate the rule, the branch, the constant, the comparison, and the wiring. Leave the
narration alone.

**Do not chase a score.** Killing every possible mutant is not the goal and is not
worth what it costs. The useful output is a *named surviving mutation on the lines
this change touched*, reported where someone can act on it. A ratio invites a
threshold, and a threshold invites tests written to move it.

---

## The autopsy — where every trap below came from

A defect that reached a human is the cheapest evidence about a suite anyone ever
gets, because it has already been paid for. Spend it: **a bug-fix ticket answers
two questions in its `spec.md` before the fix is written.**

1. **Which test should have caught this?** Name it. If none exists, that is the
   finding, and the regression test is the answer.
2. **Why didn't it?** It ran and asserted the wrong thing; it never ran at all; its
   fixture made the branch unreachable; it moved with the constant it was pinning.
   This is the answer worth having, and it is the one that gets skipped.

If (2) names a shape not already below, add it as a trap. If it names one that *is*
below, say which — a trap with a second victim has earned its place, and a rule
already written that did not hold is a `CLAUDE.md` → Gotchas line rather than a new
trap.

"Nothing could have caught it" is a legitimate answer exactly once per behavior:
record it as a gated check and name what would close it. Twice is the trap.

---

## Trap 1 — The rule is pinned, the wiring is not

The domain rule has thorough unit tests. Every one of them calls the rule directly.
Nothing calls the composed production path, so the line that wires the rule into
the application can be deleted, reverted, or given the wrong argument with the
entire suite green.

**Seen as:** a configurable limit that production constructed with the default,
never the configured value — "configurable" was true of the type and false of the
program. Reverting the wiring failed exactly one test, and only because someone
had written it on purpose.

**Fix:** at least one test drives the composed path. When a value is configurable,
that test must configure it to something *other* than the default — a test
asserting the shipped default passes with the feature deleted.

---

## Trap 2 — The assertion is written in terms of the constant it should pin

Every test expresses its inputs as `Solver.DefaultTolerance`, so every test moves
with the constant. Widening the tolerance tenfold passed the whole suite.

**Fix:** when a constant is itself a product promise, one test asserts it as a
**literal**. The symbolic tests still earn their keep for behavior around the
boundary; they simply cannot detect the boundary moving.

---

## Trap 3 — The fixture makes the branch unreachable

The test claims to prove that a tie resolves deterministically. The two candidates
are placed far enough apart that neither is ever in range, so no tie ever forms.
Both tests passed against a solver with the tie-break deleted.

The second attempt placed them close enough to tie and close enough to overlap —
an arrangement that cannot legally exist, so the tie was between two invalid states.

**Fix:** a fixture for a tie must satisfy *all* its conditions at once. **Assert in
the arrange step that the precondition holds** before asserting how it resolves.
Compute such inputs; do not guess at them. Both wrong versions looked obviously right.

---

## Trap 4 — Both sides of the comparison move together

An overflow test compared a computed area against a threshold derived from the same
inputs. Narrowing every integer cast in the file left the whole suite green, because
32-bit arithmetic wrapped *both* sides past the same multiple of 2³².

**Fix:** find the asymmetric case — inputs where one side wraps, saturates, or
rounds and the other does not — and pin it there.

---

## Trap 5 — The sequence is compared after collapsing it

A test asserted that a value changes at the right moment by comparing the collapsed
*sequence of changes*. On a monotone path across one boundary, every rule that flips
exactly once yields the same two-element sequence — including a "hold the previous
answer for N frames" damper, which the requirement forbade outright.

**Fix:** two tests, not one. Compare the value at **every sample point**, and run
the same path at **two sample densities** requiring agreement at every shared
position. The first alone accepts a temporal damper; the second is the only thing
that catches it.

---

## Trap 6 — The counting assertion has one half

"Browsing must never load full content" is asserted with a counting test double:
content requests are zero. That assertion also holds for a component that talks to
nothing at all, which is what it will become after the next refactor.

**Fix:** every counting assertion has two halves — the forbidden call count is
zero **and** the expected call count is above zero.

---

## Trap 7 — An inert branch claimed as covered

A flag, guard, or error path that no reachable input can currently trigger. The
test exercises it by calling the function directly, and the report says "covered".

**Fix:** record it as **inert**, with what would make it reachable. A branch that
is unreachable today and reachable after a named future change is a note carried
forward, not a covered case. Real example: a validation flag on the dragged item
was refused three times over before ever reaching the checked code path, and was
recorded as inert rather than counted.

---

## Trap 8 — The mutation is caught by accident

The test uses an input where the correct rule and the broken rule happen to agree,
or where the broken rule gives the right answer for the wrong reason. It fails on
some mutations and looks like a real test.

**Fix:** for each test, name the mutation it must catch, then run *that* mutation.
Seen as: a probe that appeared to prove the new selection rule, but whose input was
also selected correctly by the old bounding-box rule. It proved nothing until the
input moved to where the two rules disagree — and the test now asserts its own
hardness (the old rule, at this input, must give the wrong answer).

---

## Trap 9 — The absence claim with no direct form

"Changing X must not affect Y" often has no non-vacuous direct assertion: the
function under test does not take X, so it passes even with the feature deleted.

**Fix:** an **absence sweep** — assert structurally that X does not appear where it
must not (a grep-shaped test over the modules that must not know about it). Module
boundaries stop a reference; they do not stop someone copying a bare integer field
into a DTO.

---

## Trap 10 — A signal that is wrong often enough to ignore

A test that fails ~40% of runs for reasons unrelated to the code trains everyone to
re-run rather than read. It is worse than no test, because it also drains the
credibility of the tests around it.

**Fix:** diagnose it once. If the platform cannot distinguish the real signal from
the noise, **delete the test** and move that guarantee to a gated check — and record
the deletion with the conditions that would justify reopening it. A red that is
usually wrong is not a safety net.

---

## Trap 11 — Loosening a threshold quietly disarms unrelated tests

Relaxing a validity threshold made several existing tests stop discriminating: at
the old strict value they caught a millimetre of drift, at the new one they did not.
They still passed, and their comments still claimed teeth they had lost.

**Fix:** when a threshold moves, re-derive the geometry of every test that depends on
it and move each into the band where it still discriminates. Correct the comments
that describe what a test proves. This is the honest cost of loosening a rule.

---

## Two habits that prevent most of the above

**Argue why the obvious fixture is vacuous.** Before writing the test, say out loud
what makes *this* input able to fail. If the answer is "it just should", the fixture
is not chosen yet.

**Compute before booking scarce verification.** A quantity a user perceives in one
unit but authored in another (a tolerance, a margin, a threshold) is worth deriving
on paper first. In this project, arithmetic showed a tolerance measured 0.7–2.6
points depending on zoom — so *no single capture could settle it*. Minutes at a
terminal turned "unvalidated, needs a device session" into a located defect with
numbers. Scarce verification time is for what only the real environment can answer.
