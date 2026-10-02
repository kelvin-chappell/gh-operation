# Context: Operations

The language for undertakings carried out over a collection of repositories.

## Purpose of this repository

`gh-operation` enables an operator, in one Copilot session in a dev container, to
generate a **complete operation repository** from an intent. Preparation should be
quick: assemble the facts, resolve essential decisions with the operator and produce
the repository contents using existing templates and tools.

The Creator actively guides the operator to that result through pointed questions
and grilling. It recommends concrete answers, challenges vague or contradictory
requirements and follows up until essential decisions are settled. Questions come
in focused rounds, covering decisions whose prerequisites are already known; facts
are assembled by the Creator rather than turned into homework for the operator.
Speed means purposeful questioning and reuse, not skipping decisions or handing
over unresolved placeholders as a complete repository.

The operator then opens that repository in its own dev container. In **one Copilot
session there**, they can authorise the start and coordinate the whole operation
through discovery, action, adversarial review, verification and completion. Multiple
agents and checkpoint invocations are activities within that coordinating session,
not reasons to require a new top-level session for each step.

The generated repository must contain the context, definitions, agents, skills,
environment configuration and usable procedures needed for that end-to-end journey.
It must not depend on returning to this creator or developing missing machinery
before it can operate. An incomplete draft is an intermediate result, not the normal
finished product.

Creation and execution remain distinct: generating files does not discover or change
targets and does not activate the operation. Human decisions and external prerequisites
remain explicit. One-session completion is the intended supported journey, not a
promise to bypass blockers, merge gates or resource limits. If something prevents
completion, record an incomplete outcome rather than claiming success.

---

## The unit of work

**Operation** — One bounded undertaking: a single operator intent, clarified into a
versioned operation definition, aimed at a target set, under a single
resource envelope. The operation is the unit of work and the unit of accounting;
all operational execution belongs to an operation. Everything else in this language
describes an operation, is a part of one, or is something done to one.
Revisions may change its definition while continuing that intent; unrelated intent
belongs to a new operation.

*Not to be confused with:* the intent that starts it, or with any one exploration or
action within it.

**Operation Repository** — The shared context and agent guidance for carrying out
exactly one operation, together with its operational record. Each operation has
exactly one such repository, created after its initial definition has been approved.

**Preparation** — Turning an intent into an approved operation definition and
creating its complete operation repository within one creator session, ready for
the operator's end-to-end execution session. Its AI-credit limit and usage are
accounted for separately from operational execution.

**Creator** — The agent that governs preparation, bringing interrogation, assembly,
validation and repository creation together. It does not supervise the resulting
operation.

**Definition Bundle** — The complete, inspectable operation definition assembled
before its operation repository exists. Approval covers that bundle, not merely its
summary.

**Creation Plan** — The declared destination and resources to materialise for an
operation. Approval binds this plan as well as the definition bundle.

**Approval Receipt** — The record of an actual authorised person's approval of a
specific bundle and creation plan. A creator's assertion of consent is not a receipt.

**Materialisation** — Creating the approved operation repository and its associated
resources from a definition bundle. It does not activate the operation.

**Handover** — Transferring the created repository, records and setup report to the
operator. Outstanding external setup is stated explicitly, not mistaken for readiness.

**Intent** — The operator's opening statement of what they want, before clarification.
Deliberately vague and not yet actionable.

**Operator** — The human who owns an operation. Supplies the intent, answers
interrogation, and holds final authority over the operation.

**Delegate** — A person explicitly authorised by the operator to make specified
operational decisions. Repository access alone does not confer that authority.

**Operation Standing** — Where an operation as a whole sits: *draft*, *specified*, *scoped*,
*active*, *concluded*, or *abandoned*.

**Completion** — Exploration has finished, and every qualified target has satisfied
the final goal or has an explicit waiver. A conclusion caused by exhausted resources
or unresolved failures is not completion.

---

## The activities

An operation is carried out through activities. Four are named here; three of them
(interrogation, exploration, action) do the work, and supervision governs them.

**Interrogation** — Converting an intent into a complete operation definition by
structured questioning of the operator. Ends when the definition is ready for
approval. *Also called:* grilling.

**Exploration** — Finding repositories that belong to an operation, and qualifying them.

**Action** — Performing the task applicable to one qualified target: either one stage
transition or an unstaged operation's task. It produces at most one change proposal;
a target may need several actions and proposals. A **substantive failure** is the work
failing; a **transient failure** is the machinery failing — a rate limit, a lost runner,
a network fault — and is retried without counting as an attempt.

**Supervision** — Governing an operation: deciding how much interrogation, exploration,
and action to run, when to pause, and when the operation is concluded, within the
operation's envelope. Supervision does not itself interrogate, explore, or act.

**Reconnaissance** — Exploration performed without action, to measure the size and
cost of an operation before committing to it. *Also called:* a dry run. Produces a
**Feasibility Report**.

---

## The roles

**Agent** — The umbrella for automated roles in preparation and execution. Creator,
supervisor, elicitor, explorer and actor are each agents. "Agent" names the category
and is never one of its own members.

**Supervisor** — The agent that performs supervision. One per operation.
**Elicitor** — The agent that performs interrogation with the operator.
**Explorer** — An agent that performs exploration.
**Actor** — An agent that performs action. Several may work an operation at once.

*On "actor":* the word also means "the user who triggered a run" in the host
platform's own vocabulary. Within this language it means only the agent that performs
action.

---

## The specification

**Specification** — The versioned, checkable definition of an operation: what to find,
what to do, and how to tell whether it was done. It can be revised within the same
operation. It contains criteria and either one task with acceptance, or an
ordered set of stages and transitions with their tasks and acceptance.

**Specification Revision** — One identifiable version of an operation's specification.
A revision changes the definition without, by itself, creating a different operation.

**Operation Definition** — The specification and shared context that equip agents
to carry out an operation: its intent, scope, language, milestones, responsibilities
and working constraints.

**Execution Revision** — An identifiable, approved version of an operation definition
that governs work. Changing the specification or the prescribed behaviour requires
a newly approved execution revision, not necessarily a new operation.

**Operation Manifest** — The record of how an operation definition's parts fit
together, making its structure inspectable and reusable.

**Task Generation** — One approved definition of a transition's work or an unstaged
task. Materially changing its task, destination checks, acceptance or bounds creates
a new generation; an unrelated runtime repair does not.

**Criteria** — The conditions a repository must meet to belong to the operation.
Criteria range from *metadata* (language, topic, activity) to *content* conditions that
require reading repo files — for example, which versions of Scala the build declares.
All criteria are checked by code, never asserted by a model.

**Stage** — A named, verifiable milestone in a target's progress towards the
operation's goal, such as *Scala 2.13*. A target's current stage describes what it
has achieved, not what work is underway.

**Transition** — The prescribed move from one stage to the next, with its own task
and acceptance. Targets follow the same ordered stages, but start at the stage
their repository already satisfies.

**Final Stage** — The last stage required by the operation. Reaching an intermediate
stage is progress, not completion.

**Task** — The work prescribed for a transition, or for an unstaged operation.
Its intent is fixed within that scope; its implementation may adapt to each target,
its intent may not.

**Acceptance** — The checks that decide whether an action succeeded. In a staged
operation they establish that the destination stage has been reached.

---

## The targets

**Candidate** — A repository surfaced during exploration that has not yet been qualified.

**Target** — A repository that met the criteria and belongs to the operation. An operation's
targets are its **target set**.

**Qualification** — The decision that promotes a candidate to a target. It is
deterministic: a validator checks the criteria, extracting file content where a
content condition requires it.

**Target Standing** — Where a target sits in its lifecycle: *discovered*, *ready*,
*in action*, *proposal open*, *accepted*, *no change needed*, *needs operator*,
*failed*, *waived*, or *excluded*. Standing describes the work's lifecycle, independently
of the target's current stage.

**Claim** — The exclusive right, held by one operation and exercised by at most one
actor at a time, to perform one transition or unstaged task on a target. It covers
proposal review and verification and is released between transitions. A claim carries
a **lease**; an expired lease requires operator-confirmed recovery before takeover.

**Attempt** — One try by an actor at performing the applicable task on a target.
Retries of a transition are attempts, not additional stages.

**Change Proposal** — What a successful action produces: a proposed modification to a
target, offered to the operator or the target's owners for review.

**Disposition** — A target's recorded outcome: accepted, no change needed, waived,
failed, or excluded. *Needs operator* is unresolved, not a disposition; failure
does not count as completion.

---

## The operator's decisions

**Operator decision** — An explicit intervention by the operator on an operation or
one of its targets, outside the automatic loop. The system never makes one on its own.

**Activation** — The operator's explicit authorisation to start an approved execution
revision once the operation is ready. Creation of its repository is not activation.

**Reactivation** — Explicitly starting a previously concluded operation again under
a newly approved execution revision that continues the same intent.

**Revision Handover** — Replacing the active execution revision after active actions
have finished and outstanding proposals have been resolved. Two execution revisions
do not dispatch work concurrently within one operation.

**Reset** — Returns a *failed* target to *ready* with a fresh attempt budget.
Earlier achieved stages are retained.
**Resume** — Re-checks a blocker and the target's progress, returning it to the
applicable work or recognising a verified final outcome.
**Dismiss** — Closes an unqualified candidate as *excluded*. It cannot excuse
unfinished work on a qualified target; that requires a waiver.

**Waiver** — An operator's explicit, reasoned decision to leave a qualified target
short of the goal. It gives the target the standing *waived* and counts towards
completion without claiming that the final stage was reached. It carries forward
only while the relevant obligations remain unchanged; changed work requires reaffirmation.

**Emergency Stop** — Immediately halting dispatch, active work and publication.
It does not free claims or resolve proposals without safe recovery.

---

## The records and resources

**Board** — The record of an operation's targets, their stages and their standings.
One board per operation.

**Envelope** — The operation's declared limits: cost in AI credits, concurrency and
wall-clock. Its credit threshold stops new dispatch; in-flight overshoot is recorded.
Cost is not measured or forecast in currency.

**Quota** — Limits imposed from outside the operation on how fast it may operate.

**Feasibility Report** — The projection reconnaissance produces: the expected size,
cost, and duration of an operation.

**Session Log** — The structured record of a supervision session, including its
inputs, decisions, explicit rationale and usage. Available permitted traces may
supplement it; undisclosed internal reasoning is not required.

---

## Relationships

- An operation has one operator, one versioned definition, one board, one envelope
  and exactly one operation repository.
- Preparation assembles a definition bundle and creation plan, obtains approval
  and materialises the repository. Handover does not activate it.
- An operation repository defines and carries out exactly one operation, retaining
  its operational record across specification revisions.
- An operation definition includes the specification and prescribed behaviour; an
  approved execution revision governs each action.
- Creating an operation repository does not activate it. Conclusion or abandonment
  stops new work while preserving the operation's record.
- A specification contains criteria and either one task with acceptance, or stages
  linked by transitions with their own tasks and acceptance.
- Interrogation turns an intent into an operation definition.
- Exploration produces candidates; qualification promotes them to targets.
- Action performs the applicable task on a target and may produce a change proposal.
- A target can enter at any verified stage; each transition advances it towards the
  final stage. Accepting an intermediate proposal does not finish the target.
- Stage describes achieved progress; standing describes the lifecycle of the work.
- Completion requires every qualified target to satisfy the goal or have a waiver.
- Supervision governs interrogation, exploration, and action, and may pause or conclude
  the operation.
- Reconnaissance is exploration without action.
- A candidate becomes a target only by qualification; a target ends only in a disposition.

---

## Avoided terms

| Avoided | Use instead |
|---|---|
| mission | operation |
| item, repo item | target |
| status, state | standing |
| column (meaning achieved progress) | stage |
| grilling | interrogation |
| dry run | reconnaissance |
| PR, pull request | change proposal |
| job, run | exploration or action (as the case may be) |
| batch, sweep | operation |
| agent (meaning one specific role) | the role's own name — supervisor, elicitor, explorer, actor |
