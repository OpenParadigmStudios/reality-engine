# Architecture Overview

**Status**: 🔴 Not started  
**Last interrogated**: Not yet

---

## Purpose

This document describes the high-level architecture of Reality Engine: the system philosophy, domain boundaries, key constraints, and non-goals.

---

## System Philosophy

> **Only immutable, approved World Events change game state.**
> Everything else exists to propose, gate, or explain those events.

### Narrative ECS Model

Reality Engine treats a tabletop campaign as a living narrative system, borrowing concepts from:
- **ECS (Entity-Component-System)**: GameObjects as entities, data as components, triggers as systems
- **Event Sourcing**: All state changes as an append-only event log
- **Declarative Configuration**: Rules and schemas defined in YAML, not code

### Domain Separation

| Domain | Purpose | Properties |
|--------|---------|------------|
| **World** | Authoritative game state | Immutable, event-sourced, deterministic, replayable |
| **Workflow** | Human intent & collaboration | Editable, commentable, approval-gated |

These domains **must remain distinct**. Workflow never directly mutates World.

---

## Constraints

- **Declarative first**: No arbitrary user code execution in v1
- **Determinism**: Given the same event log, replay produces identical state
- **GM authority**: GM retains final say via approvals and overrides
- **Blast-radius control**: Cascades have explicit limits
- **Modularity**: Game systems are defined in portable Paradigms

---

## Non-Goals

- Virtual tabletop features (maps, tokens, live dice)
- Real-time synchronization
- AI-generated content (v1)
- Game-specific mechanics hardcoded in backend
- Performance optimization beyond correctness

---

## Open Questions

*To be populated during interrogation.*

---

## Decisions Made

*None yet — awaiting interrogation.*

---

_This document will be updated during `/interrogate spec/architecture/overview` sessions._
