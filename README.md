# gh-operation

**One creation session, one execution session.** An operator uses a single Copilot
session here to generate a complete operation repository. They then open its dev
container and use one Copilot session there to start and coordinate the whole operation
through completion. Each generated repository contains one operation's context,
definition, agents, skills, environment and usable end-to-end procedures.

**Status: initial agent-context base implemented.** The Creator agent, preparation
skill and `templates/operation/` provide Markdown-first definitions, four operational
roles and a shared operation-cycle skill. Work starts only within Copilot sessions
in dev containers, not GitHub Actions workflows.
Authentic approval capture, materialisation and target execution are not implemented.
The earlier TypeScript application scaffolding and Node tooling have been removed.
No operation repository has been created by this implementation.

## Start here

| Document | Purpose |
|---|---|
| [`CONTEXT.md`](CONTEXT.md) | Canonical domain glossary |
| [`DESIGN.md`](DESIGN.md) | Operation-repository contract; §20 defines the creator architecture and build sequence |
| [`docs/adr/`](docs/adr/README.md) | Decisions and superseded alternatives |
| [`fixtures/`](fixtures/README.md) | Synthetic preparation and operation contract cases |
| [Creator](.github/agents/creator.agent.md) | Preparation entry point |
| [Preparation skill](.github/skills/prepare-operation/SKILL.md) | Interrogation, assembly, review and blocked/verified handover |
| [Operation base](templates/operation/MANIFEST.md) | Files assembled into an independently inspectable operation definition |

Select `creator` with Copilot's `/agent` command and supply an intent. Preparation
asks for an approved private workspace outside this checkout. The template is a
draft with explicit `{{...}}` fields, not an approved runnable operation. Review those
fields and the complete definition within the session before approval.

**Quick assembly is the default:** use supplied facts and existing templates, resolve
essential decisions and deliver a complete inactive repository in the same session.
The Creator guides the operator with pointed questions and focused grilling,
including recommendations, challenges to ambiguous answers and follow-up rounds
until the essential decisions are settled.
Drafts may contain pending decisions while assembling; they are not the finished
product. Creation does not perform target discovery or changes. Multiple agents and
checkpoint invocations can be coordinated within the subsequent execution session.

Dev container creation supplies Copilot and `gh`, so neither is declared in
`.tool-versions`. The retained pin is for the dev container generator. There is no
Node dependency, application build or process-triggering workflow. Record actual
tool versions in session evidence; structural checks do not establish operational
enforcement.

Each agent declares its model in YAML frontmatter. Creator and Actor use `gpt-6-sol`
for definition synthesis and repository changes; Supervisor uses `gpt-6.1-sol` for
lifecycle reasoning;
Elicitor uses `gpt-5.4` for structured clarification; Explorer and the
tool-free credit-meter probe use `gpt-5.4-mini` for bounded evidence collection
and minimal responses. These are starting defaults, not proof of model availability
or data-handling approval. Review operation-specific changes before use.

**Before every pull request**, including drafts, obtain independent adversarial
review of the exact final diff. All non-reviewer agents use OpenAI models; the single
`reviewer` uses Anthropic's `claude-opus-4.8`, read-only in a separate agent context.
Unknown authorship or a provider-policy violation blocks publication. Resolve findings and repeat
review after edits; missing or invalid review blocks publication. This session-level
policy applies here and in generated operations, separately from human approval.

## What this repository will do

Provide a **Creator agent**, preparation skills and maintained agent-context starting
points, using existing session/platform tools and only necessary small helpers:

```text
intent -> interrogate/inspect -> assemble -> validate -> human approval
       -> create and verify repository resources -> inactive handover
```

Copilot assembles operation-specific definitions and agent/skill content. Small
deterministic helpers require a demonstrated need; a copied application is not the
base. Drafts, review artifacts,
credit accounting and creation journals live in resumable private workspaces outside
this public checkout. Explicit predecessor revisions are inspected as quarantined
data and adapted onto a supported base, not automatically loaded into the Creator.

The operator reviews and approves exact contents and destination within the creation
session. Available tools materialise approved resources and recover only verified
owned partial resources. A hypothetical custom materialiser is not a dependency.
Never silently overwrite, rename or delete resources.

Preparation has a separate AI-credit threshold. Label estimates distinctly, combine
them with measured usage/reservations for continuation decisions, and replace them
when actual usage arrives without double counting. Estimated preparation accounting
does not relax the operation's authoritative-meter requirement.

Handover transfers the approved definition, approval/provenance, usage and setup
report into operation-owned records. External access setup remains explicit operator
work. The operation is **inactive**, never automatically started by creation.
V1 is initial preparation/creation only, not a fleet manager or central updater.

Complete means the next session has what it needs without returning here or building
missing machinery. Human approvals, merge decisions and external prerequisites remain
explicit; blockers or exhausted resources produce an incomplete outcome, not a false
promise of single-session success.

## What the generated repositories do

Interrogation first turns an intent into a complete executable definition. The
operator approves it **before** the operation repository is created. Preserve that
approval and provenance, then require a separate activation after readiness and
reconnaissance. Creation does not automatically start target changes.

An operation repository runs through **Copilot sessions in a dev container**, using
local agent/skill definitions once execution gates are proven. Its manifest and overview
make the structure inspectable. Approved definitions live on a protected branch;
append-only operational records live on a separate branch in the same repository.
It owns its tracking issues and associated Projects v2 board.

Only approved operation configuration governs execution. Target files remain
untrusted data. Qualification, claims, credit accounting, publication and completion
require independently verified mechanisms, not a model's assertion of success.
The current base describes these requirements but does not implement enforcement.

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

ADRs 0010/0011 correct the earlier runtime-heavy and workflow-triggered sequence in
`DESIGN.md`. Start with
agent context and local preparation; next prove approval, isolation and accounting
boundaries, then implement the smallest materialisation/recovery and execution
mechanisms needed. Session startup and structural review are not operational readiness.
