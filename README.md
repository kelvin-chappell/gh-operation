# gh-operation

This repository will **create repositories that define concrete operations**. Each
operation has exactly one independently runnable **Operation Repository**, containing
its manifest, specification, agents, skills, workflows and local controller source.
Those repositories run and manage their operations, not this creator repository.

**Status: operation-repository and creator architectures agreed.** There is no
runtime or generator implementation yet. The documentation records the confirmed
boundary and the validation gates required before an operation may execute.

## Start here

| Document | Purpose |
|---|---|
| [`CONTEXT.md`](CONTEXT.md) | Canonical domain glossary |
| [`DESIGN.md`](DESIGN.md) | Operation-repository contract; §20 defines the creator architecture and build sequence |
| [`docs/adr/`](docs/adr/README.md) | Decisions and superseded alternatives |
| [`fixtures/`](fixtures/README.md) | Synthetic preparation and operation contract cases |

## What this repository will do

Provide a **Creator agent**, preparation skills, a tested runtime base, curated
starting points and deterministic validation/materialisation tools:

```text
intent -> interrogate/inspect -> assemble -> validate -> human approval
       -> create and verify repository resources -> inactive handover
```

Copilot generates operation-specific specifications, agent/skill content and bounded
adapters. It does not rewrite safety machinery per operation. Drafts, review artifacts,
credit accounting and creation journals live in resumable private workspaces outside
this public checkout. Explicit predecessor revisions are inspected as quarantined
data and adapted onto a supported base, not automatically loaded into the Creator.

Copilot may invoke materialisation after conversational approval, but a trusted human
receipt must bind the exact bundle and destination. Deterministic tools verify that
receipt, create the repository/branches/protections/board and recover only verified
owned partial resources. They never silently overwrite, rename or delete resources.

Preparation has a separate AI-credit threshold. Label estimates distinctly, combine
them with measured usage/reservations for continuation decisions, and replace them
when actual usage arrives without double counting. Estimated preparation accounting
does not relax the operation's authoritative-meter requirement.

Handover transfers the approved definition, approval/provenance, usage and setup
report into operation-owned records. External access setup remains explicit operator
work. The operation is **inactive**, never automatically started by creation.
V1 is initial preparation/creation only, not a fleet manager or central updater.

## What the generated repositories do

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

## Build sequence

`DESIGN.md` §20.9 starts with proving authentic approval, isolation and accounting
boundaries; establishing the tested operation runtime base; building pure preparation
tools; adding the Creator experience; and testing materialisation/recovery and handover.
The agent, skills, runtime and helpers are planned components, not implemented commands.
