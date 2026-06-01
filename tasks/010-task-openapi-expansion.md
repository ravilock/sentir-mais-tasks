# Task: OpenAPI Expansion

Type: Task
Status: Not Started

## Goal

Bring `sentir-mais-backend/openapi.yaml` in line with the actual and planned MVP backend surface.

## Context

- The current OpenAPI file covers auth, chats, and weekly dashboard only.
- Missing API surface includes timeline and future analysis-driven dashboard behavior.

## Scope

- Update current endpoint docs where implementation has evolved
- Add new endpoints after they are implemented
- Keep request and response schemas aligned with `internal/api/requests` and `internal/api/responses`

## Deliverables

- Updated OpenAPI contract
- Validation that the documented payloads match the code

## Acceptance Criteria

- The OpenAPI file is not stale relative to the backend implementation.
