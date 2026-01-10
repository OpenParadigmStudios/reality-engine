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
A schema defining a type of GameObject. Kinds are defined in Paradigms and specify the structure, allowed components, field types (including Meters and Refs), and available actions for objects of that type.

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

Ref fields enable:
- Referential integrity alerting (dangling reference detection)
- Reverse lookup indexing
- Clean trigger matching on relationship changes

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
The authoritative data on a GameObject. Designer-defined fields that represent the "true" state of the object.

### Status (GameObject)
Derived or projection data on a GameObject. Computed from Events and Spec. May include cached aggregates, computed properties, or denormalized data for query efficiency.

---

## Sessions & Time

### Session
A container for Events that occurred during a play session. Groups Events chronologically and associates them with attendance (which players/GMs were present). Events optionally reference their Session via `session_id`.

### Campaign Timeline
The ordered sequence of all Events in a Game. Ordered by `(game_id, seq)`. Can be segmented by Sessions. Supports review, replay, and auditing.

---

## Identity & Ordering

### ULID
Universally Unique Lexicographically Sortable Identifier. Used for Event IDs. Embeds timestamp for rough chronological sorting while remaining globally unique.

### Sequence Number
A monotonic counter per Game. Each Event gets a `seq` value that provides strict total ordering within a Game. The authoritative ordering mechanism for replay.

---

_Last updated during interrogation 2026-01-09. Terms are added as they emerge in spec discussions._
