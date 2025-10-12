# Step 08 — Actions API (Player-submitted)

## Summary
Add an endpoint for players to submit actions with parameters/options and a narrative write-up, with server-side validation.

## Contribution
Creates the structured entry point for player intent that drives rule processing and event generation.

## Acceptance Criteria
- `POST /api/v1/actions` to submit actions with schema validation.
- Actions reference the campaign, actor (user/character), and target objects.
- Preview endpoint (optional) validates inputs and computes proposed effects (stubbed).
- Tests cover validation, permissions, and basic flows.
