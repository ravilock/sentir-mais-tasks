# Task: OpenRouter Client Integration

Type: Task
Status: Superseded

## Goal

Replace the stub LLM implementation with a real external client in `sentir-mais-backend`.

## Context

- The current backend only has `internal/llm/stub.go`.
- `HANDOFF.md` requires a real LLM for supportive replies.
- The provider decision and prompt format should come from `001-spike-llm-provider-and-prompting.md`.
- This task assumed direct backend-to-provider integration and is now superseded by the `sentir-mais-prompter` service direction.

## Current Status Notes

- Do not implement this task as originally written.
- Replace or rewrite it as backend-to-prompter integration.

## Scope

- Add an LLM client package under `internal/llm/`
- Add config/env vars for the provider
- Wire the client in `internal/api/api.go`
- Preserve an option to disable the external provider locally if needed

## Deliverables

- Real supportive reply generation
- Config support for API key, base URL, model, and timeout
- Unit tests for request/response mapping and failure handling

## Acceptance Criteria

- Chat creation and chat message sending use the external LLM instead of the stub when configured.
- The backend still starts with clear behavior when the LLM is not configured.
