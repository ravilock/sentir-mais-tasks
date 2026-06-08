# Task: Backend Feature Authorization

Type: Task
Status: Cancelled

## Goal

Add authorization in `sentir-mais-backend` so expensive product features such as conversation access are not freely usable by any authenticated user when the MVP is deployed.

## Context

- Authentication already exists in `sentir-mais-backend`.
- There is currently no authorization layer for usage-gated features.
- The requirement is that this authorization flow must happen in `sentir-mais-backend`.

## Scope

- Define a minimal authorization model for MVP
- Decide how user entitlements are stored
- Protect costly routes, starting with conversation endpoints
- Return a clear error response when a user is authenticated but not allowed to use the feature

## Candidate MVP Shapes

- user-level boolean flags
- plan or subscription tier field
- allowlist-based access

## Deliverables

- Domain model changes for entitlements
- Authorization checks in backend services or middleware
- Protected chat route behavior
- Tests for allowed and denied access paths

## Acceptance Criteria

- A user can be authenticated but still denied access to conversation features unless explicitly authorized.
- The authorization decision is enforced in `sentir-mais-backend`, not delegated only to edge infrastructure or another service.
