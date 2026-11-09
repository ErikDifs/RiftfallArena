# Ranks and Pickups

Ability Ranks are the only upgrade and rarity system for the first version. There is no separate Common, Rare or Epic layer.

[Milestone 2](milestone-2-tickets.md) caps Ranks at 2 through a config value, with every world Pickup at Rank 1. Rank 3 and its behaviour changes come later.

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

**Rift Wardens** ([Ticket 23](milestone-2-tickets.md#23-rift-warden)) are an exception to "stronger monsters produce Rank 2". Each killed Warden drops exactly one ordinary **Rank 1** Ability Pickup, pre-dealt at Match setup (uniform random, distinct across the Match's Wardens) as bonus supply on top of the 24-drop Cache economy. Rank 2 comes only from collecting a duplicate, as for every other source. Milestone 2's Rank cap of 2 stands, and Rank 3 is not reachable.

### Monster Families

Each family has recognizable territory or encounter locations and a themed, weighted Drop Table. Players learn where Build pieces come from, players pursuing compatible Builds cross paths, and contested areas become organic PvP objectives. For example: "I have Tidal Bolt, so I should head toward the Storm area for Arc Burst."

| Family | First Drop Table |
|---|---|
| Tide creatures | Primarily Tidal Bolt, with room for a small chance of a compatible future Water or Rift Ability |
| Storm creatures | Primarily Arc Burst |
| Frost creatures | Primarily Frost Lance |
| Stone/Rift creatures | Seismic Slam, Rift Wall and Rift Step |

### Rift Caches

Rift Caches sit along useful routes, including routes from the spawn points. They are world objects available to everyone, not private rewards for losing players, so others can steal or contest them. This creates natural catch-up without explicit comeback buffs.

### Starting state and fast opening

Players begin each Match with a Basic Attack, a Dash and empty Ability Slots, with no random starting Ability. Milestone 2 has two slots, on Q and E. The long-term number is unresolved, with three kept as a likely test, as set out in [Milestone 2](milestone-2-tickets.md).

While a player has at least one empty Ability Slot, their next eligible monster kill guarantees a Pickup from that family's Drop Table. Normal chance-based drops then resume. The rule is configurable, because playtesting may show that filling the slots needs to happen faster. It applies only to normal eligible monster rewards, so killing trivial or invalid targets cannot generate extra drops. A Rift Warden is not an eligible monster for this rule.

## Pickup readability and ownership

- Players must be able to identify a Pickup's Ability and Rank in the world from a reasonable combat distance, without hovering or opening a menu.
- Pickups are first-come, first-served, and monster drops are not reserved for the player who dealt the most damage. Stealing, contesting and intercepting drops are intentional.

## Collecting a Pickup

| Player's situation | Result |
|---|---|
| Does not own the Ability, has a free Ability Slot | Acquires it at the Pickup's Rank. |
| Does not own the Ability, Kit is full | Can deliberately replace a chosen Ability, which drops at its current Rank. Milestone 2 uses holding F and pressing Q or E, as set out in [Milestone 2](milestone-2-tickets.md). |
| Owns the Ability below Rank 3 | `newRank = min(3, max(currentRank + 1, pickupRank))` |
| Owns the Ability at Rank 3 | Cannot collect it, and the Pickup stays in the world. |

The formula means duplicates always help, while a genuinely high-Rank Pickup is still exciting.

## Death

Superseded by one life per Match ([ADR-0007](adr/0007-one-life-arena-match.md)). The old rule, where the highest-Rank Ability lost a Rank and a Rank 1 copy dropped while the player respawned with their slots, is kept in that ADR as the rejected alternative. An Eliminated player drops their whole Kit, each Ability as its own Pickup at its current Rank, as settled in [Milestone 2](milestone-2-tickets.md).

## Architecture rule

Rank belongs to the owned Ability instance and to Pickup data. The Ability implementation stays shared and data-driven: it reads its current Rank and derives its configuration and behaviour from that. Do not create separate per-Rank Abilities or systems, such as `TidalBoltRank1` and `TidalBoltRank2`.

Monster Families, Drop Tables, Pickups, Rank weighting and eligible reward conditions are all data-driven. Individual monsters reference a family or drop definition rather than implementing their own loot logic.
