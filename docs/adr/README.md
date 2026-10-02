# Architecture Decision Records

Load-bearing decisions for Operation Runner. Format: Status / Context / Decision /
Consequences. Vocabulary follows `CONTEXT.md`; the design itself is `DESIGN.md`.

| ADR | Decision |
|---|---|
| [0001](0001-board-is-github-projects-v2.md) | The board is a GitHub Projects v2 project, one per operation |
| [0002](0002-qualification-is-deterministic.md) | Qualification is deterministic; the model records evidence but never promotes |
| [0003](0003-one-change-proposal-per-target.md) | Original one-proposal-per-target packaging, superseded by 0007; human merge gate retained |
| [0004](0004-public-control-repo-public-only-targets.md) | Public control repo, with v1 restricted to public/non-sensitive targets |
| [0005](0005-custom-agent-harness.md) | A custom agent harness, not an agent framework |
| [0006](0006-github-actions-runtime.md) | The runtime is GitHub Actions, with supervision as a scheduled job |
| [0007](0007-stages-and-completion.md) | Optional ordered stages, one proposal per transition, verified final goal or explicit waiver for completion |
