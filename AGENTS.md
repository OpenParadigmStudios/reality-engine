# AGENTS.md: reality-engine

## Project Context
- **Type:** Backend API (FastAPI + SQLAlchemy)
- **Domain:** Tabletop RPG campaign management system
- **Architecture:** Event-sourced, REST API, role-based auth, YAML-configurable
- **Key Files:**
  - `PROJECT.md` — full technical architecture and domain model
  - `/docs` — detailed documentation (future)
  - `/plan` — phased implementation steps for agents to follow

## Tech Stack
- **Language:** Python 3.11
- **Package Manager:** `uv`
- **Web Framework:** FastAPI
- **ORM:** SQLAlchemy
- **Migrations:** Alembic
- **Validation:** Pydantic
- **DB (dev):** SQLite
- **DB (prod):** PostgreSQL
- **Config Format:** YAML

## Setup Commands
```bash
# Create virtual environment with uv
uv venv

# Activate virtual environment
source .venv/bin/activate  # Unix/macOS
# .venv\Scripts\activate   # Windows

# Install dependencies
uv pip install -r requirements.txt

# Initialize database
alembic upgrade head

# Run development server
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

## Testing Commands
```bash
# Run all tests
pytest

# Run with coverage
pytest --cov=app --cov-report=term-missing

# Run specific test file
pytest tests/test_<module>.py

# Run with verbose output
pytest -v
```

## Code Quality Commands
```bash
# Format code
black app/ tests/

# Lint code
ruff check app/ tests/

# Type check
mypy app/

# Run all checks before commit
black app/ tests/ && ruff check app/ tests/ && mypy app/ && pytest
```

## Database Commands
```bash
# Create new migration
alembic revision --autogenerate -m "description"

# Apply migrations
alembic upgrade head

# Rollback one migration
alembic downgrade -1

# View migration history
alembic history

# Reset database (dev only)
rm -f app.db && alembic upgrade head
```

## Project Structure (Expected)
```
reality-engine/
├── app/
│   ├── __init__.py
│   ├── main.py              # FastAPI app entry point
│   ├── config.py            # Settings/config
│   ├── database.py          # DB session management
│   ├── models/              # SQLAlchemy models
│   │   ├── __init__.py
│   │   ├── base.py          # GameObject base
│   │   ├── user.py
│   │   ├── campaign.py
│   │   ├── character.py
│   │   ├── group.py
│   │   ├── node.py
│   │   ├── sheet.py
│   │   ├── trait.py
│   │   ├── meter.py
│   │   ├── session.py
│   │   ├── event.py
│   │   ├── action.py
│   │   └── approval.py
│   ├── schemas/             # Pydantic schemas
│   ├── routers/             # API route handlers
│   │   ├── auth.py
│   │   ├── campaigns.py
│   │   ├── characters.py
│   │   ├── groups.py
│   │   ├── nodes.py
│   │   ├── sessions.py
│   │   ├── actions.py
│   │   ├── events.py
│   │   └── feeds.py
│   ├── services/            # Business logic
│   ├── utils/               # Helpers
│   └── yaml_config/         # YAML import/export
├── tests/
│   ├── __init__.py
│   ├── conftest.py          # pytest fixtures
│   ├── test_models/
│   ├── test_routers/
│   ├── test_services/
│   └── test_yaml_config/
├── alembic/                 # Migration files
├── docs/                    # Documentation
├── plan/                    # Implementation plan files
├── .env.example             # Environment template
├── requirements.txt         # Pinned dependencies
├── pyproject.toml           # Project metadata
├── alembic.ini              # Alembic config
├── PROJECT.md
├── README.md
└── AGENTS.md
```

## Coding Standards

### Python Style
- **Version:** Python 3.11+
- **Formatter:** `black` (line length: 88)
- **Linter:** `ruff`
- **Type Checker:** `mypy` (strict mode)
- **Imports:** Sort with `isort` (black-compatible profile)
- **Docstrings:** Google style

### Testing Requirements
- **Coverage:** Minimum 80% for new code
- **Unit tests:** Every function/method must have tests
- **Integration tests:** For API endpoints
- **Fixtures:** Use pytest fixtures in `conftest.py`
- **Naming:** `test_<function_name>_<scenario>`

### Dependency Management
- Use `uv` for all package operations
- Pin all dependency versions in `requirements.txt`
- Separate `requirements-dev.txt` for dev tools
- Document why each dependency is needed

### API Conventions
- **Versioning:** Prefix routes with `/api/v1`
- **Response Format:** JSON with Pydantic schemas
- **HTTP Status Codes:** Use standard REST codes
- **Auth:** Bearer token in `Authorization` header
- **Errors:** Return `{"detail": "message"}` structure
- **Pagination:** Query params `?skip=0&limit=100`

### Database Conventions
- **Naming:** snake_case for tables and columns
- **Primary Keys:** `id` (UUID or integer)
- **Timestamps:** Include `created_at`, `updated_at`
- **Soft Deletes:** Use `deleted_at` nullable field
- **Foreign Keys:** Explicit naming `<table>_id`

### Commit Message Format
```
<type>(<scope>): <subject>

<body>

<footer>
```
- **Types:** feat, fix, docs, style, refactor, test, chore
- **Scope:** auth, models, api, config, db, tests
- **Subject:** Imperative, lowercase, no period
- **Body:** Optional detailed explanation
- **Footer:** Reference issues/PRs

### Git Workflow
- **Branch naming:** `feature/<name>`, `fix/<name>`, `refactor/<name>`
- **Main branch:** `main` (protected)
- **Before commit:** Run all code quality checks
- **PR requirements:** Tests pass, coverage maintained, reviewed

## Domain Model Quick Reference

### Core Objects (Polymorphic GameObject base)
- **Character** — PC or NPC, optional player link, has sheet
- **Group** — Faction/organization, collection of characters/nodes
- **Node** — Location, resource, or abstract power; contended over

### Composition Objects
- **Sheet** — Structured state container attached to objects
- **Trait** — Reusable descriptor/tag (campaign or system-defined)
- **Meter** — Clock/tracker with current/max; triggers on threshold

### Narrative Objects
- **Event** — Append-only state change record with timestamp
- **Session** — Container for events, attendance, touched objects
- **Action** — Player-submitted intent with parameters
- **Approval** — GM review workflow (approve/reject/edit)

### System Objects
- **User** — Account with role per campaign
- **Campaign** — World container with settings
- **Feed** — Materialized event view with visibility filters

### Key Relationships
- User → many CampaignMemberships (role per campaign)
- Campaign → many GameObjects, Sessions, Events
- GameObject → one Sheet (optional), many Traits, many Meters
- Session → many Events, many attending Users
- Event → many affected GameObjects, one Session
- Action → one rule template → many Events (when approved)

## Environment Variables
```bash
# Required
DATABASE_URL=sqlite:///./app.db  # or postgresql://...
SECRET_KEY=<generate-secure-key>

# Optional
DEBUG=false
API_V1_PREFIX=/api/v1
ACCESS_TOKEN_EXPIRE_MINUTES=30
```

## Implementation Phase Context
Refer to `PROJECT.md` Roadmap section for phased implementation order:
- **Phase 0:** Environment setup (in progress)
- **Phase 1:** Core data model + feeds
- **Phase 2:** Actions + approvals
- **Phase 3:** Multi-GM tools
- **Phase 4:** Config/YAML
- **Phase 5:** Stretch (AI, advanced rules)

## Common Pitfalls
- Don't hardcode game system mechanics; use rule templates
- Always filter events by visibility before returning to players
- Event records are append-only; never update/delete
- Timestamps must be UTC
- YAML import must be idempotent and validate schemas
- Auth checks must verify both role and campaign membership

## Security Requirements
- **Auth tokens:** JWT or similar, short-lived, refresh flow
- **Password storage:** bcrypt or argon2
- **API keys:** For service-to-service if needed
- **Input validation:** Pydantic schemas on all inputs
- **SQL injection:** Use SQLAlchemy ORM, no raw SQL
- **CORS:** Configure allowed origins explicitly
- **Rate limiting:** Implement per-endpoint limits

## Performance Considerations
- **Event feed:** Index on campaign_id, created_at, visibility
- **Object queries:** Eager load sheets/traits when needed
- **Pagination:** Required for all list endpoints
- **N+1 queries:** Use `joinedload` or `selectinload`
- **Denormalization:** Materialize feeds for fast reads

## YAML Config Schema (Future)
Objects, sheets, traits, meters, and rules will be importable/exportable as YAML:
```yaml
# Example trait definition
traits:
  - id: vampire_clan_ventrue
    name: Ventrue
    type: clan
    description: Blue-blood aristocrats
    modifiers:
      - stat: willpower
        value: +1
```

## Success Validation
System works correctly when:
- GM feed shows all campaign events
- Player feed shows only authorized events (own chars, bonds, global)
- Event timeline is consistent and replayable
- YAML config round-trips without data loss
- All API endpoints respect role-based access
