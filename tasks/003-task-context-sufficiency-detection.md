# Task: Context Sufficiency Handling And Persistence

Type: Task
Status: Not Started

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

## Acceptance Criteria

- The backend can use the extracted sufficiency result to decide whether to continue supportive chat or proceed to the next analysis stage.
