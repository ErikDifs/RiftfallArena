# One life per Arena Match

Decided 2026-11-08, while settling [Milestone 2](../milestone-2-tickets.md).

In the primary Arena Match, a player has one life. Every player starts a fresh run with an empty Kit, builds it from loot and PvE, and fights other players with it. Death ends the run: the player is Eliminated from the Match, never respawns into it, and can Spectate or Return to Hub. The last living player wins. The flow it's built for is **Hub → Match entry → a fresh run → loot, PvE and a Kit → PvP → death ends the run → Spectate or Return to Hub**.

The Match itself, not Roblox's respawning, decides whether each player is Alive, Eliminated and Spectating, or leaving for the Hub. Respawning still belongs in the Hub, in Reference Kit combat testing such as Stage 3, and possibly in alternative modes like deathmatch later. It never returns an Eliminated player to a primary Arena Match.

## What stays

Gather, Clash and the Rift Collapse keep their purposes. Gather is early, lower-pressure loot and PvE that gets players into viable Kits. Clash brings increasing conflict and convergence. The Rift Collapse is the final pressure against hiding. Only the respawn model under them changes, so none of them depends on cheap deaths or respawn timers any more.

## The superseded model

The working design before this decision, in [Match structure](../match-structure.md) and [Ranks and Pickups](../ranks-and-pickups.md), had:

- quick respawns during Gather, so players could experiment cheaply and everyone reached a viable combat state
- slower respawns during Clash, as fights grew more contested
- permanent death only in the Rift Collapse, which settled the winner
- a recoverable death penalty: the highest-Rank Ability lost one Rank, a Rank 1 copy dropped where the player died, and the player kept every slot, so they always respawned with a working Kit and couldn't fall into a death spiral
- an anti-farming window after a respawn, during which another death dropped nothing
- catch-up shaped around respawning: monsters and Rift Caches along respawn routes, and guaranteed low-tier Upgrades

Its reasoning still holds as a fallback. Death had a real but recoverable cost, kills still produced something worth contesting, and new players could learn the combat by dying without losing the session. If playtesting shows one-life runs are too punishing, this is the model to reconsider first.

## Considered Options

- **Respawns until the Rift Collapse (the model above):** superseded. A run with respawns has no clear stakes, a run can't end in a clean return to the Hub, and the death penalty needs its own machinery to avoid both farming and death spirals.
- **One life per Match:** chosen. Every Pickup and every fight carries the weight of the whole run, the loop ends cleanly in the Hub, and there's no respawn penalty to balance.

## Consequences

- Gather's original goal, letting players experiment and reach a viable Kit, now has to come from loot density, sensible spawn placement, forgiving early PvE, map pacing, and possibly later catch-up mechanics. Quietly restoring respawns isn't an acceptable fix.
- The Rank-loss death rule isn't carried forward. What an Eliminated player drops is a new decision, being settled in Milestone 2.
- Catch-up for a returning player no longer exists inside a Match, and nobody can join one late. Every player starts a Match at the same moment with the same empty Kit.
- Spectate and a Hub become part of the core loop, even in their smallest temporary form.
- An early death costs a player the rest of that Match, which matters most in small test groups.
