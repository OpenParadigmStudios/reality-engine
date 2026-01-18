# Reality Engine

*A campaign operating system for narrative-heavy tabletop RPGs*

---

## For GMs and Players

Want to understand what Reality Engine is and how to think about your game in its terms?

**Start here:** [User Documentation](docs/README.md)

You'll find:
- How to conceptualize your campaign in Reality Engine terms
- What becomes a GameObject, Event, or Trigger
- Concrete examples from games like Blades in the Dark
- [Glossary of terms](docs/reference/glossary.md)

---

## For Developers

Reality Engine uses an ECS-inspired, event-sourced architecture. The core principle: **only immutable, approved World Events change game state.**

**Start here:** [Specification Index](spec/README.md)

Key architectural concepts:
- **Events** — Immutable records of state changes
- **GameObjects** — Entities with Kinds (characters, factions, clocks, etc.)
- **Projections** — Derived read models computed from event history
- **Triggers** — Automatic reactions that cascade events

### Tech Stack (planned)
- Python 3.11+ / FastAPI / SQLAlchemy / Pydantic
- SQLite (MVP 0), PostgreSQL (MVP 1)

---

## Repository Structure

```
reality-engine/
├── docs/           # Human-readable documentation for GMs and players
├── spec/           # Technical specifications (structured for LLM consumption)
│   ├── MASTER.md   # Central index and status tracker
│   ├── architecture/
│   ├── domains/
│   └── implementation/
├── paradigms/      # Example YAML paradigm definitions
└── src/            # Source code (future)
```

---

## Status

Reality Engine is in the **specification and design phase**. No production code yet.

---

*For project conventions and development workflow, see [.claude/CLAUDE.md](.claude/CLAUDE.md).*
