# Events — Domain Specification

**Status**: 🟢 Complete
**Last interrogated**: 2026-01-09
**Depends on**: None (primitive)
**Depended on by**: [game-objects.md](game-objects.md), [actions-drafts.md](actions-drafts.md), [triggers.md](triggers.md), [projections.md](projections.md)

---

## Overview

Events are the atomic primitive of the Reality Engine. They are the **sole mechanism** for changing world state. Every mutation to the game world is recorded as an immutable Event, forming a complete, ordered history that can be replayed deterministically.

**Architectural principle**: Only immutable, approved World Events change game state.

---

## Core Concepts

### EventType vs Event

- **EventType**: A named schema defined in a Paradigm. Specifies targets, parameters, and what operations to perform. Examples: `DowntimePasses`, `ProgressFaction`, `HealCharacter`.

- **Event**: An instantiated EventType. Records the type name, targets, parameters, the atomic operations performed, and metadata. Immutable once created.

### Two-Layer Model

1. **Semantic Layer**: The EventType name gives meaning (`DowntimePasses`, `TakeDamage`). Triggers primarily match on this layer.

2. **Operations Layer**: The atomic changes applied (`meter.delta on HP`, `ref.add to bonds`). Projections consume this layer to update state.

Both layers are stored on every Event.

---

## Decisions

### Event Identity

- **Decision**: Events use ULID for unique identifier
- **Rationale**: Lexicographically sortable, embeds timestamp, globally unique across Games
- **Implications**: Can sort by ID for rough chronological order; ID encodes creation time

### Event Ordering

- **Decision**: Strict ordering via per-Game sequence number
- **Rationale**: Each Game maintains a monotonic counter. Events get `(game_id, seq)` as their ordering key
- **Implications**: Replay is deterministic; ULID provides secondary uniqueness; sequence number is the authoritative order

### Multi-Object Targeting

- **Decision**: Events have `scope.primary` + `scope.refs`
- **Rationale**: One primary target object with optional references to other affected objects
- **Implications**: `ref.add` has one object as primary, the other in refs; Triggers can match on both primary and refs

### Payload Storage

- **Decision**: Store EventType + parameters AND the atomic operations performed
- **Rationale**: Semantic type enables trigger matching; operations enable projection rebuilds
- **Implications**: Events are self-contained; can replay without re-expanding EventType definitions

### Previous Values

- **Decision**: Do not store previous values
- **Rationale**: Previous state can be computed by replaying events up to that point
- **Implications**: Simpler, smaller payloads; full state reconstruction requires replay

### Payload Validation

- **Decision**: Typed schema per EventType (Pydantic)
- **Rationale**: Catches errors at Event creation time
- **Implications**: Each EventType in Paradigm YAML defines its parameter schema

### EventType Definition

- **Decision**: Paradigm-only for MVP; EventTypes are mutable (latest definition wins on replay)
- **Rationale**: No runtime EventType creation keeps model simple; accepting potential drift from EventType changes
- **Implications**: Paradigm YAML has `event_types` section; changes to EventType affect replay of old Events

### Causality Tracking

- **Decision**: Track immediate parent + root only
- **Rationale**: `parent_event_id` (what triggered this) and `root_draft_id` (originating action) are stored; full chain is reconstructible
- **Implications**: Can answer "what action caused this cascade" and "what's the trigger chain"

### Actor Tracking

- **Decision**: Required actor reference on every Event
- **Rationale**: Essential for audit trails and permission verification
- **Implications**: Actor is user_id, 'system', or 'trigger:{trigger_id}'

### Narrative Field

- **Decision**: Optional narrative field with template rendering
- **Rationale**: EventTypes can define a template string that renders to human-readable description
- **Implications**: Activity feeds can show stored narrative; not required for all Events

### Tags

- **Decision**: Optional tag list
- **Rationale**: Useful for filtering activity feeds and additional trigger matching dimensions
- **Implications**: Tags can be defined in EventType or added at instantiation

### Signal Events

- **Decision**: Events with no operations target the Game object itself
- **Rationale**: The Game is a GameObject. Global events like `DowntimePasses` target it
- **Implications**: Triggers on other objects watch for events scoped to Game

### Session Association

- **Decision**: Optional session_id field
- **Rationale**: Useful for grouping by play date, but some Events happen outside sessions
- **Implications**: Session is a separate entity that Events can reference

---

## Operation Types

These are the atomic primitives that EventTypes compose to express mutations:

| Operation | Description | Payload |
|-----------|-------------|---------|
| `object.create` | Instantiate new GameObject | `kind`, `initial_data` |
| `object.update` | Modify data field(s) on GameObject | `path`, `value` |
| `object.archive` | Soft-delete GameObject | (none) |
| `meter.delta` | Adjust numeric meter by amount | `meter`, `delta` |
| `meter.set` | Set numeric meter to absolute value | `meter`, `value` |
| `ref.set` | Set a singular reference field | `field`, `target_id` |
| `ref.add` | Add to a list reference field | `field`, `target_id` |
| `ref.remove` | Remove from a list reference field | `field`, `target_id` |

**Schema-aware operations**: Meters and Refs are declared as special field types in the GameObject Kind schema. Operations on these fields use the specialized ops above, enabling validation, indexing, and trigger matching.

**Removed from original list**:
- `link.add`/`link.remove` — replaced by ref operations
- `text.annotate` — Notes moved to Workflow Domain as separate entity
- `component.set` — subsumed by `object.update` (components are just nested data)

---

## Event Schema

```python
from datetime import datetime
from typing import Any

class Event:
    # Identity
    id: str                      # ULID
    game_id: str                 # Which Game
    seq: int                     # Monotonic sequence within Game

    # Semantic Layer
    type: str                    # EventType name (e.g., "DowntimePasses")
    parameters: dict[str, Any]  # EventType-specific parameters

    # Operations Layer
    operations: list[Operation]  # Atomic changes performed

    # Targeting
    scope: EventScope            # Primary object + refs

    # Lineage
    causality: Causality         # Parent event, root draft

    # Metadata
    actor: ActorRef              # Who/what caused this
    timestamp: datetime          # When created
    session_id: str | None       # Associated session
    narrative: str | None        # Human-readable description
    tags: list[str]              # Filtering labels


class EventScope:
    primary: str                 # Main object ID (can be Game ID for signals)
    refs: list[str]              # Other affected objects


class Causality:
    parent_event_id: str | None  # Immediate trigger (None if root)
    root_draft_id: str | None    # Originating action's Draft
    depth: int                   # How deep in cascade (0 = root)


class ActorRef:
    type: str                    # 'user', 'system', 'trigger'
    id: str                      # user_id or trigger_id


class Operation:
    op: str                      # Operation type (e.g., "meter.delta")
    target: str                  # Object ID being mutated
    payload: dict[str, Any]      # Operation-specific data
```

---

## Operation Payload Schemas

### object.create
```yaml
payload:
  kind: string           # Kind name
  initial_data: object   # Starting data conforming to Kind schema
```

### object.update
```yaml
payload:
  path: string           # JSON path to field (e.g., "spec.name", "status.wounds")
  value: any             # New value
```

### object.archive
```yaml
payload: {}              # No additional data; target is in Operation.target
```

### meter.delta
```yaml
payload:
  meter: string          # Meter field name
  delta: number          # Amount to add (can be negative)
```

### meter.set
```yaml
payload:
  meter: string          # Meter field name
  value: number          # Absolute value to set
```

### ref.set
```yaml
payload:
  field: string          # Ref field name
  target_id: string      # Object ID to reference (or null to clear)
```

### ref.add
```yaml
payload:
  field: string          # Ref list field name
  target_id: string      # Object ID to add
```

### ref.remove
```yaml
payload:
  field: string          # Ref list field name
  target_id: string      # Object ID to remove
```

---

## EventType Definition (Paradigm YAML)

EventTypes are defined in Paradigm files:

```yaml
event_types:
  DowntimePasses:
    description: "Signals that a downtime phase has begun"
    targets:
      primary: Game          # Must target the Game object
    parameters: {}           # No parameters
    operations: []           # Signal only, no mutations
    narrative_template: "Downtime begins"
    tags: [downtime, phase]

  ProgressFaction:
    description: "Advances a faction's project progress"
    targets:
      primary: Faction       # Must target a Faction
    parameters:
      amount:
        type: integer
        default: 1
      reason:
        type: string
        optional: true
    operations:
      - op: meter.delta
        target: $primary
        payload:
          meter: project_progress
          delta: $amount
    narrative_template: "{{primary.name}} advances project by {{amount}}{{#reason}} ({{reason}}){{/reason}}"
    tags: [faction, progress]
```

---

## Cascade Example

**Scenario**: GM invokes "Start Downtime" action

1. **Action Invoked**: GM clicks "Start Downtime" targeting the Game
2. **Draft Created**: Draft with EventType `DowntimePasses`
3. **Draft Approved**: GM approves (or auto-approved)
4. **Event Emitted**:
   ```
   Event(
     type="DowntimePasses",
     scope={primary: game_id, refs: []},
     operations=[],  # Signal only
     causality={parent: None, root_draft: draft_id, depth: 0}
   )
   ```
5. **Triggers Fire**: Each Faction has a Trigger watching for `DowntimePasses` on Game
6. **Cascade Events**: Each Faction's Trigger emits a `ProgressFaction` Event:
   ```
   Event(
     type="ProgressFaction",
     scope={primary: faction_id, refs: [game_id]},
     operations=[
       Operation(op="meter.delta", target=faction_id, payload={meter: "project_progress", delta: 1})
     ],
     causality={parent: downtime_event_id, root_draft: draft_id, depth: 1}
   )
   ```

---

## Storage Considerations (MVP 0)

- **Partitioning**: By Game (single table with game_id, single-game instance anyway)
- **Indexing**: By game_id + seq (primary), by type, by scope.primary, by timestamp range
- **Retention**: Keep all Events forever (they are the source of truth)
- **Format**: SQLite table with JSON columns for complex fields

---

## Implications for Other Specs

| Spec | Implication |
|------|-------------|
| [game-objects.md](game-objects.md) | Kinds must declare Meter and Ref field types in schema |
| [actions-drafts.md](actions-drafts.md) | Actions reference EventTypes; Drafts hold EventType + parameters |
| [triggers.md](triggers.md) | Triggers match on EventType name (semantic) + can filter on operations |
| [projections.md](projections.md) | Projections consume Operation layer to update state |
| [paradigms.md](paradigms.md) | Paradigm YAML needs `event_types` section |

---

_Interrogation complete. This document reflects decisions made 2026-01-09._
