# MVP Scope Definition

This document defines the boundaries between MVP 0 (Single-Game Engine) and MVP 1 (Multi-Game Platform).

---

## MVP 0: Single-Game Engine

### Goal
Prove the core loop works: Action → Draft → Approval → Event → Trigger → Cascade → Projection.

### Scope

#### In Scope
- **Single Game instance** per running process
- **Single GM** with full authority
- **N Players** with character associations
- **Full event loop**: Actions, Drafts, Events, Triggers, Projections
- **YAML paradigm ingestion** from a local directory
- **CLI or simple HTTP API** for interaction
- **SQLite or in-memory** persistence
- **Deterministic replay** from event log
- **Basic activity feed** (GM sees all, players see filtered)

#### Out of Scope
- Multi-tenancy (multiple Games in one instance)
- User accounts shared across Games
- Paradigm registry or management UI
- Authentication/authorization system
- Real-time sync or websockets
- Performance optimization
- Cross-game analytics
- AI assistance features

### Deployment Model
- Run N separate instances for N games
- Each instance loads one paradigm directory
- No shared state between instances
- Could be Docker containers, separate processes, or serverless functions

### Success Criteria
- [ ] Can instantiate a Game from YAML paradigm
- [ ] Can create GameObjects of defined Kinds
- [ ] Can invoke Actions that produce Drafts
- [ ] GM can approve/reject Drafts
- [ ] Approved Drafts emit Events
- [ ] Triggers fire on matching Events
- [ ] Cascades propagate correctly with depth limits
- [ ] Projections reflect current state
- [ ] Full event log can replay to identical state
- [ ] Blades in the Dark crew downtime is modelable
- [ ] A structurally different game (e.g., Ars Magica covenant) is modelable

### Technical Decisions (MVP 0)

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Persistence | SQLite file | Simple, portable, sufficient for single-game |
| API | FastAPI REST | Clean, typed, good for CLI and future clients |
| Auth | None (trust caller) | Simplifies MVP; add in MVP 1 |
| Paradigm loading | Directory scan at startup | No runtime paradigm changes |
| Event storage | Append-only table | Simple event sourcing |
| Projections | Rebuild on query or startup | No caching complexity |

---

## MVP 1: Multi-Game Platform

### Goal
Prove the architecture scales to a multi-tenant platform with proper isolation and user management.

### Scope (Incremental from MVP 0)

#### Adds
- **Shared User accounts** across Games
- **Multiple Games** per instance with proper isolation
- **Paradigm registry** (upload, version, select)
- **Authentication** (OAuth or token-based)
- **Authorization** (role-based, selector-driven)
- **Multi-GM support** with coordination tools
- **PostgreSQL** persistence
- **API versioning** for client stability

#### Still Out of Scope
- Real-time sync
- VTT features
- AI generation
- Performance optimization beyond correctness

### Success Criteria
- [ ] Users can create accounts and join multiple Games
- [ ] Games are fully isolated (no data leakage)
- [ ] Paradigms can be uploaded and versioned
- [ ] Multiple GMs can coordinate on a single Game
- [ ] Permission system enforces visibility correctly
- [ ] API is versioned and stable

---

## Migration Path: MVP 0 → MVP 1

### Data Migration
- MVP 0 SQLite files can be imported into MVP 1 PostgreSQL
- Event logs are the source of truth; projections rebuild

### Code Evolution
- MVP 0 code should avoid hard-coding single-game assumptions
- Use Game ID scoping even in MVP 0 (just always the same ID)
- Auth middleware is a no-op in MVP 0, real in MVP 1

### Paradigm Evolution
- MVP 0 YAML format is the canonical format for MVP 1
- MVP 1 adds metadata (author, version, dependencies) but core schema unchanged

---

## Phasing Rationale

**Why start with MVP 0?**
- Proves the hard part (event loop, triggers, cascades) without platform complexity
- Lets you run real games sooner
- Validates paradigm abstractions with real use cases
- Failures are cheap (restart with different YAML)

**Why not skip to MVP 1?**
- Platform concerns (auth, multi-tenancy) are distracting
- Risk of over-engineering before validating core model
- Harder to debug cascade issues with more moving parts

---

## Open Questions

- [ ] Should MVP 0 support multiple paradigm directories (combine paradigms)?
  - **Tentative**: Yes, but no conflict resolution—just fail on conflict
- [ ] Should MVP 0 persist workflow events (draft lifecycle)?
  - **Tentative**: Yes, for auditability, but not required for core loop
- [ ] What's the CLI interface for MVP 0?
  - **Tentative**: Simple REPL or HTTP API with curl-friendly endpoints

---

_This document is updated as scope decisions are made during spec interrogation._
