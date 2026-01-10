# Actions & Drafts — Domain Specification

**Status**: 🔴 Not started  
**Last interrogated**: Not yet  
**Depends on**: [events.md](events.md), [game-objects.md](game-objects.md)  
**Depended on by**: [triggers.md](triggers.md), [permissions.md](permissions.md)

---

## Overview

Actions are the mechanism by which users (players, GMs) express intent to change the world. ActionDefs declare what actions are possible; Drafts capture specific invocations awaiting approval. Only when a Draft is approved do its proposed Events enter the world.

---

## What We Know (From PRD)

### ActionDef Purpose
- Declarative definition of what an action *is*
- Defined in Paradigms (modules)
- Specifies valid targets, conditions, parameters, and effect templates

### ActionDef Required Fields (From PRD)
- `id` — unique identifier
- `on_kind` — which Kind(s) this action targets
- `requirements` — conditions that must be true
- `effects` — event templates to propose
- `approval_policy` — how approval is determined

### Draft Purpose
- Pending, editable proposal to change the world
- Captures user intent and parameters
- Contains proposed (not yet real) events
- Manages approval workflow

### Draft Required Fields (From PRD)
- `id`
- `action_def_id` — which ActionDef this invokes
- `actor` — who is proposing
- `target` — what object(s) this affects
- `params` — user-supplied parameters
- `proposed_events[]` — Events that will be created if approved
- `status` — pending, approved, rejected, committed

### Key Rules (From PRD)
- Drafts never affect projections
- Draft edits do not emit world events
- Draft approval gates world mutation
- Action Offers (available actions) are computed, not stored

---

## Open Questions

### ActionDef Structure
- [ ] How are `requirements` expressed? DSL? JSON logic?
- [ ] How are `effects` templated? Variable interpolation?
- [ ] Can one ActionDef produce multiple Events?
- [ ] Conditional effects based on parameters?

### Draft Lifecycle
- [ ] Who can edit a Draft? Just the actor? GM?
- [ ] Can a Draft be revised after rejection?
- [ ] Draft expiration?
- [ ] Draft history/versioning?

### Approval Flow
- [ ] How is approval policy determined? Per ActionDef? Per Kind? Per object?
- [ ] Multiple approvers required?
- [ ] Partial approval (approve some effects, reject others)?
- [ ] Comment/feedback on Drafts?

### Event Proposal
- [ ] How are `proposed_events` generated from ActionDef + params?
- [ ] Preview mode (show what would happen without committing)?
- [ ] Validation of proposed events against current state?

### Parameters
- [ ] Parameter schema per ActionDef?
- [ ] Parameter validation?
- [ ] Default values?
- [ ] Computed parameters from current state?

---

## Decisions Made

*None yet — awaiting interrogation.*

---

_This document will be updated during `/interrogate spec/domains/actions-drafts` sessions._
