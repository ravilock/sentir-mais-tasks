# Task: Prompter Service

Type: Task
Status: In Progress

## Goal

Create a secondary backend named `sentir-mais-prompter` that receives prompt-oriented requests from `sentir-mais-backend` and proxies them to a backing LLM provider such as OpenRouter, OpenAI, or Gemini.

## Context

- The current backlog assumes direct LLM integration inside `sentir-mais-backend`.
- The updated architecture requires a separate service boundary for prompt execution.
- This service should isolate provider-specific logic from the main backend.

## Scope

- Create a new root folder: `sentir-mais-prompter`
- Expose API endpoints for the prompt operations required by the backend
- Support at least:
  - supportive conversation generation
  - structured extraction generation
- Add provider abstraction so the service can target different backends
- Add thin inter-service authentication using an API key

## Deliverables

- New service scaffold and runtime config
- Prompt request and response contracts
- Provider interface and first provider implementation
- API key authentication for requests coming from `sentir-mais-backend`
- Local run instructions and tests

## Current Status Notes

- Completed:
  - `sentir-mais-prompter` Python service scaffold
  - API-key auth
  - generic `POST /generate` endpoint
  - OpenAI-compatible provider client configured for OpenRouter by default
  - tests
  - Dockerfile
  - GitHub Actions pipeline for tests, Docker build, and GHCR publish
- Remaining work:
  - wire `sentir-mais-backend` to call `sentir-mais-prompter`
  - add richer OpenRouter routing/privacy controls from the research
  - add stricter structured extraction schema support if needed
  - verify with a live provider key

## Acceptance Criteria

- `sentir-mais-backend` can call `sentir-mais-prompter` over HTTP using an API key.
- The prompter can route requests to a configured backing provider without leaking provider-specific logic into the main backend.
