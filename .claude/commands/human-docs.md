# Human Documentation Generator

You are in **Documentation Generation mode** for the Reality Engine project.

Your goal is to transform agent-centric spec documents into human-readable documentation.

**Target**: $ARGUMENTS

If no argument provided, generate docs for all completed specs (🟢 status in MASTER.md).

---

## Philosophy

Agent specs are written for LLMs and developers — they contain:
- Technical decision blocks
- Open questions and TODOs
- Implementation details (schemas, code snippets)
- Status tracking and cross-references
- Rationale aimed at implementers

Human docs are written for **users, GMs, and curious humans** — they should contain:
- Plain language explanations of concepts
- What things *do*, not how they're built
- Examples and analogies
- Practical guidance on usage
- No code, no schemas, no implementation jargon

---

## Context Loading

Before generating, read:
1. `/spec/MASTER.md` — understand overall system
2. `/spec/glossary.md` — canonical terms (but simplify for humans)
3. Target spec document(s)
4. Any existing human docs in `/docs/` to maintain consistency

---

## Transformation Rules

### Remove
- Status indicators (🔴🟡🟢🔄)
- "Last interrogated", "Depends on", "Depended on by" metadata
- Open questions and `[ ]` checkboxes
- Code blocks (Python, YAML schemas)
- Technical rationale aimed at implementers
- Decision blocks in their structured format
- References to PRD, MVP milestones, implementation phases
- Anything that references "the agent" or "interrogation"

### Transform
- **Decision blocks** → Prose explanations of how the system works
- **Technical terms** → Plain language (or brief parenthetical definitions)
- **Field lists** → Narrative descriptions of what information is tracked
- **Canonical types** → Examples of what can happen in a game
- **Constraints/invariants** → Rules explained in terms of gameplay

### Add
- **Examples** from actual play scenarios
- **Analogies** to familiar concepts (index cards, spreadsheets, etc.)
- **"Why this matters"** context for GMs/players
- **Practical tips** where relevant

### Preserve
- Core concepts and their relationships
- The narrative voice of the system
- Logical structure (but reorganize for readability)

---

## Output Structure

Generate documentation in `/docs/` mirroring the spec structure but reorganized for humans:

```
docs/
├── README.md                    # "What is Reality Engine?" overview
├── concepts/
│   ├── events.md               # How the history system works
│   ├── game-objects.md         # Characters, factions, locations, etc.
│   ├── actions-and-approval.md # How players and GMs interact
│   ├── triggers.md             # Automatic consequences
│   └── feeds.md                # What you see and when
├── guides/
│   ├── for-gms.md              # GM-focused practical guide
│   ├── for-players.md          # Player-focused guide
│   └── for-designers.md        # Creating game paradigms
└── reference/
    └── glossary.md             # Human-friendly term definitions
```

---

## Writing Style

### Tone
- Warm but not patronizing
- Confident but not arrogant
- Practical, grounded in actual play
- Assumes intelligence, not technical background

### Structure
- Short paragraphs (3-4 sentences max)
- Headers that are questions or action-oriented ("How Events Work", not "Event System")
- Examples early and often
- No bullet points for prose explanations (save for actual lists)

### Avoid
- Passive voice where active is clearer
- Jargon without explanation
- "The system" as subject (prefer "Reality Engine" or "the campaign")
- Implementation details masquerading as features
- Marketing speak or hype

### Examples of Good Transformations

**Spec (agent-centric):**
```markdown
### Event Purpose
- Atomic, immutable record of world state change
- Append-only (never modified or deleted)
- Only **approved** events affect projections or trigger execution
```

**Human doc:**
```markdown
## Everything That Happens Gets Recorded

When something changes in your campaign — a character takes damage, a faction gains territory, a clock ticks forward — Reality Engine records it as an **event**. These events form a complete history of your campaign, like a detailed journal that never forgets.

Once something happens, it can't be erased or rewritten. This might sound limiting, but it's actually freeing: you always have a clear record of what led to where you are now. If a player asks "wait, when did the Duke betray us?", you can find that moment.
```

**Spec (agent-centric):**
```markdown
### Canonical Event Types (From PRD)
- `object.create` — new GameObject instantiated
- `meter.delta` — numeric value adjusted by amount
- `link.add` — relationship created
```

**Human doc:**
```markdown
## What Kinds of Things Get Recorded?

Reality Engine tracks several types of changes:

**New things appearing**: When you create a new character, introduce a faction, or establish a location, that's recorded. The campaign now has something it didn't have before.

**Numbers changing**: Health dropping, clocks ticking, resources depleting, influence growing — any time a tracked number goes up or down, that's an event.

**Relationships forming or breaking**: When two factions become allies, when a character joins a crew, when a bond forms between NPCs — these connections are tracked as events too.
```

---

## Process

1. **Assess the spec**: Is it complete enough (🟢 or solid 🟡) to document?
   - If 🔴 or sparse 🟡, note that human docs would be premature
   
2. **Extract the essence**: What does a human need to understand about this concept?

3. **Find examples**: What would this look like in actual play?

4. **Write the draft**: Transform spec content using the rules above

5. **Review for jargon**: Read it as if you've never seen a spec document

6. **Place the file**: Put in appropriate location in `/docs/`

7. **Update cross-references**: Ensure links between human docs work

8. **Report what was generated**: List files created/updated

---

## Single Document Mode

If `$ARGUMENTS` is a specific spec file (e.g., `spec/domains/events.md`):
- Generate only that document's human equivalent
- Place in appropriate `/docs/` location
- Note any prerequisite concepts that should be documented first

## Full Generation Mode

If `$ARGUMENTS` is `all` or empty:
- Check MASTER.md for all 🟢 specs
- Generate in dependency order (primitives first)
- Create the full `/docs/` structure
- Generate README.md as the entry point

---

## Output Report

After generation, report:

```markdown
## Human Documentation Generated

**Files created**:
- `docs/concepts/events.md` — from `spec/domains/events.md`
- ...

**Files updated**:
- `docs/README.md` — added link to new concept

**Skipped** (spec not ready):
- `spec/domains/triggers.md` — status 🔴, needs interrogation first

**Suggested next steps**:
- Review generated docs for accuracy
- Add more examples from your actual campaigns
- Run `/interrogate` on skipped specs to enable their documentation
```

---

## Example Invocations

```
/human-docs spec/domains/events.md
→ Generates docs/concepts/events.md

/human-docs all
→ Generates all docs for completed specs

/human-docs guides/for-gms
→ Generates the GM guide (pulls from multiple specs)
```

---

*This command transforms agent-centric specifications into documentation humans will actually want to read.*
