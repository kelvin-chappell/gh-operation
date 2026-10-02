# Context: Operation Runner

The language for undertakings carried out over a collection of repositories.

---

## The unit of work

**Operation** — One bounded undertaking: a single operator intent, clarified into a
single frozen specification, aimed at a single set of targets, under a single
resource envelope. The operation is the unit of work and the unit of accounting;
nothing happens outside an operation, and everything else in this language either
describes an operation, is a part of one, or is something done to one.

*Not to be confused with:* the intent that starts it, or with any one exploration or
action within it.

**Intent** — The operator's opening statement of what they want, before clarification.
Deliberately vague and not yet actionable.

**Operator** — The human who owns an operation. Supplies the intent, answers
interrogation, and holds final authority over the operation.

**Operation Standing** — Where an operation as a whole sits: *draft*, *specified*, *scoped*,
*active*, *concluded*, or *abandoned*.

---

## The activities

An operation is carried out through activities. Four are named here; three of them
(interrogation, exploration, action) do the work, and supervision governs them.

**Interrogation** — Converting an intent into a specification by structured
questioning of the operator. Ends only when the specification is complete enough to
freeze. *Also called:* grilling.

**Exploration** — Finding repositories that belong to an operation, and qualifying them.

**Action** — Performing an operation's task on one qualified target, producing one change
proposal. A **substantive failure** is the work failing; a **transient failure** is the
machinery failing — a rate limit, a lost runner, a network fault — and is retried
without counting as an attempt.

**Supervision** — Governing an operation: deciding how much interrogation, exploration,
and action to run, when to pause, and when the operation is concluded, within the
operation's envelope. Supervision does not itself interrogate, explore, or act.

**Reconnaissance** — Exploration performed without action, to measure the size and
cost of an operation before committing to it. *Also called:* a dry run. Produces a
**Feasibility Report**.

---

## The roles

**Agent** — The umbrella for every role in an operation. A supervisor, an elicitor, an
explorer, and an actor are each agents. "Agent" names the category and is never one
of its own members.

**Supervisor** — The agent that performs supervision. One per operation.
**Elicitor** — The agent that performs interrogation with the operator.
**Explorer** — An agent that performs exploration.
**Actor** — An agent that performs action. Several may work an operation at once.

*On "actor":* the word also means "the user who triggered a run" in the host
platform's own vocabulary. Within this language it means only the agent that performs
action.

---

## The specification

**Specification** — The frozen, checkable definition of an operation: what to find, what
to do, and how to tell whether it was done. Frozen before any action; changes require
a new operation. The specification is composed of *criteria*, a *task*, and *acceptance*.

**Criteria** — The conditions a repository must meet to belong to the operation.
Criteria range from *metadata* (language, topic, activity) to *content* conditions that
require reading repo files — for example, which versions of Scala the build declares.
All criteria are checked by code, never asserted by a model.

**Task** — The work performed on every target. The same in intent across all targets;
its implementation may adapt to each target, its intent may not.

**Acceptance** — The checks that decide whether an action succeeded.

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
*failed*, or *excluded*.

**Claim** — The exclusive right, held by one actor, to act on a target. A claim carries
a **lease**; an expired lease frees the target to be claimed again.

**Attempt** — One try by an actor at acting on a target. A target may accumulate
several attempts.

**Change Proposal** — What a successful action produces: a proposed modification to a
target, offered to the operator or the target's owners for review.

**Disposition** — A target's terminal outcome: accepted, no change needed, needs
operator, failed, or excluded.

---

## The operator's decisions

**Operator decision** — An explicit intervention by the operator on a single target,
outside the automatic loop. The system never makes one on its own.

**Reset** — Returns a *failed* target to *ready* with a fresh attempt budget.
**Resume** — Re-checks a blocker and returns a target that is *needs operator* to *ready*.
**Dismiss** — Closes a target that is *needs operator* as *excluded*.

---

## The records and resources

**Board** — The record of an operation's targets and their standings. One board per operation.

**Envelope** — The operation's declared limits: cost, concurrency, and wall-clock.

**Quota** — Limits imposed from outside the operation on how fast it may operate.

**Feasibility Report** — The projection reconnaissance produces: the expected size,
cost, and duration of an operation.

**Session Log** — The record of a supervision session, including its reasoning.

---

## Relationships

- An operation has one operator, one specification, one board, and one envelope.
- A specification is made of criteria, a task, and acceptance.
- Interrogation turns an intent into a specification.
- Exploration produces candidates; qualification promotes them to targets.
- Action consumes targets and produces change proposals.
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
| grilling | interrogation |
| dry run | reconnaissance |
| PR, pull request | change proposal |
| job, run | exploration or action (as the case may be) |
| batch, sweep | operation |
| agent (meaning one specific role) | the role's own name — supervisor, elicitor, explorer, actor |
