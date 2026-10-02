# Operation definition

## Identity and authority

- Operation identity: {{operation_id}}
- Operator's intent: {{intent}}
- Operator and explicitly authorised delegates, with decision scope: {{authority}}
- Approved definition revision and approval evidence: {{approval_evidence}}
- Definition and records branches, repository visibility and board: {{resources}}

An inactive draft may name approval evidence as pending, with a blocker. Pending
approval is not an approved execution revision.

## Scope and qualification

- Permitted organisations and repositories: {{target_scope}}
- Exclusions: {{exclusions}}
- Metadata/content criteria: {{criteria}}
- Deterministic commands/API queries, expected results and failure handling:
  {{qualification_checks}}
- Evidence minimisation and access requirements: {{evidence_policy}}

Model interpretation may annotate evidence. Only successful deterministic criteria
checks qualify a candidate.

## Goal, task and acceptance

- Final goal: {{final_goal}}
- Unstaged task, or ordered stages and adjacent transition tasks: {{tasks}}
- Deterministic entry-stage detection, or explicitly unstaged: {{stage_detection}}
- Task generations and what constitutes a material change: {{task_generations}}
- Per-task/transition acceptance commands, expected results and failure outcomes:
  {{acceptance_checks}}
- Final default-branch acceptance: {{final_acceptance}}
- Allowed paths, changes and publication destinations: {{change_bounds}}
- Proposal review and human merge policy: {{review_policy}}

## Resources and working constraints

- Execution AI-credit dispatch threshold and reservation policy: {{credit_envelope}}
- Concurrency and wall-clock limits: {{execution_limits}}
- Substantive attempt limit and transient retry bounds: {{retry_policy}}
- Required container capabilities, approved models and data-handling evidence:
  {{model_policy}}
- Record/artifact visibility, access and retention: {{retention_policy}}
- Operation-specific shared context and skills: {{additional_context}}

## Readiness and activation

- Verified approval, configuration isolation, target claims, credit measurement,
  acceptance and publication mechanisms, with evidence or named blockers:
  {{readiness_evidence}}
- Reconnaissance results and freshness: {{reconnaissance}}
- Authenticated activation decision, or explicitly not activated: {{activation}}

Every required mechanism needs verification before target execution. Starting a
Copilot session or passing structural review is not that verification. Record actual
Copilot/`gh` versions from the dev container with execution evidence.

## Completion and handover

- Discovery/qualification exhaustion checks: {{discovery_completion}}
- Final-goal and waiver verification procedure: {{completion_checks}}
- Pause, emergency stop, recovery and retirement procedure: {{stop_policy}}
- Handover resources, provenance, preparation usage and setup blockers: {{handover}}

Apply the lifecycle and authority rules in [GOVERNANCE.md](GOVERNANCE.md). Incomplete
work remains explicit when resources are exhausted.
