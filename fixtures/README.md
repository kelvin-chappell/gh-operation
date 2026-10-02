# Fixtures - worked examples (`DESIGN.md` §17 and §18)

Synthetic data for the operation `2026-10-scala-2-upgrade`. Nothing here touches a real
repository; a test harness replaces the GitHub API and the content probes with the
inventory in `2026-10-scala-2-upgrade/repositories.yaml`.

These are partial definition and outcome fixtures, not complete runnable operation
repositories. `operations/<id>/` groups test examples here; an actual operation
repository has one root manifest and `specification/`, not a multi-operation directory.
Execution hashes, generation/claim ids, model names and credit values are synthetic.
Cost is **AI credits only**; the values are not converted historical currency amounts.
Each board declares its one-revision identity, shared by its entries.

## Why a *reduced* org

`DESIGN.md` §17 illustrates a realistic scan (140 candidates → 12 targets). This fixture
uses a **20-repository synthetic org** so it stays readable and runs instantly. It
reproduces the same **12 targets and standings** as §17;
Scan counts and exploration forecasts differ. Total projected credits and runner
minutes include exploration as well as action work.

## Files

| File | What it is |
|---|---|
| `operations/2026-10-scala-2-upgrade/explorer.yaml` | the exploration specification (criteria) |
| `operations/2026-10-scala-2-upgrade/actor.yaml` | the action specification (task, acceptance, bounds) |
| `2026-10-scala-2-upgrade/repositories.yaml` | the synthetic org inventory: metadata + `build.sbt` content |
| `2026-10-scala-2-upgrade/expected-feasibility-report.yaml` | expected reconnaissance output |
| `2026-10-scala-2-upgrade/expected-board.yaml` | expected board: 12 targets, including 3 unresolved; not complete |
| `2026-10-scala-2-upgrade/expected-session-log.jsonl` | expected supervision session-log shape (`DESIGN.md` §13.1) |

## What a test should assert

1. **Discovery + qualification.** Feeding `repositories.yaml` through the deterministic
   validator yields exactly the 12 targets in `expected-board.yaml` — the 8 near-miss
   repositories are filtered out (each for the reason recorded in its entry).
2. **Reconnaissance.** The feasibility report matches `expected-feasibility-report.yaml`,
   and **leads with the target count**.
3. **Action outcomes.** The standings match `expected-board.yaml`:
   7 `accepted`, 2 `no change needed`, 2 `needs operator`, 1 `failed`. The operation
   remains incomplete until the three unresolved targets succeed or are explicitly waived.
4. **The edge cases, specifically.** `legacy-etl` stops at `needs operator` on a forbidden
   dependency edit; `reporting` stops at `needs operator` on a foreign change proposal;
   `search` reaches `failed` after escalation; `billing`/`scheduler` land on
   `no change needed` because of drift, not because the criteria changed.
5. **Logging.** A supervision invocation produces records shaped like
   `expected-session-log.jsonl`, including explicit `rationale`, execution identity,
   target-local claim acquisition, reservations and trusted AI-credit usage.

## Staged Scala upgrade

`DESIGN.md` §18 uses four targets starting at different milestones. Version values
are illustrative test data, not assertions about available Scala releases.

| File | What it is |
|---|---|
| `operations/2026-10-scala-staged-upgrade/explorer.yaml` | admission across all stages, not just the oldest version |
| `operations/2026-10-scala-staged-upgrade/actor.yaml` | stage detection and all three transition tasks |
| `2026-10-scala-staged-upgrade/scenarios.json` | starting versions, action events and expected progress, including failure cases |
| `2026-10-scala-staged-upgrade/expected-feasibility-report.yaml` | 4 targets, 6 remaining transitions and 6 expected proposals |
| `2026-10-scala-staged-upgrade/expected-board.yaml` | complete with 3 achieved, 1 explicitly waived and 5 merged proposals |

A harness should detect exactly one stage from all probed versions, choose only
the next transition, retain the achieved stage while a proposal is open, and advance
only on verified default-branch progress. Check that each transition has a distinct
branch/proposal identity including its task generation, retries reuse it, and
unchanged-task retries, measured credits and history survive execution revisions.

The scenario cases cover starting midway or already final, externally completed
steps, unrecognised and mixed versions, intermediate merges, duplicate/late events,
closed-unmerged proposals, verification failure, waiver authority, claim/proposal
guards and completion with unresolved or undiscovered work. A waiver counts towards
completion but never changes the achieved stage. Failed work and exhausted resources
must not become success-shaped outcomes.

## Operation repository lifecycle

`operation-repository/scenarios.json` records contract cases for creation versus
activation, readiness/privacy gates, revision handover, task generations, waiver
reaffirmation, credit thresholds and safe claim release. They specify future runtime
expectations; they do not prove Copilot isolation/metering or live GitHub atomicity.

## Creator preparation

`creator/scenarios.json` records exact approval binding, labelled preparation
estimates, actual-usage corrections, resource ownership on retries and inactive
handover. These are declarative contract cases, not materialiser API arguments:
approval/ownership flags represent facts verified by trusted components, not
booleans accepted from a model.

Preparation may continue on meaningful labelled estimates. The operation lifecycle
fixture still rejects missing authoritative operational usage; the two accounting
policies must not be conflated. A created repository with explicit external setup
blockers can be handed over inactive, but missing required creation resources or
untransferred approval/provenance is not a completed handover.
