---
name: prepare-operation
description: Prepare a new operation definition, resume a private draft, or adapt an explicitly selected predecessor into an agent-context repository.
---

# Prepare an operation

The deliverable is a complete operation repository, produced in this session, that
supports the whole operation in one subsequent coordinating Copilot session.
Preparation does not run the
operation. Account/scope answers define future discovery, not permission to perform
it now. Live target enumeration or inspection requires separate explicit
authorisation for preparation evidence; reconnaissance, builds, migration and
proposal publication remain operational activities. Apply this boundary to delegated
agents too. Record unknown target facts as pending decisions or execution-time checks.

## Default: quick assembly

Assemble supplied facts onto `templates/operation/` and produce a complete inactive
repository. Use existing context and readily available authoritative references only
where a fact is needed to write the files. Use focused grilling to settle essential
decisions quickly; avoid speculative research, repetitive questions and additional
machinery.

Ask focused questions for missing decisions needed to make the operation operable.
Budgets, versions, resource bindings and approval may be pending during drafting,
but essential gaps must be resolved before calling the result complete. Repository-
specific facts legitimately learned during execution need explicit discovery
procedures, not a precomputed target inventory.

Perform assembly, review and operator approval within the creation session. Check
that the next session can start, discover, act, review, record, reconcile and finish
using the supplied guidance and available tools. Local draft files are an intermediate
result. Publishing GitHub resources requires approval of their exact destination;
activation and target execution belong exclusively to the generated repository.

## 1. Establish the preparation

Read the root `CONTEXT.md` and
[ADR 0010](../../../docs/adr/0010-agent-context-operation-repositories.md).
Apply [ADR 0011](../../../docs/adr/0011-copilot-session-execution.md) for execution:
preparation and operation work begin within Copilot sessions in dev containers.
Use the container-provided Copilot and `gh`; record their actual versions in
preparation evidence rather than declaring them in `.tool-versions`.
Record the supplied operator, intent and preparation AI-credit threshold; mark missing
values pending for a draft. Confirm an operator-owned workspace outside this checkout.
Obtain approval for that workspace path
before writing there. Record the preparation's standing, decisions, usage basis
and unresolved questions there. Resume an existing draft instead of replacing it.

Complete when the intent and workspace permit assembly and unknown authority/accounting
facts are explicitly pending. Never label unknown usage as zero. Formal budget
continuation decisions require measured usage or a labelled, coverage-supported estimate.

## 2. Interrogate the definition

Populate `templates/operation/OPERATION.md` from supplied facts and actively guide
the operator through the decisions required to complete it:

1. Map unresolved decisions and their dependencies. Use answers already supplied;
   look up readily available facts yourself within the preparation boundary.
2. Ask a focused round of pointed, numbered questions whose prerequisites are
   settled. Give a recommended answer and its consequence for each. Batch related
   independent decisions; defer questions that depend on unanswered ones.
3. Grill vague, conflicting or unworkable answers with concrete examples and
   trade-offs. Distinguish the operator's decision from your recommendation and
   confirm consequential choices rather than silently treating suggestions as consent.
4. Record answers directly in the definition and recompute the next question round.
   Continue guiding until all essential branches are settled; pending fields are
   temporary drafting aids, not the end of interrogation.
5. Present the resulting intent, scope, tasks, acceptance, limits and completion
   criteria concisely, and obtain confirmation of shared understanding before final
   approval. If the operator pauses or declines a necessary decision, preserve the
   draft and state the specific blocker rather than inventing an answer.

For staged work, define ordered milestones, adjacent transitions, deterministic
stage detection and destination acceptance. For unstaged work, define one task and
final acceptance.

Record exact permitted organisations, qualification checks, task bounds, operator
and delegates, proposal review, resources, privacy, retention and completion.
Every qualification and acceptance check needs a command or API query, expected
result and an explicit failure outcome. Scope read-only target inspection before
using it; preparation performs no target actions or untrusted builds.

Complete when essential definition decisions are answered and confirmed, and
execution-time facts have explicit discovery/check procedures. Genuine unresolved
decisions mean interrogation is incomplete, not that placeholder assembly succeeded.

## 3. Assemble agent context

Copy `templates/operation/`, including its hidden configuration directories, into a fresh
`bundle/` within the approved preparation workspace. Preserve the base governance.
Fill the definition and overview; add operation-specific context and skills only
where needed. Keep the role boundaries and skill pointers reachable.

Every agent must declare an explicit `model` in its YAML frontmatter. Start from
the role-specific defaults; adapt actors to the operation's actual task complexity.
All non-reviewer agents must use OpenAI models; the single `reviewer` uses Anthropic.
Record model availability and data-handling approval as pending until verified.
Before invoking those agents, confirm the model is available in the session and
permitted by policy. Include selections and their rationale in review.
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

Complete assembly when every local reference resolves and agents have operation-
specific tasks and checkable completion criteria. Describe how one supervising
session invokes the roles and advances checkpoints until completion. Include usable
record, claim, accounting and publication procedures rather than references to
unimplemented services. Review remaining gaps before final handover.

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

## 5. Hand over a complete repository

Use available session/platform tools to materialise only the approved contents and
destination. Record genuine operator approval and created resource identities.
Resume only verified owned resources; an existing name alone is not ownership.
Unexpected resources/settings block creation and remain intact. A separate custom
materialiser application is not a prerequisite; missing required capabilities are
blockers, not hypothetical tools to cite as if they exist.

Verify repository visibility, protected definition/records separation, associated
board and transferred approval/provenance/usage. Report missing external setup
separately from failed creation. Handover is complete only when required resources
and records are verified and the operation is inactive.

Handover names the repository location and entry point for the next session, approved
goal/bounds, supplied procedures and any external prerequisites. The operator must
not need another creator session or missing implementation to carry out the operation.
If essential gaps remain, label the result incomplete and state them plainly.
Single-session completion never permits bypassing human review or claiming unfinished
work succeeded.
