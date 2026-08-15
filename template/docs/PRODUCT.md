# Product

## What belongs here

Why this exists, what it is not, and what is being worked on next. Two sections
with very different rates of change — the top half should be nearly frozen, the
bottom half is edited weekly.

## What does not belong here

Requirements for a specific piece of work (`work/<id>-<slug>/spec.md`),
architecture (`ARCHITECTURE.md`), or decisions (`DECISIONS/`).

---

## Vision

<Two or three sentences. What experience or outcome does this produce, and for
whom?>

## Pillars

The small number of things that, if removed, make this a different product. Three
to five. A feature request that serves none of them is a "no" by default.

- **<Pillar>** — <one line>
- **<Pillar>** — <one line>

## Baseline

- <Target platform / runtime, minimum supported version>
- <Performance floor>
- <Online / offline posture>

## Not building

Split these two, because "deferred" gets quoted back to you as a commitment:

**Deferred** — plausible later, not now.

- <…>

**Excluded** — a different product. Saying no once, here, is cheaper than saying
it four times in review.

- <…>

---

The tracker holds what is **in flight**. This half holds what has been agreed, what
has been parked, and why — the things a tracker holds badly. Do not mirror status
here; a second copy of it goes stale without anyone noticing.

## Next

Agreed, not started. **Unordered** — pick whichever is most useful when a slot
opens. If two genuinely must happen in order, say so on the line rather than
implying it by position.

- <…> — <one line>
- <…> — <one line, "needs #42 first">

## Someday

Ideas kept so they stop being re-proposed. No commitment implied.

- <…>

---

## Choosing what's next

`Next` is a set, not a queue. These are the considerations that make one ticket the
better pick — and the reasoning that tells you whether a proposed order is safe.
Update it when the reasoning changes.

- **Risk first.** Whatever could invalidate the plan goes early.
- **A spike is a ticket.** When feasibility is genuinely unknown, timebox an
  investigation with a go/no-go outcome, and let it be allowed to fail.
- **Rules before polish.** Work that changes the rules underneath work that tuned
  the feel wastes the tuning. Either sequence it, or accept the rework explicitly.
- **Do not book scarce verification twice.** When two items both need the same
  expensive check — a device session, a load test, a user test — batch them, or
  say why you are paying for it twice.
