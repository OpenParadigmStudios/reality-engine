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

## Relationships With Context

Connections can carry tags that describe *how* things are related, not just *that* they're related.

Marcus isn't just "connected to" the Lampblacks—he's their "informant" and "founding member." The Dimmer Sisters aren't just "allied with" the Hive—it's a "blood oath" alliance. Cassandra's bond with Elena isn't just "exists"—it's marked as "strained" and "family."

These tags let you capture the nuance of relationships. When you're looking at a faction's allies, you can immediately see which ones are reliable and which ones are grudging. When you're reviewing a character's bonds, you can see the texture of those connections.

Tags are just simple labels—you can use whatever makes sense for your game. The system doesn't enforce any particular vocabulary; it just remembers what you tell it.

## Finding Related Things

Reality Engine can answer questions about your world: "Which stories does Marcus own?" "What characters are in the Docks district?" "Are there any active clocks threatening the crew?"

These questions work across your entire game. You're not limited to following links from one thing to another—you can ask broad questions and get answers that span the whole world.

This is especially powerful for calculated values. A faction's "active projects" can automatically include all stories where that faction is an owner and the story is tagged as active. A character's "enemies" can include everyone who has a bond with them tagged as "rival" or "vendetta." These collections update automatically as your world changes.

## Containing Things in Other Things

Some things naturally contain other things: rooms are in buildings, buildings are in districts, districts are in cities.

Every thing can optionally have a "parent" it lives inside. This creates a natural hierarchy you can navigate. "Show me everything in the Docks district" works automatically.

## Stories Within Stories

Stories are how you track narrative arcs—quests, character goals, faction schemes, ongoing plots.

Here's what makes stories special: **stories can contain other stories**. A campaign-level arc ("The Fall of Doskvol") can contain faction-level arcs ("The Lampblacks' Rise"), which can contain personal arcs ("Marcus's Revenge"), which can contain scene-level beats ("The Warehouse Confrontation").

Whether something is a "beat" or an "arc" depends on where you're standing. From the campaign level, "Marcus's Revenge" is just a beat. From Marcus's perspective, it's an entire arc with its own structure.

**A Story** tracks:
- What's happening (current description)
- A summary for activity feeds
- Who owns it (which characters or factions—and in what role)
- What state it's in (active, completed, abandoned, on-hold)
- What events have touched it (automatically tracked)

**Story ownership with roles.** A story can have multiple owners with different roles. The Lampblacks' faction project might have the faction as "primary" owner, with three characters tagged as "leads" and one as "saboteur." These roles help you remember who's involved and how.

When you want to see how a story evolved, you look at its child stories and related events. The full arc is preserved in the hierarchy—you can always trace how the narrative unfolded from campaign down to individual scenes.

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
