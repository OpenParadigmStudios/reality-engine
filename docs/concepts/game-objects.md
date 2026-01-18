# Things in Your World

Reality Engine tracks everything that exists in your game world. Characters, factions, locations, clocks, items, bonds—all of these are "game objects."

Think of them like index cards in a well-organized GM binder. Each card has a type (is it a character? a faction?), some information written on it, and maybe some references to other cards.

## What Kinds of Things Can You Track?

You can track anything you'd write on an index card:

- **Characters** — PCs and NPCs alike
- **Factions** — Organizations, gangs, guilds, governments
- **Locations** — Districts, buildings, rooms, planets
- **Clocks** — Progress trackers, countdowns, project timelines
- **Items** — Equipment, artifacts, resources
- **Stories** — Quests, character arcs, ongoing plotlines
- **Sessions** — Your actual play sessions

And anything else your game needs. The system is flexible—you define what kinds of things exist in your world.

## Templates for Things

Each type of thing follows a template (called a "Kind"). The Character template might include fields for HP, stress, and faction membership. The Faction template might include tier, hold, and project progress.

These templates are defined by your game system. A Blades in the Dark campaign will have different templates than a Dungeon World campaign.

## What You Set vs. What the System Calculates

Every thing in your world has two types of information:

**What you set** — The character's constitution score. Their faction membership. Their name and description. This is the authoritative data that only changes through events.

**What the system calculates** — Maximum HP (based on constitution). Total bond count. Whether they're wounded. This derived data is computed automatically from what you set.

For example, you might set a character's constitution to 4. The system calculates that their max HP is 17 (based on a formula like "constitution × 3 + 5"). When constitution changes, max HP automatically updates.

## Numbers That Matter: Meters

Some numbers are special—they have limits and can trigger reactions when they change.

**Hit points** can't go below 0 or above maximum. **Stress** might trigger trauma when it hits 9. **Project progress** might trigger completion when it hits 8.

These bounded, reactive numbers are called "meters." When you try to push a meter past its limit:
1. The value stops at the limit (no negative HP)
2. The system records how much "overflow" or "underflow" there was
3. Triggers can react to that overflow (like converting excess healing to armor)

## Relationships Between Things

Things in your world connect to each other:

- A character **belongs to** a faction
- A faction **operates in** a district
- A character **has bonds with** other characters
- A room **is inside** a building

These connections are tracked explicitly. When you ask "who are Marcus's allies?" the system knows. When you ask "what characters belong to the Lampblacks?" the system knows.

Connections can be one-to-one (a character has one home district) or one-to-many (a character has many bonds).

## Containing Things in Other Things

Some things naturally contain other things: rooms are in buildings, buildings are in districts, districts are in cities.

Every thing can optionally have a "parent" it lives inside. This creates a natural hierarchy you can navigate. "Show me everything in the Docks district" works automatically.

## Stories and Story Beats

Stories are how you track narrative arcs—quests, character goals, faction schemes, ongoing plots.

Unlike most things that change through meter adjustments ("HP went down by 3"), stories evolve through **story beats**. Each beat is a snapshot of what happened in that story at that moment.

**A Story** tracks:
- What's happening (current description)
- Who owns it (which character or faction)
- Who's involved
- What state it's in (active, completed, abandoned)

**A Story Beat** records:
- What just happened
- A short summary for activity feeds
- Links to the events or characters that prompted it

When a story progresses, you add a beat. When it resolves, you add a final beat and mark it complete. The full arc is preserved—you can always see how a story evolved.

## Removing Things (But Keeping History)

Characters die. Factions disband. Locations get destroyed.

When you "archive" something, it's marked as no longer active, but it isn't deleted. All the history involving that thing remains intact. You can still see that Marcus was bonded with Elena before she died, even though Elena is now archived.

When you archive something, the system also flags any now-broken relationships. If Elena was Marcus's contact in the Lampblacks, that relationship gets flagged so the GM can decide what happens next.

## Permissions: Who Sees What

Each thing in your world can have visibility settings:
- GMs see everything
- A character's player sees their character fully
- Other players might see limited information

If you can see a thing, you can see all of it. There's no partial visibility within a single thing—that's handled by what gets shown in activity feeds and summaries.

---

Next: [Glossary](../reference/glossary.md) — Plain-language definitions of all the terms used in Reality Engine
