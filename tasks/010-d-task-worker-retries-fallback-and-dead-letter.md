# Task: Worker Retries, Fallback, And Dead Letter

Type: Task
Status: Done

## Goal

Make the async analysis worker resilient with bounded retries, extraction fallback, and dead-letter persistence.

## Context

- Prompter and classifier calls can time out or fail intermittently.
- Summary writes can also fail independently from extraction/classification.
- Failed jobs must remain debuggable without blocking the chat UX.

## Scope

- Extraction retries: 3 attempts with 5-second backoff
- Classifier retries: up to 10 attempts
- Summary retries: 3 attempts with 5-second backoff
- Increase API->Prompter and API->Classifier timeouts per attempt
- Fallback to raw-message classification after extraction retry exhaustion
- Persist dead-letter records in MongoDB with debug metadata

## Deliverables

- Retry policy implementation per stage
- Extraction fallback behavior
- Dead-letter MongoDB collection and repository
- Tests for retry exhaustion and dead-letter writes

## Acceptance Criteria

- Each stage follows the agreed retry policy.
- Extraction exhaustion falls back to raw-message classification.
- Final unrecoverable failures are persisted to MongoDB for debug.
