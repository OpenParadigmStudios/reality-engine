# GameObjects — Domain Specification

**Status**: 🟡 In progress
**Last interrogated**: 2026-01-18
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

### ID Generation

- **Decision**: System generates ULID on commit
- **Rationale**: Caller specifies Kind and data; system handles ID assignment
- **Implications**: No caller-provided IDs; deterministic replay uses event log, not ID prediction

### Kind System

- **Decision**: No inheritance for MVP; each Kind is standalone
- **Rationale**: Simpler implementation; shared structure achieved through similar field definitions
- **Implications**: Future: add mixins/traits or single inheritance. For now, copy shared fields across Kinds.

### Game Kind

- **Decision**: Paradigm must explicitly define Game Kind
- **Rationale**: No magic built-in Kind; full customization possible
- **Implications**: Every Paradigm needs a Game Kind definition. System doesn't provide defaults.

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

- **Decision**: Always synchronous on first read after invalidation
- **Rationale**: Guarantees freshness; simpler model than async background jobs
- **Implications**: May slow reads for complex computations; status is never stale

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

### Archived Ref Resolution

- **Decision**: Return archived object data with `archived: true`
- **Rationale**: Consumer decides how to display/handle archived references
- **Implications**: Archived objects remain queryable; UI can show "(archived)" or similar

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

### Schema Drift

- **Decision**: Lenient handling of Kind schema changes
- **Rationale**: New fields default to null on old objects; removed fields preserved in data but ignored
- **Implications**: No migration scripts required for MVP. Data survives schema changes gracefully.

### Object Versioning

- **Decision**: Track version counter on each GameObject
- **Rationale**: Incremented on each Event that modifies the object; useful for optimistic locking
- **Implications**: Version available for conflict detection and change tracking

### Object Tags

- **Decision**: Built-in `tags: string[]` on all GameObjects
- **Rationale**: Enables filtering and querying across Kinds
- **Implications**: `tags` is a reserved field; can be used in expressions and queries

### Field Visibility

- **Decision**: Object-level ACL only (no per-field visibility)
- **Rationale**: Most player interaction is through Feeds; direct object inspection uses object-wide permissions
- **Implications**: If you can see the object, you see all its fields. Filtering happens at object level.

---

## Reserved Fields

These fields exist on all GameObjects regardless of Kind:

| Field | Type | Description |
|-------|------|-------------|
| `id` | ULID | Unique identifier (system-generated) |
| `game_id` | ULID | Parent Game |
| `kind` | string | Kind name |
| `name` | string | Display name (required, used for search/display) |
| `description` | string | Optional description/flavor text |
| `parent` | ref | Optional hierarchical parent |
| `tags` | string[] | Filtering labels |
| `owner` | selector | Who owns this object |
| `acl` | ACLEntry[] | Access control |
| `archived` | boolean | Soft-delete flag |
| `version` | integer | Modification counter |
| `created_at` | datetime | Creation timestamp |
| `updated_at` | datetime | Last modification timestamp |

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

**Note**: Enum/choice fields deferred to MVP 1. Use strings for now.

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

### Meter Bound Behavior

When a meter operation (delta/set) would exceed bounds:

1. **Clamp**: Value is clamped to min/max
2. **Emit Event**: System generates a `MeterOverflow` or `MeterUnderflow` event
3. **Trigger Potential**: Triggers can react to these events (e.g., convert excess HP to shields)

Overflow/Underflow events:
- **Primary target**: The object whose meter clamped
- **Refs**: Includes the Game for global watchers
- **Payload**: Meter name, attempted value, clamped value, overflow/underflow amount

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

### Ref Traversal Behavior

- **Archived exclusion**: Traversal like `bonds.*` excludes archived objects by default
- **Null handling**: Null refs are skipped silently in traversal
- **Explicit inclusion**: Use filter syntax to include archived: `bonds[all].*`

### Tagged Refs

All refs can carry optional metadata via **tags**—a simple string array attached to each reference.

**Structure:**

Refs are stored as objects rather than bare IDs:

```yaml
# Simple ref (no tags)
faction: char_123

# Tagged ref
faction:
  target: char_123
  tags: ['primary', 'founder']

# Ref list with mixed tagging
owners:
  - target: char_001
    tags: ['primary']
  - target: faction_002
    tags: ['supporting', 'sponsor']
  - target: char_003    # No tags
```

**Use cases:**
- **Role annotation**: `owners` refs tagged with `['primary']`, `['antagonist']`, etc.
- **Relationship types**: `bonds` refs tagged with `['ally']`, `['rival']`, `['family']`
- **Provenance**: `related_events` refs tagged with `['trigger']`, `['manual']`

**Operations:**

The `ref.add` operation accepts an optional `tags` parameter:

```yaml
operations:
  - op: ref.add
    target: $story_id
    payload:
      field: owners
      target_id: $character_id
      tags: ['primary', 'founder']
```

See [events.md](events.md) for full operation details including `ref.tag` for modifying tags on existing refs.

**Query access:**

Tagged refs can be filtered in expressions:

```
# Find owners with 'primary' tag
self.spec.owners[tags contains 'primary']

# Check if any owner is tagged as antagonist
self.spec.owners.any(o => o.tags contains 'antagonist')
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
    id: str                      # ULID (system-generated)
    game_id: str                 # Which Game this belongs to
    kind: str                    # Kind name (e.g., "Character")

    # Display
    name: str                    # Display name (reserved field)
    description: str | None      # Optional description/flavor

    # Hierarchy
    parent: str | None           # Optional parent GameObject ID

    # Classification
    tags: list[str]              # Filtering labels

    # Data
    spec: dict[str, Any]         # Authored data (matches Kind schema)
    status: dict[str, Any]       # Computed/derived values
    status_dirty: bool           # Whether status needs recomputation

    # Permissions
    owner: str | None            # Selector or user ID
    acl: list[ACLEntry]          # Access control list

    # Lifecycle
    archived: bool               # Soft-deleted
    version: int                 # Modification counter
    created_at: datetime
    updated_at: datetime         # Last event that modified this
```

---

## Expression Language

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
sum(self.spec.bonds.*.spec.loyalty)

# List membership
target in self.spec.allies

# Cross-object traversal
self.spec.bonds.*.spec.loyalty    # Returns list of values
sum(self.spec.faction.spec.members.*.spec.influence)
```

### Query DSL

The expression language includes projection-style chainable queries for finding and filtering GameObjects. Queries are available in all expression contexts (computed fields, trigger conditions, action preconditions, etc.).

**IMPORTANT: All queries are scoped to the current Game.** Cross-game queries are forbidden—Games are strict isolation boundaries. This is a permanent invariant, not an MVP limitation.

#### Basic Syntax

```
find('<Kind>').where(<condition>).where(<condition>)...
```

**Examples:**

```
# Find all Stories owned by the current object
find('Story').where(owners contains self)

# Find active Stories for a character
find('Story').where(owners contains self).where(tags contains 'active')

# Find Locations with a specific parent
find('Location').where(parent == some_city)

# Find Characters in a faction
find('Character').where(faction == target)
```

#### Comparison Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `==` | Equals | `where(kind == 'Character')` |
| `!=` | Not equals | `where(status != 'archived')` |
| `<`, `>` | Less/greater than | `where(spec.tier > 2)` |
| `<=`, `>=` | Less/greater or equal | `where(spec.hp <= 0)` |
| `contains` | List contains value | `where(tags contains 'active')` |
| `in` | Value in list | `where(self in spec.members)` |

#### List Predicates

For complex filtering on list fields, use `any`, `all`, or `none`:

```
# Any owner has 'primary' tag
owners.any(o => o.tags contains 'primary')

# All bonds are to Characters
bonds.all(b => b.kind == 'Character')

# No related events have 'system' tag
related_events.none(e => e.tags contains 'system')

# String operations in predicates
tags.any(t => t startsWith 'role-')
tags.all(t => t != 'deprecated')
```

#### Query Results

Queries return collections that can be:
- Checked for existence: `.any()` returns boolean
- Counted: `len(find(...))` returns integer
- Iterated: `.*.field` traverses into matched objects
- Used in conditions: `find(...).where(...).any()`

**Example computed status fields:**

```yaml
computed:
  # Count of active child stories
  active_story_count:
    expr: len(find('Story').where(parent == self).where(tags contains 'active'))

  # Boolean: has any completed projects
  has_completed_project:
    expr: find('Story').where(owners contains self).where(tags contains 'completed').any()

  # List: all faction members' names
  member_names:
    expr: find('Character').where(faction == self).*.name
```

### Expression Constraints

- **No time references**: Expressions cannot access `now` or timestamps. Time-based logic handled by Events/Triggers.
- **Null handling**: Return null when paths don't resolve; GM is notified via system note.
- **Archived refs**: Skipped in traversal by default.
- **Deterministic**: Same inputs always produce same outputs.
- **Game isolation**: Queries ALWAYS scope to the current Game—cross-game queries are forbidden.

### Error Handling

- **Decision**: Return null and create system note for GM
- **Rationale**: Keeps status computable while not silently hiding issues
- **Implications**: GM sees notification of expression errors; can investigate and fix

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

## Well-Known Kinds

### Required by System

| Kind | Purpose |
|------|---------|
| Game | The campaign instance. Every Paradigm must define this. Targets for signal Events. |

### Common (Advisory)

Most Paradigms will define variations of these:

| Kind | Purpose |
|------|---------|
| Character | PCs and NPCs |
| Faction | Organizations, groups |
| Location | Places in the world |
| Clock | Progress trackers |
| Item | Equipment, resources |
| Session | Play session metadata |
| Story | Narrative arcs, quests, projects, goals (recursively nestable) |

See separate examples documentation for game-specific implementations.

**Note:** Each well-known Kind will eventually have its own detailed spec in `spec/domains/kinds/`. Current definitions here are structural frameworks; full field schemas, actions, and patterns will be elaborated in those per-Kind documents.

### Story Kind (Nestable Pattern)

Stories represent narrative arcs that evolve over time. Stories are **recursively nestable**—a Story can contain child Stories, enabling hierarchical narrative structure (campaign arcs containing character arcs containing scene-level beats).

**Key design decisions:**
- **No separate StoryBeat Kind** — Stories nest within Stories. "Beat" vs "arc" is contextual based on hierarchy position.
- **Polymorphic ownership** — `owners` ref list supports multiple owner types (Characters, Factions, etc.) with optional tags for roles.
- **Auto-linked Events** — Events targeting a Story auto-populate `related_events` (opt-out via `link_to_story: false` on the Event).

**Story captures:**
- `summary` — short text for Feed display
- `owners` — polymorphic ref list (Characters, Factions, etc.) with optional tags
- `related_events` — Events that prompted or affected this Story (auto-populated)
- `parent` — self-referential for nesting (uses standard parent field with `target_kind: [Story]`)

**Lifecycle via tags** (no formal states):
- `active` — in progress
- `completed` — finished successfully
- `abandoned` — gave up
- `on-hold` — paused

**Example schema:**

```yaml
kinds:
  Story:
    description: "A narrative arc, quest, project, or goal (recursively nestable)"
    fields:
      # Short description for Feed display
      summary:
        type: string

      # Polymorphic ownership with tagged refs
      owners:
        type: ref[]
        target_kind: [Character, Faction]
        # Tags like 'primary', 'founder', 'antagonist' add context

      # Related entities
      involved_characters:
        type: ref[]
        target_kind: Character
      related_stories:
        type: ref[]
        target_kind: Story
      related_events:
        type: ref[]
        # Auto-populated when Events target this Story
        # Events can opt-out with link_to_story: false

    # Self-referential nesting via standard parent field
    # parent: {type: ref, target_kind: [Story]}

    computed:
      # Example: find active child stories
      active_children:
        expr: find('Story').where(parent == self).where(tags contains 'active')

    actions: []
```

**Nestable Pattern:**

Stories demonstrate the "nestable" pattern—any Kind can be nestable by declaring a `parent` ref with `target_kind` set to itself:

```yaml
# Nestable Location example
kinds:
  Location:
    fields:
      # ... other fields ...
    # Enables Room → Building → City → Region hierarchy
    # Uses reserved parent field with self-referential constraint
```

The nestable pattern uses the existing `parent` reserved field. Kinds that need nesting simply constrain `parent` to their own Kind.

**Narrative Gate Pattern:**

Stories enable "mechanical + narrative" progression gates:

```yaml
# Example: Advance faction tier requires progress AND a completed Story
actions:
  AdvanceFactionTier:
    preconditions:
      - expr: self.spec.project_progress >= 8
      - expr: find('Story').where(owners contains self).where(tags contains 'completed').any()
```

This pattern enforces that players must both accumulate mechanical progress AND wrap up the narrative before advancing.

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
| [events.md](events.md) | `object.create` payload includes `kind` and `initial_data` matching Kind schema; MeterOverflow/Underflow events defined; `ref.add` accepts `tags` parameter; `ref.tag` operation for modifying tags |
| [actions-drafts.md](actions-drafts.md) | ActionDefs can target specific Kinds; Kinds can list available Actions; can use Query DSL in preconditions |
| [triggers.md](triggers.md) | Triggers can match on object Kind and MeterOverflow/Underflow events; expressions and Query DSL in match conditions |
| [projections.md](projections.md) | Status fields are projections; synchronous recomputation on read; Query DSL overlaps with projection queries—relationship needs clarification |
| [paradigms.md](paradigms.md) | Paradigms define Kinds with the schema format above; plus default_objects; must define Game Kind; Story schema format changed |

---

## Open for Future Refinement

- [ ] Full expression language specification (formal grammar)
- [ ] Kind inheritance/mixins
- [ ] More complex player-character association
- [ ] Enum/choice field types
- [ ] Schema validation rules beyond types

---

_Last interrogated 2026-01-18. Story/StoryBeat restructured to recursive Story nesting. Tagged refs and Query DSL added. Individual Kinds need deep interrogation in `spec/domains/kinds/`._
