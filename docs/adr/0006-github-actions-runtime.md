# 0006 — The runtime is GitHub Actions, with supervision as a scheduled job

- **Status:** Accepted
- **Date:** 2026-10-01
- **Deciders:** operator
- **Related:** `DESIGN.md` §1, §2 (constraint 5), §11, §12

## Context

The system must fan out explorations and actions, run reconciliation on a cadence, and
mint short-lived scoped tokens. Two homes were plausible: **GitHub Actions** (matrix
jobs, schedules, secrets, App tokens) or an **always-on service** we operate.

## Decision

Run on **GitHub Actions**. Operations fan out via a matrix; reconciliation runs on a
schedule tick (~15 minutes); the **supervisor is a scheduled job**, not a service.
Concurrency groups plus optimistic claiming make over-provisioning safe, so the
supervisor may dispatch slightly more actions than there are targets to hide startup
latency.

## Consequences

**Positive**
- Nothing to operate: no servers, no scheduler, no token store of our own.
- Naturally resumable and stateless per tick, which suits a planning function that
  re-reads the board each time.
- Secrets and per-job installation tokens are handled by the platform.

**Negative**
- Parallelism is finite (matrix and concurrent-job ceilings, runner minutes): "how many
  actions" is bounded by runner limits and budget, not by the number of eligible targets.
- Tick latency is minutes, not seconds; a service would be needed if responsiveness
  mattered.
- Actions run **untrusted repository code**, so sandboxing and least privilege are
  mandatory rather than incidental.
