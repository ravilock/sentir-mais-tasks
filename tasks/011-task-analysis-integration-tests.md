# Task: Analysis Integration Tests

Type: Task
Status: Not Started

## Goal

Add integration coverage for the end-to-end analysis path.

## Context

- Auth integration tests already exist.
- Missing integration coverage includes chat analysis persistence and dashboard reads.

## Scope

- Cover chat creation and message send against MongoDB
- Verify `message_analyses` persistence
- Add coverage for dashboard endpoints once they use real data
- Stub external dependencies where needed at the HTTP boundary

## Deliverables

- Integration tests for chat plus analysis
- Integration tests for dashboard reads

## Acceptance Criteria

- The backend test suite verifies the real Mongo-backed analysis flow instead of only unit behavior.
