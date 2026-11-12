# Minimap and Objective Awareness: Ticket 27 design history

The record of how Ticket 27 was settled, on 2026-11-11: an initial design pass, then six technical refinement points grilled one at a time with the developer against the real repository, with the options considered, what was chosen and why. The build contract is the Ticket 27 entry in [`milestone-2-tickets.md`](milestone-2-tickets.md#27-minimap-and-objective-awareness). **If this file and the ticket ever differ, the ticket wins.** Status: specified, not built, and staged.

**Terms.** The **Minimap** is the persistent corner view and the **Tactical map** is the expanded view (see `CONTEXT.md`). Section, Warning, Consumed, Last Stand and Riftlight are as in [`rift-collapse-design.md`](rift-collapse-design.md) and `CONTEXT.md`.

**Intent that governed every point.** Solve navigation, objective discovery and Collapse awareness **without making the game feel like a radar-based shooter**. The arena is small (about Ø260), fights are fast, the camera is over-the-shoulder with the mouse locked, and ground disappears on a schedule. So the map is a reading aid: fixed, authored and quiet. It never tells a player where another player is, and where a feature would duplicate what the world or the HUD already says, the feature was cut.

## What the repository check changed

The initial design (from an earlier session) was reviewed against the docs and the Studio place (read-only). Four integration questions were investigated, and they changed the design:

- **Sections, not Shards.** The arena is 15 Sections and 12 connectors, a Shard spans several layers, and connectors are consumed independently with their own layer. The map's unit is the Section.
- **Removed is an absence.** A consumed model is reparented into a server-only holding folder, so a client never reads a `Removed` state. Removed means an authored id whose model is absent while the Collapse runs.
- **No revisions exist** on replicated state, and there is no late joining (only Hub spectators arrive mid-Match). The design's "reject stale updates" has nothing to apply to, and state is derived instead.
- **The game has no touch, gamepad or `Esc` handling at all.** Controls are keyboard and mouse, with the mouse locked to the centre. Everything built on mobile or controller input was building on nothing (Point 1).
- **Spectator leakage is a non-problem for a client-derived map,** because a spectator's client already has everything the world replicates. The real requirement is only to read the `Spectating` target.
- **The 30-stud rule is presentation, not secrecy.** Every Pickup and Cache already replicates to every client.

## Point 1. Platform scope for input

**Options.** A: keyboard and mouse only, with portable layout rules. B: A plus a clickable button. C: build touch and gamepad in this ticket.

**Settled: A.** M toggles, and any combat input closes the map without consuming the action. Touch, gamepad, `Esc` and safe areas wait for a platform-input ticket.

**Why.**
- There was nothing to be compatible with: the audited `Input` handles the mouse, Q, E, F, LeftShift and G only.
- A button is unreachable while the mouse is locked.
- C would hide a platform-support project inside a map feature.
- `Esc` belongs to Roblox's own menu.

**Update 2026-11-12.** That platform-input ticket is now [Ticket 29](milestone-2-tickets.md#29-platform-input-and-mobiletablet-support), settled with the developer, and it does not redesign this feature. On touch, a tap on the Minimap toggles the Tactical map (only at touch end, within a small movement and time limit). The Tactical map never takes touches, and its expanded size is the largest square that fits the free area, not a fixed fraction of the screen. Fire, Q and E, and Dash close it through the same read-only `Input` hook, which touch feeds. The Minimap sits top-left under `CoreUISafeInsets`. Gamepad is still a later ticket, and this ticket's own scope (keyboard and mouse) is unchanged.

## Point 2. What the player marker shows

**Options.** A: one triangle showing the camera's yaw. B: a character-facing triangle plus a camera line past 30 degrees. C: character facing only.

**Settled: A.** The position follows the character, the orientation follows the camera's horizontal yaw, and pitch is ignored.

**Why.**
- No client script sets `AutoRotate` or `RotationType`, so the character most likely turns toward its movement, and a `ChargeTelegraphs` comment says the facing and the aim differ.
- A player orients by the crosshair. A character-facing glyph is just recent movement, and the 30 degree rule would flicker during ordinary strafing.
- One glyph answers the real question.

## Point 3. Cache and dropped Kit markers

**Options.** A: neither. B: Kit drops only, within 30 studs. C: B plus Caches within 30 studs on the Tactical map.

**Settled: B.** A neutral glyph, from `PickupFromElimination`, on both maps, with horizontal distance measured from the active or spectated character at about 5 Hz.

**Why.**
- Cache positions are fixed and learnable, and a Cache within 30 studs is nearly always already in view.
- Cache state (`CacheBroken`, `CacheSealed`) is live information about other players, and the Crown seal would add a case.
- A Kit drop is the one objective that is both transient and Collapse-relevant, and the flag already exists. The maximum vertical difference in the arena is about 5 studs, so 3D distance would change nothing.
- **Caches are deferred**, to be reconsidered only if playtesting shows a discovery problem.

## Point 4. Elevation indicators

**Options.** A: none. B: the Dais and its ramps on the Tactical map. C: B plus ruin platforms.

**Settled: A.** The Dais is its own Section polygon, labelled on the Tactical map.

**Why.**
- Everything walkable is at y = 0 except the Dais (about +5) and small ruin platforms (+3 to +4, about 14 to 20 studs across). None is a route, and none is a barrier. Whole-Shard elevation differences were ruled out on purpose by Ticket 21.
- The Dais face is probably jumpable, so ramps are not chokepoints.
- Ruin platforms and ramps have no Section id to key to, so marking them would need a second geometry source that can drift.

## Point 5. Staging and dependencies

**Options.** A: two stages, Stage 1 after Ticket 22 is accepted and Stage 2 after Ticket 23. B: one stage after Tickets 22 and 23. C: Stage 1 now, in parallel with Ticket 22's remaining runs.

**Settled: A.**

**Why.**
- Stage 1 needs only built work (Tickets 17, 17B, 21 and 22). The Warden marker is a small addendum and must not hold it back.
- C breaks the single Studio driver rule, since Ticket 22's last step needs two clients on the real map, and it risks building against attributes still being validated.
- Ticket 24 has no relationship: Riftlings and Sentries are never displayed.
- Ticket 26 is an optional integration. Its shared accessor is reused when it exists, with no duplicate Collapse model, and Ticket 22's attributes are sufficient without it.

## Point 6. Warden marker visibility and lifecycle

**Options.** A: global from GO, at the live root. B: global from `MatchCrownOpensAt`, at `WardenHome`, presence only. C: discovery-based, with client memory.

**Settled: B.**

**Why.**
- **A leaks.** Which Warden slots are occupied depends on the roster: selection counts occupied pads per Gate, and with 2 players the Warden takes the empty Gate. A global marker at GO therefore gives away the spawn distribution, which is an enemy-position hint. A marker at the live root would also show where a fight is.
- At 60 seconds the occupancy hint is stale, the marker is fixed, and one glyph shows no fight.
- C needs client memory that can go stale, which contradicts "unknown means hidden".
- A disappearing marker may say that a Warden fell. That is accepted objective information, not enemy tracking.

## Decisions carried over from the initial pass, and their review

| Decision | Outcome |
|---|---|
| Hybrid Minimap and Tactical map | Kept. One art at two sizes, non-modal, with no zoom, pan, waypoints or tooltips. |
| Fixed north-up, authored 2D, no live render | Kept. No `ViewportFrame`. The polygons are baked once from the authored footprints, with a structural check against drift. |
| Collapse states Intact, Warning, Removed | Kept. Warning adds a static fracture pattern so colour is never the only channel. |
| **A countdown on the map** | **Dropped by the developer.** The Collapse HUD line and the local danger alert already carry the timing, and a third copy would only add a source to keep in step. The map shows state only and reuses the existing presentation state, so stalls and shifts need no map-side handling. |
| No enemy information of any kind | Kept as a fixed rule. No marker, last-known position, proximity indicator, ping or endgame reveal. |
| Marker priority and overlap suppression | Kept, with the Cache row removed (Point 3). |
| Camera indicator past 30 degrees | Replaced (Point 2). |
| Expanded-map elevation | Cut (Point 4). |
| Esc, mobile tap, D-pad and safe areas | Moved to a later platform-input ticket (Point 1). Touch and safe areas are now [Ticket 29](milestone-2-tickets.md#29-platform-input-and-mobiletablet-support), and the D-pad and `Esc` stay deferred. |
| Update-rate tiers | Collapsed to one tick of about 20 Hz for the marker, with everything else event-driven. |
| Stale-update rejection | Dropped. State is derived and idempotent (see above). |

## Recorded risks

- **The rendering technique is unproven.** There is no `ViewportFrame`, and no asset may be uploaded without approval. The ticket makes a gated spike the first step, with `EditableImage` first only if platform support is verified, and a stop-and-ask if only an upload would work.
- **Connector identity may not be unique.** The documented attributes may not tell the 12 connectors apart, and an additive `ConnectorId` is the named fallback.
- **A fixed north-up map of six similar Shards can disorient.** The `ShardId` labels and the own-Shard mark address it.
- **No real-device mobile pass is possible yet,** because the game has no touch controls. The spike measures a proxy, and the real pass is a gate of the platform-input ticket (now [Ticket 29](milestone-2-tickets.md#29-platform-input-and-mobiletablet-support)).

## Not decided here

- The corner, the sizes and the glyph shapes, which are chosen in Play against the real HUD.
- The north constant's value, checked against the azimuth convention when built.
- Any Cache markers, ramp markers or platform input.
