# Step 05 — Sheets, Traits, and Meters (Clocks)

## Summary
Add `sheets`, `traits`, and `meters` models and attach them to game objects; include APIs to manage them.

## Contribution
Allows structured state, modular descriptors, and progress tracking that power the narrative and mechanics.

## Acceptance Criteria
- SQLAlchemy models and relationships for Sheets, Traits, Meters.
- Attach/detach APIs for adding/removing to/from objects.
- Meters support current/max and thresholds.
- Validation with Pydantic; tests cover creation, linking, and updates.
