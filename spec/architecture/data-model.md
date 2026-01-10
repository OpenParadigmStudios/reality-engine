# Data Model

**Status**: 🔴 Not started  
**Last interrogated**: Not yet  
**Blocked by**: Domain specs (events, game-objects, actions-drafts, triggers)

---

## Purpose

This document describes the entity relationships, storage approach, and data architecture for Reality Engine.

---

## Entity Overview

```
Game
├── GameMemberships (User roles)
├── GameObjects (by Kind)
│   ├── Characters
│   ├── Factions
│   ├── Locations
│   ├── Meters
│   └── ...
├── Events (append-only log)
├── Drafts (pending proposals)
├── Sessions (play session containers)
└── Projections (derived views)
```

---

## Open Questions

- [ ] Polymorphism approach for GameObjects
- [ ] JSON vs relational for spec/status data
- [ ] Event partitioning strategy
- [ ] Projection materialization approach

---

## Decisions Made

*None yet — blocked by domain specs.*

---

_This document will be updated after domain specs are complete._
