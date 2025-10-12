# Step 03 — Auth Skeleton (Roles & Campaign Scoping)

## Summary
Introduce basic auth scaffolding: JWT tokens, user model, campaign membership, and role placeholders (Player, GM, Head GM).

## Contribution
Establishes the foundation for role-based visibility and access controls used across the API.

## Acceptance Criteria
- `/api/v1/auth/token` issues JWT for a dev user.
- `User`, `Campaign`, and `CampaignMembership` models exist with role enum.
- Dependency for extracting current user and campaign scope resolves and is testable.
- Protected test route demonstrates role-gated access.
