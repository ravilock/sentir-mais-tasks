# Task: Per-Chat Ordering And Idempotency

Type: Task
Status: Done

## Goal

Prevent out-of-order analysis processing and duplicate analysis writes.

## Context

- Multiple user messages from the same chat may be queued close together.
- Async processing must not classify a later message before an earlier one from the same chat.
- Duplicate queue deliveries must be harmless.

## Scope

- Add a per-chat processing lock using Redis
- Delay or requeue jobs when a chat is already being processed
- Skip work when a `message_analysis` already exists for the target `message_id`
- Preserve `message_id` uniqueness as a hard persistence guard

## Deliverables

- Per-chat locking behavior
- Idempotent worker processing
- Tests for concurrent same-chat jobs and duplicate job delivery

## Acceptance Criteria

- Jobs for the same chat are processed in order.
- Duplicate jobs do not produce duplicate analyses or inconsistent summaries.
