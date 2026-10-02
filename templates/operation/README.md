# {{operation_name}}

**Standing: inactive.** This repository defines exactly one operation. Creation and
structural checks do not authorise target work.

Read [the definition](OPERATION.md), [shared context](CONTEXT.md) and
[governance](GOVERNANCE.md). [The manifest](MANIFEST.md) inventories the definition.
Agents start with [AGENTS.md](AGENTS.md).

All work starts within a Copilot session in a dev container. Container creation
supplies Copilot and `gh`; they are not declared in `.tool-versions`. Select the
appropriate local agent in the session. GitHub Actions does not trigger operation
processes, and opening a session does not activate the operation.

Review required context, local references and unresolved fields within the session.
Target execution remains blocked until governance requirements have been verified
and the operator separately authorises activation.

Definitions are approved as a whole, including agents, skills and container configuration.
Operational records belong on a separate restricted records branch, not in reused
definitions. The associated board records target stages and standings.
