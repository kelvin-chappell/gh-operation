# 0008 - One independently runnable repository per operation

- **Status:** Accepted
- **Date:** 2026-10-02
- **Deciders:** operator
- **Related:** `DESIGN.md`; `CONTEXT.md` (Operation Repository, Execution Revision, Task Generation)
- **Supersedes:** ADR 0004's public-only restriction, ADR 0005's custom model loop,
  and earlier new-operation-on-revision assumptions

## Context and decision

A shared control repository hides the boundary between reusable structure and one
operation's live execution. Instead, create exactly one operation repository for
each operation, after approving its initial complete executable definition. It owns
its manifest, agents, skills, local controller source, workflows, associated board
and operational record, and runs pinned Copilot CLI in GitHub Actions without a live
dependency on this creator repository.

Use a protected definition branch with approved execution commits and a separate
append-only records branch. Activation is explicit. Same-intent revisions remain in
the repository, draining actions and resolving proposals before handover. Preserve
spending, history and unchanged-task retries; material work changes create approved
task generations. Reopening requires explicit reactivation. Unfinished removed
targets require waivers; changed obligations require waiver reaffirmation.

Claims live in target repositories as an atomic reference protocol, held through one
transition's review/verification and released between transitions. Expired ownership
requires operator-confirmed recovery. Private targets require private, equally
restricted records and verified model data-handling guarantees. Only operation-approved
configuration controls Copilot; target files are untrusted data, and control code
enforces qualification, claims, publication and completion.

Measure cost **only in AI credits**, with a metered dispatch threshold and disclosed
in-flight overshoot. Missing authoritative usage blocks execution; no unverified hard
ceiling is promised. Keep minimised structured audit permanently, with sensitive raw
payloads in restricted artifacts under explicit retention, not indefinite Git history.

Reuse selected approved structure and provenance, not live state. Private-to-public
reuse requires a reviewed sanitised export and fresh approval. Conclusion stops new
work; archive only after resolving outstanding actors, claims and proposals.

## Trade-offs and boundaries

Local snapshots duplicate controller source and require reviewed repairs rather than
central hotfixes, but make each operation independently inspectable and executable.
Per-operation records and privacy policies cost more setup than one public control
repository, but support private targets without public leakage. Target-local claims
avoid a central live coordinator, but need proven race-safe acquisition and release.

The operator confirmed the five-round interview. Copilot isolation/credit metering,
claim atomicity and confidentiality are implementation-validation gates, not capabilities
declared to exist. This repository's detailed creator architecture remains the next
discussion; this ADR does not implement a generator or approve a central live runner.
