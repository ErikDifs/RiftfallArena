# Match structure

This is the target structure for the first mode: a free-for-all where the last surviving player wins. Times are tuning targets, not fixed rules.

**Priority:** do not implement this before the [first combat prototype](first-combat-prototype.md) has proven the combat. Team modes can be considered after the core solo combat works.

## Phases

| Phase | Approximate time | What happens | Respawns |
|---|---|---|---|
| Gather | 0–5 min | Players explore, fight monsters and assemble their Build. The goal is experimentation and getting everyone into a viable combat state. | Quick |
| Clash | 5–9 min | Stronger monsters, events and Pickups pull players toward contested areas. PvP increasingly becomes the best way to gain power or deny it to others. | Somewhat slower |
| Rift Collapse | 9–12 min | The playable area collapses inward through Rift-themed world effects. | None. Death is permanent for the rest of the match. |

## Death before the Rift Collapse

Death has a real but recoverable cost. Dying must not create this spiral: death → weaker Kit → harder to recover → another death → even weaker.

- A player does not lose one of their three Abilities by default.
- The penalty is losing a Rank and dropping a Rank 1 Pickup, as set out in [Ranks and Pickups](ranks-and-pickups.md).
- The drop gives the killer and nearby players a reason to contest the death location.
- The respawning player always returns with a functional Kit.
- The design must stay flexible enough to test the more severe rule later, where an actual Ability drops.

## Catch-up

A player who respawns during Gather or Clash needs a reasonably quick path back to viability. Catch-up comes from natural opportunities in the world, such as:

- monsters near respawn routes
- guaranteed low-tier Upgrade opportunities
- weaker, uncontested PvE areas
- diminishing value from farming already-dominant locations, if needed

Explicit buffs for losing players are deliberately left out for now.

## Rift Collapse

- It is driven by time, not by remaining player count, so the match always reaches a conclusion even if players disconnect or avoid combat.
- It physically removes safe territory, so hiding cannot delay the end indefinitely.
- It should feel like unstable Rifts consuming the world, not a generic battle-royale storm. The exact implementation comes later.
