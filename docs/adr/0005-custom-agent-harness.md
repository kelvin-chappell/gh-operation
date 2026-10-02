# 0005 — A custom agent harness, not an agent framework

- **Status:** Superseded by [0008](0008-one-repository-per-operation.md)
- **Date:** 2026-10-01
- **Deciders:** operator
- **Related:** `DESIGN.md` §1, §8, §12, §13.1, §14

## Context

The four roles — Elicitor, Supervisor, Explorer, Actor — each run a loop over an LLM API
plus GitHub tools. Mature agent frameworks exist and would supply retries, tool
plumbing, and tracing.

But three requirements are load-bearing and unusual:

1. **Verbatim reasoning traces must be logged** for every supervision invocation (§13.1);
   frameworks often abstract away or reshape the raw model output.
2. **Tool sets must be minimal and non-exfiltrating**, because repository content is
   untrusted input (prompt injection).
3. **Model output must be constrained to structured file-edit operations** that are
   validated before applying.

## Decision

The following records the original runtime choice. ADRs 0010/0011 establish the
current choice: agent context executed within Copilot sessions in dev containers,
with structured audit and only available permitted traces.

Build a **minimal custom loop** over an LLM API with hand-written GitHub tools. No agent
framework.

## Consequences

**Positive**
- Full control over prompts, tool schemas, and — critically — capturing the model's full
  reasoning output, not just its final answer.
- A tiny dependency surface, and a tool set we can reason about for injection safety.
- Structured, validated output is first-class rather than bolted on.

**Negative**
- We build and maintain what a framework would provide: retries, tracing, tool plumbing.
- The loop is coupled to specific model/provider APIs, including the reasoning-trace
  availability that §13.1 depends on.
- Fewer batteries included means more of the harness is ours to test.
