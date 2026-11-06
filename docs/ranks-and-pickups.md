# Ranks and Pickups

Ability Ranks are the only upgrade and rarity system for the first version. There is no separate Common, Rare or Epic layer.

## Ranks

| Rank | What it adds |
|---|---|
| 1 | The base Ability. |
| 2 | A numerical improvement, using data-driven Ability parameters. |
| 3 | A further numerical improvement, plus one meaningful behaviour change. |

Rank 3 changes should be readable and relatively small. They make an Ability exciting without turning it into an unrelated Ability. Example behaviour changes:

- Tidal Bolt pierces one target.
- Arc Burst chains to one nearby target.
- Frost Lance gains a small impact burst.
- Seismic Slam has a larger central Shatter area.

## Pickup Rank

A Pickup's Rank is also its rarity:

- Common monsters mostly produce Rank 1.
- Stronger monsters and events can produce Rank 2.
- Elites, bosses and highly contested rewards can produce Rank 3.

## Where Pickups come from

| Source | Produces | Purpose |
|---|---|---|
| Monster Families | Mostly Rank 1, from the family's Drop Table | The main intentional Build path |
| Rift Caches | At least Rank 1, from the broader Ability pool | Secondary recovery and exploration |
| Elites and events | Rank 2, occasionally Rank 3 | High-value rewards and PvP hotspots, increasingly important during Clash |
| Player deaths | Rank 1, under the death rules below | A reward worth contesting after PvP |

### Monster Families

Each family has recognizable territory or encounter locations and a themed, weighted Drop Table. Players learn where Build pieces come from, players pursuing compatible Builds cross paths, and contested areas become organic PvP objectives. For example: "I have Tidal Bolt, so I should head toward the Storm area for Arc Burst."

| Family | First Drop Table |
|---|---|
| Tide creatures | Primarily Tidal Bolt, with room for a small chance of a compatible future Water or Rift Ability |
| Storm creatures | Primarily Arc Burst |
| Frost creatures | Primarily Frost Lance |
| Stone/Rift creatures | Seismic Slam, Rift Wall and Rift Step |

### Rift Caches

Rift Caches sit along useful routes, including routes a respawning player might naturally take. They are world objects available to everyone, not private rewards for losing players, so others can steal or contest them. This creates natural catch-up without explicit comeback buffs.

### Starting state and fast opening

Players begin each Match with a Basic Attack, a Dash and three empty Ability Slots, with no random starting Ability.

While a player has at least one empty Ability Slot, their next eligible monster kill guarantees a Pickup from that family's Drop Table. Normal chance-based drops then resume. The rule is configurable, because playtesting may show that filling three slots needs to happen faster. It applies only to normal eligible monster rewards, so killing trivial or invalid targets cannot generate extra drops.

## Pickup readability and ownership

- Players must be able to identify a Pickup's Ability and Rank in the world from a reasonable combat distance, without hovering or opening a menu.
- Pickups are first-come, first-served, and monster drops are not reserved for the player who dealt the most damage. Stealing, contesting and intercepting drops are intentional.

## Collecting a Pickup

| Player's situation | Result |
|---|---|
| Does not own the Ability, has a free Ability Slot | Acquires it at the Pickup's Rank. |
| Does not own the Ability, Kit is full | Can hold to replace an existing Ability, which drops at its current Rank. |
| Owns the Ability below Rank 3 | `newRank = min(3, max(currentRank + 1, pickupRank))` |
| Owns the Ability at Rank 3 | Cannot collect it, and the Pickup stays in the world. |

The formula means duplicates always help, while a genuinely high-Rank Pickup is still exciting.

## Death before the Rift Collapse

1. Find one of the player's highest-Rank Abilities, choosing randomly if several are tied.
2. If it is above Rank 1, reduce it by one Rank.
3. Drop a Rank 1 Pickup of that Ability at the death location.

The player keeps the Ability Slot, so they always respawn with a functional Kit. If every Ability is already at Rank 1, the Kit is not reduced further, but a Rank 1 Pickup copy of one owned Ability still drops. This duplication is intentional, because PvP kills should produce something worth contesting. A player who owns no Abilities drops no Ability Pickup, which also reduces spawn farming.

### Anti-farming

A player who dies again within a short post-respawn protection window does not generate another Ability Pickup. The duration is configurable. Tracking repeated killer and victim pairs is left out for now, and can be added later if needed.

## Architecture rule

Rank belongs to the owned Ability instance and to Pickup data. The Ability implementation stays shared and data-driven: it reads its current Rank and derives its configuration and behaviour from that. Do not create separate per-Rank Abilities or systems, such as `TidalBoltRank1` and `TidalBoltRank2`.

Monster Families, Drop Tables, Pickups, Rank weighting and eligible reward conditions are all data-driven. Individual monsters reference a family or drop definition rather than implementing their own loot logic.
