# Risk Register

## What belongs here

Material risks: things that could invalidate the plan, the architecture, or the
schedule. One row each, with a named mitigation, the work that surfaced it, and the
ticket that acts on it.

## What does not belong here

Bugs (the tracker), or generic engineering hazards that apply to every project.

---

| ID | Risk | Likelihood / Impact | Mitigation | Raised by | Acted on by | Status |
|----|------|---------------------|------------|-----------|-------------|--------|
| R-01 | <one sentence> | M / Critical | <what reduces it> | #41 | #42 | Open |
| R-02 | <…> | L / M | <…> | ADR-0003 | — | Accepted |

**Likelihood:** L / M / H. **Impact:** L / M / H / Critical.

## Conventions

- **Raised by** is the ticket, ADR, or spike the risk came out of; **Acted on by** is
  the ticket that closes it. They are rarely the same one, and the first is what
  tells you whether the risk still applies once that work has landed. Both are
  **provenance** — they cite an issue number, never a `docs/work/` path, because an
  issue is permanent and a folder is deletable. Nothing reads them to decide
  anything.
- A risk with no mitigation and no owner is **accepted** — say so explicitly rather
  than leaving the column blank. An unspoken accepted risk is indistinguishable
  from a forgotten one.
- Close a risk with the evidence that closed it, not with a date.
- When a ticket touches a risk, name it in that ticket's `spec.md` → Risks. A
  register nobody cross-references stops being read. The sentence lives in the spec;
  the mitigation, owner, and status live here — not a copy in both.
- New risks most often come from what a change makes **reachable**. When an ADR
  fills in "what this makes reachable", check whether a row belongs here.
