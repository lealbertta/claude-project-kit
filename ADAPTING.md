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
- [ ] `ask`: dependency manifests, build config, CI config, `git merge`, `git rebase`,
      `git push`.
- [ ] `deny`: secrets, generated output, build artifacts, logs, force-push, and
      destructive git.

The posture that works: friction-free for anything that produces evidence or can be
undone, confirmation for anything that changes shared state, refusal for anything
irreversible. Loosen it once the verification scripts are reliable — not before.

`git commit` sits on the allowed side of that line because nothing it writes has
left the machine, and the loop asks for one per verified increment. A prompt on
every commit is a prompt you stop reading, which costs more than it protects.

`git push` **asks rather than denies**, which is the one place this posture gives
ground, and it gives it for a reason: Verify opens a PR, and there is no PR without
a push. That makes it the loop's single outward step and therefore the one prompt
worth reading, rather than the tenth in a row you have stopped seeing. Force-push
stays denied — it is the push that destroys work instead of publishing it. `gh pr
create` and `gh pr comment` are outward too, and are on no list at all, so they
prompt by default. `merge` and `rebase` rewrite history and still ask.

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

### 5. `docs/TRACKER.md`

- [ ] The configuration table: repo, labels, board URL, project and field IDs. The
      test and verification commands are not here — `CLAUDE.md` → Commands owns
      those, and a second copy is the one that goes stale.
- [ ] Get the IDs with:

```bash
gh project list --owner <owner>
gh project field-list <N> --owner <owner> --format json
```

- [ ] Two labels are enough to start: a ticket label and a blocked label. Add the
      gated label the first time something ships with a criterion outstanding — it
      is what makes `gh issue list --label <gated>` the standing list of everything
      unproven. Add priority labels only if you will actually filter on them — the
      reason lives in the spec either way, and a label with no reason behind it is
      the thing this kit is trying not to have.
- [ ] **Decide who owns priority and who owns blocked, once** — a board field or a
      label, never both. `TRACKER.md` → One owner per fact has the argument; a
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
- [ ] No board? Keep **One owner per fact** and delete the rest of the file — that
      section is what tells you the spec's `Status:` line is the status rather than
      a copy. The loop works without a board; it does not work without the written
      plan.

### 6. First ticket

- [ ] Read `docs/work/EXAMPLE-42-restore-reads-last-support/` first — spec, plan,
      and notes, filled in. It is faster than reading the templates and it is the
      only place the kit shows what a rejected alternative, a gated criterion, and
      a recorded divergence look like when they are real rather than bracketed.
- [ ] `cp -R docs/work/TEMPLATE docs/work/<issue>-<slug>`, delete `PRD.md` unless
      the work spans several tickets, and fill in `spec.md`. Use the `write-spec`
      skill. The number is the issue's, with no leading zeros — `42-`, never `042-`.
- [ ] Delete the example folder once you have two tickets of your own. It cites a
      trap, a Gotchas line, a risk row and an ADR that the kit deliberately does not
      ship — its own header says so — so deleting it leaves nothing dangling behind.
      A borrowed example that outlives its usefulness gets cited as if it were house
      style.

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
| `docs/WORKFLOW.md` | More than one thing is in flight and the stages stop being obvious |
| `docs/TRACKER.md` | You put a tracker or a board in front of the work |
| `pick-ticket` | Choosing what to work on stops being obvious — more eligible tickets than you can hold in your head |
| `PRODUCT.md` → Next / Someday | You start forgetting what you decided not to do |
| `PRD.md` | One piece of work clearly needs several branches |
| `sync-tickets` | Keeping the tracker in step by hand becomes the annoying part |
| `RISK_REGISTER.md` | A risk survives more than one conversation |
| `investigator` | A bug first costs you two wrong diagnoses, or planning starts reading more than one session can hold |
| `architect` | The same pattern turns up in a third place, or an ADR stops describing the code |

`reviewer` and `architect` are the two halves of Verify, but they arrive in that
order for a reason: the reviewer answers a question you need answered on every
ticket, and the architect answers one that has nothing to say until the codebase is
big enough to be incoherent. Adding the architect on day three produces a paragraph
of "this is fine for now", which is the fastest way to teach yourself to skip it.

`investigator` arrives with the bugs rather than with the codebase. On a small
project the planner can hold the whole search in one head and the first diagnosis is
usually right; the day it is not, dispatching three of them is worth more than
anything else on this list.

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

The objection is always that the risky item depends on a duller one, and it is
usually a symptom rather than a constraint: the slice was cut along a layer instead
of through it. *Serialize the field* and *read it back* are one item, not two, and
the plan skill already says so — an item that only works once the next one lands is
not two items. Split that way, the first thing to land is a field nothing reads,
the suite passes either way, and the risky half is second. Where the dependency is
genuine and cannot be folded in, the plan names which item is the risky one, so
that the order reads as chosen rather than inherited.

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

**Why the planner dispatches subagents but is not one.** Planning reads far more
than it keeps, which is the exact shape delegation is for — and it is also the one
stage that argues with the human, which is the exact shape delegation is not for. A
subagent cannot ask a question; it reports once and stops. Make the planner a
subagent and the spec ambiguity it should have raised, the ticket that turns out to
be two, the one-way door that needed agreeing before it was built, all come back
either unmentioned or answered by assumption — and the assumption reads like a
decision by the time it reaches `plan.md`. So the split runs along a different seam:
the session keeps the judgement and sends out the reading. That is also why the
handoff *between* stages stays a written artifact and never a subagent. A stage
boundary is a context boundary, and the plan is the only thing that crosses it.

**Why `investigator` is one file and not four.** Survey and hypothesis briefs ask
the same kind of question — *what is actually there* — and differ only in what they
are pointed at, so they share a definition and get their difference from the
dispatch. `architect` and `reviewer` are two files because they genuinely ask
different questions and return different things; splitting the investigator the same
way would produce two files whose bodies you would have to keep in step by hand. It
is also not the built-in `Explore` agent, which skips `CLAUDE.md` to stay cheap:
worth it for a file search, wrong here, because an investigator that has not read the
non-negotiables will happily report a precedent that violates one.

**Why the hypothesis brief says "kill it" rather than "check it".** An agent asked
to confirm a cause confirms it — it finds the evidence that fits and stops, and the
report reads exactly like one from an agent that tried and failed to break the same
claim. Only one of those is worth planning against. The asymmetry costs nothing to
state and is the whole value of the brief. The parallel dispatch is the other half:
left to itself an agent finds one plausible explanation and stops looking, and every
search after the first bends toward it, so the diagnosis you end up with is the one
that occurred to you earliest. On a bug that lands directly on plan item `.1`, where
the riskiest assumption *is* the diagnosis — and a wrong one there is discovered
after the fix is built, or after it ships.

**Why a finding gets re-run before it is built on.** An investigator's report is a
claim, and a fluent one. Verify already has this rule for the case where two
reviewers disagree — settle it by running the thing rather than by taking the more
confident report — and Plan needs it more, because at Plan there is no second
opinion arriving to expose the first. Hence the `traced` / `ran` labels on every
line: they are not decoration, they are which claims the planner has to reproduce.
Skip that and the machinery produces confident diagnoses faster than anyone can
check them, which is worse than not having it.

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
that the half nobody has to act on gets skimmed and dropped. So consolidation has
rules: merge duplicates and say that both raised it, settle a factual disagreement
by *running the thing* instead of trusting the more confident report, and give every
finding exactly one disposition — new ticket, ADR proposal, risk row, watch, or
dismissed with the reason — all written down, dismissals included. That last part is
the whole mechanism. A finding does not need to block to be useful; it needs to be
impossible to leave unanswered.

**Why both reviewers can block, and what that replaced.** Each posts its own review
to the PR and marks what it judges blocking. The kit did not start here. The
original design let the architect block through exactly one checkable rule — a
violation of an Accepted ADR or a non-negotiable, on a line this branch changed —
and left everything else advisory, on the reasoning that the reviewer answers a
closed question that gates a merge while the architect asks an open one about the
whole tree, where the honest answer is often "this is fine for now". Make an open
question a gate, the argument ran, and it becomes either a rubber stamp or a fight
at the worst possible moment.

**That argument is still the one to weigh if you are reversing this.** What replaced
it is the view that an agent which has just read the whole tree is better placed to
judge what matters than a rule written before it looked, and that a rule narrow
enough to be checkable is also narrow enough to miss the finding you most wanted
caught. The cost is real and it is the one the old rule named: discretion widens.
The guards against it are that a blocker refuted on fact does not gate — consolidation
runs anything the two disagree on — and that scope remains the reviewer's axis, so
wanting a refactor is explicitly not grounds to block. Watch the architect's blockers
over a few months; if they trend toward "I would have built this differently", the
narrow rule is in the history and it worked.

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

It is not a free pass, and the fence has two halves. A gated criterion ships only
when a human **accepts it outstanding on the record**, answering a report already
posted to the issue rather than a sentence in a chat window — and the accepted check
becomes its own labelled issue, because the `notes.md` holding it dies with the
ticket. The other half is decided earlier: a criterion that must not ship unproven
is marked `[Gated, blocks merge]` when it is *written*, since at Verify the question
arrives with a finished branch on the table. That is the same reason one-way doors
are agreed before they are built and not after.

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
gets answered from memory. It is the closest thing to an exception to "never mirror
state", and it survives only because it is fenced: the tracker wins any
disagreement, nothing is gated on the copy, a stale one is cosmetic rather than
wrong, and **`sync-tickets` is the only thing that writes it** — one writer is what
keeps a copy from being a mirror, since a mirror is two writers and no owner.
Anything else — assignee, dates, percent complete — has no such excuse and goes
stale silently, which is the failure this kit spends most of its pages avoiding.

The fence only describes the configuration that has a tracker. **Without one the
line is not a copy at all** — it is the status, the stages move it, and the
reviewer's closing check reads it because there is nothing else to read. Which of
the two you are in is not something a spec template can know, so the authority on
it is `docs/TRACKER.md` → One owner per fact, and everything else defers there.

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
