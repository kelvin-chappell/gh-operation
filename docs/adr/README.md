# Architecture Decision Records

The operation repository boundary is recorded in ADR 0008 and the creator architecture
in ADR 0009. Earlier records retain their history and identify superseded choices.

| ADR | Decision |
|---|---|
| [0001](0001-board-is-github-projects-v2.md) | One associated Projects v2 board and operation-owned tracking issue per target |
| [0002](0002-qualification-is-deterministic.md) | Qualification is deterministic; model annotations never promote candidates |
| [0003](0003-one-change-proposal-per-target.md) | Original proposal packaging, superseded by 0007/0008; draft/human merge gates retained |
| [0004](0004-public-control-repo-public-only-targets.md) | Original public-only restriction, superseded by 0008 |
| [0005](0005-custom-agent-harness.md) | Original custom model loop, superseded by Copilot CLI in 0008 |
| [0006](0006-github-actions-runtime.md) | Each operation runs GitHub Actions with pinned Copilot CLI |
| [0007](0007-stages-and-completion.md) | Stage/standing separation, transitions and explicit final-goal/waiver completion |
| [0008](0008-one-repository-per-operation.md) | Independent operation repositories, approved revisions, private-data controls, atomic claims and AI-credit accounting |
| [0009](0009-copilot-driven-operation-creator.md) | Copilot-driven preparation, tested base, authentic approval, journalled creation and inactive handover |
