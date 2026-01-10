# GameObjects — Domain Specification

**Status**: 🟡 In progress
**Last interrogated**: 2026-01-10
**Depends on**: [events.md](events.md)
**Depended on by**: [actions-drafts.md](actions-drafts.md), [triggers.md](triggers.md), [projections.md](projections.md)

---

## Overview

GameObjects are the base entity type—the "Entities" in the ECS model. They represent anything that exists in the game world: characters, factions, locations, clocks, bonds, resources, sessions, the Game itself, etc.

**Key principle**: GameObjects are the targets of Events. Every mutation happens via Events, and GameObjects are what get mutated.

---

## Decisions

### GameObject Identity

- **Decision**: GameObjects use ULID for unique identifier
- **Rationale**: Consistent with Events; sortable, embeds timestamp, globally unique
- **Implications**: Can sort by ID for rough creation order; IDs are opaque but consistent across the system

### Kind System

- **Decision**: No inheritance for MVP; each Kind is standalone
- **Rationale**: Simpler implementation; shared structure achieved through similar field definitions
- **Implications**: Future: add mixins/traits or single inheritance. For now, copy shared fields across Kinds.

### Character Modeling (PC vs NPC)

- **Decision**: Single Character Kind with optional PC-specific fields
- **Rationale**: Unified treatment for shared mechanics (combat, social, etc.); nullable fields for player association
- **Implications**: PC-only fields like `player` ref are optional. Null handling must be graceful throughout.

### Player-Character Link

- **Decision**: Ref field on Character pointing to Player (for MVP)
- **Rationale**: Simple and explicit; Character knows its player, query is direct
- **Implications**: NPCs have null player. Future: separate association entity for more complex scenarios.

### Spec vs Status

- **Decision**: Spec is authored data; Status holds all computed/derived values
- **Rationale**: Clean separation; spec is what you set, status is what the system computes
- **Implications**: Meter max values, computed properties, cached aggregates all live in status

### Status Recomputation

- **Decision**: Hybrid caching with invalidation
- **Rationale**: Balance between always-fresh and performance
- **Implications**: Status is cached. Events mark it dirty. Recomputed on next read or in background process.

### Hierarchical Containment

- **Decision**: All GameObjects have an optional `parent` ref; plus domain-specific refs
- **Rationale**: Built-in hierarchy for containment (Rooms in Buildings in Cities); other relationships use explicit refs
- **Implications**: `parent` is a reserved field on all GameObjects. Orphan detection when parent is archived.

### Actions on Kinds

- **Decision**: Bidirectional binding—Actions target Kinds AND Kinds list Actions
- **Rationale**: Actions can declare their valid targets; Kinds can also explicitly include Actions. Available actions = union.
- **Implications**: More flexibility; Action availability computed from both sources.

### Archiving Behavior

- **Decision**: Dangling refs allowed with GM alert
- **Rationale**: Don't cascade or block; give GM control to resolve orphaned references
- **Implications**: System tracks dangling refs; surfaces them in GM workflow. No automatic cleanup.

### Templates/Prototypes

- **Decision**: Default Objects in Paradigm (no template GameObjects)
- **Rationale**: Paradigms define seed objects; created when Game starts or on demand
- **Implications**: No special template flag; seeding is a Paradigm concern, not a GameObject concern.

### Field Requiredness

- **Decision**: All fields optional at schema level; validation at use
- **Rationale**: Maximum flexibility at creation; validation when Actions/Triggers actually need values
- **Implications**: No schema-level required/optional distinction. Runtime validation as needed.

### Field Defaults

- **Decision**: Multi-layer defaults—Kind definition + EventType override
- **Rationale**: Kinds specify sensible defaults; EventTypes can override for specific creation scenarios
- **Implications**: Default resolution: EventType default > Kind default > null

---

## Kind Schema Format

Kinds are defined in Paradigm YAML files. Each Kind specifies the structure for GameObjects of that type.

### Field Types

| Type | Syntax | Description |
|------|--------|-------------|
| Primitive | `string`, `number`, `boolean` | Basic scalar types |
| List of primitives | `string[]`, `number[]` | Arrays of scalars |
| Object | `{...}` | Nested structure with fields |
| Meter | `type: meter` | Numeric field with min/max bounds |
| Ref (singular) | `type: ref` | Reference to another GameObject |
| Ref (list) | `type: ref[]` | List of references |

### Meter Declaration

```yaml
fields:
  hp:
    type: meter
    min: 0
    max: self.status.max_hp    # Expression referencing computed value
    default: 10

  stress:
    type: meter
    min: 0
    max: 9                      # Static bound
    default: 0
```

Meter bounds (`min`, `max`) can be:
- Static numbers
- Expressions referencing other fields (using the expression DSL)

### Ref Declaration

```yaml
fields:
  player:
    type: ref
    target_kind: Player         # Optional: constrain to specific Kind

  faction:
    type: ref
    target_kind: Faction

  bonds:
    type: ref[]
    target_kind: [Character, Faction, Location]  # Multiple allowed Kinds

  allies:
    type: ref[]                 # No constraint—any GameObject
```

### Nested Structures

```yaml
fields:
  appearance:
    type: object
    fields:
      height: string
      hair_color: string
      distinguishing_marks: string[]

  background:
    type: object
    fields:
      origin:
        type: ref
        target_kind: Location
      family: string
```

---

## Example Kind Definition

```yaml
kinds:
  Character:
    description: "A person in the game world—PC or NPC"

    fields:
      # Core identity
      name:
        type: string
        default: "Unknown"

      # PC-only (null for NPCs)
      player:
        type: ref
        target_kind: Player

      # Numeric meters
      hp:
        type: meter
        min: 0
        max: self.status.max_hp
        default: 10

      stress:
        type: meter
        min: 0
        max: 9
        default: 0

      # Attributes (used to compute max_hp, etc.)
      constitution:
        type: number
        default: 2

      # Relationships
      faction:
        type: ref
        target_kind: Faction

      bonds:
        type: ref[]
        target_kind: [Character, Faction, Location]

      # Nested data
      appearance:
        type: object
        fields:
          height: string
          build: string
          notable_features: string[]

    # Computed values (defined here, computed into status)
    computed:
      max_hp:
        expr: self.spec.constitution * 3 + 5

      bond_count:
        expr: len(self.spec.bonds)

    # Actions available on Characters
    actions:
      - HealCharacter
      - TakeDamage
      - FormBond
```

---

## GameObject Schema

```python
from datetime import datetime
from typing import Any

class GameObject:
    # Identity
    id: str                      # ULID
    game_id: str                 # Which Game this belongs to
    kind: str                    # Kind name (e.g., "Character")

    # Hierarchy
    parent: str | None           # Optional parent GameObject ID

    # Data
    spec: dict[str, Any]         # Authored data (matches Kind schema)
    status: dict[str, Any]       # Computed/derived values
    status_dirty: bool           # Whether status needs recomputation

    # Permissions
    owner: str | None            # Selector or user ID
    acl: list[ACLEntry]          # Access control list

    # Lifecycle
    archived: bool               # Soft-deleted
    created_at: datetime
    updated_at: datetime         # Last event that modified this
```

---

## Reserved Fields

These fields exist on all GameObjects regardless of Kind:

| Field | Type | Description |
|-------|------|-------------|
| `id` | ULID | Unique identifier |
| `game_id` | ULID | Parent Game |
| `kind` | string | Kind name |
| `parent` | ref | Optional hierarchical parent |
| `archived` | boolean | Soft-delete flag |
| `owner` | selector | Who owns this object |
| `acl` | ACLEntry[] | Access control |

---

## Expression Language (Overview)

A simple DSL used throughout the system for:
- Meter bounds (min/max)
- Computed status fields
- Trigger conditions
- Action preconditions
- Conditional event operations

### Supported Features

```
# Path references
self.spec.constitution
self.status.max_hp
target.spec.hp

# Arithmetic
self.spec.constitution * 3 + 5

# Comparisons
self.spec.hp < self.status.max_hp
self.spec.stress >= 6

# Logical
x and y
x or y
not x

# Conditionals
if self.spec.hp > 0 then 'alive' else 'dead'

# Functions
len(self.spec.bonds)
min(self.spec.hp, 10)
max(x, y)
floor(x)
ceil(x)

# List membership
target in self.spec.allies
```

**Note**: Full expression language spec to be defined separately. This is the expected feature set.

---

## Default Objects

Paradigms can define seed objects that are created when a Game starts:

```yaml
default_objects:
  - kind: Game
    id: $GAME_ID                 # Special: the Game itself
    spec:
      name: "New Campaign"
      paradigms: [blades-in-the-dark]

  - kind: Faction
    spec:
      name: "The Bluecoats"
      tier: 3
      hold: strong

  - kind: Faction
    spec:
      name: "The Hive"
      tier: 2
      hold: weak
```

---

## Dangling Reference Handling

When a GameObject is archived, refs pointing to it become "dangling":

1. **Detection**: System tracks all ref fields. On archive, queries for objects with refs to the archived ID.
2. **Alert**: GM is notified via workflow (similar to Draft approval queue).
3. **Resolution options**:
   - Re-assign ref to a different object
   - Clear the ref (set to null)
   - Create a replacement object
   - Ignore (keep dangling for historical record)
4. **Historical queries**: Dangling refs are valid for historical views—the object existed at that time.

---

## Implications for Other Specs

| Spec | Implication |
|------|-------------|
| [events.md](events.md) | `object.create` payload includes `kind` and `initial_data` matching Kind schema |
| [actions-drafts.md](actions-drafts.md) | ActionDefs can target specific Kinds; Kinds can list available Actions |
| [triggers.md](triggers.md) | Triggers can match on object Kind; expressions use the shared DSL |
| [projections.md](projections.md) | Status fields are projections; invalidation tracking needed |
| [paradigms.md](paradigms.md) | Paradigms define Kinds with the schema format above; plus default_objects |

---

## Open for Future Refinement

- [ ] Full expression language specification
- [ ] Kind inheritance/mixins
- [ ] More complex player-character association
- [ ] Status computation optimization strategies

---

_This document reflects decisions made 2026-01-10. Marked for future refinement pass._
