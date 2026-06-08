# Task: Observability And OpenAPI Doc Alignment

Type: Task
Status: Done

## Goal

Document and observe the async analysis pipeline accurately after the implementation is complete.

## Context

- The previous `010` task was a generic OpenAPI cleanup task.
- After the async analysis migration, the main documentation risk is stale behavior rather than missing routes alone.

## Scope

- Update `openapi.yaml` to reflect async analysis behavior where relevant
- Document eventual consistency for analysis-driven dashboard updates
- Add structured logs for enqueue, consume, retry, fallback, dead-letter, and success
- Align README behavior descriptions with the implemented async pipeline

## Deliverables

- Updated OpenAPI contract
- Updated README/docs
- Structured logging for worker lifecycle and job stages

## Acceptance Criteria

- Documentation is not stale relative to the implemented async pipeline.
- Logs clearly expose enqueue, consume, retry, fallback, dead-letter, and success transitions.
