# GameObjects — Domain Specification

**Status**: 🔴 Not started  
**Last interrogated**: Not yet  
**Depends on**: [events.md](events.md)  
**Depended on by**: [actions-drafts.md](actions-drafts.md), [triggers.md](triggers.md), [projections.md](projections.md)

---

## Overview

GameObjects are the base entity type—the "Entities" in the ECS model. They represent anything that exists in the game world: characters, factions, locations, clocks, bonds, resources, sessions, etc.

---

## What We Know (From PRD)

### Purpose
- Represents anything that exists in the world
- Carries structured data and components
- Serves as scope target for events and permissions

### Required Fields (From PRD)
- `id` — unique identifier
- `kind` — schema name (from Paradigm)
- `owner` — selector
- `acl` — selector-based permissions
- `spec` — authoritative data
- `status` — derived/projection data

---

## Open Questions

### Kind System
- [ ] How are Kinds defined? Schema format?
- [ ] Kind inheritance or composition?
- [ ] Can Kinds change after object creation?

### Spec vs Status
- [ ] What lives in `spec` vs `status`?
- [ ] How is `status` computed/updated?
- [ ] Validation of `spec` against Kind schema?

### Object Lifecycle
- [ ] Soft delete vs hard delete?
- [ ] Archive/inactive states?
- [ ] Object versioning?

### Relationships
- [ ] How do objects reference each other?
- [ ] Hierarchical containment (locations containing locations)?
- [ ] Link semantics (typed relationships)?

### Components
- [ ] What are "components" in this system?
- [ ] How do Meters, Tags, etc. attach to objects?
- [ ] Component schema validation?

---

## Decisions Made

*None yet — awaiting interrogation.*

---

_This document will be updated during `/interrogate spec/domains/game-objects` sessions._
