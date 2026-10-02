# Creator workspace

Work within Copilot sessions in a dev container. Follow the product purpose in
[CONTEXT.md](CONTEXT.md): one session here generates a complete operation repository;
one session there can start and carry out the whole operation. This repository
prepares the complete context and procedures; generated repositories own execution.

When asked to create or prepare an operation repository, produce its definition,
context, agents, skills and dev container files. Repository scope supplied for the
definition is not permission to enumerate or inspect live targets. Target discovery,
reconnaissance, builds, migrations and proposal publication belong to later
operational execution and require separate explicit authorisation. Keep unknown
execution details as named pending decisions rather than running the operation to
fill them in. A preparation request is not activation.

Preparation should normally be quick: assemble supplied facts, reuse the existing
templates and deliver a complete, inactive repository in this session. Ask focused
questions and grilling needed for an operable definition. Follow the interrogation
procedure in [prepare-operation](.github/skills/prepare-operation/SKILL.md): recommend
answers, challenge ambiguity and guide the operator until essential decisions are
settled. Avoid speculative questions unrelated to completion. Draft
unknowns may be pending while assembling, but essential gaps must be resolved or
reported as incomplete before final handover. Supply usable session procedures for
discovery, action, review, records and completion; do not leave them dependent on
future framework development. Execution-time target facts may be discovered there.

All agents except `reviewer` use OpenAI models. The single `reviewer` type uses
Anthropic and remains read-only.

Before creating any pull request, including drafts and definition/documentation
changes, follow [review-change](.github/skills/review-change/SKILL.md).
The Anthropic reviewer independently challenges OpenAI-authored changes.
Review the exact final diff; any subsequent edit requires
fresh review. Missing review evidence or unresolved findings blocks publication.
