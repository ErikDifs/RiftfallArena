# Statuses and Reactions as the shared interaction model

Ability interactions are the most likely source of Riftfall Arena's identity, so they come from one shared, data-driven system rather than individual ability scripts. Abilities only apply Statuses and deal typed effects, and the interaction system decides whether those inputs produce a Reaction. World Zones are not being built yet, but Reaction inputs must be able to accept either a character Status or a world or environment Status or Zone, so that something like an Oil ground zone can join the same system without a second combination framework.

## Rules

- Each Reaction definition decides what happens to its setup Status: consume it, remove one stack, replace it with another Status, or leave it active. Consuming it is the default for the first Reactions, not a global rule.
- Anyone can trigger another player's setup, including stealing it.
- Players and monsters follow the same interaction rules, so PvE teaches PvP.
- The server is authoritative over Statuses and Reactions. Clients may show lightweight predictive visuals where safe, but the payoff, bonus damage and crowd control happen only after Server Confirmation.
- A Status used for a Reaction must have a clear character or world visual that stays readable during combat, not only a UI icon. Readability outranks spectacle.
- The triggering hit owns the immediate damage and kill credit, but both Applied By and Triggered By are tracked for future assists, stats, quests or progression.
- Self-interaction is controlled by Reaction targeting rules, not special cases in combat code. Enemy attacks may trigger a Reaction on a Status you applied to yourself, while your own attacks do not trigger harmful Reactions on you unless an ability explicitly says so.

## First Reactions

- Water applies Wet. Lightning on Wet triggers a stronger Lightning Reaction.
- Frost progresses a target toward Frozen. A Shatter-capable hit on Frozen triggers Shatter.

## Considered Options

- **Full world simulation:** rejected because it is the hardest option to read, balance and run efficiently in a solo prototype.
- **Hand-written combos:** rejected because they scale badly and go against the data-driven principle.
- **Building World Zones now:** deferred until playtests support it.
