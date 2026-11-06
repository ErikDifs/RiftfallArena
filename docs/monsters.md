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

Build only one Elite at first. It is recognizably based on an existing Monster Family, with more health or stronger tuning and one additional Telegraphed attack or behaviour. Complex boss-phase AI is left out for now.

## AI and movement

- A central server-side AI scheduler updates monsters, rather than a continuous loop per monster.
- Active AI starts at around 5 Hz, and the rate is tunable.
- Sleeping monsters do effectively no AI work, and wake when relevant players enter range.
- Monsters move directly toward their target where terrain allows, without constant pathfinding.
- Pathfinding is used only when a monster is stuck or the environment requires navigation.
- Profile before adding more sophisticated optimisation.

## Kill credit

The Last Hit decides the authoritative killer, who receives kill-specific effects such as the fast-opening guaranteed drop. The resulting Pickup is still a world object that anyone can take.

Lightweight contribution data is also tracked, including which players damaged the monster and their recent damage. It is not used for reward splitting yet, but leaves room for assists, Elite rewards, anti-sniping rules, quests and statistics.
