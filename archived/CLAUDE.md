# Reality Engine

Tabletop roleplaying game narrative operating system.

**Note:** Files in `plan/archived/` are historical documents. Ignore them unless explicitly asked to review archived content.

## Commands

```bash
uv run pytest              # Run tests
uv run ruff check src      # Lint
uv run mypy src/zos        # Type check
```

## Behavior
- After you make a change, make sure to update any relevant markdown files if the change needs to be reflected in that documentation

## Structure

```
tests/             # pytest tests
plan/              # Architecture docs
```

## Key Files

- `config/config.example.yml` - All config options
- `plan/project.md` - Architecture and design spec
- `plan/plan.md` - Phased implementation plan

