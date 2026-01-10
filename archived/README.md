## reality-engine

A backend platform for running narrative-heavy tabletop RPG campaigns. It helps GMs replace scattered notes and index cards with structured, searchable objects and timelines—while giving players a clean, filtered view of what’s relevant to them.

### What it is
- **A GM-focused campaign CMS** for intrigue-driven games with factions, politics, and long-running continuity.
- **Backend-only (to start)**: provides an API that multiple clients can use (web app, Discord bot, CLI).
- **Event-driven history**: every change becomes an event, building timelines you can review and replay.
- **Config-driven and extensible**: game paradigms (traits, sheets, clocks, rules) can be defined in YAML and imported/exported.

### Core concepts
- **Campaigns**: self-contained worlds with users, sessions, events, and timelines.
- **Game Objects**: Characters, Groups/Factions, and Nodes (locations/resources/abstract power). These are composed with Sheets, Traits/Tags, and Meters (clocks).
- **Events & Sessions**: sessions contain events; events record state changes and narrative write-ups; timelines track progression.
- **Actions & Approvals**: players submit actions; GMs review/approve/edit; rules templates convert actions into events and meter updates.
- **Feeds & Visibility**: a news feed shows what’s happening. GMs see all; players see filtered, relevant updates tied to their characters and bonds.

### What it’s not
- Not a virtual tabletop (no live maps/tokens/dice).
- Not tied to a single game system; it’s system-agnostic and extensible.
- Not real-time chat or session voice tooling.

### Who it’s for
- **GMs** who run narrative-heavy games and want structure without losing flexibility.
- **Players** who want a clean, relevant view of campaign updates.
- **Multi-GM groups** who need shared continuity and collision resolution tools.

### How it works (high level)
- **Event-sourcing light**: events are the audit trail and power the feed; object state is materialized for fast reads.
- **Rules-driven**: declarative templates (YAML) translate actions into events and state changes (including clock/meter ticks).
- **Auth & visibility**: role-based access (Player, GM, Head GM) with per-campaign scoping and object-level filtering.

### Technology (planned)
- **API**: FastAPI (Python 3.11+), versioned endpoints.
- **Data**: SQLAlchemy ORM; SQLite for local dev, PostgreSQL-ready; Alembic migrations.
- **Models**: Pydantic for requests/responses and YAML schema validation.
- **Config**: YAML import/export for objects, sheets, traits, meters, and rules.

### Project status
Early planning and architecture design. See `PROJECT.md` for the deeper technical overview, domain model, and phased roadmap.

### Roadmap (very high-level)
1. Environment & skeleton (FastAPI app, SQLite, basic roles).
2. Core data model & feeds (GameObjects; sheets/traits/meters; events/sessions; GM/Player feeds).
3. Interaction & actions (actions API; approvals workflow; rules templates).
4. Multi-GM & narrative tools (history views; reconciliation; performance/materialized feeds).
5. Config & extensibility (YAML import/export; paradigms packaging; schema validation).
6. Stretch goals (AI-assisted session logs; richer rules modules; optional job queue).

### Learn more
Start with `PROJECT.md` for the full project vision, object structure, architecture, and detailed implementation phases.
