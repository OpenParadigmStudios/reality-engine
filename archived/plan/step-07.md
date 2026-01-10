# Step 07 — GM/Player News Feeds with Visibility

## Summary
Build GM and Player feeds derived from events, filtered by role, campaign membership, and object bonds.

## Contribution
Validates the core value proposition: correct information to the right person at the right time.

## Acceptance Criteria
- GM feed: all campaign events with pagination and sorting.
- Player feed: filtered to global, own characters, groups, bonds, and session participation.
- Visibility rules enforced in queries; tests verify access control and correctness.
- Indexed queries for performance (campaign_id, created_at, visibility).
