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

---

## MVP Scope

### MVP 0: Single-Game Engine
- One Game instance, one GM, N players
- No multi-tenancy or shared users
- YAML paradigm ingestion (single directory)
- In-memory or SQLite persistence
- No auth system (trust the caller)
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
| Events | [events.md](domains/events.md) | 🟡 | Canonical event types, payload schemas, causality tracking |
| GameObjects | [game-objects.md](domains/game-objects.md) | 🔴 | Kind schema, spec vs status, polymorphism |
| Actions & Drafts | [actions-drafts.md](domains/actions-drafts.md) | 🔴 | ActionDef structure, Draft lifecycle, approval policies |
| Triggers | [triggers.md](domains/triggers.md) | 🔴 | Match syntax, guard conditions, emission rules, cascade limits |
| Projections | [projections.md](domains/projections.md) | 🔴 | What projections MVP needs, rebuild strategy |
| Permissions | [permissions.md](domains/permissions.md) | 🔴 | Selector syntax, ACL model, visibility filtering |
| Paradigms | [paradigms.md](domains/paradigms.md) | 🔴 | YAML schema, dependency resolution, conflict detection |

---

## Implementation Specs

### MVP 0: Single-Game Engine

| Epic | Doc | Status | Blocked By |
|------|-----|--------|------------|
| Overview | [overview.md](implementation/mvp-0/overview.md) | 🔴 | Domain specs |
| Epic 1: Event Core | TBD | 🔴 | events.md |
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
Events (primitive)
    ↓
GameObjects (what Events target)
    ↓
Actions & Drafts (how Events are proposed)
    ↓
Triggers (how Events cause more Events)
    ↓
Projections (derived from Events)
    ↓
Permissions (gates Actions, filters Projections)
    ↓
Paradigms (packages all of the above)
```

---

## Glossary

See [glossary.md](glossary.md) for canonical definitions of all terms.

---

## Last Updated
_This document is maintained by the `/interrogate` command._
