# Spike: LLM Provider And Prompting

Type: Spike
Status: Done

## Goal

Define the first production LLM integration for `sentir-mais-backend`, including provider choice, request/response contract, supportive prompt, extraction prompt, and safety prompt behavior.

## Context

- `task-control/HANDOFF.md` defines OpenRouter as the current MVP assumption.
- `task-control/plan.md` requires an external LLM service with two jobs:
  - supportive conversation
  - structured extraction
- The backend currently uses `internal/llm/stub.go`.

## Questions To Resolve

- Which provider and model should be used first?
- What config values are required?
- What should the supportive system prompt contain?
- What should the extraction prompt return?
- How should crisis-risk language change the prompt or response path?
- What retry and timeout behavior is acceptable for MVP?

## Deliverables

- Written provider decision
- First prompt drafts
- JSON schema for extraction output
- Proposed backend config/env vars
- Error-handling notes for provider failures

## Current Status Notes

- OpenRouter MVP/provider research is complete and documented in `task-control/openrouter-mvp-research.md`.
- Sentir Mais-specific prompt and extraction decisions are documented in `task-control/llm-prompting-spec.md`.

## Acceptance Criteria

- A follow-up implementation task can be executed without re-deciding provider and prompt basics.
