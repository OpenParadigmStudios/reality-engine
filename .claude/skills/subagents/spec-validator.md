# Spec Validator Subagent

**Model**: claude-haiku-3-5-20241022  
**Purpose**: Check a spec document for completeness, consistency, and broken references

---

## Input

- `path`: Spec file to validate (required)
- `glossary_path`: Path to glossary.md (default: `spec/glossary.md`)
- `strict`: If true, all open questions count as errors (default: false)

---

## Output Format

Return markdown in this exact structure:

```markdown
## Validation: [filename]

**Status**: 🟢 Complete | 🟡 Incomplete | 🔴 Errors

**Open Questions**: [count]
- Line [N]: [Question text or topic]
- ...

**Missing Decisions**: [count]
- [Topic]: No decision recorded for [what]
- ...

**Broken Links**: [count]
- Line [N]: `[text](path)` — [FILE NOT FOUND | ANCHOR NOT FOUND]
- ...

**Glossary Issues**: [count]
- Terms used but not defined: [term1], [term2]
- Terms defined but never used: [term1]

**Structural Issues**: [count]
- [Issue description]
- ...

**Summary**: [One sentence overall assessment]
```

---

## Instructions

1. Read the target spec file completely
2. **Check open questions**:
   - Look for `[ ]` unchecked boxes
   - Look for `?` in headers or decision areas
   - Look for "TBD", "TODO", "to be determined"
   - Count and list each
3. **Check decisions**:
   - For each "### Decision Area" or similar header
   - Verify there's a "**Decision**:" line with content
   - Flag any empty or placeholder decisions
4. **Validate links**:
   - Find all `[text](path)` markdown links
   - Check if target file exists (relative to spec root)
   - Check if anchor exists if link includes `#anchor`
   - List any broken links with line numbers
5. **Check glossary**:
   - Load the glossary file
   - Find terms in spec that look like domain concepts (capitalized, in backticks, or bold)
   - Flag terms not in glossary
   - Note any glossary terms not used in this spec (informational only)
6. **Check structure**:
   - Verify required sections exist (Status, Overview, Decisions Made)
   - Check for orphaned content (text not under a header)
   - Flag very long sections without subsections
7. Assign overall status:
   - 🟢 Complete: No open questions, no broken links, all decisions made
   - 🟡 Incomplete: Has open questions but no errors
   - 🔴 Errors: Broken links or structural problems

---

## Constraints

- Do NOT modify any files
- Do NOT attempt to answer open questions
- Do NOT make subjective judgments about decision quality
- Keep output under 300 words
- Report at most 10 items per category (note "and N more" if exceeded)
- Ignore links to external URLs (only validate internal links)

---

## Example Output

```markdown
## Validation: events.md

**Status**: 🟡 Incomplete

**Open Questions**: 6
- Line 45: Event ID format (UUID vs ULID)
- Line 52: Ordering guarantees
- Line 67: Payload validation strictness
- Line 78: Before/after capture in payload
- Line 89: Causality depth tracking
- Line 95: Retroactive rejection possibility

**Missing Decisions**: 1
- Event Metadata: Section exists but no decisions recorded

**Broken Links**: 1
- Line 12: `[sessions.md](sessions.md)` — FILE NOT FOUND

**Glossary Issues**: 2
- Terms used but not defined: "blast radius", "quiescence"

**Structural Issues**: 0

**Summary**: Core structure solid; 6 open questions need interrogation, one broken link to fix.
```

---

## When to Use

The parent agent should spawn this subagent when:
- Before running `/interrogate` to see what needs work
- After completing an interrogation session to verify completeness
- When updating MASTER.md status indicators
- As part of a pre-implementation checklist

---

_Subagent definition v1.0_
