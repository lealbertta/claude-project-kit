# Adapting the kit

## Fill-in checklist

Work top to bottom. Under an hour for the whole pass.

### 1. `CLAUDE.md` — the only file that loads every session

- [ ] The one-line description of what this is and who it is for.
- [ ] **Stack** — languages, frameworks, datastores, with versions. This is what
      stops the agent proposing an API your version does not have.
- [ ] **Commands** — install, run, test-one-file, test-all, lint, typecheck, build.
      The most-used facts in the file. Name the fast test command explicitly, or
      the slow one gets run every time.
- [ ] **Structure** — the three to five directories that matter, one line each.
- [ ] **Non-negotiables** — rules 1–6 are supplied and load-bearing; keep them.
      Add two or three project invariants of your own, phrased as prohibitions with
      an escape hatch: *"Never <X> without an approved ADR."* A rule with no way to
      change it gets violated silently; a rule with one gets amended on the record.
- [ ] **Gotchas** — leave empty at first. Add a line every time you correct the
      agent twice on the same thing. That repetition is the signal a rule is missing,
      and this list is where the kit actually earns its keep over months.
- [ ] Repo etiquette: branch and commit conventions, what needs asking first.

Then check the length. Past ~100 lines, something in it is mis-shelved: subsystem
detail → `.claude/rules/`, procedure → `.claude/skills/`, decisions →
`docs/DECISIONS/`, anything that changes weekly → not here at all.

`AGENTS.md` is a symlink to `CLAUDE.md`, so Cursor, Codex, and Copilot read the same
file. Keep the symlink; edit `CLAUDE.md`.

### 2. `.claude/settings.json`

- [ ] `allow`: your verification commands, so test runs never prompt.
- [ ] `ask`: dependency manifests, build config, CI config, `git commit`.
- [ ] `deny`: secrets, generated output, build artifacts, logs, `git push`, and
      destructive git.

The posture that works: friction-free for anything that produces evidence,
confirmation for anything that changes shared state, refusal for anything
irreversible. Loosen it once the verification scripts are reliable — not before.

### 3. `.claude/rules/*.md`

- [ ] Replace the `paths:` globs. **This is the mechanism** — a rule file auto-loads
      only when the agent touches a matching path. A wrong glob means it never
      loads and nothing tells you.
- [ ] `code.md`: replace the language specifics; keep the structure.
- [ ] Delete `data-and-migrations.md` if nothing persists.
- [ ] Verify a glob fires: open a file that should match, confirm the rule appears.

### 4. `docs/`

- [ ] `PRODUCT.md`: vision, three to five pillars, baseline, and the deferred /
      excluded split. Then Now / Next / Someday — Next is **unordered on purpose**.
- [ ] `TEST_STRATEGY.md`: your layers, your fixtures, your commands.
- [ ] `TESTING_TRAPS.md`: keep verbatim. It costs nothing until it saves a release,
      and you will add project-specific entries within a month.
- [ ] `ARCHITECTURE.md`: the module table and dependency direction, at minimum.
- [ ] Write ADR-0001 for whatever you already decided while setting this up. You
      have made three or four decisions by now; they are worth more written down
      today than reconstructed in six months.

### 5. `docs/WORKFLOW.md`

- [ ] The configuration table: repo, labels, board URL, project and field IDs,
      test and verification commands.
- [ ] Get the IDs with:

```bash
gh project list --owner <owner>
gh project field-list <N> --owner <owner> --format json
```

- [ ] Two labels are enough: a work-item label and a blocked label.
- [ ] No board? Delete the Board state section. The loop works without one; it does
      not work without the written plan.

### 6. First work item

- [ ] `cp -R docs/work/TEMPLATE docs/work/001-<slug>`, delete `PRD.md` unless the
      work spans several items, and fill in `spec.md`. Use the `write-spec` skill.

---

## Scaling down

A stale document is worse than a missing one, because it is still read. Take a
subset rather than filling in templates you will not maintain.

**Minimum useful subset (four files):**

```
CLAUDE.md                    stack, commands, non-negotiables, gotchas
docs/TESTING_TRAPS.md        keep verbatim
docs/DECISIONS/              ADR template + your first decision
.claude/settings.json        permission posture
```

**Add as the project earns them:**

| Add | When |
|-----|------|
| `.claude/rules/` | A subsystem has rules that are noise elsewhere |
| `plan-feature` + `implement-feature` | A change first spans several sessions |
| `docs/work/<id>-<slug>/` | You first lose track of what a change was for |
| `reviewer` | You first ship something a self-review missed |
| `docs/WORKFLOW.md` | You have a tracker and more than one thing in flight |
| `PRODUCT.md` → Next / Someday | You start forgetting what you decided not to do |
| `PRD.md` | One piece of work clearly needs several branches |
| `sync-tickets` | Keeping the tracker in step by hand becomes the annoying part |
| `RISK_REGISTER.md` | A risk survives more than one conversation |
| `architect` | The same pattern turns up in a third place, or an ADR stops describing the code |

Do not start with all of it on a weekend project. Do start with the `CLAUDE.md`
non-negotiables and `TESTING_TRAPS.md`, including on the weekend project.

---

## Why the shape is what it is

Notes from the project this was extracted from — the reasoning behind the parts
that look fussy.

**Why the instruction file is short.** It loads in full, every session, for the
life of the project, and every line competes with the code the agent needs to read.
Community measurements put reliable instruction-following at a couple of hundred
items with the harness already consuming some of that; treat the number as folklore
but the direction as real. The failure mode is not a missing rule — it is a file so
long that none of it is read closely.

**Why work items get their own folders.** The source project tracked everything in
one ordered task table instead. It worked until it didn't: the order implied
dependencies that did not exist, finishing anything out of sequence meant editing
the table, and the per-task status cells grew into thousands of words of findings
that existed nowhere else and could not be found by anyone looking for them. A
folder per item fixes all three — independent status, no implied order, and
`notes.md` that is explicitly disposable once its durable findings are promoted.

**Why the plan is written to a file and posted to the issue.** The three-session
split only works if the plan survives the session boundary. It is also a forcing
function: a plan you must write as a table of changes mapped to criteria and to the
mutation each test must catch is a plan you have actually thought through.

**Why review agents are separate, and why neither can edit.** A reviewer that fixes
things stops reporting them, and the finding disappears into a diff nobody reads —
so neither `architect` nor `reviewer` gets `Edit` or `Write`. Both do get `Bash`,
one to read git history and the other to actually run the suite, and that is exactly
the grant that could slide from reviewing into fixing. Where the tool list stops
being the constraint the instructions have to be, so each says what its `Bash` is for
and states outright that it never modifies the tree.

**Why the architect is advisory and the reviewer is not.** The reviewer answers a
closed question — does this change do what its spec said, do its tests have teeth —
and the answer gates a merge. The architect asks an open one about the whole tree,
where the honest answer is often "this is fine for now". Make that a gate and it
becomes either a rubber stamp or an argument at the worst possible moment, so its
output is work items and ADR proposals a human weighs later instead.

**Why the reviewer's verdict is mechanical.** `NEEDS WORK` if and only if there is at
least one `blocker`, and every finding is tagged with exactly one severity when it is
written. The alternative — a reviewer weighing its own findings into an overall
impression at the end — is a loop that does not terminate, because the same branch
can read as *nearly there* on one pass and *not quite* on the next with nothing having
changed. Deciding severity per finding, and deriving the verdict by counting, is what
makes "send it back to Implement" an answer rather than a mood. It also forces the
useful discipline: to call something a blocker you have to say which line and what
fix, and a finding that cannot survive that is a `should`.

**Why mutation checks are in the loop rather than a nice-to-have.** In the source
project, an independent review ran mutations the implementing session had not, and
found five cases that had passed a 921-test suite: production wiring nothing pinned,
a mapping that was an identity on current data, an ordering nothing compared, a
guard branch nothing reached, and two fields nothing compared. None were bugs in the
change. All were tests that did not work. The suite was large and mostly green and
that was not evidence of much.

**Why "Gated" is a first-class outcome.** Without it, a criterion only real hardware
can settle gets reported as passing (false) or failing (also false, and it blocks
the merge). It becomes passing. The third outcome — plus naming the check that would
close it — is what keeps an honest list of outstanding items instead of a green
board with holes in it.

**Why acceptance criteria never become tickets.** An AC is how work is judged, not a
unit of work. A subtask per criterion manufactures busywork and separates
verification from the thing verified, so a task closes with its criteria sitting in
unclosed children. One ticket per requirement, criteria in the body as the
definition of done.

**Why the ticket mapping is a label, not a file.** A hand-maintained
`ticket-map.json` is a third source of truth: nobody updates it when an issue is
filed in the tracker's UI, and it conflicts on every branch. A `spec:<id>:<FR>` label
on the issue is the same mapping with no maintenance, and it survives renames and
moves. Anything local is generated from it and never hand-edited.

**Why every deferred note names an owner.** One six-slice phase generated dozens of
"we noticed X and did not fix it". The ones that named the ticket that owned them
got closed. The ones that did not, did not.
