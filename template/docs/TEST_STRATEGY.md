# Test and Verification Strategy

## What belongs here

Which layer covers what, the fixtures this project must maintain, the verification
commands, and what can only be settled outside the test suite.

## What does not belong here

How a specific test went vacuous (`TESTING_TRAPS.md`), or the workflow that runs
the tests (`WORKFLOW.md`).

---

## Principle

The developer and the assistant need executable evidence. A plausible code review
is not a substitute for a test run, a build, a fixture, or a real-environment capture.

## Test layers

### Fast / isolated

<!-- Deterministic rules with no environment. List the actual rule families. -->

- <Domain rule family 1>
- <Validation and boundary rules>
- <Transaction apply / undo / redo>
- <Serialization, migrations, corruption detection, unknown-record preservation>
- <Parsing, normalization, ID uniqueness>

Characterization tests belong here too: when the design depends on a behavior of a
library or runtime you do not control, pin that behavior in its own test. An
upgrade that changes it then fails loudly instead of invalidating the design silently.

### Integrated / environment

<!-- Things the fast layer cannot reach. -->

- Composition-root wiring
- Input and state-transition flows
- Resource loading and release
- Suspend / resume / reconnect
- <Rendering, UI virtualization, or other environment-only behavior>

Some behavior is only testable in the heavier layer, and the lighter one will pass
*vacuously* rather than fail. Note each such case here so nobody "moves the test
down" to make it faster.

### Gated — outside the suite entirely

<!-- Real hardware, real users, real load, human eyes. -->

- <Feel, latency, precision>
- <Thermals, memory, frame pacing, sustained load>
- <Visual judgement>
- <Low-resource and failure-recovery behavior>

A criterion in this list is never recorded as verified by a test run. Name the
check that would close it.

## Required fixtures

- <Schema v1 and every released version thereafter, in `<fixtures-dir>`>
- <A file containing a record the current code cannot interpret, so the
  preserve-and-re-serialize rule is pinned against real released bytes>
- <Edge-case data shapes: deepest valid nesting, a cycle attempt, boundary cases>
- <Scale fixtures, seeded, at the sizes that make the behavior engage>

**Never regenerate a fixture to make a test pass.** A fixture failure is a
migration to write.

Scale fixtures must be large enough that the mechanism actually engages. A
virtualized list that only recycles above ~40 rows, tested against 20, will pass
"zero rows created during a scroll" for the wrong reason.

## Verification commands

```bash
<./scripts/test.sh>              # narrow — deterministic rules
<./scripts/test-integration.sh>  # wiring, lifecycle, resources
<./scripts/verify.sh>            # full check, before closing a ticket
```

A task report must state which commands ran, where results were written, and any
gated criteria left outstanding.

## Performance baselines

Store a versioned note per representative capture, recording at minimum:

- Build commit
- Environment and version
- Configuration / quality tier
- Input seed and workload size
- <The metrics this project actually cares about>

Do not compare captures from different workloads without noting the difference.
