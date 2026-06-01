# Task: Structured Event Extraction

Type: Task
Status: Not Started

## Goal

Add the extraction stage that turns conversation history into structured event/context data.

## Context

- The intended flow is `conversation -> extraction -> classifier`.
- The backend currently classifies raw user messages directly.

## Scope

- Define the extracted event schema
- Call the LLM with a dedicated extraction prompt
- Parse and validate the structured extraction output
- Persist the extracted data

## Deliverables

- Extraction service/module
- Extracted event model in `internal/domain`
- Storage and repository support
- Tests for valid and invalid extraction responses

## Acceptance Criteria

- Once the conversation is deemed sufficient, the backend produces a stored structured event object.
