# Step 13 — Feed Performance & Materialized Views

## Summary
Optimize feeds and timelines with indexes and optional materialized views for common queries.

## Contribution
Improves responsiveness and scalability for campaigns with large event volumes.

## Acceptance Criteria
- DB indexes on campaign_id, created_at, visibility for feeds.
- Optional materialized view or denormalized table for latest events per object.
- Pagination defaults and limits enforced.
- Benchmarks demonstrate improved latency on sample datasets.
