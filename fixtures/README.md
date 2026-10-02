# Fixtures — worked example (`DESIGN.md` §17)

Synthetic data for the operation `2026-10-scala-2-upgrade`. Nothing here touches a real
repository; a test harness replaces the GitHub API and the content probes with the
inventory in `2026-10-scala-2-upgrade/repositories.yaml`.

## Why a *reduced* org

`DESIGN.md` §17 illustrates a realistic scan (140 candidates → 12 targets). This fixture
uses a **20-repository synthetic org** so it stays readable and runs instantly. It
reproduces **every outcome branch** and the same **12 targets and dispositions** as §17;
only the scan counts differ.

## Files

| File | What it is |
|---|---|
| `operations/2026-10-scala-2-upgrade/explorer.yaml` | the exploration specification (criteria) |
| `operations/2026-10-scala-2-upgrade/actor.yaml` | the action specification (task, acceptance, bounds) |
| `2026-10-scala-2-upgrade/repositories.yaml` | the synthetic org inventory: metadata + `build.sbt` content |
| `2026-10-scala-2-upgrade/expected-feasibility-report.yaml` | expected reconnaissance output |
| `2026-10-scala-2-upgrade/expected-board.yaml` | expected final board: 12 targets and their dispositions |
| `2026-10-scala-2-upgrade/expected-session-log.jsonl` | expected supervision session-log shape (`DESIGN.md` §13.1) |

## What a test should assert

1. **Discovery + qualification.** Feeding `repositories.yaml` through the deterministic
   validator yields exactly the 12 targets in `expected-board.yaml` — the 8 near-miss
   repositories are filtered out (each for the reason recorded in its entry).
2. **Reconnaissance.** The feasibility report matches `expected-feasibility-report.yaml`,
   and **leads with the target count**.
3. **Action outcomes.** The final standings and dispositions match `expected-board.yaml`:
   7 `accepted`, 2 `no change needed`, 2 `needs operator`, 1 `failed`.
4. **The edge cases, specifically.** `legacy-etl` stops at `needs operator` on a forbidden
   dependency edit; `reporting` stops at `needs operator` on a foreign change proposal;
   `search` reaches `failed` after escalation; `billing`/`scheduler` land on
   `no change needed` because of drift, not because the criteria changed.
5. **Logging.** A supervision invocation produces records shaped like
   `expected-session-log.jsonl`, including a verbatim `reasoning_trace`.
