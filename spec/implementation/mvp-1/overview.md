# MVP 1: Multi-Game Platform — Implementation Overview

**Status**: 🔴 Not started  
**Blocked by**: MVP 0 completion

---

## Goal

Evolve the single-game engine into a multi-tenant platform with proper isolation, authentication, and paradigm management.

---

## Scope Summary

### Adds (From MVP 0)
- Shared User accounts across Games
- Multiple Games per instance
- Paradigm registry
- Authentication (OAuth/token)
- Authorization (role-based, selector-driven)
- Multi-GM support
- PostgreSQL persistence
- API versioning

### Still Out
- Real-time sync
- VTT features
- AI generation
- Performance optimization beyond correctness

See [mvp-scope.md](../architecture/mvp-scope.md) for full details.

---

## Epic Structure

TBD after MVP 0 completion.

Anticipated areas:
- User management & authentication
- Game creation & management
- Paradigm registry
- Multi-GM coordination
- Permission system hardening
- Database migration (SQLite → PostgreSQL)

---

## Success Criteria

- [ ] Users can create accounts
- [ ] Users can join multiple Games
- [ ] Games are fully isolated
- [ ] Paradigms can be uploaded and versioned
- [ ] Multiple GMs can coordinate
- [ ] Permission system enforces visibility
- [ ] API is versioned and stable

---

_This document will be populated after MVP 0 is complete._
