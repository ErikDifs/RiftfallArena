# Rift Warden: settled design

The reasoning behind [Ticket 23](milestone-2-tickets.md#23-rift-warden). The ticket is the build contract and holds every number. This file holds why. Every point was grilled with the developer on 2026-11-09 and settled. Terms are in [`CONTEXT.md`](../CONTEXT.md).

**Terminology.** A **Rift Warden** is an optional, contested, **elite-tier (mini-boss-style) PvE guardian**. It is not a normal monster and not a true boss. It is substantially tougher than ordinary PvE, rare, tied to a visible Ability reward, and takes a competent solo player about 15 to 20 seconds. **Elite Warden**, **Boss** and **Crown Boss** are reserved for genuinely larger future encounters (the Dais slot). **Regular monsters** are a separate, later tier.

## Point 1. Role

**Settled: an optional guarded reward (A-lite) with route pressure as a side effect (B).** The Warden is a mini-objective near a valuable inner-Shard route. The decision it creates: *spend time and health on a small advantage, knowing another player may arrive and steal the payoff.*

**Why not the others.** A pure route-pressure Warden has no readable payoff. A roaming opportunity Warden has weak PvP convergence and no clear stakes. A Warden that guards something unique becomes mandatory.

**Hard invariants, structural and playtested.** A player never needs to kill a Warden to reach the Crown, use any connector or the Safe spoke, complete the initial Q/E Kit, take part in PvP or survive the Collapse. No route or baseline progression step depends on Warden state. There is no unique Warden-only reward (no exclusive Ability, Relic, currency, unlock, key or buff).

**Population.** One Warden life per occupied slot per Match, no respawn (respawn would farm and undermine one-life pacing). The count scales once with the roster at Match start (table in the ticket), is never rebalanced after GO, and has no independent despawn timer: the Collapse is the lifecycle boundary.

## Point 2. Slots, selection and Territory

**Selection: Gate occupancy.** Each of the three authored slots belongs to one Gate. The rule counts occupied spawn pads per Gate: one Warden takes the least-occupied Gate, two take all but the least-occupied, three take all. Ties break randomly. No runtime geometry.

**Why.** No truly neutral slot exists for two players. The slots sit only on Safe-spoke Shards, and every opposite pair is one Safe-shard and one Risky-shard pad in two Gates, leaving one Gate empty. The empty Gate is the least-advantaged choice (about 125 and 107 studs at the +15 to +20 degree offset, a walk difference near 1 s). All other spread-table classes are rotations of each other, so seeded-random ties are fair in expectation. A scoring optimiser buys nothing.

**Territory: a circle around the authored Home marker.** Not a custom volume (a second description of the floor's edge) and not the whole Section (too large, reaches aprons). The Warden never intentionally leaves it, damage never moves or enlarges it, and it must lie wholly on Layer 2 floor, clear of lips, connectors, aprons, Sentries and collidable props. Its radius is a geometry-derived upper bound, not a design number. The ruin platform is about 14 to 20, so it may be nearer 7 to 9 than the early 12 estimate. That is acceptable.

**Spawn safety and fairness (static validators).** No spawn pad, Home Cache or Forward Cache inside automatic aggro, for all six pad locations. Ordinary connector traversal must not automatically engage the Warden. At the original +10 degree slot, Home was about 42 studs from its own pad, so aggro is capped by geometry. The preferred slot offset is +15 to +20 degrees toward the Gate seam (about 48 and 66 studs to own and partner pads), kept only if connector, apron, Territory, Forward Cache, lip, line-of-sight and ignoring-route checks pass. Otherwise the +10 degree slot stays and aggro and Territory are sized around the real geometry. Never distort the Gate to hit an angle.

**Safe versus Risky.** The Ticket 21 hypothesis (PvE on the Safe route, void on the Risky one) stays a hypothesis. If the Safe route becomes strictly worse, tune aggro, placement, leash and reach before moving the slot.

## Point 3. Awareness

**Settled: proximity plus line of sight to acquire, accepted damage always wakes it.** Retention does not need sight, so a column is not a reset switch. `AggroRadius` is tied to the Territory (`Territory + 4`), so it never wakes for players it cannot contest.

**The outside-fire gate.** A ranged player must not chip it from beyond its combat space. A hit is accepted only from an attacker within `TerritoryRadius + AttackReach` of Home, checked at authoritative acceptance. A rejected hit does no damage, applies no Status or Reaction, adds no Threat and does not wake it, and shows a distinct "warded" response. Reasons: the principle is "you can only hurt it where it can hurt you", and it makes drag, poke and reset-loop exploits impossible without timers.

**Targeting.** Sticky target. Threat is only the damage dealt in a short window. A challenger takes over only with materially more recent damage and a small absolute minimum (a bare multiplier would let any hit beat zero), after a switch cooldown. A committed attack never retargets.

**Disengage.** After a 4 s hold with no valid target the Warden walks home vulnerable and targetable. A valid player or accepted hit during that return re-engages it. Healing waits a few seconds after arrival and then runs gradually, so a third party can finish a weakened Warden. Statuses clear only on full reset, never as a free cleanse.

**Collapse Warning.** Warning ground is completely normal, so the Warden acts normally until its Section is removed.

## Point 4. Combat

**Settled: a slow melee bruiser with one readable attack and one committed anti-kite area attack.** Exactly two attacks. No projectiles, Status, Root, Slow, Stun, leap, teleport or player Knockback in v1. Its pressure comes from occupying space, forcing movement, creating commitment windows and distracting players in PvP.

**Why this.** A ranged punish needs a bigger damage envelope and a projectile system. A control kit makes ring-outs cheap beside lethal edges and lets the Warden decide PvP fights.

**Reading and fairness.** Every attack has a Telegraph with the aim or centre locked at its start, one authoritative impact moment, and a real rooted recovery. Hit shapes hit every valid player inside them, so a Warden complicates PvP without deciding it. Evade, Shield and normal damage rules apply with no Warden exceptions. A Warden kill gives no Kill Recovery.

**Forced movement.** The Warden is immune to forced movement (Tidal Knockback, Hook pull) and cannot fall. It still takes damage, Wet and Reactions. Hits that cannot move it get distinct "movement resisted" feedback. This is a per-target property, as for Sentries, not blanket Knockback immunity.

**Difficulty is tuning, not a pass criterion.** The 15 to 20 second fight and the 35 to 50 Health cost are balance targets. A strong player may take little damage.

**Watch item.** Player Basic Attack reach (8) exceeds Swing reach (7) and attacking does not slow movement, so a one-stud backstep may dodge Swings indefinitely. Tune in this order: a shorter Swing Telegraph, Swing reach 8 to 9, a small committed forward step at impact, then chase speed. Never a projectile or leap first. A slam-only band exists between about `Territory + 7` and the envelope. Tune its levers before any new attack: Slam cooldown, then Slam reach, then speed, then envelope size.

## Point 5. Reward

**Settled: a separate bonus deal; the 24-drop Cache economy is untouched.** Each killed Warden adds one ordinary **Rank 1** Ability Pickup (up to 1, 2 or 3 extra). A Warden consumed by the Collapse adds nothing. The strict six-copies-per-Ability invariant belongs to the Cache deal.

**Identity.** Uniform random over the four Abilities, distinct across the Match's Wardens, **pre-dealt at setup** from a dedicated stream so it never perturbs the Cache or spawn draws. No local-Shard constraint, no killer or collector logic. Every such rule would couple a world object to a person or to the Shard puzzle. For a full two-Ability Kit it is a rank-up about half the time and a replacement offer otherwise.

**Rejected.** Drawing from the 24 bag (killing a Warden would gate normal supply). A unique or guaranteed reward. XP, Kill Recovery, currency, Relics or a second Pickup.

**Visibility and stealing.** The Warden carries the Ability it will drop, shown with the Pickup colour and shape (two channels). The Pickup is immediately contested with no ownership delay, and Last Hit credit is attribution only. A kill seconds before the Section goes can drop a Pickup the Collapse then consumes. That is accepted.

**Geometry consequence.** The Warden can die anywhere in its circle and the drop spreads, so the Territory must keep the loot-safe lip margin (about 15 studs from a lip, from the actual Pickup behaviour). Safe reward spawn outranks a larger Territory.

## Point 6. Locomotion

**Settled: an anchored, server-moved kinematic model.** No Humanoid locomotion, no PathfindingService, no physics impulses, no client network ownership. See [ADR-0008](adr/0008-pve-actors-are-kinematic-server-models.md). This makes "players cannot push it", "forced movement cannot move it" and "it cannot fall" true by construction. Its body does not collide with characters, so it neither body-blocks nor shoves anyone, and its combat target is the separate fixed Hitbox ([ADR-0004](adr/0004-fixed-gameplay-hitbox-independent-of-appearance.md)).

**Rules.** It approaches an on-stage target to a spacing of about 5 studs (an approach target, never a repulsion radius) and holds at the Territory boundary for an outside target. It turns at a bounded rate, faces its target before an attack, and is rooted through Telegraph, impact and recovery. The combat circle contains no collidable props. If one is found, move it or the Territory, never add pathfinding.

**Safety nets, not the primary guarantee.** One downward floor check per movement step. A no-progress rule abandons a Chase or snaps a Return home. A genuine fault despawns it with no reward, no credit and a log line, so a bug can never be a loot strategy.

## Point 7. Architecture

**Settled.** One four-phase FSM (`Idle`, `Engaged`, `Returning`, `Dead`) with a nullable timestamp-driven action record. Telegraph, recovery, the no-target hold and regeneration are timestamps, not states. A pure `WardenRules` (selection, deal, transitions, maths) and a thin `Wardens` server module mirroring `Sentries`. One Heartbeat drives all Wardens: decisions at about 5 Hz, motion and timestamps every frame. No `task.delay`. `Match` calls setup after `assignSpawns` and clear at Results. A dedicated `wardenRandom`. A new `warden` target kind with a small target-level pre-damage veto. Attacks go through `castFromServer` with a players-only filter. One idempotent first-wins `finish(record, cause)`.

**Refinements from the consistency review.**
- **Pre-GO.** The phase stays Running during the Arrival Hold, and Sentry targeting checks `goAt`. Wardens must check `goAt` the same way. The target filter alone is not enough.
- **Collapse.** Ticket 22's rule is a boundary by accepted-at time, resolved lazily by `sync(now)`. The Warden loop and its veto call `sync(now)` first. A hit accepted at or after the removal boundary finds the Warden already gone (no kill, no reward). A hit accepted before it kills normally, and the Pickup may be swept. A pending Warden impact resolves only if accepted before removal.
- **Cleanup hook.** Ticket 22's object cleanup is internal to `Collapse`, so it must call `Wardens.removeSection(sectionId)`. The Warden never needs Collapse to inspect its internals.
- **Restore order.** Wardens, like Caches and Sentries, are rebuilt from markers only after Ticket 22's restore step.

**Not a generic framework.** No `MonsterService`, no behaviour trees. A second family decides what is genuinely reusable.

## Point 8. Presentation

**Settled: world-first, plus one gated bar.** A server-owned root and fixed Hitbox, with the whole look in a client `WardenVisuals` driven by attributes. Dormant Idle (hunched, dim seams, breathing core) reads as a guardian, not scenery. A wake cue (rise, flare, grind, one pulse ring) marks aggro. A segmented Territory ring marks the stage only, never the damage envelope. Swing is a locked wedge and Slam a locked disc, both filling to the authoritative impact time in hostile red, with a different shape, pose and sound so they differ without colour. Existing Evade and Shield feedback is reused. Warded and movement-resisted hits get distinct, separate responses. A slim, proximity-gated Health bar with no numbers or text is the only UI. Death turns the core into the real Pickup at once with a short fade. A Collapse or fault removal has no kill-like effect.

## Point 9. Telemetry

**Settled: a few events plus one `WardenSummary` per Match**, in the existing console `key=value` style, with no positional sampling. Reward fate is joined from normal Pickup lines through a `source` tag. Three tiers: correctness (must be zero or exact), balance evidence (observed), and role-level review triggers (initial signals only, not read before there are enough encounters). The Warden pass-by classification (`passByAggro`, `routeTaxDamage`) tests the Safe-route hypothesis directly.

## Accepted risks

- Three-player Matches cannot be perfectly symmetric (two Wardens, three players).
- At five players one Gate has a lone occupant but still gets a Warden.
- A Safe-shard player is nearer a Warden than a Risky-shard player. The offset experiment narrows this, and equality is not required. The requirement is **no unavoidable opening PvE tax**.
- Even one ordinary Pickup may feel obligatory to strong players. Telemetry decides, and nothing is pre-empted.
- A last-second kill can drop a Pickup the Collapse consumes.

## Deferred

- **Regular monsters** (weaker, quicker to kill, scattered through the Arena) are intentionally a separate ticket, now specified as [Ticket 24](milestone-2-tickets.md#24-regular-monsters) ([design](regular-monster-design.md)). Only the Warden infrastructure that proves reusable carries over, extracted as its first implementation step.
- An elite Dais Warden or boss, Match XP, Relics and Equipment, patrol, respawn, a ranged or leap attack, final art, audio and animation, and Rank 3 behaviour.
