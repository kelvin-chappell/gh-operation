# Creator workspace

Work within Copilot sessions in a dev container. This repository prepares operation
definitions; generated repositories own their execution.

All agents except `reviewer` use OpenAI models. The single `reviewer` type uses
Anthropic and remains read-only.

Before creating any pull request, including drafts and definition/documentation
changes, follow [review-change](.github/skills/review-change/SKILL.md).
The Anthropic reviewer independently challenges OpenAI-authored changes.
Review the exact final diff; any subsequent edit requires
fresh review. Missing review evidence or unresolved findings blocks publication.
