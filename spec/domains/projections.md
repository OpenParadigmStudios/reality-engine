# Projections — Domain Specification

**Status**: 🔴 Not started  
**Last interrogated**: Not yet  
**Depends on**: [events.md](events.md), [game-objects.md](game-objects.md), [triggers.md](triggers.md)  
**Depended on by**: [permissions.md](permissions.md) (for visibility filtering)

---

## Overview

Projections are read models derived from the Event log. They compute "current state" for efficient queries while remaining fully rebuildable from Events.

---

## What We Know (From PRD)

### Purpose
- Compute "current state" from event history
- Provide efficient queries
- Be fully rebuildable from event log

### MVP Scope (From PRD)
- Latest state per GameObject
- Simple activity feed
- No caching requirements (MVP)

---

## Open Questions

### Projection Types
- [ ] What projections does MVP need?
- [ ] Per-object state (current spec/status)?
- [ ] Aggregate projections (counts, sums)?
- [ ] Relationship graphs?
- [ ] Timeline/history views?

### Materialization Strategy
- [ ] Rebuild on every query?
- [ ] Rebuild on startup?
- [ ] Incremental update on new Events?
- [ ] Cached with invalidation?

### Feed Projections
- [ ] GM feed structure (all Events)?
- [ ] Player feed structure (filtered by visibility)?
- [ ] Pagination approach?
- [ ] Feed filtering (by object, time, type)?

### Consistency
- [ ] When do projections update relative to Events?
- [ ] Eventual consistency acceptable?
- [ ] Read-your-writes guarantee?

### Rebuild
- [ ] Full replay from scratch?
- [ ] Snapshotting for faster rebuild?
- [ ] Validation that projection matches replay?

---

## Decisions Made

*None yet — awaiting interrogation.*

---

_This document will be updated during `/interrogate spec/domains/projections` sessions._
