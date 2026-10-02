# 0001 — The board is a GitHub Projects v2 project, one per operation

- **Status:** Accepted
- **Date:** 2026-10-01
- **Deciders:** operator
- **Related:** `DESIGN.md` §2, §4; `CONTEXT.md` (Board, Stage, Target Standing); ADRs 0007 and 0008

## Context

An operation needs a record of every target and the standing it currently sits in — a
record both the operator and the supervisor read each tick. The candidates were:

- **Labels** on issues (cheap, but no fields, no views, awkward to query);
- a **database** we run ourselves (flexible, but a datastore to operate and a second
  place standings can live);
- **GitHub Projects v2**, the platform's own board.

Projects v2 has a hard quirk: its *items* are issues, change proposals, or draft
issues — **not repositories**. A repository cannot be a board item directly.

An earlier decision also fixed **one board per operation**, so that operations never share
standings or concurrency.

## Decision

Use **GitHub Projects v2**, and create **exactly one project per operation**. Because a
repository cannot be an item, each target is represented by a **tracking issue** in the
operation repository, and that issue is the target's entry on the board. The board's fields
carry the machine-readable fields: `Standing`, `Stage`, `Transition`, `TaskGeneration`,
`ExecutionRevision`, `Target`, `Operation`, `Match`, `ChangeProposal`, `Attempts`,
`CostAICredits`, `Claim`, `ClaimedAt`. Claim fields mirror target-local ownership;
they do not grant it. Transition rules are
enforced in the Board Controller, never in prompts.

## Consequences

**Positive**
- Standings live where the operator already works; no extra datastore to run.
- The board is the source of truth for current target stage/standing. Approved
  definition commits govern behaviour; target references govern claims (ADR 0008).
- Native views group by operation-specific `Stage` or by lifecycle `Standing`;
  a target's tracking issue retains history across every transition (ADR 0007).

**Negative**
- The repository-is-not-an-item mismatch forces a **tracking issue per target** — more
  issues and a layer of indirection between a repo and its board entry.
- Board writes use the Projects v2 GraphQL API and need ownership/generation checks
  and reconciliation. Project fields are not an atomic cross-operation lock.
- Same-intent specification revisions retain this board and its history. Reconcile
  under the new approved definition rather than creating a new operation (ADR 0008).
