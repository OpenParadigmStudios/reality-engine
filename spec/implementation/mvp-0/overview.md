# MVP 0: Single-Game Engine — Implementation Overview

**Status**: 🔴 Not started  
**Blocked by**: Domain specs completion

---

## Goal

Build the smallest coherent backend that proves the core loop works:

```
Action → Draft → Approval → Event → Trigger → Cascade → Projection
```

---

## Scope Summary

### In
- Single Game instance per process
- Single GM, N players
- Full event loop with cascades
- YAML Paradigm ingestion
- SQLite persistence
- CLI or simple HTTP API
- Deterministic replay

### Out
- Multi-tenancy
- Authentication
- Performance optimization
- Real-time sync
- AI features

See [mvp-scope.md](../architecture/mvp-scope.md) for full details.

---

## Epic Structure

| Epic | Focus | Stories (est.) | Depends On |
|------|-------|----------------|------------|
| 1 | Event Core | ~5 | events.md |
| 2 | GameObject Core | ~5 | game-objects.md |
| 3 | Action/Draft Flow | ~5 | actions-drafts.md |
| 4 | Trigger Engine | ~5 | triggers.md |
| 5 | Projections | ~3 | projections.md |
| 6 | YAML Ingestion | ~3 | paradigms.md |

---

## Epics (To Be Detailed)

### Epic 1: Event Core
*Blocked by: [events.md](../domains/events.md)*

Establish the immutable event log and basic event operations.

Stories TBD after events.md interrogation.

### Epic 2: GameObject Core
*Blocked by: [game-objects.md](../domains/game-objects.md)*

Implement the base entity system and Kind schema.

Stories TBD after game-objects.md interrogation.

### Epic 3: Action/Draft Flow
*Blocked by: [actions-drafts.md](../domains/actions-drafts.md)*

Implement ActionDef loading, Draft creation, and approval workflow.

Stories TBD after actions-drafts.md interrogation.

### Epic 4: Trigger Engine
*Blocked by: [triggers.md](../domains/triggers.md)*

Implement trigger matching, guard evaluation, and cascade execution.

Stories TBD after triggers.md interrogation.

### Epic 5: Projections
*Blocked by: [projections.md](../domains/projections.md)*

Implement state projections and feeds.

Stories TBD after projections.md interrogation.

### Epic 6: YAML Ingestion
*Blocked by: [paradigms.md](../domains/paradigms.md)*

Implement Paradigm loading and validation.

Stories TBD after paradigms.md interrogation.

---

## Success Criteria

- [ ] Can instantiate a Game from YAML paradigm
- [ ] Can create GameObjects of defined Kinds
- [ ] Can invoke Actions that produce Drafts
- [ ] GM can approve/reject Drafts
- [ ] Approved Drafts emit Events
- [ ] Triggers fire on matching Events
- [ ] Cascades propagate with depth limits
- [ ] Projections reflect current state
- [ ] Full event log replays to identical state
- [ ] Blades in the Dark crew downtime is modelable
- [ ] A structurally different game is modelable

---

_This document will be populated as domain specs are completed._
