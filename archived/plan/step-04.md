# Step 04 — GameObject Base & Core Tables (Characters, Groups, Nodes)

## Summary
Implement polymorphic `GameObject` base and create `characters`, `groups`, and `nodes` models with CRUD endpoints.

## Contribution
Establishes the core object model the system is built around, enabling future composition via sheets/traits/meters.

## Acceptance Criteria
- SQLAlchemy models for `GameObject`, `Character`, `Group`, `Node` with migrations.
- Pydantic schemas and CRUD routers for each.
- Basic search and pagination for list endpoints.
- Unit tests for models and routers (80%+ coverage for new code).
