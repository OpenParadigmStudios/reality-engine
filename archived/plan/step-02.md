# Step 02 — Establish Project Structure & Placeholder Routes

## Summary
Create canonical repository layout (app/, models/, routers/, schemas/, services/, tests/, alembic/) and add placeholder API routes.

## Contribution
Prevents architectural drift and speeds up future implementation by providing clear, conventional locations for code.

## Acceptance Criteria
- Directory structure matches the expected layout in `AGENTS.md`.
- Placeholder routers for `/api/v1/auth`, `/api/v1/campaigns` (list-only), and a root `/api/v1/ping`.
- CI-quality commands (`black`, `ruff`, `mypy`, `pytest`) all run with zero errors on skeleton code.
