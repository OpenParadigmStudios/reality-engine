# Reality Engine Documentation

Reality Engine is a platform for running narrative-heavy tabletop RPG campaigns. It tracks what happens in your world, handles the bookkeeping, and lets consequences ripple through your game automatically.

---

## Thinking in Reality Engine

Using Reality Engine means shifting how you think about your campaign. Instead of scattered notes and spreadsheets, you'll organize your game around three core ideas:

### Your world is a collection of Things

Every character, faction, location, clock, item, and relationship in your game is a **GameObject**. These are the nouns of your world — the stuff that exists and can be referenced, changed, and connected.

A GameObject has a **Kind** that defines what it is and what properties it has. A "Character" Kind might track stress and harm. A "Faction" Kind might track tier and hold. You define the Kinds that matter for your game.

### Everything that happens is an Event

When something changes in your world, that change is recorded as an **Event**. Events are permanent and immutable — they form the complete history of your campaign.

This is different from overwriting data. Instead of "Whisper's stress is now 4," you record "Whisper gained 2 stress during the ghost heist." The current state is always calculated from the full history of events.

Why does this matter? Because you can always answer "how did we get here?" You can trace back through the history, see cause and effect, and understand the story of your campaign.

### Automatic reactions are Triggers

A **Trigger** watches for certain events and creates new events in response. They're the "when X happens, do Y" rules of your game.

When downtime happens, advance all faction clocks by 1. When a character takes their fourth harm, mark them as incapacitated. When the Bluecoats' clock fills, they raid the crew's lair.

Triggers let you encode the mechanical cause-and-effect of your game so the system handles the bookkeeping while you focus on the fiction.

---

## Mapping Your Game

When you bring your campaign into Reality Engine, you'll make decisions about what becomes what.

### What becomes a GameObject?

Anything you'd want to track, reference, or change over time:
- **Characters** — PCs and NPCs
- **Factions** — Organizations with goals and resources
- **Locations** — Places that matter to the fiction
- **Clocks** — Progress trackers, countdowns, threats
- **Items** — Significant equipment or artifacts
- **Relationships** — Bonds, debts, rivalries between entities

If you'd put it on an index card or a wiki page, it's probably a GameObject.

### What becomes an Event?

Any change that matters to the fiction or mechanics:
- Taking or healing harm
- Gaining or spending resources
- Clocks advancing or completing
- Relationships forming or breaking
- Factions making moves
- Time passing (downtime, seasons, years)

If you'd want to remember it happened, it's probably an Event.

### What becomes a Trigger?

Any automatic consequence or mechanical rule:
- "When downtime begins, advance all faction projects"
- "When stress reaches 9, trigger trauma"
- "When the score ends, notify the GM to resolve entanglements"

If you'd otherwise have to remember to do it manually every time, it's probably a Trigger.

---

## Example: A Blades in the Dark Campaign

Here's how a Blades campaign might map to Reality Engine:

**GameObjects:**
- The crew (with tier, rep, heat, coin, claims)
- Each PC (with stress, trauma, harm, abilities)
- Factions (Bluecoats, Lampblacks, Red Sashes, etc.)
- Faction clocks ("Bluecoats investigate the crew")
- Locations (Doskvol districts, the crew's lair, faction HQs)

**Events:**
- "Whisper took 2 stress during the Deathlands heist"
- "Crew gained 4 coin from the score"
- "Lampblacks clock advanced 1 segment (now 4/8)"
- "Downtime began after the Charterhall job"

**Triggers:**
- "When downtime begins, advance all active faction clocks by 1"
- "When a PC's stress reaches 9+, trigger trauma and reset stress"
- "When heat reaches 4+, alert GM about impending entanglements"

The result: you run the score, record what happened, and Reality Engine handles the faction turns, reminds you about consequences, and keeps the full history of how Doskvol has changed.

---

## Learn More

- [Events](concepts/events.md) — How history works and how changes cascade
- [Game Objects](concepts/game-objects.md) — Characters, factions, and everything in your world
- [Glossary](reference/glossary.md) — Plain-language definitions of key terms

---

*Reality Engine is currently in development.*
