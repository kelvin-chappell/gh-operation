# 0003 — One change proposal per target, opened as a draft, merged by the operator

- **Status:** Accepted
- **Date:** 2026-10-01
- **Deciders:** operator
- **Related:** `DESIGN.md` §1, §4, §9, §12; `CONTEXT.md` (Change Proposal, Disposition)

## Context

An operation applies the same task across many repositories. Two axes had to be settled:
how the work is packaged, and how much autonomy the system has.

- **Packaging:** one batched change across repositories, or one change per repository?
- **Autonomy:** merge automatically, or hold every change for a human?

One change spanning many repositories couples them: a single failing build blocks the
rest, and a rollback is all-or-nothing. Full autonomy across a fleet of repositories is a blast
radius we were not willing to accept before seeing the system work.

## Decision

Produce **exactly one change proposal per target** — one draft change proposal per
repository, and none where no change is needed. **Only the operator merges.** The task's
**intent is fixed**; its implementation may adapt within the specification's bounds, and
adaptations that would exceed those bounds stop at `needs operator` rather than widening
the diff.

## Consequences

**Positive**
- Each change is small, independently reviewable, and independently revertable.
- One repository's failure never blocks another's.
- The human gate contains blast radius until a reviewed pilot justifies more autonomy.

**Negative**
- Review load scales linearly with the target count; the feasibility report must
  therefore **lead with the target count** so the operator sees the load before
  approving.
- More change proposals and CI churn than one change spanning every repository would
  produce.
- Autonomy is deliberately left on the table for v1, to be earned later.
