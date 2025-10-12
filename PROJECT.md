## Tabletop RPG Management Platform — Project Overview

### Vision
- **Core purpose**: Replace a GM’s scattered notes and index cards with a structured, flexible system for narrative-heavy tabletop RPGs focused on intrigue, factions, and long-running continuity.
- **Primary users**: GMs and Players. GMs have universal visibility; Players see filtered, relevant information tied to their characters and bonds.
- **Campaigns**: Self-contained narrative worlds that can be private or shared across multiple GMs and playgroups, with continuity managed through events and timelines.
- **Validation heuristic**: If the news feed reflects the right events for the right people at the right time, the system is working.

### Product Principles
- **Narrative-first, system-agnostic**: Support a range of systems (e.g., political/factional play, Blades in the Dark-style clocks) by modeling narrative objects and relationships rather than hardcoding rules.
- **Event-sourced history**: Every change is an Event; timelines can be reviewed and replayed for any object (character, faction, node, session).
- **Composable objects**: Characters, Groups/Factions, and Nodes form the backbone; Sheets, Traits, Tags, and Meters (clocks) compose detail and mechanics.
- **Backend-only to start**: Provide a clean API for multiple clients (web app, Discord bot, CLI) to consume. No VTT features.
- **Config-driven**: YAML import/export defines game paradigms, traits, sheets, meters, and rules, enabling portability and extension.


## Conceptual Overview

### Users and Campaigns
- **Users & Roles**: Player, GM, optional Head GM for multi-GM coordination.
- **Campaigns**: Contain users, characters, groups, nodes, sessions, events, and a campaign timeline. Campaigns may be shared across tables.

### Game Objects
- **Characters**
  - Player-linked (PCs) or unlinked (NPCs); multiple characters per player allowed.
  - Important characters have full Sheets; minor NPCs can be lightweight profiles with tags/notes.
- **Groups/Factions**
  - Collections of characters, assets, and nodes; hold influence over Nodes and pursue goals.
  - Have group sheets for state, goals, resources, clocks.
- **Nodes (Anchors/Resources)**
  - Points of conflict: Locations, Resources/Artifacts, or Abstract Power (titles, favor, licenses).
  - Define what factions and characters can contend over.
- **Sheets, Traits, Tags, Meters**
  - Sheets: structured containers for an object’s state (stats, conditions, bonds).
  - Traits/Tags: modular descriptors that add narrative/mechanical flavor.
  - Meters: numeric trackers/clocks that fill and trigger effects.
- (Optional) **Stages/Boards**
  - Abstract arenas where factions/resources compete. Useful for some paradigms; not required.

### Narrative Flow
- **Events**
  - Atomic records of state changes with timestamp, narrative write-up, and references to affected objects.
  - Build object timelines; enable review/replay.
- **Sessions**
  - Containers for events, attendance, and touched objects; advance the campaign timeline and synchronize across groups.
- **Actions & Rules**
  - Players submit Actions (e.g., downtime moves) with options/modifiers.
  - Rules are templates that transform Actions into Events (and meter updates), with GM review.
- **Downtime Action Approval**
  - Structured GM ticketing flow to approve/reject/edit actions with comments before effects apply.

### Campaign Time & Clocks
- **Timeline**: Advanced in increments (sessions, ticks). Handles asynchronous play with reconciliation tools.
- **Project Clocks (Meters)**: Buckets that fill and trigger effects/events at thresholds.
- **Narrative Collisions**: Multi-group desync resolved by GM adjudication with system support for review and merge.

### Feeds & Visibility
- **News Feed**
  - GM: comprehensive feed of campaign events.
  - Player: filtered to global events, bonds, characters, groups, and sessions the player is part of.
- **Access model**: Visibility rules enforce who can see which events and object details.


## Technical Architecture

### Stack & Services
- **API**: FastAPI (Python 3.11+), REST-first with clear resource boundaries; versioned (e.g., `/api/v1`).
- **Persistence**: SQLAlchemy ORM. Local dev uses SQLite; production-ready path for PostgreSQL. Alembic for migrations.
- **Schema/Validation**: Pydantic models for request/response and YAML schema validation.
- **AuthN/Z**: Token-based authentication (e.g., OAuth2 password flow or access tokens). Role-based authorization (Player, GM, Head GM) with per-campaign scoping.
- **Background Jobs**: Lightweight task runner for timeline ticks and rule processing (e.g., asyncio tasks; optional queue later).
- **Config I/O**: YAML import/export for objects, sheets, traits, meters, and rules. Strict schemas, semantic validation, and idempotent operations.

### Domain Model (High-Level)
- **Users**: accounts, roles, memberships per campaign.
- **Campaigns**: world containers; settings and feature flags.
- **GameObjects (polymorphic base)**: `characters`, `groups`, `nodes`, and future types share common identifiers, metadata, and tags.
- **Sheets**: attached to objects; define fields/blocks; can reference Traits and Meters by id.
- **Traits/Tags**: reusable descriptors; may be campaign- or system-defined.
- **Meters (Clocks)**: named trackers with max/current values; thresholds trigger rule-defined effects.
- **Sessions**: date/time, attendance, touched object references.
- **Actions**: player-submitted intents with parameters; linked to rule templates.
- **Approvals**: GM review records with status, comments, and diffs.
- **Events**: append-only records referencing affected objects and the session/timeline; payload captures before/after or applied deltas.
- **Feeds**: views/materialized lists derived from events with visibility filters (GM vs Player).

### Key Patterns
- **Event-sourcing light**: Events as the audit trail and primary feed; state is materialized on objects for query performance.
- **Rules engine (template-driven)**: Declarative rule definitions (YAML) map actions to event emissions and state changes.
- **Import/Export**: Round-trippable YAML for portability of systems/paradigms; validation prevents partial/inconsistent loads.
- **Extensibility**: New object types and rule modules register against the `GameObject` base and rules dispatcher without core rewrites.


## API Surface (Initial Scope)
- Auth: login/logout, token refresh, user info.
- Campaigns: create/list/get/update.
- Characters, Groups, Nodes: CRUD, search, tagging, sheet attach/update.
- Sessions: create, list, append events, attach attendance and touched objects.
- Actions: submit, validate, preview effects.
- Approvals: GM review, approve/reject with comment, apply effects.
- Events: list by campaign/object/session; filter by visibility.
- Feeds: GM feed, Player feed endpoints with pagination and filters.
- Config: YAML import/export endpoints and offline CLI support.


## Security & Visibility
- Role-based access per campaign with object-level filtering.
- Player views restricted to own characters, groups they are part of, and designated bonds/global events.
- GMs see all campaign objects and events; Head GM manages multi-GM coordination and conflict resolution.


## Non-Goals (Initial)
- Virtual tabletop features (maps, tokens, live dice) are out of scope.
- Full-blown real-time sync; near-term design favors eventual consistency with explicit sync tools.
- Game-specific mechanics hardcoded in the backend; use rules/templates instead.


## Roadmap — High-Level Order of Implementation

### Phase 0 — Environment & Skeleton
1. Spin up local dev with SQLite and a running FastAPI app.
2. Establish basic project structure and placeholder routes.
3. Add initial auth skeleton with roles (Player, GM, Head GM) and campaign scoping.

### Phase 1 — Core Data Model & Feeds
4. Implement GameObject base; create characters, groups, nodes tables and CRUD APIs.
5. Add sheets, traits, meters (clocks) models and attachment to objects.
6. Build events and sessions models; append-only event logging; object timelines.
7. Implement GM/Player news feeds with visibility filters; pagination and sorting.

### Phase 2 — Interaction & Actions
8. Define actions API (player-submitted) with validation and preview.
9. Implement approvals workflow for GM review (approve/reject/edit with comments).
10. Introduce rules templates (YAML) to transform actions into events and state changes (including meter ticks).

### Phase 3 — Multi-GM & Narrative Tools
11. Session libraries (list touched objects), object history views, and timeline review per object.
12. Async play support: reconciliation helpers and narrative collision resolution tools.
13. Performance/materialization: feed indices, denormalized views for common queries.

### Phase 4 — Config & Extensibility
14. YAML import/export for objects, sheets, traits, meters, and rules; CLI support.
15. System paradigms packaged as YAML directories; schema validation and idempotent loads.

### Phase 5 — Stretch (Future)
16. AI-assisted session log parsing to propose events; GM review pipeline.
17. Advanced cross-system support, richer rule modules, and optional background job queue.


## Success Criteria
- News feeds correctly and consistently reflect authorized events for GMs and Players.
- Timelines and object histories can be reviewed; events form a coherent audit trail.
- YAML configs can round-trip between files and the running system without loss.
- The API is stable, versioned, and ready for multiple client frontends.
