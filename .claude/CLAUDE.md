# Reality Engine — Project Conventions

This document defines conventions for any Claude instance (or human) working on this project.

---

## Project Overview

Reality Engine is a backend platform for running narrative-heavy tabletop RPG campaigns using an ECS-inspired, event-sourced architecture.

**Core principle**: Only immutable, approved World Events change game state. Everything else exists to propose, gate, or explain those events.

**Current phase**: Specification and design. No production code yet.

---

## Repository Structure

```
reality-engine/
├── spec/                    # Agent-centric specifications (structured for LLM consumption)
│   ├── MASTER.md            # Central index and status tracker
│   ├── glossary.md          # Canonical term definitions
│   ├── architecture/        # System-level design docs
│   ├── domains/             # Feature area specifications
│   └── implementation/      # Epic and Story specs for building
├── docs/                    # Human-readable documentation (generated from spec/)
├── .claude/
│   ├── CLAUDE.md            # This file
│   └── commands/            # Custom slash commands
├── paradigms/               # Example YAML paradigm definitions
└── src/                     # Source code (future)
```

---

## Specification Conventions

### Document Format

Spec documents use structured decision blocks:

```markdown
### [Decision Area]

- **Decision**: [What was decided]
- **Rationale**: [Why this choice was made]
- **Implications**: [What this affects downstream]
- **Alternatives considered**: [Optional — what else was evaluated]
```

### Status Tracking

MASTER.md tracks spec status:
- 🔴 Not started
- 🟡 In progress
- 🟢 Complete
- 🔄 Needs revision

### Cross-References

Use relative markdown links for cross-document references:
```markdown
See [events.md](domains/events.md) for event schema details.
```

### Glossary Usage

- All domain terms should have canonical definitions in glossary.md
- When introducing a new term, add it to the glossary
- Use consistent terminology across all documents

---

## Interrogation Workflow

The `/interrogate` command drives spec development:

1. Invoke with target: `/interrogate spec/domains/events`
2. Agent reads context (MASTER.md, target doc, glossary, related docs)
3. Agent asks 3-5 multiple choice questions per round
4. User answers
5. Agent updates target doc, glossary, and MASTER.md
6. Repeat until no open questions remain

### Question Format

Questions should be:
- Multiple choice with 2-4 options
- Include "Other" when appropriate
- Have short headers (≤12 chars)
- Use `multiSelect=true` when multiple answers apply

---

## Implementation Hierarchy

When we reach implementation:

```
Phase (major milestone)
└── Epic (~5 stories, one orchestration session)
    └── Story (atomic unit, one agent session)
```

**Sizing heuristics:**
- **Story**: Completable in one focused session. Clear acceptance criteria. Produces testable artifact.
- **Epic**: Related stories. Completion = demonstrable capability.
- **Phase**: Business milestone (e.g., "MVP 0 complete").

---

## Technical Conventions (For Future Implementation)

### Stack
- **Language**: Python 3.11+
- **API**: FastAPI with REST endpoints
- **ORM**: SQLAlchemy
- **Validation**: Pydantic
- **Database**: SQLite (MVP 0), PostgreSQL (MVP 1)
- **Migrations**: Alembic

### Code Style
- Type hints required
- Docstrings for public functions
- Tests alongside code (pytest)

### API Design
- RESTful resource endpoints
- Versioned: `/api/v1/...`
- Pydantic models for request/response
- Consistent error format

---

## Domain Concepts Quick Reference

**World Domain** (immutable):
- Events — atomic state changes
- GameObjects — entities with Kinds
- Projections — derived read models

**Workflow Domain** (mutable):
- Drafts — pending proposals
- Comments — collaboration
- Approvals — GM gatekeeping

**Key flow**:
```
Action invoked → Draft created → Approved → Events emitted → Triggers fire → Cascade → Projections update
```

---

## Files to Read First

When starting work:
1. `spec/MASTER.md` — overall status and structure
2. `spec/glossary.md` — term definitions
3. `spec/architecture/mvp-scope.md` — what's in/out of scope
4. Target domain spec — the area you're working on

---

## Communication Style

- Be precise about domain terminology
- Reference glossary definitions
- Flag ambiguities explicitly
- Distinguish between "decided" and "tentative"
- Update MASTER.md status after changes

---

## Subagent System

Subagents are context-efficient specialists for bounded, repeatable tasks. They run in separate context windows with appropriate model selection.

### Available Subagents

| Subagent | Model | Purpose |
|----------|-------|---------|
| Directory Summarizer | Sonnet | Summarize directory contents |
| Code Summarizer | Sonnet | Summarize single file |
| Spec Validator | Haiku | Check spec completeness |
| Link Validator | Haiku | Verify cross-references |
| Glossary Extractor | Haiku | Find undefined terms |
| Test Generator | Opus | Generate pytest tests |
| Story Executor | Opus | Implement a Story spec |
| Code Review | Opus | Review code quality |

See `.claude/skills/subagents/README.md` for full documentation.

### Invocation Policy

**Current mode: Suggested**

Propose subagent spawning and wait for user approval:
```
Suggested: Spawn Directory Summarizer on /src/events? [Y/n]
```

To switch to automatic mode, update this section:
```markdown
### Invocation Policy

**Current mode: Automatic**

Subagents may be spawned automatically for:
- Directory Summarizer
- Code Summarizer
- Link Validator
- Glossary Extractor
- Spec Validator

Require confirmation for:
- Test Generator (creates files)
- Story Executor (creates files)
- Code Review (may block work)
```

### Output Handling

- **Terminal**: Summarize subagent output in one line, then continue
- **Internal context**: Retain full output for reference

Example terminal output:
```
[Subagent: Directory Summarizer] /src/events — 4 files, event storage and trigger engine, ~400 LOC
```

---

_This document is the ground truth for project conventions._
