# Step 09 — Approvals Workflow (GM Review)

## Summary
Implement GM review workflow for actions: approve, reject, or edit with comments before applying effects.

## Contribution
Provides GM oversight and accountability; ensures narrative quality and consistent rule application.

## Acceptance Criteria
- Approvals model with status (pending, approved, rejected) and reviewer/comment fields.
- Endpoints to list pending approvals (GM), and approve/reject/edit.
- Audit trail links actions → approvals → resulting events.
- Tests cover state transitions, permissions, and auditability.
