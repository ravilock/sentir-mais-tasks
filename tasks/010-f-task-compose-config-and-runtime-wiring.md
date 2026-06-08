# Task: Compose, Config, And Runtime Wiring

Type: Task
Status: Done

## Goal

Make the async analysis pipeline runnable locally and in deployment.

## Context

- The backend currently wires MongoDB, prompter, and classifier only.
- Redis and worker lifecycle must be added to configuration and local runtime composition.

## Scope

- Add Redis service to `sentir-mais-backend/docker-compose.yml`
- Add Redis-related environment variables to backend config
- Wire API startup to initialize Redis and start the worker
- Wire graceful shutdown to stop worker loops cleanly
- Update README and local run instructions

## Deliverables

- Redis in compose
- Runtime config for Redis and worker behavior
- Worker lifecycle in the backend process
- Documentation updates for local runs

## Acceptance Criteria

- Local compose boots Redis and the backend can start the worker successfully.
- API and worker run inside the same Go process with clean shutdown behavior.
