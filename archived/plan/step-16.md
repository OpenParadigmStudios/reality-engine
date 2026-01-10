# Step 16 — AI-Assisted Session Logs (Proposal Pipeline)

## Summary
Add optional pipeline to ingest session logs and propose events via AI, with GM review before apply.

## Contribution
Accelerates log capture and event drafting while keeping GM fully in control.

## Acceptance Criteria
- Ingestion endpoint or CLI to submit transcripts.
- AI proposal service creates draft events linked to sessions.
- GM review UI endpoints for listing, accepting, or discarding proposals.
- Tests mock the AI layer; ensure proposals never auto-apply without approval.
