# Step 01 — Local Dev Environment (SQLite + FastAPI Hello World)

## Summary
Set up a runnable local development environment using Python 3.11, `uv`, SQLite, and FastAPI with a simple health/hello endpoint.

## Contribution
Creates a reliable baseline for iterative development and testing. Ensures contributors and agents can run the app identically.

## Acceptance Criteria
- Python 3.11 environment created via `uv venv` and activated.
- Dependencies installed from `requirements.txt`.
- Alembic initialized and `alembic upgrade head` runs without error using SQLite.
- FastAPI app starts with `uvicorn app.main:app --reload` and serves `/health` returning `{ "status": "ok" }`.
- Basic `.env.example` exists with required variables.
