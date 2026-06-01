# Task: Frontend Chat API Integration

Type: Task
Status: Not Started

## Goal

Replace the frontend mock chat transport with the real backend HTTP client.

## Context

- `HANDOFF.md` explicitly calls this out as a follow-up after backend endpoints exist.
- The frontend still uses mock chat transport.

## Scope

- Replace chat mock calls with real HTTP requests
- Use the backend auth flow
- Remove temporary `sessionStorage` assumptions that conflict with backend truth where necessary

## Deliverables

- Real frontend-to-backend chat wiring
- Basic integration verification against the backend locally

## Acceptance Criteria

- A user can register/login and use the existing chat UI against the real backend instead of the in-browser mock.
