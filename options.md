## Architectural Options for `reality-engine`

A curated set of software engineering choices for a startup-scale prototype. Each decision includes a quick overview and options with pros/cons and an S–F recommendation tier.

### 1) Python environment and dependency management
**Overview**: How to create and manage the virtual environment and install deps.
- **uv (current plan)** — very fast resolver/installer, simple CLI; lighter than Poetry. Cons: newer ecosystem, fewer tutorials. — Tier: S
- **pip + venv** — ubiquitous, minimal moving parts. Cons: slower, manual extras (hashes, caching). — Tier: A
- **Poetry** — lockfile, extras, publishing. Cons: heavier, slower for simple projects. — Tier: B
- **Pipenv** — simple UX. Cons: aging, less momentum, lock quirks. — Tier: C

### 2) Python version management
**Overview**: How developers ensure consistent Python 3.11.
- **pyenv** — common, per-project versions. Cons: macOS shims friction. — Tier: A
- **asdf** — multi-runtime manager, one tool for all. Cons: extra plugin layer. — Tier: A
- **System Python only** — lowest friction upfront. Cons: drift, conflicts. — Tier: C

### 3) Web framework
**Overview**: API framework for REST endpoints.
- **FastAPI (current plan)** — async-first, Pydantic, OpenAPI, great DX. Cons: learning curve for DI. — Tier: S
- **Flask** — minimal, huge ecosystem. Cons: add-ons for typing, OpenAPI. — Tier: B
- **Django REST Framework** — batteries included. Cons: heavyweight for this domain. — Tier: C

### 4) ORM and models
**Overview**: Data mapping layer.
- **SQLAlchemy (current plan)** — mature, flexible, async support, Alembic. Cons: verbosity. — Tier: S
- **SQLModel** — SQLAlchemy + Pydantic ergonomics. Cons: project velocity varies. — Tier: B
- **Tortoise ORM** — async friendly. Cons: ecosystem smaller vs SA. — Tier: C

### 5) Migrations
**Overview**: Schema evolution tool.
- **Alembic (current plan)** — standard with SQLAlchemy, autogenerate + manual control. Cons: learning curve. — Tier: S
- **No migrations (manual SQL)** — quick to start. Cons: risky, brittle, no history. — Tier: F

### 6) Dev database
**Overview**: Local database for development.
- **SQLite (current plan)** — zero-setup, fast. Cons: SQL dialect/locking differences from Postgres. — Tier: S
- **PostgreSQL (containers)** — parity with prod. Cons: setup overhead. — Tier: A

### 7) Prod database
**Overview**: Primary production datastore.
- **PostgreSQL (current plan)** — robust, rich features, JSONB, indexing. Cons: ops overhead. — Tier: S
- **MySQL/MariaDB** — familiar to some. Cons: fewer JSON/CTE niceties. — Tier: C
- **SQLite** — simple deploy. Cons: concurrency/scale limits. — Tier: D

### 8) Auth token strategy
**Overview**: Access token approach for APIs.
- **Short-lived access + refresh** — secure, revocable, least blast radius. Cons: extra endpoints/state. — Tier: S
- **Long-lived access only** — simplest. Cons: riskier if leaked, manual rotation. — Tier: C

### 9) Password hashing
**Overview**: Algorithm for storing passwords.
- **bcrypt** — widely used, good defenses, python support solid. Cons: tuning required. — Tier: S
- **argon2id** — stronger memory-hard defaults. Cons: native build, more deps. — Tier: A

### 10) Config and settings
**Overview**: Manage secrets/config across envs.
- **Pydantic BaseSettings + .env (current plan)** — typed, simple overrides. Cons: beware env drift. — Tier: S
- **dynaconf/decouple** — convenience layers. Cons: less type safety. — Tier: B
- **Env vars only** — minimal. Cons: parsing/validation boilerplate. — Tier: C

### 11) API versioning
**Overview**: How clients target API versions.
- **Path versioning `/api/v1` (current plan)** — explicit, cache-friendly. Cons: URL churn on major changes. — Tier: S
- **Header-based** — clean URLs. Cons: tooling, discoverability. — Tier: B

### 12) Request/response validation
**Overview**: Schema modeling layer.
- **Pydantic models (current plan)** — fast, typed, integrates with FastAPI. Cons: model duplication with ORM. — Tier: S
- **Marshmallow** — flexible serialization. Cons: extra glue with FastAPI. — Tier: C

### 13) Event sourcing approach
**Overview**: How to model history and state.
- **Light event log + materialized state (current plan)** — pragmatic, readable, replay-friendly. Cons: dual-write complexity. — Tier: S
- **Full CQRS + event store** — highly scalable. Cons: overkill now, heavy infra. — Tier: C
- **CRUD only (no events)** — simplest. Cons: loses audit/replay. — Tier: D

### 14) Feeds/materialization strategy
**Overview**: How to serve feeds efficiently.
- **Query-on-read with good indexes** — simplest, fine at small scale. Cons: can slow with volume. — Tier: A
- **Denormalized tables/materialized views (optional)** — fast reads. Cons: complexity, refresh logic. — Tier: S (as optional step)
- **External cache first (Redis)** — very fast. Cons: extra infra/consistency. — Tier: B

### 15) Visibility enforcement
**Overview**: Where to apply authorization filters.
- **DB query layer (SQLAlchemy filters)** — efficient, least data leaked. Cons: more complex queries. — Tier: S
- **Post-filtering in Python** — simple to write. Cons: wasteful, risky. — Tier: D
- **DB row-level security** — strong guarantees. Cons: setup complexity. — Tier: B

### 16) Pagination
**Overview**: Strategy for large lists.
- **Limit/offset (current plan)** — easy and supported everywhere. Cons: unstable with inserts, slower at high offsets. — Tier: S
- **Cursor-based** — stable ordering at scale. Cons: added complexity. — Tier: A

### 17) Testing stack and coverage
**Overview**: Testing tools and thresholds.
- **pytest + coverage ≥80% (current plan)** — ergonomic, fixtures, plugins. Cons: discipline required. — Tier: S
- **unittest only** — stdlib only. Cons: less ergonomic. — Tier: C

### 18) Code quality tooling
**Overview**: Linting/formatting/type checks.
- **ruff + black + mypy (current plan)** — fast, consistent, strict types. Cons: type annotations effort. — Tier: S
- **flake8 + black** — lighter. Cons: fewer rules, no typing. — Tier: B
- **None** — fastest to start. Cons: debt piles quickly. — Tier: F

### 19) Background work model
**Overview**: Handling long-running or deferred tasks.
- **In-process asyncio tasks** — zero infra, good for small jobs. Cons: restarts drop tasks. — Tier: A
- **RQ/Celery (optional later)** — durable, scalable. Cons: infra, complexity. — Tier: B
- **Ad-hoc threads** — quick hack. Cons: brittle. — Tier: D

### 20) YAML schema validation
**Overview**: Validating rules/objects in YAML.
- **Pydantic models (current plan)** — reuse schemas, typed errors. Cons: mapping nested unions can be verbose. — Tier: S
- **jsonschema** — language-agnostic. Cons: separate models from code. — Tier: B

### 21) Project structure and routers
**Overview**: Organize modules and endpoints.
- **Modular per-domain packages (current plan)** — clear ownership, testable. Cons: more files. — Tier: S
- **Monolithic `app.py`** — quick start. Cons: scales poorly. — Tier: F

### 22) DB session lifecycle
**Overview**: How to manage SQLAlchemy sessions in requests.
- **Per-request session dependency** — safe, predictable cleanup. Cons: small boilerplate. — Tier: S
- **Global session** — minimal code. Cons: leaks, concurrency issues. — Tier: D

### 23) Migrations workflow
**Overview**: Generating and applying schema changes.
- **Alembic autogenerate then review** — fast iteration with safety. Cons: requires review habit. — Tier: S
- **Hand-written migrations only** — precise control. Cons: slow, error-prone. — Tier: B

### 24) Logging
**Overview**: Runtime logging approach.
- **Stdlib `logging` + JSON formatter** — simple, production-friendly. Cons: structured fields manual. — Tier: A
- **structlog** — great structured logs. Cons: extra dependency, learning curve. — Tier: A
- **Print debugging** — quickest. Cons: no levels/aggregation. — Tier: D

### 25) Rate limiting
**Overview**: Prevent abuse and accidental floods.
- **Starlette/FastAPI middleware (e.g., `slowapi`)** — easy, per-endpoint limits. Cons: memory store by default. — Tier: B
- **Edge (proxy) limits (nginx/traefik)** — robust at ingress. Cons: infra/config work. — Tier: B
- **None initially** — simplest now. Cons: risk if public. — Tier: C

### 26) CORS policy
**Overview**: Cross-origin configuration.
- **Explicit allowed origins (current plan)** — safe, controlled. Cons: needs env per deploy. — Tier: S
- **Wildcard `*`** — zero config. Cons: security risk. — Tier: D

### 27) Secrets management
**Overview**: How to store and load secrets.
- **.env locally + environment in prod (current plan)** — simple, clear. Cons: avoid committing .env. — Tier: S
- **Hosted secret manager (later)** — rotation, auditing. Cons: setup overhead now. — Tier: A

### 28) Containerization for dev
**Overview**: Local dev runtime.
- **Local venv (current plan)** — fastest iteration. Cons: machine differences. — Tier: S
- **Docker dev** — parity with prod. Cons: slower feedback, compose upkeep. — Tier: B

### 29) API docs exposure
**Overview**: How to expose OpenAPI docs.
- **FastAPI auto docs at `/docs`/`/redoc`** — instant, interactive. Cons: expose safely in prod. — Tier: S
- **External docs site** — polished. Cons: maintenance. — Tier: C

### 30) Error handling pattern
**Overview**: Standardize exceptions and responses.
- **HTTPException + global handlers** — simple, consistent 4xx/5xx shape. Cons: ensure coverage. — Tier: S
- **Ad-hoc `try/except` per route** — explicit. Cons: noisy, inconsistent. — Tier: C

### 31) Pagination defaults
**Overview**: Defaults and max limits.
- **Conservative defaults (e.g., `limit=50`, max 100)** — protects DB. Cons: sometimes more pages. — Tier: S
- **Large defaults** — fewer requests. Cons: heavier queries. — Tier: B

### 32) ID strategy
**Overview**: Primary keys for tables.
- **Integer autoincrement** — simple, fast. Cons: guessable IDs. — Tier: A
- **UUIDv4** — safer for external exposure. Cons: storage/index size. — Tier: A

### 33) Timestamps and timezone
**Overview**: Time fields policy.
- **UTC everywhere (current plan)** — consistent, portable. Cons: none meaningful. — Tier: S
- **Local times** — human-friendly. Cons: DST/offset bugs. — Tier: F

### 34) Soft delete strategy
**Overview**: Deleting records.
- **`deleted_at` nullable field (current plan)** — reversible, audit-friendly. Cons: query filters required. — Tier: S
- **Hard delete** — simpler. Cons: data loss, broken history. — Tier: C

### 35) API authentication method
**Overview**: Client auth to API.
- **Bearer JWT in `Authorization` header (current plan)** — standard, interop. Cons: manage expiry/refresh. — Tier: S
- **Session cookies** — simple for web-only. Cons: worse for multi-clients/bots. — Tier: B

---

How to use: pick one option per decision and we will update the implementation plan and scaffolding accordingly.


