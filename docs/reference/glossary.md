# Glossary

Plain-language definitions of Reality Engine terms.

---

## A

### Action
Something a player or GM can do that affects the world. "Heal a character," "Advance a faction's project," "Form a bond." Actions are the buttons you click, the choices you make.

### Archived
Marked as "no longer active" but not deleted. An archived character still appears in the history of everyone they interacted with. You just can't target them with new actions.

---

## C

### Cascade
A chain reaction of events. One event triggers others, which trigger more, until everything settles. "Downtime begins" might trigger a dozen faction advances, character recoveries, and clock ticks.

---

## D

### Draft
A proposed change waiting for approval. When a player wants to do something that needs GM sign-off, it becomes a draft first. The GM can approve, reject, or modify it.

---

## E

### Event
A permanent record of something that happened. "Marcus took 3 damage." "The Lampblacks advanced their project." "Downtime began." Events are the atoms of your campaign history—small, specific, and permanent.

---

## F

### Feed
A stream of recent activity. Your character's feed shows what's been happening to them. A faction's feed shows its recent history. Feeds are the primary way players see what's going on.

---

## G

### Game
Your campaign. One complete story being played out. All the characters, factions, events, and history belong to a single game.

### Game Object
Anything that exists in your world—a character, faction, location, clock, item, story. If it would be an index card in your GM binder, it's a game object.

---

## K

### Kind
A template defining a type of thing. The "Character" kind describes what fields a character has (HP, stress, bonds). The "Faction" kind describes what fields a faction has (tier, hold, projects). Kinds are defined by your game system.

---

## M

### Meter
A number with limits that can trigger reactions. HP (can't go below 0 or above max), stress (might cause trauma at 9), project progress (completes at 8). When meters hit their limits, interesting things can happen.

---

## N

### Nestable
A type of thing that can contain others of its own type. Stories are nestable—a story can contain other stories. Locations are nestable—a building contains rooms. This creates natural hierarchies where "parent" and "child" are relative terms.

---

## O

### Operation
The specific change an event makes. "Adjust HP by -3" or "Set stress to 6" or "Add bond to Marcus." One event might include several operations.

### Overflow
When you try to push a meter past its maximum. Healing a character at full HP creates overflow—the system records how much was "wasted" and triggers can react to it.

---

## P

### Paradigm
A rules package that defines how a game system works. A "Blades in the Dark" paradigm includes character kinds with stress and trauma, faction kinds with tier and hold, and actions like "Take Downtime." You can mix paradigms if they're compatible.

### Parent
A thing that contains another thing. A building is the parent of the rooms inside it. A district is the parent of the buildings in it. This creates a natural hierarchy for your world.

---

## R

### Ref (Reference)
A link from one thing to another. A character's "faction" field is a reference to a faction. A character's "bonds" field is a list of references to other characters. References let you navigate between connected things.

---

## S

### Signal Event
An event that announces something happened without directly changing anything. "Downtime begins" is a signal—it doesn't adjust any numbers itself. But triggers listen for it and create cascades of actual changes.

### Spec
The authoritative data about something—what you've deliberately set. A character's spec includes their constitution score, their name, their faction membership. Spec only changes through events.

### Status
Calculated data about something—what the system derives from spec. A character's status includes their max HP (calculated from constitution) and their bond count (calculated from their bonds list). Status updates automatically when spec changes.

### Story
A narrative arc being tracked over time. Stories can contain other stories, creating hierarchies from campaign-level arcs down to scene-level beats. Whether something is a "beat" or an "arc" depends on your perspective—it's all stories, nested inside each other. Stories track their owners (with roles), their state (active, completed, abandoned), and automatically collect related events.

### Tagged Ref
A relationship (reference) that carries context. Instead of just knowing that Marcus is "connected to" the Lampblacks, a tagged ref records that he's their "informant" and "founding member." Tags are simple labels that describe *how* things are related.

---

## T

### Trigger
An automatic reaction rule. "When downtime begins, advance all faction projects by 1." "When stress hits 9, roll for trauma." Triggers watch for specific events and respond with more events.

---

## U

### Underflow
When you try to push a meter below its minimum. Dealing damage to a character at 0 HP creates underflow. Triggers might react to this (like marking someone as dying).

---

## W

### World Domain
Everything in Reality Engine that's permanent and authoritative—all your game objects and events. The "source of truth" for your campaign.

### Workflow Domain
Everything that's temporary and collaborative—drafts waiting for approval, comments and notes, permission requests. The "discussion space" that feeds into the world domain.
