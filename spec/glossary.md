# Reality Engine — Glossary

Canonical definitions for all system terms. This glossary is automatically updated during spec interrogation.

---

## Core Concepts

### Game
A campaign instance and isolation boundary. Owns all world and workflow data for a single campaign. Selects active Paradigms. In MVP 0, each Game is a separate running instance. The Game itself is a GameObject and can be targeted by Events (e.g., signal Events like `DowntimePasses`).

### World Domain
The authoritative game state. Immutable, event-sourced, deterministic, and replayable. Only approved Events can modify world state.

### Workflow Domain
The layer handling human intent and collaboration. Editable, commentable, role-aware, and approval-gated. Never directly mutates world state. Includes Drafts, comments, approval workflows, and Notes.

---

## Entities

### User
A global identity (account) representing a human or agent. Has no inherent authority by itself—authority comes from GameMembership.

### GameMembership
A User's role inside a specific Game. Defines permissions, visibility, role assignments (GM, player), and character associations.

### GameObject
The base entity type (ECS "Entity"). Represents anything that exists in the game world: characters, factions, locations, clocks, bonds, etc. Carries structured data via `spec` (authoritative) and `status` (derived). Can be archived (soft-deleted) but never hard-deleted.

### Kind
A schema defining a type of GameObject. Kinds are defined in Paradigms and specify:
- Field structure (primitives, nested objects, meters, refs)
- Computed status fields (expressions that derive values)
- Available Actions (bidirectional: Kinds list Actions, Actions target Kinds)

For MVP, Kinds are standalone (no inheritance). Future: mixins/traits or single inheritance.

### Nestable Kind
A Kind where objects can have parent refs to objects of the same Kind, enabling recursive hierarchies. Implemented using the standard `parent` field with `target_kind` constrained to the Kind itself. Examples: Story (arcs containing beats), Location (Room → Building → City). No special flag required—nestability is a pattern achieved through Kind constraints.

---

## Events

### Event (World Event)
An instantiated EventType. An atomic, immutable record of a world state change. Contains both the semantic type name (for trigger matching) and the operations performed (for projection updates). The sole mechanism for changing world state.

### EventType
A named schema defined in a Paradigm that specifies:
- What targets it requires (primary object Kind)
- What parameters it accepts
- What operations it performs when instantiated
- Optional narrative template and tags

Examples: `DowntimePasses`, `ProgressFaction`, `HealCharacter`.

### Operation
An atomic mutation primitive. Operations are the building blocks that EventTypes compose. Types:
- `object.create` — instantiate GameObject
- `object.update` — modify data field
- `object.archive` — soft-delete
- `meter.delta` — adjust numeric field by amount
- `meter.set` — set numeric field to value
- `ref.set` — set singular reference field
- `ref.add` — add to reference list field
- `ref.remove` — remove from reference list field

### Causality
The chain linking an Event to its origin. Each Event tracks:
- `parent_event_id` — the Event that triggered this one (null if root)
- `root_draft_id` — the originating action's Draft
- `depth` — how deep in the cascade (0 = root)

### Actor
The entity that caused an Event. Stored as a reference with type and ID:
- `user` — a User initiated this
- `system` — the system generated this automatically
- `trigger` — a Trigger fired and emitted this

### Signal Event
An Event with no operations. Used to broadcast that something happened (e.g., `DowntimePasses`) without directly mutating state. Typically targets the Game object. Triggers react to the signal and emit their own Events.

---

## Field Types

### Meter
A special numeric field type declared in a Kind schema. Meters support:
- Validation (min/max bounds)
- Specialized operations (`meter.delta`, `meter.set`)
- Trigger matching on thresholds

### Ref (Reference)
A special field type that holds a reference to another GameObject. Can be singular or a list:
- **Singular ref**: One reference, uses `ref.set`
- **Ref list**: Multiple references, uses `ref.add`/`ref.remove`

Ref declarations can optionally constrain target Kind(s): `target_kind: Faction` or `target_kind: [Character, Faction, Location]`.

Ref fields enable:
- Referential integrity alerting (dangling reference detection)
- Reverse lookup indexing
- Clean trigger matching on relationship changes

### Tagged Ref
A ref with optional string tags for metadata. All refs can carry tags—simple string arrays that annotate the reference. Structure: `{target: <id>, tags?: string[]}`. Tags enable role annotation (e.g., `['primary', 'founder']`), relationship types (e.g., `['ally', 'rival']`), and provenance tracking. Operations `ref.add` and `ref.set` accept an optional `tags` parameter; `ref.tag` modifies tags on existing refs.

### Parent
A reserved ref field on all GameObjects providing hierarchical containment. Optional (nullable). Used for structures like Rooms inside Buildings inside Cities. When a parent is archived, child objects surface as dangling refs for GM resolution.

---

## Actions & Workflow

### ActionDef
A declarative definition of what an action *is*. Defined in Paradigms. Specifies valid targets, required conditions, editable parameters, and the EventType to instantiate when invoked.

### Action Offer
A computed (not stored) representation of "this action is available here right now." Combines ActionDef + target + current state to present available choices with defaults and editable parameters.

### Draft
A pending, editable proposal to change the world. Captures user intent, holds editable parameters, references the EventType to instantiate, and manages approval workflow. Drafts never affect projections. Drafts are Workflow Domain entities, separate from Events.

### Draft Status
The lifecycle state of a Draft:
- `pending` — awaiting approval
- `approved` — GM accepted, ready to commit
- `rejected` — GM declined
- `committed` — proposed Events have been appended to world

### Approval Policy
Rules determining whether a Draft requires explicit approval or auto-approves based on actor permissions.

---

## Triggers

### Trigger
A declarative rule that reacts to world Events. Defined in Paradigms. Listens for matching approved Events (primarily by EventType name) and emits new Events (or Drafts) in response. Enables automated narrative cascades.

### Match (Trigger)
The conditions a Trigger uses to select which Events it responds to. Primarily matches on EventType name; can also filter by scope, tags, parameters, or custom expressions.

### Guard (Trigger)
Additional conditions that must be true for a matched Trigger to fire. Prevents triggers from activating inappropriately.

### Cascade
A sequence of Events triggered by other Events. One Event triggers reactions, which emit more Events, which trigger more reactions, until the system reaches quiescence.

### Cascade Limits
Safety constraints on trigger execution:
- **Max depth** — how many levels of causality before stopping
- **Max emissions** — how many Events a single Trigger can emit
- **Self-trigger prevention** — stops infinite loops

---

## Notes

### Note
A Workflow Domain entity for comments and discussion. Attached to GameObject instances. Editable and deletable. Does not affect world state. Used for player/GM collaboration.

---

## Projections

### Projection
A read model derived from world Events. Computes "current state" for efficient queries. Consumes the Operation layer of Events. Fully rebuildable from the Event log. Examples: current GameObject state, activity feeds, aggregate counts.

---

## Permissions

### Selector
A pattern for matching users, roles, or conditions. Used in ACLs and approval policies. Examples:
- `group:game:gm` — all GMs in this game
- `group:game:players` — all players
- `expr:user == owner` — the object's owner
- `expr:user in self.spec.members` — users listed in object data

### ACL (Access Control List)
Selector-based permissions attached to a GameObject. Defines who can view, edit, or take actions on that object.

---

## Paradigms

### Paradigm
A modular, shareable package defining game system elements:
- Kinds (GameObject schemas with Meter and Ref field types)
- EventTypes (named event schemas)
- ActionDefs (available actions)
- Triggers (reactive rules)
- Default Objects (optional seeds)

Paradigms can depend on other Paradigms. Multiple Paradigms can combine in a Game if their definitions don't conflict.

### House Rules
Local modifications to a Paradigm. Implemented as a Paradigm overlay. Can be published as a new Paradigm.

---

## Structural

### Spec (GameObject)
The authoritative data on a GameObject. Designer-defined fields that represent the "true" state of the object. Spec is what you set; modified only by Events.

### Status (GameObject)
Derived or projection data on a GameObject. Computed from spec values using the expression language. Includes:
- Computed properties (e.g., max_hp from constitution)
- Cached aggregates (e.g., bond_count)
- Denormalized data for query efficiency

Status is cached and invalidated when relevant Events occur; recomputed on next read.

### Default Object
A seed GameObject defined in a Paradigm. Created when a Game starts (or on demand). Used to pre-populate the world with standard entities (factions, locations, etc.).

---

## Narrative

### Story
A well-known Kind representing a narrative arc, quest, project, or goal. Stories are **recursively nestable**—a Story can contain child Stories (using the `parent` field), enabling hierarchical narrative structure where "beat" vs "arc" is purely contextual based on hierarchy position. Stories have polymorphic `owners` (Characters, Factions, etc.) via tagged refs. Lifecycle is managed via tags (active, completed, abandoned, on-hold) rather than formal states.

### Narrative Gate
A progression pattern that requires both mechanical progress (meter thresholds) AND narrative completion (Story with `completed` tag). Example: "Advance faction tier requires 8 project progress plus completing a Story about the advancement."

---

## Sessions & Time

### Session
A container for Events that occurred during a play session. Groups Events chronologically and associates them with attendance (which players/GMs were present). Events optionally reference their Session via `session_id`.

### Campaign Timeline
The ordered sequence of all Events in a Game. Ordered by `(game_id, seq)`. Can be segmented by Sessions. Supports review, replay, and auditing.

---

## Expression Language

### Expression DSL
A simple domain-specific language used throughout the system for computed values and conditions. Features:
- Path references (`self.spec.hp`, `target.status.max_hp`)
- Arithmetic (`+`, `-`, `*`, `/`)
- Comparisons (`<`, `>`, `<=`, `>=`, `==`, `!=`)
- Logical operators (`and`, `or`, `not`)
- Conditionals (`if x then y else z`)
- Functions (`len`, `min`, `max`, `floor`, `ceil`)
- List membership (`x in list`)
- **Query DSL**: `find('<Kind>').where(<condition>)` for projection-style queries
- List predicates: `any`, `all`, `none` for complex filtering

Used in: Meter bounds, computed status fields, Trigger conditions, Action preconditions, conditional EventType operations. All queries are scoped to the current Game—cross-game queries are forbidden.

### Computed Collection
A status field that uses the Query DSL to dynamically find related objects. Example: `active_children: find('Story').where(parent == self).where(tags contains 'active')`. Computed collections are re-evaluated on read when status is invalidated, providing live views of related data without manual bookkeeping.

---

## Identity & Ordering

### ULID
Universally Unique Lexicographically Sortable Identifier. Used for Event IDs and GameObject IDs. Embeds timestamp for rough chronological sorting while remaining globally unique.

### Sequence Number
A monotonic counter per Game. Each Event gets a `seq` value that provides strict total ordering within a Game. The authoritative ordering mechanism for replay.

### Dangling Reference
A ref field pointing to an archived (soft-deleted) GameObject. When detected, the system alerts the GM via the workflow queue. Resolution options: re-assign, clear, create replacement, or ignore for historical record.

---

## System Events

System events are EventTypes defined in the Core Paradigm. They are automatically emitted by the engine during lifecycle moments. System events inherit their actor from the parent event that caused them.

### Core Paradigm
A required Paradigm that defines system EventTypes. Implicitly included in all Games (MVP 0). Contains: MeterOverflow, MeterUnderflow, ObjectCreated, ObjectArchived, RefDangling.

### MeterOverflow
Emitted when a meter operation attempts to exceed the meter's maximum bound. The value is clamped to max. Payload includes meter name, requested value, and overflow amount. Emitted as a child event (depth+1) after the parent event commits.

### MeterUnderflow
Emitted when a meter operation attempts to go below the meter's minimum bound. Symmetric to MeterOverflow. Payload includes meter name, requested value, and underflow amount.

### ObjectCreated
Emitted when an `object.create` operation commits. Primary target is the newly created object. Payload includes the Kind name. Enables triggers to react to object creation (e.g., initialize related objects).

### ObjectArchived
Emitted when an `object.archive` operation commits. Primary target is the archived object. Payload includes the Kind name. Triggers can react to archival (e.g., cleanup, notifications).

### RefDangling
Emitted once per dangling reference discovered when an object is archived. Primary target is the object containing the dangling ref. Payload includes the field name and archived object ID/Kind. Surfaces in GM workflow for resolution.

---

## Workflow Domain (Extended)

### Feed
A Workflow Domain construct that presents a filtered list of information updates, events, and notes affecting a specific object or set of objects. Feeds are the primary player-facing view of game activity. Players typically interact with the world through Feeds rather than directly inspecting GameObject data.

---

_Last updated 2026-01-18. Added: Tagged Ref, Nestable Kind, Computed Collection, Query DSL. Updated: Story (recursive nesting), Expression DSL (query capabilities). Removed: StoryBeat._
