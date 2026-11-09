# Rift Collapse: Ticket 22 design history

The record of how Ticket 22 was settled: ten points, each grilled with the developer on 2026-11-09, with the options considered, what was chosen and why. The build contract is the Ticket 22 entry in [`milestone-2-tickets.md`](milestone-2-tickets.md#22-rift-collapse). This file explains it. It does not repeat the numbers.

**Terms.** Player-facing layer names are Outer, Middle, Approach and Inner Crown. Internally they are Layers 1–4, and the Dais is Layer 5. "Consumed" means a Section the Collapse has removed from play. The Rift is the absence of ground, never a volume.

**Intent that governed every point.** The Collapse should become the normal way a Match ends, with the Timeout as a technical failsafe. Rhythm: Gather → Clash → Collapse → Final Fight. Players should die because someone beat them, displaced them, or they made a bad positional choice, such as staying on a clearly marked Section until it was consumed. They should not die to an unpredictable event, and the Collapse must not compete with Hook and Tidal displacement.

---

## Point 1. What a collapsing Section does

**Options.** A: a damage zone with the floor kept. B: warn, then Rift damage, then an evacuation grace, then physical removal. C: warn, then immediate removal, with the Section "consumed" (non-solid) rather than falling.

**Settled: C, consumed.** A Section has two gameplay states, Warning and Removed. During Warning it is completely normal. At removal `T` it leaves gameplay in one step.

**Why.**
- A fails the intent. It has no hard boundary, it needs a second kill system, and it makes Stone Guard (a damage Shield) a counter to the environment.
- B is A and C stacked. It buys a little forgiveness that a long Warning gives for free, and it costs two attribution sources and double the tuning.
- C reuses the Void pipeline (kill plane, displacement credit, the inward Kit drop), so a Collapse death is a `Void` death.
- Hook and Tidal keep their meaning: pushing or pulling someone onto marked ground is a real play.

**Locked consequences.**
- No Rift damage, ramp, tick or lethal storm. No Shield or Stone Guard interaction. No Collapse killer or source.
- No post-removal grace, floating platform, rescue timer or teleport. The Warning is the grace.
- Players fight normally until `T`. A player who is Stunned, Rooted or mid-fight at `T` falls. If that proves cheap, the fallback is a future short Unstable stage, not built now.
- **Removed geometry must leave every query**: character collision, Hook sweep, line of sight, Sentry targeting and cover, projectile queries, floor raycasts and Pickup floor validation. Setting `CanCollide = false` is not enough if the part stays queryable.
- Gameplay removal is authoritative. Presentation (cracks, glow, debris) may start before or around `T` but must never make collision ambiguous. No server-simulated falling slabs.

## Point 2. Timeline and pacing

**Options.** A: a fixed global timeline. B: scaled by player count. C: hybrid, adjusted by survivors.

**Settled: A.** One absolute schedule built at GO. 2–6 players share it, because the Arena is the same size, and unused Shards give a small Match extra loot, not a different travel problem.

**Why.**
- Learnability: the rules never change under the player.
- Survivor-based changes are exploitable: pausing lets the final duel stall, and acceleration under-kits the survivors. Neither was kept.
- Scaling gains little for the cost. If 2–3 player Matches show dead time, the first experiment is a roster-band shift of the **first warning only**, with the cadence untouched.

**Rhythm.** A 30 s Warning, then removal, then a 45 s stable period, then the next Warning (a 75 s cadence). The first removal is at 3:30. The Crown opens at 1:00 and stays independent, with a guard so the first Warning is at least about 90 s later. One survivor ends the Match as today and never waits for the Collapse.

**Failsafe.** The old 480 s limit was too close to the Collapse's end. It becomes 600 s, with a validated margin of at least 120 s after the last scheduled event. Gather, Clash, Collapse and Final Fight are derived labels, not Match states.

## Point 3. Warning and telegraph

**Options.** A: world-only. B: HUD plus world. C: global announcement plus local telegraph.

**Settled: B, with a strict hierarchy.** The four questions a player must answer instantly are when the next removal is, whether their ground is affected, how long they have, and which way is safe.

**Why.** A and C cannot show an exact time or work when the player isn't looking at the Section.

**Locked.**
- A local danger alert (only on Warning ground), a world telegraph on every Warning Section, and one global next-event line.
- All Sections of a layer enter Warning together, and the pulse is phase-locked, so "same rhythm" always means "same deadline".
- Three client-derived tiers (30–10, 10–5, 5–0) and thresholds at 30, 10 and 5 s. Per-second ticks are audible only to players on Warning ground.
- Colour is never the only channel. An arrow points toward the Section's `CollapseParent`. Connectors get no arrow.
- The countdown is derived from the authoritative removal timestamp, never from a local timer. Removal is exact, with no hidden grace for replication latency.

## Point 4. Objects on a removed Section

**Options.** A: everything is consumed with the Section. B: static objects vanish and loose loot relocates inward. C: physical fall.

**Settled: A.**

**Why.**
- B turns the Collapse into a free loot courier and rewards waiting at the boundary.
- C needs server physics for Pickups that have no physics.
- Loot left on consumed ground is lost, which is intentional scarcity that ends the Gather economy.

**Ownership.** Static objects (Caches, Sentries, future Warden slots) carry the marker's `SectionId`. Loose Pickups are consumed when their support ray finds no live floor, so no per-Pickup Section id is needed.

**Order at `T`.** Mark the Section removed, remove its static objects (cancelling any Sentry Telegraph and pending fire), sweep loose Pickups, let same-tick interactions see the removed world (removal wins), then players fall.

**The one fix.** A fallen player's Kit drop used `lastGrounded`, which may now be consumed. The existing inward-offset rule now validates against live floor, falls back along the `CollapseParent` chain, and consumes the Kit if no live floor remains. This is the existing death rule made valid against dynamic floor, not a second subsystem.

Projectiles survive their source Section's removal. A future Warden despawns with its Section, with no XP, credit or loot, unless that ticket changes it.

## Point 5. Removal order and simultaneous deaths

**Options.** A: atomic layer removal. B: a deterministic Section sequence. C: micro-stagger.

**Settled: A.** A single non-yielding update makes B observably identical to A, and C breaks the single deadline.

**The hard problem.** Fall physics and network jitter spread deaths over a few hundred milliseconds, even when everyone loses floor at the same instant. Left alone, latency would choose the winner.

**Settled.**
- **A settle window** (`collapseSettleSeconds`, 1.5 s as a measurement-dependent starting value) starts at the **applied** removal time. No Match verdict resolves inside it.
- The window is global, not tied to the cause of death, so it covers two falls, a fall plus a combat death, and one survivor.
- After the window the existing rules apply: one Alive wins, none is `NoSurvivors`. There is no timestamp, kill-count, damage or ping tiebreak, and no co-winners.
- The effective end deadline is the later of the existing end grace and the settle deadline.
- **Safety follows the physical outcome**, not the time a client received the removal, and the server never pre-classifies players as safe or doomed at `T`. With client-controlled movement (ADR-0002), "authoritative" here means the server's observation of validated movement, so the plane crossing is the authoritative record of a fall.
- A player who kills the last opponent and then falls inside the window gets `NoSurvivors`. That is accepted and measured.

The settle window is a correctness fallback, not the way the final Collapse is meant to resolve.

## Point 6. Final Crown, Dais and overtime

**Options.** A: the Inner Crown is the permanent arena. B: Inner Crown, then the Dais as a Last Stand. C: Dais, then a forced resolution (consume or shrink the Dais, an anti-stall pulse or a Sudden Death modifier).

**Settled: B, with A as a configuration fallback.**

**Why.**
- A leaves room to kite, so the Timeout would become a normal ending.
- C needs something ruled out (Rift damage, shrinking rings) or unbuilt (a combat modifier system), and consuming the Dais makes a draw the expected ending.
- On a Ø28 island every player is always within Hook range, so stalling is nearly impossible.

**Locked.**
- Layer 3 removal (6:00) starts the Crown Final Fight. Layer 4 warns at 6:45 and is consumed at 7:15, leaving the Dais.
- **Layer 5 is never consumed.** There is no overtime of any kind.
- The Layer 4 row may be omitted from the schedule, which gives the permanent Inner Crown.
- The Final Fight is Sentry-free: a Sentry's push beside a lethal-edged Dais would be a cheap non-player ring-out.
- Crown loot stays interactable from 6:00 to 7:15, and the Dais has no loot of its own.
- The Dais perimeter stays fully lethal, with no railings or safety lips.
- Stone Guard is weak there, because a Shielded hit still lands its Knockback. That is accepted, and no Collapse-specific Knockback rule is added.
- **Tidal publish gate.** The docs still contradict each other on Tidal's push distance, so Layer 4 may not be enabled in a publish candidate until tap and charged Tidal have been measured on the Dais. The gate is "perform and record", and the result decides whether Layer 4 ships enabled. If a charged hit on a victim within about 6 studs of the Dais centre reliably rings them out from neutral conditions, Layer 4 ships disabled and Tidal is not nerfed inside Ticket 22.

## Point 7. Scheduling architecture and state exposure

**Options.** A: a Match-owned controller. B: a separate service. C: a pure schedule with a stateless helper.

**Settled: a pure `CollapseRules` plus a small `Collapse` server module that `Match` drives.** This follows the existing Rules plus thin server system pattern (`MatchRules`/`Match`, `CacheRules`/`Caches`). Match owns the lifecycle. There is no multi-session framework.

**Locked.**
- The schedule stores a removal time plus a warning duration, so the warning time is always derived and the two can't contradict.
- Absolute times are built once at GO.
- Events apply strictly in order through a cursor, and each is idempotent.
- **Stall handling:** any event applied late shifts it and every later event by the same delay, so the 30 s Warning and 45 s gap survive a stall. The HUD uses effective times. The failsafe margin is checked against the effective last event.
- The settle window starts at the applied removal time and only moves forward.
- A Heartbeat step gives frame-level timing, and the Match gates (`canAct`, `canCollect`, the target filter) and the Sentry loop also `sync()`. Pure geometry queries are within about one frame.
- **Removal is by reparenting the Section into a private holding folder**, not by flipping flags, which is the strongest form of the Point 1 invariant. Restore is two idempotent calls (Results end, and a safety net at Match start) followed by an integrity assertion. A failed assertion refuses Match start.
- Cloning a pristine map per Match was rejected: it moves the map out of its folder, breaks marker discovery and the MCP workflow, and is not needed with one Match at a time.
- **Consequence:** Collapse tests that consume Sections run in Play. An Edit-mode run must never save a half-consumed Arena.

## Point 8. HUD, spectators and readability

**Options.** A: one contextual line in the existing status slot. B: a Match clock plus a separate Collapse line. C: a phase strip with labels.

**Settled: A.**

**Why.** One number always means "when does the ground next change". B adds a second clock, which Point 3 had already demoted. A phase label can be added to the same slot later.

**Locked.**
- The slot keeps the alive count and moves through hold, stable, Warning, between layers, `LAST GROUND`, and the failsafe notice in its last 60 s.
- Gather, Clash, Collapse and Final Fight are telemetry-only in v1. Layer names are Outer, Middle, Approach and Inner Crown, with no layer numbers shown to players.
- The local alert stacks beneath the global line and never overlaps it. Spectators see the global line and world effects only, with no local alert.
- Whole seconds only, rounded up, never `0` or `0:00`. The Crown opening is an event message, not a countdown.
- A small amendment to Point 7: a boolean `CollapseWarning` so the Warning style appears only once the server has actually applied it.
- A pure presenter function maps attributes and time to a presentation state, so the HUD logic can be unit tested.

## Point 9. Telemetry

**Options.** A: events only. B: events plus a compact summary. C: frequent positional sampling.

**Settled: B, plus an event-triggered occupancy cue.**

**Why.** Today's telemetry is console `key=value` lines, and the open questions need event timestamps and Section-level occupancy, not coordinates.

**Locked.**
- Collapse lines use the `[Riftfall][Collapse]` prefix, and every line carries `match=` and `t=` (seconds since GO).
- Existing `Eliminated` and `CrownOpened` lines gain fields, and `MatchEnd` and `MatchResult` stay canonical.
- Displacement records gain `chargeBand`, `hitRadius` and `hitDist`, so Dais ring-outs can be judged without position sampling.
- `CollapseExposed` is an event-triggered sample and never feeds gameplay.
- Telemetry is attempted once, failure is contained, and it never blocks progression or replays a removal.
- There is no periodic coordinates, heatmap, backend or new personal data.

## Point 10. Validation

**Settled.** Fixture-based specs (the Collapse takes its map, holding folder, clock and telemetry sink as dependencies, so specs can never consume the real Rift Crown), structural checks that are safe in Edit mode, an S1 solo matrix, an S2 two-client matrix, latency measurements, and a blocking versus evidence split.

**Notes.**
- Boundary cases are armed against an absolute server timestamp, never clicked through tool latency.
- Survival at about 1 s is recorded, not required.
- The latency pass rule is based on the server's own Void elimination offset plus a 0.25 s margin, not on what a client sees.
- A real six-player Dais fight is evidence only.
- The Tidal gate is blocking to perform and record, but not on a result.
- A mandatory check after a Play test that consumed Sections is that the Edit-mode map is still intact.

---

## Review notes (2026-11-09)

An adversarial pass against Ticket 21, Points 1–10, the existing end and Void rules, the HUD conventions and the Rules plus system pattern found no blocking contradiction. The fixes applied, and the items accepted or deferred, are recorded in the Ticket 22 entry under "Review notes".
