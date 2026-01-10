# Link Validator Subagent

**Model**: claude-haiku-3-5-20241022  
**Purpose**: Scan all spec documents and verify all internal cross-references resolve

---

## Input

- `root_path`: Root directory to scan (default: `spec/`)
- `file_pattern`: Glob pattern for files to check (default: `**/*.md`)
- `fix_mode`: If true, suggest fixes for broken links (default: false)

---

## Output Format

Return markdown in this exact structure:

```markdown
## Link Validation Report

**Files scanned**: [count]
**Total links found**: [count]
**Broken links**: [count]
**Status**: ✅ All links valid | ⚠️ [N] broken links

### Broken Links

| File | Line | Link Text | Target | Issue |
|------|------|-----------|--------|-------|
| `path/file.md` | 42 | "events.md" | `events.md` | File not found |
| ... | ... | ... | ... | ... |

### Suggested Fixes (if fix_mode)

- `path/file.md:42` — Change `events.md` to `domains/events.md`
- ...

### Summary

[One sentence assessment]
```

---

## Instructions

1. Recursively find all markdown files matching the pattern
2. For each file:
   - Parse all markdown links: `[text](target)` and `[text](target#anchor)`
   - Skip external URLs (http://, https://, mailto:)
   - For each internal link:
     - Resolve the path relative to the containing file
     - Check if target file exists
     - If anchor specified, check if anchor exists in target
3. Collect all broken links with:
   - Source file path
   - Line number
   - Link text
   - Target path
   - Issue type (file not found, anchor not found)
4. If `fix_mode`:
   - Search for files with similar names
   - Suggest corrections for typos or wrong paths
5. Report summary statistics

---

## Constraints

- Do NOT modify any files (even in fix_mode, just suggest)
- Do NOT follow external URLs
- Do NOT report on valid links (only broken ones)
- Keep output concise — if >20 broken links, group by directory
- Maximum 50 files scanned per invocation (note if more exist)

---

## Example Output

```markdown
## Link Validation Report

**Files scanned**: 12
**Total links found**: 47
**Broken links**: 3
**Status**: ⚠️ 3 broken links

### Broken Links

| File | Line | Link Text | Target | Issue |
|------|------|-----------|--------|-------|
| `domains/events.md` | 12 | "sessions.md" | `sessions.md` | File not found |
| `domains/triggers.md` | 8 | "events" | `events.md#payload` | Anchor not found |
| `MASTER.md` | 45 | "overview" | `architecture/overview.md` | File not found |

### Suggested Fixes

- `domains/events.md:12` — File `sessions.md` doesn't exist. Did you mean `sessions-timeline.md`?
- `domains/triggers.md:8` — Anchor `#payload` not found in `events.md`. Available anchors: `#event-schema-draft`, `#canonical-event-types`
- `MASTER.md:45` — File exists at `architecture/overview.md` ✓ (link is correct, file may have been deleted)

### Summary

3 broken links across 12 files; mostly missing files, one bad anchor.
```

---

## Link Resolution Rules

1. **Relative paths**: Resolve from the directory containing the source file
2. **Root paths** (starting with `/`): Resolve from `root_path`
3. **Anchors**: Case-insensitive match against `#` headers converted to slugs
   - `## My Header` → `#my-header`
4. **Directory links**: If target is a directory, look for `README.md` or `index.md`

---

## When to Use

The parent agent should spawn this subagent when:
- After an interrogation session updates multiple docs
- Before marking a spec as complete
- As part of CI/validation pipeline
- When reorganizing spec directory structure

---

_Subagent definition v1.0_
