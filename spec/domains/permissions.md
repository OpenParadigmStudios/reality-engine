# Permissions — Domain Specification

**Status**: 🔴 Not started  
**Last interrogated**: Not yet  
**Depends on**: [game-objects.md](game-objects.md), [actions-drafts.md](actions-drafts.md)  
**Depended on by**: [paradigms.md](paradigms.md)

---

## Overview

The permissions system controls who can do what and see what. It uses selector-based rules that can reference roles, groups, ownership, and object data.

---

## What We Know (From PRD)

### Selector-Based Model
ACLs and approvals target **selectors**, not users directly.

Example selectors:
- `group:game:gm` — all GMs in this game
- `group:game:players` — all players
- `expr:user == owner` — the object's owner
- `expr:user in self.spec.members` — users listed in object data

### Approval Rules (From PRD)
- If actor has `write` permission → auto-approve
- Otherwise → draft requires approval from selector set

### Visibility Model (From PRD)
- GM sees all campaign objects and events
- Players see filtered view based on:
  - Own characters
  - Groups they're part of
  - Designated bonds
  - Global/public events

---

## Open Questions

### Selector Syntax
- [ ] Full selector grammar?
- [ ] How are selectors evaluated?
- [ ] Performance implications of complex selectors?
- [ ] Selector validation/testing?

### Permission Types
- [ ] What permission types exist? (read, write, action, approve?)
- [ ] Granularity (per-object? per-field? per-action?)
- [ ] Default permissions when none specified?

### ACL Structure
- [ ] Where do ACLs live? On objects? Separate table?
- [ ] ACL inheritance (from Kind? from container?)?
- [ ] ACL override/merge semantics?

### Visibility Filtering
- [ ] How is feed visibility computed?
- [ ] Object visibility vs Event visibility?
- [ ] Partial visibility (see object exists, not its contents)?
- [ ] Time-delayed visibility?

### GM Authority
- [ ] Can GM permissions be limited?
- [ ] Head GM vs regular GM distinction?
- [ ] GM impersonation of players?

---

## Decisions Made

*None yet — awaiting interrogation.*

---

_This document will be updated during `/interrogate spec/domains/permissions` sessions._
