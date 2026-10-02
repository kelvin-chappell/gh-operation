# 0009 - A Copilot-driven creator with deterministic materialisation

- **Status:** Accepted
- **Date:** 2026-10-02
- **Deciders:** operator
- **Related:** `DESIGN.md` §20; ADR 0008; `CONTEXT.md` (Preparation, Creator, Definition Bundle, Approval Receipt)

## Decision and rationale

**Packaging correction:** ADR 0010 supersedes the copied controller/runtime-base
assumption below. Preparation, authentic approval and inactive handover remain;
the maintained base now consists primarily of agent context. ADR 0011 replaces
workflow-triggered execution with Copilot sessions in dev containers.

Make this repository a Copilot workspace taking an intent through interrogation,
assembly, validation, authentic approval and inactive handover. Maintain a tested,
versioned runtime base and curated starting points; generate operation-specific
definitions/adapters without rewriting safety machinery for each operation.

Copilot plans and assembles; deterministic tools verify the full bundle, approval
receipt and creation plan, then create and verify repository resources. Copilot may
invoke materialisation after conversational approval, but cannot self-approve or
receive creation credentials. Approval binds exact contents and destination; inability
to verify genuine human consent blocks creation.

Use resumable private preparation workspaces outside the public checkout, with
quarantined explicit-source reuse and scoped privacy-gated read-only inspection.
Journal materialisation and resume only verified owned resources. Ambiguous ownership
blocks; no automatic overwrite, rename or deletion. Complete handover transfers
approval/provenance, accounting and setup evidence into an inactive operation
repository; external access bindings and activation remain operator decisions.

Preparation has its own AI-credit threshold and permits clearly labelled estimates.
Actual usage corrects estimates without double counting. This does not weaken the
authoritative credit requirement for operational execution.

## Trade-offs and scope

A Copilot workspace avoids building another conversational application, but requires
proven configuration isolation and an authentic approval/tool boundary. A maintained
base duplicates source into generated repositories, but avoids untested safety-code
generation and preserves independent execution.

V1 uses explicit predecessors and curated patterns, not automatic discovery/ranking.
It creates initial repositories, not ongoing revisions or fleet supervision. The
operator confirmed three interview rounds; agents/helpers and the runtime are still
to be implemented and validated.
