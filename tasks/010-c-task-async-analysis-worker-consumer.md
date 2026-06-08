# Task: Async Analysis Worker Consumer

Type: Task
Status: Done

## Goal

Run the analysis pipeline from a Redis-backed worker in the same Go process as the API.

## Context

- The queue foundation and producer path are required first.
- The worker must reconstruct authoritative state from MongoDB before processing.

## Scope

- Add worker startup and shutdown lifecycle to the backend
- Consume analysis jobs from Redis
- Reload the target message and full chat history from MongoDB
- Run extraction, classification, analysis persistence, and summary update asynchronously

## Deliverables

- Worker loop running in the backend process
- Consumer service that processes typed analysis jobs
- Tests for successful async processing

## Acceptance Criteria

- The worker consumes queued jobs and persists `message_analyses`.
- Dashboard data updates eventually after successful worker processing.
