# Task: Context Sufficiency Handling And Persistence

Type: Task
Status: Done

## Goal

Implement backend-side handling of context sufficiency so the extraction result can be persisted and used in orchestration.

## Context

- `HANDOFF.md` defines four key reflection points:
  - what happened
  - what the user felt
  - how the user reacted
  - what the user expected should have happened or how they should have reacted
- `task-control/llm-prompting-spec.md` now defines that the extraction flow returns:
  - `enough_context`
  - `context_gaps`
- The backend should not maintain a competing heuristic truth source for this in the MVP.

## Scope

- Define a domain representation for context completeness
- Persist `enough_context` and `context_gaps`
- Use the persisted sufficiency result in backend orchestration
- Decide whether the chat should continue supportive conversation or proceed to extraction/classification flow based on that result

## Deliverables

- Domain and persistence support for context sufficiency state
- Tests covering enough-context and not-enough-context orchestration paths
- Clear orchestration rule for when the backend triggers the next stage

## Current Status Notes

- Implemented in `sentir-mais-backend`:
  - domain fields for `enough_context` and `context_gaps`
  - persistence support on `message_analyses`
  - backend decision function for continue-vs-next-stage behavior
  - unit tests for unknown, insufficient, and sufficient context states
  - live analysis flow now uses extracted sufficiency state to decide whether to proceed to classification
  - insufficient context now persists extraction state without advancing to the next stage

## Acceptance Criteria

- The backend can use the extracted sufficiency result to decide whether to continue supportive chat or proceed to the next analysis stage.
