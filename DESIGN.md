# Operation Repositories - Design

`gh-operation` will create repositories that define concrete operations. **One
operation has exactly one Operation Repository, and each Operation Repository
defines exactly one operation.** Those repositories, not this creator repository,
run and manage their operations.

**Product purpose:** one operator session here generates a complete operation
repository; one coordinating session there can start and complete its operation.
`CONTEXT.md` defines this current journey. A pending draft is an intermediate result,
not the normal deliverable. Supplied procedures must use available tools and require
neither a return to the creator nor future application/runtime development.
Multiple agent/checkpoint invocations remain within that execution session, subject
to human decisions, resource limits and truthful incomplete outcomes when blocked.

**Status: initial agent-context base implemented.** ADR 0010 records the operator's
correction: operation repositories contain Markdown context, agents, skills and a
dev container configuration, not a copied deterministic application. ADR 0011
requires all work to start within Copilot sessions in dev containers, with
container-provided Copilot and `gh`, not GitHub Actions triggers.
The Creator/preparation skill
and `templates/operation/` implement the preparation-facing agent context.
Approval capture, materialisation and target execution remain unimplemented.
Copilot isolation/metering, target-local claim atomicity and private-data guarantees
remain validation gates.

**Historical design:** controller/runtime-base descriptions in §§3.5, 14, 16 and 20
are retained as earlier design detail, not the current implementation prescription.
Workflow/controller examples and explicit Copilot/`gh` pins elsewhere are historical
too. ADRs 0010/0011 take precedence for packaging, execution and build order; evidence and authority
contracts elsewhere remain requirements, not proven capabilities.

`CONTEXT.md` defines the domain language. ADRs record the decisions and the earlier
choices they supersede. Platform artifacts retain their platform names: a target's
tracking issue is a Projects v2 item; a change proposal is a draft pull request.

---

## 1. Goals, non-goals and agreed decisions

Create an inspectable, independently runnable repository **after** its initial
definition has been approved. Its manifest, agents, skills and dev container
configuration describe the operation's structure; a selected approved revision
can become a starting point for another operation.

An operation discovers and qualifies targets, advances them towards its goal, and
records progress, proposals, decisions and usage. Optional ordered stages allow
different tasks at different repository milestones. Unstaged operations retain one
task and at most one proposal per task generation and target.

| Area | Decision |
|---|---|
| Repository identity | One operation per repository; revisions retain the same intent |
| Creation | After initial definition approval; preserve approval and provenance |
| Activation | Explicit operator decision after readiness checks; creation does not start work |
| Agent runtime | Copilot sessions in dev containers; container creation supplies Copilot and `gh` |
| Local runtime | Operation-owned agent/skill definitions and container configuration; no live dependency on the creator |
| Approved unit | Full executable definition, not just specification YAML |
| Definition storage | Protected definition branch; each execution uses an approved definition commit |
| Records | Separate append-only records branch, plus the operation's issues and associated board |
| Revisions | Drain actions and resolve proposals before handover; no overlapping active revisions |
| Target scope | Own organisation(s), same-repository proposal branches, no forks in v1 |
| Qualification | Deterministic code; model annotations never promote candidates |
| Stages | Optional ordered milestones; one adjacent transition at a time |
| Proposal identity | Operation, immutable repository id, logical task/transition and task generation |
| Claims | Atomic target-local Git reference protocol; one transition including review and verification |
| Expired claims | Operator-confirmed recovery; never automatic takeover in v1 |
| Authority | Named operator and explicit delegates; agents cannot authorise their own work |
| Visibility | Chosen per operation; private targets require private and equally restricted records |
| Model policy | Operation-approved Copilot models and verified data-handling guarantees |
| Cost | AI credits only; no currency estimates or conversions |
| Credit limit | Measured dispatch threshold, conservative reservations and disclosed in-flight overshoot |
| Completion | Finished discovery/qualification, all obligations achieved or explicitly waived |
| Retirement | Stop dispatch, retain records, archive after outstanding work is resolved |
| Reuse | Approved structure and provenance, never live state; sanitise private-to-public exports |

Non-goals are non-GitHub forges, multi-tenant SaaS in v1, automatic merges, unrelated
intent changes inside one operation, stacked proposals, and a central live runner in
this creator repository. Human approval and merge gates are not relaxed by using Copilot.

## 2. Constraints and trust boundaries

1. Projects v2 cannot hold a repository directly. One tracking issue in the operation
   repository represents each target on its associated board.
2. Search and core GitHub quotas are separate. More explorers cannot manufacture
   additional search quota. Use live limits, capped retries and explicit errors.
3. Target code and repository-authored instructions are untrusted. Private code also
   has a confidentiality boundary. A public creator does not make operation data public.
4. GitHub Actions concurrency groups are repository-local. They cannot replace an
   atomic target-local claim shared by independent operations.
5. Copilot may load ambient instructions, profiles, skills, hooks and external
   connections. The approved definition must control the effective configuration;
   merely copying agent files into a repository does not prove isolation.
6. A local runtime snapshot contains our controller and orchestration source, not
   Copilot's hosted model implementation. Pin tools/dependencies and record the model
   identity actually used; do not promise bit-for-bit reproducibility of hosted models.
7. Copilot plan usage is not direct model API currency billing. Authoritative,
   attributable AI-credit reporting must be verified for the pinned integration.
   Missing usage is an error, not zero spend.

## 3. Components and ownership

```text
intent -> interrogation -> approved definition -> create Operation Repository
                                                        |
                                                 readiness + activate
                                                        |
       Operation Repository: local agents, skills, workflows and controller
                 |                    |                         |
          exploration/qualification   supervision         claimed action
                 |                    |                         |
        tracking issues + board       credit ledger       target proposal
                 |                                              |
                 +----------- structured records <--- verified merge
```

The creator has no ongoing role in dispatching, claiming or maintaining the
operation's standings. Its detailed creation interface is deferred to §20.

### 3.1 Elicitor

Interrogation turns an intent into a complete definition, including tasks, acceptance,
stages, agents, skills, workflows, policies and resource limits. Initial interrogation
happens **before** the operation repository exists. Afterwards its local Elicitor
supports same-intent revision proposals. See §10.

### 3.2 Supervisor

A Copilot agent proposes an explainable plan from the active execution revision,
target stages/standings, live quota and measured AI-credit ledger. Trusted controller
code validates the plan and enforces dispatch, authority and resource constraints.
The supervisor does not change repositories or grant waivers.

### 3.3 Explorer

Code searches, filters and probes repositories read-only. Copilot may annotate
ambiguous evidence, but qualification is deterministic. The controller creates
tracking issues and sets board fields. Reconnaissance performs the same discovery
without creating targets, acquiring claims or writing to targets.

### 3.4 Actor

Copilot produces changes for one applicable task generation within the approved
bounds. Isolated acceptance checks validate the changes. Trusted publishing code,
not the model, checks authority, revision, claim ownership, destination evidence and
acceptance before pushing a branch and opening a draft proposal.

### 3.5 Controller

Operation-owned TypeScript/Node controller code with `@octokit/*` handles GitHub APIs,
claim transitions, deterministic validation, stage advancement, usage and audit.
Exact supported tool versions belong in `.tool-versions`, managed with mise.
This is control code around Copilot CLI, **not a custom model-API agent loop**.

Source-of-truth boundaries are explicit: approved commits define behaviour; the board
holds current target stage/standing; the target reference holds exclusive ownership;
the records branch retains authenticated decisions and audit. Board claim fields
mirror ownership, but cannot grant it. Readiness bindings and the active-revision
record cannot introduce unapproved behaviour.

## 4. Lifecycle, board and completion

The operation moves through:

```text
draft -> specified -> scoped -> active -> concluded or abandoned
```

`draft` and initial approval precede repository creation. Creation materialises the
approved definition with execution disabled. `scoped` requires reconnaissance for
that definition; activation additionally requires readiness (§11). Pausing dispatch
is an execution control, not successful completion.

| Board field | Type | Meaning |
|---|---|---|
| `Standing` | single select | target lifecycle |
| `Stage` | single select | verified achieved milestone; unset for unstaged work |
| `Transition` | text | applicable logical transition; unset at the final goal |
| `TaskGeneration` | text | approved work identity, including unstaged tasks |
| `ExecutionRevision` | text | approved definition governing current work |
| `Target` | text | owner/name; history also records immutable repository id |
| `Operation` | text | stable operation identity |
| `Match` | text | minimised qualification evidence |
| `ChangeProposal` | text | current proposal, or last final proposal |
| `Attempts` | number | substantive attempts for this task generation |
| `CostAICredits` | number | cumulative measured AI-credit usage |
| `Claim` | text | mirrored opaque ownership token |
| `ClaimedAt` | text | claim time; target reference remains authoritative |

### Stage and standing are separate

Staged views group by operation-defined `Stage`, such as Scala 2.12, 2.13, 3.3 and 3.9.
Cards display `Standing`; a companion view groups by standing, including `waived`.
A proposal to reach 2.13 does not move a 2.12 target into the 2.13 column. Advance only
after default-branch verification. Unknown stages remain visibly unassigned with
`needs operator`; manual board moves are not evidence.

```text
discovered -> qualify -> ready -> claim -> in action -> proposal open
proposal open -> verified intermediate merge -> ready for next transition
proposal open -> verified final goal -> accepted
verified external final progress -> no change needed, or accepted if prior work merged
conflict, detection error or verification discrepancy -> needs operator
substantive retry exhaustion -> failed
operator waive -> waived
```

`accepted` and `no change needed` require the **current revision's final acceptance**.
An intermediate accepted proposal is history, not final target standing. A final
target with no operation proposal merged is `no change needed`; otherwise `accepted`.

Only the operator or an authorised delegate may reset failed work, resume a blocker,
waive an obligation or recover an expired claim. A reset explicitly grants a new
attempt budget without deleting history or earlier stages. `dismiss` is limited to
unqualified candidates. Previously qualified unfinished targets removed by narrower
scope remain recorded and require explicit waivers.

A waiver includes authenticated identity, time, reason, applicable work/goal and last
verified stage. Resolve open proposals and active actors before waiving and safely
releasing any ownership. Unchanged obligations retain a waiver; changed obligations
require explicit reaffirmation, with both decisions preserved.

Completion requires exhausted discovery **and qualification** and every required
target at the current final goal or explicitly waived. `failed`, `needs operator`,
open proposals, intermediate stages and pending candidates are not completion.
Re-check changed default-branch heads before concluding. Record `complete` or
`incomplete`, with achieved, waived and unresolved counts; a stop cannot override this
rule. A same-intent approved revision may reopen concluded work through explicit
reactivation, readiness and fresh reconnaissance. Unarchive first if necessary.

## 5. Definition, manifest and specifications

The protected definition branch contains one root manifest, the operation's
specification and the complete local executable structure (§14). There is no
`operations/<id>/` collection inside a running operation repository.

The manifest has a schema version and binds operation identity/intent, operator and
delegates, role-to-agent mappings, skills, workflows, specification paths, runtime
versions, policies and provenance/approval references. Validate its entire executable
closure: unexpected auto-loaded instructions or unpinned external references are
not harmless extras.

An illustrative manifest shape, **not a ready-to-run template**:

```yaml
manifest_version: 1
operation:
  id: 2026-10-scala-staged-upgrade
  intent: Modernise our Scala codebases through approved milestones
  operator: fixture-operator
  delegates: []
specification:
  explorer: specification/explorer.yaml
  actor: specification/actor.yaml
agents:
  elicitor: .github/agents/elicitor.agent.md
  supervisor: .github/agents/supervisor.agent.md
  explorer: .github/agents/explorer.agent.md
  actor: .github/agents/actor.agent.md
skills:
  - .github/skills/scala-migration/SKILL.md
runtime:
  copilot: cli
  tools: .tool-versions
  source: src/control-plane
  workflows: .github/workflows
policies:
  definition_branch: main
  records_branch: records
  activation: explicit
  claim_protocol: target-local-ref-v1
  target_visibility: public
  cost_unit: ai_credits
  credit_enforcement: dispatch_threshold
  model_data_handling: policies/model-data-handling.yaml
  raw_payload_retention: policies/retention.yaml
```

Approval identifies the full definition's digest and, after materialisation, its
definition commit. Preserve initial approval outside the executable payload to avoid
a self-referential approval hash. Provisioned resource ids and access bindings must
be validated against the approved policy; they cannot silently rewrite the definition.

### 5.1 Exploration specification

`specification/explorer.yaml` declares scope, metadata/content criteria, partitioning
and deterministic qualification. `scope.visibility` may be `public`, `private` or
`mixed`; private/mixed scope requires the privacy gates in §12. A public-only fixture
may legitimately declare `private: false` without making it a system-wide restriction.

```yaml
operation: 2026-10-scala-2-upgrade
scope:
  owners: [acme]
  exclude_repos: [acme/dotfiles]
  visibility: public
criteria:
  metadata: { private: false, languages: [Scala], archived: false, fork: false }
  content:
    - id: scala_version
      match: any
      file: build.sbt
      extract: regex
      pattern: 'scalaVersion\s*:=\s*"([^"]+)"'
      in: ["2.13.12"]
partition: { strategy: by_owner, max_partitions: 1 }
qualification: { validator: deterministic }
```

Admission and progress are separate. An admitted target must not disappear merely
because its upgrade no longer matches the initial content criteria.

### 5.2 Unstaged action specification

`specification/actor.yaml` contains `task`, `acceptance`, proposal conventions, bounds,
limits and retry/escalation policy. Unchanged unstaged tasks retain identity across
execution revisions; material changes create a new task generation.

```yaml
operation: 2026-10-scala-2-upgrade
task:
  title: Bump Scala to 2.13.14
  kind: code_change
  instructions: |
    Change only scalaVersion in build.sbt to 2.13.14.
    Do not touch libraryDependencies or plugins or reformat unrelated files.
acceptance: { build: "sbt compile", test: "sbt test" }
pr:
  branch: "automation/{{operation}}-{{repo_slug}}-{{task_generation}}"
  title: "{{task.title}}"
  draft: true
adaptation:
  intent: fixed
  implementation: within_bounds
  may_add_files: true
  forbidden_changes: [dependencies, lockfiles]
limits: { max_attempts: 2, wall_clock_minutes: 30, max_files_changed: 3 }
retry:
  transient: [secondary_rate_limit, server_error, runner_lost, network_timeout]
  max_transient_retries: 3
  backoff_cap_minutes: 5
escalation: { on_failure: escalate_model, on_existing_pr: needs_human }
```

### 5.3 Staged action specification

Staged specifications use `stage_probe`, ordered `stages` and adjacent `transitions`
instead of top-level task/acceptance. Each transition has its own task and acceptance,
with explicit effective bounds. Shared defaults are resolved and approved before use.

```yaml
stage_probe:
  match: all
  file: build.sbt
  extract: regex
  pattern: 'scalaVersion\s*:=\s*"([^"]+)"'
stages:
  - { id: scala-2-12, name: Scala 2.12, when: { matches: '^2\.12\.[0-9]+$' } }
  - { id: scala-2-13, name: Scala 2.13, when: { matches: '^2\.13\.[0-9]+$' } }
  - { id: scala-3-3, name: Scala 3.3, when: { matches: '^3\.3\.[0-9]+$' } }
  - { id: scala-3-9, name: Scala 3.9, when: { matches: '^3\.9\.[0-9]+$' } }
```

The standalone staged fixture defines all three transition tasks and acceptance:
`fixtures/operations/2026-10-scala-staged-upgrade/actor.yaml`. Version values are
illustrative; a real definition pins supported destinations before approval.

Validate unique stage/transition ids, at least two stages, and exactly one forward
transition between each adjacent pair. Reject missing, backwards, branching or cyclic
paths. The last stage is the goal. Exactly one stage must match all probe values;
missing, mixed, malformed, overlapping or unsupported values yield explicit evidence
and `needs operator`, not a model-selected stage. Dynamic or cross-build declarations
need a probe covering every relevant value, not one convenient `scalaVersion` match.

Proposal-branch checks require destination evidence and transition acceptance;
they do not advance the board. After merge re-check the default branch. Final-stage
entry and external progress also require the final transition's acceptance, not a
version string alone. Verification errors retain the last verified stage and go to
`needs operator`. Within one revision, regressions do not silently reopen past work.

### 5.4 Task generations and history

A material task, destination predicate, acceptance or bounds change creates a new
approved generation. A runtime-only repair retains an unchanged generation and its
attempt budget. Reusing an accepted result requires equivalence of the relevant work
and fresh evidence under the active definition, not merely matching display names.

Proposal identity is `(operation, repository_id, logical_task, task_generation)`.
Staged branch templates include both transition and generation; unstaged templates
include generation. Retries update the same unmerged proposal. Generations do not
erase earlier proposals, attempts or spending. Every event records its execution
revision, generation and immutable repository identity.

## 6. Exploration and reconnaissance freshness

Search deterministically, filter metadata, then fetch approved content probes
read-only. Model annotations are evidence only. The controller creates one tracking
issue per admitted immutable repository id, deduplicated across partitions and reruns.

Qualification detects a fresh starting stage. Unknown stages remain qualified
obligations with `needs operator`; already-final targets require final acceptance.
Cache probes within one run only; re-probe during qualification and actor triage.
Cap pagination and record errors explicitly. Private metadata and content obey the
same confidentiality gates as coding work; read-only is not a privacy exemption.

Explorers cannot write to target repositories or grant claims. Their API permissions
are separated from the controller that creates operation-owned issues/board entries.

## 7. Claimed actions and publication

1. Check active approved revision, privacy, credit reservation and readiness. Acquire
   the target-local operation claim atomically, then grant one actor exclusive use.
2. Check existing work. Resume only this generation's recorded unmerged proposal.
   Conflicting/foreign work goes to `needs operator`; never adopt or duplicate it.
3. Clone and re-probe the default branch. Changed stages require verified re-planning,
   not applying a stale task; regressions/unrecognised states need operator attention.
4. Select the approved applicable task. Verified intermediate external progress selects
   the next transition; verified final acceptance yields the appropriate final standing.
5. Invoke the pinned Copilot CLI with the approved profile/skills/configuration and
   isolated workspace. No ambient user/organisation/target configuration may enlarge
   its capabilities. Source changes must remain within effective frozen bounds.
6. Validate the patch, destination predicate and acceptance in isolated jobs. Dependency
   and lockfile edits are forbidden by default, allowed only when explicitly approved.
   Out-of-bounds work becomes `needs operator`, not a silently wider diff.
7. Trusted publication re-checks active revision, actor/claim identity and stop state.
   If valid, push the generation's branch and open/update one draft proposal. Keep the
   source stage until default-branch verification after the operator merges.
8. Archive outcomes, advance stage/standing and reset attempts only for new work or
   an explicit reset. Release ownership after a verified transition; keep it through
   unresolved proposal review, failure or a blocker until safe resolution.

Infrastructure failures retry at job level at most three times with capped
exponential backoff and jitter; they do not consume substantive attempts. Real
failures advance the current generation towards `failed`; only explicit reset gives
it a fresh retry budget. Closing without merge does not advance progress or
automatically open a replacement.

Duplicate/late webhook events update their matching historical generation only, never
resetting later work. A merged proposal whose verification fails is recorded as merged
but unresolved; it is not edited as though still resumable unmerged work.

### Target-local atomic claim protocol

Use one reserved claim reference per target, separate from its default branch.
Ownership metadata is minimal: opaque operation/holder tokens, generation, protocol
version and lease information. It must not disclose private names, URLs or content.

The protocol must prove safe acquisition, renewal and release under concurrent and
delayed requests. GitHub documents unique reference creation and non-force,
fast-forward updates, **not an expected-SHA parameter on reference deletion**.
Simply deleting a read-then-checked reference can let a delayed release delete a newer
holder. Validate a single-parent, non-force state-transition protocol with a persistent
released marker, or an equivalently safe protocol; do not assume a naïve delete/create
lock is sufficient. Never force-overwrite or merge competing claim states.

Reference creation races, delayed release, duplicate messages, missing permissions,
tampering, expired ownership and abandoned review need adversarial integration tests.
Until they pass, claiming is not ready. Failure to establish ownership blocks work;
checking a board field or Actions concurrency group is not a substitute.

An expired lease pauses publication and competing acquisition. Recovery requires an
authenticated operator decision after stopping the old actor and resolving live
proposals/work. Emergency stop cancels work without automatically freeing its claim.

## 8. Supervision, model selection and AI-credit allocation

Read the active revision, target stages/standings, ready generations, live GitHub
limits, runner ceiling and authoritative measured credit ledger. Propose a plan;
controller code selects only work that fits remaining credits minus reservations
and reserve, API headroom, action-count and wall-clock constraints.

```text
available_credits = credit_threshold - measured_credits - reserved_credits - reserve
actions = min(ready_tasks, runner_ceiling,
              available_credits / estimated_credits_per_action,
              core_headroom / estimated_calls_per_action)
if usage is unavailable or available_credits <= 0: stop new dispatch
```

Use per-transition/generation estimates for staged work, not one cheap-step average
for a major migration. Reserve expected credits before dispatch; reconcile each
reservation with trusted usage afterwards. Do not settle a lost/unknown usage report
as zero. Suspend further dispatch until its spending is reconciled.
Validate finite non-negative measured/reserved credits and positive estimates for
model actions. Invalid accounting inputs produce an error and stop dispatch.

The threshold is **not a promised hard spending ceiling**: in-flight work may
overshoot. Record the excess explicitly. Independently enforce wall-clock, action-count
and concurrency bounds. All billed Copilot calls, including triage, retries and
supervision, belong in the credit ledger. Runner usage is operational telemetry, not
currency cost.

Copilot model configuration is part of the approved definition. Role profiles may
use different approved models, but there is no automatic provider fallback or
unreviewed model change. Record the CLI/model identity actually used. Missing required
privacy or usage capabilities blocks the call. The supervisor's rationale and emitted
plan are durable audit; unavailable internal reasoning is never invented.

## 9. Reconnaissance and feasibility report

Read-only reconnaissance measures the current definition's target set, starting
stages, remaining transitions, proposal/review load and projected **AI credits**.
Re-run after relevant definition changes and before activation/reactivation.
No target writes, claims, branches or proposals are permitted.

Lead with target count, then remaining work. One Scala 2.12 target needs three
proposals in the staged example; a 3.3 target needs one. Report unknown stages and
unpriced work explicitly, not as zero. Include inspection/verification and all model
calls in estimates; runner minutes and wall-clock are separate resource measures.

Forecasts are not reservations or admission records. Activation re-discovers and
qualifies against current content; report actual versus forecast drift. A first
operation may use a declared credit heuristic; later estimates use observed credits
for comparable work. An opt-in pilot spends credits and requires normal approval,
claims and publication gates.

The report records definition/execution identity and specification hash. Approval
reviews target count **and** total expected proposals. Currency projections are absent.
Fixture reports use synthetic credit values, not claims about real Copilot pricing.

## 10. Interrogation, approval and revisions

Before repository creation, interrogation resolves intent, scope, criteria, stages,
task generations, acceptance, bounds, agents/skills, workflows, resources, privacy,
authority and the complete executable manifest. Produce an inspectable definition
bundle and obtain initial operator approval; preserve its approval/provenance record
when materialising the operation repository. Do not place initial interrogation in
a repository that, by definition, does not yet exist.

Later revision proposals belong to the operation repository. Changes to executable
behaviour require an approved definition revision, even when specification YAML is
unchanged. Protect the definition branch; an agent can propose but never approve its
own revision. Logs changing on the records branch do not change the approved payload.

Handover pauses dispatch, drains active actions and resolves every open proposal
through verified merge or operator closure. Reconcile under the newly approved
definition before explicit activation:

- Re-probe progress and acceptance. A new final stage can reopen formerly achieved
  work; an unmappable stage needs operator attention rather than a guessed mapping.
- Preserve history, cumulative credits and unchanged-generation retries. New generations,
  resets and envelope increases must be visible in approval, not silent budget resets.
- Qualify newly added targets. Retain removed targets and waive unfinished obligations.
- Keep already-achieved removed targets as history, not falsely counted against a
  different new final goal. Completion counts refer to current required obligations.
- Carry unchanged waivers; reaffirm those affected by changed obligations.
- Repeat readiness and reconnaissance. No simultaneous active revisions.

Same-intent revisions retain repository, operation identity and board. Unrelated
intent requires a new operation. A concluded/archived operation cannot restart from
a push or webhook alone; explicit reactivation and unarchiving are required.

## 11. Copilot sessions and readiness

Preparation and operational work start only within Copilot sessions in dev containers.
Use approved definition commits, never the creator's moving branch or unapproved
target instructions. Container setup supplies Copilot and `gh`; record actual
versions in session evidence rather than pinning them in `.tool-versions`.

| Session activity | Responsibility |
|---|---|
| Activation | authenticate operator/delegate, validate readiness and start the approved revision |
| Revision | support interrogation and reviewed same-intent definition proposals |
| Reconnaissance | read-only scope assessment |
| Exploration | deterministic discovery, qualification and controlled annotations |
| Action | one claimed generation, isolated coding/acceptance and trusted publication |
| Reconciliation | inspect proposals, verify merges, settle credits and maintain valid ownership |
| Operator controls | authorised resume/reset/waive/recovery/stop/reactivation |

No workflow, schedule, push or proposal event initiates these activities. A subsequent
session reads durable records and current GitHub evidence to resume or reconcile.
Opening a session alone does not activate the operation.

Before any target execution, require an approved payload, matching active-revision
record, supported container-provided tools, manifest closure, named authority, correct repository/board
visibility, authorised GitHub/Copilot access, available approved models, current
reconnaissance, verified claim protocol, isolated effective Copilot configuration and
authoritative attributable credit metering. Missing bindings or guarantees produce
explicit diagnostics and block execution, never success-shaped defaults.

Provisioning changes may supply resource ids and access only within approved
capabilities. Credentials are external bindings, never manifest contents or reusable
definition data. The creator creates repository resources and reports required external
setup; access grants and installations remain operator-managed (§20).

Target claims must prevent conflicting work across sessions and operations.
Authenticate operator decisions and verify event provenance when inspecting GitHub evidence.
Ignore stale events for current-state mutation; audit their historical association.
Normal pause/drain permits resolution of existing work, not new task dispatch.
Emergency stop also halts active session work and blocks publication; recover claims
separately. Stopped operations may reconcile outstanding records without authorising
new target work. Archive only after outstanding actors, claims and proposals are resolved.

## 12. Security, privacy and authority

Private targets require a private operation repository and equally restricted board,
records, artifacts and logs, accessible only to people authorised for all included
private targets. A public operation must reject private targets before public evidence
or target-derived content is recorded. Source visibility is not a substitute for
checking actual resource permissions.
These protections apply before creation too: private drafts and approval artifacts
must not be stored in this public creator repository. The private preparation
workspace and handover contract are defined in §20.

Private content may reach only explicitly approved Copilot models/providers with
verified training-use, retention and access guarantees under the operation's policy.
Do not infer these guarantees from a Copilot subscription or assume a particular model
meets them. Policy/capability changes can make a previously ready operation unready.

Separate Copilot coding, untrusted acceptance and privileged controller/publication
capabilities. No org secrets, broad target-write authority or record-write credentials
belong in model context or target build environments. Validate paths, diffs and frozen
bounds independently. Only operation-approved instructions, agents, skills, hooks and
connections govern execution; target files are data. Prove the pinned CLI's isolation
and permissions, including ambient configuration and symlink/path escape cases.

The operator and explicit delegates authorise revisions, activation, waivers, recovery,
resets, emergency stops and reopening. Repo write access is insufficient. Keep the
existing human merge gate. Credit reports, qualification, completion and authority
checks are not trusted just because a model asserted them.

Reuse does not copy credentials, resource bindings, claims, targets, attempts, waivers
or execution history into a new live operation. Private-to-public reuse requires a
reviewed sanitised export, fresh approval and provenance that does not expose private
URLs/names. Preserve sensitive source provenance only in authorised private records.
Public-target branch names, labels and proposal metadata must use opaque or explicitly
approved public identities, never leak private operation names, URLs or instructions.

## 13. Audit and AI-credit usage

Accumulate measured credit use in `CostAICredits` and an operation-wide ledger.
Use unit-explicit `ai_credits` API field names; cost unit is always `ai_credits`.
Track reservations, reconciliation, remaining credits and overshoot separately.
Retain spending across revisions. There is no currency or currency-conversion adapter.

Metrics include stage/standing counts, pending candidates, remaining generations,
achieved/waived/unresolved obligations, proposal throughput, substantive attempts,
quota headroom, measured credits and runner/wall-clock usage.

### 13.1 Session log and retention

Append minimised structured JSONL to the operation's records branch at
`sessions/<session-id>.jsonl`. Record operation/repository ids, approved execution
revision, task generation, relevant inputs/evidence references, decisions and explicit
rationale, trusted tool outcomes, model/CLI identity, credit usage, wall-clock and
authenticated operator decisions.

Sensitive raw prompts/responses and available permitted traces belong in restricted,
operation-owned artifacts with explicit retention. Durable audit stores hashes and
references plus expiry/deletion evidence, not raw private payloads or credentials.
Deleting a Git file would not remove its historical contents: do not claim Git
rollups enforce raw-data deletion. Missing/expired payloads are marked, not silently
represented as retrievable. Do not feed operational logs back to agents as trusted
instructions or inherit them as state when reusing a definition.

## 14. Operation repository layout

Illustrative **generated repository**, not the layout this creator must implement:

```text
main (protected definition branch):
  operation.yaml
  README.md
  .tool-versions
  specification/{explorer.yaml,actor.yaml}
  .github/agents/{elicitor,supervisor,explorer,actor}.agent.md
  .github/skills/<skill>/SKILL.md
  .github/workflows/{activate,revise,scope,explore,act,reconcile,on-pr-event,control}.yml
  policies/{model-data-handling,retention}.yaml
  src/control-plane/
    board.ts
    claims.ts
    github-app.ts
    copilot.ts
    usage.ts
    rate-limits.ts
    estimate.ts
  schemas/
  dependency manifests and lockfiles
  local definition tests/fixtures

records (append-only operational record branch):
  approval/
  provenance/
  activations/
  sessions/<session-id>.jsonl
  decisions/
  usage/
  artifact-references/
```

The board and tracking issues are associated with this repository. The target-local
claim branch resides in each **target**, not here. Raw artifacts are restricted
operation-owned storage. Definition commits and the records branch must have
different write capabilities; record-writing jobs cannot rewrite approved definitions.

## 15. Settled decisions and validation gates

The five-round repository interview is complete and the operator confirmed the shared
model. §19 summarises reuse and ownership; ADR 0008 records the boundary change.

Before claiming implementation readiness, verify Copilot's actual configuration
isolation, usage export/attribution, available model policies and unattended access;
GitHub permissions/visibility; protected-definition versus records writes; and the
claim protocol under races/delayed requests. These are implementation facts to prove,
not user decisions silently assumed or reasons to invent fallback success.

## 16. Runtime implementation sequence

This is the dependency order for **operation-owned runtime capabilities**, maintained
as a tested base here and copied into operations. The creator's own workstream is §20.

1. Manifest/definition validation, approval/activation records, protected source and
   records separation, pinned tools and explicit readiness diagnostics.
2. Deterministic discovery/probes and read-only reconnaissance; public/private record
   and data-handling gates.
3. Proven target-local claim protocol, actor isolation, authoritative credit adapter,
   bounded task generation, independent acceptance and trusted draft publication.
4. Verified merge advancement, retry/reset/waiver semantics, revision handover,
   reactivation, emergency stop and safe retirement.
5. Supervisor allocation, credit reservations/overshoot, race/adversarial tests and
   comparable-credit estimation.

Exercise synthetic fixtures first. Do not enable target execution while required
proofs or bindings are missing. The creator's generation, validation and materialisation
helpers are distinct from this operational runtime (§20).

## 17. Worked unstaged example

Fixtures are specification/outcome test data, **not complete runnable repositories**.
Their credit values are synthetic, not actual Copilot measurements or conversions.

### 17.1 Intent

Before the operation repository exists, the operator asks:

> Modernise services still on Scala 2.13.12 to 2.13.14. Keep one small proposal per
> repository. Use a 50 AI-credit dispatch threshold; I will review and merge proposals.

### 17.2 Definition and creation

The Elicitor resolves detection, allowed targets, acceptance and bounds. The
specifications in `fixtures/operations/2026-10-scala-2-upgrade/` are approved as part of
a complete definition, alongside agents, skills, workflows, policies and runtime.
Create `acme/scala-2-upgrade-operation`, preserving approval and provenance; no
schedule may start target work merely because the repository was created.

### 17.3 Reconnaissance

The example finds 12 targets out of 140 candidates. The reduced fixture uses 20
candidates and preserves the same 12 targets. Its report leads with that count,
estimates 11 proposals and reports synthetic AI-credit and runner-time estimates.
The operator activates only after readiness and current reconnaissance pass.

### 17.4 Board

The operation repository owns the tracking issues and associated Projects v2 board.
Targets start `ready`, with a task generation and approved execution revision; `Stage`
and `Transition` remain unset because this example is unstaged.

### 17.5 Outcomes

Each action atomically acquires the target claim, invokes isolated Copilot, validates
changes and opens a draft proposal through trusted publication. `orders-service`
merges and passes final acceptance. `billing` and `scheduler` are already satisfied
by verified external changes. `legacy-etl` needs forbidden dependency edits and
`reporting` has conflicting work, so both need operator attention. `search` exhausts
substantive attempts and fails. Unknown or blocked work never becomes waived by itself.

### 17.6 Incomplete work and credits

The board has 7 `accepted`, 2 `no change needed`, 2 `needs operator` and 1 `failed`.
That is **not complete**: 9 achieved, 3 unresolved. Explicitly waiving those three
after resolving actors/proposals and claims would yield 9 achieved and 3 waived.
Stopping without those decisions concludes `incomplete`.

Synthetic measured usage is 7.4 AI credits against a 6.9-credit example forecast and
a 50-credit dispatch threshold. Retain that usage and all earlier outcomes across
later revisions; they do not reset when configuration changes.

## 18. Worked staged example

The approved goal is Scala 3.9 through **2.12 -> 2.13 -> 3.3 -> 3.9**. These are
illustrative milestones; real destinations and supported migrations must be pinned.

| Target | Entry stage | Remaining transitions | Expected proposals |
|---|---|---|---|
| `acme/old-service` | Scala 2.12 | 2.12 -> 2.13 -> 3.3 -> 3.9 | 3 |
| `acme/legacy-service` | Scala 2.13 | 2.13 -> 3.3 -> 3.9 | 2 |
| `acme/modern-service` | Scala 3.3 | 3.3 -> 3.9 | 1 |
| `acme/current-service` | Scala 3.9 | final acceptance only | 0 |

The report forecasts 4 targets, 6 transitions and 6 proposals. Synthetic credit
estimates differ by transition and sum to 5.85 credits, with 61 runner minutes
reported separately. Create `acme/scala-staged-upgrade-operation` only after approving
the full definition; activate explicitly.

`old-service` stays in the 2.12 column while its first proposal is open. Hold the
claim through review and default-branch verification, then record history, advance
to 2.13, release ownership and make the next generation ready. Three separate merged,
verified proposals eventually reach `accepted` at 3.9. Replayed earlier events cannot
reset its later work.

`legacy-service` reaches 3.3 but its final migration cannot succeed. The operator
resolves live work and explicitly waives the obligation with a reason; it remains
visible at 3.3 as `waived`. The final board has 3 achieved, 1 waived and 5 merged
proposals. Discovery is finished, so the operation concludes `complete`.

A same-intent approved revision adding Scala 3.10 may reopen achieved targets for
new work; retain spending and history, re-verify current content and reaffirm affected
waivers. It does not create another operation repository.

## 19. Ownership, reuse and confirmed boundary

The operation repository owns its definition, operational history, associated board,
activation/revision decisions and credit ledger. It can operate without a checkout,
service, moving branch or shared agent configuration from this creator.

Reuse a selected approved definition revision and record provenance. Start with
fresh identity, board, target discovery, envelope and approval. Never copy live
claims, attempts, waivers, proposals, history or bindings as operational state.
Local controller repairs arrive as reviewed definition changes, not automatic updates.
Private-to-public reuse requires a reviewed sanitised export with non-sensitive
public provenance. Definition structure, not just logs, can contain confidential data.

The earlier shared-control-repository, immutable-specification, public-only,
custom-model-loop and currency-based assumptions are superseded. Stage/standing
separation, deterministic qualification, operator merges and explicit completion
remain, now qualified by active execution revision and task generation.

## 20. Creator design: preparation to handover

**Status: creator architecture confirmed after three rounds of interrogation.**
This section defines what to build, not what already exists. This workspace owns
initial preparation and creation, not generated operations' execution or revisions.

### 20.1 Product and user journey

The product is a **Copilot-driven workspace repository**, not a separate hosted
service or a new conversational application. An operator starts from an intent, an
existing definition or an explicitly selected predecessor's approved revision.

```text
intent + scope + preparation threshold
             |
        interrogate <---- scoped read-only inspection
             |
  select supported base + curated/predecessor structure
             |
      assemble candidate definition bundle
             |
   deterministic validation + complete review report
             |
  real human approval of exact bundle and creation plan
             |
  journalled materialisation -> verified inactive handover
```

The Creator coordinates the conversation and calls deterministic tools. It asks for
missing decisions, explains unsupported capabilities and can resume a blocked
preparation. It cannot convert its own opinion into validation, approval or readiness.

### 20.2 Responsibilities and generation boundary

| Component | Responsibility |
|---|---|
| Creator agent | Interrogation, explanation, preparation planning and tool orchestration |
| Preparation skills | Structured questioning, explicit-source inspection, assembly, review and handover guidance |
| Curated starting points | Supported manifest/specification patterns and agent/skill templates |
| Runtime base | Tested controller/workflow safety machinery, copied as a versioned source snapshot |
| Assembly tools | Resolve approved base/template references, compile the candidate tree and produce its manifest |
| Validation tools | Check structure, executable closure, privacy, bounds and test evidence; emit explicit diagnostics |
| Approval channel | Capture authenticated human consent for a precise candidate and resource plan |
| Materialiser | Enforce approval, create/verify GitHub resources and journal partial results |
| Credit ledger | Keep measured/estimated preparation credits and corrections distinct |

Copilot may generate operation-specific specifications, agent/skill content and
bounded adapters. It must not rewrite claims, authority, privacy, credit enforcement
or publication guards for one operation. Workflow safety logic comes from the
maintained base; supported parameters are compiled, not arbitrary model-authored
replacement workflows. Unsupported safety requirements need a separately reviewed,
tested base change before preparation can use them.

Changing this repository's base does not hotfix existing operations. Newly created
operations copy a selected supported release; existing repositories continue with
their own source until their operator approves a local revision.

### 20.3 Private preparation workspace and lifecycle

Keep one resumable private workspace per preparation **outside this public checkout**.
It holds the intent/decisions, source references, candidate bundle, validation/review
reports, approval receipts, credit ledger and creation journal. Raw conversations and
payloads have explicit retention. Credentials are never bundle or journal contents.

```text
interrogating -> assembling -> validating -> awaiting approval
             -> approved -> materialising -> handed over
```

Failures pause at a recorded checkpoint with a reason. Abandonment does not delete
created resources. Draft changes invalidate applicable validation/approval; materialisation
uses a sealed approved snapshot, not a mutable working directory.

Model tools may edit candidate content, not trusted approval/journal records or
privileged helper source. The isolation boundary must be verified, not assumed from
placing files in different directories. Keep privileged credentials in the trusted
tool boundary; Copilot receives safe results, not tokens.

### 20.4 Inspection and reuse

V1 uses explicit predecessor repository/revision references and curated starting
points; no automatic operation discovery or ranking catalogue.

Inspect imported definitions as quarantined data. Do not run their code or auto-load
their agents, skills, hooks, workflows or MCP configuration into the Creator. Map
supported structure onto the chosen tested base and show every adaptation in review.
Missing approval/provenance, incompatible schemas and unsupported features are
reported; never silently drop features or pretend a source was verified.

Explicitly scoped read-only target inspection may ground criteria, probes and task
design after access/confidentiality checks. Preparation never claims targets, changes
target branches, opens proposals or executes untrusted target code. Generated-component
tests use synthetic data and isolated environments without privileged credentials.
Live migration rehearsals require a separately approved operation.

Private sources remain private. A public destination requires a separately reviewed
sanitised export; full private provenance is not copied into public records (§12).

### 20.5 Bundle, review and authentic approval

The candidate contains the complete manifest, specification, local agents/skills,
runtime/workflow snapshot, policy files, tool/dependency pins, overview and generated
component tests. The review report includes:

- Intent, scope, stages/tasks, acceptance, bounds, authority and privacy policy.
- Planned owner/name, visibility, definition/records branches and associated board.
- Base/template/source revisions, generated differences and source/export provenance.
- Validation/test results, required external bindings and known readiness blockers.
- Preparation credits, clearly distinguishing measurements, estimates and uncertainty.

Validate before asking approval: schemas, references and paths, complete executable
closure, unchanged tested safety base, generated-component tests, privacy/export
requirements and the consistency of the creation plan. Do not execute untrusted
candidate tests with creation credentials.

Copilot may invoke materialisation after **real conversational approval**. A trusted
channel captures consent from the registered operator/delegate and binds an approval
receipt to the exact payload digest, creation plan, visibility and runtime release.
The model cannot grant itself authority by editing a draft or passing `approved: true`.

The materialiser verifies that receipt and recomputes the digest from the exact bytes
it will publish. Changed contents, destination or release require renewed review and
approval. Resumption uses the same sealed payload and receipt, not a newly generated
approximation. If the pinned Copilot integration cannot prove authentic consent,
materialisation blocks: natural-language assertions are not a fallback.

### 20.6 Preparation credit accounting

Preparation has its own declared AI-credit threshold and ledger, separately from the
operation's execution envelope. Preserve its usage/basis in handover and provenance.
Do not deduct it silently from the new execution budget or call an estimate a measurement.

Each attributable call has one current accounting basis: measured, estimated or unknown.
Use measured credits plus outstanding meaningful estimates and reservations to decide
whether another automatic call fits the threshold. Label mixed totals as partly
estimated. Missing measurement may use a declared estimate; missing both requires
operator resolution, not fabricated zero. Estimates do not establish a hard ceiling.

When authoritative usage arrives, replace the matching estimate through an audit
correction, never add both. Deduplicate reports and do not assign unrelated account-wide
usage to one preparation. Stop new automatic calls at the threshold and disclose
uncertainty/in-flight overshoot. Deterministic inspection/recovery does not authorise
more inference. Generated operations still require authoritative usage before
execution (§8); estimated preparation accounting cannot weaken their meter gate.

### 20.7 Materialisation, recovery and handover

The trusted materialiser creates the approved operation repository, definition and
records branches, required protections and associated board, then transfers the
approved definition and records. Configure bootstrap as inactive; pushing workflows
or enabling the platform must not start target work or spend the execution envelope.

Journal intended resources before effects and verified ids/results afterwards.
On retry, verify both remote identity and its association with this preparation and
approved payload before reusing a resource. Names alone are not ownership evidence.
An interrupted response or unrelated name collision blocks for explicit resolution.
Never silently overwrite, rename or delete existing resources, including partial
resources created by this preparation.

Handover requires the necessary created resources to exist and match their approved
settings. Transfer approval, provenance, preparation usage/basis, creation evidence
and a setup/readiness report into operation-owned records. The result is **inactive**.
Access grants, installations, credentials and other external bindings remain explicit
operator work. Missing bindings are readiness blockers; failed required creation steps
are partial/blocked preparation, not a completed handover.

A later preparation-usage correction may be supplied as a receipt for import. It is
not permission to become the operation's live ledger or updater. V1 ends at initial
creation/handover; local operation tools manage subsequent revisions.

### 20.8 Planned repository layout

Illustrative build structure; none of these agents/helpers are implemented yet:

```text
.github/agents/creator.agent.md
.github/skills/
  interrogate-operation/SKILL.md
  inspect-operation-source/SKILL.md
  assemble-operation/SKILL.md
  review-operation/SKILL.md
  materialise-operation/SKILL.md
src/creator/
  preparation.ts       # lifecycle and checkpoint model
  bundle.ts            # assembly, closure and exact snapshot identity
  validate.ts          # structured diagnostics and validation gates
  credits.ts           # measured/estimated ledger and correction rules
  approval.ts          # verified receipt contract; no model self-approval
  materialise.ts       # approved plan, journal and resume orchestration
  github.ts            # restricted, effectful GitHub resource operations
runtime/               # maintained operation-owned controller/workflow base
templates/             # curated specifications, agents, skills and policy patterns
schemas/               # preparation, manifest, review and receipt contracts
tests/                 # pure-model, isolation and authorised sandbox integration tests
fixtures/              # preparation and operation contract examples
.tool-versions         # mise-managed helper/runtime tool versions
```

Keep pure lifecycle, identity, accounting and validation rules separate from filesystem,
GitHub and Copilot effects. Tool interfaces are bounded operations, not an unrestricted
GitHub API or shell gateway. The private workspace and credentials are not directories
to add to this public source tree.

### 20.9 Implementation sequence and acceptance

1. **Prove the boundaries.** Prototype authentic conversational approval, safe
   credential separation, source/configuration quarantine and usage attribution/
   labelled estimation for the pinned CLI. Unsupported authority/isolation blocks
   materialisation; do not paper over gaps with model assertions.
2. **Establish the tested foundation.** Implement schemas and the operation runtime
   workstream in §16, with versioned test evidence and safe inactive bootstrap.
   Copied code must actually exist and pass its required tests before calling a
   bundle validated or independently runnable.
3. **Build pure preparation tools.** Lifecycle/checkpoints, template assembly, exact
   digest/plan validation, review output and credit correction logic. Use
   property-based tests for identity stability, accounting and retry invariants.
4. **Add the Creator experience.** Agent/skills for intent interrogation, scoped
   read-only inspection and explicit predecessor adaptation into private workspaces.
   Exercise existing unstaged/staged fixtures; keep imported configuration inert.
5. **Materialise and recover.** Create authorised sandbox repositories/boards,
   transfer records, verify inactive behaviour and inject partial failures. Repeated
   calls must resume verified owned resources without duplicates or unrelated writes.
6. **Complete handover and harden.** Test private-to-public export, missing permissions,
   changed payloads, forged approval, unknown/estimated credits and external readiness
   blockers. Do not label published files as a successful handover when required
   resources/protections failed.

The end-to-end acceptance is: an intent becomes a reviewed, tested and genuinely
approved bundle; the creator materialises exactly that bundle into the intended
repository and associated resources; the operation stays inactive; the operator
receives truthful provenance, usage and setup blockers; rerunning after interruption
does not duplicate or corrupt resources.

## Sources for capability checks

- [Copilot CLI usage, custom agents, skills and programmatic invocation](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/use-copilot-cli)
- [GitHub REST Git reference creation, fast-forward updates and deletion](https://docs.github.com/en/rest/git/refs)

These references establish available primitives, not proof that the complete
isolation, metering or claim protocol has been implemented and validated.
