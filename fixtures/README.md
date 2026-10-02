# Fixtures - worked examples (`DESIGN.md` §17 and §18)

Synthetic data for the operation `2026-10-scala-2-upgrade`. Nothing here touches a real
repository; a test harness replaces the GitHub API and the content probes with the
inventory in `2026-10-scala-2-upgrade/repositories.yaml`.

## Why a *reduced* org

`DESIGN.md` §17 illustrates a realistic scan (140 candidates → 12 targets). This fixture
uses a **20-repository synthetic org** so it stays readable and runs instantly. It
reproduces the same **12 targets and standings** as §17;
only the scan counts differ.

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
   `expected-session-log.jsonl`, including a verbatim `reasoning_trace`.

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
branch/proposal identity, retries reuse it, attempts reset only for a new transition
or operator reset, and costs and history never reset.

The scenario cases cover starting midway or already final, externally completed
steps, unrecognised and mixed versions, intermediate merges, duplicate/late events,
closed-unmerged proposals, verification failure, waiver authority, claim/proposal
guards and completion with unresolved or undiscovered work. A waiver counts towards
completion but never changes the achieved stage. Failed work and exhausted resources
must not become success-shaped outcomes.
