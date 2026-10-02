# Shared context

This operation uses the following domain distinctions:

- **Operation:** one bounded operator intent, target set and execution envelope.
- **Specification:** criteria, tasks or ordered stage transitions, and acceptance.
- **Stage:** verified achieved progress, not the work currently underway.
- **Standing:** lifecycle position, independently of stage.
- **Candidate:** discovered repository awaiting deterministic qualification.
- **Target:** repository that met the criteria and belongs to the operation.
- **Claim:** exclusive ownership of one target task/transition, including review and
  verification; expired ownership requires operator-confirmed recovery.
- **Task generation:** identity of approved work, changed when its obligations change.
- **Change proposal:** proposed target modification offered for human review.
- **Waiver:** authorised, reasoned exemption from an unfinished qualified obligation.
- **Envelope:** AI-credit, concurrency and wall-clock limits.

Operation-specific facts belong here. Keep tasks and acceptance in
[OPERATION.md](OPERATION.md), and lifecycle rules in [GOVERNANCE.md](GOVERNANCE.md).
