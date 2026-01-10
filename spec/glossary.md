# Reality Engine — Glossary

Canonical definitions for all system terms. This glossary is automatically updated during spec interrogation.

---

## Core Concepts

### Game
A campaign instance and isolation boundary. Owns all world and workflow data for a single campaign. Selects active Paradigms. In MVP 0, each Game is a separate running instance.

### World Domain
The authoritative game state. Immutable, event-sourced, deterministic, and replayable. Only approved Events can modify world state.

### Workflow Domain
The layer handling human intent and collaboration. Editable, commentable, role-aware, and approval-gated. Never directly mutates world state. Includes Drafts, comments, and approval workflows.

---

## Entities

### User
A global identity (account) representing a human or agent. Has no inherent authority by itself—authority comes from GameMembership.

### GameMembership
A User's role inside a specific Game. Defines permissions, visibility, role assignments (GM, player), and character associations.

### GameObject
The base entity type (ECS "Entity"). Represents anything that exists in the game world: characters, factions, locations, clocks, bonds, etc. Carries structured data via `spec` (authoritative) and `status` (derived).

### Kind
A schema defining a type of GameObject. Kinds are defined in Paradigms and specify the structure, allowed components, and available actions for objects of that type.

---

## Events

### Event (World Event)
An atomic, immutable record of a world state change. The sole mechanism for changing world state. Forms a complete, ordered campaign history. Only approved Events affect projections or triggers.

### Canonical Event Types
The minimal, stable vocabulary of world mutations:
- `object.create` — new GameObject instantiated
- `object.update` — GameObject spec modified
- `component.set` — component value changed
- `meter.delta` — numeric value adjusted by amount
- `meter.set` — numeric value set absolutely
- `link.add` — relationship created between objects
- `link.remove` — relationship removed
- `text.annotate` — narrative annotation added

### Causality
The chain linking an Event to its origin. Tracks which Event (or Draft) caused this Event to be emitted, enabling cascade tracing and debugging.

---

## Actions & Workflow

### ActionDef
A declarative definition of what an action *is*. Defined in Paradigms. Specifies valid targets, required conditions, editable parameters, and the Events to propose when invoked.

### Action Offer
A computed (not stored) representation of "this action is available here right now." Combines ActionDef + target + current state to present available choices with defaults and editable parameters.

### Draft
A pending, editable proposal to change the world. Captures user intent, holds editable parameters, contains proposed Events, and manages approval workflow. Drafts never affect projections. Drafts are Workflow Domain entities, separate from Events.

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
A declarative rule that reacts to world Events. Defined in Paradigms. Listens for matching approved Events and emits new Events (or Drafts) in response. Enables automated narrative cascades.

### Match (Trigger)
The conditions a Trigger uses to select which Events it responds to. Can filter by event type, scope, tags, thresholds, or custom expressions.

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

## Projections

### Projection
A read model derived from world Events. Computes "current state" for efficient queries. Fully rebuildable from the Event log. Examples: current GameObject state, activity feeds, aggregate counts.

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
- Kinds (GameObject schemas)
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
A container for Events that occurred during a play session. Groups Events chronologically and associates them with attendance (which players/GMs were present).

### Campaign Timeline
The ordered sequence of all Events in a Game. Can be segmented by Sessions. Supports review, replay, and auditing.

---

_Last updated during interrogation. Terms are added as they emerge in spec discussions._
