# Operation Runner

A system of **elicitor**, **supervisor**, **explorer**, and **actor** agents that carry
one kind of work across many repositories. An operator states an intent; the system
clarifies it by structured interrogation, discovers the repositories that belong to the
operation, and performs the same task on each — one change proposal per repository.

**Status: design complete, ready for implementation.** There is no code yet; this repo
is the design, its glossary, its decisions, and the fixtures for its first test.

---

## Start here

| Document | What it is |
|---|---|
| [`DESIGN.md`](DESIGN.md) | The full design: components, lifecycle, data model, security, orchestration |
| [`CONTEXT.md`](CONTEXT.md) | The **domain glossary** — read this first; the design is written in its language |
| [`docs/adr/`](docs/adr/README.md) | The load-bearing decisions and why they were made |
| [`fixtures/`](fixtures/README.md) | A worked operation from intent to conclusion, as test data |

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
  **Action** performs the **task** on a target and produces a **change proposal**.
- **Reconnaissance** is exploration with no action — the read-only sweep that produces
  a **feasibility report** before anything is spent.

---

## How an operation runs

```
intent ─▶ interrogation ─▶ specification ─▶ reconnaissance ─▶ exploration
                                                                    │
                            change proposal ◀─ action ◀─ target ◀──┘
                                    │
                              operator merges ─▶ accepted
```

1. **Interrogation** turns a vague intent into a frozen specification; the operator
   approves it.
2. **Reconnaissance** measures the size and cost, read-only. The feasibility report
   leads with the target count.
3. **Exploration** discovers repositories and qualifies them onto the operation's board.
4. **Action** claims each target, triages, makes the change, and opens one draft change
   proposal — or records `no change needed`.
5. The **operator** merges every proposal. The operation concludes when every target has a
   disposition.

Each target's **standing** (`ready`, `in action`, `proposal open`, `accepted`,
`no change needed`, `needs operator`, `failed`, `excluded`, …) is tracked on the board.

---

## Decisions

The hard-to-reverse choices are recorded as ADRs. In brief:

- **One board per operation**, a GitHub Projects v2 project; targets are tracking issues
  ([0001](docs/adr/0001-board-is-github-projects-v2.md)).
- **Qualification is deterministic**; the model records evidence but never promotes
  ([0002](docs/adr/0002-qualification-is-deterministic.md)).
- **One change proposal per target**, draft, merged only by the operator
  ([0003](docs/adr/0003-one-change-proposal-per-target.md)).
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
