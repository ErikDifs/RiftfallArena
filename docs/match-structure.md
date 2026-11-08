# Match structure

This is the target structure for the first mode: a free-for-all where the last surviving player wins. Times are tuning targets, not fixed rules.

**Priority:** do not implement this before the [first combat prototype](first-combat-prototype.md) has proven the combat. Team modes can be considered after the core solo combat works.

## One life

A player has one life per Match, as [ADR-0007](adr/0007-one-life-arena-match.md) sets out. Death ends their run: they're Eliminated, don't respawn into the Match, and can Spectate or Return to Hub. The last living player wins. The ADR keeps the superseded respawn model, with quick respawns in Gather, slower ones in Clash and a Rank-loss death penalty, in case playtesting shows one life is too punishing.

Respawning still belongs in the Hub and in Reference Kit combat testing, which has no Match.

## Phases

| Phase | Approximate time | What happens |
|---|---|---|
| Gather | 0–5 min | Lower-pressure exploring, loot and PvE, getting everyone into a viable Kit. |
| Clash | 5–9 min | Stronger monsters, events and Pickups pull players toward contested areas. PvP increasingly becomes the best way to gain power or deny it to others. |
| Rift Collapse | 9–12 min | The playable area collapses inward through Rift-themed world effects, so nobody can hide to the end. |

## Reaching a viable Kit

With no respawns to absorb early mistakes, Gather's goal of letting players experiment and reach a viable Kit has to come from:

- sufficient loot density
- sensible spawn placement
- forgiving early PvE
- map pacing
- possibly later catch-up mechanics

Don't restore respawns to solve it. Explicit buffs for losing players are also left out for now.

An Eliminated player drops their whole Kit, each Ability as its own Pickup at its current Rank, as settled in [Milestone 2](milestone-2-tickets.md).

## Rift Collapse

- It is driven by time, not by remaining player count, so the match always reaches a conclusion even if players disconnect or avoid combat.
- It physically removes safe territory, so hiding cannot delay the end indefinitely.
- It should feel like unstable Rifts consuming the world, not a generic battle-royale storm. The exact implementation comes later.

## Future outer loop

Recorded on 2026-11-08 as the intended direction. It isn't in scope, and shouldn't shape the current Match architecture.

**Riftfall Hub** → the player enters an Arena portal or queue circle → matchmaking forms a Match → players teleport to a fresh Arena server → the Match runs → results and rewards → players return to the Hub.

The Hub is a shared social and queue space that forms Matches in batches. With a Match size of X, X ready players form one Match, 2X form two running at the same time, and so on, while any remainder waits for the next batch. With a Match size of 8 and 19 ready players, 8 enter Match A, 8 enter Match B and 3 stay queued. Players who arrive after those Matches start join the waiting pool, never an active Match. An Eliminated player can keep Spectating their Match, or Return to Hub and join a later batch, but can never re-enter the Match they were Eliminated from.

Conceptually: Hub player pool → a batch of X players → an Arena Match → one-life run → winner or timeout → Spectate or Hub → the next available batch.

Milestone 2 runs only one active Match at a time, as a prototype constraint rather than this architecture.

The Hub could later offer separate activities, such as Casual Arena, Ranked Arena, Training or Practice, and possibly other Riftfall experiences.

### Ranked

For now, Ranked means a free competitive queue, with a rating or MMR, divisions and seasons, a leaderboard, and cosmetic or status rewards.

Don't design paid entry or Robux prizes, such as paying Robux to enter with the winner receiving Robux. That has platform-policy and economic implications, and should only be considered after confirming that Roblox explicitly permits the exact model. Future monetization shouldn't influence the current Match architecture.
