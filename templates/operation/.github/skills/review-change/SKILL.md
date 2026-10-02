---
name: review-change
description: Obtain independent adversarial review from a different model provider before creating any pull request, or re-review a changed candidate.
---

# Pre-publication adversarial review

Applies to every pull request, including drafts, target actions, definition revisions,
documentation and changes in this creator repository. Run within the Copilot session.

## Author/publisher procedure

1. Finish the candidate and applicable checks. Identify the base commit, candidate
   commit/tree and exact diff digest. Supply the complete diff, relevant base and
   candidate files, requirements, acceptance and test outcomes, including failures
   and checks not run. Include untracked files intended for publication.
2. Record every authoring model's actual identity/provider. All non-reviewer agents
   must use OpenAI models. Invoke the single `reviewer` agent, using Anthropic.
   Unknown authorship or a non-OpenAI authoring model blocks publication until the
   provider-policy violation is resolved; do not add alternate reviewer routes.
3. Confirm the selected reviewer is available and permitted by data-handling policy.
   Invoke it in a separate agent context with the review inputs and a bounded review
   objective. Verify the actual reviewer model/provider; frontmatter alone is not
   proof. An override or fallback outside Anthropic invalidates review.
4. Fix every reported finding and rerun affected checks. Obtain fresh independent
   review of the full final candidate, including fixes. Keep earlier findings and
   their resolutions. Disputed findings remain blockers pending reviewer resolution.
5. Before publication, verify a pass verdict with zero unresolved findings and that
   base, candidate and diff digest still match. Changed content or base invalidates
   review. Record author/reviewer identities, findings, resolutions, verdict and
   reviewed identities. A missing, failed or incomplete review blocks publication.

Passing review permits only the review gate to pass; approval, claims, acceptance,
privacy, resources and human merging remain separate requirements.

## Reviewer procedure

Read actual code and surrounding behaviour, not just the author's summary. Seek
counterexamples to requirements, invariants, boundary handling, failure/retry
behaviour, concurrency, permissions, confidentiality and compatibility. Evaluate
test evidence and identify material gaps. For context/configuration changes, examine
instruction contradictions, authority boundaries and missing references.

Use concrete evidence. Each finding names severity, file/lines, triggering conditions,
consequence and recommended correction. Distinguish genuine defects from optional
style suggestions; a suggestion alone is not a blocking finding.

Return **pass** only when the complete supplied change has been reviewed and no
findings remain. Return **changes required** with actionable findings, or **blocked**
when required context/evidence is missing. Include the reviewed base/candidate/diff
identities and actual reviewer model/provider. Never manufacture unavailable
evidence, edit the candidate or treat review as human approval.
