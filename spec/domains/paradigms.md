# Paradigms — Domain Specification

**Status**: 🔴 Not started  
**Last interrogated**: Not yet  
**Depends on**: [game-objects.md](game-objects.md), [actions-drafts.md](actions-drafts.md), [triggers.md](triggers.md), [permissions.md](permissions.md)  
**Depended on by**: None (top of dependency chain)

---

## Overview

Paradigms are modular, shareable packages that define game system elements. They contain Kind schemas, ActionDefs, Triggers, and optional seed objects. Multiple Paradigms can combine to form a complete game system.

---

## What We Know (From PRD/Summary)

### Purpose
- Modular, shareable packages defining game systems
- Enable system-agnostic platform
- Support composition of multiple systems
- Enable house rules as overlays

### Contents
- **Kinds**: GameObject schemas
- **ActionDefs**: Available actions
- **Triggers**: Reactive rules
- **Default Objects**: Optional seed data

### Composition Rules
- Paradigms can depend on other Paradigms
- Multiple Paradigms can combine if definitions don't conflict
- House rules = local Paradigm overlay

---

## Open Questions

### Paradigm Structure
- [ ] Directory structure for a Paradigm?
- [ ] YAML schema for each component type?
- [ ] Versioning within a Paradigm?
- [ ] Metadata (author, description, license)?

### Dependency Management
- [ ] How are dependencies declared?
- [ ] Dependency version constraints?
- [ ] Dependency resolution order?

### Conflict Detection
- [ ] What constitutes a conflict?
- [ ] Same Kind name? Same ActionDef ID?
- [ ] How are conflicts reported?
- [ ] Conflict resolution strategies?

### Loading & Validation
- [ ] Load-time validation?
- [ ] Runtime schema enforcement?
- [ ] Error reporting format?

### MVP 0 Scope
- [ ] Just single directory, no dependency management?
- [ ] Multiple directories = implicit combine?
- [ ] Conflict = fail loudly?

---

## Decisions Made

*None yet — awaiting interrogation.*

---

_This document will be updated during `/interrogate spec/domains/paradigms` sessions._
