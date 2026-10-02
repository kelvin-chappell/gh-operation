# Operation Runner — Design

A system of **elicitor**, **supervisor**, **explorer**, and **actor** agents that operate
over a collection of repositories. An **operator** states an **intent**; the system
clarifies it by structured **interrogation**, turns it into a machine-checkable
**specification**, discovers matching repositories onto a GitHub Projects v2 **board**,
and performs the **task** applicable to each **target**. An operation may prescribe
ordered **stages**, with a separate task and change proposal for each transition.
An unstaged operation retains one task and at most one change proposal per target.

Status: **design complete, ready for implementation**. Every decision is marked
`[DECIDED]`; nothing is left `[OPEN]`.

The domain vocabulary is defined in `CONTEXT.md`. This document uses it throughout.
Load-bearing decisions are recorded as ADRs under `docs/adr/`. Where the GitHub
platform has its own name for an artifact, the platform name appears
in `backticks`:

- a **target** is held on the board as a GitHub Projects v2 *item*, which is a
  **tracking issue** in the control repo;
- a **change proposal** is, in v1, a GitHub *pull request* (so `pull_requests:*`
  scopes and the `on-pr-event` workflow keep the platform name);
- a **standing** is stored in the board's `Standing` single-select field;
- a **stage** is stored separately in `Stage`; staged board views group by that field;
- an **exploration** and an **action** are each dispatched as a GitHub Actions *job*
  (a matrix entry);
- a **session log** is a JSONL file;
- **reconnaissance** was earlier called a "dry run".

---

## 1. Goals and non-goals

### Goals
- Turn a vague intent into a **frozen, versioned specification** before any repository
  is touched.
- **Discover** candidate repositories with explicit, reproducible **criteria**.
- **Advance targets towards one goal**, applying the frozen task for their current
  transition, or one uniform task in an unstaged operation.
- **Track** each target's **stage** and **standing** on a GitHub Projects v2 board.
- **Complete** only when every qualified target has satisfied the goal or has an
  explicit operator waiver; unresolved blockers and exhausted budgets are not success.
- **Supervise**: choose how many explorers and actors to run, their concurrency, and
  their models, to hit a cost/throughput target.
- Be **idempotent and resumable** — reruns never double-open change proposals or
  corrupt standings.
- **Scope before spending**: reconnaissance reports the size and projected cost of a
  operation before any repository is touched (§9).

### Non-goals
- Perfect autonomous judgement with no human gate. The specification freeze and the
  change-proposal merge stay human-gated; autonomy expands only after a reviewed pilot
  justifies it, never by default.
- Supporting non-GitHub forges.
- Running over private or sensitive repositories in v1: the control repo is public, so
  it would expose them.
- Multi-tenant SaaS in v1.

### Locked decisions `[DECIDED]`
| Area | Choice |
|---|---|
| Agent harness | Custom minimal loop over an LLM API + GitHub tools (no agent framework) |
| Runtime | GitHub Actions (workflows, matrix fan-out, scheduled reconciliation) |
| Change proposal | At most one per target and transition; unstaged operations retain one per target |
| Stages | Optional ordered milestones, detected deterministically; one adjacent transition at a time |
| Board columns | Operation-defined stage names, separate from lifecycle `Standing` |
| GitHub auth | GitHub App (installation tokens) |
| Target scope | Our own org(s) only; same-repo branch, no forks (v1) |
| Models | Only models available through Freebuff |
| Autonomy | Operator approves the specification; every change proposal is a draft the operator merges (initially) |
| Qualification | Deterministic validator only; the explorer's LLM records evidence, never promotes |
| Board | One Projects v2 project per operation |
| Re-entry after failure | `failed` is terminal; only an operator `reset` returns a target to `ready` with a fresh attempt budget |
| Transient failures | Infrastructure failures retry at the job level (≤3, capped backoff) and never consume an `Attempt` |
| Resolving blockers | Operator `resume` → `ready`, `reset` failed work, or `waive` → `waived` with a reason; `dismiss` does not waive qualified work |
| Reconnaissance gate | An operation cannot go `active` without a reconnaissance for the current specification hash (operator override allowed) |
| Reconnaissance carry-over | A forecast, not a reservation: the real operation re-discovers; reconnaissance informs only the estimate and the plan |
| Content-probe freshness | Content is re-probed at qualification, and triage re-reads; caches live within a single run only |
| Specification revision | A revision is a **new operation** (new id, new board); the old one concludes or is abandoned |
| Cross-operation collision | An open automation change proposal on the repository from any other operation → `needs operator` |
| Language/runtime | TypeScript on Node 20 with `@octokit/*` (v1) |
| Supervision location | A scheduled GitHub Actions job |
| First-operation estimate | Accept the `heuristic_v1` estimate; no mandatory pilot |
| Control repo visibility | **Public**: the control plane, board, and session logs are all public |
| Target sensitivity (v1) | Public / non-sensitive repositories only, enforced by the specification's scope |
| Session-log retention | Full traces for 90 days (or the last N operations), then a summary + trace digest |
| Already-compliant targets | Verified final goal → `no change needed` if no proposal was merged; intermediate progress selects the next transition |
| Task divergence | Intent fixed per transition (or unstaged task); implementation may adapt within frozen bounds |
| First deliverable | This design document |
| Criteria depth | Metadata **and** file-content criteria (e.g. "which Scala versions does this repo use?"); the deterministic validator checks the extracted content |
| Existing change proposal on a target | Detect the conflict and mark `needs operator`; the actor neither duplicates, adopts, nor silently skips |
| Adaptation bounds | Dependency/lockfile changes forbidden by default; a frozen transition may explicitly permit necessary migration edits |
| Operation completion | Discovery finished and every qualified target at the final goal or explicitly `waived` |
| Operation conclusion | Completion, or an explicit incomplete stop; exhausted resources pause, not successful completion |

Settled (DECIDED): **TypeScript on Node 20** with `@octokit/*` for the control plane.
Octokit's REST + GraphQL coverage is the shortest path to Projects v2. Python remained
viable, but the choice is reversible: nothing structural depends on it.

---

## 2. Key constraints that drive the design

1. **A Projects v2 item is an issue, a change proposal, or a draft issue — it cannot
   point at a repository directly.** Resolution: each discovered repository becomes a
   **target** represented by a **tracking issue** in the *control repo*; that issue is
   the target's entry on the board, and the target repository (`owner/name`) lives in a
   board field and the issue body.
2. **Search API is separately rate-limited** (on the order of 30 requests/min for
   authenticated REST search), independent of the core API limit. Adding exploration
   parallelism beyond that buys nothing and burns **quota**. The supervisor must model
   *two* budgets: core calls and search calls.
3. **Actions run untrusted target-repo code.** Building/testing a stranger's repository
   on a runner can execute hostile code. This is a security boundary, not an
   afterthought.
4. **Repository content is untrusted input to an LLM** (prompt injection from
   READMEs/issues/CI files). The tools granted to an actor must be minimal and
   non-exfiltrating.
5. **Actions has finite parallelism** (matrix ceiling, concurrent-job ceilings,
   per-runner minutes). How many actions run at once is bounded by runner concurrency
   and budget, not by the number of eligible targets.
6. **The model catalog is fixed.** Only models available through Freebuff may be used,
   and the supervisor tier additionally needs a model that exposes reasoning traces
   (§13.1). Every tier assignment in §8 is constrained to that catalog.
7. **Criteria can require reading file contents.** A criterion may be "which Scala
   versions does this repository use?" rather than mere metadata, so exploration must
   fetch and parse repository files, and the deterministic validator must check the
   extracted content — not just search metadata. Content reads happen under a read-only
   token, so they are safe in reconnaissance too.

---

## 3. Components

```
                    ┌───────────────────────────────┐
                    │   Control repo (this project)  │
                    │  specifications · workflows ·  │
                    │  tracking issues               │
                    └───────────────┬───────────────┘
                                    │
                ┌───────────────────┼────────────────────┐
                ▼                   ▼                    ▼
        ┌──────────────┐   ┌──────────────┐   ┌───────────────────┐
        │   Elicitor   │   │  Supervisor  │   │  Board Controller │
        │(interrogation)│  │ (planning /  │   │ (Projects v2 API,  │
        │              │   │  reconcile)  │   │  claiming,         │
        │              │   │              │   │  standings)        │
        └──────────────┘   └───────┬──────┘   └─────────┬─────────┘
                                   │ emits matrix          │
                          ┌────────┴────────┐              │
                          ▼                 ▼              │
                   ┌────────────┐   ┌────────────┐        │
                   │Explorations│…  │  Actions   │…       │
                   └────────────┘   └────────────┘        │
                          │                 │             │
                          └─── create/claim targets ──────┘
                          └─── open one change proposal ──┘
                               per target and transition
```

### 3.1 Elicitor
An interactive loop that interviews the operator until the specification satisfies a
rubric, then freezes it. It produces one specification in two files: `explorer.yaml`
(criteria) and `actor.yaml` (stages, transition tasks and acceptance, or an unstaged
task and acceptance) — schemas in §5. It runs as a GitHub
issue conversation, driven by a workflow. See §10.

### 3.2 Supervisor
A stateless planning function invoked at operation start and on every reconciliation
tick. It reads the frozen specification, current stages and standings, the envelope, and live
rate-limit headroom; it emits:
- the **exploration matrix** (how many explorations, and how the search space is
  partitioned),
- the concurrency target for actions this tick,
- the **model assignment** per role,
- a stop/pause decision when the envelope or quota is exhausted.

In **reconnaissance** the supervisor instead consumes the explorer's read-only sweep and
emits a **feasibility report** (projected size, cost, wall-clock) rather than a work
plan. See §9. The supervisor does no repository work itself: it only allocates. See §8.

### 3.3 Explorer
Given one partition of the search space, an exploration runs deterministic GitHub
Search calls, applies metadata filters and any **content probes** the criteria require
(e.g. reading `build.sbt` to detect Scala versions), scores/filters candidates (a cheap
LLM only on ambiguous cases), and creates a tracking issue — the target's entry on the
board — per qualifying repository, standing `discovered`. In reconnaissance it performs
the same discovery **read-only** and reports the candidate set without creating any
targets or writing to the board. See §6 and §9.

### 3.4 Actor
An action atomically **claims** one `ready` target, verifies its current stage and
selects the prescribed transition (or unstaged task), then clones the target repository,
applies the frozen task, runs the
specification's acceptance commands, opens a change proposal, and moves the target to
`proposal open` (or `no change needed` / `failed` / `needs operator`). See §7.

### 3.5 Board Controller
A library plus a scheduled workflow wrapping the Projects v2 GraphQL API: read/write
board fields, claim with optimistic concurrency, transition **standings**, verify stage
advancement, and reconcile drift (e.g. a merged or closed change proposal whose target
is stale). See §4. The
Board Controller is a mechanism, not an agent.

---

## 4. Operation lifecycle and target standings

### Operation standing

The operation as a whole moves through:

`draft` (intent stated, interrogation open) → `specified` (specification frozen) →
`scoped` (reconnaissance run) → `active` (exploration and action underway) →
`concluded` or `abandoned`.

### Board fields

The board (Projects v2) holds one entry per target:

| Field | Type | Purpose |
|---|---|---|
| `Standing` | single select | the target's standing lifecycle (below) |
| `Target` | text | `owner/name` of the target repository |
| `Operation` | text | operation id |
| `Stage` | single select | verified achieved stage; options come from the frozen specification |
| `Transition` | text | applicable transition id; empty at the final stage or for unstaged work |
| `Match` | text | the exploration's evidence / matched criteria |
| `ChangeProposal` | text | current transition's proposal, or the last proposal once finished |
| `Attempts` | number | substantive attempts for the current transition (unstaged: the task) |
| `CostUSD` | number | accumulated model + runner cost |
| `Claim` | text | the exploration or action holding the claim |
| `ClaimedAt` | text | ISO timestamp of the claim |

### Stages and board views

Stages are optional. Without them, the existing task/acceptance format and a view
grouped by `Standing` remain valid. For a staged operation, `Stage` is independent
of `Standing`: a Scala 2.12 repository remains in **Scala 2.12** while an action or
proposal to reach 2.13 is underway. An intermediate merge advances `Stage` only
after verification on the default branch, then returns `Standing` to `ready`.

The primary staged board view groups by `Stage`, giving operation-specific columns
such as **Scala 2.12**, **Scala 2.13**, **Scala 3.3**, and **Scala 3.9**. Cards show
`Standing`, so blocked and waived targets remain visible at their last achieved stage.
A companion view groups by `Standing`, including `needs operator`, `failed`, and
`waived`. Undetermined stages are left unset and visible in the unassigned group
with `needs operator`; never invent a milestone to hide a detection failure.

One tracking issue remains the durable record for a repository. Its transition
history retains proposal URLs, attempts, verification evidence, costs and waivers;
advancing a stage resets the current attempt count, not the history or total cost.
Manual board moves are not evidence of progress: the controller re-probes before
dispatch and rejects or repairs inconsistent stage fields.

### Target standings and transitions

```
discovered ──qualify──▶ ready ──claim──▶ in action ──▶ proposal open
     │                     ▲                  │                │
     │                     └─── retry ────────┤                ▼
     ▼                                        ▼          needs operator
  excluded                            no change needed         ▲
                                                                │
                 conflicting/foreign change proposal found ─────┘
                 in action / proposal open ──▶ failed (terminal after N attempts)

Recovery edges (operator-initiated):
  failed          ──reset──▶  ready      (fresh attempt budget; own branch reused)
  needs operator  ──resume─▶  ready      (blocker re-checked before acting)
  needs operator / failed ──waive──▶ waived (operator reason required)

Verified merge:
  proposal open ──intermediate stage reached──▶ ready (next transition)
  proposal open ──final goal reached─────────▶ accepted

Verified external progress:
  ready / in action ──intermediate stage reached──▶ ready (next transition)
  ready / in action ──final goal reached─────────▶ no change needed or accepted
```

- `discovered`: an exploration created the target with evidence.
- `ready`: qualified by the **deterministic validator**, with a verified stage and
  one applicable transition (or an unsatisfied unstaged task). Admission criteria
  are not re-used as progress checks: successful upgrades must not eject targets.
- `in action`: claimed by an actor with a **lease** (`Claim` + `ClaimedAt`). A stale
  lease (older than the action's timeout) is reclaimable by reconciliation.
- `no change needed`: verification found the final goal already satisfied, without
  any merged proposal from this operation. A
  terminal standing **distinct from `failed`**; no change proposal is opened, but the
  triage cost is still recorded.
- `accepted`: the final goal is verified and this operation has at least one merged
  proposal for the target. An intermediate accepted proposal is history, not this
  target standing.
- `waived`: the operator explicitly leaves a qualified target short of the goal,
  recording a reason, identity, timestamp and last verified stage. Actors and retry
  exhaustion cannot grant waivers. Open proposals must first be resolved or closed
  by the operator, and any active claim released; never waive work still in flight.
- `excluded`: a candidate did not qualify or was outside scope. An already-qualified
  target cannot be dismissed to manufacture completion; use a waiver instead.
- `needs operator` is reached before any change proposal when the target already has a
  conflicting or foreign change proposal on the working branch (DECIDED): the action
  records the conflict and stops. It does not duplicate the proposal, adopt it, or
  silently skip the target — the operator decides.
- **Cross-operation collision (DECIDED).** Branch names embed the operation id, so a
  repository already handled by another operation would not trip the branch-level check.
  Before acting, an action also checks whether the repository already has an open
  automation change proposal from *any* operation, or is `in action` elsewhere; if so the
  target goes to `needs operator` rather than opening a competing proposal.
- **`failed` is terminal (DECIDED).** Only an operator `reset` returns the target to
  `ready` with a **fresh attempt budget**. `max_attempts` bounds the automatic retries
  within a transition, not across transitions or resets. On reset the target reuses
  its current transition's branch (rebased onto the default-branch head, acceptance
  re-run); if the branch was deleted it starts
  fresh under the same deterministic name.
- **`needs operator` leaves only by operator decision (DECIDED).** `resume` re-checks
  the blocker and current stage, then routes to `ready` or a verified final outcome.
  `waive` closes unresolved qualified work as `waived`; `dismiss` is limited to
  candidates that never qualified.
  Reconciliation may comment that a blocker appears cleared, but never transitions a
  human-gated target on its own.
- **Transient failures do not consume attempts.** A 403 secondary rate-limit, a 5xx, a
  lost runner, or a network timeout is retried at the **job level** — at most 3 times,
  exponential backoff with jitter, capped — and never increments `Attempts`.
  Substantive failures do, and advance the target toward `failed`.
- **Operation completion (DECIDED).** After discovery and qualification are exhausted,
  including every surfaced candidate (`discovery_complete` in reports), every qualified
  target must either have verified final acceptance (`accepted` or `no change needed`)
  or be explicitly `waived`. `ready`, `in action`, `proposal open`, `needs operator`
  and `failed` do not count. An intermediate stage, a merge alone, or an excluded
  qualified target cannot satisfy the rule. Before concluding, confirm each achieved
  target's verification still refers to its current default-branch head; re-verify
  changed heads. Changes after conclusion belong to a later operation.
- **Conclusion is not necessarily completion.** Keep operation standing `concluded`,
  but record the conclusion outcome as `complete` or `incomplete`, with achieved,
  waived and unresolved counts. Exhausting the envelope pauses dispatch; the operator
  may stop as `incomplete` or abandon the operation. Declaring it done cannot bypass
  the completion rule.

Transition rules are enforced in the Board Controller, not in prompts. An agent
*requests* a transition; the controller validates that the current standing and the
lease permit it. Stage and next-transition updates happen together; duplicate webhook
deliveries cannot advance twice or reset a later transition's attempt budget.

---

## 5. Data model

The specification is committed to the control repo under `operations/<operation-id>/` so
every operation is reproducible and pinned to its frozen specification. It is one
specification made of two files.

### 5.1 Exploration specification (`explorer.yaml`)
```yaml
operation: 2026-10-cve-sweep
scope:
  owners: [acme, acme-labs]        # org/user allow-list
  exclude_repos: [acme/dotfiles]
  visibility: public               # v1: private repos are refused (public control repo)
criteria:                           # all must hold; metadata AND content allowed
  metadata:                         # cheap, evaluated from search/repo metadata
    private: false                 # v1: never a private repo
    languages: [Go, TypeScript]
    topics_any: [backend, service]
    archived: false
    fork: false
    pushed_after: 2025-01-01
    min_stars: 0
    has_ci: true
    license_in: [MIT, Apache-2.0, BSD-3-Clause]
  content:                          # requires fetching and parsing repo files
    - id: scala_version
      match: any                    # all | any | none
      file: build.sbt
      extract: regex
      pattern: 'scalaVersion\s*:=\s*"([^"]+)"'
      in: ["2.13.12", "2.13.14"]   # repo must use one of these versions
partition:
  strategy: by_owner              # by_owner | by_language | by_topic | manual
  max_partitions: 8
llm_use:
  annotate_only: true              # LLM records evidence; it never promotes
  model: cheap-classifier
qualification:
  validator: deterministic         # DECIDED: criteria — metadata and content alike —
                                   # are machine-checked in code against extracted values
```

### 5.2 Action specification (`actor.yaml`)
```yaml
operation: 2026-10-cve-sweep
task:
  title: "Add Dependabot config for Go modules"
  kind: code_change              # code_change | config_change | docs
  instructions: |
    Add a .github/dependabot.yml pinned to the org standard. Do not reformat
    unrelated files. Preserve existing ecosystems.
acceptance:
  build: "make build"            # optional; skipped if absent
  test: "go test ./..."
  lint: "golangci-lint run"
pr:                                # platform: how the change proposal is opened
  branch: "automation/{{operation}}-{{repo_slug}}"
  title: "{{task.title}}"
  body_template: .github/pr-template.md
  labels: [automation, operation/{{operation}}]
  draft: true
triage:
  on_already_satisfied: no_change_needed   # no_change_needed | attempt_anyway
adaptation:
  intent: fixed                 # DECIDED: the task's intent never changes
  implementation: within_bounds # may adapt per repo, within the limits below
  may_add_files: true           # new files that are part of the change are allowed
  forbidden_changes:            # DECIDED: any of these → needs operator, never applied
    - dependencies              # manifest edits (package.json, go.mod, build.sbt, …)
    - lockfiles                 # go.sum, package-lock.json, yarn.lock, pnpm-lock, …
limits:
  max_attempts: 2
  wall_clock_minutes: 30
  max_files_changed: 15
retry:
  transient: [secondary_rate_limit, server_error, runner_lost, network_timeout]
  max_transient_retries: 3
  backoff_cap_minutes: 5
escalation:
  on_failure: escalate_model     # escalate_model | mark_failed | needs_human
  on_out_of_bounds: needs_human  # adaptation that would exceed the limits
  on_existing_pr: needs_human    # DECIDED: conflicting/foreign change proposal
```

### 5.3 Staged action specification

`actor.yaml` uses exactly one of two formats: top-level `task` and `acceptance` for
unstaged work, or `stage_probe`, `stages` and `transitions` for staged work. Shared
`pr`, `limits`, `retry`, `escalation` and adaptation defaults still apply. Transition
tasks and acceptance are frozen, not invented by the model at dispatch time.

```yaml
operation: 2026-10-scala-staged-upgrade
stage_probe:
  match: all                     # every extracted version must fit the same stage
  file: build.sbt
  extract: regex
  pattern: 'scalaVersion\s*:=\s*"([^"]+)"'
stages:                          # order defines progress; the last stage is the goal
  - id: scala-2-12
    name: Scala 2.12
    when: { matches: '^2\.12\.[0-9]+$' }
  - id: scala-2-13
    name: Scala 2.13
    when: { matches: '^2\.13\.[0-9]+$' }
  - id: scala-3-3
    name: Scala 3.3
    when: { matches: '^3\.3\.[0-9]+$' }
  - id: scala-3-9
    name: Scala 3.9
    when: { matches: '^3\.9\.[0-9]+$' }
transitions:
  - id: to-scala-2-13
    from: scala-2-12
    to: scala-2-13
    task:
      title: Upgrade to Scala 2.13
      kind: code_change
      instructions: |
        Upgrade to Scala 2.13.14, adapting code and dependencies as necessary.
        Do not reformat unrelated code.
    acceptance: { build: "sbt compile", test: "sbt test" }
  - id: to-scala-3-3
    from: scala-2-13
    to: scala-3-3
    task:
      title: Upgrade to Scala 3.3
      kind: code_change
      instructions: |
        Upgrade to Scala 3.3.6, migrating code and dependencies as necessary.
        Do not reformat unrelated code.
    acceptance: { build: "sbt compile", test: "sbt test" }
  - id: to-scala-3-9
    from: scala-3-3
    to: scala-3-9
    task:
      title: Upgrade to Scala 3.9
      kind: code_change
      instructions: |
        Upgrade to Scala 3.9.0, migrating code and dependencies as necessary.
        Do not reformat unrelated code.
    acceptance: { build: "sbt compile", test: "sbt test" }
pr:
  branch: "automation/{{operation}}-{{repo_slug}}-{{transition}}"
  title: "{{task.title}}"
  draft: true
adaptation:
  intent: fixed                  # fixed per transition, not one task for all stages
  implementation: within_bounds
  may_add_files: true
  forbidden_changes: []          # operator explicitly permits migration dependencies
limits:
  max_attempts: 2                 # per target and transition
  wall_clock_minutes: 30          # per action
  max_files_changed: 15
```

The standalone example is in `fixtures/operations/2026-10-scala-staged-upgrade/`.
The versions are illustrative milestones supplied by the operator, not a claim
about released versions. Interrogation must pin real destination versions and
supported migration paths before a real operation is frozen.

Validate unique stage ids and names, at least two stages, unique transition ids,
and exactly one forward transition between each adjacent pair. Reject missing,
backwards, branching or cyclic paths. Each transition requires a task and acceptance;
its optional adaptation block replaces the explicitly named defaults at freeze time.
The frozen effective bounds must be visible to the operator.

Stage detection reads the default branch and applies the declared probe and predicates
in code. Exactly one stage must match **all** extracted values. No matches, overlapping
matches, mixed cross-build versions, missing files, malformed values or probe errors
produce explicit evidence and `needs operator`, never a model-selected stage. An
operator resolves the detection issue or waives the target; they cannot simply assign
an unverified stage. A target already on 2.13 starts there; it does not run the 2.12 task.
The sample probe supports directly declared `scalaVersion` values. Dynamic versions
or cross-build declarations require an operator-approved probe that extracts every
relevant value; do not infer whole-repository progress from one convenient match.

Advancement requires destination-stage evidence and successful destination acceptance.
The final transition's acceptance also verifies targets that enter at the final stage
or reach it through external changes; a version string alone is not final acceptance.
CI on the proposal branch permits opening a proposal, but does not advance the board.
After merge, re-check on the default branch. Verification errors or drift go to
`needs operator`, retaining the last verified stage and the observed discrepancy.
Unexpected regressions never silently reopen a completed transition.

### 5.4 Tracking issue (one per target)
Created in the control repo; the body records the target repository, the operation id,
and the exploration's evidence. Labels: `operation/<id>`, `target`. This issue **is** the
target's entry on the board. History entries identify the operation, repository,
transition and specification hash, with from/to stages, proposal URL, attempt
outcomes, verification evidence and accumulated cost. Waivers and operator decisions
are append-only history too; updating the current board fields never erases them.

---

## 6. Explorer design

**Loop (per partition of the search space):**
1. Run deterministic search: `GET /search/repositories` with the criteria translated
   to query qualifiers. Handle pagination within the search quota.
2. Apply metadata hard filters in code (cheap, exact) to shrink the candidate set.
3. Apply **content criteria** (if any) by fetching the named files read-only and
   extracting the value with the declared method (e.g. regex over `build.sbt` to read
   the Scala version). Probes run only after metadata filters, so they are scoped to
   survivors and stay cheap; the extracted value is recorded as evidence.
4. For borderline cases only, ask the cheap model to **annotate** the candidate with a
   structured verdict + reason (JSON schema in the tool call, not free text). Its
   output is recorded as evidence only — **promotion to `ready` is decided by the
   deterministic validator against the frozen criteria, never by the model**
   (DECIDED). The validator checks extracted content values, not only metadata.
5. For each survivor: create the tracking issue, add the target to the board, set its
   fields, standing `discovered`. Qualification then detects its starting stage from
   fresh probes; an already-final target needs no action. A qualified target whose
   stage cannot be determined remains on the board as `needs operator`.
6. Emit a summary artifact (candidates seen, matched, cost, search calls used).

**Efficiency rules:**
- Deterministic first, LLM last. Never send a repository to an LLM that a query
  qualifier already disqualifies.
- Dedup by repository id across partitions **and** across reruns (query existing
  targets before creating).
- Cap pages per partition (`max_pages`) to bound the worst case.
- Cache content probes per repository within one exploration run, and
  probe only after metadata filters have shrunk the set.

**Tools granted:** `search.code`/`search.repos` (read), `contents.read` on target
repos (for content criteria), `issues.create` in the control repo, `projects.write`
(scoped to the operation project). No write access to target repositories. No secrets
from target repositories.

---

## 7. Actor design

**Loop (per claimed target and applicable task):**
1. Claim atomically (see §4); bail if lost. Pin the specification hash, observed
   default-branch head and current transition in the claim.
2. **Detect an existing change proposal** on the branch (ours, foreign, or
   conflicting). If one exists that is not this target's current transition's own
   resumable proposal, record the conflict and move the target to `needs operator` —
   do not duplicate, adopt, or
   skip (DECIDED). Also check for an open automation change proposal on the repository
   from any *other* operation (§4) and treat it as a conflict the same way.
3. Clone the default branch shallowly and re-probe its stage. If it differs from
   the claimed stage, release the claim and re-plan after verification, rather
   than applying a stale task. Unrecognised stages or regressions go to `needs operator`.
4. **Triage** (deterministic stage/acceptance checks, assisted by cheap model inspection):
   run required acceptance in the isolated runner, never with broader credentials.
   Verified final acceptance yields `no change needed` if no operation proposal has
   merged, otherwise `accepted`. Verified external progress to an intermediate
   stage selects the next transition; it never finishes the target. If there is no
   progress, select the frozen task for the current transition, then check out its
   deterministic branch. Unstaged operations retain their existing task triage.
5. Snapshot a clean baseline; capture build/test command discovery (package manager,
   CI config). Executed commands still come only from the frozen acceptance.
6. Produce the change via the coding model using the frozen `task.instructions` +
   acceptance criteria; edits are applied as file operations. The task's **intent is
   fixed for this transition**; the implementation may adapt to the repository, but
   only within `limits.max_files_changed` and the acceptance commands (DECIDED). New files that
   are part of the change are allowed. **Dependency-manifest and lockfile edits are
   forbidden by default**, unless explicitly permitted by the frozen migration
   bounds; forbidden edits end in `needs operator`. Any adaptation that would exceed
   the bounds also ends in `needs operator`, never a wider diff.
7. Verify the destination-stage predicate and run the transition's `acceptance.*`
   commands in the runner (sandboxed: no org secrets, no broad token, network egress
   restricted where possible). Unstaged work uses its top-level acceptance.
8. If green: commit, push, open the change proposal per the `pr` block; set the board's
   `ChangeProposal` field, standing `proposal open`; keep the achieved `Stage` unchanged.
9. If red or the model is stuck: per `escalation.on_failure`, either escalate the
   model for one more attempt or move to `failed` / `needs operator`.
10. Transient failures (secondary rate limit, 5xx, lost runner, network) are retried at
    the job level — at most 3 times with capped backoff — and never consume an
    `Attempt`; only substantive failures advance the target toward `failed`.

**Idempotency:** staged branch names include operation, repository slug and transition
id. Proposal identity is `(operation, repository, transition)` and is checked against
recorded ownership, not inferred from a name alone. Unstaged work keeps its existing
branch name and identity `(operation, repository)`. Only one proposal may be open
per target at a time; no stacked proposals. Retries update the same unmerged proposal,
never a previous transition's merged branch. If it pre-exists from another source — a
foreign branch, a human's change proposal, or a conflicting change — the action does not
touch it: it moves the target to `needs operator` (DECIDED). `Attempts` is incremented
for the current transition so reconciliation can enforce `max_attempts`. Advancing
starts a fresh transition budget; resetting failed work preserves earlier progress.
Closing a proposal without merge goes to `needs operator`; it does not advance the
stage or automatically create a replacement. The operator may resume the same proposal
after resolving the closure, or waive the work.

**Merge reconciliation:** match webhook events to their recorded transition and
specification hash. Re-verify destination acceptance on the current default branch,
archive the transition outcome, then advance the stage. Clear `ChangeProposal` and
`Attempts` when moving to a new transition; keep the final proposal URL on a completed
target. A duplicate or late event for an earlier transition only updates its history,
never the current stage, standing or claim.

**Tools granted:** `contents.write` on the target repository, `pull_requests.write` on
the target repository, `projects.write` on the operation project, `actions` logs read.
Secrets available to the *runner* (for pushing) are never passed into the model's
context.

---

## 8. Supervision: allocation and model selection

Invoked at operation start and on each schedule tick (e.g. every 15 min).

**Inputs:** eligible targets by stage, transition and standing, remaining budget,
live `GET /rate_limit` headroom (core + search), runner concurrency ceiling,
per-transition historical cost/time.

**Outputs:** exploration matrix size, concurrent actions for the tick, model map,
pause flag.

**Allocation policy (v1, deliberately simple and explainable):**

```
search_budget      = search_headroom_per_min * tick_minutes
explorations       = min(specification.partition.max_partitions,
                         ceil(estimated_candidates / candidates_per_exploration),
                         search_budget / search_calls_per_exploration)

actions            = min(ready_targets,
                         runner_concurrency_ceiling,
                         budget_remaining / est_cost_per_action,
                         core_headroom / core_calls_per_action)

if budget_remaining <= reserve or core_headroom < floor: pause
```

- Partitions are assigned to explorations; the supervisor grows the partition count
  only while the search quota allows it.
- Actions are dispatched as a matrix; claiming makes over-subscription safe, so the
  supervisor may slightly over-provision to hide startup latency.
- For staged work, matrix entries name the target, transition and specification hash.
  Estimate each eligible transition separately, then choose a deterministic set whose
  summed cost and calls fit the budgets and runner ceiling. A cheap patch upgrade's
  average must not fund an expensive major-version migration. `ready_targets` means
  ready **next transitions**, not all remaining steps.

**Model selection (efficiency-first):**

| Role / phase | Model tier | Rationale |
|---|---|---|
| Elicitor (interrogation) | strong reasoning | rare, high-leverage, operator-facing |
| Supervision (planning) | strong reasoning | one call per tick, small context |
| Exploration — metadata/content filter | none | pure code |
| Exploration — borderline classify | cheap/fast | high volume, narrow judgement |
| Action — triage (does this repository need changes?) | cheap/fast | skips no-op targets before paying for coding |
| Action — change | capable coding | the actual work |
| Action — escalation | stronger coding, once | only after a failed attempt |

Model IDs are configuration, not hardcoded: `models.yaml` maps tiers to concrete
models so they can change without code edits. **Constraint (DECIDED): only models
available through Freebuff may be used**, and the supervision tier must additionally
expose reasoning traces (§13.1) — if none available does, the fallback in §13.1
applies.

Every supervision invocation is **fully logged, including its reasoning traces** — see
§13.1. This is a hard requirement: a plan the operator cannot audit is a plan they
cannot trust.

---

## 9. Reconnaissance (scoping mode)

Purpose: answer "how big is this operation and what will it cost?" **before** the
specification is frozen and **before** any repository is modified. This is the mode to
run first, and to re-run whenever the criteria change.

**Properties**
- Runs the full discovery pipeline (search, hard filters, borderline classification)
  against the current candidate specification, and detects starting stages read-only.
- **Read-only by construction.** The installation token for the reconnaissance is minted
  with read-only scopes: no tracking issues, no targets on the board, no write access to
  target repositories, no branches, no change proposals. Only
  `GET /search/repositories`, metadata reads, and content reads needed for the report
  are permitted.
- Output is a **Feasibility Report**, not a change to the board. It is posted on the
  operation issue and archived as an artifact.
- Safe and cheap to re-run; the report shows deltas against the previous
  reconnaissance.
- The report **leads with the target count** (DECIDED): v1 assumes small operations, so
  the first number the operator must see is how many repositories would be acted on. It
  also states the implied review load, since every change proposal is a draft the
  operator merges. v1 does not pace work to the operator's review capacity; it surfaces
  the load and lets the operator tune criteria and re-run.
- **Staged forecasts count transitions, not just targets.** Report targets by starting
  stage, already-final and undetermined targets, remaining transitions by id, and the
  sum of expected proposals, operator merges and per-transition costs. One 2.12 target
  needs three proposals in the Scala example; one 3.3 target needs one. Targets with
  unknown stages must be reported as unresolved estimates, not priced as zero work.
- **Required before activation (DECIDED).** An operation cannot go `active` without a
  reconnaissance for the current specification hash, unless the operator explicitly
  overrides. This is what enforces "scope before spending" (§10).
- **A forecast, not a reservation (DECIDED).** The real operation re-runs discovery;
  reconnaissance informs only the estimate and the partition/concurrency plan. No
  candidate, probe value, or qualification carries over. The actual-vs-forecast target
  count is reported so drift is visible.

**Feasibility Report**
```yaml
operation: 2026-10-cve-sweep
specification_hash: <sha256>
mode: reconnaissance
targets: 366                    # headline: repositories this operation would act on.
                                # The operator sees review load before approving.
size:
  candidates_seen: 4820
  hard_filtered_out: 4481
  matched: 339
  matched_by_content: 27        # matched only via content criteria, not metadata
  borderline: 27          # default policy applied: include
  estimated_eligible: 366
  estimate_basis: heuristic_v1   # heuristic_v1 | prior_operation | pilot
quota:
  search_calls_used: 96
  search_quota_remaining: 124
  partitions_needed: 6
cost:
  exploration: { model_usd: 4.10, runner_minutes: 12 }
  action:
    est_cost_per_target_usd: 0.62
    est_runner_minutes_per_target: 7
  projected_total:
    targets: 366
    model_usd: 231.0
    runner_minutes: 2574
    wall_clock_hours_at_planned_concurrency: 5.4
review:
  change_proposals_expected: 339   # unstaged: one proposal per target needing a change
  no_change_expected: 27
  operator_actions: 339            # every draft proposal is merged by the operator
confidence: medium
recommendations:
  - "Tightening license_in to MIT-only drops ~40% of the set."
```

**Cost estimation method**
1. **First operation** — heuristic from repository metadata (size, file count, primary
   language, presence of CI, test command discovery) multiplied by configurable
   per-tier token and runner-minute rates, summed over remaining transitions for each
   target. Stage detection and post-merge verification costs are included.
2. **Later operations** — use observed per-action cost and wall-clock from prior session
   logs as the prior; report estimate vs actual so the prior self-corrects.
3. **Optional pilot** — open real change proposals on a small random sample of matched
   repositories, measure, then extrapolate to the full set. The most accurate option,
   and the only one that spends real work, so it is opt-in.

**Feedback into interrogation.** The report lands on the operation issue. The operator
tunes criteria and re-runs the reconnaissance — at zero repository risk — until size and
cost sit inside the envelope, then approves the specification freeze (§10). The frozen
specification records the `estimate_basis` so a later cost overrun can be traced to a bad
estimate vs bad work.

---

## 10. Interrogation protocol (specification elicitation)

Purpose: convert a vague intent into an unambiguous specification the machine can
check.

1. The operator opens an **operation issue** with a free-text intent and an envelope.
2. A workflow runs the Elicitor in a bounded interview loop over the issue comments. It
   asks targeted questions and **challenges** weak answers:
   - "You said 'popular repos' — give a star floor or I'll default to 0."
   - "The task says 'update deps' — which ecosystem and what's the acceptable diff size?"
   - "What commands must pass for a change proposal to be considered successful?"
   - "Which milestones are required, how is each detected, and which task advances
     each adjacent pair? Can dependencies change for these migrations?"
   - "Does completion mean the final stage, and who may waive an unreachable target?"
3. The Elicitor drafts `explorer.yaml` and `actor.yaml`, posts them, and names which
   rubric entries remain unresolved.
4. Loop until all rubric entries pass; then it writes the specification into
   `operations/<id>/` via a change proposal and requests explicit operator **approval** on
   that proposal.
5. Approval merges the specification. The operation id and a specification **hash** are
   frozen; the board `Operation` field and every tracking issue reference them. Changing
   the specification later requires a **new operation**: the new operation re-runs
   interrogation, reconnaissance, and discovery under a new id, and the old operation
   concludes or is abandoned (DECIDED). This is why reconnaissance is required before
   an operation goes `active` (§9).

Rubric (must be fully specified before freeze): scope allow/deny; target sensitivity
(public/non-sensitive only in v1); every criterion machine-checkable; partition
strategy; task instructions with non-goals; acceptance
commands; branch and change-proposal conventions; `max_attempts`; budget ceiling;
definition of done; for staged work, ordered stage predicates, pinned destination
versions, every adjacent transition's task, acceptance and effective bounds, and
the operator-only waiver policy. The operator reviews the **total proposal count**
as well as the target count.

---

## 11. GitHub Actions orchestration

Workflows in the control repo:

| Workflow | Trigger | Duty |
|---|---|---|
| `elicit.yml` | issue comment on an `operation` issue | drive the interrogation loop |
| `start-operation.yml` | `workflow_dispatch` / specification change proposal merged | initialize the board, emit the first plan |
| `explore.yml` | `workflow_dispatch` with a matrix | run explorations |
| `scope.yml` | `workflow_dispatch` (`mode: reconnaissance`) | read-only exploration sweep; posts a feasibility report (§9) |
| `act.yml` | `repository_dispatch` / schedule | claim and process `ready` targets |
| `reconcile.yml` | `schedule` (every 15 min) | supervision tick: plan, retry, unstick, update standings |
| `on-pr-event.yml` | `pull_request` (opened/closed/merged) via App webhook | verify the recorded transition, advance stage or final standing |
| `control.yml` | issue comment `/resume`, `/dismiss`, `/reset`, `/waive <reason>` on a `target` issue | authorise and record an operator decision |

Mechanics:
- **Fan-out:** `strategy.matrix` sized by the supervisor's plan; matrix values passed
  as JSON from the planning job's output.
- **Concurrency:** a `concurrency` group per operation caps live explorations and actions.
  Job-level `concurrency` keyed by repository slug prevents two actions on the same
  target.
- **Claiming:** `Claim`/`ClaimedAt` written through the Projects GraphQL API. Use a
  re-read-after-write check to detect races; the loser exits without side effects.
- **Rate limits:** call `GET /rate_limit` before loops; back off with jitter on 403
  secondary limits; treat search and core budgets separately.
- **Secrets:** GitHub App private key in the control repo's Actions secrets; each job
  mints a short-lived installation token scoped to the repositories it needs. Actions
  push using the token; the token is never placed in model context.
- **Artifacts:** each exploration and action uploads a JSON result artifact (decisions,
  calls, cost, logs) for audit and for the supervisor's next tick. Staged results
  identify from/to stage, transition id and the pinned specification hash.
- **Operator commands:** verify the caller is the operation's operator. A waiver
  requires a non-empty reason, no active claim and no unresolved open proposal.
  Reject invalid or unauthorised commands with an explicit error and leave the
  target unchanged. Never infer a waiver from failed attempts, silence or budget exhaustion.

---

## 12. Security and safety

- **Least privilege:** the GitHub App is granted `contents:write`,
  `pull_requests:write`, `issues:write`, `projects:write`, and `metadata:read` **only**;
  installation tokens are minted per job and expire in ~1 hour.
- **Untrusted code:** actions run target-repo code in an isolated job with no
  org-level secrets, minimal token scope, and no access to the control repo's
  secrets beyond the target repository.
- **Prompt injection:** treat all repository text as data. The action's tool set cannot
  exfiltrate (no arbitrary network tool), and the acceptance commands are taken from
  the **frozen specification**, not from repository files. Model output is constrained
  to structured file-edit operations that are validated before applying.
- **Human gate (DECIDED):** every change proposal is opened as a draft and only the
  operator merges. Autonomy is not expanded until a reviewed pilot justifies it.
- **Public control repo (v1).** The control plane, the board, and the session logs are
  all public. This is why v1 targets only public / non-sensitive repositories: the
  specification's scope refuses a private one, so an operation cannot leak a private
  repository through its board or its traces.
- **Reconnaissance is read-only:** the scope workflow requests a read-only installation
  token, so reconnaissance cannot create issues, targets, branches, or change proposals
  even if the code misbehaves.
- **Budget guardrails:** a hard USD ceiling per operation and per target; the supervisor
  pauses the operation when the reserve is breached.
- **Audit:** every standing transition, stage verification, waiver and model call
  is logged with the specification hash, target and transition identity, session id,
  model, tokens, and cost.

---

## 13. Observability and cost

- **Per-target cost** accumulated in `CostUSD`; **per-operation** rollup in a tracking
  issue comment updated each tick.
- **Metrics:** targets by stage and standing, remaining transitions, achieved/waived/
  unresolved counts, throughput (change proposals/day), success rate per transition
  and attempt, escalation rate, search-quota utilization, mean wall-clock per action.
- **Dashboards:**  a generated Markdown summary section on the operation issue is enough
  for v1; staged Projects views group by `Stage`, with a companion `Standing` view.

### 13.1 Supervision session log

Every supervision invocation writes an append-only **session log**, one record per step,
as JSONL committed to the control repo at `sessions/<operation>/<session-id>.jsonl` (with a
compressed copy as an artifact). A session log records, in order:

- **Reasoning traces verbatim** — the model's full thinking/reasoning output, not just
  its final answer. This is the requirement: the *why*, not only the *what*.
- The exact inputs read: the targets' stages and standings, applicable transitions,
  rate-limit headroom, envelope figures, eligible target counts.
- Prompts sent and raw responses received, per model call.
- Tool calls and their results.
- The emitted plan (matrix size, concurrency target, model map, pause decision) and
  the rule or threshold that produced each number.
- Metadata: specification hash, session id, model id, token counts, cost, wall-clock,
  timestamps; target and transition identity for staged tool calls and results.

**Commit it, don't just artifact it.** Actions artifacts expire (~90 days by default);
committing the JSONL makes the audit trail durable and diffable. Guard repo growth
with per-session files, gzip, and a rollup policy (DECIDED): full traces are kept for
90 days (or the last N operations), then rolled up to a summary plus a trace digest.

**Provider constraint.** Some providers redact or encrypt chain-of-thought. The
supervision model tier therefore has a **reasoning-trace availability requirement**;
if the chosen provider cannot expose traces, the log records the structured plan plus
an explicit rationale field and flags the gap rather than silently omitting it. See §15.

**Privacy.** Traces may contain repository-derived content, and the control repo is
**public** in v1 — so session logs are public too. That is only safe because v1 targets
only public / non-sensitive repositories (§12). Session logs are never fed back into
other agents' model context, regardless.

---

## 14. Repo layout (target)

```
.github/workflows/{elicit,start-operation,explore,act,reconcile,on-pr-event,control}.yml
src/
  control-plane/
    board.ts          # Projects v2 read/write, claim, transitions
    github-app.ts     # token minting, Octokit factories
    rate-limits.ts    # core + search budgets, backoff
    estimate.ts       # feasibility report + cost model (§9)
  agents/
    loop.ts           # minimal agent loop (no framework)
    tools/            # github tools exposed to models
    explorer.ts
    actor.ts
    supervisor.ts
    elicitor.ts
  sessions/           # per-operation supervision session logs (§13.1)
  schemas/            # zod schemas for specifications and tool I/O
operations/<id>/{explorer.yaml,actor.yaml}
models.yaml           # tier -> provider/model map
fixtures/             # synthetic data for the worked example (§17)
README.md             # orientation for contributors
DESIGN.md
CONTEXT.md            # domain glossary
docs/adr/             # architecture decision records
```

---

## 15. Decisions and open questions

Settled across interrogation and the introduction of stages: autonomy (operator
approves the specification and merges every draft change proposal); target scope (own orgs, same-repo
branches, no forks); model catalog (Freebuff-only); board (exactly one project per
operation, which alone manages its standings); already-compliant targets
(`no change needed` at the final goal, which counts as done); task divergence (intent
fixed per transition or unstaged task, bounded
adaptation); qualification (deterministic validator only, content criteria included);
existing change-proposal conflict (`needs operator`); adaptation bounds (new files
allowed, dependency/lockfile edits forbidden unless explicitly permitted by frozen
migration bounds); operation completion (discovery finished, all qualified targets
at the final goal or operator-waived), distinguished from incomplete conclusion;
optional ordered stages, deterministic stage detection, and one proposal per
target/transition with no stacking; failure re-entry
(`failed` stops automatic work, operator `reset` or `waive`); transient failures
(job-level retry, never an `Attempt`); resolving `needs operator` (operator
`resume`/`waive`, `dismiss` only
for unqualified candidates); the
reconnaissance-to-real transition (a forecast, not a reservation — the real operation
re-discovers); content-probe freshness (re-probe at qualification and triage, cache
within a run only); the reconnaissance gate (required before `active`); and
specification revision (a new operation);
cross-operation collision (`needs operator`); language/runtime (TypeScript/Node
with `@octokit/*`); supervision location (a scheduled Actions job); first-operation
estimate (accept `heuristic_v1`); control repo visibility (public, with v1
restricted to public / non-sensitive targets); and session-log retention (full traces
90 days, then rollup).

One fact remains to confirm, not a decision:

1. **Reasoning-trace availability:** which Freebuff model exposes full thinking traces
   for the supervision tier. §13.1 already specifies the fallback, so this does not
   block the design.

**Frontier: empty.** Every branch of the design tree is visited. The remaining work is
the phased plan in §16.

---

## 16. Phased implementation plan

- **Phase 0 — control plane.** GitHub App + Octokit, Projects v2 read/write, a
  `hello-world` operation that creates one tracking issue and moves it through its
  standings; separate stage fields, stage views and transition history.
- **Phase 1 — explorer.** Deterministic search + partition, metadata and content
  criteria, stage detection, create targets, standing `discovered`, plus the read-only reconnaissance
  mode and feasibility report (target count first, then size and cost) built in from
  the start, since it is the safest way to test discovery against real orgs.
- **Phase 2 — elicitor.** Interrogation loop over an operation issue producing the two
  YAML files of the specification and an approval change proposal.
- **Phase 3 — actor.** Claim → existing-proposal check → triage → clone → change →
  verify → change proposal per transition, with final `no change needed` and
  `needs operator` handling (conflicts, forbidden edits), transition-scoped
  idempotency and `max_attempts`, and verified merge advancement.
- **Phase 4 — supervisor.** Dynamic matrix sizing, rate-limit/budget awareness,
  scheduled reconciliation and drift repair, plus calibration of the cost model from
  observed vs estimated results per transition; explicit waivers and completion checks.
- **Phase 5 — hardening.** Prompt-injection tests, sandboxing, cost accounting,
  retry/escalation tuning.

Each phase should be usable on a small synthetic org before scaling to real ones.

---

## 17. Worked example

A single small unstaged operation, end to end. §18 covers staged progress and waivers.
The fixtures under `fixtures/` reproduce this example: the two specification files, a
synthetic org inventory, and the expected feasibility report, board, and session log.

### 17.1 The intent

The operator opens an operation issue on the control repo:

> "Modernise our Scala services. Anything still on 2.13.12 should move to 2.13.14.
> Keep it small — one proposal per repository. Budget: about $50, and I'll review and
> merge every one."

### 17.2 Interrogation

The Elicitor challenges the vague parts — how to *detect* the version, which
repositories count, what must pass — and drafts the two files of the specification:

```yaml
# operations/2026-10-scala-2-upgrade/explorer.yaml
operation: 2026-10-scala-2-upgrade
scope:
  owners: [acme]
  exclude_repos: [acme/dotfiles]
criteria:
  metadata:
    languages: [Scala]
    archived: false
    fork: false
  content:
    - id: scala_version
      match: any
      file: build.sbt
      extract: regex
      pattern: 'scalaVersion\s*:=\s*"([^"]+)"'
      in: ["2.13.12"]          # only services still on the old version
partition:
  strategy: by_owner          # one partition; the org is small
llm_use:
  annotate_only: true
qualification:
  validator: deterministic
```

```yaml
# operations/2026-10-scala-2-upgrade/actor.yaml
task:
  title: "Bump Scala to 2.13.14"
  kind: code_change
  instructions: |
    Change only the Scala version in build.sbt to 2.13.14. Do not touch
    libraryDependencies or plugins. Do not reformat unrelated files.
acceptance:
  build: "sbt compile"
  test: "sbt test"
pr:
  branch: "automation/{{operation}}-{{repo_slug}}"
  draft: true
adaptation:
  intent: fixed
  implementation: within_bounds
  forbidden_changes: [dependencies, lockfiles]
limits:
  max_attempts: 2
  max_files_changed: 3
retry:
  transient: [secondary_rate_limit, server_error, runner_lost, network_timeout]
  max_transient_retries: 3
  backoff_cap_minutes: 5
escalation:
  on_failure: escalate_model
  on_existing_pr: needs_human
```

The operator approves the specification via a change proposal; the operation id and
specification hash freeze, and operation standing goes `specified`.

### 17.3 Reconnaissance

The operator runs `scope.yml` (`mode: reconnaissance`) first. The read-only sweep
searches `acme`, applies metadata filters, then probes `build.sbt` on the survivors
and extracts the Scala version. It creates nothing. The feasibility report leads with
the target count:

```yaml
operation: 2026-10-scala-2-upgrade
specification_hash: 8f3c…
mode: reconnaissance
targets: 12                     # headline: 12 repositories would change
size:
  candidates_seen: 140
  hard_filtered_out: 128
  matched: 12
  matched_by_content: 12        # every match came from the build.sbt probe
  estimate_basis: heuristic_v1
quota:
  search_calls_used: 9
  search_quota_remaining: 141
  partitions_needed: 1
cost:
  exploration: { model_usd: 0.30, runner_minutes: 3 }
  action:
    est_cost_per_target_usd: 0.55
    est_runner_minutes_per_target: 6
  projected_total:
    targets: 12
    model_usd: 6.9
    runner_minutes: 75
    wall_clock_hours_at_planned_concurrency: 0.4
review:
  change_proposals_expected: 11   # one target is expected to need no change
  no_change_expected: 1
  operator_actions: 11            # 12 targets, ~$6.9 — comfortably inside the envelope
confidence: medium
```

The operator is satisfied (12 proposals to review, well under budget) and starts the
operation; standing goes `active`.

### 17.4 The board

`start-operation.yml` creates one Projects v2 project for the operation. Exploration runs
one partition; for each of the 12 matches it creates a tracking issue (the target's
entry on the board) with the exploration's evidence, and the deterministic validator
promotes each from `discovered` to `ready`. The board at tick 0:

| `Target` | `Standing` | `Match` | `Attempts` |
|---|---|---|---|
| `acme/orders-service` | `ready` | build.sbt: 2.13.12 | 0 |
| `acme/billing` | `ready` | build.sbt: 2.13.12 | 0 |
| `acme/legacy-etl` | `ready` | build.sbt: 2.13.12 | 0 |
| `acme/reporting` | `ready` | build.sbt: 2.13.12 | 0 |
| `acme/search` | `ready` | build.sbt: 2.13.12 | 0 |
| … (7 more) | `ready` | … | 0 |

### 17.5 Acting on the targets

At the first supervision tick the allocation policy computes
`actions = min(ready_targets=12, runner_ceiling=20, budget/cost=90, core/…=…) = 12`, and
`act.yml` fans out a matrix of 12 actions. The supervisor writes its session log
(`sessions/2026-10-scala-2-upgrade/<session-id>.jsonl`) recording the inputs, its
reasoning traces, and the rule behind each number.

Each action claims one target and runs its loop. Five representative outcomes:

- **`acme/orders-service` — accepted.** No pre-existing proposal; triage confirms it is
  on 2.13.12; the change edits one line of `build.sbt`; `sbt compile` and `sbt test`
  pass; a **draft** change proposal (#482) is opened and the target moves to
  `proposal open`. The operator reviews and merges it; `on-pr-event.yml` moves the
  target to `accepted`.
- **`acme/billing` — no change needed.** Triage finds `build.sbt` already declares
  2.13.14 (the reconnaissance probe matched an older cached value, or a parallel change
  landed first). No proposal is opened; the target moves straight to `no change needed`,
  which counts as done. Triage cost is still recorded.
- **`acme/legacy-etl` — needs operator.** The bump cannot be made without also editing
  a `libraryDependencies` entry, which `forbidden_changes` forbids. The action stops and
  moves the target to `needs operator` with the reason, rather than widening the diff.
- **`acme/reporting` — needs operator.** An existing change proposal already sits on
  `automation/2026-10-scala-2-upgrade-reporting`, opened by a human. The action does not
  duplicate, adopt, or skip it; it records the conflict and moves the target to
  `needs operator`.
- **`acme/search` — failed.** The bump applies, but `sbt test` fails for unrelated
  reasons. `escalation.on_failure` runs one stronger model; it also cannot go green, so
  after `max_attempts` the target moves to `failed`. Only an operator `reset` can return
  it to `ready` — a fresh attempt budget, with its own branch reused.

The remaining seven targets resolve normally: six reach `accepted` after the operator
merges their proposals, one ends `no change needed`.

### 17.6 Unresolved work

The operation is **not complete** (7 `accepted`, 2 `no change needed`, 2
`needs operator`, 1 `failed`). The two `needs operator` targets
await an operator `resume` or `waive`; the `failed` target awaits an operator `reset`
or `waive`.
Current rollup on the operation issue:

| Standing | Count |
|---|---|
| `accepted` | 7 |
| `no change needed` | 2 |
| `needs operator` | 2 |
| `failed` | 1 |
| **total targets** | **12** |

Actual model cost was $7.4 against the $6.9 estimate and the $50 envelope — comfortably
inside, and the observed per-action cost and wall-clock are now the prior for the next
operation's feasibility report (`estimate_basis: prior_operation`). The two
`needs operator` targets and the `failed` one remain on the board for a follow-up
human decision. If the operator explicitly waives all three with recorded reasons,
the operation can conclude `complete` with 9 achieved and 3 waived. An operator stop
without those decisions concludes `incomplete`, not successful completion.

---

## 18. Worked staged example

The operator wants all qualifying Scala repositories to reach **Scala 3.9**, through
**Scala 2.12 → Scala 2.13 → Scala 3.3 → Scala 3.9** where necessary. These are
illustrative milestones; the frozen specification pins the actual versions and
migration tasks. Each transition may make different code changes, while the goal,
stage order and transition intent remain fixed.

The fixtures in `fixtures/operations/2026-10-scala-staged-upgrade/` define the
specification; `fixtures/2026-10-scala-staged-upgrade/` contains expected outcomes.
Admission criteria find Scala repositories across all stages, including the final
one, rather than matching only the oldest version.

### Entry, actions and review load

| Target | Verified entry stage | Remaining transitions | Expected proposals |
|---|---|---|---|
| `acme/old-service` | Scala 2.12 | 2.12 → 2.13 → 3.3 → 3.9 | 3 |
| `acme/legacy-service` | Scala 2.13 | 2.13 → 3.3 → 3.9 | 2 |
| `acme/modern-service` | Scala 3.3 | 3.3 → 3.9 | 1 |
| `acme/current-service` | Scala 3.9 | none; verify final acceptance | 0 |

Reconnaissance reports **4 targets, 6 remaining transitions and 6 expected proposals**,
including counts and estimates per transition. Actual outcomes may reduce that load;
the report does not reserve work or promise every migration will succeed.

`old-service` starts `Stage=Scala 2.12`, `Standing=ready`. Its 2.13 proposal leaves it
in the Scala 2.12 column with `Standing=proposal open`. Once the operator merges and
default-branch verification passes, the controller records the proposal in history,
sets `Stage=Scala 2.13`, resets the next transition's attempts and returns it to
`ready`. Two more independently merged and verified proposals eventually put it in
Scala 3.9 with `Standing=accepted`. Replayed events for its 2.13 proposal cannot
advance it again or overwrite later work.

`legacy-service` reaches Scala 3.3 through one proposal but cannot migrate a required
library to 3.9. It remains in the Scala 3.3 column as `needs operator` or `failed`.
The operation is not complete yet. The operator may fix/reset the work, or issue
`/waive Required library has no compatible replacement` after resolving any open
proposal and claim. That records `Standing=waived`, the last verified stage and the
operator's reason, rather than pretending it is on 3.9.

`modern-service` needs one proposal. `current-service` is verified against the final
stage predicate and acceptance and becomes `no change needed`.

### Completion and edge cases

With discovery finished, the final board has **3 achieved targets and 1 waived**,
**5 merged proposals**, and no unresolved work. The operation concludes `complete`.
The Scala 3.3 column still contains the waived repository; completion does not claim
every repository literally runs 3.9.

The scenario fixture also covers intermediate external progress, final external
progress after earlier merges, unknown or mixed versions, post-merge verification
failure, duplicate events, transition-scoped attempts, a rejected automatic waiver
and budget exhaustion. Unknown stages need operator attention, not a guessed task.
Closing a proposal without merge leaves its source stage unchanged. Exhausted
attempts or budget alone never satisfy completion.
