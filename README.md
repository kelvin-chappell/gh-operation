# gh-operation

This repository will **create repositories that define concrete operations**. Each
operation has exactly one independently runnable **Operation Repository**, containing
its manifest, specification, agents, skills, workflows and local controller source.
Those repositories run and manage their operations, not this creator repository.

**Status: operation-repository contract agreed; creator design is next.** There is no
runtime or generator implementation yet. The documentation records the confirmed
boundary and the validation gates required before an operation may execute.

## Start here

| Document | Purpose |
|---|---|
| [`CONTEXT.md`](CONTEXT.md) | Canonical domain glossary |
| [`DESIGN.md`](DESIGN.md) | Operation-repository contract; §20 defers the detailed creator design |
| [`docs/adr/`](docs/adr/README.md) | Decisions and superseded alternatives |
| [`fixtures/`](fixtures/README.md) | Synthetic unstaged/staged specifications and expected outcomes |

## Definition, creation and execution

Interrogation first turns an intent into a complete executable definition. The
operator approves it **before** the operation repository is created. Preserve that
approval and provenance, then require a separate activation after readiness and
reconnaissance. Creation does not automatically start target changes.

An operation repository runs pinned **Copilot CLI in GitHub Actions**, using local
agent/skill definitions and controller source. Its versioned manifest and overview
make the structure inspectable. Approved definitions live on a protected branch;
append-only operational records live on a separate branch in the same repository.
It owns its tracking issues and associated Projects v2 board.

Only approved operation configuration governs execution. Target files remain
untrusted data. Qualification, claims, credit accounting, publication and completion
are enforced by trusted code, not granted because a model asserted success.

## Stages, actions and completion

**Stage** records achieved progress; **standing** records whether work is ready,
underway, awaiting review, blocked or finished. Stage-based board columns can be
operation-specific without losing the lifecycle view.

For example, a Scala 2.12 target follows **2.12 -> 2.13 -> 3.3 -> 3.9**, with three
separate proposals. A target already at 3.3 needs only the final transition. Each
task generation has one proposal, and only one proposal may be open per target.
Hold an atomic target-local claim through review and default-branch verification,
then release between verified transitions. Expiry never authorises automatic takeover.

After discovery and qualification finish, completion requires every obligation to
reach the current final goal or be explicitly **waived**. A waived repository remains
visible at its last verified stage with the operator's reason. Failure, a blocker,
an intermediate merge or exhausted resources is not completion.

## Revisions, privacy and cost

Same-intent revisions remain in the same operation repository. Drain actions and
resolve proposals before handover. Preserve history, cumulative spending and
unchanged-task retry counts; re-verify progress. Material work changes create new
approved task generations, and changed obligations require waiver reaffirmation.
Reopening a concluded operation requires explicit reactivation, not merely a push.

Private targets require a private operation repository, equally restricted records
and verified Copilot/model data-handling guarantees. Reuse copies approved structure
and provenance, never live state. Private-to-public reuse needs an explicitly reviewed
sanitised export and fresh approval.

**Cost is measured only in AI credits.** There are no currency forecasts. The measured
credit threshold stops new dispatch with conservative reservations; in-flight
overshoot is disclosed. Runner time and concurrency are separate resource limits.
Missing authoritative credit metering blocks execution.

Keep minimised durable audit with explicit rationale. Sensitive raw payloads and
optional available traces have restricted storage and explicit retention; they are
not committed permanently to Git. Stop work on conclusion or abandonment; archive
after outstanding work is resolved. Emergency stop cancels active work without
automatically freeing claims.

## Next discussion

`DESIGN.md` §20 identifies the next topic: this repository's inputs, interaction
model, definition assembly/approval, reuse, validation, creation and provisioning
assistance. Its detailed architecture remains deliberately undecided. It will not
be a central live runner or a second source of truth for its generated operations.
