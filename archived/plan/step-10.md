# Step 10 — Rules Templates (YAML) → Events/State Changes

## Summary
Introduce declarative rule templates (YAML) that map action types to event emissions and state changes (incl. meter ticks).

## Contribution
Enables system-agnostic mechanics and portable paradigms; separates configuration from code.

## Acceptance Criteria
- YAML schema for rules with validation.
- Dispatcher applies rule templates to approved actions, producing events and updates.
- Error handling for invalid/missing rules.
- Tests for rule parsing, dispatch, and event generation.
