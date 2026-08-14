# Adapting the kit

## Fill-in checklist

Work top to bottom. Under an hour for the whole pass.

### 1. `CLAUDE.md` — the only file that loads every session

- [ ] The one-line description of what this is and who it is for.
- [ ] **Your role** — who the agent is, who it works with, and what that person
      knows that it does not, so a domain question has somewhere to go. Then the
      goal you share, stated so it can be traded against, and the tension in it:
      what pulls toward shipping the smallest thing, and what makes a merely
      plausible answer unshippable anyway. Skip this and the non-negotiables below
      read as ceremony with no cause behind them.
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

- [ ] `allow`: your verification commands, so test runs never prompt. `git commit`
      is here too — see below.
- [ ] `ask`: dependency manifests, build config, CI config, `git merge`, `git rebase`.
- [ ] `deny`: secrets, generated output, build artifacts, logs, `git push`, and
      destructive git.

The posture that works: friction-free for anything that produces evidence or can be
undone, confirmation for anything that changes shared state, refusal for anything
irreversible. Loosen it once the verification scripts are reliable — not before.

`git commit` sits on the allowed side of that line because `git push` is denied, so
a commit changes nothing anyone else can see, and the loop asks for one per verified
increment. A prompt on every commit is a prompt you stop reading, which costs more
than it protects. `merge` and `rebase` rewrite history and still ask.

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
      today than reconstructed in six months. Write the *rule* each one implies into
      `.claude/rules/` or `CLAUDE.md` → Non-negotiables in the same pass — nothing
      loads `docs/DECISIONS/` by default, so an ADR nothing cites constrains nobody
      (`record-decision`).

### 5. `docs/WORKFLOW.md`

- [ ] The configuration table: repo, labels, board URL, project and field IDs,
      test and verification commands.
- [ ] Get the IDs with:

```bash
gh project list --owner <owner>
gh project field-list <N> --owner <owner> --format json
```

- [ ] Two labels are enough to start: a ticket label and a blocked label. Add
      priority labels only if you will actually filter on them — the reason lives in
      the spec either way, and a label with no reason behind it is the thing this
      kit is trying not to have.
- [ ] **Decide who owns priority and who owns blocked, once** — a board field or a
      label, never both. `WORKFLOW.md` → One owner per fact has the argument; a
      field and a label for the same fact will disagree within a month, and the
      field is the one people sort by.
- [ ] Confirm the board's **item closed → Done** automation is enabled. It is on by
      default, it can be switched off, and the workflow tells the agent never to set
      `Done` by hand — so if it is off, every ticket parks in `In review`.
- [ ] Check the token you will use has `project` scope. Projects is GraphQL-only;
      `GITHUB_TOKEN` in Actions does not have it, so board moves that work locally
      can fail in CI with an error that reads like a missing project.
- [ ] Raise the `--limit` in the item lookup past your board's **total** item count,
      Done ones included. The default of 100 fails silently — the item is simply not
      in the page, and nothing says so.
- [ ] No board? Delete the Board state section. The loop works without one; it does
      not work without the written plan.

### 6. First ticket

- [ ] Read `docs/work/EXAMPLE-42-restore-reads-last-support/` first — spec, plan,
      and notes, filled in. It is faster than reading the templates and it is the
      only place the kit shows what a rejected alternative, a gated criterion, and
      a recorded divergence look like when they are real rather than bracketed.
- [ ] `cp -R docs/work/TEMPLATE docs/work/001-<slug>`, delete `PRD.md` unless the
      work spans several tickets, and fill in `spec.md`. Use the `write-spec` skill.
- [ ] Delete the example folder once you have two tickets of your own — **and the
      `R-04` row in `RISK_REGISTER.md` with it**, since the two are a cross-reference
      pair and the reference check will report the survivor. A borrowed example that
      outlives its usefulness gets cited as if it were house style.

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
| `plan-ticket-implementation` + `implement-ticket` | A change first spans several sessions |
| `docs/work/<id>-<slug>/` | You first lose track of what a change was for |
| `reviewer` | You first ship something a self-review missed |
| `docs/WORKFLOW.md` | You have a tracker and more than one thing in flight |
| `PRODUCT.md` → Next / Someday | You start forgetting what you decided not to do |
| `PRD.md` | One piece of work clearly needs several branches |
| `sync-tickets` | Keeping the tracker in step by hand becomes the annoying part |
| `RISK_REGISTER.md` | A risk survives more than one conversation |
| `architect` | The same pattern turns up in a third place, or an ADR stops describing the code |

`reviewer` and `architect` are the two halves of Verify, but they arrive in that
order for a reason: the reviewer answers a question you need answered on every
ticket, and the architect answers one that has nothing to say until the codebase is
big enough to be incoherent. Adding the architect on day three produces a paragraph
of "this is fine for now", which is the fastest way to teach yourself to skip it.

With no tracker installed, the spec header block is the whole system: `Status:` is
read where the tracker would have been, and `grep -rl 'Priority:\*\* High'
--include=spec.md docs/work` is how the next thing to do gets found. That is the one
configuration where the spec's own fields are authoritative rather than a copy —
worth knowing before the first time you add a tracker and have to say which side
wins.

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

**Why tickets get their own folders.** The source project tracked everything in
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

**Why the plan's first item is the riskiest and not the easiest.** Ordering by
convenience means the assumption the whole plan rests on gets tested last, when the
only remaining options are expensive. Ordering by risk — the thinnest slice that
runs end to end through every layer the change touches, first — is the walking
skeleton idea applied at the scale of one ticket: prove the chain works while
being wrong is still cheap. A wrong assumption found on the first afternoon is an
amendment. The same assumption found on the third day is a rewrite, and by then
there is a branch to argue about.

**Why the pre-mortem is written in the past tense.** Because the tense is doing the
work. Asking "what could go wrong" and asking "what did go wrong" are not the same
question: people are markedly better at explaining an outcome than at forecasting
one, and the premortem borrows that fluency for an outcome that has not happened.
The effect has been measured — Mitchell, Russo and Pennington reported prospective
hindsight improving the identification of reasons for a future outcome by around
30%, and Klein's 2007 HBR write-up is where the practice got its name. Treat the
percentage as folklore and the direction as real, which is the same posture this kit
takes toward the context-window numbers. It costs a few minutes and it is where a
plan most often actually changes. The discipline that makes it more than theatre is
that every answer has to name where it went — a plan item, a test, a risk row, or
*Unverifiable here*. A pre-mortem answer with nowhere to go was not an answer.

**Why one-way doors get their own heading in the plan.** "Rollback: revert the
commits" is true of most changes and quietly false for the ones that matter. Data
already written in the new format, an id minted, a message published, a column
dropped — a revert does not reach any of them, because something else has already
read the output. Naming them at plan time is what makes the alternative visible
while it is still free: the standard answer is to split the change *expand →
migrate → contract* so each half reverts on its own, and the standard failure is a
single item that swaps the old form for the new and looks exactly like every other
row in the table. Where it genuinely cannot be made reversible, the point of the
heading is that a human agreed to it before it was built rather than after.

**Why the plan grep for call sites is a named step.** The most common reason a plan
gets amended mid-implementation is not a bad design; it is a file that had four
callers the plan did not know about. Naming the file you will edit is cheap and
feels like investigation. Listing what reads it is the part that actually converts
an assumption into a fact, and it is skippable precisely because nothing goes wrong
until later. "None beyond the files above" is a fine answer — it is just not one you
get to give without having looked.

**Why review agents are separate, and why neither can edit.** A reviewer that fixes
things stops reporting them, and the finding disappears into a diff nobody reads —
so neither `architect` nor `reviewer` gets `Edit` or `Write`. Both do get `Bash`,
one to read git history and the other to actually run the suite, and that is exactly
the grant that could slide from reviewing into fixing. Where the tool list stops
being the constraint the instructions have to be, so each says what its `Bash` is for
and states outright that it never modifies the tree.

**Why both run, and why they run at the same time.** Verify dispatches the reviewer
and the architect together, neither seeing the other's report. Running them in
sequence is cheaper and worse: the second one reads the first, and a finding it
would have raised independently arrives instead as agreement with something already
on the page. Two reviews that have read each other are one review and a
confirmation of it. The cost of independence is that they will sometimes duplicate
each other and occasionally contradict each other outright — which is the point.
Independent agreement is the strongest signal the pair produces, and independent
*disagreement about a fact* is the second strongest, because it means something is
ambiguous enough that two careful readers got different answers.

**Why consolidation is a step and not a formatting exercise.** Two reports do not
compose into next steps on their own, and the failure mode is not confusion — it is
that the advisory half gets skimmed and dropped. So consolidation has rules rather
than judgement: merge duplicates at the reviewer's severity, settle a factual
disagreement by *running the thing* instead of trusting the more confident report,
and give every architect finding exactly one disposition — new ticket, ADR proposal,
risk row, watch, or dismissed with the reason — all of which are written down,
dismissals included. That last part is the whole mechanism. Advisory findings do not
need to block to be useful; they need to be impossible to leave unanswered.

**Why the architect still does not carry a verdict, with one exception.** The
reviewer answers a closed question — does this change do what its spec said, do its
tests have teeth — and the answer gates a merge. The architect asks an open one about
the whole tree, where the honest answer is often "this is fine for now". Make that a
gate and it becomes either a rubber stamp or an argument at the worst possible
moment. So it returns no verdict, and consolidation promotes exactly one class of
its findings to a blocker: a violation of an **Accepted** ADR or a non-negotiable,
**on a line this branch changed**. That is not an exception to the principle so much
as an admission that this particular finding was never the open question — it is the
reviewer's own convention axis, and the architect happened to reach it first.

Both conditions are checkable from the architect's output, which is why its report
carries a `rule:` citation and an `in-diff:` flag on every finding including the many
where the answers are `none` and `no`. A promotion rule that reads a field the
architect fills in at its own discretion is not a rule; it is the architect deciding
what blocks, one honest-sounding sentence at a time.

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

**Why there is no mutation score, and no coverage threshold.** Coverage measures
that a line ran, not that anything checked it — the correlation with test
effectiveness largely disappears once you control for how many tests were written,
and methods at full line coverage routinely still have untested behaviour. Mutation
is the better signal, but the industrial lesson from running it at scale is that
mutation *adequacy* is neither practical nor desirable: what works is mutating only
the changed lines, suppressing arid ones, and delivering each survivor as a review
comment on a specific line. So the kit reports a named surviving mutation and never
a ratio. A ratio invites a threshold, a threshold invites tests written to move it,
and the cheapest way to move either number is to execute lines without asserting
anything.

**Why a bug fix owes an autopsy.** A regression test stops that one defect coming
back. The autopsy — which test should have caught this, and why it didn't — is what
stops the next one, and it is the only measurement of the suite that arrives already
paid for. Without it a project accumulates one test per bug and never learns the
shape it keeps falling for, which is exactly how a file like `TESTING_TRAPS.md` ends
up empty in a codebase that badly needs it. The answer has somewhere to go: a new
shape is a trap, a repeat is a `CLAUDE.md` → Gotchas line, because a trap claiming a
second victim means the rule is not holding where it is currently written.

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

**Why a spec records what it rejected.** The approach that shipped is legible from
the code for as long as the code exists. The three that lost are legible from
nothing, so the first person who did not sit in the conversation proposes one of
them again — and because the reason it lost was never written, the argument runs
from the beginning, with less information than it had the first time. A rejection
with no reason attached does not help: "we preferred the other one" is the part
everyone already inferred. What transfers is the cost — the case it handles worse,
the thing it makes unreachable, the work it doubles. The *do nothing* baseline is
in the list for the same reason and one more: it is the only line that asks
whether the item was worth doing, and an item nobody asked that about is how a
backlog fills with work that was merely proposed.

**Why the spec carries a `Status:` line when the tracker owns status.** Because a
spec gets pasted into a chat window, and one that cannot say what state it is in
gets answered from memory. It is the single exception to "never mirror state", and
it survives only because it is fenced: the tracker wins any disagreement, nothing
is gated on the spec's copy, and a stale one is cosmetic rather than wrong. The
fence is what makes it safe, so it is written down in `sync-tickets` next to the
rule it breaks. Anything else — assignee, dates, percent complete — has no such
excuse and goes stale silently, which is the failure this kit spends most of its
pages avoiding.

**Why there is a priority at all, in a kit that refuses to order work.** Priority
and sequence are different claims, and conflating them is what made the source
project's ordered task table unusable. A sequence says *this cannot start until
that lands*, which is a fact about dependencies and lives on the item as *Depends
on*. A priority says *this is worth doing sooner*, which is a judgement, is
frequently wrong, and goes stale — so it is written with its reason attached and
in the same line, where the next reader can see it has expired. `Next` stays an
unordered set; priority narrows what to pick from it, and never overrides a
dependency.

**Why the kit ships one filled-in ticket.** Every other file here is a template
with angle brackets, which is honest about what is missing and useless as a model.
People do not learn a documentation habit from a form; they learn it from one
completed instance they can copy the *tone* of — how specific a criterion has to be
before someone else can check it, how short a rejected alternative can be and still
carry its reason, what a divergence reads like when it was recorded instead of
edited away. It costs one folder and it is the first thing worth deleting once the
project has two of its own.
