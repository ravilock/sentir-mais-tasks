# Task: Analysis Data Model Expansion

Type: Task
Status: Done

## Goal

Expand persistence so the backend stores the full analysis lifecycle, not only message-level classifier output.

## Context

- `message_analyses` exists.
- Missing pieces include structured event storage and context-completeness state.

## Scope

- Extend domain models
- Add MongoDB repositories and indexes
- Decide whether extracted events live inside `message_analyses` or in a dedicated collection

## Deliverables

- Final document shape for analysis records
- Repository implementation
- Migration notes for local environments if needed

## Acceptance Criteria

- MongoDB persistence can represent:
  - context sufficiency state
  - extracted event/context
  - classifier outputs
  - model/provider metadata
