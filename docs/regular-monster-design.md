# Regular monsters: settled design

The reasoning behind [Ticket 24](milestone-2-tickets.md#24-regular-monsters). The ticket is the build contract and holds every number. This file holds why. Every point was grilled with the developer on 2026-11-10, one at a time. The first regular monster has the placeholder in-world name **Riftling**.

**Terminology.** A **regular monster** is a weak, quick-to-kill, more numerous PvE enemy. It is not a [Rift Warden](rift-warden-design.md) and not a Monster Family member. "Monster Family" and its weighted Drop Tables stay reserved for a later tier with several themed families.

**Not built until Ticket 23 is.** Regular monsters reuse primitives proven by the Warden, so Ticket 24's implementation starts only after Ticket 23 (and Tickets 21 and 22) are built. The design can be finished first.

## Point 1. Role

**Settled: ambient PvP pressure and catalysts first, a small reward source second.** A regular monster is a small, mobile pressure source in contested space. It comes to players, makes noise and gets pulled into fights. The Warden is something a player chooses to fight, and a regular monster usually comes to the player. It is not a weaker Warden.

- **Commitment:** about 3 to 5 seconds for a competent solo player.
- **Lethality:** one rarely kills a healthy player alone. Danger comes from groups, PvP overlap and Collapse edges. No Knockback.
- **Reward:** most monsters drop nothing. A small, hidden, configurable chance of one ordinary Rank 1 Ability Pickup.
- **No Match XP in this ticket.** XP needs a level curve, level effects and a HUD, which is a progression system rather than an enemy ticket. At the time, nothing in the docs defined it. This ticket logs kills so a later ticket has data. That ticket, [Ticket 25](milestone-2-tickets.md#25-match-milestones), specified **Milestones instead of XP** ([ADR-0009](adr/0009-match-progression-through-one-time-milestones.md)), and a Riftling kill is one input to its Explore feat only.
- **No respawn.** The Match is short and Collapse removes ground, so respawn would mainly create farming.
- **Never gating.** No route, Cache, Kit step or the Crown requires killing one. They may sit close enough that fighting is sometimes the easiest choice.

**Why not the others.** Pure ambient clutter is "annoying mosquitoes". A pure progression source needs XP and invites grinding. A pure catalyst needs a reward or it feels like random interference. The hybrid, with the reward deliberately small, fits all three risks.

## Point 2. Behaviour archetype

**Settled: one melee chaser family.** A ranged regular monster is deferred. It would add projectile lanes and Telegraph clutter beside the Sentry and Tidal Bolt, and a mixed population is too much for a first ticket.

- **Noticing:** proximity plus line of sight to acquire. Accepted damage always wakes it. Retention needs no sight.
- **Leash:** a small circular Home Territory with a hard leash. The monster never leaves the circle and never crosses a connector or pursues across Shards. Players cannot drag it across the map, and it is Collapse-safe and deterministic.
- **Targeting:** one target at a time, a sticky target with recent-damage Threat. It joins PvP naturally, and it never fights other monsters.
- **Coordination:** independent. No pack AI, no formation, no dynamic avoidance. Pairs share a Home with authored slot offsets and a small approach offset.
- **Movement:** the Warden's anchored, server-moved kinematic model ([ADR-0008](adr/0008-pve-actors-are-kinematic-server-models.md)). No Humanoid locomotion, no `PathfindingService`, no character body-blocking.

The cost of the kinematic model is a placement cost: every Home needs a validated clear-floor circle with no blocking prop, lethal edge or connector overlap. That is preferred to adding navigation.

## Point 3. Combat envelope

**Settled: one narrow frontal claw swing.** No lunge (a gap-closer defeats "easy to disengage"), no leap, no second attack.

- A narrow locked wedge with one authoritative impact moment. Aim locks at Telegraph start with no tracking, so a sidestep or Evade is the answer.
- Reach is below the Basic Attack's, so skilled players can trade at reach. A one-stud backstep may dodge swings. That is a balance watch item, not a blocker: for a weak monster, being outplayed by clean spacing is acceptable.
- Evade and Shield are reused with no exceptions. No Status, Knockback, Root or Stun against players. Monsters take Wet and Reactions normally and are immune to forced movement.
- **No hard single-target filter.** The monster picks one target and aims at one target, and the swing can still hit another valid player inside the wedge, as the Warden's shapes do. This keeps one shared server-cast path with no special case.
- **Dogpile strategy:** a maximum of 2 monsters per Home, and a 0.3 to 0.7 s stagger on the **first attack after acquisition only**, never ongoing jitter, so pairs follow a deterministic cycle afterwards. Overlapping impacts are logged. A per-target Telegraph cap is added only if testing shows pairs routinely sync into unfair bursts.

## Point 4. Shared primitives

**Settled: extract only what both real implementations genuinely need, and no `MonsterService`.** Classified against the Ticket 23 plan and the Ticket 18 Sentries:

| Candidate | Class |
|---|---|
| Kinematic movement, floor-following, Territory clamp and Home-return maths | **Extract** (one pure primitive) |
| Sticky target and recent-damage Threat | **Extract** |
| The generic action and timestamp record (not the FSM) | **Extract** |
| The client Telegraph builder (wedge and disc) | **Extract** |
| A small PvE actor-id predicate (`warden`, `monster`, `sentry`) | **Extract** |
| Target registration, immovable handling | Reuse directly |
| Proximity and LOS helper | Reuse directly |
| `castFromServer` | Reuse directly |
| Status and Reaction participation | Reuse directly |
| `removeSection(sectionId)`, setup, clear and the Heartbeat loop | **Keep separate** per system |
| Telemetry | Reuse the sink, **keep summaries separate** |

**Process rule.** Ticket 24 starts by extracting only the proven shared primitives from the implemented Warden, with Warden behaviour and specs unchanged and green. If a supposed primitive is tangled with Warden-specific behaviour, do not force it: leave it in the Warden and write the monster version separately.

## Point 5. Leash and outside fire

**Settled: no Warden-style outside-fire gate.** Wardens defend an objective and therefore need anti-cheese protection. Regular monsters are weak ambient enemies, so ranged advantage is allowed as long as the reset prevents infinite free poking.

- Damage from anywhere is accepted and may wake the monster. The Territory stays one hard validated circle.
- A target is **unreachable** when it is beyond `TerritoryRadius + AttackReach` of Home. A target between the circle and that distance is contestable from the edge, and the monster simply fights it.
- For an unreachable target the monster moves to the nearest Territory edge point. **The 1.5 s hold starts only once it has arrived there and still cannot contest the target**, so briefly crossing the boundary cannot cause an early reset. If the target becomes reachable first, normal engagement continues.
- Then it **Returns**: still vulnerable, and **hits from outside do not re-engage it**. Only a valid player inside the Territory re-engages. This closes the poke loop that the Warden's re-engage rule would allow.
- On reaching Home: immediate full Health, Statuses cleared, Threat and target cleared, back to Idle. No gradual regeneration, because a monster is disposable and there is no third-party "finish it" window to preserve.
- The reset is the cost of cheesing: a ranged player gets a short free-damage window, and the kill is accepted only if the burst is fast enough.
- **Escalation if evidence demands it:** the Warden's `gate(hit)` veto, applied to monsters only. The monster's `gate` hook already exists for the Collapse check, so it would be a small addition.

**Why not the others.** Option B (the Warden's ward) is heavy-handed across 6 to 10 weak enemies and adds a rejected-hit response nobody asked for. A soft leash needs a larger validated area and drifts toward free-roaming AI.

## Point 6. Core numbers

All tunable. Sized against the real player values: Health 100, walk 16 studs/s, Basic Attack 6 per 0.4 s with reach 8, Tidal Bolt 12, Arc Burst 14 (plus a 16 Wet Reaction), Rift Hook 8.

| Value | Initial |
|---|---|
| Health | 45 |
| Melee damage | 8 |
| Chase speed | 13 studs/s |
| `TerritoryRadius` | about 7 (6 to 9, geometry-derived) |
| `AggroRadius` | `Territory + 4` |
| Reach and shape | 5.5, about 50 degrees. Approach spacing 3.5 |
| Telegraph, Recovery, gap | 0.45 s, 0.8 s rooted, 0.4 s movement allowed, so about 1.65 s per cycle |
| Edge hold | 1.5 s |
| First-attack wake stagger | 0.3 to 0.7 s |

**Why 45 Health.** At 40 the Rank 1 Tidal Bolt into Arc Burst combo (42) kills outright and a Kit-rich player skips the fight. At 45 it leaves 3 Health, so a geared player is strongly advantaged without the encounter being deleted. A Basic-only kill is about 4 to 4.5 s for a competent player (8 hits, a perfect run is 2.8 s).

**Why 8 damage.** One monster does about 5 DPS if every swing lands, a third of the Basic Attack's potential. Damage taken from monsters persists in a one-life Match, since monster kills restore no health, so the cost is real without more damage. Two monsters, or one monster plus an enemy player, becomes meaningful (about 30 to 45 Health over a 6 to 9 s fight), and two monsters plus a player can be lethal.

**Tuning rules.** If monsters feel unavoidable, raise the Telegraph toward 0.55 s before lowering damage. If kills run long, lower Health toward 40 before changing player damage assumptions or the attack cadence. Backstep farming is evidence, not a blocker. Rank 2 Ability values must be read from `Config` before balance is final.

## Point 7. Placement and population

**Settled: 8 monsters, one active Home per outer Shard, from an authored pool of 12.**

- 6 active Homes: **4 singles and 2 pairs**. No player-count scaling. 0 Crown monsters.
- A dedicated `monsterRandom` picks which two Shards get pairs (there is one Home per Shard, so no variants: revised in Ticket 24 step 1b). The selection is logged.
- **Why no Crown monsters.** The Crown terrace is r 14 to 32 and a Home needs a clear circle of about 6 to 7 plus a margin from the lip (15 in the first design, 5 after Step 1b), leaving a band too narrow. The Crown also already carries Sentries and the Final Fight, and the Dais is reserved.
- **Why every Shard.** With 2 players four Shards are unoccupied, and they must stay worth visiting.

**Rules** (revised in Ticket 24 step 1b after a search of the real map: the first values, `max(45, aggro + 20)` from every pad and a 15 stud lip margin, left no legal Home on any Shard; the reward now appears at Home): at least `AggroRadius + 20` from every pad; Home at least 20 from Home Caches; 20 to 32 from the Forward Cache; no overlap with a Warden's aggro circle plus clearance; at least 50 from Crown Sentries; Territory edge at least 5 from any lethal lip; aggro ring clear of connectors and aprons plus 2; an ignoring route from every pad; the whole Territory in exactly one Section.

**Aggro versus loot.** Cache standing spots and drop spreads stay outside the aggro ring. Monsters are around loot without guarding it, and no Cache needs a fight.

**Fallback if a Shard has no valid Home.** Relax in this order: the Forward Cache band first, then the Home may move onto Layer 1 (there is one Home per Shard, no variant B). Never relax spawn safety, Home Cache safety or the lethal-lip margin to force a slot. If a Shard cannot support one valid Home at all, that is a Ticket 21 map-placement contradiction to review locally, never silently dropped from the rule. Layer 2 is preferred (the inward, Forward side, so monsters sit on the inward route as Layer 1 collapses), but not required.

## Point 8. Reward

**Settled: a 12.5% hidden flag per monster, pre-dealt at setup.**

- Budget: about 0.5 expected Ability drops per Match, assuming about half the monsters are killed (4 of 8). Even a full clear is 1.0 expected, below the guaranteed Warden supply (1, 2 or 3 by player count).
- Each monster independently gets a hidden 12.5% flag and a uniformly drawn Ability. One ordinary **Rank 1** Pickup, repeats allowed, since the extras sit outside the 24-drop bag.
- **Pre-deal, not roll at death:** deterministic specs, kill order cannot affect the draw, the reward stays hidden, and the shared `Match`, Cache and Warden streams stay untouched.
- **No hard cap.** At 8 kills the tail of 3 or more is rare and the economy stays small. A hard cap adds a rule to explain and test for little gain. A Match budget is added only if telemetry shows the tail matters.
- No pity, no guaranteed first kill, no killer-specific loot, no XP, currency or Relic, no Kill Recovery, no player-count scaling. Collapse, Fault and Match end give no drop.
- **The old fast-opening rule does not apply.** `ranks-and-pickups.md` says a player with an empty Ability slot gets a guaranteed Pickup from their next eligible monster kill. Regular monsters, like Wardens, are excluded, which keeps this consistent with the no-guarantee decision. Ticket 18 already has no guaranteed-first-kill rule.
- **Honest cost:** with 4 kills there is about a 59% chance of no monster drop in a Match. Most clears pay nothing, which is the role. The risk is "pointless mosquitoes", and `p` is the lever after tester feedback.

## Point 9. Presentation

**Settled: world-first, with no bar and no ring.**

- **Look:** a small, low-profile rift crawler about 3.5 studs long, dark stone with violet-teal seams and an oversized forelimb. It differs from the Warden's tall guardian.
- **No overhead Health bar and no nameplate.** These are weak enemies, so exact remaining Health is not strategically important. Damage shows on the body through three deliberately coarse stages (healthy, wounded, nearly dead at about 66% and 33%), never an exact bar substitute.
- **No Territory ring.** Six or more repeated leash circles add more clutter than value. Hold-and-return body language shows the leash.
- Idle, wake, chase and return read through pose, motion and seams, not colour alone. The wake cue plays during the wake stagger, so players always see "it noticed me" before the first Telegraph.
- The 0.45 s claw wedge uses the shared Telegraph language, lighter and smaller than the Warden's.
- Hit, Evade and Shield are reused. The shared forced-movement-resisted feedback is reused.
- **Death:** a fast collapse. A rewarded monster is **identical to an unrewarded one before death**. A rare Pickup appears at once on a legitimate kill, with the Cache-break reward cue, so other players can notice it. Collapse and Fault removal is a quiet non-kill disappearance. Most no-drop deaths still get a distinct kill-confirm.
- **Status visuals:** if Ticket 23 has already generalised Wet and Reaction visuals to non-player PvE targets, reuse them. If not, do not create a second competing implementation: fix or extract the shared PvE Status presentation once.
- Placeholder audio and VFX only, no uploads. Small-screen readability is blocking.

## Point 10. Telemetry

**Settled: a few low-volume events plus one `MonsterSummary` per Match**, in the existing console `key=value` style, with no positions or movement samples, and failures contained. If the per-Home summary becomes complicated, prioritise the counters that directly answer gameplay questions, and build no aggregation system for eight monsters.

- **Events:** `MonsterDeal` (reconstructs which Home or monster carried which hidden reward), `MonsterKill`, `MonsterPlayerDeath`, `MonsterFault`.
- **Derived evidence:** `accidentalEngagements` and `routeTaxDamage` (route tax), `flawlessKills` (backstep farming), the overlap and burst counters (pairs), `outside*` counters (outside fire), and `finishedAfterPlayerDamage`.
- **Kill attribution is not changed in Ticket 24.** A player killed by a monster is eliminated with no killer, as for Wardens. We revisit it only if playtesting shows monsters frequently steal otherwise legitimate PvP eliminations.
- **Three tiers:** correctness invariants (blocking), balance evidence (observed), and role-level review triggers (initial signals only, not read before about 10 Matches). Do not make balance decisions from tiny samples: combine telemetry with real playtest feedback.

## Accepted risks

- **Pairs may sync into a burst.** Watched through the overlap counters, with the per-target Telegraph cap as the named lever.
- **Outside fire may dominate.** Watched through `outsideKills`, with the Warden-style gate as the named escalation.
- **Layer 2 may not fit every Home.** The fallback order above applies, and a Shard with no valid Home reviews Ticket 21's local geometry.
- **Monsters may feel pointless.** About 59% of 4-kill Matches pay nothing by design. Read the reward counters with tester feedback before moving `p`.
- **A monster finishing a damaged player** costs the first attacker the kill. Logged, not changed.

## Deferred

- A ranged or mixed regular-monster family, and Monster Families with themed Drop Tables.
- Match XP, levels, currency, Relics (Match progression is Milestones, [Ticket 25](milestone-2-tickets.md#25-match-milestones), not XP).
- Respawn, pack AI, cross-Shard pursuit, patrol, a Knockback monster.
- Monsters on the Crown.
- An outside-fire gate, a per-target Telegraph cap and a Match drop budget, all added only on evidence.
- An elite Dais Warden or boss, final art, audio and animation.
