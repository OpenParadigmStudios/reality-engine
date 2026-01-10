# Architecture Critique: Reality Engine

A constructive analysis of the proposed architecture and implementation plan, intended to inform re-architecture and refinement.

---

## Executive Summary

Reality Engine has a **clear vision** and **well-documented architecture**, but several structural issues need addressing before implementation:

1. **Multi-GM is the primary use case but deferred to Phase 3** — this should shape the core architecture from day one
2. **Event sourcing "light" is underspecified** — the rollback/undo semantics need concrete definition
3. **Real-time approval workflow conflicts with async design** — needs WebSocket/realtime strategy
4. **The YAML rules engine is a project unto itself** — high risk, underscoped
5. **Scope creep risk** — 17 steps + AI stretch goals for a community tool MVP

---

## Critical Issues

### 1. Multi-GM Should Drive Architecture, Not Be Deferred

**The Problem:**
Multi-GM with shared continuity is listed as the *primary* use case, yet it's relegated to Phase 3 (Steps 11-13). This is backwards. Multi-user concurrent writes to shared narrative state is a fundamentally different problem than single-GM CRUD.

**Specific Concerns:**
- **Optimistic vs. pessimistic concurrency** — If two GMs edit the same Character/Node simultaneously, who wins? This affects every model and endpoint.
- **"Narrative collision" is undefined** — What constitutes a collision? Same object? Same timeline window? Same Session?
- **Sync mechanisms are missing** — How do GMs know when the other GM changed something?

**Recommendation:**
Move multi-GM considerations to Phase 0-1. At minimum:
- Define the concurrency model (optimistic locking with version fields? Last-write-wins? Explicit locks?)
- Add `updated_at` + `version` to all mutable models from the start
- Define what triggers a "collision" vs. a normal concurrent edit

**Architect Response:**
Agree on moving up multi-GM considerations. I think that the system in terms of minimizing collisions is helped by the permissions system, and many things having a single Owner that approves what edit ultimately happens. Should probably support something like two submitted edits being able to be merged into a hand negotiated middle ground outcome by the owner and final approver, if that's not difficult, but that could be part of future facing stuff. Beyond that though, I would say that all editors have to submit the version they think they were editing along with their payload, and if that version submitted is outdated from current reality it rejects the edit and pops it back to them for revision with a note about its new state.

---

### 2. Event Sourcing Semantics Need Definition

**The Problem:**
The docs describe "event sourcing light" — append-only events plus materialized state — but the actual semantics of rollback, replay, and consistency are unclear.

**Key Unanswered Questions:**
- **What gets stored in an Event?** The full before/after state? Just deltas? References to changed fields?
- **How does rollback work?** Creating a "counter-event" that reverses changes? Replaying from a checkpoint? Soft-deleting the event?
- **What happens to downstream effects?** If Event A caused Meter X to tick to threshold, which triggered Event B, and you roll back Event A... does Event B also roll back?
- **Dual-write consistency** — When you write to both `Event` table AND update `Character.health`, what happens if one succeeds and one fails?

**Recommendation:**
Before implementing, document:
1. Event payload schema (what's stored)
2. Rollback algorithm (counter-events vs. checkpoint restore vs. tombstones)
3. Transactional guarantees (both writes in same DB transaction? Eventual consistency?)
4. Cascading effects policy (roll back downstream events or leave them?)

Consider whether you actually need event sourcing, or if a simpler **audit log** (append-only record of who changed what, without replay capability) would suffice for the "undo" use case.

**Architect Notes:**
Okay, this is one of those kill your baby decisions, huh? I love the idea of this because of the clean elegant end results of being able to parse out causality, but the practical under the hood black box of complexity isn't worth it. Let's go with the audit log and cut this for this version. We may revisit this later in a re-arch.

---

### 3. Real-Time Support is Missing

**The Problem:**
The architecture is REST-only, but the approval workflow needs to work during live sync play. Players submit Actions, GMs approve, Events emit — this needs to feel fast.

**Gaps:**
- No WebSocket, SSE, or long-polling strategy mentioned
- No push notifications for "your action was approved"
- Feeds are described as paginated REST endpoints, not real-time streams

**Recommendation:**
Add to Phase 1 or early Phase 2:
- WebSocket gateway for live updates (new events, approval status changes)
- Event bus pattern internally (when Event is created → push to connected clients)
- Consider whether REST polling is acceptable for MVP, but design models to support push later

**Architecture Notes:**
Grug brain say REST polling fine probably. If WebSocket gateway is low effort maybe include that, but really think about if the juice is worth the squeeze on that one.

---

### 4. The YAML Rules Engine is High-Risk

**The Problem:**
Step 10 introduces declarative YAML rules that "map actions to event emissions and state changes (incl. meter ticks)." This is essentially building a domain-specific programming language.

**Underestimated Complexity:**
- **Conditionals** — "If character has Trait X, apply modifier Y" requires expression evaluation
- **Loops/iteration** — "For each character in Group, tick their project clock" requires iteration
- **Error handling** — What happens when a rule references a nonexistent Trait? Silent failure? Rollback? Log warning?
- **Debugging** — How does a GM figure out why their rule isn't working?
- **Testing rules** — Can you unit test a YAML rule definition?
- **Ordering** — When multiple rules match, which fires first?

**Recommendation:**
Dramatically reduce scope. Options:
1. **Hardcode the first rules** — Implement "Approve action → create Event → tick Meters" in Python. Add YAML configurability later.
2. **Template-only** — Rules are templates with placeholders, not expressions. No conditionals, no loops. Just "Action type X creates Event with these fields."
3. **Full DSL but Phase 5** — Move rules engine to stretch goals. It's a project in itself.

---

### 5. Visibility Model Needs Concrete Rules

**The Problem:**
Player visibility is described conceptually (own characters, groups they're in, bonds, global events) but not formally specified.

**Ambiguities:**
- **Transitive visibility** — If my Character is Bonded to NPC Bob, and Bob knows Secret Event X, can I see Event X?
- **Group membership visibility** — If I'm in Faction A, can I see ALL events affecting Faction A, or only ones that explicitly include me?
- **Partial object visibility** — Can a player see that a Character exists but not their Sheet? Or is it all-or-nothing?
- **Session attendance** — If I attended Session 5, can I see all events from Session 5, or only events involving objects I can see?

**Recommendation:**
Define explicit visibility rules as a formal policy document before implementation. Example format:
```
PLAYER can see EVENT if:
  - EVENT.visibility = 'global', OR
  - EVENT affects CHARACTER owned by PLAYER, OR
  - EVENT affects GROUP containing CHARACTER owned by PLAYER, OR
  - EVENT.session attended by PLAYER
```

---

## Moderate Issues

### 6. "Node" Abstraction is Overloaded

**The Problem:**
"Node" is defined as "Locations, Resources/Artifacts, or Abstract Power (titles, favor, licenses)." These are three very different things crammed into one model.

- A **Location** has coordinates, contains other things, has neighbors
- A **Resource** is consumable/tradeable, has quantity
- **Abstract Power** is a status/permission, has holders, can be contested

Treating these identically will either limit the model (can't represent location-specific behaviors) or bloat it (every Node has optional location fields, quantity fields, holder fields).

**Recommendation:**
Consider whether Node should be split into subtypes with a shared base, or if the polymorphism should be more explicit. At minimum, clarify what fields/behaviors are shared vs. type-specific.

---

### 7. The Phased Plan is Waterfall-ish

**The Problem:**
The 17-step plan reads like a waterfall: Phase 0 done before Phase 1, etc. But some Phase 3 concerns (multi-GM) should inform Phase 1 models.

**Risks:**
- Building models in Phase 1 that need restructuring in Phase 3
- No usable system until many phases complete (Feeds require Events require Sessions require GameObjects require Auth require Environment...)

**Recommendation:**
Consider vertical slices instead:
- **Slice 1**: One campaign, one GM, Characters only, manual Event creation, simple Feed → *usable end-to-end*
- **Slice 2**: Add Groups, Nodes, Sheets, Traits
- **Slice 3**: Actions + Approvals workflow
- **Slice 4**: Multi-GM, collision detection
- **Slice 5**: YAML config import/export

Each slice delivers usable functionality.

---

### 8. Missing: Search and Query Patterns

**The Problem:**
GMs need to find things: "Which characters have the Vampire trait?" "What events happened at Location X?" "Show me all of Bob's bonds."

The API surface mentions "search" but doesn't specify:
- Full-text search? Structured queries? Both?
- What's searchable? (Names? Descriptions? Trait names? Event narratives?)
- Index strategy for search performance

**Recommendation:**
Add search requirements to Phase 1. Options:
- SQLite FTS5 / PostgreSQL full-text for narrative fields
- Structured filters for typed queries (trait name, object type, date range)
- Consider whether you need a search service (Elasticsearch) or if DB search suffices

---

### 9. Missing: Backup and Campaign Portability

**The Problem:**
YAML import/export is Phase 4, but GMs need backup capability earlier. If the database corrupts, are campaigns lost?

**Recommendation:**
Add minimal export capability to Phase 1:
- Full campaign dump to JSON/YAML (objects, events, sessions)
- Corresponding import to restore
- This also enables testing with realistic data

---

### 10. Auth Complexity May Be Underestimated

**The Problem:**
Auth is listed as Step 03 "skeleton" — JWT, roles, campaign scoping. But the actual requirements are nuanced:

- Users can have **different roles in different campaigns**
- Role checking happens on *every* endpoint
- Token refresh, password reset, possibly OAuth for Discord integration later

**Recommendation:**
Flesh out the auth design:
- Campaign-role junction table (user_id, campaign_id, role)
- Middleware/dependency for role checking per-route
- Consider whether Step 03 should also include campaign membership CRUD

---

## Minor Issues / Suggestions

### 11. Terminology Inconsistencies

- "Node" vs. "StoryNode" vs. "Anchor" — pick one
- "Sheet" is both a container and a concept — "ObjectSheet" or "CharacterSheet" might be clearer
- "Meter" vs. "Clock" — BitD fans will expect "Clock"; the code uses "Meter"

### 12. Testing Strategy

The plan calls for 80% coverage, but doesn't address:
- How to test visibility rules (many edge cases)
- How to test the rules engine (if built)
- Integration tests for multi-step workflows (submit action → approve → event created → feed updated)

### 13. Soft Delete Complexity

`deleted_at` is good for audit, but means every query needs `.where(deleted_at.is_(None))`. Consider a mixin or query filter to enforce this globally.

### 14. Consider GraphQL

With complex nested relationships (Campaign → Characters → Sheets → Traits → Perks), and varied client needs (Discord bot wants minimal data, web app wants full objects), GraphQL might be a better fit than REST. Not a must-have, but worth considering.

---

## Positive Observations

To balance the critique, things that are well-done:

1. **Clear vision and validation heuristic** — "If the news feed reflects the right events for the right people at the right time, the system is working" is an excellent north star.

2. **Technology choices are solid** — FastAPI + SQLAlchemy + Pydantic is a pragmatic, well-supported stack. `uv` is a good modern choice.

3. **The options.md decision log** — Having explicit architectural decisions with pros/cons documented is excellent practice.

4. **System-agnostic aspiration** — The goal to separate game mechanics from core structure (via Traits, Perks, rules templates) is admirable and well-motivated.

5. **Soft delete and audit-friendly design** — Including `deleted_at`, timestamps, and append-only events shows forethought for data integrity.

6. **AGENTS.md is comprehensive** — The setup instructions and coding standards are clear enough for onboarding developers or AI agents.

---

## Recommended Priority Changes

1. **Define multi-GM concurrency model before Phase 1** — Add version fields, define collision semantics
2. **Specify event payload and rollback semantics** — Document before implementing Event model
3. **Defer YAML rules engine** — Hardcode initial rules in Python
4. **Add minimal real-time support to Phase 2** — At least design for WebSocket later
5. **Define visibility rules formally** — Write them as logical statements
6. **Consider vertical slices** — Get something usable end-to-end earlier

---

## Questions for Further Discussion

1. **Offline/local-first?** — Should the system work offline with sync later? This dramatically affects architecture.

2. **Campaign templates** — Is there a concept of "start a new campaign from template X" with pre-populated objects?

3. **Versioned API or evolving?** — Will `/api/v1` stay stable, or evolve? What's the deprecation policy?

4. **Discord-first or web-first?** — Which client will be the initial test consumer? This affects API design priorities.

5. **AI integration scope** — Phase 5 mentions AI-assisted session logs. Is this local LLM, API calls, or out of scope for MVP?

---

## Conclusion

Reality Engine has a solid conceptual foundation and clear documentation. The main risks are:
- **Architectural decisions deferred that should be upfront** (multi-GM concurrency, event semantics)
- **Scope creep** (rules engine, AI features)
- **Missing real-time strategy** for sync play

Addressing these before implementation will save significant rework. The phased plan is reasonable but would benefit from vertical slicing to deliver usable functionality earlier.

This critique is intended to strengthen the architecture, not discourage the project. The vision is compelling and the documentation quality suggests the project has a good chance of success with some structural adjustments.
