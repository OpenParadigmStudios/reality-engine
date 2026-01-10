# Reality Engine — Spec Master Index

## How This Document Works

This is the central index for all design specifications. Each domain area links to its detailed spec doc.

**Status indicators:**
- 🔴 Not started — needs initial interrogation
- 🟡 In progress — has content, needs deepening
- 🟢 Complete — no open questions remain
- 🔄 Needs revision — downstream decisions may have invalidated something

**Workflow:**
1. Run `/interrogate spec/domains/<area>` to deepen any spec
2. Answer questions until the agent has no more to ask
3. Agent updates the spec doc, glossary, and this index
4. Repeat for next area

---

## Architectural Principles

> **Only immutable, approved World Events change game state.**
> Everything else exists to propose, gate, or explain those events.

**Domain separation:**
- **World Domain**: Immutable, event-sourced, deterministic, replayable
- **Workflow Domain**: Editable, commentable, approval-gated, never directly mutates world

**Key decisions:**
- Drafts are separate entities from Events (honors World/Workflow split)
- Events are append-only and immutable
- Triggers can emit direct Events OR Drafts (configurable per trigger)
- EventTypes are named semantic schemas; Events store both type + operations
- Game itself is a GameObject (can be targeted by signal Events)

---

## MVP Scope

### MVP 0: Single-Game Engine
- One Game instance, one GM, N players
- No multi-tenancy or shared users
- YAML paradigm ingestion (single directory)
- In-memory or SQLite persistence
- Simple auth system (automatic trust the caller or GM approval, set in schema)
- Simple Sqlite DB
- **Goal**: Prove the core loop works

### MVP 1: Multi-Game Platform
- Shared User accounts across games
- Multiple Games per instance
- Paradigm registry
- Proper auth and permissions
- Database-backed persistence
- **Goal**: Prove the architecture scales

See [mvp-scope.md](architecture/mvp-scope.md) for full details.

---

## Architecture Specs

| Area | Doc | Status | Notes |
|------|-----|--------|-------|
| System Overview | [overview.md](architecture/overview.md) | 🔴 | Philosophy, constraints, non-goals |
| Data Model | [data-model.md](architecture/data-model.md) | 🔴 | Entity relationships, storage approach |
| MVP Scope | [mvp-scope.md](architecture/mvp-scope.md) | 🟡 | MVP 0 vs MVP 1 boundaries |

---

## Domain Specs

Spec order (conceptual depth):
1. Events — the atomic primitive
2. GameObjects — what Events happen to
3. Actions & Drafts — how Events get proposed
4. Triggers — how Events cause more Events
5. Projections — derived read models
6. Permissions — who can do/see what
7. Paradigms — modular system definitions

| Area | Doc | Status | Key Open Questions |
|------|-----|--------|-------------------|
| Events | [events.md](domains/events.md) | 🟢 | — |
| GameObjects | [game-objects.md](domains/game-objects.md) | 🟡 | Expression language details; future inheritance/mixins |
| Actions & Drafts | [actions-drafts.md](actions-drafts.md) | 🔄 | Actions reference EventTypes; bidirectional Kind binding |
| Triggers | [triggers.md](domains/triggers.md) | 🔄 | Match on EventType name; expression DSL for guards |
| Projections | [projections.md](domains/projections.md) | 🔄 | Consume Operations; Status invalidation/caching |
| Permissions | [permissions.md](domains/permissions.md) | 🔴 | Selector syntax, ACL model, visibility filtering |
| Paradigms | [paradigms.md](domains/paradigms.md) | 🔄 | Kinds with schema format; default_objects; event_types |

---

## Implementation Specs

### MVP 0: Single-Game Engine

| Epic | Doc | Status | Blocked By |
|------|-----|--------|------------|
| Overview | [overview.md](implementation/mvp-0/overview.md) | 🔴 | Domain specs |
| Epic 1: Event Core | TBD | 🔴 | events.md ✅ |
| Epic 2: GameObject Core | TBD | 🔴 | game-objects.md |
| Epic 3: Action/Draft Flow | TBD | 🔴 | actions-drafts.md |
| Epic 4: Trigger Engine | TBD | 🔴 | triggers.md |
| Epic 5: Projections | TBD | 🔴 | projections.md |
| Epic 6: YAML Ingestion | TBD | 🔴 | paradigms.md |

### MVP 1: Multi-Game Platform

| Epic | Doc | Status | Blocked By |
|------|-----|--------|------------|
| Overview | [overview.md](implementation/mvp-1/overview.md) | 🔴 | MVP 0 complete |

---

## Dependency Graph

```
Events (primitive) ✅
    ↓
GameObjects (what Events target) 🟡
    ↓
Actions & Drafts (how Events are proposed) 🔄
    ↓
Triggers (how Events cause more Events) 🔄
    ↓
Projections (derived from Events) 🔄
    ↓
Permissions (gates Actions, filters Projections)
    ↓
Paradigms (packages all of the above) 🔄
```

---

## Recent Changes

### 2026-01-09: Events Spec Complete

Key decisions made:
- **EventType vs Event**: EventTypes are named semantic schemas (e.g., `DowntimePasses`). Events are instances that store both the type name AND the operations performed.
- **Two-layer model**: Semantic layer for trigger matching, Operations layer for projection updates.
- **8 operation types**: `object.create`, `object.update`, `object.archive`, `meter.delta`, `meter.set`, `ref.set`, `ref.add`, `ref.remove`
- **Meters and Refs**: Special field types declared in Kind schemas with dedicated operations.
- **Signal Events**: Events with no operations that target the Game object for global broadcasts.
- **Notes**: Moved to Workflow Domain as separate entity (not an operation type).
- **ID format**: ULID for global uniqueness + timestamp embedding.
- **Ordering**: Per-Game sequence number for strict ordering.
- **Causality**: Immediate parent + root draft tracking.

Specs needing revision due to Events decisions:
- `game-objects.md` — Kinds must support Meter and Ref field types
- `actions-drafts.md` — Actions reference EventTypes
- `triggers.md` — Match on EventType names
- `projections.md` — Consume Operations layer
- `paradigms.md` — Needs `event_types` section

### 2026-01-10: GameObjects Spec Deepened

Key decisions made:
- **ULID for GameObject IDs**: Consistent with Events
- **No Kind inheritance for MVP**: Standalone Kinds; future mixins/traits
- **Single Character Kind for PC/NPC**: Optional nullable fields for PC-specific data (player ref)
- **Spec vs Status**: Spec is authored data; Status holds computed values (cached, invalidated)
- **All GameObjects have optional `parent` ref**: Built-in hierarchy for containment
- **Actions: bidirectional binding**: Actions target Kinds AND Kinds list Actions
- **Dangling refs allowed with GM alert**: No cascade/block; GM resolves
- **Kind schema format defined**: Meters, Refs, nested objects, computed fields
- **Expression DSL introduced**: For meter bounds, computed status, conditions

Specs affected by GameObject decisions:
- `actions-drafts.md` — Bidirectional Action/Kind binding
- `triggers.md` — Expression DSL for conditions
- `projections.md` — Status caching and invalidation
- `paradigms.md` — Full Kind schema format, default_objects

---

## Glossary

See [glossary.md](glossary.md) for canonical definitions of all terms.

---

## Last Updated
_2026-01-10 — GameObjects spec interrogation in progress._
