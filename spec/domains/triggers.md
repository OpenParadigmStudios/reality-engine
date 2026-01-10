# Triggers — Domain Specification

**Status**: 🔴 Not started  
**Last interrogated**: Not yet  
**Depends on**: [events.md](events.md), [game-objects.md](game-objects.md), [actions-drafts.md](actions-drafts.md)  
**Depended on by**: [projections.md](projections.md)

---

## Overview

Triggers are declarative rules that react to world Events. They enable automated narrative cascades: when something happens, other things happen in response. This is the "Systems" part of the ECS model.

---

## What We Know (From PRD)

### Purpose
- Declarative reactions attached to the system (or to specific objects/Kinds)
- Listen for specific kinds of Events (by type, scope, thresholds, tags)
- When matched, emit new Events (or Drafts requiring approval)
- Enable automated narrative cascades

### Required Fields (From PRD)
- `id`
- `match` — event type + conditions
- `guards` — additional conditions to check
- `emit` — event templates to produce

### Constraints (From PRD)
- Triggers only consume **approved world events**
- Triggers must enforce:
  - Max depth (prevent infinite cascade)
  - Max emissions (limit blast radius)
  - Self-trigger prevention (avoid loops)

---

## Open Questions

### Trigger Definition
- [ ] Where are Triggers defined? Paradigm-level? Object-level? Both?
- [ ] Can Triggers be enabled/disabled per Game?
- [ ] Trigger priority/ordering?

### Match Syntax
- [ ] How do we express "match this event type with these conditions"?
- [ ] Match on event type only? Payload values? Scope?
- [ ] Pattern matching on affected objects?
- [ ] Match on causality (only trigger on root events? on cascaded events?)?

### Guards
- [ ] How do guards differ from match conditions?
- [ ] Access to game state in guards? (e.g., "only if meter X > 5")
- [ ] Guard failure behavior (silent skip? log? error?)?

### Emission
- [ ] Triggers emit Events directly vs emit Drafts?
- [ ] Configuration per Trigger? Per emission?
- [ ] Event templating (interpolate values from matched event)?
- [ ] Multiple emissions from one Trigger?

### Cascade Control
- [ ] How is max depth enforced? Global setting? Per Trigger?
- [ ] What happens when limit is hit? Error? Warning? Silent stop?
- [ ] Cycle detection approach?
- [ ] Transaction semantics (rollback cascade on error?)?

### Execution Model
- [ ] Synchronous cascade (process all Triggers before returning)?
- [ ] Async/queued execution?
- [ ] Trigger execution order when multiple Triggers match?

---

## Key Design Considerations

### Direct Events vs Drafts

Triggers can emit:
- **Direct Events**: Immediately appended, no approval needed. For system-level reactions that should always happen.
- **Drafts**: Require GM approval before Events are created. For cascades that GM should review.

**Decision needed**: How does a Trigger specify which mode? Per Trigger? Per emission? Based on event type?

### Narrative Cascades Example

```
Player action: "Attack the Baron"
  → Event: meter.delta (Baron.health, -5)
    → Trigger: "When Baron health < 10, Baron flees"
      → Event: object.update (Baron.location = "Hidden")
        → Trigger: "When Baron flees, Guard faction hostility +1"
          → Event: meter.delta (GuardFaction.hostility, +1)
            → Trigger: "When hostility > 5, Guards attack on sight"
              → Event: text.annotate ("Guards now hostile to party")
```

Each step could be:
- Auto-approved (mechanical consequence)
- Draft-requiring (GM wants to review/modify)

---

## Decisions Made

*None yet — awaiting interrogation.*

---

_This document will be updated during `/interrogate spec/domains/triggers` sessions._
