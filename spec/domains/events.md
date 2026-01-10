# Events — Domain Specification

**Status**: 🟡 In progress  
**Last interrogated**: Not yet  
**Depends on**: None (primitive)  
**Depended on by**: [game-objects.md](game-objects.md), [actions-drafts.md](actions-drafts.md), [triggers.md](triggers.md), [projections.md](projections.md)

---

## Overview

Events are the atomic primitive of the Reality Engine. They are the **sole mechanism** for changing world state. Every mutation to the game world is recorded as an immutable Event, forming a complete, ordered history that can be replayed deterministically.

**Architectural principle**: Only immutable, approved World Events change game state.

---

## What We Know (From PRD)

### Event Purpose
- Atomic, immutable record of world state change
- Append-only (never modified or deleted)
- Only **approved** events affect projections or trigger execution
- Forms complete campaign history
- Enables deterministic replay

### Required Fields (From PRD)
- `id` — unique identifier
- `game_id` — which Game this Event belongs to
- `type` — canonical event type
- `timestamp` — when the Event was created
- `scope` — primary object + references
- `payload` — event-specific data
- `causality` — parent event / root draft
- `approval_state` — only approved Events take effect

### Canonical Event Types (From PRD)
- `object.create` — new GameObject instantiated
- `object.update` — GameObject spec modified
- `component.set` — component value changed
- `meter.delta` — numeric value adjusted by amount
- `meter.set` — numeric value set absolutely
- `link.add` — relationship created
- `link.remove` — relationship removed
- `text.annotate` — narrative annotation added

---

## Open Questions

### Event Identity & Ordering

- [ ] **ID format**: UUID? ULID? Sequential integer? 
  - *Consideration*: ULID gives time-ordering + uniqueness
- [ ] **Ordering guarantees**: Strict total order? Logical clocks? Wall-clock timestamp?
  - *Consideration*: Affects replay determinism and multi-client scenarios

### Scope & Targeting

- [ ] **Scope structure**: Just primary object ID? List of affected objects? Hierarchical?
  - *Consideration*: Affects trigger matching and projection updates
- [ ] **Cross-object events**: Can one Event affect multiple objects directly?
  - *Consideration*: `link.add` naturally involves two objects

### Payload Schema

- [ ] **Payload structure**: Freeform JSON? Typed per event type? 
- [ ] **Before/after capture**: Store previous value? Just the change? Neither?
  - *Consideration*: Affects undo capability and debugging
- [ ] **Payload validation**: Strict schema per type? Or loose?

### Causality Tracking

- [ ] **Causality depth**: Track immediate parent only? Full chain? Depth limit?
- [ ] **Root tracking**: Always link back to originating Draft?
- [ ] **Cascade identification**: How to identify all Events from one Action?
  - *Consideration*: Useful for "undo action" and debugging

### Approval Model

- [ ] **Approval states**: Binary (approved/not)? Or include pending, rejected?
  - *Note*: PRD suggests Drafts handle pending/rejected, Events are only created when approved
- [ ] **Retroactive rejection**: Can an approved Event ever be "undone"?
  - *Consideration*: Affects immutability guarantee

### Event Metadata

- [ ] **Actor tracking**: Who/what caused this Event?
- [ ] **Session association**: Link to Session object?
- [ ] **Narrative text**: Structured field for human-readable description?
- [ ] **Tags/labels**: For filtering and categorization?

### Storage & Performance

- [ ] **Partitioning**: By Game? By time? By object?
- [ ] **Retention**: Keep all Events forever? Archival strategy?
- [ ] **Indexing**: What queries must be fast?
  - *Candidates*: By game, by object, by type, by time range, by causality chain

---

## Decisions Made

*None yet — awaiting interrogation.*

---

## Event Schema (Draft)

```python
# Tentative — to be refined during interrogation

class Event:
    id: str                    # ULID? UUID?
    game_id: str
    type: str                  # From canonical vocabulary
    timestamp: datetime        # When created
    
    # Targeting
    scope: EventScope          # Primary object + refs
    
    # Data
    payload: dict              # Type-specific data
    
    # Lineage
    causality: Causality       # Parent event, root draft
    
    # Metadata
    actor: ActorRef            # Who/what caused this
    session_id: str | None     # Associated session
    narrative: str | None      # Human-readable description
    tags: list[str]            # Filtering labels


class EventScope:
    primary: str               # Main object ID
    refs: list[str]            # Other affected objects
    # Or more structured?


class Causality:
    parent_event_id: str | None
    root_draft_id: str | None
    depth: int                 # How deep in cascade
    # Full chain? Or just immediate parent?
```

---

## Canonical Event Types — Details

Each type needs its payload schema defined:

### object.create
```yaml
payload:
  kind: string       # Kind of object created
  initial_spec: {}   # Starting data
```

### object.update  
```yaml
payload:
  path: string       # JSON path to updated field?
  previous: any      # Old value (optional?)
  value: any         # New value
```

### component.set
```yaml
payload:
  component: string  # Component name
  key: string        # Field within component
  value: any
```

### meter.delta
```yaml
payload:
  meter: string      # Meter name
  delta: number      # Amount to add (can be negative)
  # Result included? Or computed?
```

### meter.set
```yaml
payload:
  meter: string
  value: number
  previous: number   # Optional?
```

### link.add
```yaml
payload:
  link_type: string  # Type of relationship
  target: string     # Other object ID
  # Direction? Bidirectional?
```

### link.remove
```yaml
payload:
  link_type: string
  target: string
```

### text.annotate
```yaml
payload:
  text: string       # The annotation
  # Attachment point? Just on scope.primary?
```

---

## Notes for Interrogation

Key areas to probe:
1. **Event ordering** — critical for determinism
2. **Scope semantics** — affects trigger matching
3. **Payload schemas** — affects validation and projection logic
4. **Causality tracking** — affects debugging and undo
5. **Metadata needs** — what do we need for feeds and queries?

---

_This document will be updated during `/interrogate spec/domains/events` sessions._
