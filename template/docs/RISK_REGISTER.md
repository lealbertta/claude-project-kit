# Risk Register

## What belongs here

Material risks: things that could invalidate the plan, the architecture, or the
schedule. One row each, with a named mitigation and the phase that acts on it.

## What does not belong here

Bugs (the tracker), or generic engineering hazards that apply to every project.

---

| ID | Risk | Likelihood / Impact | Mitigation | Acted on by | Status |
|----|------|---------------------|------------|-------------|--------|
| R-01 | <one sentence> | M / Critical | <what reduces it> | Phase <N> | Open |
| R-02 | <…> | L / M | <…> | — | Accepted |

**Likelihood:** L / M / H. **Impact:** L / M / H / Critical.

## Conventions

- A risk with no mitigation and no owner is **accepted** — say so explicitly rather
  than leaving the column blank. An unspoken accepted risk is indistinguishable
  from a forgotten one.
- Close a risk with the evidence that closed it, not with a date.
- When a work item touches a risk, name the risk in its `spec.md`. A register
  nobody cross-references stops being read.
- New risks most often come from what a change makes **reachable**. When an ADR
  fills in "what this makes reachable", check whether a row belongs here.
