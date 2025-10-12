# Step 14 — YAML Import/Export (Objects, Sheets, Traits, Meters, Rules)

## Summary
Implement YAML import/export endpoints and CLI to load and persist paradigms and campaign configurations.

## Contribution
Enables portability, backups, and rapid bootstrapping of systems and campaigns.

## Acceptance Criteria
- YAML schemas and validators for objects, sheets, traits, meters, and rules.
- Idempotent import: repeated loads do not create duplicates or break referential integrity.
- Export endpoints and CLI commands to write YAML directories.
- Tests cover round-trip integrity on representative samples.
