# Task: Classifier From Extracted Context

Type: Task
Status: Done

## Goal

Update the analysis flow so the classifier uses normalized extracted context instead of the raw user message.

## Context

- The classifier integration already exists.
- The current implementation classifies raw message text.
- `HANDOFF.md` and `plan.md` expect extracted context to be the classifier input.

## Scope

- Change the classifier orchestration input
- Decide which extracted fields are sent to the classifier
- Update analysis persistence to include extraction-to-classifier linkage

## Deliverables

- Updated orchestration in chat/analysis flow
- Expanded analysis model if needed
- Unit and integration tests

## Acceptance Criteria

- Classification runs from extracted context and stores the resulting feelings and confidence scores.
