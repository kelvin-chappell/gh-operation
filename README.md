# Operation Runner

A system of **elicitor**, **supervisor**, **explorer**, and **actor** agents that carry
one kind of work across many repositories. An operator states an intent; the system
clarifies it by structured interrogation, discovers the repositories that belong to the
operation, and advances each towards its goal. An operation may have ordered stages,
with a separate task and change proposal for each transition.

**Status: design complete, ready for implementation.** There is no code yet; this repo
is the design, its glossary, its decisions, and the fixtures for its first test.

---

## Start here

| Document | What it is |
|---|---|
| [`DESIGN.md`](DESIGN.md) | The full design: components, lifecycle, data model, security, orchestration |
| [`CONTEXT.md`](CONTEXT.md) | The **domain glossary** — read this first; the design is written in its language |
| [`docs/adr/`](docs/adr/README.md) | The load-bearing decisions and why they were made |
| [`fixtures/`](fixtures/README.md) | Unstaged and staged operations, as test data |

---

## The language

The design uses a small, deliberate vocabulary; `CONTEXT.md` is the authority. The
short version:

- An **operation** is one bounded undertaking: one intent, one frozen **specification**,
  one **board**, one **envelope**. It is the unit of work and of accounting.
- Four agent roles carry it out. **Agent** is the umbrella; the roles are
  **Elicitor** (interrogates the operator into a specification), **Supervisor**
  (allocates — how much exploration and action, when to pause, when the operation is
  done), **Explorer** (finds repositories), and **Actor** (does the work).
- **Exploration** finds **candidates**; **qualification** promotes them to **targets**.
  A **stage** is a verified milestone, such as Scala 2.13. A **transition** prescribes
  the task and acceptance for moving to the next stage. **Action** performs that task
  on one target and may produce a **change proposal**.
- **Stage** describes achieved progress; **standing** describes the work's lifecycle.
  Staged boards group by operation-defined stage names, without losing lifecycle
  standings. Unstaged operations retain their single task.
- **Reconnaissance** is exploration with no action — the read-only sweep that produces
  a **feasibility report** before anything is spent.

---

## How an operation runs

```
intent ─▶ interrogation ─▶ specification ─▶ reconnaissance ─▶ exploration
                                                                    │
                            change proposal ◀─ action ◀─ target ◀──┘
                                    │
                       operator merges ─▶ verify stage ─▶ next action
                                               │
                                          final goal ─▶ accepted
```

1. **Interrogation** turns a vague intent into a frozen specification; the operator
   approves it.
2. **Reconnaissance** measures the size and cost, read-only. The feasibility report
   leads with the target count.
3. **Exploration** discovers repositories and qualifies them onto the operation's board.
4. **Action** claims a target, verifies its stage, selects the frozen transition task,
   and opens one draft proposal. Each transition has its own proposal; only one may
   be open per repository at a time.
5. The **operator** merges every proposal. Verification on the default branch advances
   the target to the next stage, then the next action, until the final goal is reached.
6. The operation is **complete** only after discovery has finished and every qualified
   target has reached the final goal or been explicitly **waived** by the operator.
   `failed` and `needs operator` do not count as completion.

Each target's **standing** (`ready`, `in action`, `proposal open`, `accepted`,
`no change needed`, `needs operator`, `failed`, `waived`, `excluded`, …) is tracked
separately from its stage.

For example, a Scala 2.12 target follows **2.12 → 2.13 → 3.3 → 3.9**, with three
separate proposals. A target already on 3.3 needs only the final transition. A waived
target remains visible at its last achieved stage, with the operator's reason.

---

## Decisions

The hard-to-reverse choices are recorded as ADRs. In brief:

- **One board per operation**, a GitHub Projects v2 project; targets are tracking issues
  ([0001](docs/adr/0001-board-is-github-projects-v2.md)).
- **Qualification is deterministic**; the model records evidence but never promotes
  ([0002](docs/adr/0002-qualification-is-deterministic.md)).
- **One change proposal per target and transition**, draft, merged only by the
  operator; stages are separate from standings and completion requires the final
  goal or a waiver ([0007](docs/adr/0007-stages-and-completion.md), superseding the
  packaging rule in [0003](docs/adr/0003-one-change-proposal-per-target.md)).
- **The control repo is public**, so v1 targets only public/non-sensitive repositories
  ([0004](docs/adr/0004-public-control-repo-public-only-targets.md)).
- **A custom agent harness**, not a framework
  ([0005](docs/adr/0005-custom-agent-harness.md)).
- **GitHub Actions** is the runtime
  ([0006](docs/adr/0006-github-actions-runtime.md)).

---

## What's next

`DESIGN.md` §16 lays out the phased plan. **Phase 0** is the control plane: GitHub App
auth, Projects v2 read/write, and a `hello-world` operation that moves one target through
its standings. Each phase should run against a small synthetic org first — the fixtures
in `fixtures/` are the starting point.
