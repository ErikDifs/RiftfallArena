# Rift Collapse visuals and atmosphere: Ticket 26 design history

The record of how Ticket 26 was settled: thirteen points and three blocking decisions, each grilled with the developer on 2026-11-10, with the options considered, what was chosen and why. The build contract is the Ticket 26 entry in [`milestone-2-tickets.md`](milestone-2-tickets.md#26-rift-collapse-visuals-and-atmosphere). **If this file and the ticket ever differ, the ticket wins.** Status: specified, not built, and staged.

**Terms.** The **Unmaking** is the visual identity: the arena is being *erased*. **Riftlight** is the rose-magenta accent. The **Unravel** is a Section's removal presentation. The **Last Anchor** is the one light on the Dais. Layer, Section, Warning, Consumed and Last Stand are as in [`rift-collapse-design.md`](rift-collapse-design.md) and `CONTEXT.md`.

**Intent that governed every point.** Make the arena's destruction one of the game's defining visual experiences without touching fairness, combat readability or performance. **Disappearing ground remains the only Collapse hazard.** Ticket 22's rules (atomic, phase-locked, server-authoritative removal) were never reopened, and every effect is client-side, cosmetic and non-interactive. Where drama and clarity conflicted, clarity won, but the Unmaking must stay dramatic and not be flattened for contrast.

---

## Point 1. Collapse presentation

**Options.** A: ground destruction only. B: a traditional storm or fog wall that advances inward. C: a hybrid, with a global storm that intensifies, plus Warning Sections that destabilise.

**Settled: C, with strict constraints.**

**Why.**
- B contradicts Ticket 22. A wall arriving at different Sections at different times implies per-Section deadlines, breaks phase-locked removal, and teaches players that fog is the hazard.
- A is clear but weak from a distance.
- C reacts as a world while ground removal stays the rule.

**Locked.** The storm is global, schedule-driven and purely atmospheric. It has no advancing wall, no airborne volume and no spatial fog mechanics. Fog is a capped backdrop beyond combat range. Ground warnings always outrank atmosphere.

## Point 2. Visual identity

**Options.** A: Ember Ruin (a burning citadel). B: the Unmaking (erasure, a pale void and one hot accent). C: Dimensional Aurora (aurora ribbons on indigo).

**Settled: B.**

**Why.** Ability colours are already taken (blue, amber, yellow-white), and a generic purple void is what we wanted to avoid. A desaturated backdrop makes every telegraph and Ability pop, and it is the cheapest to render.

**Locked.** Cold, muted stone at the start, progressive bleaching of sky and distance, a pale void, and Riftlight in rose-magenta (not violet) concentrated in fractures, warnings and removal bursts. **No literal white-out in the Last Stand:** dark stone and structural contrast are preserved. Exact palette values are an art-pass decision.

## Point 3. Ground Warning language

**Options.** A: surface fractures only. B: layered fractures, cut-lines and beacons. C: a full-surface overlay sheet.

**Settled: B.**

**Why.** A can't be read from other elevations, and C is a flat coloured area that looks like an Ability telegraph, covers Pickups and overdraws.

**Locked.**
- **Fractures** are jagged, branching, and grow in over the first few seconds.
- **Cut-lines** are jagged and mark where ground will separate.
- **Beacons** are thin and vertical, one per Shard or logical group, not one per Section.
- Three tiers match Ticket 22's windows. The Final tier intensifies fractures, cut-lines and pulse, never the whole Section's brightness.
- The pulse is phase-locked with peaks on whole seconds before removal.
- Jagged and vertical marks distinguish warnings from smooth Ability telegraphs, and baseline seams stay static.
- Mobile keeps fractures and cut-lines, and drops dust and beacons first.

## Point 4. Collapse destruction animation

**Options.** A: shatter and fall as physics debris. B: sink or break away. C: the Unravel (erasure).

**Settled: C.**

**Why.** Ticket 22 removes a Section by reparenting it, so every client loses the real model and its collision in one replication event. A client cannot animate the real slab, and any ghost slab would look standable. A and B also contradict the identity.

**Locked.**
- The client plays the Unravel when it **observes** the Section leave the world, never from a timer, so effect and collision cannot disagree.
- Nothing plays before removal. The ground vanishes completely at frame 0.
- A thin cut-line flash (about 0.2 s) and small fragments (gone in about 1 s) follow, at or below floor level and never obstructing combat.
- Stagger applies to the cosmetics only, never to ground.
- A global fragment budget per removal event degrades fragments first, and the flash is mandatory for every Section.
- A synchronised distant pulse keeps the removal legible from another Shard.

## Point 5. Timing, synchronisation and reliability

**Options.** A: the server sends VFX events. B: a state-derived, edge-detected client controller. C: client-predicted timers.

**Settled: B.**

**Why.** A needs new remotes (the code-structure rule is no RemoteEvent per visual effect) and misses late joiners. C can play an effect while collision still exists, which breaks Point 4.

**Locked.**
- Tier, pulse phase and fracture growth are pure functions of the server clock and replicated timestamps.
- The Unravel is deduped by `(matchId, SectionId)` and never plays for a Section the client never saw, or on restore.
- Section bounds are cached before removal.
- Atmosphere is a target derived from state, so a late joiner never replays history.
- Recovery points replace per-frame reconciliation.
- **Stage comes from the world** (layers with no Sections present), not from `CollapseLayer = 0`. Ticket 22 defines `0` only as a "final-ground sentinel", and with Layer 4 disabled it can mean the Inner Crown.

## Point 6. Atmosphere, lighting and environmental effects

**Options.** A: property tweens only. B: tweens plus a capped procedural Rift horizon. C: an authored sky texture set.

**Settled: B.** C is a later art-pass upgrade, and it needs uploads.

**Locked.**
- Stage presets of Atmosphere and ColorCorrection, with no Blur, DepthOfField, SunRays or strong Bloom.
- A modest 30 s Warning ramp, a removal pulse, then a new baseline.
- The horizon is distant, restrained and non-spatial.
- Everything is client-owned.
- **Ownership:** Ticket 26 owns a named property list and uses compare-and-restore, so it never overwrites another system's changes.

## Point 7. Combat readability, contrast and accessibility

**Options.** A: fixed art constraints with measured tests. B: a dynamic contrast system. C: A plus permanent character outlines.

**Settled: A, with C as a conditional fallback.** B is rejected as complexity.

**Locked.**
- A priority ladder: HUD, players and Abilities, ground warnings, the Unravel, then atmosphere.
- Starting values for void luminance, stone-to-void contrast, haze, HUD contrast and the hue target. Readability testing decides acceptance, and blocking gameplay failures must still be fixed.
- Pulses are at most 3 Hz, with no large-area flashes.
- Colour is never the only channel.
- Everything holds with post-processing disabled.
- The outline fallback must not reveal players through walls, and needs evidence.

## Point 8. Anticipation and environmental storytelling

**Options.** A: global story only. B: truthful beats. C: foreshadowing before the Warning.

**Settled: B.** C is a pre-Warning cue, which Ticket 22 and the developer rule out. It would also create false positives that make players leave early or doubt safe ground.

**Locked.**
- No pre-Warning cue on any Section or place. Anticipation before a Warning is only the global stage and the HUD.
- Instability vocabulary is reserved for Warning Sections. Stable ground never gains wear.
- The horizon records only what has happened.
- The exposed edge reuses Ticket 21's authored edge treatment, and any trim needs test evidence.

## Point 9. The final Dais and the Last Stand

**Options.** A: an unchanged Dais with only the final preset. B: the Last Anchor. C: a transformed Dais (runes, orbiting slabs, a vortex).

**Settled: B.**

**Why.** C clutters a Ø28 combat area and a vortex implies a hazard. B makes the Dais the last lit, fully coloured thing against a bleached world, which is the Unmaking's inversion and costs one light.

**Locked.**
- The Last Stand look begins only after Layer 4 is consumed, and is dormant by data when Layer 4 is disabled.
- One neutral cool-white light, and no Riftlight, runes, decals or geometry on the Dais.
- A clear cylinder around the Dais with nothing above floor level.
- The light is an enhancement. The stone-against-void contrast must carry the identity without it.
- The Last Stand gets its intensity from contrast and scale, not extra brightness or particles.

## Point 10. Audio, camera and screen feedback

**Options.** A: audio-led and camera-neutral. B: audio plus a subtle camera impulse. C: cinematic.

**Settled: A.**

**Why.** The camera aims Hook and Tidal, and removal is exactly when players are repositioning. Shake costs competitive play more than it gives.

**Locked.**
- Global stings at 30 s and 10 s and a removal thump.
- Positional hums and impacts grouped by Shard or group, never per Section. Hums are sustained tones with no ticks.
- The local per-second cue stays exclusive to players on Warning ground.
- A 2 to 3 s draining sound on the final Collapse.
- An eerie, hollow, reality-draining sonic identity.
- Collapse audio sits below combat audio with no ducking system.
- Every cue has a visible equivalent.
- The camera fallback stays unbuilt.

## Point 11. Performance budgets and mobile degradation

**Options.** A: two fixed tiers with hard caps. B: an adaptive runtime manager. C: one budget for everyone.

**Settled: A.**

**Why.** B changes effects under the player and can degrade warnings under load. Warnings never overlap in Ticket 22's 75 s cadence, so the worst case is always one layer event, which makes fixed caps predictable.

**Locked.**
- Full and Lite, chosen once at Match start from graphics-quality signals, with Lite when unknown. Touch alone is not treated as low-end.
- Global per-event budgets and an essentials floor that always wins.
- A fixed drop order, and pooling and prewarming in Countdown.
- A forced-tier override for Studio.
- A budget conflict between the fracture floor and Lite is flagged and resolved before implementation, never silently accepted.

**Initial budgets** (tunable, per layer event):

| Item | Full | Lite |
|---|---|---|
| Warning effect parts | at most 250 | at most 120 |
| Beacons | at most 8 | at most 3 |
| Unravel fragments | at most 150 | at most 40 |
| Horizon emitters | at most 8 | 0 |
| Audio voices | at most 8 | at most 5 |
| Total effect instances | at most 400 | at most 200 |

## Point 12. Arena reset, lifecycle and cleanup

**Options.** A: one scoped client controller. B: per-subsystem reset hooks. C: server-driven reset signals.

**Settled: A.**

**Why.** B misses a subsystem and leaks. C needs a new protocol, which contradicts Point 5.

**Locked.**
- Three contexts: Participant, Spectator or None.
- One `matchId`-keyed state accessor.
- One scope owns every effect, tween, sound, connection, callback and Lighting claim, with epochs guarding delayed callbacks.
- At Match end the animation stops and the final static look **holds through Results**, then restores on leaving the Match context.
- Errors are contained to cosmetic systems with protected boundaries only around lifecycle and family handlers.
- Participant to Spectator keeps the scope.
- **Concurrency:** Lighting is client-local, so one client's Collapse look can't reach another client or the Hub. The real limit is that the replicated state is a single global source, which matches the Milestone 2 one-Match constraint. The accessor is the only preparation, and per-session state stays deferred.

## Point 13. Testing, telemetry and failure handling

**Options.** A: layered tests with specs, matrices and a real device. B: manual QA only. C: a client telemetry framework.

**Settled: A.** C needs new remotes and a backend, which contradict Point 5 and Ticket 22.

**Locked.**
- Pure specs, small fakes, S1 and S2 Play matrices, a capture matrix, and a real low-end phone pass. All of it reuses Ticket 22's infrastructure.
- Concise client console diagnostics with no remotes and no per-frame logs.
- A blocking versus evidence split.
- The real-device pass is a release gate that does not block implementation.
- A written identity checklist and the developer's sign-off verify that the Unmaking is dramatic and recognisable, not merely technically correct.

---

## Blocking decisions from the completeness review

**Decision 1. Placement data for the Warning vocabulary.**
- **Options.** A: bounds-only. B: minimal authored anchors. C: runtime geometry sampling.
- **Settled: B.** Ticket 22 says clients never receive `CollapseParent` or the registry, and "do not build geometry sampling". Bounds are wrong on rings, concave Sections and bridges, and could hang a line over void. Ticket 21 is unbuilt, so adding metadata is cheap now.
- **Locked.**
  - Ordered `CutLine` Attachments per Section, a `VfxGroup` attribute defaulting to `ShardId`, and one `BeaconAnchor` per group per layer.
  - Anchors are generated once at authoring time, reviewed, and checked by additive validator rules.
  - A missing cut-line is omitted and never guessed. Fractures, HUD and audio are preserved. A missing required anchor fails validation before release.
  - The metadata is a presentation requirement on Ticket 21, not a gameplay change.

**Decision 2. Reserved colours and identity conflicts.**
- **Options.** A: strict 40° hue separation everywhere. B: colour roles plus rules for confusable pairs. C: move the identity, or recolour the Warden.
- **Settled: B.** The specified Warden draws its attack telegraphs in hostile red as filled wedges and discs, about 20° to 35° of hue from rose-magenta. The Riftling has violet-teal seams. Red, rose and violet are adjacent, so a universal rule would squeeze Riftlight or force changes to Tickets 23 and 24.
- **Locked.**
  - Riftlight means ground instability and erasure. Hostile red means an incoming attack. Ability colours mean Abilities. Violet-teal is the Riftling.
  - Separation from hostile red rests on shape, placement, motion and luminance. The 40° gap from Ability and Pickup colours is an art target, not a pass rule.
  - A Warden attack and a Collapse warning must never erase each other.
  - Any future Ability using related hues needs explicit visual testing.

**Decision 3. Scope and staging.**
- **Options.** A: one complete ticket. B: one ticket with three gated stages. C: split into several tickets.
- **Settled: B.**
- **Why.** C would split the shared controller, lifecycle and tests across tickets. A has no safe stop point.
- **Locked.**
  - A metadata prerequisite, then Stage 1 (Readable Collapse, a complete playable minimum), Stage 2 (the Unmaking atmosphere) and Stage 3 (the Last Stand).
  - Stage 1 replaces or extends Ticket 22's minimal presentation in place, with no duplicate systems.
  - Each stage has its own checks and regression run, and stops for the developer's approval.
  - Stage 3 stays dormant when Layer 4 is disabled, with no flag.

## Contract summary

The ticket governs. This is the shape of it, for readability.

| Stage | Delivers | Blocking checks |
|---|---|---|
| **Prerequisite** | `CutLine`, `VfxGroup`, `BeaconAnchor` and the validator checks | Required anchors exist and sit on the boundary |
| **1. Readable Collapse** | The scoped controller, the Warning vocabulary, the Unravel, the distant pulse, the stage presets, Full and Lite, placeholder audio, the Results hold and Hub restore | Stage derivation, once-only Unravel, no pre-Warning effects, pulse sync and the 3 Hz cap, non-interactive effects, lifecycle and a 3-Match leak check, isolation, Lite essentials, failure containment, the readability protocol, the implementation checks, and Ticket 21 and 22 regression |
| **2. The Unmaking atmosphere** | The horizon, far-field fragments, dust, the ambience bed, positional hums and the audio identity pass | Caps and cleanup including the horizon, hums with no ticks, a bed below the combat ceiling, the readability protocol rerun, and the developer's identity sign-off |
| **3. The Last Stand** | The Last Anchor, the draining sound and the Last Stand ambience | Dormant with Layer 4 disabled, readable without the light, a six-player Dais capture, and clean transition and cleanup |

**Release gates:** a real low-end mobile Lite pass, the developer's visual identity approval, the Tidal gate (which decides whether Layer 4 and so Stage 3 ships), and approval for any upload.

## Reconciliations written into the ticket

The completeness review found no blocking contradiction, and these were settled by wording:

- **Riftlight appears in exactly four places:** baseline seams, the Warning vocabulary, the Unravel, and the non-spatial horizon, which never uses the Warning rhythm.
- **Stage derivation runs only while Running** and freezes for the Results hold, so the server's restore can't drop the Results look.
- **Hums are sustained tones with no ticks**, so Ticket 22's "no global per-second ticks" and the on-ground cue stay intact.
- **Pulse rates** start at 1, 2 and 2 Hz, with 3 Hz as the validated ceiling.
- **Recovery points** are named, with no per-frame reconcile.
- **Beacons are per group per layer**, because a Shard spans layers.
- **Registration is event-driven**, so late replication is handled.
- **The outline fallback** must check the Highlight budget against Ticket 25's Highlight-style visuals.

## Review notes (2026-11-10)

An adversarial pass against Tickets 21 to 25 found no blocking contradiction and no gameplay change. The fixes applied, and the items accepted or deferred, are recorded in the Ticket 26 entry under "Review notes".
