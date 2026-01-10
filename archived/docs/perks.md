# Perks

Perks are small bits of re-usable mechanical special rules that can be attached to Traits to let characters do things they normally wouldn't be able to.

They can do things like grant you extra dice on rolls, give you refunds on spent resources in certain conditions, give you extra slots or progress on meters on your character sheet, do special moves or actions in certain conditions, or other introductions of new rules to the normal way of resolving the game. Perks are conditional special cases.

They are typically attached to a Trait, and may either be a Basic or Advanced Perk on that Trait. A Basic Perk is one that anyone Character with that Trait will have, while an Advanced Perk is one that not all Characters with that Trait will have. Instead, every Trait with Advanced Perks has a list of all of the Perks that are available for purchase as an Advanced Perk, and then a Character may acquire that Advanced Perk through character advancement mechanisms.

Perks are created and managed in the system in a story narrative independent, purely rules related state, with the only narrative flavor in the description being extremely settings agnostic in a way that allows it to be hooked into easily by any Trait. Each Perk also has a line of text that describes how it relates to the particular Trait, which varies from Trait-Perk connection to Trait-Perk connection and gives guidelines on when it may apply. 

For example, both the Introspective and the Reckless Trait may give access to the Confident Perk. The difference in the narrative context line, though, may be the following:
Introspective's Confident: "Due to long time spent analyzing your own strengths and weaknesses, you may be Confident when you already feel you have a strong chance of success, defined as when you already have 3d or more in your dice pool for a roll."
Reckless's Confident: "You are Confident in your chances of success when you probably shouldn't be, letting you use this Perk whenever you have 1d or less in your dice pool for a roll."

Swordmaster's Specialty: "You may apply your Specialty whenever you are fighting with, evaluating, crafting, or otherwise doing anything involving a sword."
Scientist's Specialty: "Your Specialty applies whenever you are doing anything that your scientific knowledge would help in."
Strong's Specialty: "Your Specialty is in engaging your muscles to move, push, pull, crush, or smash things or otherwise do heroic acts of great strength."

Strong's Expanded Inventory: "Due to your great strength, you are easily able to carry more than other people, giving you Expanded Inventory."
Merchant's Expanded Inventory: "You are practiced at packing very efficiently, giving you Expanded Inventory."
Quartermaster's Expanded Inventory: "You're used to packing extra to account for the needs of the party as a whole, giving you an Expanded Inventory."

Motivator's Backup: "You easily fall into work harmony with anyone you share any Bond or Trait with, acting as excellent Backup for them."
Bond between two Character's Backup: "Because of the close relationship between the two of you, you may always act as Backup for each other when you work together."

Cook's Boost Morale: "You may Boost Morale during camp by cooking delicious meals for the party."
Quartermaster's Boost Morale: "You always keep extra treats, creature comforts, and the little necessities of life that allow you to Boost Morale for anyone you're making camp with."
Bond between two Character's Boost Morale: "You act as a source of comfort and solidarity with your companion, allowing you to Boost Morale for each other."

Healer's Make Healing Potion: "You are trained to Make Healing Potions, and can manufacture them if given access to healer supplies."
Alchemist's Make Healing Potion: "One of the main recipes you have access to allows you to Make Healing Potions if you have access to an alchemy lab."
Survivalist's Make Healing Potion: "You know how to Make Healing Potions out of herbs and plants that you can forage in the Wilds."


List of Perk examples:
```
Specialty
- description: |
    When you Push Yourself using this Perk, it costs 1 Stamina instead of 2

Expanded Inventory
- description: |
    Gain +2 Gear slots

Effortless
- description: |
    When you Push Yourself using this Perk, you recover 1 Stamina if the results of the roll are a Success or Crit

Confident
- description: |
    Get +1d on a roll, but pay Stamina cost based on result:
    - Botch: Lose 3 Stamina, +1 per 1 rolled
    - Fail: Lose 3 Stamina
    - Mixed: Lose 2 Stamina
    - Success: Lose 1 Stamina
    - Crit: No cost

Practiced
- description: |
    If you Botch a roll this Perk applies to, you may spend 1 Stamina per 1 rolled to turn it into Failure instead

Careful
- description: |
    Spend 1 Stamina before rolling, but if you had a Mixed result for your roll, it's treated as Success instead

Second Wind
- description: |
    Spend 1 Stamina before rolling. Recover Stamina based on result:
    - Botch, Fail, Mixed: No Stamina recovered
    - Success, Crit: Gain +1 Stamina per 6 rolled

Supporter
- description: |
    If you Aid an ally and they Succeed or Crit on their roll, refund the 1 Stamina it cost to Help them.

Helpful
- description: |
    If you Aid an ally, you may spend an additional 1 Stamina to give them an additional +1d on their roll.

Backup
- description: |
    If you Aid an ally, they may treat a Mixed result as Success instead

Make Healing Potions
- description: |
    Spend 1 Free Time or 1 Coin to gather and process the ingredients required to create a Healing Potion consumable with 5 charges (this can be split)

Make Fire Bombs
- description: |
    Spend 1 Free time or 1 Coin to gather and process the ingredients required to create a Fire Bomb consumable with 5 charges (this can be split)

Make Poison Flasks
- description: |
    Spend 1 Free Time or 1 Coin to gather and process the ingredients required to create a Poison Flask consumable with 5 charges (this can be split)

Press Your Advantage
- description: |
    Spend 1 Momentum in a Challenge before any character rolls to add to the result:
    - Botch: Lose 1 Momentum per 1 rolled
    - Failure, Mixed: Nothing extra happens
    - Success, Crit: Gain 1 Momentum and lose 1 Tension per 6 rolled

Turn the Tables
- description: |
    In a Challenge, spend all (1+) Momentum to roll current Tension as dice pool:
    - Botch: Gain 3 Tension, +1 per 1 rolled, full discharge on user
    - Failure: Gain 2 Tension
    - Mixed: Refund 1 Momentum, gain 1 Tension
    - Success, Crit: Each 6 rolled removes up to 3 Tension

Bodyguard
- description: |
    When a target ally is about to make a defensive roll, you may pay 1 Stamina to intervene and make the roll on their behalf with your own dice pool with the following added effect:
    - Botch: You both suffer the consequences of your defense roll 
    - Failure: They suffer the consequences of your defense roll 
    - Mixed: You suffer the complications of your defense roll 
    - Success, Crit: You successfully defend both your target ally and yourself with your defense roll

Boost Morale
- description: |
    When you make camp, you may spend 1 Supply consumable charge to recover 1 Stamina for every character

Forager
- description: |
    When you make camp, you may spend 1 Stamina to gather 3 Supply consumable charges

Maintenance
- description: |
    When you make camp, you may spend 1 Stamina to give one piece of Gear +1d to all rolls using it the next day

```

