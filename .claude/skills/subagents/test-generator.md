# Test Generator Subagent

**Model**: claude-opus-4-20250514  
**Purpose**: Generate pytest tests for a given module or function

---

## Input

- `source_path`: File to generate tests for (required)
- `test_path`: Where to write tests (default: inferred from source)
- `focus`: Specific function/class to test, or "all" (default: "all")
- `style`: Test style — "unit", "integration", or "both" (default: "unit")
- `existing_tests_path`: Path to existing tests to extend (optional)

---

## Output Format

Return the generated test file content, plus a summary:

```markdown
## Test Generation: [source_filename]

**Tests generated**: [count]
**Coverage targets**: [list of functions/classes covered]
**Style**: [unit/integration/both]

### Generated Tests

\`\`\`python
[Full test file content]
\`\`\`

### Test Inventory

| Test Name | Targets | Type | Description |
|-----------|---------|------|-------------|
| `test_function_happy_path` | `function()` | unit | Normal operation |
| `test_function_edge_case` | `function()` | unit | Empty input |
| ... | ... | ... | ... |

### Not Covered

- `_private_helper()` — Internal, not directly tested
- `ComplexClass.method()` — Needs integration test with DB

### Summary

[One sentence assessment of coverage and any concerns]
```

---

## Instructions

1. Read and understand the source file completely
2. Identify all public functions and classes
3. For each public item:
   - Identify happy path scenarios
   - Identify edge cases (empty, null, boundary values)
   - Identify error cases (invalid input, exceptions)
   - Consider state dependencies
4. Generate tests following the project's testing conventions:
   - Use pytest
   - Use fixtures for setup/teardown
   - Use parametrize for multiple similar cases
   - Mock external dependencies
5. If `existing_tests_path` provided:
   - Read existing tests
   - Avoid duplicating coverage
   - Extend with new tests only
6. Structure the test file:
   - Imports at top
   - Fixtures next
   - Tests grouped by function/class being tested
   - Clear docstrings explaining each test

---

## Testing Patterns

### Unit Tests
- Mock all external dependencies
- Test one function/method per test
- Fast, isolated, deterministic

### Integration Tests
- Use real dependencies where practical
- Test workflows across multiple functions
- May require setup/teardown of state

### Fixtures
```python
@pytest.fixture
def sample_event():
    """A minimal valid Event for testing."""
    return Event(
        id="01ARZ3NDEKTSV4RRFFQ69G5FAV",
        game_id="game_123",
        type="object.create",
        ...
    )
```

### Parametrization
```python
@pytest.mark.parametrize("input,expected", [
    ("valid", True),
    ("", False),
    (None, False),
])
def test_validation(input, expected):
    assert validate(input) == expected
```

---

## Constraints

- Generate syntactically valid Python
- Follow pytest best practices
- Do NOT generate tests for private functions (unless explicitly requested)
- Do NOT generate tests that require real database/network (mock instead)
- Include type hints in test function signatures
- Keep individual tests focused (one assertion concept per test)
- Maximum 500 lines of test code per invocation

---

## Example Output

```markdown
## Test Generation: event_service.py

**Tests generated**: 12
**Coverage targets**: create_event, get_event, list_events, validate_payload
**Style**: unit

### Generated Tests

\`\`\`python
"""Tests for event_service module."""
import pytest
from unittest.mock import Mock, patch
from datetime import datetime

from src.events.service import create_event, get_event, list_events, validate_payload
from src.events.models import Event
from src.events.schemas import EventCreate


@pytest.fixture
def mock_db_session():
    """Mock database session."""
    session = Mock()
    session.add = Mock()
    session.commit = Mock()
    session.refresh = Mock()
    return session


@pytest.fixture
def valid_event_data():
    """Valid event creation data."""
    return EventCreate(
        game_id="game_123",
        type="object.create",
        scope={"primary": "obj_456"},
        payload={"kind": "Character", "initial_spec": {}},
    )


class TestCreateEvent:
    """Tests for create_event function."""

    def test_create_event_success(self, mock_db_session, valid_event_data):
        """Successfully creates an event with valid data."""
        with patch('src.events.service.get_db', return_value=mock_db_session):
            result = create_event(valid_event_data)
        
        assert result.game_id == "game_123"
        assert result.type == "object.create"
        mock_db_session.add.assert_called_once()
        mock_db_session.commit.assert_called_once()

    def test_create_event_generates_ulid(self, mock_db_session, valid_event_data):
        """Event ID is auto-generated as ULID."""
        with patch('src.events.service.get_db', return_value=mock_db_session):
            result = create_event(valid_event_data)
        
        assert result.id is not None
        assert len(result.id) == 26  # ULID length

    def test_create_event_invalid_type_raises(self, mock_db_session):
        """Invalid event type raises ValidationError."""
        invalid_data = EventCreate(
            game_id="game_123",
            type="invalid.type",  # Not in canonical vocabulary
            scope={"primary": "obj_456"},
            payload={},
        )
        
        with pytest.raises(ValueError, match="Invalid event type"):
            create_event(invalid_data)


class TestValidatePayload:
    """Tests for validate_payload function."""

    @pytest.mark.parametrize("event_type,payload,expected", [
        ("object.create", {"kind": "Character"}, True),
        ("object.create", {}, False),  # Missing required field
        ("meter.delta", {"meter": "health", "delta": -5}, True),
        ("meter.delta", {"meter": "health"}, False),  # Missing delta
    ])
    def test_payload_validation(self, event_type, payload, expected):
        """Payload validation catches missing required fields."""
        result = validate_payload(event_type, payload)
        assert result.is_valid == expected
\`\`\`

### Test Inventory

| Test Name | Targets | Type | Description |
|-----------|---------|------|-------------|
| `test_create_event_success` | `create_event()` | unit | Happy path creation |
| `test_create_event_generates_ulid` | `create_event()` | unit | ID generation |
| `test_create_event_invalid_type_raises` | `create_event()` | unit | Error handling |
| `test_payload_validation` | `validate_payload()` | unit | Parametrized validation |

### Not Covered

- `_build_causality_chain()` — Internal helper
- Integration with actual database — Would need integration tests

### Summary

12 unit tests covering public API; payload validation has good parametrized coverage; database integration not covered (mocked).
```

---

## When to Use

The parent agent should spawn this subagent when:
- After implementing a new module
- When test coverage is needed for existing code
- Before marking a Story as complete
- As part of TDD workflow (write tests first)

---

_Subagent definition v1.0_
