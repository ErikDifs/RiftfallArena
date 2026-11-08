# Future Abilities

Candidate ideas for a later roster, each with its [Casting Style](first-combat-prototype.md#casting-styles). **None of these are in the current milestone.** The [first combat prototype](first-combat-prototype.md) and [playtest plan](playtest-plan.md) decide when more Abilities may be added.

The design rule for the roster: Charging should remain a distinctive tactical mechanic, not the default casting pattern for every spell.

## Candidate roster

| Ability | Casting Style | Note |
|---|---|---|
| Tidal Bolt | Chargeable | In the first loop |
| Arc Burst | Chargeable | In the first loop |
| Frost Lance | Instant | Already in the prototype's later build order |
| Seismic Slam | Committed Cast | Already in the prototype's later build order |
| Rift Wall | Instant placement | Already in the prototype's later build order |
| Rift Step | Instant mobility | Already in the prototype's later build order |
| Rift Hook | Instant | New idea |
| Gale Kick | Instant | New idea |
| Stone Guard | Instant | New idea |
| Meteor Drop | Committed Cast | New idea |
| Rift Spear | Committed Cast | New idea |
| Rift Beam | Chargeable | New idea |

Blink Strike and Oil Flask are described below as ideas, but aren't in this candidate roster.

## Ideas

### Rift Hook

**Instant.** A short-range skill shot that creates a rift tether. On a hit, it pulls the target toward the caster and deals low or moderate damage. It counts as physical displacement, so it can interrupt Charging.

Purpose: pull enemies into Arc Burst or melee range, reposition players, interact with ledges and create aggressive combo opportunities.

### Gale Kick

**Instant.** A very short-range frontal blast with low damage and strong Knockback. Its main purpose is positioning rather than damage.

Purpose: push players away or off ledges, interrupt Charging and create space.

### Blink Strike

**Instant.** A short offensive teleport toward the aim direction, followed by a close-range strike.

Purpose: an aggressive gap closer that sets up Arc Burst or melee. It's conceptually different from Rift Step, which is primarily mobility.

### Stone Guard

**Instant.** A brief defensive Ability. It might first absorb one meaningful hit, or give a small temporary damage shield, but it is never full invulnerability.

Purpose: a defensive answer other than Dash.

### Meteor Drop

**Committed Cast.** Select a nearby ground location, and after a fixed Telegraphed wind-up a meteor hits the area. The wind-up is the cost, and it isn't Chargeable.

Purpose: area denial, punishing predictable movement, and a possible future interaction with World Zones such as Oil.

### Rift Spear

**Committed Cast.** A narrow, high-speed projectile with a short fixed wind-up, possibly piercing. The challenge is predicting the target during the wind-up, rather than Charging.

### Rift Beam

**Chargeable.** A narrow beam where Charging mainly increases range. Unlike Arc Burst, it stays narrow and doesn't greatly increase in width, which gives Chargeable Abilities different identities.

### Oil Flask

**Instant.** Throw a flask that creates an Oil World Zone. World Zones aren't being built yet, so this is a future expansion of the interaction system.

Possible future Reactions:

- Oil + Fire → Burning Ground
- Oil + movement or Knockback → slippery or altered movement

The goal is to expand Riftfall from character Status interactions into battlefield interactions, without replacing the existing Reaction system.

## To settle before building these

- **Pulls and Charging:** settled 2026-11-08. Knockback now means forced displacement away from or toward its source, so Rift Hook's pull is Knockback and ends a Charge under the existing rule. Rift Hook and Stone Guard are being built in [Milestone 2](milestone-2-tickets.md).
- **Oil and movement:** [ADR-0003](adr/0003-statuses-and-reactions-as-the-shared-interaction-model.md) has a typed effect meeting a Status. Movement or Knockback isn't a typed effect, so "Oil + movement" doesn't fit that model yet.
- **Fire:** no Fire effect type exists yet.
