# Monsters

Normal monsters teach combat, provide Build progression, add pressure during PvP and create positioning problems. They should not be individually threatening to a competent player, and Gather must not become a PvE grind. They become dangerous when several are active, when the player makes mistakes, or when another player joins the fight.

## Family behaviours

Each normal family has one defining behaviour.

| Family | Behaviour | Teaches |
|---|---|---|
| Tide | Slow ranged attacker with a clear projectile Telegraph, applies Wet | Projectile dodging, Dash timing, Wet readability |
| Storm | Small, fast groups, weak individually | AoE, short-range control, avoiding being surrounded |
| Frost | Tougher, slower melee with a clearly Telegraphed attack that applies Frost | Spacing, the Frozen and Shatter interaction |
| Stone/Rift | Heavy, with a long wind-up charge or slam that applies Knockback | Timing, terrain positioning |

## Shared combat rules

Monsters use the same systems as players wherever practical: health, damage, Statuses, Reactions, Knockback, Frozen and Frost Immunity, and hit confirmation. There is no separate simplified elemental system for NPCs. For example, a Tide creature can make a player Wet, and another player can arrive and trigger the Wet Reaction with Arc Burst.

Player attacks can hit both players and monsters. Stray attacks pulling monsters into a PvP fight are intentional.

## Telegraphs

Every meaningful damaging monster attack has a readable wind-up, communicated through the world, character animation and VFX, not only through UI. A player paying attention should have a reasonable chance to avoid it.

## Targeting

- Monsters normally target the nearest valid player.
- Damage can make a monster switch target, using a simple, configurable Threat value.
- Monsters do not intentionally fight other monsters.

## Home Territory

Every monster or spawn group has a Home Territory. If pulled too far, it stops pursuing, returns toward home and recovers or resets if appropriate. Players must not be able to drag entire camps across the map.

## Elite

The Rift Warden (below) is a separate guardian tier and is **not** this Elite. Build only one Elite at first. It is recognizably based on an existing Monster Family, with more health or stronger tuning and one additional Telegraphed attack or behaviour. Complex boss-phase AI is left out for now.

## Rift Wardens (guardian tier)

The first mobile PvE enemy built is the **Rift Warden** ([Ticket 23](milestone-2-tickets.md#23-rift-warden), [design](rift-warden-design.md)), an optional, contested, **elite-tier (mini-boss-style) guardian**. It is **not** a normal monster and **not** a true boss, so the rules above for normal monsters apply to it only where stated. It is rare, tough (a competent solo player takes about 15 to 20 seconds), tied to a visible Ability reward and limited to one life per slot per Match. **Regular monsters** (this document's normal families) are a later tier: weaker, quicker to kill, more numerous and scattered through the Arena. They are specified in [Ticket 24](milestone-2-tickets.md#24-regular-monsters) ([design](regular-monster-design.md)) and reuse only what the Warden proves reusable. See "Regular monsters" below.

Where the Warden deliberately differs from the general rules here:

- **Knockback.** It is immune to forced movement and deals none, because the Arena has lethal edges. The Stone/Rift family's Knockback slam above is a trait of a later family, not of the Warden and not of the first regular monster (the Riftling deals no Knockback).
- **Movement.** It is an anchored, server-moved kinematic model with a small circular Home Territory and no pathfinding ([ADR-0008](adr/0008-pve-actors-are-kinematic-server-models.md)).
- **Individual threat.** It is meant to be individually dangerous enough to need attention, unlike a normal monster.
- **Reward.** Its Ability Pickup is pre-dealt at Match setup, always Rank 1, never a Drop Table roll, and never counts as an "eligible monster kill" for the fast-opening guaranteed-drop rule in [`ranks-and-pickups.md`](ranks-and-pickups.md).
- **Targeting and Threat.** Nearest valid player first, then a sticky target with a short recent-damage Threat, as above. Damage can wake it only from inside its contestable range.
- **Kill credit.** The Last Hit, as above. Contribution data is only logged, never used to split the reward.
- **Scheduler.** It uses one shared Heartbeat loop (decisions at about 5 Hz, motion every frame) instead of the scheduler wording below, and an Idle Warden does only cheap proximity checks.

"Elite Warden", "Boss" and "Crown Boss" stay reserved for genuinely larger future encounters, such as the one the Dais slot is reserved for.

## Regular monsters (first: the Riftling)

The first regular monster is the **Riftling** (placeholder name, [Ticket 24](milestone-2-tickets.md#24-regular-monsters), [design](regular-monster-design.md)): a small, weak, quick-to-kill melee chaser (about 3 to 5 seconds for a competent solo player). Its role is ambient PvP pressure first, so it comes to players and gets pulled into fights, and a small hidden Ability reward second. It is **not** a member of any family in the table above, so "Monster Family" and its weighted Drop Tables stay reserved for a later tier. Where it deliberately differs from the general rules here and from the Warden:

- **Territory.** A small circular Home with a hard leash. It never leaves the circle and never crosses a connector or follows across Shards, which is the one allowed reading of "returns toward home" here.
- **Movement.** The Warden's anchored, server-moved kinematic model with no pathfinding ([ADR-0008](adr/0008-pve-actors-are-kinematic-server-models.md)), so the "pathfinding when stuck" wording under "AI and movement" does not apply to it. Obstacle handling is a validated clear-floor Home.
- **Attack.** One narrow melee claw swing with a 0.45 s Telegraph, no Knockback and no Status.
- **Outside fire.** There is no outside-fire ward. A target beyond reach gets a short hold at the edge, then the monster Returns and resets fully, and hits during the Return do not re-engage it.
- **Reward.** Each monster has a hidden 12.5% flag pre-dealt at setup, for one Rank 1 Pickup. Most drop nothing. It is never a guaranteed or Drop Table roll, and it is not an "eligible monster kill" for the fast-opening rule in [`ranks-and-pickups.md`](ranks-and-pickups.md).
- **Population.** 8 per Match, one Home per outer Shard (4 singles, 2 pairs), none on the Crown, no respawn, no player-count scaling.

## AI and movement

- A central server-side AI scheduler updates monsters, rather than a continuous loop per monster.
- Active AI starts at around 5 Hz, and the rate is tunable.
- Sleeping monsters do effectively no AI work, and wake when relevant players enter range.
- Monsters move directly toward their target where terrain allows, without constant pathfinding.
- Pathfinding is used only when a monster is stuck or the environment requires navigation.
- Profile before adding more sophisticated optimisation.

## Kill credit

The Last Hit decides the authoritative killer, who receives kill-specific effects such as the fast-opening guaranteed drop (not in force for Wardens or regular monsters in Milestone 2). The resulting Pickup is still a world object that anyone can take.

Lightweight contribution data is also tracked, including which players damaged the monster and their recent damage. It is not used for reward splitting yet, but leaves room for assists, Elite rewards, anti-sniping rules, quests and statistics.
