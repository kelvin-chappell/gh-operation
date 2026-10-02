# Definition manifest

The approved unit is this complete file inventory. Add operation-specific files here
before review. No creator checkout or live runtime service is required to read it.

During assembly, include and inventory `.devcontainer/devenv.yaml` and
`.devcontainer/shared/devcontainer.json` from the reviewed creator configuration.
Copilot and `gh` are supplied by container creation, not `.tool-versions`.

| File | Purpose |
|---|---|
| [README.md](README.md) | Operator entry point and implementation boundary |
| [AGENTS.md](AGENTS.md) | Shared agent entry point and role routing |
| [OPERATION.md](OPERATION.md) | Intent, criteria, tasks, acceptance and resources |
| [CONTEXT.md](CONTEXT.md) | Domain and operation-specific shared facts |
| [GOVERNANCE.md](GOVERNANCE.md) | Authority, lifecycle, accounting and completion |
| [MANIFEST.md](MANIFEST.md) | Complete definition inventory |
| [.tool-versions](.tool-versions) | mise-managed dev container generator version |
| [Elicitor](.github/agents/elicitor.agent.md) | Definition revision guidance |
| [Supervisor](.github/agents/supervisor.agent.md) | Planning and reconciliation guidance |
| [Explorer](.github/agents/explorer.agent.md) | Discovery and qualification guidance |
| [Actor](.github/agents/actor.agent.md) | One-target action guidance |
| [Reviewer](.github/agents/reviewer.agent.md) | Anthropic adversarial review of OpenAI-authored changes |
| [Operation cycle](.github/skills/operation-cycle/SKILL.md) | Branch-specific operational procedure |
| [Review change](.github/skills/review-change/SKILL.md) | Mandatory cross-provider pre-publication gate |
