# Task: Backend To Prompter Integration

Type: Task
Status: Done

## Goal

Integrate `sentir-mais-backend` with `sentir-mais-prompter` so the backend uses the prompter service for supportive generation and later structured extraction.

## Context

- The current backend still uses `internal/llm/stub.go`.
- The architecture is now:
  - `sentir-mais-backend -> sentir-mais-prompter -> OpenRouter`
- Direct backend-to-provider integration is no longer the intended path.
- Provider and prompt decisions are already documented in:
  - `task-control/openrouter-mvp-research.md`
  - `task-control/llm-prompting-spec.md`

## Scope

- Add a prompter client in `sentir-mais-backend`
- Add backend config/env vars for:
  - `PROMPTER_BASE_URL`
  - `PROMPTER_API_KEY`
  - `PROMPTER_TIMEOUT_SECONDS`
- Replace the stub supportive generation path with a prompter-backed implementation
- Keep the backend behavior clear when the prompter is not configured
- Prepare the interface so extraction can also be routed through the same service later

## Deliverables

- HTTP client for `sentir-mais-prompter`
- request/response mapping for `POST /generate`
- composition root wiring in the backend
- service-level error handling for prompter failures
- unit tests for client behavior and orchestration

## Current Status Notes

- Implemented in `sentir-mais-backend`
- Added prompter client and supportive message mapping
- Added backend config for `PROMPTER_BASE_URL`, `PROMPTER_API_KEY`, and `PROMPTER_TIMEOUT_SECONDS`
- Wired the composition root to use the prompter when configured and fall back to the local stub otherwise
- Added local compose and Makefile support for the prompter service
- Added tests and verified with `make test`

## Acceptance Criteria

- Chat creation and message sending use `sentir-mais-prompter` for supportive replies when configured.
- The backend sends the configured API key to the prompter.
- Provider/proxy failures are surfaced as controlled backend errors instead of silent failures.
