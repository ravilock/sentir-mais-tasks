# Task: Context Sufficiency Detection

Type: Task
Status: Not Started

## Goal

Implement the logic that decides when a conversation has enough context for structured extraction.

## Context

- `HANDOFF.md` defines four key reflection points:
  - what happened
  - what the user felt
  - how the user reacted
  - what the user expected should have happened or how they should have reacted
- This decision point does not exist in the backend yet.

## Scope

- Define a domain representation for context completeness
- Add analysis logic that inspects conversation history
- Store whether the chat has already reached extraction-ready state

## Deliverables

- Context sufficiency evaluator
- Tests covering enough-context and not-enough-context cases
- Persistence strategy for the evaluation result

## Acceptance Criteria

- The backend can decide whether to continue normal supportive chat or trigger extraction.
