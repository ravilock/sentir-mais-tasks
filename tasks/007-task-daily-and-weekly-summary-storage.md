# Task: Daily And Weekly Summary Storage

Type: Task
Status: Not Started

## Goal

Implement persistence for `daily_summaries` and `weekly_summaries`.

## Context

- These collections are explicitly expected in `HANDOFF.md`.
- The backend currently has no repositories for summaries.

## Scope

- Define summary document models
- Add repositories for daily and weekly summaries
- Decide whether summaries are computed on demand, cached, or persisted after analysis

## Deliverables

- Domain models for daily and weekly summaries
- MongoDB repositories and indexes
- Tests for summary persistence behavior

## Acceptance Criteria

- The backend can store and retrieve daily and weekly summary data.
