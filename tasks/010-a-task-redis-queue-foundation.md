# Task: Redis Queue Foundation

Type: Task
Status: Done

## Goal

Introduce Redis-backed queue primitives and configuration for async analysis jobs.

## Context

- Chat analysis is currently executed inline in the request path.
- The backend needs a queue abstraction before request/worker responsibilities can be split.

## Scope

- Add Redis configuration to the backend
- Add a Redis client dependency
- Define the async analysis job payload
- Define queue operations for enqueue, consume, ack, and delayed retry

## Deliverables

- Backend Redis config fields
- Queue package/module
- Typed async analysis job schema
- Tests for queue serialization and basic operations

## Acceptance Criteria

- The backend can connect to Redis and enqueue/dequeue typed analysis jobs in tests.
