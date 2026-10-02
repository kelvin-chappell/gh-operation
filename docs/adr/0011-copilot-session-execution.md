# 0011 - Work starts in Copilot sessions inside dev containers

- **Status:** Accepted
- **Date:** 2026-10-02
- **Decider:** operator
- **Supersedes:** ADR 0006 and workflow-trigger/tool-pinning assumptions in ADRs 0008-0010

## Decision

Preparation and operation processes are initiated only within GitHub Copilot
sessions in dev containers. GitHub Actions workflows, schedules, pushes and proposal
events do not initiate these processes. Agents inspect and reconcile GitHub evidence
from their sessions.

Dev container creation supplies Copilot and `gh` through the `github-copilot`
module. Remove their explicit `.tool-versions` declarations. Retain the devenv
generator pin and record actual session tool/model versions as execution evidence.
Generated operations include their own reviewed dev container configuration.

## Consequences

Remove the template's structural workflow. Perform definition checks during
preparation within the Copilot session. The Creator has no scheduler, and an
operation resumes by reading durable records in a subsequent session rather than
waiting for a workflow tick.

Opening a session is not activation. Human authority, approved definitions,
readiness, claims, privacy, credit accounting and completion requirements remain.
Container-provided tools do not by themselves prove isolation or readiness.
