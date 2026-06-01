# Task: Structured Event Extraction

Type: Task
Status: In Progress

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

## Current Status Notes

- Implemented in `sentir-mais-backend`:
  - extracted event model in `internal/domain`
  - prompter-backed extraction client in `internal/llm`
  - extraction persistence on `message_analyses`
  - chat analysis flow now stores extracted event data when the prompter is configured
  - tests for valid and invalid extraction responses plus persistence behavior
- Still remaining:
  - stricter schema-level validation beyond JSON decoding
  - later orchestration refinement to trigger downstream stages based on extracted sufficiency state
  - classifier input is still the raw user message; moving it to extracted context belongs to task `005`

## Acceptance Criteria

- Once the conversation is deemed sufficient, the backend produces a stored structured event object.
