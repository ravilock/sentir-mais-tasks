# Task: Dashboard Week Aggregation

Type: Task
Status: Not Started

## Goal

Replace the placeholder weekly dashboard service with real aggregation from stored analysis data.

## Context

- `internal/dashboard/services/get_week.go` currently returns an empty summary.
- The product needs weekly feelings and notable events over time.

## Scope

- Aggregate from stored message analyses and extracted events
- Produce dominant feelings, main events, and timeline points
- Use persisted summaries if `007-task-daily-and-weekly-summary-storage.md` is completed first

## Deliverables

- Real `GET /dashboard/week` service logic
- Tests for aggregation behavior

## Acceptance Criteria

- `/dashboard/week` returns non-placeholder data derived from stored user analysis records.
