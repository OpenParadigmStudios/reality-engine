# Subagents — Usage and Configuration

## Overview

Subagents are context-efficient specialists that handle bounded, repeatable tasks. They run in separate context windows using cheaper models where appropriate, preserving the main agent's context for complex reasoning.

---

## Invocation Mode

**Current mode: Suggested**

The main agent will propose spawning a subagent and wait for your approval before proceeding.

Example:
```
I need to understand the /src/events directory structure before modifying triggers.

Suggested: Spawn Directory Summarizer subagent on /src/events?
[Y/n]
```

### Switching to Automatic Mode

To enable automatic subagent invocation, add this to your `.claude/CLAUDE.md`:

```markdown
## Subagent Policy

Subagents may be spawned automatically without confirmation for:
- Directory Summarizer
- Code Summarizer
- Link Validator
- Glossary Extractor
- Spec Validator

Subagents requiring confirmation:
- Test Generator (creates files)
- Story Executor (creates files)
- Code Review (may block work)
```

Or for full automatic mode:

```markdown
## Subagent Policy

All subagents may be spawned automatically. Summarize outputs in one line before continuing.
```

---

## Output Handling

**Terminal output**: Summarize in one line, then continue
**Internal context**: Silently consume full output, continue unless error

Example terminal output:
```
[Subagent: Directory Summarizer] /src/events — 4 files, event storage and trigger engine, ~400 LOC
```

The main agent retains the full structured output internally for reference.

---

## Model Configuration

Models are specified per-subagent in each skill file. Current assignments:

| Subagent | Model | Rationale |
|----------|-------|-----------|
| Directory Summarizer | Sonnet | Compression task |
| Code Summarizer | Sonnet | Compression task |
| Spec Validator | Haiku | Rule-checking |
| Link Validator | Haiku | Simple validation |
| Glossary Extractor | Haiku | Pattern matching |
| Test Generator | Opus | Code generation |
| Story Executor | Opus | Code generation |
| Code Review | Opus | Judgment required |

### Changing Models

To override a model for a specific subagent, edit the `model` field in the skill file:

```markdown
**Model**: claude-sonnet-4-20250514
```

Available models:
- `claude-opus-4-20250514` — Most capable, highest cost
- `claude-sonnet-4-20250514` — Balanced capability/cost
- `claude-haiku-3-5-20241022` — Fastest, lowest cost

### Cost Considerations

Approximate relative costs (as of late 2024):
- Opus: 1x (baseline)
- Sonnet: ~0.2x
- Haiku: ~0.04x

For high-volume tasks (link validation across 50 files), Haiku saves significant cost.

---

## Adding New Subagents

1. Create a new file in `.claude/skills/subagents/`
2. Follow the template structure:
   - Model specification
   - Purpose statement
   - Input format
   - Output format
   - Instructions
   - Constraints
3. Add to the subagent index in this README

### Template

```markdown
# [Subagent Name]

**Model**: [model-string]
**Purpose**: [One sentence]

## Input
- `param1`: Description
- `param2`: Description

## Output Format
[Describe the structure]

## Instructions
1. Step one
2. Step two
...

## Constraints
- Do NOT...
- Keep...
- Maximum...
```

---

## Subagent Index

| Subagent | File | Purpose |
|----------|------|---------|
| Directory Summarizer | `directory-summarizer.md` | Summarize directory contents |
| Code Summarizer | `code-summarizer.md` | Summarize single file |
| Spec Validator | `spec-validator.md` | Check spec completeness |
| Link Validator | `link-validator.md` | Verify cross-references |
| Glossary Extractor | `glossary-extractor.md` | Find undefined terms |
| Test Generator | `test-generator.md` | Generate pytest tests |
| Story Executor | `story-executor.md` | Implement a Story spec |
| Code Review | `code-review.md` | Review code quality |

---

## Debugging

If a subagent produces unexpected results:

1. Check the skill file for unclear instructions
2. Run manually with verbose output: ask the main agent to "show full subagent output for debugging"
3. Check if the input format matches what the subagent expects
4. Consider if the task is too complex for the assigned model

---

_Last updated: Initial creation_
