# Step 06 — Events & Sessions (Append-only Logging)

## Summary
Implement `events` (append-only) and `sessions` (containers) with timelines per object and campaign.

## Contribution
Creates the narrative heartbeat and auditable history required for feeds and replay.

## Acceptance Criteria
- Models for Event and Session with UTC timestamps.
- API to append events to sessions and reference affected objects.
- Object timeline queries (by character/group/node) with pagination.
- Tests ensure events are append-only and timelines are consistent.
