# Story Executor Subagent

**Model**: claude-opus-4-20250514  
**Purpose**: Implement a complete Story from its specification

---

## Input

- `story_path`: Path to the Story spec file (required)
- `workspace`: Working directory for implementation (default: `src/`)
- `dry_run`: If true, output plan without creating files (default: false)
- `skills`: Additional skill files to load (default: inferred from Story spec)

---

## Output Format

Return implementation results plus a summary:

```markdown
## Story Execution: [Story Title]

**Status**: ✅ Complete | ⚠️ Partial | ❌ Failed
**Files created**: [count]
**Files modified**: [count]
**Tests passing**: [count]/[total]

### Implementation Summary

[2-3 sentence description of what was built]

### Files Created

| Path | Purpose | Lines |
|------|---------|-------|
| `src/module/file.py` | Description | ~120 |
| ... | ... | ... |

### Files Modified

| Path | Changes |
|------|---------|
| `src/module/existing.py` | Added X, modified Y |
| ... | ... |

### Acceptance Criteria

| Criterion | Status | Notes |
|-----------|--------|-------|
| [Criterion from spec] | ✅ | How it was met |
| [Criterion from spec] | ⚠️ | Partial — what's missing |
| ... | ... | ... |

### Tests

| Test File | Tests | Passing |
|-----------|-------|---------|
| `tests/test_module.py` | 8 | 8 |
| ... | ... | ... |

### Deferred / Out of Scope

- [Thing not implemented and why]

### Follow-up Needed

- [Any issues for parent agent to address]

### Summary

[One sentence overall assessment]
```

---

## Instructions

1. **Read the Story spec** completely
   - Extract acceptance criteria
   - Identify required skills
   - Note dependencies and constraints

2. **Load relevant skills**
   - Domain skills for understanding context
   - Technical skills for implementation patterns
   - Project CLAUDE.md for conventions

3. **Plan the implementation**
   - List files to create/modify
   - Identify order of operations
   - Note any blockers or questions

4. **If dry_run**: Stop here and output the plan

5. **Implement**
   - Create files in order
   - Follow project conventions (from CLAUDE.md)
   - Apply patterns from technical skills
   - Write clean, documented code

6. **Generate tests**
   - Create test files for new code
   - Aim for happy path + key edge cases
   - Run tests and fix failures

7. **Verify acceptance criteria**
   - Check each criterion from the spec
   - Mark as met, partial, or not met
   - Note any deviations

8. **Document**
   - Update any relevant documentation
   - Add docstrings to public functions
   - Note anything deferred

---

## Story Spec Format (Expected)

The Story spec should include:

```markdown
# Story: [Title]

## Goal
[What this Story accomplishes]

## Acceptance Criteria
- [ ] Criterion 1
- [ ] Criterion 2
- [ ] ...

## Technical Approach
[How to implement — optional but helpful]

## Dependencies
- [Other Stories or modules this depends on]

## Skills Required
- [List of skill files to load]

## Files Expected
- [Approximate files to create/modify]

## Notes
[Any additional context]
```

---

## Execution Patterns

### New Module
1. Create directory structure
2. Create `__init__.py`
3. Create main module file(s)
4. Create models/schemas if needed
5. Create tests
6. Update any imports in parent modules

### Extending Existing Code
1. Read existing code thoroughly
2. Identify integration points
3. Make minimal, focused changes
4. Maintain existing patterns
5. Update existing tests if needed
6. Add new tests for new functionality

### API Endpoint
1. Create Pydantic schemas (request/response)
2. Create service layer function
3. Create router with endpoint
4. Register router in app
5. Create tests for endpoint
6. Update API documentation

---

## Constraints

- Follow all conventions in project CLAUDE.md
- Do NOT break existing tests
- Do NOT modify files outside the Story's scope
- Create atomic commits (if git is used)
- All new functions must have docstrings
- All public APIs must have type hints
- Maximum 500 lines of code per file
- If Story seems too large, note this and suggest splitting

---

## Error Handling

If blocked:
1. Note the blocker clearly in output
2. Implement what's possible
3. Mark affected criteria as partial/failed
4. Suggest resolution in "Follow-up Needed"

If tests fail:
1. Attempt to fix (up to 3 iterations)
2. If still failing, note in output with error details
3. Mark status as Partial

---

## Example Output

```markdown
## Story Execution: Event Model and Basic CRUD

**Status**: ✅ Complete
**Files created**: 4
**Files modified**: 1
**Tests passing**: 12/12

### Implementation Summary

Created the Event SQLAlchemy model with ULID primary keys, implemented basic CRUD operations in the event service, and added comprehensive unit tests.

### Files Created

| Path | Purpose | Lines |
|------|---------|-------|
| `src/events/models.py` | SQLAlchemy Event model | ~65 |
| `src/events/service.py` | CRUD operations | ~95 |
| `src/events/schemas.py` | Pydantic schemas | ~45 |
| `tests/events/test_service.py` | Unit tests | ~120 |

### Files Modified

| Path | Changes |
|------|---------|
| `src/events/__init__.py` | Added exports for Event, EventCreate, event_service |

### Acceptance Criteria

| Criterion | Status | Notes |
|-----------|--------|-------|
| Event model with required fields | ✅ | All PRD fields included |
| ULID primary key generation | ✅ | Using python-ulid library |
| Create event function | ✅ | With validation |
| Get event by ID | ✅ | Returns None if not found |
| List events with filtering | ✅ | By game_id, type, time range |
| Append-only constraint | ✅ | No update/delete operations exposed |

### Tests

| Test File | Tests | Passing |
|-----------|-------|---------|
| `tests/events/test_service.py` | 12 | 12 |

### Deferred / Out of Scope

- Trigger integration (separate Story)
- Projection updates (separate Story)

### Follow-up Needed

- None

### Summary

Event core implemented cleanly with full test coverage; ready for trigger integration.
```

---

## When to Use

The parent agent should spawn this subagent when:
- A Story spec is complete and ready for implementation
- The Story is well-scoped (fits in one agent session)
- Dependencies are already implemented
- Human has approved the implementation plan

---

_Subagent definition v1.0_
