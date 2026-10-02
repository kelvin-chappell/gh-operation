# 0010 - Operation repositories are agent context

- **Status:** Accepted
- **Date:** 2026-10-02
- **Decider:** operator, in the final clarification of the preceding session
- **Supersedes:** controller/runtime-base packaging in ADRs 0008/0009 and DESIGN §§3.5, 14, 16, 20

## Decision

**Execution correction:** ADR 0011 supersedes the thin-workflow assumption below.
All work now starts within Copilot sessions in dev containers.

Build operation repositories primarily from Markdown: shared context, operation
definitions, agents and skills, with a thin GitHub workflow. Agents carry out the
operation using existing platform tools. A copied deterministic application is not
the product. Add small deterministic helpers only where there is a demonstrated
need; prefer existing commands and platform mechanisms.

The Creator interrogates, assembles and reviews this context before approved
materialisation. It has no continuing runtime ownership. A generated repository
contains one operation and remains independently inspectable and inactive at handover.

## Retained contracts and limits

This changes packaging, not authority or evidence requirements. Human approval and
activation, deterministic qualification/acceptance, exclusive target claims,
visibility controls, AI-credit accounting, staged progress and explicit completion
remain requirements. Instructions describe them but cannot prove their enforcement.

Use platform permissions, protected resources and independently verifiable checks
where possible. If an essential guarantee cannot be established without additional
machinery, report it as a blocker and review the smallest necessary mechanism.
Do not silently weaken a guarantee or regenerate the rejected application.

## Implementation

The base is `templates/operation/`: local Markdown guidance and a devenv generator
pin. The Creator and preparation skill are available in `.github/`. Preparation
includes reviewed dev container configuration and performs structural checks within
the Copilot session, without a Node dependency or application build.

There is no automated approval capture, materialiser or implemented target execution
yet. The redundant TypeScript application scaffolding, Node test harness, dependency
manifests and Node tool pins have been removed at the operator's request. No target
execution or GitHub resource creation is enabled by this change.
