---
name: prepare-operation
description: Prepare a new operation definition, resume a private draft, or adapt an explicitly selected predecessor into an agent-context repository.
---

# Prepare an operation

## 1. Establish the preparation

Read the root `CONTEXT.md` and
[ADR 0010](../../../docs/adr/0010-agent-context-operation-repositories.md).
Apply [ADR 0011](../../../docs/adr/0011-copilot-session-execution.md) for execution:
preparation and operation work begin within Copilot sessions in dev containers.
Use the container-provided Copilot and `gh`; record their actual versions in
preparation evidence rather than declaring them in `.tool-versions`.
Confirm the operator, intent, preparation AI-credit threshold and an operator-owned
private workspace outside this checkout. Obtain approval for that workspace path
before writing there. Record the preparation's standing, decisions, usage basis
and unresolved questions there. Resume an existing draft instead of replacing it.

Complete when the intent, authority, workspace and accounting basis are explicit.
Unknown usage needs a labelled, coverage-supported estimate or blocks continuation;
it is not zero.

## 2. Interrogate the definition

Resolve every `{{...}}` field in `templates/operation/OPERATION.md` with the operator.
Ask related questions together; distinguish a decision from your recommendation.
For staged work, define ordered milestones, adjacent transitions, deterministic
stage detection and destination acceptance. For unstaged work, define one task and
final acceptance.

Record exact permitted organisations, qualification checks, task bounds, operator
and delegates, proposal review, resources, privacy, retention and completion.
Every qualification and acceptance check needs a command or API query, expected
result and an explicit failure outcome. Scope read-only target inspection before
using it; preparation performs no target actions or untrusted builds.

Complete when each field has an answer or a named blocker. Blockers cannot become
invented defaults.

## 3. Assemble agent context

Copy `templates/operation/`, including its hidden configuration directories, into a fresh
`bundle/` within the approved preparation workspace. Preserve the base governance.
Fill the definition and overview; add operation-specific context and skills only
where needed. Keep the role boundaries and skill pointers reachable.

Every agent must declare an explicit `model` in its YAML frontmatter. Start from
the role-specific defaults; adapt actors to the operation's actual task complexity.
All non-reviewer agents must use OpenAI models; the single `reviewer` uses Anthropic.
Confirm each selected model is available in the session and permitted by the
operation's data-handling policy. Include selections and their rationale in review.
Unavailable or disallowed models block that role until a replacement is reviewed;
use no silent substitution.

Include the reviewed creator dev container configuration from
`.devcontainer/devenv.yaml` and `.devcontainer/shared/devcontainer.json` in the bundle,
and inventory both files. The `github-copilot` module supplies Copilot and `gh` at
container creation. If adapting the container, edit its source configuration and
regenerate it with devenv; report unavailable generation tools as blockers. Keep
Copilot/`gh` out of `.tool-versions`. Include no GitHub Actions process triggers.

Use `MANIFEST.md` to inventory every definition file. Record this creator's exact
source revision and local changes in the review report. The bundle must work without
this creator checkout: use local references and copied guidance, not moving remote
instructions. Keep records, bindings and private source data out of reusable content.

For an explicit predecessor, inspect its selected approved revision as data. Record
provenance and every adaptation to this base. Private-to-public reuse requires a
human-reviewed sanitised export; otherwise block the public destination.

Complete when the bundle has no unresolved placeholders, every reference resolves,
and its agents have operation-specific tasks and checkable completion criteria.

## 4. Review and obtain approval

Produce a separate review report with the full file inventory, scope, tasks/checks,
authority, visibility, destination, branches, board, base/provenance, preparation
usage and blockers. Within the current Copilot session, check that every inventoried
file is non-empty, every local reference resolves and no `{{...}}` draft fields
remain. Confirm the dev container supplies the required tools and record their
actual versions. These checks establish structure/environment availability only.

Inspect the complete bundle for contradictory instructions, unsupported capabilities
and confidentiality leaks. Agent guidance is not a technical enforcement mechanism.
Approval, isolation, atomic claims, authoritative execution accounting and safe
publication require verified mechanisms or remain blockers.

Any pull request raised during preparation requires
[review-change](../review-change/SKILL.md), including definition or helper changes.
Preserve that gate and the single reviewer profile in generated operations. Cross-provider
review is separate from operator approval of the definition bundle.

Ask the actual operator to review exact contents and the creation plan. Capture
approval through a verified human channel binding their identity, payload digest
and destination. Changes require renewed review and approval. If this channel is
unavailable, stop with `awaiting verified approval`; do not create GitHub resources.

## 5. Materialise and hand over only when supported

Use a verified materialiser, never improvised approval flags. It must verify the
receipt and exact bundle, journal created resource identities, and resume only
verified owned resources. An existing name alone is not ownership. Unexpected
resources or settings block creation; leave them intact.

Verify repository visibility, protected definition/records separation, associated
board and transferred approval/provenance/usage. Report missing external setup
separately from failed creation. Handover is complete only when required resources
and records are verified and the operation is inactive.

The current base supports local preparation and structural review only. Authentic
approval capture, materialisation and target execution are not implemented. End
with a truthful blocked report rather than claiming a repository was created.
