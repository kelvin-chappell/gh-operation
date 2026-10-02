# 0007 - Stages separate achieved progress from the lifecycle of work

- **Status:** Accepted
- **Date:** 2026-10-02
- **Deciders:** operator
- **Related:** `DESIGN.md` §4, §5.3, §7, §9, §18; `CONTEXT.md` (Stage, Transition, Completion, Waiver)
- **Supersedes:** ADR 0003's one-proposal-per-target packaging rule only

## Context and decision

A repository may need several independently reviewed upgrades before reaching an
operation's goal. Treating each upgrade as a new operation loses the shared goal,
board and envelope; treating milestone names as lifecycle standings hides whether
work is ready, underway or blocked.

Use optional ordered **stages** as verifiable repository milestones, separate from
**standing**. Each adjacent **transition** has a frozen task, acceptance and bounds.
Targets enter at their verified current stage, advance only after verification on
the default branch, and have at most one proposal per transition, with no stacked
proposals. The draft and operator-only merge gates remain unchanged.

Completion requires finished discovery and every qualified target at the final goal
or explicitly **waived** by the operator with a reason. Failure, a blocker or exhausted
resources is not success. Waivers retain the last achieved stage; conclusion records
whether the operation actually completed. Unstaged operations retain one task and
one proposal per target, subject to the same explicit completion rule.

## Consequences

Board columns can express Scala 2.12, 2.13, 3.3 and 3.9 without discarding lifecycle
information. Proposal identity, attempt budgets and history must be transition-scoped.
Reconnaissance must sum the remaining transitions and review load, not assume one
proposal per repository. Separate proposals cost more review and CI time than one
large upgrade, but provide intermediate verification and independent rollback.
