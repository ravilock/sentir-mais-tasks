# Task: Async Analysis Job Producer

Type: Task
Status: Done

## Goal

Move chat request handling from synchronous analysis to enqueue-only analysis dispatch.

## Context

- `CreateChatService` and `SendMessageService` currently execute extraction, classification, and summary updates inline.
- The user-facing chat response should remain synchronous only for message persistence and immediate assistant reply generation.

## Scope

- Refactor chat services to publish an analysis job after message persistence
- Keep assistant reply generation synchronous
- Remove inline extraction/classification/summary update from the request path
- Fail the request if the analysis job cannot be enqueued

## Deliverables

- Refactored chat request path
- Producer integration with the Redis queue
- Tests proving chat requests no longer call analysis inline

## Acceptance Criteria

- `POST /chats` and `POST /chats/{chatId}/messages` return without waiting for extraction/classification.
- An analysis job is published with `chat_id`, `message_id`, `user_id`, and `message_created_at`.
