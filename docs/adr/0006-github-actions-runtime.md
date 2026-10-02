# 0006 — The runtime is GitHub Actions, with supervision as a scheduled job

- **Status:** Superseded by [0011](0011-copilot-session-execution.md)
- **Date:** 2026-10-01
- **Deciders:** operator
- **Related:** `DESIGN.md` §1, §2, §11, §12; ADR 0008

## Context

The system must fan out explorations and actions, run reconciliation on a cadence, and
mint short-lived scoped tokens. Two homes were plausible: **GitHub Actions** (matrix
jobs, schedules, secrets, App tokens) or an **always-on service** we operate.

## Decision

Run on **GitHub Actions in each operation repository**, invoking pinned Copilot CLI.
Operations fan out via a matrix; reconciliation runs on a
schedule tick (~15 minutes); the **supervisor is a scheduled job**, not a service.
Local concurrency groups supplement, but never replace, atomic target-local claims
shared across operations. Approved definition commits govern execution; the creator
repository is not a live scheduler or shared runtime.

## Consequences

**Positive**
- No always-on scheduler service to operate; target-local coordination, external
  access bindings and authoritative credit reporting still require validation.
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
