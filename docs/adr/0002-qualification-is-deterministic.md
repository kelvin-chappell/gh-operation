# 0002 — Qualification is deterministic; the model records evidence but never promotes

- **Status:** Accepted
- **Date:** 2026-10-01
- **Deciders:** operator
- **Related:** `DESIGN.md` §2, §5.1, §6; `CONTEXT.md` (Qualification, Criteria); ADR 0008

## Context

Exploration surfaces candidate repositories and must decide which belong to the
operation. Some criteria are mechanical, but others need reading content — *"which Scala
versions does this repository use?"* — where a model could help. The tempting design is
to let the model decide membership, especially on borderline cases.

The hazard: a model-decided target set is not reproducible. A rerun could admit or drop
repositories, and no one could explain why one repo was in and another out. The set of
repositories an operation touches is exactly the thing that must be auditable.

## Decision

A candidate becomes a **target** only when a **deterministic validator** confirms the
criteria — metadata conditions and content conditions alike (content values extracted
by declared methods, e.g. regex over `build.sbt`). The explorer's LLM may **annotate** a
candidate and record structured evidence, but it **never promotes** a candidate.
Machine-checkable criteria are a required rubric item before an execution revision is approved.

## Consequences

**Positive**
- Qualification is reproducible from identical evidence and the same approved revision;
  fresh discovery may observe changed repositories and must report that drift.
- Qualification is auditable — the reason a repo is in the operation is a checkable rule.
- Ambiguity is pushed into the specification, where the operator can see and approve it.

**Negative**
- Fuzzy criteria cannot be expressed as judgement; they must be reduced to metadata or
  a content probe (a file plus an extraction method).
- Content criteria require fetching and parsing repository files, so exploration needs
  read access to repository contents (read-only in reconnaissance, still subject to
  confidentiality gates).
